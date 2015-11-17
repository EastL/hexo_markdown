title: Linux Operating System-6
date: 2015-11-01 16:57:31
tags: Linux Operating System
categories: Linux Operating System
---
來源：[Memory Addressing: Paging Unit (18 ~ 47)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-3.ppt)，[wiki:Paging](http://wiki.osdev.org/Paging)

上禮拜講到Paging Unit是利用兩層page table來完成address轉換，這邊接著講為何需要用到兩層table。原本兩層的話是利用linear address最前面10個bit來表示page directory的entry，中間的10個bit表示page table的entry，最後12bit則是physical address的offset，由此可知如果要將兩個page table簡化為一個table，那個前面20bit就是代表這一個table的entry，意思就是說在只有一個table情況下這table會有$2^{20}$個entry，相當於4MB，相較於兩層的page table是各佔4KB，所以用兩層table省了不少記憶體空間，再者雖然兩個table來轉換效率比一個table慢，但由於大多數的linear address都不會被用到，只有在process正要執行的address才會進行page table轉換，還沒有要用的linear address是不會被搬到記憶體上的。

<h2> Structures of Page Directories And Page Tables Entries </h2>
其實Page Directory跟Page Table的結構是一樣的，雖然細節部分有些許差異，但實際設計上的架構是把兩方會用到的參數都設定，如果是有一方有用到而另一方沒用到的參數就直接浪費掉那個欄位。這樣做的原因是：如果專門為Page Directory跟Page Table量身定做兩個Structure，那麼會浪費更多的記憶體。下圖為Page Directory Structure：

![](/images/page_dir.jpg)

G跟"0"這兩個欄位就是剛剛講的概念，這邊是Page Directory沒用到但Page Table會用到。

S：Page Size，是指Page Directory 的entry所指向的Page Table size，此欄位設定為0的話代表所指的Page Table大小為4KB，設定為1的話所指的Page Table大小為4MB。

A：Accessed flag，表示此entry所指向的page table有沒有被使用過，有的話設定1沒有設定0，此欄位是給OS用的，當發生記憶體不夠用時OS會去找一塊記憶體swap out，這時就要依據此欄位來看哪個page要先被踢掉。

D：Cache Disable，設定此bit則page frame將不會被[cache](http://enginechang.logdown.com/posts/249025-discussion-on-memory-cache)，否則就會。

W：這也是提供cache機制，如果設定為1表示write-through cache，設定為0表示write-back cache，有關write-through以及write-back可以參考[Computer Weekly](http://www.computerweekly.com/feature/Write-through-write-around-write-back-Cache-explained)。

U：User\Supervisor flag，顧名思義用來檢查權限用，雖然protected mode底下有ring 0 ~ 3，但這邊只有一個bit，因此如果此bit為1代表所有人都能存取，如果為0只有ring 0能存取。

R： 'Read/Write' permissions flag，讀寫權限，在segmentation有4個bit來表示讀寫以及執行權限，但這邊只有一個bit，因此如果此bit為1則能夠讀寫，為0則僅能讀取。

P：Present bit，表示所指向的page frame有沒有在記憶體裡面，有的話設定為1沒有設定為0，沒有在記憶體時則page fault。

接下來下圖為page table structure：

![](/images/page_table.jpg)

這邊有跟Page Directory重複的就不再贅述。

G：Global flag，此bit設定為1能避免被[TLB](http://wiki.osdev.org/TLB) flush，如果要用到此flag需要將cr4暫存器的Page Global Enable flag設定1。

![](/images/cr4_PGE.jpg)
D：Dirty flag，此flag與Accessed flag搭配使用，這邊是給OS的Enhanced Second-Chance Algorithm來決定要將哪些page做swap out，有關Enhanced Second-Chance可以參考[Page-Replacement Algorithms](https://www.cs.utah.edu/~mflatt/past-courses/cs5460/lecture10.pdf)。

下圖為兩個process在配置記憶體時的示意圖，可以看到linear addres轉換後的physical address是被打散在實體記憶體當中的，所以一個process所佔用的記憶體並不會全部都放到實體記憶體當中，只會有正在使用的記憶體才會被放入，而有了paging機制之後，每個process確保不會用到同一個實體記憶體區塊，而且如果要進行shared memory的話利用paging也方便許多。

![](/images/memory_vir_phy_layout.jpg)


<h2> Extended Paging </h2>
此機制是在Pentium CPU之後才有的，由於kernel space每次需要存取大量的記憶體，但paging unit一次只給4KB，因此Extended Paging將page frame增為4MB。如下圖所示，page directory大小不變，一樣是1024個entry，但之後就直接指向physical address了，每個page frame大小為4MB：

![](/images/extend_paging.jpg)

要使用Extended Paging Page Size 設定為1，代表每個entry所指向的page frame為4MB，cr4的Page Size Extension(PSE) bit也要設為1。

![](/images/cr4_pse.jpg)

在extend paging下linear address就只切成兩段，將原本paging unit的page table 10 bit與offset 12bit合併為offset 22bit，所以一個page frame會有$2^{22}$個offset相當於4MB，Directory一樣是10個bit。

<h2> The Physical Address Extension (PAE) </h2>
在Pentium Pro之後的CPU有36條address lines，因此可以有$2^{36}$的記憶體空間，也就是64GB，不過如果要使用到64GB的記憶體位置的話，需要將cr4的PAE bit開啟。

![](/images/cr4_pae.jpg)

那麼這邊衍生一個問題，CPU裡的所有暫存器都還是32位元，那要如何執行到64GB的RAM呢？因此有了新的paging機制。如果我們將64GB的記憶體以4KB為單位，那麼總共會有$2^{24}$個page frame，接著我們看PAE的機制：

![](/images/PAE.jpg)

與原本paging的差別多了一個Page Directory Pointer Table (PDPT)，PDPT總共有4個entry，每個entry有8個byte。由於原本的entry只有4byte，紀錄位址的地方只有20個bit，這邊要記錄的位址須要36-12=24個bit(page frame size = 4KB)，因此直接擴大兩倍每個entry為8byte。而後面的page directory與page table都是512個entry，每個entry一樣為8byte。由此可知在linear address的切割方式為：前兩個bit給PDPT，接下來的9個bit給page directory，再來9個給page table，最後的12bit為offset。

回到剛剛問題，那暫存器只有32個bit要如何存取到64GB的記憶體？由於我們這邊每個entry增為8byte，可以記錄的是整個64GB的記憶體，因此我就能夠存取64GB當中的任意位置，如果用的是以前的paging，那麼每次entry只能記錄20bit，因此就只能夠存取到最前面的4GB了。雖然entry紀錄位置不夠的問題解決了，但一開始的CR3要去指向PDPT要怎麼辦呢？由於CR3只有32bit，然後真正可以紀錄PDPT的base只有27bit，如下圖：

![](/images/cr3_register.jpg)

為了解決此問題，首先，PDPT就故定放在前4GB的位置，這時代表著PDPT的位址前4個bit一定為0；再來，將PDPT故定放在32的倍數，如此一來後面的5個bit又都是0，因此只需要記錄中間的36-4-5=27個bit就足夠了，之後拿出來前四個bit跟後五個bit都塞0就是PDPT的位置。
