title: Applied Cryptography-5
date: 2015-10-21 21:20:38
tags: Applied Cryptography
---
來源：[chapter 3(23~31)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-3%20Number%20Theory.pdf)[Number Theory II (3 ~ 4)](http://web.math.isu.edu.tw/yeh/2013Fall/GE/Lectures/L5/L5.pdf)

在上一次的尤拉函數中，我們推得了費瑪理論的通式{% math %} a^{ \phi (n) } \ \ mod \ \ n \ \ = \ \ 1 \ \ if \ \ gcd(a,n)=1 {% endmath %}，這個式子對於餘數系統的乘法反元素非常有幫助，以上面例子來說，a在餘數系統中的乘法反元素就是{% math %} a^{ \phi (n) - 1} {% endmath %}，但如果考慮到計算量的話我們會想讓a的指數越小越好，因此在n=pq, p、q都是質數情況下，我們可以將{% math %} a^{ \phi (n) } \ \ mod \ \ n \ \ = \ \ 1 {% endmath %} 改成{% math %} a^{ lcm(p-1,q-1) } \ \ mod \ \ n \ \ = \ \ 1 {% endmath %}。(關於此式子小弟還沒想通，之後再補上證明)

有了餘數系統的乘法反元素後，RSA將他應用在加密演算法上：

令{% math %} \phi (n) \ \ + \ \ 1 \ \ = \ \ A \ \ \times \ \ B {% endmath %}
(A, n)為public key
密文 {% math %}C \ \ = \ \ m^A \ \ mod \ \ n {% endmath %}，m為明文
(B, n)為private key
解密 {% math %} C^B \ \ mod \ \ n \ \ = \ \ m^{AB} \ \ mod \ \ n \ \ = \ \ m^{ \phi (n)+1} \ \ mod \ \ n \ \ = \ \ m {% endmath %} 
    
這是RSA剛開始的雛形。上式的解密用到{% math %} (m^A \ \ mod \ \ n)^B \ \ mod \ \ n \ \ = \ \ m^{AB} \ \ mod \ \ n {% endmath %}，此證明我在[Applied Cryptography-1](http://eastl.github.io/2015/09/28/Applied-Cryptography-1/)中講Diffie-Hellman如何傳送public key時有證明，可以參考。

<h2> Another Approach for Computing Multiplicative Inverse </h2>
這邊介紹在餘數系統時另一種找反元素的方法，會用到的是歐基里德的輾轉相除法：
*   for non-negative a and b:
    gcd(b, a mod b) = gcd(a, b)  (a ≥ b)

    proof:
    <center> let gcd(a, b) = t, a = ta', b = tb' and gcd(a', b') = 1</center>
    <center> gcd(a mod b, b) = gcd(a-kb, b) = gcd(t(a'-kb'), tb') </center>
    <center> {% math %} \because {% endmath %} gcd(a', b') = 1 </center>
    <center> {% math %} \therefore {% endmath %} gcd(a'-kb', b') = 1 </center>
    <center> {% math %} \Rightarrow {% endmath %} gcd(a mod b, b) = t = gcd(a, b) </center>

有了gcd(b, a mod b) = gcd(a, b)就能用輾轉相除法來球最大公因數。下面為輾轉相除法的演算法：

EUCLIDEAN(a, b), a≥b>0
{% codeblock %}  
X = a; Y = b
if (Y==0) then return X (X=gcd(a, b))
R = X mod Y
X = Y
Y = R
goto step 2
{% endcodeblock %}

上列輾轉相除顯而易懂，做完mod後除數變成被除數，餘數變成除數，若最後除數為零時被除數即為最大公因數。接下來介紹輾轉相除優化過的演算法：

*   Binary Euclidean algorithm:
    EUCLIDEAN(a, b), a≥b>0
{% codeblock %}
X = a; Y = b; g = 1
while (both X and Y are even) do
    X=X/2; Y=Y/2; g=2g

while (X≠0) do
    while (X is even) do X=X/2
    while (Y is even) do Y=Y/2
    T=|X-Y|/2
    if (X≥Y) then X=T; else Y=T
return Y*g (=gcd(a, b))
{% endcodeblock %}

由於在電腦做運算時，除以2相當於往右shift一個bit，判斷是否為偶數也是看最後一個bit是否為0，因此先將所有2的因數提出來，紀錄在g，接著X與Y沒有2的公因數後代表著有2的因數也沒用了，除掉；接下來利用剛剛的gcd(b, a mod b) = gcd(a, b)原理，可以知道gcd(b, a - b) = gcd(a, b)，可以將X跟Y相減後又跑出2的因數了(奇數-奇數=偶數)，除掉，然後再繼續計算gcd(|X-Y|/2, XorY)，值到最後X為零時Y為因數2以外的公因數，因此Yg為最大公因數。

接著利用輾轉相除法來計算模數的乘法反元素，首先先來看一下下列的線性組合證明：
*   ax + by = gcd(a,b)  (a,b,x,y 為整數) 證明此線性組合存在
    
    <center> 假設 ax + by = m，m為a與b的線性組合最小正整數解 </center>
    <center> gcd(a, b)|a，gcd(a, b)|b，{% math %} \therefore {% endmath %} gcd(a, b)|m </center>
    <center> 得知gcd(a, b) ≤ m  -----(1) </center>
    <center> 令 a = mq + r (0 ≤ r < m) </center>
    <center> a = (ax + by)q + r </center> 
    <center> r = a(1 - xq) - b(yq) ，r為a與b線性組合解 </center>
    <center> {% math %} \because {% endmath %} r < m, m為最小正整數解 </center>
    <center> {% math %} \therefore {% endmath %} r = 0, a = mq </center>
    <center> {% math %} \Rightarrow {% endmath %} m|a，同理可證 m|b </center>
    <center> {% math %} \Rightarrow {% endmath %} m 為 a 跟 b 的公因數 </center>
    <center> 得知 m ≤ gcd(a, b) -----(2) </center>

由上述(1)(2)可以知道gcd(a, b) ≤ m且m ≤ gcd(a, b)，因此 m = gcd(a, b)得證。所以如果要找 mod f 情況下d的反元素(f,d互質)，我們可以知道gcd(f,d)=1，因此 fa + db = 1必定存在，而{% math %} fa \ \ + \ \ db \ \ \equiv \ \ db \ \ (mod \ \ f) {% endmath %}，故{% math %} d^{-1} \ \ \equiv \ \ b \ \ (mod \ \ f){% endmath %}。
下列為歐基里德輾轉相除找線性組合的演算法：
*   Extended EUCLIDEAN(d, f), f>d>0
    {% codeblock %}
    (X1,X2,X3) = (1, 0, f), (Y1,Y2,Y3) = (0, 1, d)
    if (Y3==0) return gcd(d, f) = X3
    if (Y3==1) return gcd(d, f) = Y3
    Q = floor(X3 / Y3)
    (T1,T2,T3) = (X1-QY1, X2-QY2, X3-QY3)
    (X1,X2,X3) = (Y1,Y2,Y3)
    (Y1,Y2,Y3) = (T1,T2,T3)
    goto step 2
    {% endcodeblock %}   

