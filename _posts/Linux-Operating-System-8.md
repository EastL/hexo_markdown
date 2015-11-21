title: Linux Operating System-8
date: 2015-11-21 12:35:38
tags: Linux Operating System
categories: Linux Operating System
---
來源：[Memory Addressing (22 ~ 54)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/linuxLecture_3_9-5-1.ppt)

<h2> Phase two </h2>

Phase one只會用到實體記憶體中的24MB，phase two則是會用到所有的記憶體空間來放置kernel code。phase two有兩種不同狀況：CONFIG_HIGHMEM與CONFIG_NOHIGHMEM，CONFIG_NOHIGHMEM是指kernel能存取的實體記憶體小於1GB，而CONFIG_HIGHMEM則是kernel能夠存取到大於1GB的實體記憶體。而CONFIG_HIGHMEM又能分為三種case，分別是

![](/images/configure_kernel_case.jpg)

接著我們以這三種case分別討論：
<h4> RAM Size Is Less Than 887MB </h4>

在mapping過程中setup_arch()會將swapper_pg_dir的位置寫入cr3([load_cr3(swapper_pg_dir)](http://lxr.cpsc.ucalgary.ca/lxr/linux+v3.9/arch/x86/kernel/setup.c#L870))，然後會對swapper_pg_dir做initial的動作，function流程如下：

![](/images/phase_two_function_call.jpg)

接著為實際mapping，從linear address的0xc0000000開始到887MB為止，透過page table mapping到整個實體記憶體：

![](/images/phase_two_less_887.jpg)

由於最前面的4MB所對應到的實體記憶體位置是從0x00000000開始，因此會有一些BIOS以及I/O share memory，因此這邊所給的是4KB的page table，才能避開那些不能用到的空間。而中間的880MB就直接是4MB的mapping了，因為kernel code通常要的記憶體空間都很大，所以當kernel要記憶體時一次給他4MB比較有效率。最後就剩下3MB，只能給4KB的mapping了。下圖為page table 示意圖：

![](/images/phase_two_less_pagetable.jpg)

這邊上課有人提到，既然實體記憶體上全部都是kernel space，那user space的資料要放哪？當kernel一起來的時後整個時體記憶體都歸kernel掌控，而在user space要去存取記憶體時會再經由page table得到一塊4kB記憶體，此時的實體記憶體當中會有兩個address，一個是來自kernel的4MB page table mapping過來的，另一個則是由user space經過4KB的page table對應過來。

<h4> RAM Size Is between 887MB and 4096MB </h4>

由於kernel space只有1GB，因此沒辦法mapping到所有的實體記憶體空間，故跟case 1一樣將kernel space的東西mapping到887MB的地方，而剩下多出來的實體記憶體空間則給user space使用。

![](/images/phase_two_between.jpg)

因為在kernel所對應的方式跟case 1一樣，因此code部分也就沒有變化。下圖為page table示意圖，其實跟case 1一樣只差了physical address多了紫色區塊：

![](/images/phase_two_between_pagetable.jpg)

<h4> RAM Size Is More Than 4096MB </h4>

這邊mapping方式比較特別，前面的0 ~ 443 entry各自對應到了2MB的實體記憶體，第443 entry透過4KB的page table對應到1MB的page frame，而最後的68個entry則是保留給noncontiguous memory allocation。

![](/images/phase_two_more.jpg)

最後要注意的是，經過initialized後kernel會將swapper_pg_dir複製到initial_page_table，然後再對initial_page_table做存取動作。

![](/images/clone_page_range.jpg)


