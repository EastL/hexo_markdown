title: Applied Cryptography-7
date: 2015-11-07 14:00:25
tags: Applied Cryptography
---
來源：[chapter 3(50 ~ 54)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-3%20Number%20Theory.pdf)，[chapter 4(1 ~ 18)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-4%20Public-Key%20Cryptography%28v2%29.pdf)

<h2> Primitive Root </h2>
上禮拜有講到generator的概念，剛好我自己補充的東西也有提到order的概念，這邊會統整一下所有的概念。

Theorem: If λ is the order of α, then λ|(p-1).
proof: 這邊利用反證法，令 λ 不是p-1的因數，則 p-1 = t*λ + k，1≤ k ≤ λ −1，則：
<center> {% math %} \alpha^{p-1} \ \ \equiv \ \ \alpha^{t \lambda} \ \ \times \ \ \alpha^{k} \ \ \equiv \ \ 1 \ \ \times \ \ \alpha^{k} \ \ \equiv \ \ \alpha^{k} \ \ \equiv \ \ 1 \ \ (mod \ \ p) {% endmath %} </center>
得到 k < λ，{% math %} \alpha^{k} \ \ \equiv \ \ 1 \ \ (mod \ \ p) \ \ \rightarrow \leftarrow {% endmath %}得證。

給定一正整數n，n的RSR(Reduced set of residues)標記為{% math %} Z_{n}^{*} {% endmath %}，若a的次方mod n底下能夠產生{% math %} Z_{n}^{*} {% endmath %}的所有元素，也就是a為{% math %} Z_{n}^{*} {% endmath %}集合的generator，則我們稱a為n的Primitive Root。而{% math %} Z_{n}^{*} {% endmath %}的個數尤拉定義為φ(n)，因此如果a為n的Primitive Root，則a必須產生所有的{% math %} Z_{n}^{*} {% endmath %}，也就是說a的order必須為φ(n)，如果a的order比φ(n)小的話，代表a的次方還沒到達φ(n)就已經是一個周期了，此時不可能產生出{% math %} Z_{n}^{*} {% endmath %}集合的所有元素。下圖是摘錄自[wiki](https://en.wikipedia.org/wiki/Primitive_root_modulo_n)，圖中可以看到n選的數為14，3跟5都能夠將14所有的RSR產生出來，因此3跟5為14的primitive root：

![](/images/Primitive_root_modulo_n.jpg)

這邊值得注意的是，並不是所有的正整數n都能夠找到primitive root，像剛剛wiki裡面就有舉例子，15就會找不到primitive root了。那麼如果是有primitive root的數，那麼這個數的primitive root會有幾個？這邊查了一下蠻多文獻都寫(Burton 1989)但是都查不到證明....n的primitive root個數為φ(φ(n))。在之後的密碼系統會用到的是質數居多，這邊我就以p來表示質數，因此的primitive root個數就是φ(p-1)，從這邊可以看得出來質數的generator還蠻多的，不用怕找不到。

接下來介紹老師51頁投影片紅色框框的式子，這邊我用wiki上的[Eulers totient function](https://en.wikipedia.org/wiki/Euler%27s_totient_function)來介紹。這式子是由高斯所發現的，他發現正整數可以用尤拉函式的和來表示：
Divisor sum:
<center> {% math %} n \ \ = \ \ \sum_{d | n} \phi (d) {% endmath %} </center>

這式子乍看之下蠻神奇的，也就是說把n的所有因數經過totient function後的總和會得到n。這邊我用n=10來舉例子，我將0~1切成10等分，所有的分數形式如下：
<center> {% math %} \frac{1}{10}, \ \ \frac{2}{10}, \ \ \frac{3}{10}, \ \ \frac{4}{10}, \ \ \frac{5}{10}, \ \ \frac{6}{10}, \ \ \frac{7}{10}, \ \ \frac{8}{10}, \ \ \frac{9}{10}, \frac{10}{10} {% endmath %} </center>
接著我們約分：
<center> {% math %} \frac{1}{10}, \ \ \frac{1}{5}, \ \ \frac{3}{10}, \ \ \frac{2}{5}, \ \ \frac{1}{2}, \ \ \frac{3}{5}, \ \ \frac{7}{10}, \ \ \frac{4}{5}, \ \ \frac{9}{10}, \ \ \frac{1}{1} {% endmath %} </center>
到這邊可能還有點模糊，我們將上列數字依照分母分類然後由小排到大：
<center> {% math %} \frac{1}{1}, \ \ \frac{1}{2}, \ \ \frac{1}{5}, \ \ \frac{2}{5}, \ \ \frac{3}{5}, \ \ \frac{4}{5}, \ \ \frac{1}{10}, \ \ \frac{3}{10}, \ \ \frac{7}{10}, \ \ \frac{9}{10} {% endmath %} </center>
有沒有發現一件神奇的事情，分類完後分母就是10的因數，分子是分母無法整除的數(廢話嗎不然怎叫約分)，既然如此上面不就可以表示成各個因數的totient function總和了？理解完畢後老師的投影片的式子就比較容易理解：

![](/images/divisor_sum.jpg)

接著來看在mod 質數p之下如何找出generator：剛剛有講過generator的特性，假設g為p的generator，則g的order必須為φ(p)，也就是說g的次方在p-1之前都不能等於一，利用這一個特性我們可以在小於p的所有數都拿來測，只要他一次方兩次方三次方到p-2次方都不為一，只有在p-1次方mod p等於一時就是generator了。

不過這麼做好像有點暴力，我們剛剛有一個理論是" If λ is the order of α, then λ|(p-1). "，這樣就好辦多了，既然order只會發生在p-1的因數裡面，那我們只需要檢查p-1的因數就行了。至於該如何檢查p-1的因數呢？這邊可以不用每一個因數都試，舉個簡單的小例子：假設p-1 = a*b*c，那我們只需要測ab,bc,ac這三個看看這些次方會不會等於一就好了。因為如果$g^{ab}$ mod p不等於一，那麼$g^{a}$跟$g^{b}$都不會等於一了，以此類推，但相反的如果只測試a,b,c這三個，那麼有可能$g^{a}$跟$g^{b}$都不是1，但是$g^{ab}$卻是1這種狀況。

接著我們看實際的演算法，用剛剛的例子套用到這上面來看的話，他每次就是挑掉一個因數，然後測試其餘的乘積的次方是否為一：

![](/images/generator_try_and_error.jpg)

有了這個演算法後我們就能找出質數p的generator了～但是實際上並沒有這麼順利，在現實生活中p是個很大的質數，所以要將p-1做質因數分解是很難的，那我們剛剛不就白解了嗎....實際運用上通常都是設定p-1等於兩倍的令一個質數，這樣質因數分解就是2跟剛剛設定的質數，之後會講到如何製造質數。


