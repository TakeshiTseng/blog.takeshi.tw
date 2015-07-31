Ryu 簡介
====
----

這邊簡介一下 Ryu 這一套 SDN Framework 的大概內容以及架構。

Ryu 開發群
----
----

Ryu 是來自於日本 NTT 所開發以及設計，針對 SDN 的控制器開發框架（Framework）。

核心的開發人員（Maintainer大概有兩位）：Yamamoto 跟 Fujita 這兩位，他們在 Mailling list 中也常常協助
其他開發人員去解決有關 ryu 的問題。

詳細的內容除了到官方網站查看以外，NTT 官方也有[相關的頁面][2]說明

<img src="http://osrg.github.io/ryu/css/images/LogoSet02.png" width="200" />


Framework? Controller?
----
----

剛剛提到他是一套 **Framework** ，沒有錯，他就是一套 Framework，畢竟官方網站就是這樣
定義的：**Ryu** is a component-based software defined networking **framework**.

但是他其實也包含了 OpenFlow（以及其他部分協定） Controller 的功能，所以大部分都會稱它是一個 **Controller**。

簡單來說，它其實最主要是一個負責派送 Event 的一套框架，上面跑了一支名為 ofp_handler 的應用程式，
透過這一個框架把 OpenFlow Message 送到

架構
----
----

下圖是我個人所理解的 Ryu 整體架構

![ryu architecture][1]

Well, 我已經盡量簡化它的架構，有部分可能沒有這麼的正確，但如果沒有要深入了解的話，其實這樣已經很足夠了 XD。

★ This figure is not accurate, just simplify ryu architecture to let people understand it.

↑ 避免有不懂中文的人看到這一篇，卻導致他們觀念錯誤所標示。

在圖片中我們可以看到，最主要會有一個 `ryu-manager` 去將所有資訊保存，包含每一個 Ryu 應用程式，每一個
Datapath 以及其他 "Context" 這類東西，全部都會放在裡面。

`Ryu App` 中預設會有一個名為 `OFPHandler` 的 Ryu App 去理一些基本的 OpenFlow Message 如 Hello，Feature message
 等等。

再來是 `Datapath` 這一層，其實正確的圖型應該是：OFPHandler 包住 Controller 這一個物件，Controller 物件
用來產生以及儲存 Datapath 物件，Datapath 物件所產生的時機是當有新的 Switch 連上時就換產生。

每一個 Datapath 都是獨立的執行緒，原則上不會互相干擾，當 Switch 傳送非同步訊息給它時，則它就會產生
出事件（Event）物件，並將它送給有註冊的 Ryu App。

舉例來說，Switch 傳送一個 Packet In 訊息時，則換產生一個 EventOFPPacketIn 類別的物件
並將它傳送給有註冊 EventOFPPacketIn 的 Ryu App。

Ryu Event 的細節我會找時間在撰寫另一篇來說明。

最後是底層，原則上 Ryu 會依據設定建立 Stream Server 去接收來自特定埠口（如：6653 port）的連線，
有可能建立的連線是沒有加密的 TCP 連線，或是有加密的 SSL 連線。

Ryu Manager 的使用方式我也會用另一篇來陳述。

參考
----
----

1. [Ryu official website][3]
2. [NTT - ryu][2]
3. [Ryu github repo][4]

[1]: /images/ryu-structure.svg
[2]: https://www.ntt-review.jp/archive/ntttechnical.php?contents=ntr201408fa4_s.html
[3]: http://osrg.github.io/ryu/
[4]: https://github.com/osrg/ryu
