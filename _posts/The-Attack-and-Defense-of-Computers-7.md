title: The Attack and Defense of Computers-7
date: 2015-11-08 15:50:58
tags: The Attack and Defense of Computers
categories: The Attack and Defense of Computers
---
來源：[Botnet (11 ~ 66)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/2_1_Botnet.ppt)

<h2> Botnet </h2>
Botnet是指一群被入侵的電腦，這些電腦可以同時被一個攻擊者下指令來操控，而這攻擊者又叫做botmaster，bot為被入侵的電腦。那botnet可以用來幹嘛呢？botnet所控制的電腦數量可大可小，大至五萬台bot都有可能，有五萬台的殭屍電腦相當於有五萬個不同的IP，這可不是開玩笑的，接下來來看運用botnet可以做什麼樣的攻擊。

<h4> Distributed Denial-of-Service Attacks </h4>
這就是我們常聽到的DDos攻擊，他可以造成某個使用者無法使用某個服務，或者吃掉受害者使用的網路，甚至阻斷某個線上服務。

<h4> Spamming </h4>
發送垃圾信件，當一個受害的電腦被開啟sock proxy，那麼他就會幫攻擊者傳送垃圾信件。在2010年全球所有的信件當中有89%都是垃圾信件，數字非常驚人，而Grum Botnet在當時發送的垃圾信件佔了全球垃圾信件的18%，是當時發送最多垃圾信件的botnet，同時也是全球第三大的botnet。

不光是正常的電腦或手機會發送垃圾信件，現在大多數智慧聯網的家電都沒有做任何的防護措施，因此有蠻多的垃圾信件都是藉由家裡的冰箱冷氣等連上網的家電發送而來。攻擊者能利用這點，對他的botnet下達指令，同時送100000份的垃圾信件，這些垃圾信件可能來自電腦或是行動裝置甚至是家電，同時有100000個IP在發送垃圾信件，造成防禦者很難封鎖IP。而在所有的垃圾信件當中，來源是從家電發出的大概佔了所有垃圾信件的25%。下圖為各個botnet的垃圾信件發送量：

![](/images/spamming_capacity.jpg)

<h4> Sniffing Traffic </h4>
當然有些bot可以拿來當作sniffer，觀察該網路的封包，如果受害電腦的網段中有使用沒有加密的軟體，那麼便有機會拿到敏感資訊。

<h4> Keylogging </h4>
如果攻擊者想竊取一些bot的資料，那麼他就能夠使用keylogger，紀錄受害者的鍵盤歷史紀錄。

<h4> Spreading New Malware </h4>
攻擊者也可以利用這些bot來傳送病毒或惡意程式，來擴充自己的botnet。

<h4> Installing Advertisement Addons </h4>
這是個利用botnet賺錢的好方法XD google打廣告的方式如下：

![](/images/advertisment.jpg)

只要有使用者點進上圖的有黃色廣告字眼的連結，那麼google就可以跟向google買廣告的人收費。通常google做法是給消費者一個額度，只要有人點進這個聯結就會扣取一些，直到額度被扣完後這個廣告就會被google移除。而google除了自己可以賺廣告費外，他也提供了[Google AdSense](https://www.google.com/adsense/start/?subid=ww-zh_tw-ha-adsense)，他能讓任何人都能賺廣告費用，申請後只要將google給你的廣告貼在你的部落格上，凡是只要從你的部落格點廣告進去的你就能收取一定的費用。

由於google是靠他自己的service來判斷是從哪邊點進去的，他只能夠依靠使用者給的資訊來判斷，因此只要有人點進網址google就會算錢，而這些只點進去就跳走的人google沒辦法知道，只有跟google買廣告的人才知道他的網站流量，而這些廣告費用就這樣浪費掉了，這些損失就會落在買廣告的廠商。而這些廠商當然會不服器找google理論，而google為了維持名譽所以只要廠商提供證據google就照單全收，平均而言這些點擊次數有15%左右都是充人數用的，如果這問題有辦法解決的話google每年就不會平白損失了這些金額了。

那攻擊者要如何利用botnet來點擊這些廣告呢？由於攻擊者擁有非常大量不同的IP，google根本無從判斷是否為同一個人，而攻擊者又能夠在受害電腦上安裝一些程式，使受害電腦的使用者只要一瀏覽網頁就會自動去點擊攻擊者指定連結，如此一來攻擊者的網站就會有數萬個IP點擊他的廣告了。

<h4> Manipulating online Polls/Games </h4>
因為攻擊者擁有不同的IP，而現在大多的投票系統也都是用IP來區分使用者，因此攻擊者就能操控網路上的投票結果了。而現在的線上遊戲也都是用IP來區分不同玩家，所以當同一個人擁有多個IP可以做的事情也就更多了。

<h4> Mass Identity Theft </h4>
這也是個嚴重的問題，當攻擊者擁有botnet，這時他能夠操作數萬台電腦，也就是說這數萬台電腦裡面所擁有的個資他都能夠掌控，搭配釣魚網站跟keylogger，攻擊者能蒐集到的個資非常可觀。

<hr>

<h2> IRC (Internet Relay Chat)  </h2>
IRC是個線上聊天平台，當你連上IRC server時會有很多個channel，這些channel有的是公開的有的不公開，你可以在特定的channel跟特定的群組聊天。下圖是IRC網路架構：

![](/images/IRC-network.jpg)

圖中可以看到每一個client都會連到某個server，這些client當中可能存在一些bot。IRC bot是指一些script或是程式，他會像一般使用者一樣掛在線上，但他不會回你，他做的事情是等待控制者下達命令或者是做一些被預設指定的事情。而Command and Control (C&C) server是botmaster所設定的，通常會利用C&C server來對IRC bot下command，通常一個IRC botnet會加入某個channel來竊聽訊息。

C&C server大致可分為幾種類型，第一種是centralized model，他的特性是所有的IRC bot都會經由他下達命令，因此攻擊者會找一個頻寬較高的host來當C&C server。一些知名的bot像是AgoBot、SDBot跟 RBot都屬於centralized model，因為此種模式botmaster容易控制他底下的bot，而且Messaging latencies較低，因此botmaster與這些botnet溝通上非常快速，發起攻擊會較順利。水能載舟亦能覆舟，此方法雖然便利，但同時也非常脆弱，因為botmaster所有的指令都要透過C&C server來跟botnet溝通，因此C&C server是最常有訊息出入的地方，因此如果要攻擊這個攻擊者的話打他的C&C server就好了，而且只要C&C server掛掉整個botnet都不能使喚了。

由於centralized model有上述缺失因此有了另一種類型：P2P-Based C&C Model。此種類型相較於centralized model之下較不易被發現，也比較不容易被摧毀。但相對的同時能夠下達命令的botnet也較少，能夠同時間對話的只有10 ~ 50個user，而對centralized model來說1000個botnet都算小數字；而P2P systems也沒有保證message delivery 與propagation latency，所以在發動攻擊時可能會有botnet loss掉訊息，導致攻擊失敗。雖然P2P-Based C&C Model相對於centralized model有不少缺點，但因為他比較難被察覺與攻擊，因此被使用的趨勢逐漸升高，下圖是使用P2P protocol的時間軸：

![](/images/timeline_of_p2p.jpg)
