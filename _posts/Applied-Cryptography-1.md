title: Applied_Cryptography_1
date: 2015-09-28 11:21:06
tags: Applied Cryptography
---
來源 ： [投影片(1~23)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-1%20Introduction%20to%20Cryptography.pdf)

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
<center> proof : ${Y_A}^{X_B} \ \ mod \ \ p \ \ = \ \ {Y_B}^{X_A} \ \ mod \ \ p$</center>
<center> ${Y_A}^{X_B} \ \ mod \ \ p \ \ = \ \ ({g^{X_A} \ \ mod \ \ p})^{X_B} \ \ mod \ \ p$ </center>
<center> 令$g^{X_A} \ \ mod \ \ p \ \ = \ \ a$，利用餘式定理$a = g^{X_A} - pK$ </center>
<center> 則 ： $({g^{X_A} - pK})^{X_B} \ \=\ \g^{{X_A}{X_B}}\ \+\ \C^{X_B}_1 g^{\left {X_B}-1\right b}pK +‧‧‧+({pK})^{X_B}$ </center> 
