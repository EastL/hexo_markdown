title: The Attack and Defense of Computers-5
date: 2015-10-25 11:11:26
tags: The Attack and Defense of Computers
categories: The Attack and Defense of Computers
---
來源：[Internet Worms, Buffer Overflow Attacks, and Heap Overflow Attacks (43 ~ 82)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/2_BOA.ppt)

<h2> Countermeasures of Buffer Overflow Attacks </h2>
之前講了幾種buffer overflow的攻擊，接下來要看針對這些攻擊所採取的防禦機制：

Data Execution Prevention (DEP)：
這種機制出現在是讓data段的資料不能被執行，code段不能寫入資料，如此攻擊者無法執行shell code。此機制預設是開的，要關掉的話compile時要加上-z execstack。

StackGuard：
compiler會在return address之前加一段檢查碼，最後要ret時會去檢查剛剛加的檢查碼有沒有被改過，有的話就跳出警示。以下列程式為例(引用[ptt](https://www.ptt.cc/bbs/C_and_CPP/M.1367803345.A.2AC.html))：

{% codeblock %}
#include <stdio.h>
#include <string.h>
int main()
{
    char c[10];
    strcpy(c, "AAAAAAAAAAAAAAAAAAAAAAAAAAAA");
}
{% endcodeblock %} 

下圖列出compiler的StackGuard(canary word)機制：
   
![](/images/canary.jpg)

如圖，stack剛長出來時會增加一段canary(紅框)，最後要leave之前會將剛剛的canary拿出來比對，發現有誤就跳至__stack_chk_fail。

Return Address Defender(RAD)：
找[資料](https://en.wikibooks.org/wiki/GNU_C_Compiler_Internals/Return_Address_Defense_4_1)時意外發現.....@@。此方法是將return address存在另一個地方(return address area (RAR))，最後真正要ret時會將目前return address與RAR做比對，如果RAR裡頭不存在此return address，那麼就會被視為stack攻擊。

Address Space Layout Randomization(ASLR)：
此機制會隨機載入記憶體位置，甚至對調stack跟heap相對位置，導致攻擊者很難猜到正確位置。

![](/images/ASLR.jpg)

如圖最左邊為原本正確的memery長相，經過ASLR隨機載入加上插空隙後就變成了最右邊的樣子。

<h2> Return Oriented Programming (ROP) </h2>
由於DEP會讓資料段的程式碼無法執行，所以發展出了ROP這種攻擊。[ROP](https://en.wikipedia.org/wiki/Return-oriented_programming)主要精神就是利用ret，在現有的程式碼片段中組合成攻擊者想要執行的程式，再不斷的蓋return address來跑那些程式碼片段。ret做的事情為 pop eip：

![](/images/rop1.jpg)

如上圖在ret之前，esp指向return address，eip指向ret指令。

![](/images/rop2.jpg)

執行ret後，esp會往上指，return address存到eip中，因此eip指回caller繼續執行。攻擊者要如何操控eip流程呢？假設今天攻擊者找到兩段他可以用的程式片段，分別為
mov %eax, %ecx
ret
以及
add $03, %eax
ret
事實上這兩個程式碼片段是在不同的記憶體位置，一個在0x08900200一個在0x09012800，攻擊者透過stack buffer smashing把return address以及return address+4的位置蓋成了剛剛兩個程式碼片段的所在位置，那麼程式就會去跑剛剛兩串程式碼片段。下圖為攻擊者程式執行之前的內容，esp指向return address，eip指向ret：

![](/images/rop3.jpg)

第一次ret跳到了攻擊者蓋的0x08900200，eip指著下一行指令。

![](/images/rop4.jpg)

接著執行了攻擊者期望的第一行指令：mov %eax, %ecx，eip指著ret，esp指著攻擊者的第二個程式碼片段。

![](/images/rop5.jpg)

執行ret，此時eip跳到了第二個程式碼片段0x09012800，eip指著下一行攻擊碼。

![](/images/rop6.jpg)

執行了攻擊者第二行指令：add $03, %eax。

![](/images/rop7.jpg)

剛剛所講的程式碼片段就是ROP Gadget，這邊有一個[ROP Gadget tool](https://github.com/JonathanSalwan/ROPgadget)可以幫你找Gadget。

<hr>

<h2> ASLR on Linux </h2>
在2005年ASLR正式用在linux kernel 2.6.12，微軟在2007年也引進他們的kernel，不過當初開發ASLR是給linux用的，因此微軟的ASLR有時後效果並沒有那麼好。
雖然每個process都會有ASLR，但並不是每個記憶體區塊被隨機載入時都能執行。而如果要在code段實施隨機載入，則在編譯程式碼時要開啟Position Independent Executable(PIE)，library段則是一開始compile時就幫你開啟PIE了，所以library段都會是隨機載入，gcc開啟PIE的方法為加參數"-fPIE -pie"即可。下列程式是印出EIP位址：

{% codeblock %}
#include <stdlib.h>
#include <stdio.h>

void* getEIP () {
    return __builtin_return_address(0)-0x5;
};

int main(int argc, char** argv){
    printf("EIP located at: %p\n",getEIP());
    return 0;
}
{% endcodeblock %}

這邊我compile時沒有開啟PIE，因此EIP位址是不會變的，但是library是隨機載入：

![](/images/nopie_eip.jpg)

接下來我開啟PIE，可以看到EIP位址是被隨機載入記憶體當中：

![](/images/pie_eip.jpg)


