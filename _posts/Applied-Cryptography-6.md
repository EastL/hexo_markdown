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

