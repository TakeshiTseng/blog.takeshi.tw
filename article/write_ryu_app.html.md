撰寫一支 Ryu App
====
----

由於最近在翻譯 Ryu 官方文件，所以想說依據官方文件做一些改良並且把它寫成另外一篇教學。

有關 Ryu 應用程式
----

Ryu 這一套 Framework/Controller 不像是其他的 Controller 一樣，開起來就
提供了大量的功能給開發者會是使用者使用，它只有提供一些相當簡單的功能而已，
他們最主要是希望能夠提供需多可以使用的 API 或是「零件」讓開發者能夠用相當簡單
的方式去自行撰寫想要的網路功能。

不過 ryu 本身也有提供相當多的範例可以使用，例如 simple_xxxxx、rest_xxxxx 等等，
但是多數的情況未必是開發者想要的，所以一般而言不會直接拿官方範例作為一般用途。

Ryu 應用程式區塊
----
----

我認為 Ryu 的應用程式分為幾個部分可以拿出來探討：

 * 初始化帶入的東西
   * CONTEXTS
   * EVENT
   * OFP_VERSIONS
 * 各式各樣的 Event Handler
 * 自定義的 Function


Ryu 應用程式類別
----
----

原則上每一個 Ryu 應用程式都必須繼承自 RyuAPP 這一個類別，如下：

<pre><code class="python">from ryu.base import app_manager import RyuApp

class MyRyuApp(app_manager.RyuApp):
    def __init__(self, *args, **kwargs):
        super(MyRyuApp, self).__init__(*args, **kwargs)

</code></pre>

OFP_VERSIONS
----
----

接下來我們可以在裡面加入 OFP_VERSIONS 這一項給 Class 的屬性，這一個屬性表示了這一個 RyuApp 支援的
OpenFlow 版本

<pre><code class="python">OFP_VERSIONS = [ofproto_v1_3.OFP_VERSION]</code></pre>

若 Switch 的 OpenFlow 版本不符合 App 的話，則 Ryu 會顯示錯誤訊息。

CONTEXTS
----
----

在 Ryu 的應用程式中，有時候會使用到額外的函式庫或是而外的應用程式，此時我們可以將他放在 CONTEXTS 中
，放在 CONTEXTS 中的類別，在 Ryu manager 載入時，會將他們初始化並且將他們放入建構子中的 kwargs
中，CONTEXTS 是一個 dict 資料型態，key 表示了 kwargs 中對用的 key，而value 則是類別。

如果放在 CONTEXTS 中的類別剛好是一個 RyuApp 的子類別話，則他除了產生物件之外，還會被放到 app_manager
的 Service brick 中，這樣一來，他也會去處理他有註冊的事件。

註：在不同的 Ryu 應用程式中，CONTEXTS名稱 _不可以相同_

以下是 CONTEXT 寫法：

<pre><code class="python">_CONTEXTS = {
    'some_thing': SomeThing
}
</code></pre>

如果 SomeThing 不是一個 RyuApp 的子類別的話，則他的效果 _類似_ 這樣

<pre><code class="python">kwargs['some_thing'] = SomeThing()</code></pre>


EVENT
----
----

在 Ryu 的官方文件有提到，每一個 App 都可以自定 Event 並且傳送給其他有註冊的的 Ryu 應用程式，
但是 Ryu manager 並不會去掃描每一行程式看你這一支程式是否會送出特定的 Event，在這邊有兩種作法
能夠讓 Ryu manager 知道某一支 Ryu 應用程式會送出特定的 Event。

第一種就是將 Event 跟 Ryu App 放置在相同的 python 檔案下面，並且在該程式的建構子中將他的
name 屬性設定成為該檔案（模組）的名稱。

舉例來說 snort lib 這一支 RyuApp

https://github.com/osrg/ryu/blob/master/ryu/lib/snortlib.py#L40

<pre><code class="python">class EventAlert(event.EventBase):
    def __init__(self, msg):
        super(EventAlert, self).__init__()
        self.msg = msg


class SnortLib(app_manager.RyuApp):

    def __init__(self):
        super(SnortLib, self).__init__()
        self.name = 'snortlib'
        self.config = {'unixsock': True}
        self._set_logger()</code></pre>
        
        
Ryu manager 會自動的去認定說該 App 會送出 EventAlert 這一個事件

第二中方法是在類別中撰寫 EVENT 資訊，會這樣寫著要是因為 Event 類別跟 Ryu App 放在
不同的 module 中，或是開發者避免去修改到 name 屬性以免影響其他事件傳送。

EVNET 寫法比較單純，只需要在類別中加入 EVENT 變數即可。

<pre><code class="python">_EVENTS = [EventBlaBlaBla]</code></pre>

這樣一來 manager 就會認定這一支 Ryu App 在執行過程中可能會送出 EventBlaBlaBla 事件類別所產生的事件。


Event handler
----
----

接下來另一個應用程式的主要部分就是處理事件的處理器（Handler），Ryu 這一套架構本身可以視為一個事件分配器
，他會協助不同的應用程式傳遞事件。

每一個應用程式都必須要事先註冊好他想要處理的事件，而 OpenFlow Message 所產生的事件只是其中一部分而已。

補充：在一個 OpenFlow 訊息送至 Controller 後，所產生的事件 _並不會_ 依照特定的順序傳送給有註冊的 App，
其他的控制器如 Floodlight 有實作事件傳遞的順序，而 Ryu 並沒有實作，但是 _他是有辦法_ 實作出這樣的功能
，只是 _目前沒有人做_ 而已。

補充2：Ryu 會將訊息傳遞至 _所有_ 有註冊該訊息的應用程式，開發者不能限制某個訊息在經過某個 App 後就停止傳遞
，同上一個補充，Floodlight 或是 ONOS 可以做到。


要將一個類別中的方法註冊成為 Event Handler，只需要在該方法前加入一個 python decorator 即可，
如下（節錄自 simple_switch_13.py）：

<pre><code class="python">from ryu.controller.handler import set_ev_cls
# skip.....
@set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
def _packet_in_handler(self, ev):
    # do something....
</code></pre>

每一個 Event Handler 中都會有一個參數（ev），這一個參數會是在 decorator 中第一個參數類別所產生的
事件實體，而 set_ev_cls 中第二個表示了要接收哪一個階段的事件，舉例來說如果想要處理在交握階段（Hand shake）
的事件，後方的參數就會是 HANDSHAKE_DISPATCHER，當然，可以使用 list 作為第二個參數以表示多個
階段的事件監聽。


自定義 function
----
----

這部份其實沒有什麼好說明，一般而言除了 event handler 以外，通常在處理複雜的網路規則時，不會把
所有的東西都塞在同一個 method 中（至少我是這樣），這會讓程式變得複雜且難維護。


參考
----
----

1. [Ryu documant][1]
2. [Ryu Book][2]

[1]: http://ryu.readthedocs.org/en/latest/
[2]: http://osrg.github.io/ryu-book/zh_tw/html/


