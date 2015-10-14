title: Linux Operating System-1
date: 2015-10-14 20:02:44
tags: Linux Operating System
---
來源：[Introduction (1~52)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-1.ppt)，[x86-64](https://zh.wikipedia.org/wiki/X86-64)，[x86](https://zh.wikipedia.org/wiki/X86)

<h2> Intel x86 Architecture </h2>

講x86之前當然要先講一下它的由來，x86是intel的指令集架構，Intel這家公司成立於1968年，他最早創造了全球第一顆微處理器4004，然後演變成了8位元的CPU 8008，到了1978年x86架構正式誕生，第一款x86架構的處理器是8086，它是一個16位元的處理器，直到80386之後變成了32位元的cpu，也許是因為都是86結尾所以才叫做x86?到了2003年AMD這家公司針對x86架構開發了64位元的CPU，取名叫做AMD64，這AMD64可以相容之前的16、20、32位元的程式，所以廣受大家歡迎，Intel就採用了AMD64，並且把它叫做"Intel 64"，也就是現在統稱的x86-64或簡稱x64，「IA-32e」及「EM64T」也是指相同的東西，唯一要注意的是IA-64並不是指現在的x64，IA-64是在AMD64之前Intel與其他公司合作開發出來的架構，但他跟x86架構不相容，所以市場比較冷淡。下圖是老師投影片中的微處理器進化史：

![Evolution of Intel Microprocessors](/images/microprocessors.jpg)

接下來的x86跟x64暫存器這邊就不介紹了.....

<h2> Real Mode vs. Protected Mode </h2>

在x86架構的處理器有好幾種mode，這邊介紹IA-32的real mode跟protected mode。IA32的CPU開機後，會先執行BIOS的程式，BIOS檢查程序執行完後會將執行權交給boot loader，然後boot loader才會將OS從硬碟中拉到記憶體，控制權再交給OS。在剛開機時，IA32 CPU是在real mode，real mode最大只能存取1MB的記憶體，而protected mode最大能存取4GB的記憶體，這是因為兩種mode的Addressing方式不一樣。  

real mode addressing 是直接將logical address轉換成physical address，logical address表示方式為segment:offset，轉換過程其實很簡單，就把segment往左shift 4 bit然後加上offset就是physical address了。  
protected mode addressing感覺非常麻煩，他要先將logical address轉成linear address，然後再從linear address轉成physical address，這邊之後上課會詳細的介紹paging unit的memory addressing。

![Addressing in Protected Mode](/images/protected_mode.jpg)

<h5> Interrupts </h5>

Interrupt是指某樣事件發生後會去觸發某個動作，而處理相對應動作的稱做Interrupt Service Routine簡稱ISR，或者叫Interrup Handler。在real mode底下有一個Interrupt Vector Table(IVT)，裡面總共有256個entry，每一個emtry會指向ISR，如下圖：

![Interrupt Vector Table](/images/IVT.jpg)

而在protected mode下做法其實類似於real mode，它是將ISR存在Interrupt Descriptor Table(IDT)當中，而IDT的base address存在於Interrupt Descriptor Table Register(IDTR)，IDT裡存放的是8 byte的descriptor陣列，而descriptor又分為三種：task-gate descriptor、interrupt-gate descriptor、和 trap-gate descriptor。詳細可以看[這裡](http://www.csie.ntu.edu.tw/~wcchen/asm98/asm/proj/b85506061/chap4/idt.html)。

![Interrupts in Protected Mode](/images/interrupt_pro.jpg)

這邊注意一下CR0或MSW的PE flag是決定processor現在是real mode還是protected mode。

<h3> Endian Order </h3>

根據byte存放順序可分為Little Endian跟Big Endian，Little Endian會將排序較後面的byte放在較低的address，Big Endian會將排序較前面的byte放在較低的address。而Intel processors是little endian。舉個例子，有一個4byte的int為：Byte3, Byte2, Byte1, Byte0，則：

![Little Endian](/images/little_endian.jpg)

![Big Endian](/images/big_endian.jpg)


