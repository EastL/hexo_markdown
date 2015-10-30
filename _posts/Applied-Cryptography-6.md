title: Applied Cryptography-6
date: 2015-10-28 16:25:35
tags: Applied Cryptography
---
來源：[chapter 3 (33 ~ 49)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-3%20Number%20Theory.pdf)

上禮拜最後講中國餘數定理，可以將大世界的複雜計算分給很多個小世界，小世界計算完後透過中國餘數定理merge回來，接下來的二次剩餘將會運用到中國餘數定理。

<h2> Quadratic Residues </h2>
Quadratic Residues探討在模數世界開根號的問題，既然是開根號就分為兩種：可以開根號跟不能開根號。
*   Quadratic Residue ({% math %} QR_p {% endmath %})
    定義p為質數，若某數平方後mod p能找到a，則我們稱a為{% math %} QR_p {% endmath %}：
    {% math %} x^2 \ \ \equiv \ \ a \ \ (mod \ \ p) {% endmath %}
*   Quadratic Nonresidue({% math %} NQR_p {% endmath %})
    若mod p底下有一個數a，他找不到任何x使得{% math %} x^2 \ \ \equiv \ \ a \ \ (mod \ \ p) {% endmath %}，則代表a在mod p底下不能開根號，稱為{% math %} NQR_p {% endmath %}。

知道了在模數底下分為可開根號跟不可開根號後，接下來我們來看看在mod p底下(p為質數)，會有多少個{% math %} QR_p {% endmath %}跟{% math %} NQR_p {% endmath %}呢？mod p 底下總共會有0~p-1共p個數，0在這邊暫且不談，我們將其他的數都做平方來觀察：
<center> {% math %} 1^2, \ \ 2^2, \ \ 3^2, ... , \ \ ( \frac{p-1}{2} )^2, \ \ ( \frac{p+1}{2} )^2, \ \ ..., \ \ (p-2)^2, \ \ (p-1)^2  {% endmath %} </center>

觀察$1^2$跟$(p-1)^2$，這兩數平方後mod p都是1，所以1的平方根為1跟p-1，其他數也是如此，其實如果把p改成0就是我們平常認知的平方根了，所以k根p-k一組，剛好從中間切半，那左半邊的平方後mod p會不會有兩個根對到同一個數呢？下面利用反證法來證明：

假設{% math %} x^2 \ \ \equiv \ \ a \ \ \equiv \ \ y^2 \ \ (mod \ \ p){% endmath %}，且x,y不相等，{% math %} 1 \ \ \leq \ \ x,y \ \ \leq \ \ \frac{p-1}{2} {% endmath %}，則x不是跟y一樣就是p-y，但這兩種都不符合條件，因此平方根不會同時有兩個小於(p-1)/2，也就是在小於(p-1)/2的情況下會有兩個數平方後mod p會是同一個數。

接下來說明如何區分{% math %} QR_p {% endmath %}跟{% math %} NQR_p {% endmath %}：
<center> {% math %} y^{ \frac{p-1}{2}} \ \ \equiv \ \ \{_{-1 \ \ (mod \ \ p) \ \ \rightarrow \ \ y \ \ \in \ \ NQR_p }^{1 \ \ (mod \ \ p) \ \ \rightarrow \ \ y \ \ \in \ \ QR_p } {% endmath %} </center>

proof:

先驗證{% math %} y^{ \frac{p-1}{2}} {% endmath %} mod p是否只會有1跟-1兩種結果：假設{% math %} y^{ \frac{p-1}{2}} {% endmath %} mod p = t，則
<center>
{% math %} 
t  \ \ \equiv \ \ y^{ \frac{p-1}{2}} \ \ (mod \ \ p) \\ 
\Rightarrow t^2 \ \ \equiv \ \ y^{ p-1} \ \ (mod \ \ p)   \\
\Rightarrow t^2 \ \ \equiv \ \ 1 (mod \ \ p)  
{% endmath %} 
</center>
因此t確實只有正負1。接下來為什麼mod p後等於一就是$QR_p$了呢？假設y是a的平方根，則
<center>
{% math %} 
y \ \ \equiv \ \ a^2 \ \ (mod \ \ p) \\
\Rightarrow \ \  y^{ \frac{p-1}{2}} \ \ \equiv \ \ a^{p-1} \ \ (mod \ \ p)  \\
\Rightarrow \ \ y^{ \frac{p-1}{2}} \ \ \equiv \ \ 1 \ \ (mod \ \ p)
{% endmath %}  
</center>
所以剩下的$NQR_p$就是等於-1了。
<h4> Legendre symbol </h4>
接下來利用剛剛{% math %} y^{ \frac{p-1}{2}} {% endmath %} mod p的特性我們定義Legendre symbol：

![](/images/legendre_symbol.jpg)

由於剛剛我們沒有把0討論進去，這邊0不算$QR_p$也不算$NQR_p$，之後密碼學也不太會用到就不管他XD這邊要注意的是p最小是3，等於二的話套入剛剛式子就變成$y^{1/2}$了。
<h4> Characteristic of $QR_p$ and $NQR_p$ </h4>
<center>
{% math %}
QR_p \ \ \times \ \ QR_p \ \ \rightarrow \ \ QR_p \\
QR_p \ \ \times \ \ NQR_p \ \ \rightarrow \ \ NQR_p \\
NQR_p \ \ \times \ \ QR_p \ \ \rightarrow \ \ NQR_p \\
NQR_p \ \ \times \ \ NQR_p \ \ \rightarrow \ \ QR_p 
{% endmath %}
</center>
這邊就跟國中教的正正得正負正得負一樣了...兩個$QR_p$相乘還是$QR_p$，一個$QR_p$跟$NQR_p$相乘會變成$NQR_p$這些都比較好理解，因此這邊來說明一下"負負得正"：給定兩個$NQR_p$為X跟Y，則：
<center> 
{% math %} 
X^{ \frac{p-1}{2}} \ \ \equiv \ \ -1 \ \ (mod \ \ p) \ \ and \ \ Y^{ \frac{p-1}{2}} \ \ \equiv \ \ -1 \ \ (mod \ \ p)  \\  
\Rightarrow \ \ (X \ \ \times \ \ Y)^{ \frac{p-1}{2}} \ \ \equiv \ \ X^{ \frac{p-1}{2}} \ \ \times \ \ Y^{ \frac{p-1}{2}} \ \ (mod \ \ p)  \\
\Rightarrow \ \ (X \ \ \times \ \ Y)^{ \frac{p-1}{2}} \ \ \equiv \ \ -1 \ \ \times \ \ -1 \ \ \equiv \ \ 1 \ \ (mod \ \ p)
{% endmath %} 
</center>
<h4> Jacobi symbol </h4>
如果遇到的模數不是質數而是合成數的話，可以將合成數拆成質數後再去算Legendre symbol，下列為Jacobi symbol Properties：
<center> 
{% math %} 
J( \frac{a}{pqr}) \ \ = \ \ L( \frac{a}{p}) \ \ \times \ \ L( \frac{a}{q}) \ \ \times \ \ L( \frac{a}{r}) \ \ p,q,r \ \ \in \ \ odd \ \  prime  \\
J( \frac{a_1*a_2}{n}) \ \ = \ \ J( \frac{a_1}{n}) \ \ \times \ \ J( \frac{a_2}{n})
{% endmath %}
</center>
這邊要注意的是，如果要使 {% math %} a \ \ \in \ \ QR_n {% endmath %}，則a必須是所有n的質因數的QR才行，剛剛講的QR跟NQR特性是在同一個質數的模數底下，這邊用Jacobi symbol拆開的模數是不同的質數。這邊舉個例子：

假設要算{% math %} J( \frac{a}{pq}) {% endmath %} ，如果{% math %} J( \frac{a}{p}) {% endmath %} 與{% math %} J( \frac{a}{q}) {% endmath %} 都是-1，則不能像剛剛那樣(-1)*(-1)=1，因為這邊模數一個是p一個是q，因此只要有一個Jacobi symbol值為-1則a就是NQR。

<hr>

講了這麼多數論的東西，終於要進到第一個實際運用的系統了(灑花~~)，繼續講數論的話很多同學可能看到數學就...

![](/images/holucan.gif)

好啦回歸正題，我們可以運用剛剛的QR以及NQR來設計一個帳密系統，這個帳密系統只能在local端登入，並且使用者去註冊時帳號密碼都是系統給你的，使用者無從選擇。帳號密碼管理通常就是建一張表格，表格內容每個entry就是一組帳號對應到密碼，如果要安全一點的可能把密碼丟進[hash](https://en.wikipedia.org/wiki/Hash_table)存hash值。這邊我們可以利用QR跟NQR來儲存帳號相對應的密碼，而且密碼只需要存2個bit就夠了。

這系統既然會用到QR跟NQR，那他就一定是要開根號囉？沒錯，這系統主要運作原理就是選定一個帳號，然後將帳號開根號後得到密碼，丟給使用者，那麼你一定會問，如果密碼是把帳號開根號，那我自己開根號就好了啊，有帳號不是就等於有密碼了？這邊的關鍵點就在模數底下對合成數的開根號。現今在這世上對合成數開根號的演算法是利用中國餘數定理，把合成數拆成質數分別做開根號，如果不用中國餘數定理做的話就跟暴力法沒兩樣了，所以我們能利用這點，設計一個合成數n，n為兩個質數p,q相乘，n能公開但p跟q不公開，將帳號在mod n底下開根號得到密碼，如此一來攻擊者如果不知道如何將n拆成p跟q，那麼他也無法將帳號做開根號的動作來得到密碼了。有了這概念後接下來問題來了，你說要在模數底下開根號，可是我們剛剛才講過，開根號的世界可以分成QR跟NQR，並不是所有數都能開根號啊，所以接下來系統裡面會有一張表，這張表就是來處理這問題，讓所有帳號進來都能夠被開根號。

那這張表到底有什麼神奇魔力能夠讓NQR也可以開根號？其實就是用剛剛講到的QR跟NQR的特性，就是那個國中講的負負得正啦！如果今天帳號進來是NQR，那就讓他在乘上一個NQR不就可以開根號了~接著我們來詳細介紹這張表的內容：

![](/images/QR_system.jpg)

首先n = pq剛剛有講過，然後mod p可以分為QR跟NQR，mod q也能分為QR跟NQR，所以pq組合起來會有四種，後面的-1表示NQR，+1表示QR，如此一來帳號在mod n底下的所有可能性都在這張表上面了，然後我們在這四種狀況分別找一個數，就是前面的$ \alpha \ \ \beta \ \ \gamma \ \ 1$。接著我們來看管理者要如何儲存帳號跟密碼：

![](/images/ID_QR_table.jpg)

帳號(ID)在mod n底下也會是那四種可能的其中一種，所以密碼就記錄對p是+1還是-1，對q是+1還是-1，因此密碼欄位只需要兩個bit。有了上面這兩張表後，當我選定了一個ID，對應到密碼欄位(i.j)得知他的QR、NQR狀態，再去剛剛的表格看要選$ \alpha \ \ \beta \ \ \gamma \ \ 1$哪一個k來跟帳號相乘，如此一來相乘結果就能開根號了。下表是將剛剛講的兩張表結合整理：

![](/images/ID_QR_k_table.jpg)

*   Password generation：
    <center> 
    {% math %} 
    (Pw_i)^2 \ \ \equiv \ \ ID_i \ \ \times \ \ k \ \ (mod \ \ n) \ \ n \ \ = \ \ p*q \\
    (choose \ \ the \ \ smallest) 
    {% endmath %} 
    </center>
注意這邊ID開根號完會有4個密碼，因為是在mod合成數底下造成的，mod一個質數開根號後會有兩個根，我們設定的n擁有兩個質數，因此密碼會有四個根，至於為什麼要選擇最小的則是因為攻擊者如果拿到這四個根，他就有辦法解出p跟q，詳細狀況待會說明，先來看看這四個根如何解。
要解合成數的開根號當然是運用中國餘數定理，將原本mod n底下拆成mod p跟mod q，分別計算出根號值之後再merge回來。下圖為示意圖：

![](/images/mix_root.jpg)

$a_p$開根號後假設值為$S_1$, p-$S_1$，$a_q$開根號後為$S_2$, q-$S_2$，而T在mod n底下的四個值假設為x, n-x, y, n-y，則x會是$S_1$跟$S_2$ 利用中國餘數定理merge而成，n-x會是 p-$S_1$ 跟 q-$S_2$ merge而成，同理y會是$S_1$跟 q-$S_2$ merge，n-y是$S_2$跟 p-$S_1$ merge。簡單來說就是兩兩成對，四個根會有兩對pair。基本上到目前為止整個帳密系統輪廓都描繪出來了，就還差一個細節，記得剛剛講說利用中國餘數定理將合成數拆成質數，然後就能計算出開根號，那mod 質數底下要怎麼找開根號的兩個根呢？在1891年Alberto Tonelli發展了一個演算法解決了這問題，但此演算法效能不佳，在1973年[Daniel Shanks](https://en.wikipedia.org/wiki/Daniel_Shanks)發展了[Tonelli–Shanks algorithm](https://en.wikipedia.org/wiki/Tonelli%E2%80%93Shanks_algorithm)，也就是接下來要講的演算法。
<h4> Tonelli–Shanks algorithm </h4>
此演算法要解決的問題如下：p為質數，n為$QR_p$，找出x使得
<center> {% math %} x^2 \ \ \equiv \ \ n \ \ (mod \ \ p) {% endmath %} </center>
其實[Joseph-Louis Lagrange](https://en.wikipedia.org/wiki/Joseph-Louis_Lagrange)有找到當{% math %} p \ \ \equiv \ \ 3 \ \ (mod \ \ 4) {% endmath %}情況下，{% math %} x \ \ \equiv \ \ \pm n^{ \frac{p+1}{4}} {% endmath %} ([from wiki](https://en.wikipedia.org/wiki/Quadratic_residue))，會說明這個特殊狀況是因為待會演算法裡面有這個...

Algorithm:
Input : p, n，p為odd prime，n為integer且$QR_p$
Output : R, {% math %} R^2 \ \ \equiv \ \ n \ \ (mod \ \ p) {% endmath %}

步驟一：將p-1所有2的因數除掉，除到最後為奇數Q，令S為2的個數：{% math %} p \ \ - \ \ 1 \ \ = \ \ Q \ \ * \ \ 2^S {% endmath %}，當S=1時return {% math %} \pm n^{ \frac{p-1}{4}} {% endmath %} (剛剛講的特殊狀況)  
步驟二：選擇一個z使得z為$NQR_p$，並且令{% math %} c \ \ \equiv \ \ z^Q  \ \ (mod \ \ p){% endmath %}
步驟三：令{% math %} R \ \ \equiv \ \ n^{ \frac{Q+1}{2}} \ \ (mod \ \ p), \ \ t \ \ \equiv \ \ n^Q, \ \ M \ \ = \ \ S {% endmath %}
步驟四：Loop:
*   if t $ \equiv $ 1 (mod p)，return R
*   找到最小的i，0 < i < M，使得{% math %} t^{2^i} \ \ \equiv \ \ 1 \ \ (mod \ \ p) {% endmath %}
*   {% math %} b \ \ \equiv \ \ c^{2^{M-i-1}}, \ \ R \ \ \equiv \ \ bR, \ \ t \ \ \equiv \ \ tb^2\ \ (mod \ \ p) {% endmath %}, M = i

證明：  
其實只看演算法會看到霧煞煞，我們來整理一下：看一下步驟四的迴圈，當t=1時回傳R，因此我們可以知道R就是根號n，那跟t=1有什麼關係？由於我們設定{% math %} R \ \ \equiv \ \ n^{ \frac{Q+1}{2}} \ \ (mod \ \ p), \ \ t \ \ \equiv \ \     n^Q{% endmath %}，因此{% math %} R^2 \ \ \equiv \ \ nt \ \ (mod \ \ p) {% endmath %}，所以當t=1時R為n的平方根。那t什麼時候才會變成1呢？說明這個之前先補充一個預備知識：order，給定{% math %} m \ \ \in \ \ N, \ \ a \ \ \in \ \ Z  {% endmath %}，滿足gcd(a,m)=1，若{% math %} n \ \ \in \ \ N {% endmath %}是最小正整數滿足{% math %} a^n \ \ \equiv \ \ 1 \ \ (mod \ \ m) {% endmath %}，則稱n為a在mod m底下的order，並以{% math %} ord_m(a) \ \ = \ \ n {% endmath %}表之。(參考[基礎數論](http://math.ntnu.edu.tw/~li/ent-html/))

接著我們看步驟二的c，由於{% math %} c \ \ \equiv \ \ z^Q  \ \ (mod \ \ p){% endmath %}，因此{% math %} c^{2^S} \ \ \equiv \ \ (z^Q)^{2^S} \ \ \equiv \ \ z^{Q2^S} \ \ \equiv \ \ z^{p-1} \ \ \equiv \ \ 1 \ \ (mod \ \ p) {% endmath %}，此時還不確定$2^S$就是c的order，因此我們檢查$2^{S-1}$：{% math %} c^{2^{S-1}} \ \ \equiv \ \ z^{ \frac{p-1}{2}} \ \ \equiv \ \ -1 \ \ (mod \ \ p){% endmath %}，最後的等於-1是Legendre symbol，由此可知c的order為$2^S$。

接下來是關鍵的t，雖然{% math %} t^{2^S} \ \ \equiv \ \ 1 \ \ (mod \ \ p) {% endmath %}，但由於c是利用z是NQR來讓{% math %} c^{2^{S-1}} {% endmath %}算出來結果是-1，t在這邊無法如法炮製，因此我們假設t的order為$2^{S'}$。接著我們為了讓下一回合保持{% math %} R^2 \ \ \equiv \ \ nt \ \ (mod \ \ p) {% endmath %}，在步驟四中設定了{% math %} b \ \ \equiv \ \ c^{2M-i-1}, \ \ R \ \ \equiv \ \ bR, \ \ t \ \ \equiv \ \ tb^2 \ \ (mod \ \ p) {% endmath %}，如此一來就可以看成{% math %} (bR)^2 \ \ \equiv \ \ n*b^2*t \ \ (mod \ \ p) {% endmath %}，$b^2$消掉的話就跟原式一樣。

接著我們看一下c'，{% math %} (c')^{2^{S'}} \ \ \equiv \ \ b^{2^{S'} \ \ * \ \ 2} \ \ \equiv \ \ b^{2^{S'+1}} \ \ \equiv \ \ c^{2^{S-S'-1+S'+1}}  \ \ \equiv \ \ c^{2^S} \ \ \equiv \ \ 1 \ \ (mod \ \ p){% endmath %}，這邊就可以了解S-S'-1的-1是為了消掉後面的平方，-S'是為了後來要讓$2^{S'}$消掉，而{% math %} (c')^{2^{S'-1}} \ \ \equiv \ \ c^{2^{S-1}} \ \ \equiv \ \ -1 \ \ (mod \ \ p) {% endmath %}，故c'的order為$2^{S'}$。

這邊先來整理一下思緒，我們剛剛假定t的order為$2^{S'}$，然後推導到後來c'居然order也一樣是$2^{S'}$，由此可知下一輪的t'跟c"會有S" < S'，而他們的order為$2^{S"}$。所以當S"為0時，$2^{S"}$ = 1，也就是說{% math %} t^{2^{S"}} \ \ \equiv \ \ t^1 \ \  \equiv \ \ 1 \ \ (mod \ \ p) {% endmath %}，我們最想看到的t=1終於出現了，如果S"不為0沒關係繼續下一輪，因為S只會越來越小，所以當最後t找不到order時，t的次方選項就剩下0可以選了，此時的R即為根號解。

