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

假設{% math %} x^2 \ \ \equiv \ \ a \ \ \equiv \ \ y^2 \ \ (mod \ \ p){% endmath %}，且x,y不相等，{% math %} 1 \ \ \leq \ \ x,y \ \ \leq \ \ /frac{p-1}{2} {% endmath %}，則x不是跟y一樣就是p-y，但這兩種都不符合條件，因此平方根不會同時有兩個小於(p-1)/2，也就是在小於(p-1)/2的情況下會有兩個數平方後mod p會是同一個數。
