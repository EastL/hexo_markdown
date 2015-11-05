title: The Attack and Defense of Computers-6
date: 2015-11-05 09:30:33
tags: The Attack and Defense of Computers
---
來源：[Drive-by Download(83 ~ 106)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/2_BOA.ppt)，[Botnet(1 ~ 10)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/2_1_Botnet.ppt)

<h2> Heap Spray </h2>
Heap spray主要概念就是在受害程式的heap段注入大量的shellcode以及nop，這項技術是由SkyLined所提出，他利用heap spray打下了IE的許多漏洞，例如[MS04-040](http://www.microsoft.com/technet/security/Bulletin/MS04-040.mspx)以及[MS05-020](http://www.microsoft.com/technet/security/Bulletin/MS05-020.mspx)。下圖為Heap spray概念圖：

![](/images/heap_spray.jpg)

首先受害程式必須要有可以注入heap段的地方，我們所注入的程式必須在user space的heap段當中，圖中可以看到是在0x7fffffff以下，這邊是因為在windows環境，memory有一半是user space，而所注入的程式碼為nop+shellcode，如此一來再配合其他手段，將程式流程導向heap段變能發起攻擊。

實際要攻擊web的話可以利用Javascript的字串，就跟buffer overflow有點像，將javascript的string塞爆蓋到heap段為止。在Adobe Flash裡面也可以利用ActionScript來達到heap spray。下面為Javascript heap spray程式碼：

{% codeblock %}
<SCRIPT language="text/javascript">
  shellcode = unescape("%u4343%u4343%...");
  oneblock = unescape("%u0D0D%u0D0D");

  var fullblock = oneblock;
  while (fullblock.length<0x40000) {
      fullblock += fullblock;
  }

  sprayContainer = new Array();
  for (i=0; i<1000; i++) {
      sprayContainer[i] = fullblock + shellcode;
  }
</SCRIPT>
{% endcodeblock %}

上面可以看到注入的東西為shellcode以及NOP，整個注入的大小大概是262MB。因此ASLR在這邊對攻擊者而言影響不大。(參考[Nozzle: A Defense Against Heap-spraying Code Injection Attacks](http://research.microsoft.com/pubs/76528/tr-2008-176.pdf))     
而現在我們常用的影像檔案也可以拿來當作heap spray攻擊媒介，由於讀取影像檔的程式是動態讀取影像大小，因為影像檔大小無法預測，故我們可以依照讀取影像程式的寫法來塞heap spray到影像檔當中，這樣在受害者開起影像時就能將程式碼注入到開起影像的程式的heap段了。

<h2> Memory Corruption Exploit </h2>
講完heap spray後接下來看如何將程式流程導向heap段當中。第一種方法為Mishandling Tag Attribute Values，他是利用tag的屬性來讓memory crash。攻擊者可以自己加一段iframe，這個iframe大小為0因此受害者看不到，接著在裡面的屬性填上大量的字元，使得瀏覽器在執行抓取tag時造成memory corruption。範例如下：

![](/images/mishandling_tag_value.jpg)

圖中可以看到攻擊者在src以及name裡面塞了大量的字元，而塞完結果導致eip停在0x769f682f，代表我們塞的字串蓋到了return address，使得程式流程導向了非法的記憶體位置，因此我們便可利用這點加上一些denug工具，將程式流程導向我們剛剛所注入的heap段當中，變能完成攻擊。

<h4> Virtual Table </h4>
我們也能利用Virtual Table 來達成攻擊。Virtual Table是用來儲存一些function位置，在call一些lib時可以先到此table查詢function位置，然後再到function的真正位置，C裡面的GOT就是此類型的table。我們在overflow時可以將指向table的值蓋成我們自己的table，接著table裡面再去指向我們的heap spray段，如此便能達到攻擊者想要的流程：

![](/images/virtual_table.jpg)

上圖可以看到我們利用a[100]來蓋掉vptr，因此程式流程導向我們的table最後到達heap spray。

<h2> Drive-by Download Attacks  </h2>

