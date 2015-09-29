title: Applied-Cryptography-2
date: 2015-09-29 05:56:36
tags: Applied Cryptography
---
來源 ： [chapter 1 (24~28)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-1%20Introduction%20to%20Cryptography.pdf)，[chapter 2 (1~21)](http://staff.csie.ncu.edu.tw/yensm/lecture/Cryptography/Chapter-2%20Secret-Key%20Cryptography.pdf)，[wiki(Lagrange polynomial)](https://en.wikipedia.org/wiki/Lagrange_polynomial)

在前一篇提到金鑰管理，是利用方程式上的點來分散管理，那麼要如何將點還原成方程式呢？

## Lagrange polynomial interpolation

定義：  
給定一集合，此集合有k+1個不重複的點 
<center> $(x_0,y_0) \ \ , \ \ ... \ \ , \ \ (x_i,y_i) \ \ , \ \ ... \ \ , \ \ (x_k,y_k)$ </center>
我們定義Lagrange form為：
<center> {% math %}L(x) \ \ := \ \ \sum_{j=0}^k y_j l_j(x) {% endmath %} </center>
<center> {% math %}l_j(x) \ \ := \ \ \prod_{0 \leq m \leq k,m \neq j} \ \ \frac{x-x_m}{x_j-x_m} {% endmath %} </center>
L(x)即為此k+1個點的方程式。由於{% math %}l_j(x){% endmath %}在{% math %}x=x_j{% endmath %}時等於1，其餘值時等於0，故：
<center>  {% math %} l_j(x_i) \ \ = \ \ \delta_{ij} \ \ = \ \ \{^{1, \ \ if \ \ j=i}_{0, \ \ if \ \ j \neq i} {% endmath %} </center>
所以：
<center> {% math %} L(x_i) \ \ = \ \ \sum_{j=0}^k y_j l_j(x_i) \ \ = \ \ \sum_{j=0}^k y_j \delta_{ij} \ \ = \ \ y_i {% endmath %} </center>
故得到{% math %} L(x_i) \ \ = \ \ y_i {% endmath %}，此k+1個點為L(x)上的點。
