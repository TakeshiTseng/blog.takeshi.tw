Ryu Event 運作原理（初始化、傳遞、接收）
====
----

這一篇文章要來提一下有關 Ryu 在 Event 處理上面的一些細節，其實在了解細節之後，會發現 Ryu 處理事件上面還算是蠻簡單的。

這一篇另一個重點是 Ryu 整個初始化的階段，一開始沒有想要寫很多，結果發現越寫月多 XD，於是就納入重點中了。

Ryu Book 簡略說法
----
----

下圖是 [Ryu book][1] 中所付的 Ryu 簡略架構，可以看出它實際運作的過程其實相當的簡單。

<img src="/images/ryu-book-ryu-arch.png" width="600">

其實他的架構有點像是我之前所寫的 Ryu 架構，沒有看過的可以先到[這一篇][2]看看。

原則上就是透過其他的 App 或是在 Ryu 中的函式庫去傳遞 Event 給其他的 App 中的 Queue 使用，這整套系統分為以下幾的
部分：

1. 初始化 Observers
2. 傳遞 Event 給所有（或是特定）的 Observer（App）
3. 接收到該 Event 並且處理


初始化 Observers
----
----

接下來這幾個步驟我將會直接使用 Ryu 裡面所提供的程式原始碼來讓大家了解。

首先我們必須知道 Ryu 他的起始點，一般來說我們使用的 ryu-manager 如下：

    ryu/ryu/cmd/manager.py

接下來來看程式碼，我們主要看幾個關鍵的部份：

<pre><code class="python">app_mgr = AppManager.get_instance()
app_mgr.load_apps(app_lists)
contexts = app_mgr.create_contexts()
services = []
services.extend(app_mgr.instantiate_apps(**contexts))
</code></pre>

在整個 Ryu 的執行階段當中，無論有多少隻 App 在執行，都只會有一個 AppManager 的 instance 存在，所以
要取得 AppManager 的實體就必須呼叫 AppManager.get_instance() 才行。

接著是將所有的 App 都載入到 App Manager 當中，在 load_apps 中做了以下幾件事情：

1. 將 App 列表第一個 App pop 出來，並檢查一個它是否已經存在於其他 App 的 Contexts 中
（詳見[撰寫一支 Ryu App][3]），若有，則忽略它。
2. 尋找該 module（ex: ryu.app.simple_switch）中的「第一隻」App 的「Class」並把它塞到 applications_cls(dict) 中。
3. 將剛剛找到的類別裡面的 CONTEXTS 抽出，並且塞到 App Manager 的 contexts_cls 中（用於第一項檢查）
，並且檢查 CONTEXTS 是否有跟其他人使用的重複（例：只能有一個 App 使用 wsgi 模組）
4. 檢查該 App 以及該 App 的 Contexts App 是否有用到一些會自動呼叫 register_service 的 event，舉例來說
所有 ofp_event 下的 Event 都會註冊 ofp_handler，一但有任何人註冊該事件，則 Ryu 在啟動時會自動把 ofp_handler
加入欲執行的清單當中，這也是為什麼我們不需要自己開 ofp_handler 的原因（當然，你要開也不是不行）
5. 最後，將還未處理的 App （例如剛剛從 CONTEXTS 以及第 4 點所找到的東西）再次塞回 App 列表中，並回到步驟 1 直到
完全沒有為止。

load_app 處理完之後，接者是
<pre><code class="python">contexts = app_mgr.create_contexts()</code></pre>

這一個步驟比較簡單，主要是初始化剛剛所講的 CONTEXTS 中的東西，判斷一個 Context 是否為 RyuApp，如果是的話就
使用 Ryu Manager 中的 _instantiate 這一個方法去初始化它，_instantiate 中做了幾件事情：

1. 若該 App 有包含了 OFP_VERSIONS 屬性，則進行檢查該版本是否存在。
2. 檢查該 App 是否已經存在 Manager 的 applications 屬性中。
3. 若無，則使用該 App 的建構子進行初始化並且把一些相關參數帶入（args, kwargs）
4. 執行 register_app
    * 將 App 放入一個名為 SERVICE_BRICKS 的 dict 當中，這一個變數是用於存放所有 App(Servics)用，未來要尋找
相關的 App(service)時就會經過它。
    * 將該 App 中找到它有註冊哪一些 event，並且設定它的 Event Handler method。（RyuApp 中的 event_handlers 屬性）
5. 把初始化好的 App 塞到 applications 中（App 名稱 -> App instance）
6. 回傳這一個 App instance

回到 create_contexts 這邊，初始化完所有 contexts 之後，會把這些 contexts 回傳（return）出去。

最後一個是：

<pre><code class="python">services.extend(app_mgr.instantiate_apps(**contexts))</code></pre>

這一個的用途是將所有在 Manger 中的 applications_cls 取出並且呼叫 _instantiate 方法取初始化他們，
其中他們的參數跟 context 那一個階段有一些些的不同，不同點如下：

1. 這邊會帶入 App 的名稱，context 那邊只會帶入 None。
2. 這邊會帶入 args 以及 kwargs 這兩個參數，其中 kwargs 就是我們的 contexts，所以我們可以在 App 建構子
當中直接使用定義好的名稱去取得他們的實體。


除了初始化 App 以外，它也做了以下幾件事情：

1. _update_bricks：將所有 Event 傳送者以及 Event Handler（Observer）連接上，也就是說
所有會發送 Event 的 Ryu App 都會依據不同的 Event 有一個接收者的清單，舉例來說 SimpleSwitch 
註冊了 EventOFPPacketIn，則 ofp_handler 在這一個 Event 下面的其中一個接收者就會有它的 Handle method。
2. report_brick：Debug 用，會將每一個關係印出。
3. 由於給一個 RyuApp 都是一個 Thread，所以在這邊會呼叫他們的 start 方法，並且把產生好的 Thread 物件放置
到一個 list 中並回傳出去。

所有的 Ryu App Thread 產生完成之後，就會由 Eventlet 去對所有的 Thread 做出 Join 的動作，等待所有 App 都結束
才會停止 Ryu Manager。

初始化到這邊，接下來是 Event 傳遞以及接收並處理的部分，原則上比上面的簡單許多 XD


Event 傳遞、接收與處理
----
----

[1]: http://osrg.github.io/ryu/resources.html
[2]: /article/ryu_intro.html
[3]: /article/write_ryu_app.html

