撰寫一支 Ryu App
====
----

由於最近在翻譯 Ryu 官方文件，所以想說依據官方玩見做一些改良並且把它寫成另外一篇教學。

有關 Ryu 應用程式
----

Ryu 這一套 Framework/Controller 不像是其他的 Controller 一樣，開起來就
提供了大量的功能給開發者會是使用者使用，它只有提供一些相當簡單的功能而已，
他們最主要是希望能夠提供需多可以使用的 API 或是「零件」讓開發者能夠用相當簡單
的方式去自行撰寫想要的網路功能。

不過

開始撰寫
----

我們透過 Ryu 應用程式將一個 OpenFlow 交換器轉變成為一個第二層(OSI Layer 2)交換器

開啟文字編輯器，並撰寫以下程式：

.. code-block:: python
   
   from ryu.base import app_manager
   
   class L2Switch(app_manager.RyuApp):
       def __init__(self, *args, **kwargs):
           super(L2Switch, self).__init__(*args, **kwargs)

由於 Ryu 應用程式是使用 python 所撰寫而成，所以你可以儲存成任何名稱以及任何
副檔名，在這個範例中，我們將它儲存成 'l2.py'，並放置在家目錄中。

目前這一支程式並沒有任何的功能，但是他卻是一支完整的 Ryu 應用程式，事實上，
你可以輸入以下指令來執行這一支程式::
   
   % ryu-manager ~/l2.py
   loading app /Users/fujita/l2.py
   instantiating app /Users/fujita/l2.py

在上述程式中我們可以知道，要撰寫一支 Ryu 應用程式，你只需要將你的應用程式類別繼承自 RyuApp 即可。

接者讓我們新增一個可以接收來自所有埠口(port)封包進入事件(Packet in event)的功能，
其程式碼如下

.. code-block:: python
   
   from ryu.base import app_manager
   from ryu.controller import ofp_event
   from ryu.controller.handler import MAIN_DISPATCHER
   from ryu.controller.handler import set_ev_cls
   
   class L2Switch(app_manager.RyuApp):
       def __init__(self, *args, **kwargs):
           super(L2Switch, self).__init__(*args, **kwargs)
   
       @set_ev_cls(ofp_event.EventOFPPacketIn, MAIN_DISPATCHER)
       def packet_in_handler(self, ev):
           msg = ev.msg
           dp = msg.datapath
           ofp = dp.ofproto
           ofp_parser = dp.ofproto_parser
   
           actions = [ofp_parser.OFPActionOutput(ofp.OFPP_FLOOD)]
           out = ofp_parser.OFPPacketOut(
               datapath=dp, buffer_id=msg.buffer_id, in_port=msg.in_port,
               actions=actions)
           dp.send_msg(out)


在 L2Switch 中新增一個叫做 'packet_in_handler' 的方法，這一個方法會在
Ryu 接收到 OpenFlow packet_in 訊息時被呼叫。
我們可以透過 'set_ev_cls' 去讓 Ryu 知道當 packet in 訊息傳入時，
它需要將這一個事件帶入此一方法當中。

註：同一事件可以註冊給多個不同的應用程式以及方法

在 set_ev_cls 中，第一個參數(ev)是這一個事件的類別，該類別定義包含事件所需要
的訊息，當 Ryu 收到該事件所對應到的訊息時，都會透過該類別產生事件，
並去呼叫已註冊的方法。

第二個參數則表示該事件要在交換器的某一個狀態下發送，例如開發者一般來說
在交換器與 Ryu 在完成連接之前不希望收到 packet in 訊息。
使用 'MAIN_DISPATCHER' 可以確保交換器與 Ryu 在完成連接之後才會收到
該訊息。

接下來我們來看一下 'packet_in_handler' 中的一些細節。

* ev.msg 表示 packet_in 訊息的資料

* msg.dp 表示這一個訊息是由哪一個 Datapath(switch) 送過來，我們能夠透過
  這一個物件去與該交換器去做互動。

* dp.ofproto 以及 dp.ofproto_parser 表示了這一個訊息使用了哪一個版本
  的 OpenFlow 協定，以及它的解析器。

* OFPActionOutput 是在 packet_out 訊息中，用於設定它應該要走哪一個交換器的
  埠口(port)出去。這一個應用程式會將一個封包送到所有其他的埠口，因此我們這邊使
  用了 OFPP_FLOOD 這一個常去來設定它的目的地。

* OFPPacketOut 類別用來建立 packet_out 訊息

* 如果你呼叫了在 Datapath 中的 send_msg 方法，並給予 OpenFlow 訊息物件，
  Ryu 會將訊息轉換並且送至該交換器中。


在這邊你完成了你的第一個 Ryu 應用程式，你已經能夠使用這個應用程式去讓網路
能夠以第二層交換器的邏輯運作。

若你覺得 L2 交換器過於笨拙以及簡單，您可以參考其他的
`應用程式範例
<https://github.com/osrg/ryu/blob/master/ryu/app/simple_switch.py>`_

你也可以在
`ryu/app
<https://github.com/osrg/ryu/blob/master/ryu/app/>`_ 
資料夾以及 `綜合測試
<https://github.com/osrg/ryu/blob/master/ryu/tests/integrated/>`_
中學到其他的應用程式以及網路功能的撰寫方式
