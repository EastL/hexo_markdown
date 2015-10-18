title: The Attack and Defense of Computers-4
date: 2015-10-18 13:36:51
tags: The Attack and Defense of Computers
---
來源：[Internet Worms, Buffer Overflow Attacks, and Heap Overflow Attacks (21 ~ 40)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/2_BOA.ppt)

<h2> Return-into-libc Attacks </h2>

上一堂課講的stack smashing是利用return address跳到攻擊者所寫的shell code，由於攻擊者注入的shell code會是在data段，現在大多電腦data段是無法執行code的，所以這邊發展了另一種形式的buffer overflow，叫做Return-into-libc Attacks，它是利用受害程式裡頭現有的程式碼下去執行，所以不會造成data段的code無法執行這問題。上一章節講過會利用buffer overflow幾乎都是要拿來拿shell，這邊return to libc也是一樣，是利用libc裡面的system function，不過如果要執行shell的話必須執行system(/bin/sh)，這邊來看一下如何將參數"/bin/sh"塞給system。

在[Linux Operating System-2](http://eastl.github.io/2015/10/15/Linux-Operating-System-2/)有講過calling conventions，簡單來說就是caller在呼叫callee之前會用push的方式將要傳的參數丟到stack上，callee要使用這些參數時是直接存取ebp的相對位置，因此我們可以利用這一點將/bin/sh傳給system。在如何傳參數前要先了解compiler幫我們增加的程式碼是做什麼用的。

<h6> Function Prologue and Epilogue </h6>
其實Prologue and Epilogue所做的事情就是幫我們長出stack跟回收stack。這邊有個小程式來幫助理解：

{% codeblock %}
int G(int a, int b, int c, int d, int e, int f, int g, int h)
{
    int aa;
    aa = 1;
    return g;
}

void main()
{
    int a;
    a = G(1, 2, 3, 4, 5, 6, 7, 8);
}
{% endcodeblock %}

上面程式碼就只是main function call了G function，只是參數傳的有點多...接下來我們來看Prologue是如何長出stack的，下面是compiler把上面程式翻譯後G function的組語：

{% codeblock %}
080483eb <G>:
 80483eb:       55                      push   %ebp
 80483ec:       89 e5                   mov    %esp,%ebp
 80483ee:       83 ec 10                sub    $0x10,%esp
 80483f1:       c7 45 fc 01 00 00 00    movl   $0x1,-0x4(%ebp)
 80483f8:       8b 45 20                mov    0x20(%ebp),%eax
 80483fb:       c9                      leave
 80483fc:       c3                      ret
{% endcodeblock %}

可以看到我真正的程式碼在組語裡面是第四行的 movl   $0x1,-0x4(%ebp)，這行動作是assign local變數，而這行的前三行做的事情就是幫我們製造stack出來：首先先將原本ebp值存起來，接著將目前指著頂端的esp指向ebp，然後esp往前了0x10個byte，這麼一來stack就往下長了16byte的空間出來給G function使用。

那G function做完事情後要如何回收stack呢？接著我們談Epilogue，看一下剛剛組語的最後兩行，leave跟ret，這兩行指令就是在回收stack用的，看一下老師的投影片：

![](/images/proepi.jpg)

上面紅色部分是剛剛將得如何長出stack，下面的藍色就是回收stack，可以看到leave做的事情是將esp從stack頂端指回ebp，接著將之前製造stack時存起來的ebp還給現在的ebp，ebp就指向caller的ebp。用GDB來看證在執行的程式目前狀況來解釋：

![](/images/gdb_epi.jpg)

上圖是gdb執行的結果，這張圖有三個部分，最上面的是目前所有暫存器的值，中間那塊是記憶體中的code段，會有eip指著下一行要執行的指令，最下面則是stack內容，從esp開始往下共28byte。這張圖目前執行到的指令是我藍色框起來的部分，也就是在G function裡面leave的前一行指令，這行指令是將ebp+20位置的值移到eax，很明顯的這是要處理G function要return給main的值，我程式碼中回傳的是g，g=7，所以可以看到左上腳紅色框現在eax的值是7。接著執行下一行指令：

![](/images/gdb_epi2.jpg)

執行完leave後可以看到stack裡面最上面目前剩下return address，接著是之前main所push的參數，所以應證了老師投影片裡面的leave做的事情，我們來比較執行leave前後暫存器裡面的值：

![](/images/gdb_epi3.jpg)

執行leave前ebp的位置是0xbffff530，ebp指向0xbffff568，0xbffff568是main的ebp位置；執行leave後會先讓esp指到ebp，所以esp會先變成0xbffff530，接著pop ebp，pop會把目前stack頂端的值回收，所以esp值+4變成0xbffff534，而pop還會將stack頂端裡面存放的值丟給後面的參數，所以將原本stack頂端的0xbffff568存到ebp裡面，因此ebp就變成了0xbffff568。  
接著執行ret，ret做的事情就是pop eip，讓eip指向caller call function的下一行指令，所以程式就會回到main function。

整理一下剛剛學到的訊息，stack開始前會先push ebp，結束時會將stack回收再回到caller，回到剛剛的問題，要如何將參數"/bin/sh"給system()呢？當然第一步跟之前的stack smashing一樣，先把system()的位置蓋在return address上，那麼受害程式執行完leave、ret後，會到達system()的位置，然後system()執行Prologue會push ebp，接著system()要執行caller給他的參數時會去ebp+8的位置去拿，如果這時他在ebp+8拿到的是"/bin/sh"我們就成功了，看一下下面這張圖：

![](/images/ret2lib_stack.jpg)

上圖中我們將return address蓋成system()位置外，我們還將return address+8的位置蓋成了字串/bin/sh的起始位置。

![](/images/ret2lib_stack2.jpg)

接著受害程式執行完leave，return address底下的stack被回收了。

![](/images/ret2lib_stack3.jpg)

執行ret，再將return address pop給eip，因此eip目前指向system()的位置。

![](/images/ret2lib_stack4.jpg)

接著執行system()時，會先push ebp，接著準備好要給system()用的stack，此時我們剛剛塞的"/bin/sh"字串起始位置正好是ebp+8，大功告成。   
這邊額外講一下我一開始的盲點，我一直認為return address上面的參數不是會被回收嗎？那剛塞的/bin/sh字串不就被回收掉了......其實回收這動作是在caller當中，而你return到的地方正常來講應該是要到caller，但是這邊攻擊流程是讓他到達system()，所以參數並沒有被回收。

/* FIXME */
heap跟BSS的overflow老師這邊很快就帶過，我有時間研究一下後再補上。
