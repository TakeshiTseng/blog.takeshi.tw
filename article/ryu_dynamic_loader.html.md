Ryu Dynamic Loader 實作 -- 簡介
====
----

<iframe src="//www.slideshare.net/slideshow/embed_code/key/jAyBwNCnH7q6Nz" width="425" height="355" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe>

Ryu Dynamic Loader 這一個 Project 是我在參與交大 OpenSource 計畫時開始撰寫的，
在上一個學期中只有把基礎架構跟基本功能訂定出來，也因為要寫這一個 Project，仔細的專研了
一下 Ryu 核心程式，發現其實沒有這麼困難，要實作一個能夠動態載入的功能只是時間問題。

其實在很久以前就有人在 [Mailing List][1] 中詢問

基本知識
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




[1]: http://sourceforge.net/p/ryu/mailman/search/?q=dynamic
