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

<h2> Memory Addressing  </h2>

接下來要講linux kernel的初始化，而實體記憶體當中有些位置是被保留的，就算是kernel也不能放在那些保留的位置中，這些被保留的位置放置了BIOS以及一些I/O Port，如下圖所配置：

![](/images/memory_reserve.jpg)

在主機板上的BIOS程式所放置的實體記憶體位置為0xF0000到0xFFFFF，而一些其他的硬體設備像是graphics cards的BIOS放置的位置則是在0xc0000到0xc7fff之間。而一些device的IO share memory則是會放置在0xa0000到0xfffff之間，這些程式是在OS起來之前所需要的，也就是剛開機時所執行的程式，雖然OS起來後就不會用到BIOS，但是下一次開機還是會用到，因此這些程式無論如何都不能被動到，不然你下次都開不了電腦了....而這些code都在0xfffff之前，也就是佔用了實體記憶體中的前16MB，因此這16MB的空間都不能動到，linux kernel會從16MB後開始放置kernel的東西。

<h4> How Kernel Initializes Its Own page tables </h4>

kernel在初始page table會有兩個階段，在phase one時會初始化kernel code data segment，以及page table跟動態資料結構。kernel code+data總共有7MB，而在剛開機時是靠bootmen allocator來配置記憶體，bootmen allocator所佔的記憶體空間為1MB，所以phase one當中會有24MB的virtual address需要mapping：

![](/images/phase_one_mapping_size.jpg)

24MB的virtual address需要有6個entry來mapping到physical address，由於在paging機制開啟之前virtual address等於physical address，而在phase one當中會遭遇到從沒有paging到paging開啟這段過程，因此為了讓physical address一致，這邊會希望在phase one時mapping後的address會跟沒有paging時直接轉換的address相等。下圖為physical address layout：

![](/images/24MB_physical_address.jpg)

而在phase one中會呼叫到一些位置在0xc0000000之後的code，而我們希望mapping到physical時也會再前24MB的地方，因此我們的paging table有些特別，它會讓0x00000000與0xc0000000指到同一塊physical address：

![](/images/phase_one_map_method.jpg)

相當於在0xc0000000之後的virtual address只要減去0xc0000000就會變成physical address。那要如何做到呢？在linux kernel中Page Global Directory會存在initial_page_table，而第二層的Page Tables會存在 _brk_base，配合剛所講的24MB會需要6個entry，因此設定entry 0~5以及768 ~ 773以外的entry為0，而entry0跟entry 768所指向的_brk_base會是同一個entry，故相當於768 entry以後的address都會減掉0xc0000000的offset了：

![](/images/phase_one_page_table_layout.jpg)

接下來看整個initial page table過程，首先在paging機制還沒開啟前，virtual address等於physical address，linux kernel程式都會在0x00000000到0x00800000：

![](/images/phase_one_initial_pagetable1.jpg)

在這個startup_32()中會把paging叫起來，它將initial_page_table load到cr3暫存器當中，接著將cr0的PG flag enable，因此paging開始在處理器上運行，此時virtual address轉換到physical address時會經過paging unit：

![](/images/phase_one_initial_pagetable2.jpg)

而在initial過程中它乎叫了i386_start_kernel()程式，這是C程式，放置的位置是在0xc0000000之後，因此可以看到我們剛剛所設計的page table用意在此：

![](/images/phase_one_initial_pagetable3.jpg)
