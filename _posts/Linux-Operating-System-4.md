title: Linux Operating System-4
date: 2015-10-18 17:11:21
tags: Linux Operating System
---
來源：[Introduction and Memory Addressing: Segmentation Unit (21 ~ 49, 72 ~ 83)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-2.ppt)

<h2> Memory Addressing(x32) </h2>

在進入主題之前，先來複習一下第一堂課講過的三種address：  
<h6> Logical Addresses </h6>
主要是由segment base address加上offset組成，segment base address會在segment register。  
<h6> Linear Addresses </h6>
又稱做Virtual Address，主要是在logical轉換成實體位置之前的過度位址，有4G byte。  
<h6> Physical Address </h6>
最後的實體位置，對應著實際的memory cell，每個cell長度為一byte。

當compiler在配置記憶體給變數時，都是配置logical的offset給變數，然後CPU在執行每行指令時，會將logical address經過Segmentation Unit變成linear address，然後在經由Paging Unit變為最後的實體位置。

![](/images/address_trans.jpg)

上圖為Address Translation過程，這邊注意的是這些轉換過程都是寫在CPU電路上的，並不是我們寫程式去執行的。在memory中有可能會有兩個processor同時要去存取同一塊記憶體，因此有Memory Arbitrator機制來保證不會發生此狀況。  
接著開始進入Segmentation Unit，Segmentation Unit是在intel processor的protected mode底下addressing的機制，透過logical address 的 16bit segment selector以及32 bit 的 offset來計算出linear address。16 bit 的segment selector存在於segment register，IA-32 processor的segment register有六種，其中常用的為cs(code段用)、ds(data段用)，而segment selector的16個bit結構如下：

![](/images/segment_selector.jpg)

最右邊兩個bit是RPL(Request Privilege Level)，代表著執行權限，有分為kernel權限(00)以及user權限(11)。cs暫存器中的RPL稱做CPL(Current Privilege Level)，代表著目前執行的process所擁有的執行權限。從logical address轉到linear address時，processer會透過segment selector來找到該位址的segment base，再將offset加上這個base address便是完整的linear address，而這些segment會用一種方式將他儲存在Descriptor中，這些Descriptors又會放在table裡面，所以當processor要去找相對應的segment需要先找到table，然後再去找table裡面的某個欄位才會是processor要的segment descriptor。中間的TI(Table Indicator)就是用來辨認這一次要找的segment base要去哪個table找，而最前面剩下的13個bit就是描述table的第幾欄位(index)。

講了這麼多，到底什麼是Descriptor，而這些Descriptor又放在什麼table呢？
*   Segment Descriptor 

    為了方便管理記憶體，將記憶體切成一個區塊一個區塊，每個區塊叫做segment。而每個區塊的起始位置以及大小等資訊都會用 Descriptor 來記錄， Descriptor 大小為 8 byte：

    +   Base field (32): the linear address of the first byte of the segment.
    +   G granularity flag (1): 0 (byte); 1 (4K bytes).
    +   Limit field (20).
    +   S system flag (1): 0 (system segment); 1 (normal segment).
    +   Type field (4): segment type and its access rights.
    +   DPL (Descriptor privilege level) (2):
    +   Segment-present flag
    +   D/B flag
    +   Reserved bit
    +   AVL flag

    DPL是表示此segment的執行權限，CPU查到此segment時，會將這個Descriptor的DPL跟segment register的CPL做比對，來看目前是否有權限執行。而Segment Descriptor又分為以下幾種：

    +   Code Segment Descriptor
    +   Data Segment Descriptor
    +   Task State Segment Descriptor 
    +   Local Descriptor Table Descriptor 

    Task State Segment Descriptor(TSSD)是用在process切換時，儲存睡眠的proces暫存器狀態。下圖是Segment Descriptors的實際結構圖：

    ![](/images/segment_descriptor.jpg)

*   GDT v.s. LDT

    Segment Descriptors 會存在Global Descriptor Table (GDT) 或者Local Descriptor Table (LDT)這兩個table當中。在GDT裡面儲存的Descriptors是所有process都能夠使用的，所以一顆CPU只會有一個GDT；而LDT則是給各別的process使用，process無法存取別人的LDT。GDT的起始位置會存在gdtr暫存器裡，gdtr是32 bit的linear address以及16 bit的 table 大小：

    ![](/images/gdtr.jpg)

    而LDT的起始位置是存在ldtr當中：

    ![](/images/ldtr.jpg)

由於Segment Selector 的index只有13個bit，因此GDT最多只能有{% math %} 2^{13} {% endmath %}個entry，GDT的index最小從0開始。接下來看Intel processor如何將logical address轉換成linear address的：
 
![](/images/trans_address.jpg)

透過selector找到相對應的GDT或LDT後，根據index找到descriptor，再從descriptor裡面找到base address，然後跟offset相加就得到linear address啦！
