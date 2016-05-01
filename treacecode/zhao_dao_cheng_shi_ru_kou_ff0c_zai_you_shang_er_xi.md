# 找到程式入口，再由上而下抽絲剝繭


根據需要決定展開的層數，或展開特定節點，並記錄樹狀結構，然後適度忽略不需要了解的細節─這是一個很重要的態度。因為你不會一次就需要所有的細節，閱讀都是有目的的，每次的閱讀也許都在探索程式中不同的區域。


探索系統架構的第一步，就是找到程式的入口點。找到入口點後，多半採取由上而下（Top-Down）的方式，由最外層的結構，一層一層逐漸探索越來越多的細節。

我們的開發團隊曾針對Winamp的iPod plug-in進行閱讀及探索，不僅找到入口點，也找出、並理解它最根本的基礎架構。從這個入口點，可以往下再展開一層，分別找到三個重要的組成及其意義：
```c
● init()：初始化動作
● quit()：終止化動作
● PluginMessageProc()：以訊息的方式處理程式所必須處理的各種事件
```

展開的同時，隨手記錄樹狀結構
當我們從一個入口點找到三個分支後，可以順著每個分支再展開一層，所以分別繼續閱讀init、quit、以及PluginMessageProc的內容，並試著再展開一層。閱讀的同時，你可以在文件中試著記錄展開的樹狀結構。
```c
● init()：初始化動作
● itunesdb_init_cc()：建立存取iTunes database的同步物件
● 初始化資料結構
● 初始化GUI元素
● 載入設定
● 建立log檔
● autoDetectIpod()：偵測iPod插入的執行緒
● quit()：終止化動作
● itunesdb_del_cc()：終止存取iTunes database的同步物件
● 關閉log檔
● 終止化GUI元素
● PluginMessageProc()：以訊息的方式處理程式所必須面臨的各種事件
● 執行所連接之iPod的MessageProc()
```

這部分必須要留意幾個重點。首先，應該一邊閱讀，一邊記錄文件。因為人的記憶力通常有限，對於陌生的事物更是容易遺忘，因此邊閱讀邊記錄，是很好的輔助。

再者，因為我們採取由上而下的方式，從一個點再分支出去成為多個點，因此，通常也會以樹狀的方式記錄。除此之外，每次只試著往下探索一層。從init()來看你便會明白。以下試著摘要init()的內容：

```c
int init() {
    itunesdb_init_cc();
    currentiPod=NULL;
    iPods = new C_ItemList;
    conf_file=(char*)SendMessage(plugin.hwndWinampParent,WM_WA_IPC,0,IPC_GETINIFILE);
    m_treeview = GetDlgItem(plugin.hwnd LibraryParent,0x3fd);
    //this number is actually magic :)
    g_detectAll = GetPrivateProfileInt("ml_ipod", "detectAll",0,conf_file)!=0;
    g_log=GetPrivateProfileInt("ml_ipod","log",0,conf_file)!=0;
    g_logfile=fopen(g_logfilepath,"a");
    autoDetectIpod();
    return 0;
}

```

因為我們只試著多探索一層，而目的是希望發掘出下一層的子動作。所以在init()中看到像「itunesdb_init_cc();」這樣的函式呼叫動作時，我們知道它是在init()之下的一個獨立子動作，所以可以直接將它列入。但是當看到如下的程式行：

```c
currentiPod=NULL;
iPods = new C_ItemList;
```

我們並不會將它視為init()下的一個獨立的子動作。因為好幾行程式，才構成一個具有獨立抽象意義的子動作。例如以上這兩行構成了一個獨立的抽象意義，也就是初始化所需的資料結構。

理論上，原來的程式撰寫者，有可能撰寫一個叫做init_data_structure()的函式，包含這兩行程式碼。這樣做可讀性更高，然而基於種種理由，原作者並沒有這麼做。身為閱讀者，必須自行解讀，將這幾行合併成單一個子動作，並賦予它一個獨立的意義──初始化資料結構。

###無法望文生義的函式，先試著預看一層
對於某些不明作用的函式叫用，不是望其文便能生其義的。當我們看到「itunesdb_init_cc()」這個名稱時，我們或許能從「itunesdb_init」的字眼意識到這個函式和iPod所採用的iTunes database的初始化有關，但「cc」卻實在令人費解。為了理解這一層某個子動作的真實意義，有時免不了要往前多看一層。

原來它是用來初始化同步化機制用的物件。作用在於這程式一定是用了某個內部的資料結構來儲存iTunes database，而這資料結構有可能被多執行緒存取，所以必須以同步物件（此處是Windows的Critical Section）加以保護。

所以說，當我們試著以樹狀的方式，逐一展開每個動作的子動作時，有時必須多看一層，才能真正瞭解子動作的意義。因為有了這樣的動作，我們可以在展開樹狀結構中，為itunesdb_init_cc()附上補充說明：建立存取itunes database的同步物件。這麼一來，當我們在檢視自己所寫下的樹狀結構時，就能輕易一目瞭然的理解每個子動作的真正作用。

###根據需要了解的粒度，決定展開的層數
我們究竟需要展開多少層呢？這個問題和閱讀程式碼時所需的「粒度（Granularity）」有關。如果我們只是需要概括性的瞭解，那麼也許展開兩層或三層，就能夠對程式有基礎的認識。倘若需要更深入的瞭解，就會需要展開更多的層次才行。

有時候，你並不是一視同仁地針對每個動作，都展開到相同深度的層次。也許，你會基於特殊的需求，專門針對特定的動作展開至深層。例如，我們閱讀Winamp iPod plug-in的程式目錄，其實是想從中瞭解究竟應該如何存取iPod上的iTunes DB，使我們能夠將MP3歌曲或播放清單加至此DB中，並於iPod中播放。

當我們層層探索與分解之後，找到了parseIpodDb()，從函式名稱判斷它是我們想要的。因為它代表的正是parse iPod DB，正是我們此次閱讀的重點，也就達成閱讀這程式碼的目的。

我們強調一種不同的做法：在閱讀程式碼時，多半採取由上而下的方式；而本文建議了一種記錄閱讀的方式，就是試著記錄探索追蹤時層層展開的樹狀結構。你可以視自己需要，瞭解的深入程度，再決定要展開的層數。你更可以依據特殊的需要，只展開某個特定的節點，以探索特定的細目。

適度地忽略不需要了解的細節，是一個很重要的態度，因為你不會一次就需要所有的細節，閱讀都是有目的的。每次的閱讀也許都在探索程式中不同的區域；而每次探索時，你都可以增補樹狀結構中的某個子結構。漸漸地，你就會對這個程式更加的瞭解。