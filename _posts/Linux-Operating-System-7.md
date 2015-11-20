title: Linux Operating System-7
date: 2015-11-15 12:02:37
tags: Linux Operating System
categories: Linux Operating System
---
來源：[Paging in Linux(23 ~ 32, 40, 52 ~ 55)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-4.ppt)，[Memory Addressing (1 ~ 21)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-5-1.ppt)

<h2> Paging in Linux </h2>

由於x86在32bit下有兩層的page table(PAE沒開情況下)，而在64bit下有更多層，因此linux為了要在兩種處理器底下都能跑選擇了較高層的paging level。在2.6.10版本linux paging level有三層，從2.6.11版本之後增加到了四層page table。

如果四層都會用到的話，依照virtual address轉換到physical address的順序依序為：Page Global Directory、Page Upper Directory、Page Middle Directory、Page Table。如下圖所示：

![](/images/linux_paging.jpg)

會用到這四層主要是x86_64的處理器需要用到，那對於32bit PAE沒有開啟情況以及Pentium這些只有兩層就夠的該如何處理呢？Linux這邊直接將中間兩層的大小設為0，也就是Page Upper Directory跟Page Middle Directory這兩層table消失，讓Page Global Directory直接指向第四層的page table，如下圖：

![](/images/linux_paging_twolevel.jpg)

而在32bit下PAE開啟的話有三層page table，此時對應到的linux四層table如下：

![](/images/linux_paging_PAE.jpg)

每個process都有屬於他的Page Global Directory以及page tables，當process switch發生時會將執行到一半的process的cr3暫存器的值存到descriptor中，然後再從GDT中找出將要執行的process的descriptor存到cr3，如此便能正確的指到page table。




