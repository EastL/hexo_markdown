title: Linux Operating System-2
date: 2015-10-15 20:04:06
tags: Linux Operating System
categories: Linux Operating System
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

<h2> x64 Calling Conventions </h2>
x64與x32不一樣的是，如果傳遞的參數在六個以下時，它是利用暫存器去傳送的，看一下下面程式：

{% codeblock %}
int G(int a, int b, int c, int d, int e, int f, int g, int h)
{
    return g;
}

void main()
{
    int a;
    a = G(1, 2, 3, 4, 5, 6, 7, 8);
}
{% endcodeblock %}

我把上面的程式碼丟到64 bit ubuntu編譯後，objdump如下圖：

![](/images/x64_call.jpg)

可以看到紅色框的部分6個參數都移到了暫存器當中，而當到第七個參數時，compiler是把它放到了rsp的位置，從stack底端往上放。

![](/images/x64_callerstack.jpg)

而callee要拿參數時，前六個當然是從暫存器拿，第七個開始就從rbp + 16開始：

![](/images/x64_callee.jpg)

由於我是return g，它是第七個參數，所以將rbp + 10(hex)移到eax。

![](/images/x64_calleestack.jpg)

<h2> Callee-Save Register vs. Caller-Save Register </h2>
這可能會被文字混淆...

caller saved register意思是當caller要呼叫callee時，caller會預先把某幾個暫存器的值存在stack上，然後呼叫了callee後callee可以使用者些暫存器。所以caller saved register是給callee用的。

而callee saved register是當caller呼叫了callee時，為了確保不會更動到register的值，callee會先把暫存器的值存起來，等到做完所有動作要return回caller之前再把剛存起來的值還給暫存器，所以callee saved register是callee幫caller存的。

<h2> IA32 Process Address Space Layout </h2>

Address space意思是一堆address的集合，在32bit的linux環境中每個process都會有4G byte的address space，前3G byte為user address space，從0x00000000 ~ 0xBFFFFFFF；最後1G byte則是kernel address space，從0xC0000000 ~ 0xFFFFFFFF，所以學linux kernel的人要知道0xC0000000，這非常重要!!!!(老師說的XDD)。下圖就是IA32 linux的address space：

![IA32 Linux Process Address Space Layout](/images/IA32_address_space.jpg)

<h2> x64 Process Address Space Layout </h2>

x64的address space比較特別，由於他有{% math %} 2^{64} {% endmath %}bit的address，相當於16EB，如果要將這麼大的virtual address都implement到實際的記憶體上的話，會造成page table複雜度提升，況且正常作業系統也用不到這麼龐大的記憶體空間，於是AMD決定只用前48bit的address，使用空間大概是256TB，創造了一個叫做canonical form的規則。

![canonical form](/images/cannonical.jpg)

這規則其實還蠻人性化的(?  看一下目前48bit的設計，整個address space分為兩大區塊，一邊從底部長另一邊從頂部長，先說明一下canonical form的規則，它除了規定只能用前48個bit之外還外加了一個規定，就是沒用到的bit，從第49個bit開始往後都要跟第48個bit一樣，也就是說如果第48個bit為0之後就全部都要是0，如果第48個bit為1之後就都要是1，有了這個規則後，如果你的記憶體是單向長的話，當你長到00008000 00000000的時候，你的第48個bit為1，但第49bit後卻全都是0，這樣就違反了canonical form。

而user space跟kernel space剛好一人一半(從kernel 2.6.11版本後)，高位址的部分是kernel address，另一半低位址則是user space。下面的位址分部是參考[這邊](https://www.kernel.org/doc/Documentation/x86/x86_64/mm.txt)。

{% codeblock %}
0000000000000000 - 00007fffffffffff (=47 bits) user space, different per mm
hole caused by [48:63] sign extension
ffff800000000000 - ffff87ffffffffff (=43 bits) guard hole, reserved for hypervisor
ffff880000000000 - ffffc7ffffffffff (=64 TB) direct mapping of all phys. memory
ffffc80000000000 - ffffc8ffffffffff (=40 bits) hole
ffffc90000000000 - ffffe8ffffffffff (=45 bits) vmalloc/ioremap space
ffffe90000000000 - ffffe9ffffffffff (=40 bits) hole
ffffea0000000000 - ffffeaffffffffff (=40 bits) virtual memory map (1TB)
... unused hole ...
ffffec0000000000 - fffffc0000000000 (=44 bits) kasan shadow memory (16TB)
... unused hole ...
ffffff0000000000 - ffffff7fffffffff (=39 bits) %esp fixup stacks
... unused hole ...
ffffffff80000000 - ffffffffa0000000 (=512 MB)  kernel text mapping, from phys 0
ffffffffa0000000 - ffffffffff5fffff (=1525 MB) module mapping space
ffffffffff600000 - ffffffffffdfffff (=8 MB) vsyscalls
ffffffffffe00000 - ffffffffffffffff (=2 MB) unused hole
{% endcodeblock %}
