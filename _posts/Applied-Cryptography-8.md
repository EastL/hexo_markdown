title: Applied Cryptography-8
date: 2015-11-22 11:21:39
tags: Applied Cryptography
categories:  Applied Cryptography
---
來源：[chapter 4 (19 ~ 38)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-4%20Public-Key%20Cryptography%28v2%29.pdf)

<h2> Encryption by Encryption by Public Discussion in 1974 </h2>

這種加密方法是類似於D-H公開交換金鑰方法的精神，但也因為這樣所以man-in-the-middle attack也對此加密方法有效。

一樣用Alice跟Bob做例子，假設Alice要運用此演算法加密，它先將明文算到x次方後mod p，{% math %} C_1 \ \ = \ \ m^x \ \ mod \ \ p {% endmath %}，將$C_1$傳給Bob。接著Bob收到$C_1$後，將它算到y次方mod p，{% math %} C_2 \ \ = \ \ C_1^y \ \ mod \ \ p \ \ = \ \ m^{xy} \ \ mod \ \ p {% endmath %}，然後將$C_2$傳回給Alice。(注意這傳送過程中x跟y都只有加密的那個人才知道)

接著Alice用乘法反元素算出x'，將Alice一開始加密的x消掉，記算出{% math %} C_3 \ \ = \ \ C_2^{x'} \ \ mod \ \ p \ \ = \ \ m^y \ \ mod \ \ p {% endmath %}，接著將$C_3$傳給Bob。最後Bob依樣畫葫蘆再計算出y'消掉y，計算出{% math %} m \ \ = \ \ C_3^{y'} \ \ mod \ \ p {% endmath %}就得到m啦！
{% math %} (x' \ \ \times \ \ x \ \ \equiv \ \ 1 \ \ (mod \ \ p-1), \ \ y' \ \ \times \ \ y \ \ \equiv \ \ 1 \ \ (mod \ \ p-1) ) {% endmath %}

<h2> Security Consideration of RSA </h2>

接下來探討RSA的安全性，RSA演算法第一步就是要找質數，我們來看一下"質數"這東西：首先先介紹Prime-counting function π(x)， π(x)代表的是x以下總共有多少個質數，而當x趨近於無窮大時，{% math %} \lim_{x \rightarrow \infty} \frac{\pi (x)}{x/ln(x)} \ \ = \ \ 1 {% endmath %} ，而我們在實作時所選的質數大概是512 bit，那麼所涵蓋的質數可以看到下表：

![](/images/Prime-counting_function.jpg)

所以不太需要擔心所選的質數會跟別人一樣。而質數的密度有多高呢？注意一下π(x) = x/ln(x)，我們可以看成1/ln(x)就是質數的密度，這是在小於x底下的所有正整數有1/ln(x)是質數，而所有小於x的正整數有一半是奇數，我們知道質數一定是奇數，所以如果直接從奇數裡面來抓質數的話密度又更高了，在奇數裡質數所佔的密度為2/ln(x)，我們x代512的話可以得到2/ln(512) = 1/177，每177個奇數數裡面就有一個質數。

接下來分析攻擊RSA的方法，要攻擊的話會想要做的事情是取得private key(d, n)，而public key(e, n)是已知的。所以如果我們要知道d的話必須知道φ(n)才能計算，而不管攻擊者是直接計算出d或者是直接計算出φ(n)，這兩者都相當於質因數分解n。而Shamir & Tromer有發展出TWIRL device可以分解RSA 1024bit，不過需要耗費1000萬美金，6個禮拜才有辦法破解，成本相當高。

而大質數得質因數分解目前還是個難題，但如果這個大質數是由一群小質數所組成，又或者因數裡只有一個較大的質數，則能夠用[quadratic sieve](/papers/quadsievex.pdf)分解開來，所以之後發展出了strong prime的結構：

![](/images/strong_prime_struct.jpg)

這結構是這樣的：一個大質數P在+1或-1後一定能被2分解，分解完後如果還有其他小質數沒關係，但一定要有一個很大的質數在，而這個很大的質數暫且稱做P'，而P'再經過+1或-1後，仍然還要有剛剛講的特性，必須分解完後還要再擁有一個較大的質數，此結構稱為strong prime。雖然現在的[generalized number field (GNF)](https://en.wikipedia.org/wiki/General_number_field_sieve) sieve分解方法不管你是不是strong prime都沒差，不過有些特殊系統還是會需要用到此特性。
