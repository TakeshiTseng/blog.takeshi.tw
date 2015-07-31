Epoch - SDN simulator for Mac OSX
====

緣起
----

由於最近在 Ryu 的 [Mailling List][1] 中有人貼了一個[小型版本的 SDN Switch][3]，所以我就看了一下他們公司的網頁

沒想到卻發現了 [Epoch][0] 這一個給 Mac OSX 用的 SDN Simulator XD

對於使用 Mac 的開發者算是一大福音，畢竟如果我們這些人要在 Mac 上面模擬 SDN 網路的話，我個人的作法是
開一個 Linux 虛擬機器，然後在裡面開 [mininet][2] 然後再把它連到我的電腦上的 Controller 才行。

這個過程算是蠻繁瑣的，如果有一個可以直接跑在 Mac 上面的 SDN 網路模擬器可以省下很多時間跟資源。

安裝 TunTap
----

首先我們需要安裝的是 [tuntap][4] 這一套 Driver，Epoch 透過它來讓 Mac 與自己建立的 SDN 拓樸連接，
可以注意的是，這一套 Driver 好像有點久了？就連 [Sourceforge][5] 上面的連結好像都掛了 XD

安裝完 tuntap 之後需要重新開機，OSX 10.10 以上的需要注意要把 tun 跟 tap 都重開

> sudo /Library/StartupItems/tun/tun start

> sudo /Library/StartupItems/tap/tap start

重開完成之後，應該會看到 /dev 下面應該會有出現很多個 tun 跟 tap 開頭的東西


> $ ls /dev/tun*  
/dev/tun0  /dev/tun11 /dev/tun14 /dev/tun3  /dev/tun6  /dev/tun9  
/dev/tun1  /dev/tun12 /dev/tun15 /dev/tun4  /dev/tun7  
/dev/tun10 /dev/tun13 /dev/tun2  /dev/tun5  /dev/tun8  

> $ ls /dev/tap*  
/dev/tap0  /dev/tap11 /dev/tap14 /dev/tap3  /dev/tap6  /dev/tap9  
/dev/tap1  /dev/tap12 /dev/tap15 /dev/tap4  /dev/tap7  
/dev/tap10 /dev/tap13 /dev/tap2  /dev/tap5  /dev/tap8  

在未來我們會用到 tap0

安裝 Epoch
----

安裝完 tuntap 之後，就是安裝 Epoch 了

目前 Epoch 放在 HP SD App Store 裡面，仔細看一下，裡面大多數的 App 是針對 HP 自行設計的 Controller，
也有針對 OpenDayLight 所設計的 App。

<img src="/images/sdn-app-store.png" width="600" />

[這邊][6] 是 Epoch for Mac OSX 的下載連結，它除了給 Mac OSX 以外，也有提供 Linux 的版本
（但是應該不會筆 Mininet 好用就是了）。

下載的時候需要註冊 HP SDN App Store 的帳號......

我也不確定擅自把它的程式放到這邊妥不妥當，所以就請大家自行下載吧 >_<

下載完之後解壓縮即可，無需安裝

在資料夾中會發現他的東西真的非常非常少（以 mininet 來做對比），真懷疑它能不能運作 XD

> $ tree  
.  
├── ConfigGen  
├── EULA.txt  
├── Epoch  
├── Examples  
│   ├── Basic_Network.cfg  
│   └── Simple_Network.cfg  
├── Readme.txt  
├── StartEpoch.sh  
└── tuntap_20111101.tar.gz  

網路拓樸設定
----

在這一篇教學桌會用的得東西只有兩個：`StartEpoch.sh` 以及 `Simple_Network.cfg`

StartEpoch.sh 中只有執行一行指令

> sudo ./Epoch -d -n Examples/Simple_Network.cfg

其實他在 Readme 裡面提供的[網址][7]有說明

`-d` 表示 low debug mode

`-D` 則是 high debug mode，我們在這邊切換成這個，因為可以看到比較多東西

然後 -n 後方加上一個 cfg 檔案表示要跑的網路拓樸

再來是 Simple_Network.cfg 的結構，依據[說明][7]，可以知道以下資訊

`<Serial>` 標籤好像是之後不是 Beta 版所需要提供的，似乎未來會需要付錢？

`<Switches>` 是有哪些 Switch 在網路中，參數依序為：名稱、MAC Address、IP Address，Controller IP，Controller Port

`<Hosts>` 則是有哪些主機，參數依序為：名稱，MAC Address，IP Address，要連接的 Switch，該 Switch 的 Port

`<Link>` 是 Switch 與 Switch 的連結，參數依序為：Switch A 名稱，Switch A Port，Switch B 名稱，Switch B Port

`Gateway` 這邊表示要接上的 tuntap device，當他接上之後，你會發現`ifconfig`會多出一個有 IP 的 tapX 網路介面，此時
就可以透過它與該網路連線。

它的參數依序為：Gateway 名稱，Tuntap ID（用哪個Tap Device），IP Address，Netmask，要連上哪一台 Switch，Switch Port

Simple_Network.cfg 裡面是一個單一 Switch 連接 8 個 Host 的案例。

執行 Epoch
----

在啟動 Eopch 之前，需要先啟動 Controller，不然 Epoch 會無法啟動

啟動 Controller 之後，只需要輸入以下指令並輸入密碼（sudo 用）即可啟動 Epoch

> StartEpoch.sh

這邊補充一下，就目前看來，Epoch 只支援 OpenFlow 1.0 而已，目前也沒有辦法幫他設定版本。

啟動之後應該會在網路介面下多出一個 tap0 才對

> $ ifconfig  
tap0: flags=8843&lt;UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST&gt; mtu 1500  
    ether 56:94:88:d2:01:84  
    inet 192.168.1.1 netmask 0xffffff00 broadcast 192.168.1.255  
    open (pid 2561)  


此時就可以用電腦自己 ping 看看裡面的 Host，原則上如果 Controller 有起來的話應該會看到 packet in 以及
ping 會成功才對。

>$ ping 192.168.1.2  
PING 192.168.1.2 (192.168.1.2): 56 data bytes  
64 bytes from 192.168.1.2: icmp_seq=0 ttl=64 time=6.152 ms  
64 bytes from 192.168.1.2: icmp_seq=1 ttl=64 time=0.386 ms  
64 bytes from 192.168.1.2: icmp_seq=2 ttl=64 time=0.336 ms  
^C  
--- 192.168.1.2 ping statistics ---  
3 packets transmitted, 3 packets received, 0.0% packet loss  
round-trip min/avg/max/stddev = 0.336/2.291/6.152/2.730 ms  


參考：
----
* [Epoch][0]
* [Epoch support][7]


[0]: http://northboundnetworks.com/epoch/
[1]: http://sourceforge.net/p/ryu/mailman/ryu-devel/
[2]: http://mininet.org/
[3]: https://www.kickstarter.com/projects/northboundnetworks/zodiac-fx-the-worlds-smallest-openflow-sdn-switch
[4]: http://www.northboundnetworks.com/download_files/tuntap_20111101.tar.gz
[5]: http://tuntaposx.sourceforge.net/download.xhtml
[6]: https://hpn.hpwsportal.com/catalog.html#/Product/{%22productId%22%3A%221626%22}/Show
[7]: http://northboundnetworks.com/support/