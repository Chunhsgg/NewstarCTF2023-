## Re的开始

### 初始

​		逆向讲究的是**孰能生巧**，一定要多练，在练中积累调试经验。还有一定要有耐心，有时候题真的很坑，出题人自己写的加密算法，花里胡哨，心平气和。CTF虽然以web题目为主，但是可以尝试一下把re，pwn，安卓三种类型的题目分值相加，也是占非常大比例的，而且二进制方面的题目难度系数普遍偏高，想要高的名次，还是需要二进制来拉开差距的。Re，pwn，安卓三种关系相当于re是基础，pwn是进阶，安卓是衍生，变体，二进制师傅任重道远。

来源：https://zhuanlan.zhihu.com/p/336007893



首先是看看大二夏季学期上课时，学长讲解re时给的工具吧

![image-20240323185137109](E:\LSoft\typora\typora-user-images\image-20240323185137109.png)

这三个ExeinfoPe是查看给的程序的详细信息，比如是32位的还是64位的，加壳了没有。查壳工具

另一个就是IDA了，关于re的工具，我当时应该就下载了这么多

如果是64位程序，则用IDAx64打开，32位程序则使用IDA.exe打开？

关于IDA的使用，IDA是一款反编译软件，对于一个可执行程序，使用IDA打开则会以汇编语言显示程序？这样说不知道贵不规范

一些基本使用操作：

​		空格键/Tab			  查看方式切换(平铺or运行块)

​		F5							 反汇编，将汇编语言转换为类C语言

​		Shift+F12				字符串窗口

​		Ctrl+X					  交叉引用                            (可以知道哪个函数引用了这个字符串or数据)

​		G							  跳转地址							(我们可以复制地址在看其他位置时粘贴该地址进行跳转)

​		alt+T						搜索指令							(搜索地址或者指令......)

​		N							  重命名								(复杂的函数名字，为求方便，更改名称)

​		Ctrl+Z   					撤销操作

​		/								注释									(添加注释方便做题)

​		D							  将字符串转数据形式

​		A							   转化为字符串

​		C							  转化为汇编代码

​		U							  原始字节形式

​		shift+E					 导出数据

​		Ctrl+E					   起始位置

​		R								转化为字符型(各种形式转化都有功能)

​		\ 								隐藏   (看起来舒服些)

​		Shift+F7 					Segments窗口    各种段名

然后就什么也不知道了.......

从NewstarCTF 2023开始练习

### Week1 easy_RE

<img src="E:\LSoft\typora\typora-user-images\image-20240323190519165.png" alt="image-20240323190519165" style="zoom:50%;" />

题目写的很明白，打开就有。下载附件，拖到exeinfope发现是64位程序，然后用IDAx64打开

![image-20240323190628968](E:\LSoft\typora\typora-user-images\image-20240323190628968.png)



在main函数处看到前半部分flag{we1c0m

shift+F12(或者直接往后拖拖就看到了)

可以找到后半部分flag：![image-20240323190755445](E:\LSoft\typora\typora-user-images\image-20240323190755445.png)

所以得到flag{we1c0me_to_rev3rse!!}

仔细看反汇编结果的话，可以知道程序的逻辑是将你的输入记入v5

与v8对比，相等的话就输出Right！      而v8是由v10赋值，v10由v7和v6组合

v6就是后半部分flag     v7起初循环构造了11次，对应前半部分flag的11个字符，不过v7构造时只有在汇编语言下才能看到，反汇编看不出来怎么构造的，就知道循环了11次，应该是构造函数提前设计好的，所以在main函数这里看不到？



Re的题目基本上是一个可运行的程序，比如Windows中的exe，安卓系统的apk文件，还有可能是linux系统下的ELF文件，一般程序就是在运行的时候输入正确的flag，才能通过程序。 



### 解题的一般步骤

第一步 通过查壳工具(ExeinfoPe，DIE)查看文件的属性，它的属性决定对应的使用工具

第二部 打开对应的IDA找到主函数，或找到其他加密点

第三步 分析程序的加密逻辑

第四步 逆推逻辑写出脚本解出flag



### 一般的考点

算法

base64加密，存在base64加密 码表，(原来还有码表，看来理论知识要学习的)

 rc4

Tea

Aes



壳

Upx





### Week1 壳

<img src="../typora-user-images/image-20240324144623401.png" alt="image-20240324144623401" style="zoom:50%;" />

题目的意思应该是进行了加壳，而我们就需要去壳操作

首先将题目附件下载，先看看exe文件，还是传统输入flag的程序，接下来放入exeinfope检测

<img src="../typora-user-images/image-20240324144836869.png" alt="image-20240324144836869" style="zoom:50%;" />

可以看到是存在upx壳的，64位程序

如果没有脱壳的情况下，我们在IDA打开看看情况

![image-20240324144949491](../typora-user-images/image-20240324144949491.png)

可以看到函数只有这几个，显然是有壳存在的因故

我们也没有找到 please input flag关键字样，所以我们需要进行脱壳

手动去壳暂时没学，先试试工具去壳    FFL(没用)  我们使用upx.exe

![image-20240324145419764](../typora-user-images/image-20240324145419764.png)

可以看到脱壳成功，再次使用IDA打开分析

<img src="../typora-user-images/image-20240324145542662.png" alt="image-20240324145542662" style="zoom:50%;" />

现在就可以看到很多函数了

<img src="../typora-user-images/image-20240324150813555.png" alt="image-20240324150813555" style="zoom:50%;" />

找到主函数进行反汇编，我们可以看到程序是把我们的输入保存在Str1中，然后对Str1中每个字符做+1操作，最终和enc比较，相等则输出Wow，也就是flag正确，所以我们只需要查看enc字符串，然后将其每位对应-1就可以得到flag，算是一个凯撒密码吧。(这里涉及的不仅是字母)

gmbh|D1ohsbuv2bu21ot1oQb332ohUifG2stuQ[HBMBYZ2fwf2~

写个脚本还原回去就行

```python
secret="gmbh|D1ohsbuv2bu21ot1oQb332ohUifG2stuQ[HBMBYZ2fwf2~"
p=""
for i in range(len(secret)):
    p+=chr(ord(secret[i])-1)
print(p)
```

得到flag{C0ngratu1at10ns0nPa221ngTheF1rstPZGALAXY1eve1}









### Week1 Segments

<img src="../typora-user-images/image-20240324153218537.png" alt="image-20240324153218537" style="zoom:67%;" />

题目提示要打开IDA的Segments窗口，应该是shift+F7键

在IDA打开后，进入Segments窗口

<img src="../typora-user-images/image-20240324153748789.png" alt="image-20240324153748789" style="zoom:50%;" />

看到片段名可以构成flag{You_ar3_g0od_at_f1nding_ELF_segments_name}   (没有extern)

所以说Segments窗口里面的东西是什么，有什么用，值得我们关注什么。？

这里看一下官方题解：

​		本题主要是帮助新生同学理解如何开启IDA的Segment窗口。 本题有两种解法，首先是可以通过题目名称来解，将文件拖入IDA，然后按下shift+f7即 可看到flag被拆分放在各个段中，由于IDA的问题可能无法显示大括号，不过细心观察应 该是很容易能看出来flag的。

​		第二种解法可以使用python的lief库来读取，可以看到flag是从低到高的顺序保存的，通 过lief库可以读出相应的名字。

​		存在的疑问就是LOAD段和extern段是什么，看题解中将0x1000一下的地址和.dynamic段过滤后便得到了flag，所以是这两个段属于.dynamic段么？（并不是

​		LOAD 段是用于将可执行文件或共享库中的内容加载到内存中的段。每个 LOAD 段对应于文件中的一个区域，并且包含有关如何将这些数据加载到内存中的详细信息，例如偏移量、虚拟内存地址、段的大小等。

<img src="../typora-user-images/image-20240324155041285.png" alt="image-20240324155041285" style="zoom:50%;" />





### Week1 ELF

<img src="../typora-user-images/image-20240324155321835.png" alt="image-20240324155321835" style="zoom:50%;" />

确实不是Windows文件，可执行链接文件ELF，是Linux的文件吧。

但不妨碍使用IDA查看

![image-20240324155535625](../typora-user-images/image-20240324155535625.png)

​		直接来看main函数，可以看到这个程序也是要我们输入flag，我们输入的flag保存到s中，对s进行了一次加密v6=enconde(s),然后再进行一次s1=base64_encode(v6,v3)，

​		如果s1==VlxRV2t0II8kX2WPJ15fZ49nWFEnj3V8do8hYy9t 表示flag正确。

​		encode的加密逻辑，我们点击直接跳转到该函数

<img src="../typora-user-images/image-20240324155825307.png" alt="image-20240324155825307" style="zoom:50%;" />

​		可以看到对参数a1做的操作是，将a1的每一位与0x20异或再加上16，赋值给v4.最后一位补0，但已经和a1没有关系了。为什么要补0？一般都是添加’\0‘做结尾的吧。

​		我们再看base64_encode()的逻辑

<img src="../typora-user-images/image-20240324160334668.png" alt="image-20240324160334668" style="zoom:50%;" />

​		有这串码表就已经知道了，就是我们平常见到的base64加密，到时候直接在线解密就行。

所以现在对密文VlxRV2t0II8kX2WPJ15fZ49nWFEnj3V8do8hYy9t的操作就是：	

```
base64解密-->(每位-16)^0x20
```

先进行base64解密：V\QWkt $_e'^_ggXQ'u|v!c/m

```python
secret="V\QWkt $_e'^_ggXQ'u|v!c/m"
p=""
for i in range(len(secret)):
    p+=chr((ord(secret[i])-16)^0x20)
print(p)
```

得到flag{D04ou7nowwha7ELF1s?}     这正确答案差了几个_

在线base64解码一些不可打印字符串显示不出来？

所以还是需要我们全程写Python脚本

```python
import base64
secret="VlxRV2t0II8kX2WPJ15fZ49nWFEnj3V8do8hYy9t"
s=base64.b64decode(secret)
p=""
for i in range(len(s)):
    p+=chr((s[i]-16)^0x20)
print(p)
```

​		flag{D0_4ou_7now_wha7_ELF_1s?}

![image-20240324162201674](../typora-user-images/image-20240324162201674.png)

这边也可以看到，base64解密后确实是有一些不可打印字符的，所以有时候不能依赖在线网址。





### Week1 Endian

<img src="../typora-user-images/image-20240324162339465.png" alt="image-20240324162339465" style="zoom:67%;" />

放入IDA反汇编查看代码

![image-20240327144624438](../typora-user-images/image-20240327144624438.png)

​		这里要清楚_DWORD     这段代码中使用了`*(_DWORD *)v5`的语法来表示将`v5`强制转换为指向DWORD类型的指针，然后取出该指针指向的四个字节作为一个DWORD整数值。在这种情况下，每个DWORD整数值是一个32位的数据，由四个字节组成。

​		因此，通过取`v5`的四个字节，程序可以将用户输入的字符串按照32位的DWORD整数值进行处理，从而与预定义的DWORD数组`array`中的值进行比较。

​		在循环的每次迭代中，`v5`指针会按照指针的类型进行递增。在这里，`v5`是一个指向字符串的指针，而`_DWORD`在大多数情况下被认为是32位整数，因此递增指针会移动到下一个32位整数的位置，即下一个四个字节的位置。因此，`v5 += 4`将指针移动到下一个四字节的位置。

​		所以对于array数组来说，五个部分的内容分别与0x12345678异或，即可顺序得到flag的五个部分。

```python
import base64
array=["0x75553A1E","0x7B583A03","0x4D58220C","0x7B50383D","0x736B3819"]
expect=[hex(0x75553A1E^0x12345678)[2:],hex(0x7B583A03^0x12345678)[2:],hex(0x4D58220C^0x12345678)[2:],hex(0x7B50383D^0x12345678)[2:],hex(0x736B3819^0x12345678)[2:]]
a=[]

for i in range(5):
    for j in range(4):
        t=int(expect[i][2*(4-j)-2:2*(4-j)],16)
        a.append(t)
print(a)
flag=""
for i in range(len(a)):
    flag+=chr(a[i])
print(flag)
```

起初是t=int(expect[i] [2*j,2*j+2],16),但结果是galf......很显然是反了，所以这里需要是小端序，因此倒过来截取，最后可以得出flag。

'flag{llittl_Endian_a}'

这里给出官方题解的代码，更加简洁，需要学习

```python
data = [0x75553A1E, 0x7B583A03, 0x4D58220C, 0x7B50383D, 0x736B3819] 

from Crypto.Util.number import * 

flag = b'' 

for t in data: 

	flag += long_to_bytes(t^0x12345678)[::-1] 

print(flag)
```





### Week1 AndroXor

<img src="../typora-user-images/image-20240331134514487.png" alt="image-20240331134514487" style="zoom:50%;" />

关键词是异或

下载下来是一个apk文件。apk文件逆向之前没怎么碰到啊，直接看看题解。

题解没说用什么工具，但确定的是就是一个安卓异或。现在需要下载相应的工具。

一篇博客：[Android逆向CTF实用入门——apk](https://blog.csdn.net/weixin_63576152/article/details/132110224)

这里使用了jadx进行分析，在github上开源，有好多版本，我也不太懂，随便下载了一个版本。

程序的主要逻辑是在MainActivity中，如下：

<img src="../typora-user-images/image-20240331142025659.png" alt="image-20240331142025659" style="zoom:50%;" />

可以看到一个异或逻辑，我们写出对应的还原脚本即可

```python
cArr=[14, ord('\r'), 17, 23, 2, ord('K'), ord('I'), 7, ord(' '), 30, 20, ord('I'), ord('\n'), 2, ord('\f'), ord('>'), ord('('), ord('@'), 11, ord('\''), ord('K'), ord('Y'), 25, ord('A'), ord('\r')]
print(len(cArr))
kArr=['h','a','p','p','y','x','3']
flag=""
for i in range(len(cArr)):
    t = ord(kArr[i % len(kArr)]) ^ cArr[i]
    flag += chr(t)

print(flag)
```



​		期间遇到的问题，`cArr` 中既包含字符（如 `'K'`）又包含字符串（如 `'14'`）。在 Python 中，单引号或双引号括起来的内容都被视为字符串，即使其中只有一个字符。

​		`		ord` 函数用于获取字符的 ASCII 码值。如果参数不是字符，而是一个字符串，`ord` 函数将引发 `TypeError` 错误。为了确保 `ord` 函数正常工作，您需要先确保 `cArr[i]` 是一个字符。如果 `cArr[i]` 是一个字符串，您需要将其转换为字符，然后再应用 `ord` 函数。

​		我是这样写的脚本，这里再贴出来官方的：

```python
data=['\u000E', '\r', '\u0011', '\u0017', '\u0002', 'K', 'I', '7', ' ',
'\u001E', '\u0014', 'I', '\n', '\u0002', '\f', '>', '(', '@', '\u000B',
'\'', 'K', 'Y', '\u0019', 'A', '\r']

key=list('happyx3')
flag1=''
for i in range(len(data)):
    flag1+=chr(ord(data[i])^ord(key[i%len(key)]))
print(flag1)
```

​		显然官方的更好，因为能跑出来正确得答案，这里放一下对比

![image-20240331143740959](../typora-user-images/image-20240331143740959.png)

​		在第8个元素也就是下标为7进行异或得出的结果不一样这是我异或的，我得出o是因为数字7进行了异或，而官方这是把7当作字符进行异或就得到了_,,,,,,,有什么区别

​		应该是数字7和字符7有什么不同么，这里我看了看应该是数字7直接参与异或与ord('7')字符7参与异或结果不同，

再仔细看看代码中

![image-20240331144139557](../typora-user-images/image-20240331144139557.png)

​		7这里是有单引号的，而其他数字是没有的，所以这里7是作为字符的，而不是数字，所以在我的代码中需要把7作为ord('7'),即可获取正确结果。







### Week1 EzPE

<img src="../typora-user-images/image-20240331144515473.png" alt="image-20240331144515473" style="zoom:50%;" />

<img src="../typora-user-images/image-20240331144700840.png" alt="image-20240331144700840" style="zoom: 67%;" />

​		使用exeinfo打开后发现提示该文件不是exe可执行程序。

​		现在猜测是不是程序的头部被修改了，可以用winhex打开看一看

<img src="../typora-user-images/image-20240331144921273.png" alt="image-20240331144921273" style="zoom:50%;" />

​		这边就能够看到提示我们能否修好PE头。

​		简单改了一下幻数

<img src="../typora-user-images/image-20240331145817101.png" alt="image-20240331145817101" style="zoom: 67%;" />

​		发现还是不能运行，然后我就和之前做的题目exe文件进行对比，发现不仅幻数不一样，最后第61个字节也不一样，这里该文件是90，而之前的那个exe文件第61个字节是80，我就改了一下，这次两个exe文件的前64个字节是完全一样的，然后再次打开题目文件。发现成功运行！

<img src="../typora-user-images/image-20240331151153043.png" alt="image-20240331151153043" style="zoom:50%;" />

​		这里是有侥幸在的，修复PE头是需要仔细琢磨的。

​		这里应该主要是DOS MZ header，其总共64个字节

```
 	第一个成员占2个字节，它被用于表示一个MS-DOS兼容的文件类型，他的值是固定的----“4D5A”
	（注意：因为我们是在十六进制编辑器下写数据，所以所有的数据格式都是十六进制式的。但是我们在开发环境中默认都是十进制的，所以必须在数据前加 0x ,即：0x4D5A。而我为了方便，就直接写成“4D5A”，也就是直接输入到编辑器中的值，是十六进制，后面的都照此规定书写。）
```

    	第2个成员到第18个成员总共58个字节，是对DOS程序环境的初始化等操作，对于我们这个程序来说，没什么影响，我们通通用“00”来填充。（如果读者想对其进行详细了解，请查阅相关书籍。）注意：因为我们不可能把PE结构所有的东西都面面俱到，他十分的庞大。当然也没有必要都去记他，只需掌握关键的地方就可以了。以后我们都将把不影响程序执行的成员填充为零，这样做，一方面使程序看起来简洁，另一方面可以使您快速定位PE结构中要重点掌握的地方。
    	第19个成员非常重要，他占4个字节，用来表示“PE文件标志”在文件中的偏移，单位是byte。

​		在这一题，我们修改的就是两个地方，一个是幻数，还有一个就是PE文件标志的偏移，也就上面说的第19个成员，非常重要。

​		<img src="../typora-user-images/image-20240331151836045.png" alt="image-20240331151836045" style="zoom:67%;" />

​		现在即可正常分析程序，使用IDA64打开。

<img src="../typora-user-images/image-20240331164620613.png" alt="image-20240331164620613" style="zoom:67%;" />

​		程序的主函数部分可以看到主要还是进行了异或加密，加密后的结果与data比较。

​		我们可以找到data的值，然后进行逆向还原。

​		这里写了好久的脚本，不知道遇到什么问题了，首先按照题目的逻辑，还原data的话需要倒着还原，最后一位input是加密不了的，IDA也可以看到最后一位是}，之前一直犟着用len()-i-1进行逆向还原，现在我也改了，用标准的range(len-1,0,-1):   

​		但结果还是不对

​		可以看看这是官方的
![image-20240331164937488](../typora-user-images/image-20240331164937488.png)

​		这是我的

![image-20240331164956716](../typora-user-images/image-20240331164956716.png)

​		但结果并不对......

​		后续再仔细检查发现果然是data抄错了，少抄了一个16进制数，

​		运行结果如下：flag{Y0u_kn0w_what_1s_PE_File_F0rmat}

​		`end=""` 参数是用于指定 `print()` 函数在打印结束时要添加的字符串。默认情况下，`print()` 函数在打印完成后会添加一个换行符 `\n`，使得下一个 `print()` 输出的内容会从新的一行开始。但是通过指定 `end=""`，您可以指示 `print()` 函数在打印完成后不添加任何额外的字符，即输出不换行，而是继续在同一行输出后续的内容。





### Week1 lazy_activity

<img src="../typora-user-images/image-20240331165611584.png" alt="image-20240331165611584" style="zoom:50%;" />

​		又是一道apk题目，按照前面做的题目，主要还是先找mainactivity

![image-20240404205838150](../typora-user-images/image-20240404205838150.png)

​		但找到这里却并没发现什么东西。

​		目录里有一个flagactivity可以看一看，把里面的代码丢给GPT看一看

<img src="../typora-user-images/image-20240404210108803.png" alt="image-20240404210108803" style="zoom:50%;" />

大致猜出来这个程序是让我们点击10000次，然后会出现flag？

![image-20240404210819512](../typora-user-images/image-20240404210819512.png)

所以说弹出的消息是editTextTextPersonName2

使用全局搜索

![image-20240404211054238](../typora-user-images/image-20240404211054238.png)

记住资源要勾选上。

<img src="../typora-user-images/image-20240404211136012.png" alt="image-20240404211136012" style="zoom:50%;" />

GPT也说明了，代码是获取视图的，布局文件和代码文件不同，布局文件相应视图的ID，然后引用视图，所以搜索所有的该名称的资源，即可找到视图中的flag。

```
flag{Act1v1ty_!s_so00oo0o_Impor#an#}
```





至此，newstarctf的第一周逆向题过了一遍，re算是进行了一个小开始。



下面会依次做一下后续几周的。





## Week2 

### 第一题 PZthon

<img src="../typora-user-images/image-20240406192238746.png" alt="image-20240406192238746" style="zoom: 80%;" />

用IDA打开

![image-20240406192617914](../typora-user-images/image-20240406192617914.png)

主函数里是一些py的东西？

是要手动脱壳难道？看到上面说ollydbg了，但是not packed不会骗人

所以这里应该是python打包成的exe，题目也有提示。

<img src="../typora-user-images/image-20240406194033416.png" alt="image-20240406194033416" style="zoom:67%;" />

在web版工具进行了操作https://pyinstxtractor-web.netlify.app/

使用在线pyc反编译对得到的pyc操作，查看代码

![image-20240406194409239](../typora-user-images/image-20240406194409239.png)

可以看到就是简单的异或加密，写出解密脚本即可

```python
enc = [
    115, 121, 116, 114, 110, 76, 37, 96, 88, 116, 113, 112, 36, 97, 65, 125, 103, 37, 96, 114, 125,
    65, 39, 112, 70, 112, 118, 37, 123, 113, 69, 79, 82, 84, 89, 84, 77, 76, 36, 112, 99, 112, 36,
    65, 39, 116, 97, 36, 102, 86, 37, 37, 36, 104
]

dec = []
for byte in enc:
    dec_byte = byte ^ 21
    dec.append(dec_byte)

dec_str = bytearray(dec).decode()
print("Decrypted:", dec_str)
```

flag{Y0uMade1tThr0ughT2eSec0ndPZGALAXY1eve1T2at1sC001}

```
pyinstxtractor使用
python pyinstxtractor.py xx.exe
运行后生成xx.exe_extracted文件夹，进去后是各种文件，在里面找到 PZthon.pyc，该文件就是
python文件的字节码文件，而字节码我们不好分析，需要反编译成python语言来分析，使用pycdc
./pycdc 文件名.pyc
如果出现bad magic！
./pycdas.exe 文件名.pyc  得到字节码
然后交给GPT。

```









### 第二题 AndroGenshin

​		安装逆向

​		先使用jadx打开apk文件

![image-20240407142932690](../typora-user-images/image-20240407142932690.png)

​		程序的主要代码如图所示，可以看到是生成的窗口填写用户名和密码

​		然后用户名和base64_table作为参数传递给函数it_is_not_RC4,查看此函数发现就是RC4加密，其中username是作为密钥的，然后对base64_table进行加密。

​		加密结果和密码作为下一个函数的参数，发现这个函数是一个换表base64加密，然后加密结果和

​		"YnwgY2txbE8TRyQecyE1bE8DZWMkMiRgJW1="作比较，相等则会祝贺。

​		所以我们需要对此解密。

​		首先需要进行rc4加密获取后面的base64码表。

​		然后有此码表以及密文，我们再进行base64解密即可得到明文。

```python
rc4_key=b"genshinimpact"
base64_table=[125, 239, 101, 151, 77, 163, 163, 110, 58, 230, 186, 206, 84, 84, 189, 193, 30, 63, 104, 178, 130, 211, 164, 94, 75, 16, 32, 33, 193, 160, 120, 47, 30, 127, 157, 66, 163, 181, 177, 47, 0, 236, 106, 107, 144, 231,250,16, 36, 34, 91, 9, 188, 81, 5, 241, 235, 3, 54, 150, 40, 119, 202, 150]
def rc4(key,data):
    S=list(range(256))
    j=0
    out=[]
    for i in range(256):
        j=(j+S[i]+key[i%len(key)])%256
        S[i],S[j]=S[j],S[i]
    i=j=0
    for t in data:
        i=(i+1)%256
        j=(j+S[i])%256
        S[i],S[j]=S[j],S[i]
        out.append(t^S[(S[i]+S[j])%256])
    return out
import base64
def base64_custom_decode(data,custom_table):
        original_table="ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
        decode_table=str.maketrans(custom_table,original_table)
        decoded_data=base64.b64decode(data.translate(decode_table))
        return decoded_data
retval=rc4(rc4_key,base64_table)
new_table="".join([chr(t) for t in retval])
enc_data="YnwgY2txbE8TRyQecyE1bE8DZWMkMiRgJW1="
print(base64_custom_decode(enc_data,new_table))
```

运行得到flag{0h_RC4_w1th_Base64!!}



### 第三题 SMC

![image-20240407155803111](../typora-user-images/image-20240407155803111.png)

32位可执行程序，放到对应IDA查看

![image-20240407210955659](../typora-user-images/image-20240407210955659.png)

根据一些经验什么的，进行一些重命名，

这个程序对我们输入处理的逻辑主要是在403040函数这里，第二个参数应该就是密文了

那么前面的sub_401042是什么？

<img src="../typora-user-images/image-20240407211314846.png" alt="image-20240407211314846" style="zoom:50%;" />

可以看到是对403040函数进行异或了，对这个函数进行了解密

因为我们查看403040时看不懂是些什么。

题目SMC意思也是这个，对代码进行了加密，对抗逆向的一种手段，那么我们怎么知道这个函数到底什么意思呢？

还有就是加密了程序为什么还能正常运行呢？因为在运行这段代码之前，程序又进行了解密，而sub_401042就是解密函数。

我们需要知道代码解密后的状态来进行分析。

一个很方便方法是进行调试，还有方法就是对smc_data处理来得到解密后的代码

<img src="../typora-user-images/image-20240407212925222.png" alt="image-20240407212925222" style="zoom:67%;" />

在函数对smc_data处理时下断点

补：这里对两个窗口处理对应

![image-20240407213022805](../typora-user-images/image-20240407213022805.png)

可以右键

![image-20240407213105935](../typora-user-images/image-20240407213105935.png)

就可以跳转对应了

Synchronize with意思就是同步于

第二个要注意的点是该程序也有反调试的代码，我们需要进行绕过

![image-20240407213310265](../typora-user-images/image-20240407213310265.png)

绕过的方法主要是PEB标志位，这里是在汇编中的jz指令，更改zf标志位为1进行绕过(改变控制流)

在这里也需要下一个断点(jz指令处)

![image-20240407213843073](../typora-user-images/image-20240407213843073.png)

然后在local Windows debugger下进行调试运行

![image-20240407214141559](../typora-user-images/image-20240407214141559.png)

这里可以看到运行到断点处，左侧闪烁，说明要跳转到此处，即检测到debug调试

![image-20240407214443043](../typora-user-images/image-20240407214443043.png)

此处修改标志位ZF

F2下断点 F9继续运行

![image-20240407215205784](../typora-user-images/image-20240407215205784.png)

之后程序运行到此断点处，可以看到此时smc_data已经经过解密函数。

我们点击函数到达其位置处，我们按快捷键C转换为Code，然后快捷键P让IDA识别一下函数

识别出来是汇编语言，然后按tab键进行反编译，看代码

![image-20240407215828539](../typora-user-images/image-20240407215828539.png)

可以看到是将我们的输入与0x11异或然后+5

编写解密脚本

```python
enc = [0x7c,0x82,0x75,0x7b,0x6f,0x47,0x61,0x57,0x53,0x25,0x47,0x53,0x25,0x84,0x6a,0x27,0x68,0x27,0x67,0x6a,0x7d,0x84,0x7b,0x35,0x35,0x48,0x25,0x7b,0x7e,0x6a,0x33,0x71]
flag=""
for i in range(len(enc)):
    t=(enc[i]-5)^0x11
    flag+=chr(t)
print(flag)
```

得到flag{SMC_1S_1nt3r3sting!!R1ght?}





### 第四题 Petals

![image-20240408200905468](../typora-user-images/image-20240408200905468.png)

应该是linux中的程序，这里就直接拖到ida中分析了

查看main函数可以看到，程序将我们的输入保存在byte_4080

然后密码长度应该是25位

flag{md5(passwd)}   sub_160C是比较两者是否相等，但是loc_1209是对4080进行了怎么样的操作呢？

题目是花瓣，再结合分析应该是有关花指令的。

![image-20240408213155671](../typora-user-images/image-20240408213155671.png)

这里的jz和jnz指令会直接跳转到13B1地址处，我们先按U键进行undefine展开

![image-20240408213356374](../typora-user-images/image-20240408213356374.png)

我们可以看到指令执行时会跳过13B0，不执行这个字节，这个就是所谓的花指令，这个字节不会执行但是它会阻扰IDA分析整个函数，导致IDA对此处函数的识别出现问题，所以我们现在需要将此指令转为空指令。0x90

![image-20240408213810352](../typora-user-images/image-20240408213810352.png)

并且B1处C键转换为代码，然后在1209处P键重新识别一下函数，F5进行反汇编即可(有时候不需要操作IDA就会自动识别出函数了，只要你处理好了花指令了)

![image-20240408214002035](../typora-user-images/image-20240408214002035.png)

这段代码就是加密逻辑，首先是v5数组，进行的操作是循环i^a2取反，这里的a2就是我们输入的长度值是25.

得到这样的一个v5数组后，对我们的输入遍历，进行+v5操作，然后保存

最后和4020比较。

写出解密脚本

```python
enc = [0xD0, 0xD0 ,0x85 ,0x85 ,0x80 ,0x80 ,0xC5 ,0x8A ,0x93 ,0x89 ,0x92 ,0x8F ,0x87 ,0x88 ,0x9F ,0x8F,0xC5 ,0x84 ,0xD6 ,0xD1 ,0xD2 ,0x82 ,0xD3 ,0xDE, 0x87 ]
print(len(enc))
table=[]
for i in range(256):
    t=(~(i^25))&0xff
    table.append(t)
flag=""
for i in range(25):
    flag+=chr(table.index((enc[i])))
print(flag)
for i in range(25):
    print(chr(table[enc[i]]),end="")
```

不明白为什么返回的索引值，和密文作为索引结果是相同的

还有就是为什么异或0xff就可以，不异或就不行

flag{md5(66ccff#luotianyi#b074d58a)}

flag{d780c9b2d2aa9d40010a753bc15770de}

使用 & 0xff 做一个掩码操作； 0xff在二进制中是11111111，跟0xff进行与操作可以将其他位数清零，确保只剩下8bit大小的；











### 第五题 C?C++?

程序是Strange.exe

![image-20240424223735152](../typora-user-images/image-20240424223735152.png)

看不懂

![image-20240424223826125](../typora-user-images/image-20240424223826125.png)

反编译时提示不可反编译

考点：了解C#以及其反编译器dnSpy

所以我们需要下载dnSpy





### 第六题 R4ndom
