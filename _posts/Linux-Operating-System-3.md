title: Linux Operating System-3
date: 2015-10-16 12:19:29
tags: Linux Operating System
categories: Linux Operating System
---
來源：[Introduction (99 ~ 116)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-1.ppt)，[Introduction and Memory Addressing: Segmentation Unit (1 ~ 20)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-2.ppt)
<h2> Execution Mode of IA32 </h2>
在x86架構的processor分為real mode以及protected mode，而在protected mode底下又分為ring0、ring1、ring2、ring3，linux kernel只用到了ring0以及ring3，分別是kernel mode以及user mode，這兩種mode有著不同的權限，在上一堂課講到的address space分為kernel space跟user space，在kernel mode底下所有的address space都能夠執行，但是在user mode底下是無法執行kernel space的。在user space底下存放了以下這些東西：

*   user-level functions
*   variables
*   user-level data
*   library functions
*   the heap
*   the user-level stack

而kernel address space裡面有存放這些東西：

*   Kernel data
*   Kernel functions
*   each process’s kernel-level stack

既然在user mode底下無法執行kernel space的東西，那使用者在使用電腦時是如何做一些kernel才能做的事情呢？這邊有一個概念叫做mode switch，如果在user mode底下有需要用到kernel space東西時，會經由system call才能變為kernel mode，此時才有辦法存取"特定"kernel space的東西。這邊講的特定是：當你經由system call進到kernel mode前，你會先告訴system call你要做的事情，而當你通過system call進到kernel mode時，你就只能做你剛剛預告的事情，這邊先提個大概，在之後的課程會完整的說明流程。

而有些process是專門處理kernel的，他完全不會跟user溝通，只會run 在kernel address space裡，這種process叫做kernel threads，通常system剛啟動時或者系統被關閉時才會出現的程序。

<h4> Uniprocessors vs. Multiprocessing </h4>
不管是Uniprocessors或是Multiprocessing，一顆processor只能執行一個process，那我們的電腦是如何執行這麼多程式的？其實在CPU中會不會的切換process，才會讓使用者有同時執行很多程式的錯覺，那麼OS是如何切換程式呢？
*   Context Switch 
    也可以稱做process switch，當一個process要切換到另一個process時，必須先切換到kernel mode，然後在kernel mode底下將控制權轉移給另一個process。通常有下列四種事件發生時會進行context switch：

    *   System calls
    *   Exceptions
    *   Interrupts
    *   Kernel thread
    
    下圖為mode切換示意圖：
    ![](/images/mode_switch.jpg)

    Exceptions通常是process造成的，可能是page fault、除以零或者是有違法的address出現等，而Interrupts通常是硬體有事件發生，造成其他process先暫停，先處理interrupt，由interrupt handler處理。

*   Reentrant Kernels
    如果有兩個process同時要執行時，必須先暫停一個process，直到另一個process執行完畢。Interrupts可以利用此機制暫停目前正在執行的process，執行interrupt handler。

Interrupts與Exceptions其實蠻常發生的，process常常會被切換來切換去。
![](/images/interleaving.jpg)


