Ryu Event 運作原理
====
----

這一篇文章要來提一下有關 Ryu 在 Event 處理上面的一些細節，其實在了解細節之後，會發現 Ryu 處理事件上面還算是蠻簡單的。


Ryu Book 簡略說法
----
----

下圖是 [Ryu book][1] 中所付的 Ryu 簡略架構，可以看出它實際運作的過程其實相當的簡單。

<img src="/images/ryu-book-ryu-arch.png" width="600">

其實他的架構有點像是我之前所寫的 Ryu 架構，沒有看過的可以先到[這一篇][2]看看。

原則上就是透過其他的 App 或是在 Ryu 中的函式庫去傳遞 Event 給其他的 App 中的 Queue 使用，這整套系統分為以下幾的
部分：

1. 初始化 observers
2. 傳遞 Event 給所有（或是特定）的 Observer（App）
3. 接收到該 Event 並且處理



[1]: http://osrg.github.io/ryu/resources.html
[2]: /article/ryu_intro.html
