Ryu 官方線上文件中文翻譯
====

最近我在撰寫 [Ryu 線上文件中文版本][1]，歡迎大家協助提出錯誤修正或是[協助翻譯][2]


緣起
----

原先只是為了打發時間所以就來進行翻譯的動作，剛好 ryu 官方文件並沒有提供任何中文版本（當然，Ryu Book 除外）。

另一方面，也可以順便練練英文以及順便確認一下對於 ryu 那邊還不太熟悉 XD

文件架構
----

>.  
├── Makefile  
└── source  
........├── （略）  
........├── api_ref.rst  
........├── app  
........│   ├── ofctl.rst  
........│   └── ofctl_rest.rst  
........├── app.rst  
........├── components.rst  
........├── conf.py  
........├── configuration.rst  
........├── developing.rst  
........├── getting_started.rst  
........├── gui.png  
........├── gui.rst  
........├── index.rst  
........├── （下略）  

Ryu 的文件是使用 [ReStruct Text][3] 所撰寫而成，並使用 [Sphinx][4] 去進行建置的動作（rst -> html）。

要 Build 出 html 檔案很簡單，只需要輸入`make html`即可，它就會自動從 Mackfile 裡面設定的指令去產生文件，
當然，也可產生其他像是 txt，LaTex 之類的文件。

如何翻譯？
----

原則上翻譯沒有很困難，主要是要把所有 rst 你看的到的英文（目錄結構與設定除外）轉換成中文，目前（2015/07/31）
我已經翻譯完 70% 左右，卡在 ryu API 那邊正在慢慢翻譯。

此外，部分的 API 文件是使用 Sphinx 中的 auto class 與 auto module 去產生的，除非去翻譯 python 程式碼裡面
原始 pyDoc 的內容，否則沒有辦法翻譯它。

翻譯主要分為兩個階段：

1. 全部 rst 可以翻到的地方
2. Python Doc 內容翻譯

以上兩個部分都是在我 fork 出來的 repo 的另一個獨立的 branch 中進行，以確保不會影響到原始內容。

會這麼做主要的原因是因為 ryu 官方使用的是 <readthedocs.org> 這一個文件平台，他可以直接輸入一個
repo 並且將裡面的 doc 資料夾下的內容編譯成 html （依據 Sphinx 規則），所以如果要製作一個中文版的
說明文件，只需要將自己 fork 出來的文件翻譯，並且在該網站創立一個新的專案即可。

而翻譯的版本如果要跟原版連結在一起，需要到他的設定中將翻譯的 Project 加入，這樣一來在原始版本以及翻譯版本中的 
Language 底下就會出現不同的語言連結可以使用。

<img src="/images/readthedoc-translate-setting.png" width="600" />

翻譯成果
----

翻譯的結果可以到[這邊][1]觀看，但是目前是沒有完就是了......


[1]: http://ryu-zhdoc.readthedocs.org/
[2]: https://github.com/TakeshiTseng/ryu/tree/zhdoc/doc/source
[3]: https://en.wikipedia.org/wiki/ReStructuredText
[4]: http://sphinx-doc.org/