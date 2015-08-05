Ryu 在 Query 含有大量 flow 的 switch 所出現的問題以及解法
====
----

今天在 Ryu mailing list 中看到有人提出了一個問題：

[RYU can't get more than 21K flow entry to curl client][1]

<pre><code class="plain">Dear All,  
i'm using RYU v3.19 to test Noviflow switch in lab.  
[ something i did ]  
    1. push 21,000 flow through RYU to Novi switch.  
    2. using curl to query those 21,000 flow.  
during my test,  
install/get 18000 entry is OK, but with 21,000 flow, ryu can't get all 21,000 entry.  
tcpdump on RYU,  
can see Novi send much MultiPart_Reply to RYU.  
looks like ryu didn't return results to CURL client.  
i got empty list, below.
（下略）</code></pre>

用簡短的說法就是他裝了一大堆的 Flow Entry 在他的 Switch 中，然後用 Ryu REST API 去取得
那一個 Switch 中的 Flow Entry，結果出來的內容卻是錯的。

我一開始想到的是之前聽 Rascov 說 ONOS 在有大量的 Flow Entry 時會發生錯誤，他主要問題是
大量的 Query 導致網路流量被塞滿，然後導致 OpenFlow 的 Echo 機制出了問題，但是後來看了一下信件所附的
pcap 以及 debug message 檔案，發現並不是這麼一回事。

於是我就從 REST API 這邊開始去 trace 看問題點釋出在哪邊，順便學一下 ofctl 實作的方法。

ofctl 主要有幾個東西是我們需要知道的：

1. waiter: 用於接收某一個指定的 Query（透過 xid 去區別）
2. lock: 實際上是一個 hub.Event() 實體，用於等待一段時間（timeout，長度預設為一秒）
3. msgs: 用於保存接收到的 reply

在呼叫 ofctl_v1_x.get_flow_stats 時，會將 waiter 帶入，要注意的是這一個 waiter 在這一支
應用程式中只會有一個，他所儲存的內容是一個 dict 資料結構，大致的內容如下：

waiters -> {dpid: waiter} : 每一個 switch 都會有對用的 waiter

waiter -> {xid: (lock, msgs)} : 

每一個事件回應的 xid 都是在送出事件是就決定好了

因此可以用 xid 推論回送出的事件，並且找出他的回應應該要存在哪邊（msgs），以及決定該
事件的 lock

這一個 lock 有兩個用途：

* 用於等待時間(timeout)
* 另一個是當這一個 lock 如果有被設定（lock.is_set()）的話，則表示該事件有正常的結束回應（收到一個 flag 為 0 的 reply），否則
就將該 waiter 直接移除，隨後相同 xid 的 reply _將不會在被收到_

get_flow_status 中會呼叫一個名為 send_stats_request 的 method，他的內容如下：

<pre><code class="python">def send_stats_request(dp, stats, waiters, msgs):
    dp.set_xid(stats)
    waiters_per_dp = waiters.setdefault(dp.id, {})
    lock = hub.Event()
    waiters_per_dp[stats.xid] = (lock, msgs)
    dp.send_msg(stats)

    lock.wait(timeout=DEFAULT_TIMEOUT)
    if not lock.is_set():
        del waiters_per_dp[stats.xid]</code></pre>
        
參數說明如下：

1. dp: 表示要送往的 Switch
2. stats: 一個 OpenFlow message，在這邊給予 xid
3. waiters: 同剛剛的說明，他會把 lock, msgs, xid 放到這裡面
4. msgs: 用來保存接收到的訊息

再來他會將該設定的東西設定好，例如 waiters 裡面會需要有 dpid -> waiter -> xid -> (lock, msgs) 這樣的資訊
，並且會給予要送出去的訊息一個 xid，再來就是建立 lock。

接者他就使用 lock.wait 去等待，最多等待一秒鐘。

_問題來了_ ，假設這一個 lock 等待了一秒鐘，但是 switch 要送的東西 _還沒有送完_ ，這樣會發生什麼事情？

剛剛有提到，當時間到了，如果 lock.is_set() 為否的話，waiter 將會被刪除，導致送回來的訊息不會在被接收以及儲存，
我們可以從原始郵件中的 pcap 檔案看到他執行超過一秒鐘，所以導致了這一個問題發生。

<img src="/images/ryu-flow-query-problem.png" width="600" />

解決辦法
----
----

* 最簡單的解決辦法就是將 DEFAULT_TIMEOUT 調長，讓他有充分的時間可以把所有的資訊接收完畢，但是
這樣會發生一個問題，就是當網路出現了問題導致他傳送相當的緩慢，舉例來說每次傳送間隔都是幾秒鐘，
在這樣的情況下我們也許會捨棄這一次的查詢，以避免整個應用程式被我們卡住，但我們又把 Timeout
調長了，這樣一次的查詢就是好幾秒鐘這麼久，讓整個程式相當的沒有效率。

* 另一種解法就是，在每一次的 reply 時，把目前 lock 的 timeout 時間重設，但是 timeout 還是一樣的短，
這樣一來查詢的時間就比較貼近查詢的量。


目前情況
----
----

在對方將 timeout 調長之後，問題就解決了，但是我會希望用第二種解法會比較好。

<pre><code class="plain">HI Sir,

yes!
after change timeout to 5s, i can get all flows.

Thanks for great help

Mark</code></pre>


[1]: http://sourceforge.net/p/ryu/mailman/message/34347766/
