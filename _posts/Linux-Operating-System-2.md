title: Linux Operating System-2
date: 2015-10-15 20:04:06
tags: Linux Operating System
---
來源：[Introduction (64~98)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-1.ppt)

Linux Source Code Tree Overview跟System Boot up我就跳過了...

<h2> x32 Calling Conventions </h2>
這邊要介紹的是，當C呼叫一個function時參數的傳遞。關於stack frame我在[網路攻防](http://eastl.github.io/2015/10/12/The-Attack-and-Defense-of-Computers-1-3/)這邊有提到一些基礎知識，重複的在這邊就不再贅述。

假設今天G function要呼叫H function，則我們稱呼叫者G為caller，被呼叫的H function我們稱callee。為了方便講解老師投影片的圖，看一下下面這段code：

{% codeblock %}
#include <stdlib.h>
#include <stdio.h>

int G(int a)
{
    H(3);
    a = a + 1;
    return a;
}

int H(int b)
{
    char c[100];
    int i = b - 3;

    while((c[i++] = getchar()) != EOF)
    {
        printf("Hi");
    }
    return i;
}

void main()
{
    int example = 1;
    G(example);
}
{% endcodeblock %}

可以看到main function呼叫了G function，G function再去呼叫H function。現在把焦點放在G function裡面呼叫的H(3)，當G呼叫H之前，它會先把之後要給H的參數push到stack上，之後再去call H function，下圖為上面這支程式的執行檔經過objdump產生的內容：

![](/images/callerdump.jpg)

上圖可以看到G在call H之前將3這個值push到stack，那callee要如何使用這參數呢？看一下下面這張stack frame：

![](/images/call_stack_frame.jpg)

當G call了H function後，它裡面會有一個動作是將return address push上去，之後到H的stack frame時會先push ebp，因此可以利用ebp的相對位置找到參數，如下圖：

![](/images/calleedump.jpg)

由於我程式裡面做的事情是i = b - 3，可以看到compiler將ebp+8的位置載入eax，接著再將它減去3。看完caller如何傳參數給callee後，接下來我們看callee如何將參數return給caller，其實很簡單，caller只要把要return的值塞進去eax暫存器就好了：

![](/images/return_conv.jpg)

看一下剛剛G function，我直接將傳進來的參數a return回去，所以assembler就直接將ebp + 8位置的值丟進eax了。



