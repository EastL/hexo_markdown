title: The Attack and Defense of Computers (1~3)
date: 2015-10-12 21:06:59
tags: The Attack and Defense of Computers
---
來源：[Malware: Logic Bombs, Key Logger, URL Injection, Browser Hijackers, Trojan Horses, and Spyware](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/1_malware.ppt)，[Internet Worms, Buffer Overflow Attacks, and Heap Overflow Attacks (1~20)](http://www.csie.ncu.edu.tw/~hsufh/COURSES/FALL2015/2_BOA.ppt)

由於前兩次上課名詞解釋部分較多，這邊做筆記感覺也是把投影片再抄一次(汗...  
所以就從第三堂課開始~~(絕對不是偷懶XDD

<h2> Buffer Overflow Attacks </h2>
<h4> Stack Smashing attacks </h4>
Stack Smashing attacks 簡單來說就是你預先寫好一段程式碼，然後將它放到正在執行的程式的stack裡，再利用正常流程會執行到的function，將它的return address換成你剛剛放進的位置。聽起來好像有點複雜，這邊一步一步講解。

首先要先了解電腦中正在執行的程式在記憶體裡面是長什麼樣子。以下的例子環境是在linux 32bit的OS上：

![圖一](/images/code_memory.jpg)

上圖左半部是程式的原始C檔，右邊是process正在執行時記憶體內容(注意這邊記憶體位置不是實體位置)，由於此攻擊是針對stack，所以要先懂stack(紫色區域)跟code(紅色區域)這兩塊在幹嘛。  
當你編譯C程式成為執行檔，它會先轉成組合語言再變成binary，而那些binary就放在紅色框框的code裡面。而你所寫的程式轉成組合語言後，基本上做的事情就是一直存取stack，所以stack上放的東西是你要執行時會用到的資料。下面我寫一個簡單的C程式：
{% codeblock %}
#include <stdlib.h>

int w;
int x;
int y = 5;
int z = 6;

int addTwoValues(int first, int second)
{
    return first + second;
}

int main(void)
{
    w = 1;
    x = 2;
    y = 3;
    z = 4;
    z = addTwoValues(w, x);
}

{% endcodeblock %}
這裡宣告了四個global變數，然後main function裡面對這四個變數assign值後呼叫了addTwoValues function。接下來將此檔案編譯成執行檔用objdump -d來看：

![objdump file](/images/add_od.jpg)

我只擷取了main function，最左邊(紅色)那排是程式碼所在的位置，也就是圖一中code那個地方，中間那欄(藍色)就是code裡面放的內容，由於binary不易閱讀，objdump很好心的幫我們翻譯成組合語言，在右邊那欄(黃色)。這邊可以看到組合語言做的事情都是在記憶體位置以及暫存器之間搬動值，而所有的push、pop以及接下來call的function都在stack裡面。  
這邊額外補充一下老師所講的global變數儲存位置，在linux中C如果宣告了global變數，有初始值的會被放在圖一中的data段，沒有初始的會被放在bss段(BlockStarted by Symbol)，可以看一下剛剛的程式，w跟x沒有初始值，y跟z有初始值：

![](/images/cvsas.jpg)

上圖中可以看到，w跟x是連續記憶體，但y開始並沒有連續下去，代表y跟z被放到另一個區段。會區分這兩個區段是因為沒有初始值的變數c compiler並不會將它存在執行檔裡，不然當你宣告了很多變數沒有初始值，而這些又要存在執行檔的話，執行檔會變的超大。

