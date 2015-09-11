Ryu Application Dynamic Loader 實作 -- 簡介
====
----

<iframe src="//www.slideshare.net/slideshow/embed_code/key/jAyBwNCnH7q6Nz" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>

Ryu Dynamic Loader 這一個 Project 是我在參與交大 OpenSource 計畫時開始撰寫的，
在上一個學期中只有把基礎架構跟基本功能訂定出來，也因為要寫這一個 Project，仔細的專研了
一下 Ryu 核心程式，發現其實沒有這麼困難，要實作一個能夠動態載入的功能只是時間問題。

其實在很久以前就有人在 [Mailing List][1] 中詢問

安裝 App
----
----

依據 ryu-manager cli 中的內容，我們可以分析出在安裝時有幾個步驟

<img src="/images/ryu_load_apps.png" />

一開始認為只需要照這些步驟做就可以把 App 安裝好了，但是事情沒有這麼單純。

至於為什麼「一次載入全部」跟「後來在載入一個」會不一樣？，主要是在於載入
每一個 _CONTEXTS 那部分，Ryu 會先把所有的 class 掃過，並且把所有的 Context
放到一個列表當中，這一個列表混和了所有跟 RyuApp 相關的類別，他們稱為「Service」

而安裝一個 App 會更新到的資料如下：

1. SERVICE_BRICKS
2. applications
3. contexts

因此，我們在實作 install 的時候只需要更新這些東西並啟動他就可以了

以 Network Operating System 的觀點來看就好像是在某個系統上安裝應用程式，
然後再啟動他這樣。

其他有關安裝、初始化以及啟動的部份，在[這一篇][2]已經有詳細說明了，在這邊不再闡述

解除安裝 App
----
----

這部份比較複雜，主要是因為官方並沒有 _正式提供_ 解除安裝的功能，所以需要仔細去研究

原本我以為只需要呼叫 manager 中的 uninstantiate 就好了，但事實不是這樣 XD

以下是解除安裝的一些步驟：

1. 從 app manager 中得 applications（dict）中找出要移除的 app instance
2. 呼叫 app manager 的 uninstantiate 方法，參數為該 app 的名稱（app.name）
3. 停止 app 的 thread（app.stop()）
4. 取得該 app 中的 contexts 並且關閉及回收 _沒有其他 App 在使用的 context_

這邊補充一下第四點，這邊所指的回收其實是要從三個地方去移除他，就是我上方所寫的：

* SERVICE_BRICKS
* applications
* contexts

必須要把它從這些地方移除掉，且必須要交叉比對卻人沒有其他的 App 沒有使用才行

上述就是整個 Dynamic application loader 的核心功能

目前這一個計畫已經改名為 Dragon Knight 了，為一個獨立的計畫，他不只包含了
安裝以及解除安裝的功能，也包含了：

1. Show bricks：顯示目前 event 關係
2. Ascii topology：在文字介面上顯示拓樸

這幾項功能，不過目前還是有一些 Bug 存在，這些 Bug 比較不好處理

舉例來說，假設 Ryu manager 已經開啟好且執行一端時間了，較表示目前的「狀態」
已經從 Handshake、Config 變成了 Main

如果今天要安裝一個會去監聽（Observe）在 Config 階段的 Ryu app則會導致他沒有
辦法抓到，最簡單的例子就是 simple switch for openflow 1.3 這一個版本，
他一開始會先接收 EventSwitchFeature 並把預設的 flow 安裝進去，但是如果是後來
才安裝他進去的話，就會導致他不會安裝預設的 flow entry，table miss 時就不會
有 packet in message。

Dragon Knight 的原始碼可以到這邊查看：

<https://github.com/Ryu-Dragon-Knight/Dragon-Knight>

[1]: http://sourceforge.net/p/ryu/mailman/search/?q=dynamic
[2]: http://blog.takeshi.tw/article/ryu_event_intro.html

