Ryu Event handler Block 問題以及解決方式
====
----

在[上一篇][1]有提到 Ryu 的 Event 傳遞以及呼叫的方式，這邊我們分析 Ryu event handler 有什麼樣的問題。


問題
----
----

依據官方說法：

    while an event handler is blocked, no further events for the 
    Ryu application will be processed.

所以當一個 Event Handler 裡面出現 block 的問題時，會導致整個 Ryu App 被卡住，其他的程式
送給他 Event 他都不會回應。

來看一下問題出現的點，下面程式出自於 app_manager.py 中的 _event_loop 方法：

<pre><code class="python">def _event_loop(self):
    while self.is_active or not self.events.empty():
        ev, state = self.events.get()
        if ev == self._event_stop:
            continue
        handlers = self.get_handlers(ev, state)
        for handler in handlers:
            handler(ev)</code></pre>
            
[上一篇][1]也有提到相關的流程，只要執行 handler 的部份就是最下面的``handler(ev)``這一個呼叫，
而如果 handler 中的程式卡住的話，就不會繼續執行下一個 handler，也不會去從 event queue 中再去拿取
新的 event。


解決辦法
----
----

要解決這種問題有兩種方法：

1. 開發者自己要注意不能有 Block
2. ryu 自行將太久的 handler 中斷掉

第一個方法是官方建議的，而第二種方法要修改到 ryu 的核心程式，這部份我已經修改完成了，但還不確定要不要送
Patch 給 Ryu，畢竟直接把 handler 中斷掉的話有可能會造成無法預期的錯誤。

修改的方法如下：

1. 在 set_ev_cls 中加入一個 timeout 的參數，預設一秒鐘
2. 修改 _event_loop 方法，在 handler(ev) 這一行加入 timeout，Eventlet 有提供 Timeout 的功能可以使用

_event_loop 修改之後如下

<pre><code class="python">def _event_loop(self):
    while self.is_active or not self.events.empty():
        ev, state = self.events.get()
        if ev == self._event_stop:
            continue
        handlers = self.get_handlers(ev, state)

        for handler in handlers:
            caller = handler.callers.get(ev.__class__, None)

            if not caller.timeout or not caller:
                handler(ev)
                continue

            try:
                with hub.Timeout(caller.timeout):
                    handler(ev)

            except hub.Timeout:
                self.logger.debug("Event handler %s timeout", handler)</code></pre>

在程式中，我們可以看到我們從 handler 透過 event class 中取出一個名為 caller 的物件，這一個物件
包含了我們在 set_ev_cls 的屬性，例如 event class, dispatcher 以及 timeout。

透過 with hub.Timeout(caller.timeout) 這一行，就可以保證 handler 他只會執行最多 timeout 秒鐘
，如果時間超過，則他會拋出 Timeout Exception。

詳細的修改請見[這一個 Commit][2]



[1]: /article/ryu_event_intro.html
[2]: https://github.com/TakeshiTseng/ryu/commit/aa75f116111239ecba6b49601f0c6ad5e54aace3