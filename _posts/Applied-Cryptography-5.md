title: Applied Cryptography-5
date: 2015-10-21 21:20:38
tags: Applied Cryptography
categories: Applied Cryptography
---
來源：[chapter 3(23~32)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-3%20Number%20Theory.pdf)，[Number Theory II (3 ~ 4)](http://web.math.isu.edu.tw/yeh/2013Fall/GE/Lectures/L5/L5.pdf)

在上一次的尤拉函數中，我們推得了費瑪理論的通式{% math %} a^{ \phi (n) } \ \ mod \ \ n \ \ = \ \ 1 \ \ if \ \ gcd(a,n)=1 {% endmath %}，這個式子對於餘數系統的乘法反元素非常有幫助，以上面例子來說，a在餘數系統中的乘法反元素就是{% math %} a^{ \phi (n) - 1} {% endmath %}，但如果考慮到計算量的話我們會想讓a的指數越小越好，因此在n=pq, p、q都是質數情況下，我們可以將{% math %} a^{ \phi (n) } \ \ mod \ \ n \ \ = \ \ 1 {% endmath %} 改成{% math %} a^{ lcm(p-1,q-1) } \ \ mod \ \ n \ \ = \ \ 1 {% endmath %}。

在證明之前先講會用到的理論，當x mod A = 1且x mod B = 1，則x mod AB = 1。

再來我們看一下費瑪定理：
<center> {% math %} a^{p-1} \ \ mod \ \ p \ \ = \ \ 1 {% endmath %} </center>
<center> {% math %} a^{q-1} \ \ mod \ \ q \ \ = \ \ 1 {% endmath %} </center>

故{% math %} a^{(p-1)k} \ \ mod \ \ p \ \ = \ \ 1 {% endmath %}，以p-1為周期到達p-1的倍數mod p後又會等於1。因此我們可以找到lcm(p-1, q-1)，使得{% math %} a^{ lcm(p-1,q-1) } \ \ mod \ \ p \ \ = \ \ 1 {% endmath %} 且{% math %} a^{ lcm(p-1,q-1) } \ \ mod \ \ q \ \ = \ \ 1 {% endmath %}，再利用剛剛的理論得到{% math %} a^{ lcm(p-1,q-1) } \ \ mod \ \ n \ \ = \ \ 1 {% endmath %}。

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

由於在電腦做運算時，除以2相當於往右shift一個bit，判斷是否為偶數也是看最後一個bit是否為0，因此先將所有2的因數提出來，紀錄在g，接著X與Y沒有2的公因數後代表著有2的因數也沒用了，除掉；接下來利用剛剛的gcd(b, a mod b) = gcd(a, b)原理，可以知道gcd(b, a - b) = gcd(a, b)，可以將X跟Y相減後又跑出2的因數了(奇數-奇數=偶數)，除掉，然後再繼續計算gcd(|X-Y|/2, XorY)，直到最後X為零時Y為因數2以外的公因數，因此Yg為最大公因數。

接著利用輾轉相除法來計算模數的乘法反元素，首先先來看一下下列的線性組合證明(貝祖定理)：
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
下列為歐基里德輾轉相除找線性組合的演算法(貝祖等式)：
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

此演算法其實就是貝祖等式的[Iterative](https://en.wikipedia.org/wiki/Iterative_method)做法，普遍高中利用輾轉相除求線性方程的整數解都是用Recursive Method，Recursive做法就是先把a跟b輾轉相除算出最大公因數後，再將每次相除的過程式子一一列出，列到最大公因數出現後，再一行一行的把數字替換成a跟b的倍數，詳細介紹可以看[這邊](https://ccjou.wordpress.com/2012/11/16/%E5%88%A9%E7%94%A8%E5%9F%BA%E6%9C%AC%E5%88%97%E9%81%8B%E7%AE%97%E5%AF%A6%E7%8F%BE%E6%93%B4%E5%B1%95%E6%AD%90%E5%B9%BE%E9%87%8C%E5%BE%97%E6%BC%94%E7%AE%97%E6%B3%95/)。而接下來將介紹貝祖等式的Iterative概念，我們以80跟36來做例子：

![](/images/pezu.jpg)

上列式子先給定兩個式子①跟②，而這兩個式子要做的事情就是右邊的輾轉相除法得到的結果，圖中可以看到我一直維持著 80x + 36y = a 的線性式子。

![](/images/pezu2.jpg)

上圖可以知道最後輾轉相除結果是0時，最大公因數就是上一次的答案，也就是4，而我算完最大公因數時，貝祖等式也就出現了(Iterative)。上面的演算法就是利用此精神撰寫，首先初始化X，X1為f的係數，X2為d的係數，X3為f跟d的線性組合結果，Y跟T也是此結構，所以每次做完一輪輾轉相除Y是最新的結果，因此當最後Y3等於0時表示X3就是最大公因數，等於1時兩數互質。

<hr>

<h2> Chinese Remainder Theorem (C.R.T.) </h2>
中國餘數定理起源為[韓信點兵](http://episte.math.ntu.edu.tw/articles/sm/sm_01_01_2/index.html)，「兵不知其數，三三數之剩二，五五數之剩三，七七數之剩二。」這句話點出了中國餘數定理的精隨，也成為現代數論跟幾何重要定理之一。上面這句話有個重要的概念：當你要處理某個巨大複雜的數據時(兵不知其數)，可以將此數據分散處理(三三數之剩二，五五數之剩三，七七數之剩二)，處理完後再合併為原來數據。接下來介紹中國餘數定理(孫子定理)：
令{% math %} z_1, \ \ z_2, \ \ ... \ \ , \ \ z_k {% endmath %}為兩兩互質的整數，且{% math %} x \ \ mod \ \ z_k \ \ = \ \ x_k {% endmath %}，$x_k$為已知，那麼當{% math %} n \ \ = \ \ \prod_{i=1}^k \ \ z_i {% endmath %} 且 {% math %} x \ \ \in \ \ [0, n-1]{% endmath %}，則x能透過CRT計算出來。

![](/images/crt.jpg)

CRT在現代工業上用在許多地方，這邊用電腦當例子，上圖中左邊為超級電腦，可以處理n位元，右邊四台為一般市面的筆電。假設今天要計算很大的數字的加法跟乘法運算，我們可以將很大的數字分配給右邊四台電腦，而且四台電腦能夠同時處理較小的數字，處理完後再丟回原本的超級電腦。

<h4>CRT Form:</h4>
<center> {% math %}let \ \ n \ \ = \ \ \prod_{i=1}^k z_i \ \ and \ \ x \ \ mod \ \ z_i \ \ = \ \ x_i, \ \ then \ \ x \ \ = \ \ \sum_{i=1}^k x_i \ \ * \ \ ( \frac{n}{z_i} ) \ \ * \ \ [( \frac{n}{z_i})^{-1} \ \ mod \ \ z_i] \ \ mod \ \ n {% endmath %} </center>
這邊用i=2時說明：令$z_1$ = p，$z_2$ = q，則$x_1$在[o, p-1]，$x_2$在[0, q-1]，且pq互質，x可以透過以下算式得到：
<center> {% math %} x \ \ = \ \ (x_1 * a * q \ \ + \ \ x_2 * b * p ) \ \ mod \ \ (p*q) {% endmath %} </center>
a,b定義如下：{% math %} a \ \ \equiv \ \ q^{-1} (mod \ \ p) \  \ and \  \ b \ \ \equiv \ \ p^{-1} (mod \ \ q)  {% endmath %}
這邊運用到一項概念：(L mod(pq))mod p = L mod p，所以當x mod p時，$x_2$那項會被消掉，但我們的目的是要讓x mod p = $x_1$，因此還需要消掉q，所以我們定義了一個a在mod p底下為q的反元素，因此最後就會只剩下$x_1$。

<center> {% math %} x \ \ mod \ \ p \ \ = \ \ (x_1aq \ \ + \ \ x_2bp \ \ mod \ \ pq) \ \ mod \ \ p  {% endmath %} </center>
<center> {% math %} = \ \ (x_1aq \ \ + \ \ x_2bp) \ \ mod \ \ p {% endmath %} </center>
<center> {% math %} = \ \ (x_1 * 1 \ \ + \ \ 0) \ \ mod \ \ p \ \ ( \because aq \ \ mod \ \ p \ \ = \ \ 1 ) {% endmath %} </center>
<center> {% math %} = \ \ x_1 {% endmath %} 同理可得 {% math %} x \ \ mod \ \ q \ \ = \ \ x_2 {% endmath %} </center>

實例：

![](/images/CRT_example.jpg)
