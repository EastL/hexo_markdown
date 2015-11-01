title: Linux Operating System-6
date: 2015-11-01 16:57:31
tags: Linux Operating System
---
來源：[Memory Addressing: Paging Unit (18 ~ 47)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-3.ppt)

上禮拜講到Paging Unit是利用兩層page table來完成address轉換，這邊接著講為何需要用到兩層table。原本兩層的話是利用linear address最前面10個bit來表示page directory的entry，中間的10個bit表示page table的entry，最後12bit則是physical address的offset，由此可知如果要將兩個page table簡化為一個table，那個前面20bit就是代表這一個table的entry，意思就是說在只有一個table情況下這table會有$2^{20}$個entry，相當於4MB，相較於兩層的page table是各佔4KB，所以用兩層table省了不少記憶體空間，再者雖然兩個table來轉換效率比一個table慢，但由於大多數的linear address都不會被用到，只有在process正要執行的address才會進行page table轉換，還沒有要用的linear address是不會被搬到記憶體上的。
