title: Applied Cryptography-3
date: 2015-10-10 23:38:40
tags: Applied Cryptography
---
來源：[chapter 2 (23~51)](/papers/Chapter2.pdf)

在進入Block Cipher之前，先來思考一個問題：
![](/images/QAQ.gif)
<h5> Question: 加解密，錯誤更正碼(encoder、decoder)，壓縮先後順序 </h5>
假設今天你要將資料m透過網路傳給別人，但資料量太大必須壓縮，而且資料必須保密不能讓第三者看到因此要加密，並且需要保證接收方收到的資料正確無誤所以你希望加個錯誤更正碼降低接收方的錯誤率，問題來了，這三道程序的先後順序要如何排列呢？如果不按照"正確"的程序進行會發生什麼事呢？

![](/images/AC3_Q.jpg)

一次要看三道程序的話不免過於複雜，學資工的應要有資工的解法，這邊我們利用divide and conquer的想法(誤XDD  
首先我們先來看看加解密與錯誤更正碼哪個應該優先做，分別為下兩a、b兩種方法：

![](/images/cry_coder.jpg)

如果你的更正碼用的是linear block code，若你想要code自動錯誤更正n個bit，你至少要增加2n+1個bit的更正碼，不然更正碼可能會更正到另一組更正碼認為是正確的coset。這邊用簡單的圖來幫助理解，先想像一下，你今天收到的資料與原本資料假設有k個bit不一樣，也就是錯了k個bit，這樣就代表著你收到的資料與正確資料距離為k，ok，再來理解錯誤更正碼，假設你今天送的資料加了錯誤更正碼，它能夠自動錯誤更正n個bit，也就是我今天收到的資料如果錯的bit數小於n的話，他就有辦法自己更正成正確的，那也就是在方圓半徑n以內的我都有辦法將他拉到原點，如下圖所示：A點為正確資料，半徑為n，{% math %} r_1 {% endmath %}能夠自動更正回來，{% math %} r_2 {% endmath %}無法。

![](/images/err_1.jpg)

OK，問題來了，假設今天只有一個圓，也就是只有一組正確資料，那沒什麼問題，但現實情況不可能只有傳一筆資料吧....所以當你有兩筆以上的資料時，如果每筆資料靠的太近，那就可能會發生相交甚至重疊，此時如果你剛好錯誤的地點發生在重疊部分，錯誤更正碼就可能將你導向"另一筆"正確資料。如下圖所示，{% math %} r_1 {% endmath %}點落在A與B重疊處，原本正確資料為A，錯誤更正完後可能會變成B。

![](/images/err_2.jpg)

為了避免上述情況發生，我們增加了2n+1個bit的更正碼來增加正確資料的距離，避免錯誤更正範圍重疊。如下圖，只要錯誤的點不要落在錯誤更正範圍外，都能夠正確的解回來。

![A與B距離為2n+1](/images/err_3.jpg)

有了錯誤更正碼的概念後，我們可以發現：錯誤更正碼能力是有限的，他只能容許錯誤在一定範圍內，因此如果錯誤更正碼要搭配加密，那麼必須是在加密之後再來加錯誤更正碼，因為加密後只要錯了一個bit，解回來時肯定不只錯一個bit，所以如果你用了方法b，原本只錯了一個bit的密文解密後錯更多，在進行錯誤更正碼就大亂了，因此應該要使用方法a。

接下來我們來看看加解密與壓縮，一樣分為兩種方法：

![](/images/cry_comp.jpg)

這邊可以用效能來看，傳統加解密會需要耗費許多時間，因此需要被加解密的資料越小越好，當然選擇先壓縮再加密。而另一個角度是這兩種演算法的特性：資料壓縮的原理是將重複的pattern用特定的符號來取代，所以重複的pattern越多壓縮效果越好；而加密演算法特性是將資料打得越亂越好，因此不太有機會出現重複的pattern，所以如果先加密再壓縮的話，壓縮的效果可能不佳，甚至可能壓縮後檔案會變大。

綜合以上推論，正確順序應該為：壓縮→加解密→錯誤更正碼。

<hr>

<h3> Block Cipher </h3>

一開始先介紹Block Cipher的概念，如下圖所示，將訊息切割成n段後，每一段訊息(block)都進入加密器加密，最後再merge回完整cipher。

![Block Cipher](/images/block_cipher.jpg)

接下來將利用典型的DES(Data Encryption Standard)來帶領大家進入Block Cipher的世界。首先DES每次取的block為64bit，而DES的key也是64個bit，那DES是如何將這64bit明文加密的呢?可以參考下圖：

![](/images/DES.jpg)

我們可以看到明文會經過16回合後變成cipher，key會每回合shift、permuted後進來，接下來我們來看這16回合實際運作。

![](/images/Feistel_cipher.jpg)
<center> [Feistel cipher from wiki](https://en.wikipedia.org/wiki/Feistel_cipher) </center>

DES這16回合是來自於Feistel cipher，它的概念是這樣：將明文分為左右兩部分，左半邊為{% math %} L_n {% endmath %}右半邊為{% math %} R_n {% endmath %}，n為回合數。首先把{% math %} R_0 {% endmath %}與{% math %} K_0 {% endmath %}當作input，丟進F後的output與{% math %} L_0 {% endmath %}做{% math %} \bigoplus {% endmath %}，此為第一回合，第二回合開始將第一回合{% math %} \bigoplus {% endmath %}結果與{% math %} K_1 {% endmath %}當作input再丟入F，得到output再與第一回合的{% math %} R_0 {% endmath %}做{% math %} \bigoplus {% endmath %}，以此類推至16回合結束。解密的部分稍後再詳細說明。

接下來介紹F裡面做了什麼事情，在介紹之前先介紹第一堂課探討過的取代跟重排。取代跟重排不管你單獨重複多次或者交叉使用多次，都能夠化減為單一回合的取代。為了解決此問題，[Substitution-permutation network](https://en.wikipedia.org/wiki/Substitution-permutation_network)利用了不同size的S-box與P-box進行重排與取代，此方法經過重複多次後無法用單一回合的取代來代替。

![S-P Network](/images/SP.jpg)

Substitution-permutation network一開始的8bit substitution為key，故每次的key不一樣導致替換的結果不同，後面的64bit permutation會造成[Avalanche effect 雪崩效應](https://en.wikipedia.org/wiki/Avalanche_effect)，所以不同input只差一個bit會導致ciphertext完全不一樣。

有了Substitution-permutation network概念後接下來看F做的事情，如下圖所示：

![](/images/DES1.jpg)

圖中可以看到它把右邊的32bit先做Expansion以及permutation成48bit，實作如下圖：

![Expansion/Permutation](/images/expper.jpg)

得到48bit後與key做{% math %} \bigoplus {% endmath %}，XOR完之後就是S-box變形與P-box，下圖為P-box實作：

![P-box](/images/DES_pbox.jpg)

上面兩個permutation的圖都是代表第幾個位置為原本位置的值，例如Expansion/Permutation的圖，第一個bit的值被替代後變成了原本的第32bit的值。  
而這邊的S-box比較特殊，它是8個6bit轉4bit的替換器，替換方式如下：{% math %} b_1 {% endmath %}與{% math %} b_6 {% endmath %}代表第幾列，{% math %} b_2 {% endmath %}到{% math %} b_5 {% endmath %}代表第幾行。舉個例子：假設今天送進{% math %} S_1 {% endmath %}的訊息為000001，則替換的數字為第01列第0000行，所以output為0000(0)。

![S-box](/images/DES_sbox.jpg)

這邊可以發現到S-box內容是不變的，但在之前Substitution-permutation network有提到S-box應隨著key做改變，看一下下列這張圖：

![](/images/DES2.jpg)

在進行S-box之前會先將資訊與key作XOR，此動作會導致替換結果隨著key做改變。

下圖為DES的key schedule，key schedule是決定每次round要給予的subkey的值，圖中可以看到key拿掉8 bit的parity後經過PC1被分成左右兩半，之後分別進行left shift，每次shift有可能是1bit或2bit，取決於右邊的Number of Left Shifts，shift完之後再經由PC2從28bit當中挑選24bit，合併後傳送給DES做XOR。

![](/images/key_schedule.jpg)

有關PC1與PC2的permutation如下(圖的看法與上面的permutation一樣)：

![](/images/PC1and2.jpg)

<h5> DES decryption </h5>

終於到了解密部分了.....是否還記得剛剛加密時的S-box？它是將6bit轉換為4bit，但如果按照正常邏輯，要解密時不是該變成反函式，要將4bit轉回6bit嗎？但這件是不可能發生啊!!!!!!除非你會通靈....

{% youtube sToP9FpZ4Dk %}

4bit轉6bit代表你要1對多，在一對多的狀況時你要如何知道哪個才是你真正要的原文呢？因此此方法行不通。那麼該如何解密呢？觀察一下下面這張圖：

![](/images/DES_decrypt.jpg)

首先先看左邊加密時的狀況，在最後一個回合時，右邊的藍色跟黃色做XOR後得到紅色，然後被拉到左邊；而左邊的黑色根紫色做XOR後變成綠色，被拉到右邊。而右邊的黃色是原本的左邊綠色經過F後得到的，而左邊的紫色是右邊的藍色經過F所得到的。左邊加密看完後接下來看右邊的解密，假如我直接把得到的密文再丟入DES一次會發生什麼事情呢？右邊的綠色往上丟，經過F會變成黃色(由左邊加密時得知)，而左邊紅色也往上丟，與剛剛出爐的黃色做XOR得到藍色(藍XOR黃=紅，so黃XOR紅=藍)，有沒有發現一件神奇的事情，我剛剛做的事情就是原本加密時的第16回合，也就是說，照原本DES的加密上式進行下去，key只要倒著排，就能夠把密文還原回明文，酷吧！

<hr>

<h4> Cipher Block Chaining (CBC) </h4>

由於DES有個致命的缺點，一樣的明文送進DES第二次會出現與上一次一模一樣的密文，這樣攻擊者只要蒐集密文並分析就有可能推測出明文，為了避免像是ciphertext searching這種攻擊，block cipher有一種模式叫做cipher block chaining，它是在每次要進行加密前，多XOR一串vector，這樣就不會使每次的明文加密完後都一樣。

![](/images/CBC.jpg)

如上圖，第一個block會XOR一個Initial Vector(IV)，之後的block都會XOR前一個block的cipher，這麼做的好處是cipher是加密過後的，所以他的值非常難預測，幾乎是亂數值。而CBC的最後一個cipher也能拿來當作Message Authentication Code (MAC)，由於你要拿到最後一個cipher一定要將整個明文做一次CBC mode的加密才有辦法拿到，所以此cipher能拿來當作MAC，MAC概念類似於checksum或者是parity check。[wiki在此](https://en.wikipedia.org/wiki/Message_authentication_code)

最後老師講了一個CBC的攻擊，不過我看投影片還有其他的攻擊，這部分就保留到下次在一起寫吧。
