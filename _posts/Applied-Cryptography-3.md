title: Applied Cryptography-3
date: 2015-10-10 23:38:40
tags: Applied Cryptography
---
來源：[chapter 2 (23~51)](/papers/Chapter2.pdf)

在進入Block Cipher之前，先來思考一個問題：
![](/images/QAQ.gif)
<h5> Question: 加解密，錯誤更正碼，壓縮先後順序 </h5>
假設今天你要將資料m透過網路傳給別人，但資料量太大必須壓縮，而且資料必須保密不能讓第三者看到因此要加密，並且需要保證接收方收到的資料正確無誤所以你希望加個錯誤更正碼降低接收方的錯誤率，問題來了，這三道程序的先後順序要如何排列呢？如果不按照"正確"的程序進行會發生什麼事呢？

![](/images/AC3_Q.jpg)


