title: Applied Cryptography-9
date: 2015-11-29 14:49:31
tags: Applied Cryptography
categories: Applied Cryptography
---
來源；[chapter 4 (39 ~ 56)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-4%20Public-Key%20Cryptography%28v2%29.pdf)

<h2> Rabin’s schemes </h2>
接下來談談rabin為何要選擇除四餘三的質數：這是因為開根號非常方便，假設今天p是這種除四餘三的質數，假設a為$QR_p$，則a的開根號為{% math %} \pm a^{(p+1)/4} \ \ mod \ \ p {% endmath %}。

證明：因為a為$QR_p$，所以{% math %} a^{(p-1)/2} \ \ \equiv \ \ 1 \ \ mod \ \ p {% endmath %}，我們計算{% math %} \pm a^{(p+1)/4} \ \ mod \ \ p {% endmath %}的平方：

{% math %} (\pm a^{(p+1)/4})^2  \ \ \equiv \ \ a^{(p+1)/2} \ \ \equiv \ \ a^{(p-1)/2 \ \ + \ \ 1} \ \ \equiv \ \ a \ \ \times \ \ a^{(p-1)/2} \ \ \equiv \ \ a \ \ (mod \ \ p) {% endmath %}

確定平方後會變回a，而p又是除四餘三的質數，所以(p+1)/4一定是整數。

<h4> Blum Integer </h4>

rabin所選的n就是Blum Integer，由兩個除四餘三的質數相乘。我們來看一下blum prime有什麼神奇功用：

<center> {% math %} SQ_n: \ \ y \ \ = \ \ x^2 \ \ mod \ \ n \ \ (x \ \ \in \ \ QR_n) {% endmath %} </center>

觀察上列式子會發現，x根y都是$QR_n$，也就是說定義域與對應域是一樣的，那我們感興趣一個問題，會不會有多對一狀況發生？假設$x_1$與$x_2$不相等而他們兩個平方後卻一樣，則只有兩種情況：$x_1$ = $x_2$，此種狀況不符合假設，排除；$x_1$ = $- x_2$，由於我們一開始設定的定義域是$QR_n$，故$x_2$為$QR_n$，根據上禮拜證明過的，$-x_2$一定不是$QR_n$，那$x_1$就不會等於$-x_2$了，故假設錯誤，得證此函式為一對一。

既然此函式為一對一而且定義域等於對應域，那做了此和數轉換後就相當於對定義域這集合做重排了。既然每次的平方或者開根號都是重排一次此集合，那不就代表我能夠無限次的重排，也就是無限次的開根號了？當然重排後會有一個週期，而這周期取決於n。

one-time password：我們可以利用上面的函式來做一個one-time password系統，one-time password顧名思義就是密碼只會用一次，用完就丟，那利用上式要如何做one-time password呢？首先我向系統註冊一組帳號密碼，密碼選擇呢就是隨機挑選上述的任何一個$QR_n$ x，接著用上式計算出y，y即為向系統註冊的密碼。接著之後要登入此系統時，你必須輸入的密碼就是根號y，系統會將你的密碼做平方來跟目前的y做比對，比對正確後會把目前的y值改為你這次登入的密碼，也就是說你下次要再登入時必須輸入根號x，以此類推，每次的密碼都是上一次的開根號。

使用此函式來當密碼的安全性相當於rabin了，因為如果你要破解密碼的話你必須對目欠的密碼做開根號，而如果你要對在mod n底下開根號你必須知道pq，所以要破解此one-time password就要先破解rabin了。

而大家都知道rabin開根號之後會有四個根，而這四個根當中只有一個能夠繼續被開根號，那有什麼方法能只接計算出可以繼續被開根號的根呢？

<center> {% math %} SQ_n^{-1}(y) \ \ = \ \ y^{[(p-1)(q-1)+4]/8} \ \ mod \ \ n {% endmath %} </center>

證明：我們一樣將開根號結果拿來平方看看會不會得到y。
<center>
{% math %}
\{ y^{[(p-1)(q-1)+4]/8} \}^2 \ \ \equiv \ \ y^{[(p-1)(q-1)+4]/4} \\
\equiv \ \ y^{[(p-1)(q-1)]/4} \ \ \times \ \ y \ \ (mod \ \ n)
{% endmath %}
</center>

此時$y^{[(p-1)(q-1)]/4}$如果等於1就皆大歡喜了，而這個就要用到[之前](http://eastl.github.io/2015/10/21/Applied-Cryptography-5/)所講的lcm(p−1,q−1)概念了，因為y到lcm(p−1,q−1)就已經是1了，所以再到lcm(p−1,q−1)的倍數時一定是1，那lcm(p−1,q−1)會比(p-1)(q-1)/4小嗎？由於pq都是質數，減一之後至少有一個2的因數，所以(p-1)(q-1)/4是在p跟q除了2以外沒有任何公因數情況的最大公因數，換句話說lcm(p−1,q−1)最大就是(p-1)(q-1)/4。因此我們能確定$y^{[(p-1)(q-1)]/4} \ \ = \ \ 1$。

那[(p-1)(q-1)]/4會是整數嗎？我們將pq特性代入式子中：

<center>
{% math %}
\{ y^{[(p-1)(q-1)+4]/8} \}^2 \ \ \equiv \ \ y^{[(p-1)(q-1)+4]/4} \\
\equiv \ \ y^{[(4a+3-1)(4b+3-1)]/4} \ \ \times \ \ y  \\
\equiv \ \ y^{(2a+1)(2b+1)} \ \ \times \ \ y \ \ (mod \ \ n)
{% endmath %}
</center>

a跟b都是整數，因此確定次方是能夠開根號而且為整數。既然此方法是拿來開根號，那能夠拿來做數位簽章嗎？答案是可以的，而且這會比原來利用$\pm a^{(p+1)/4}$計算再判斷$QR_n$慢兩倍，因為此種方法計算需要算出(p-1)(q-1)，而先用中國餘數定理的則是分別計算p跟q。

那有辦法拿來觧密嗎？不可能每次加密的訊息都剛好是可以開根號的那一個
所以無法用來解密。

<h2> Primality Test </h2>

質數測試我們小時候都學過，不過那套方法不能在模數底下作測試..而之後有利用費瑪定理來測試，不過缺點是錯誤機率剛好一半，而且還有 Carmichael numbers的問題。最後大多數人採用的是Miller-Rabin Test，我們先來看一下原理：

首先，費瑪告訴我們如果n為質數，則$a^{n-1} \ \ \equiv \ \ 1 \ \ (mod \ \ n)$。利用這個定理，我們進一步將n-1做分解，使$n-1 \ \ = \ \ 2^S r$的S最大，分解完後代入費瑪定理：
<center> $a^{2^S r} \ \ \equiv \ \ 1 \ \ (mod \ \ n) \ \ $所以$n \ \ | \ \ a^{r \times 2^S} \ \ - \ \ 1 $</center>
接著用國中教過的因式分解：
<center>
{% math %}
n \ \ | \ \ a^{r \times 2^S} \ \ - \ \ 1 \\
n \ \ | \ \ (a^{r \times 2^{S-1}} \ \ + \ \ 1)(a^{r \times 2^{S-1}} \ \ - \ \ 1)  \\
n \ \ | \ \ (a^{r \times 2^{S-1}} \ \ + \ \ 1)(a^{r \times 2^{S-2}} \ \ + \ \ 1)...(a^r \ \ + \ \ 1)(a^r \ \ - \ \ 1)  \\
{% endmath %}
</center>

因此如果$a^r$ mod n會是1，或者是$a^{r \times 2^j}$ mod n是n-1，則上面那一串就有可能是n的倍數，換句話說a就有可能是質數。而此演算法也不是百分之百正確，會有$ \phi (n) /4$是錯的，機率大概是1/4。

演算法：
(let $n \ \ - \ \ 1 \ \ = \ \ 2^S r$)
{% codeblock %}
Input : n and  security parameter t ≥ 1
for i = 1 to t do
  randomly select base a
  y = a^r mod n

  if (y≠1 and y≠-1) then
    j=1

  while (j ≤ s-1 and y≠-1) do 
    y = y^2 mod n
    j++

  if (y≠-1) then return “n IS composite”

Return “MAYBE n is prime”
{% endcodeblock %}

那t要取多少次呢？因為每一次有可能是質數的機率是3/4，所以重複多次來測每次都說他是質數的機率就會比較精確，正確的機率為$1 - (1/4)^t$。

<h4> Trial Division Sieve </h4>
這邊還有個小問題，由於2,3,5,7這些小質數的倍數太多個了，如果在測質數時所抓的亂數是這些質數倍數，那這個亂數就浪費了上面演算法找質數的時間。因此我們會把這些小質數的倍數去掉再丟到上面演算法做測試，那小質數要取到多少比較好呢？下表x是小質數的範圍，Q(x)則是隨機取一個亂數，此亂數不為x範圍內質數倍數的機率：

![](/images/division_sieve.jpg)

而我們通常會取1000，將機率做倒數後大概是12，代表說我每取亂數12個才會有一個數沒有1000以內的因數。

