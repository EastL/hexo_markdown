title: Applied_Cryptography_1
date: 2015-09-28 11:21:06
tags: Applied Cryptography
---
來源 ： [chapter 1 (1~23 and 29)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-1%20Introduction%20to%20Cryptography.pdf), [Prove equivalence of Diffie-Hellman shared secret](http://math.stackexchange.com/questions/61358/prove-equivalence-of-diffie-hellman-shared-secret)

## 對稱式加密系統(Symmetric Key Cryptography, SKC)
此加密系統特點是加解密所用的key為同一把，所以又稱為對稱式加密。

![](/images/skc.jpg)

對稱式加密系統主要加密方式有兩種：取代與重排

![](/images/skc1.jpg)

問題：
(1)疊了很多個取代後，能否只用一個取代器來達到相同效果?  
(2)重排與取代相互使用多次後，能否兜出一套電路來代替?  

實作：因64bit都要取代實作上有困難，所以每8個bit取代一次，最後64bit進行重排

![](/images/skc2.jpg)

SKC的對稱式金鑰造成了一個嚴重的問題：當傳送方要使用SKC時，必須先把Key傳給接收方，而要如何把Key傳給接收方為一大問題，Diffie-Hellman這對師生提出了Public Key Exchange解決此問題。

![](/images/skc3.jpg)

如圖所示，Alice與Bob首先溝通好公開的g與p，然後各自選擇了$X_A$與$X_B$，依照上圖式子計算出Y之後互相交換，這邊以Alice的角度講解：決定好$X_A$後，計算出$Y_A$並將$Y_A$告訴Bob(透過公開頻道)，此時Bob也會將他算出來的$Y_B$告訴Alice，Alice收到$Y_B$後，利用 Z = ${Y_B}^{X_A}$ mod p 算出Z，Bob收到$Y_A$後同理算出Z，此時雙方接拿到共同的key。  
*能夠公開$Y_A$與$Y_B$，是利用離散對數難題，讓得知Y,g,p的人無法計算出X

這邊提出一個問題：兩個人拿到的Z是一樣的嗎?  
proof : ${Y_A}^{X_B} \ \ mod \ \ p \ \ = \ \ {Y_B}^{X_A} \ \ mod \ \ p$
<center> ${Y_A}^{X_B} \ \ mod \ \ p \ \ = \ \ ({g^{X_A} \ \ mod \ \ p})^{X_B} \ \ mod \ \ p$ </center>
<center> 令$g^{X_A} \ \ mod \ \ p \ \ = \ \ a$，利用餘式定理$a = g^{X_A} - pK$ </center>
<center> 則 ： $({g^{X_A} - pK})^{X_B} \ \ = \ \ g^{X_A X_B}\ \ + \ \ C^{X_B}_1 g^{({X_B}-1) b}pK +...+({pK})^{X_B}$ </center> 
<center> $= g^{X_A X_B} + pK(m)$ </center>
<center> 由此可知$({g^{X_A} \ \ mod \ \ p})^{X_B} \ \ mod \ \ p \ \ = \ \ g^{X_A X_B} \ \ mod \ \ p$</center>
<center> 同理可推得$({g^{X_B} \ \ mod \ \ p})^{X_A} \ \ mod \ \ p \ \ = \ \ g^{X_A X_B} \ \ mod \ \ p$</center>
<center> 故$({g^{X_A} \ \ mod \ \ p})^{X_B} \ \ mod \ \ p \ \ = \ \ g^{X_A X_B} \ \ mod \ \ p \ \ = \ \ ({g^{X_B} \ \ mod \ \ p})^{X_A} \ \ mod \ \ p$</center>
<center> 得證${Y_A}^{X_B} \ \ mod \ \ p \ \ = \ \ {Y_B}^{X_A} \ \ mod \ \ p$ </center>

## 公開金鑰加密系統(Public Key Cryptography, PKC)

![](/images/pkc.jpg)

公開金鑰特點在於加解密所用的Key是不同把，加密的鑰匙能夠透過公開頻道傳送。  
PKC利用質因數分解與離散對數難題來產生鑰匙。  
最典型的例子為RSA加密：  
收方選取兩個大質數1933和2131，算出兩個的乘積  
n=p×q=1933×2131=4119223  
選取一數e當作加密金匙K (公開金鑰)  
$K_{er}  \ \ : \ \ e \ \ = \ \ 11$   
計算e之乘法反元素d，作為解密金匙{% math %}K_{dr}{% endmath %}(私密金鑰)  
使得 e×d=1 mod 1932×2130   
{% math %}K_{dr}  \ \ : \ \ d \ \ = \ \ 748211{% endmath %}  
加密時將明文M=2245613 轉換成密文 C   
$C \ \ = \ \ 2245613^{11} \ \ mod \ \ 4119223 \ \ = \ \ 771889$  
解密時將密文C轉換回明文   
$M \ \ = \ \ 7718897482^{11} \ \ mod \ \ 4119223 \ \ = \ \ 2245613$  

## 金鑰管理 (Key Management)

所有在系統上運作的key能夠在用一個master key來加密，但此master key該如何保管？  
秘密分享 (secret sharing)  
令 master key 值為三次方程式與y軸交點，再把方程式上的六個點交給不同人保管  
之後要還原方程式時只需要四個點就夠了（防止意外，能夠允許其中兩點不見）  


## A Cryptographic Game

<center>在牆上有一個寶箱，上面有兩道鎖，這兩道鎖的鑰匙分別由Alice與Bob保管，這兩把鑰匙只能開啟屬於自己的那到鎖，令一道鎖不能開。</center>  
<center>有一天Alice將月餅放入寶箱裡面，並且用他的鑰匙將寶箱鎖上，人就走了。</center>
<center>過了一天後，Bob來到寶箱前看到上面有一到鎖，於是也用自己的鑰匙鎖上第二到鎖。</center>
<center>又過了一天，Alice看到Bob鎖上後變把自己的那到鎖解開。</center>
<center>最後，Bob用他的鎖將寶箱打開後，開心的吃著他的月餅。</center>
<center>請問：這是SKC還是PKC？</center>

我個人覺得這是SKC，我把這寶箱的兩道鎖看成兩種加密演算法，這樣一來Alice與Bob就分別用他們的key對月餅加解密，然後Alice解開他自己的鎖，之後Bob在解開自己的鎖。

