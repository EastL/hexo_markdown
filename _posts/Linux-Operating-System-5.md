title: Linux Operating System-5
date: 2015-10-26 21:55:23
tags: Linux Operating System
categories: Linux Operating System
---
來源：[Introduction and Memory Addressing: Segmentation Unit (80 ~ 100)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-2.ppt)，[Memory Addressing: Paging Unit (1 ~ 18)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-3.ppt)

由於現在的電腦都是多核心架構，一台電腦在run時不會只有一顆CPU，而接下來要講的GDT這個table每一顆CPU都會有一個GDT，因此在C語言有一種特殊的宣告方式：[per-CPU](http://www.gelato.unsw.edu.au/archives/linux-ia64/0604/18078.html#start)，他會將你宣告的值存在陣列當中，而這個陣列的個數就是你CPU的個數。

![](/images/per_CPU_concept.jpg)
<center> [per CPU](http://thinkiii.blogspot.tw/2014/05/a-brief-introduction-to-per-cpu.html) </center>

而實際的[GDT宣告](http://lxr.cpsc.ucalgary.ca/lxr/linux+v3.9/arch/x86/kernel/cpu/common.c#L91)如下：

{% codeblock %}
DEFINE_PER_CPU_PAGE_ALIGNED(struct gdt_page, gdt_page) = { .gdt = {
#ifdef CONFIG_X86_64
        /*
         * We need valid kernel segments for data and code in long mode too
         * IRET will check the segment types  kkeil 2000/10/28
         * Also sysret mandates a special GDT layout
         *
         * TLS descriptors are currently at a different place compared to i386.
         * Hopefully nobody expects them at a fixed place (Wine?)
         */
        [GDT_ENTRY_KERNEL32_CS]         = GDT_ENTRY_INIT(0xc09b, 0, 0xfffff),
        [GDT_ENTRY_KERNEL_CS]           = GDT_ENTRY_INIT(0xa09b, 0, 0xfffff),
        [GDT_ENTRY_KERNEL_DS]           = GDT_ENTRY_INIT(0xc093, 0, 0xfffff),
        [GDT_ENTRY_DEFAULT_USER32_CS]   = GDT_ENTRY_INIT(0xc0fb, 0, 0xfffff),
        [GDT_ENTRY_DEFAULT_USER_DS]     = GDT_ENTRY_INIT(0xc0f3, 0, 0xfffff),
        [GDT_ENTRY_DEFAULT_USER_CS]     = GDT_ENTRY_INIT(0xa0fb, 0, 0xfffff),
#else
        [GDT_ENTRY_KERNEL_CS]           = GDT_ENTRY_INIT(0xc09a, 0, 0xfffff),
        [GDT_ENTRY_KERNEL_DS]           = GDT_ENTRY_INIT(0xc092, 0, 0xfffff),
        [GDT_ENTRY_DEFAULT_USER_CS]     = GDT_ENTRY_INIT(0xc0fa, 0, 0xfffff),
        [GDT_ENTRY_DEFAULT_USER_DS]     = GDT_ENTRY_INIT(0xc0f2, 0, 0xfffff),
        /*
         * Segments used for calling PnP BIOS have byte granularity.
         * They code segments and data segments have fixed 64k limits,
         * the transfer segment sizes are set at run time.
         */
        /* 32-bit code */
        [GDT_ENTRY_PNPBIOS_CS32]        = GDT_ENTRY_INIT(0x409a, 0, 0xffff),
        /* 16-bit code */
        [GDT_ENTRY_PNPBIOS_CS16]        = GDT_ENTRY_INIT(0x009a, 0, 0xffff),
        /* 16-bit data */
        [GDT_ENTRY_PNPBIOS_DS]          = GDT_ENTRY_INIT(0x0092, 0, 0xffff),
        /* 16-bit data */
        [GDT_ENTRY_PNPBIOS_TS1]         = GDT_ENTRY_INIT(0x0092, 0, 0),
        /* 16-bit data */
        [GDT_ENTRY_PNPBIOS_TS2]         = GDT_ENTRY_INIT(0x0092, 0, 0),
        /*
         * The APM segments have byte granularity and their bases
         * are set at run time.  All have 64k limits.
         */
        /* 32-bit code */
        [GDT_ENTRY_APMBIOS_BASE]        = GDT_ENTRY_INIT(0x409a, 0, 0xffff),
        /* 16-bit code */
        [GDT_ENTRY_APMBIOS_BASE+1]      = GDT_ENTRY_INIT(0x009a, 0, 0xffff),
        /* data */
        [GDT_ENTRY_APMBIOS_BASE+2]      = GDT_ENTRY_INIT(0x4092, 0, 0xffff),

        [GDT_ENTRY_ESPFIX_SS]           = GDT_ENTRY_INIT(0xc092, 0, 0xfffff),
        [GDT_ENTRY_PERCPU]              = GDT_ENTRY_INIT(0xc092, 0, 0xfffff),
        GDT_STACK_CANARY_INIT
#endif
} };
{% endcodeblock %}

可以看到每個entry都是人工刻上去的...在linux上GDT總共只有32個entry，在這32個當中有11個是沒在用的，如下圖：

![](/images/linux_GDT.jpg)

接下來講GDT的每個entry，GDT的entry就是8 byte的descriptor，在這八個byte裡存的資料是segment的起始位置、大小，細到dpl這種只有一個bit的資訊。在舊版的linux kernel中，descriptor的宣告就只有很簡單的兩個int，但目前的版本有將每個欄位化分出來，每個欄位都有移個變數可以控制，因此為了調和這兩種新舊的宣告方式，在現在的linux kernel使用union，讓兩種宣告方式都行的通。

*   union
    union的member所佔的記憶體空間是相同的，這邊做個小實驗：

    {% codeblock %}
    #include <stdio.h>

    union data
    {
        int a;
        int b;
    };

    void main(void)
    {
        union data test;

        test.a = 10;
        printf("a = %d, b = %d\n", test.a, test.b);

        test.b = 99;
        printf("a = %d, b = %d\n", test.a, test.b);
    }

    {% endcodeblock %}
 
    這邊我宣告了union裡面有a,b兩個成員，當我assign值給a時看一下a跟b目前的值： 
  
    ![](/images/union_test.jpg)
   
    上面為執行結果，從結果可以看出不管我assign給哪個member，令一個member都會輸出一樣的值，在union底下所有的member是共用記憶體區塊的。因此我們可以利用union來建構descriptor的兩種不一樣宣告方式，在union底下宣告兩種struct：([原始碼](http://lxr.cpsc.ucalgary.ca/lxr/linux+v3.9/arch/x86/include/asm/desc_defs.h#L22))

    {% codeblock %}
    struct desc_struct {
        union {
            struct {
                unsigned int a;
                unsigned int b;
            };
            struct {
                u16 limit0;
                u16 base0;
                unsigned base1: 8, type: 4, s: 1, dpl: 2, p: 1;
                unsigned limit: 4, avl: 1, l: 1, d: 1, g: 1, base2: 8;
            };
        };
     __attribute__((packed));
    {% endcodeblock %}

<h3> Task State Segment </h3>
Intel x86有提供Task State Segment(TSS)，當process要切換到另一個process時，必須把目前的暫存器的值存起來，這時可以將這些值存在Task State Segment。而在linux kernel每個processor只會有一個TSS，要呼叫時用TR selector選擇該processor的GDT裡的TSS descriptor，再從descriptor裡面的資訊找到segment：

![](/images/tss.jpg)

TSS架構：

![](/images/tss_struct.jpg)

接下來介紹兩種在linux kernel會看到但不常見的語法：
*   typeof Operator 
    通常typeof是來看被宣告的變數是什麼型態，由於linux kernel的code非常長，要用某個變數時時常不曉得他在哪邊被宣告的，於是我們就直接用typeof來看遍數為什麼型態，甚至能用來宣告變數。
    {% codeblock %}
    int e;
    __typeof__(e + 1) j;  /* the same as declaring int j; */ 
    {% endcodeblock %}
    上面的j被宣告為與e相同型態。

*   Comma Expressions
    在C語言裡所有的敘述都會有回傳值，Comma Expressions會由左邊先計算，回傳值為最右邊。考慮下列程式：
    {% codeblock %}
    r = (a, b, ..., c);
    {% endcodeblock %}
    則計算過程為:a;b;...;r = c

<h2> Paging Unit </h2>
之前講到x86的位址轉換，會先由logical address經由segment unit轉為linear address，之後再轉成memory可以用的physical address，從linear address轉為physical address過程就是paging unit。下圖為page unit示意圖：

![圖一](/images/page_unit.jpg)

linear address會被切成3段，結構如下：

![](/images/linearaddress_struct.jpg)

最左邊10個bit是查directory用，中間的10個bit是查page table用，最後的12bit則是offset。page的概念是一堆linear address的集合，每一個page大小為固定；page table裡面每一個entry就對應到一個page；page directory裡面的每一個entry就對應到一個page table。page table(這邊指的是圖一中page table以及page directory，老師投影片中講的小寫page table就是在這邊用到，因為常常把table跟directory統稱為page table)固定有1024個entry，每個entry為4 byte，所以一個page table大小為4KB。

在電腦剛開機時，processor進入real mode，此時paging的機制沒有開啟，在進到protected時，必須把cr0暫存器的PG flag設定為1才會開啟page unit。

![](/images/page_cr0.jpg)

當PG=0時，processor會直接把linear address當做physical address送出去。
