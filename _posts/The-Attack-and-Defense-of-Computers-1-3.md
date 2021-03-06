title: The Attack and Defense of Computers (1~3)
date: 2015-10-12 21:06:59
tags: The Attack and Defense of Computers
categories: The Attack and Defense of Computers
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
當你編譯C程式成為執行檔，它會先轉成組合語言再變成binary，而那些binary就放在紅色框框的code裡面。而你所寫的程式轉成組合語言後，基本上做的事情就是一直存取stack，所以stack上放的東西是你要執行時會用到的資料。當你開始執行程式時，code段裡面會有eip指著你現在執行到哪一行指令，stack段會有esp指著stack目前的頂端。下面我寫一個簡單的C程式：
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

在原始C檔的main function裡，w,x,y,z分別assign了1,2,3,4，對應到組語可以看到有四個記憶體位置被放置了1,2,3,4。上圖中可以看到，w跟x是連續記憶體，但y開始並沒有連續下去，代表y跟z被放到另一個區段。會區分這兩個區段是因為沒有初始值的變數c compiler並不會將它存在執行檔裡，不然當你宣告了很多變數沒有初始值，而這些又要存在執行檔的話，執行檔會變的超大。

接下來看stack frame裡面長什麼樣子，下圖中G function裡呼叫了H function，H function裡面可以讓使用者輸入字串。右邊的圖是在使用者輸入了abc之後產生的stack frame，最上面是G function的stack frame，接下來的b是G function push的參數要給H function用的，接著是return address，這邊的return address是給code段的eip用的，它裡面存的是G function裡呼叫H function之後下一行指令的位置，以這張圖來講就是add_g這個tag的位置，當H function執行完畢後eip才知道要回到哪行指令繼續執行下去。接下來就是ebp，ebp裡面存的位置是上一個呼叫此function的ebp，由於stack執行完後會有回收的動作，H function被回收掉時需要知道上一個ebp位置，而ebp暫存器又只有一個，所以就將它存在這個地方。ebp之後就是H function的local變數，首先是100個字元陣列c，這邊注意的是，雖然stack是由高位置往低位置長，但是local變數的低位置一樣是在低位置的地方，所以c[0]是在最下方，裡面存的是使用者輸入的a，以此類推a的上面是b，接著是c。最後是integer i的位置，目前這個位置是stack的頂端，所以esp會指著這個地方。

![stack frame](/images/sf.jpg)

了解stack frame長什麼樣子後，接下來看如何攻擊，當使用者不是像剛剛那樣乖乖的輸入合法字串，而是輸入超過指定範圍的字串，也就是超過c陣列定義的100byte，輸入的資訊會在stack裡面繼續往上長，也就是會蓋掉剛剛所講的ebp，接下來蓋掉return address，此時H function執行完畢後會執行return address裡面存的位置的指令，如果使用者是隨意輸入的話他會找不到位置，然後就segment fault，但如果攻擊者是輸入將要執行的攻擊碼位置，那執行完H function後就會跳到攻擊者要執行的程式了。

![stack smashing attack](/images/sf_attack.jpg)

上圖可以看到攻擊者先輸入一段Injected Code，之後在return address的地方輸入Injected Code的位置，攻擊便能執行。通常攻擊者會想辦法拿到shell，這樣之後才能做更多事情，所以Injected Code通常都是執行shell居多，而執行shell是以這支受害程式的權限去執行的，所以當此受害程式有root權限時，攻擊者就能夠利用此漏洞拿到root shell了。通常server上會掛一些程式，這些程式會提供使用者輸入，攻擊者會利用剛剛所講的漏洞執行shell並將它導到socket連回自己電腦，這樣就可以遠端拿到server的控制權。

但此攻擊會遇到一個問題，攻擊者輸入的資訊是在stack上的某個變數，要如何知道這個變數距離return address多遠？如果攻擊者有受害程式的原始碼也是沒用的，因為大多數compiler都不會按照原始碼宣告變數的順序來排stack上的順序，再來，如果攻擊者擁有原始執行檔也是沒用的，server上執行時的stack可能跟你local執行時的stack長的不一樣，所以該如何知道變數跟return address的offset呢？

![buffer offset](/images/buf_offset.jpg)

為了解決此問題，可以在快接近return address時，重複return address，雖然我不知道正確位置，但總有一個會蓋到真正的return address吧？再來我到底要從什麼地方開始注入我的攻擊碼呢？有一個東西叫做NOP，它的代碼是0x90，它做的事情是不做任何事情跳到下一行指令，這下好辦拉，我可以在我的攻擊碼前面加一堆NOP，這樣我的return address只要跳到這堆NOP，就可以一路滑到我的攻擊代碼了。因為之後防禦者知道有這種攻擊，在偵測病毒時會偵測有無0x90，攻擊者因而發展出其他的NOP sled，像是0x41、0x43，或者是mutiple byte的，像是0x0d0d0d0d，它做的事情是跟暫存器eax做or，對攻擊者來說也算沒做事情，加一堆0x0d後最後再補幾個NOP就能到達攻擊者的攻擊程式了。這邊有個[網站](https://www.onlinedisassembler.com/odaweb/)可以將十六進位碼轉成組合語言。
