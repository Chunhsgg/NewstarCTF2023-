# NewstarCTF 2023 Crypto

### 第五题 babyencoding

part 1 of flag: ZmxhZ3tkYXp6bGluZ19lbmNvZGluZyM0ZTBhZDQ=
part 2 of flag: MYYGGYJQHBSDCZJRMQYGMMJQMMYGGN3BMZSTIMRSMZSWCNY=
part 3 of flag: =8S4U,3DR8SDY,C`S-F5F-C(S,S<R-C`Q9F8S87T`

第一部分是base64编码

flag{dazzling_encoding#4e0ad4

第二部分是base32编码       [**30余种加密编码类型的密文特征分析（建议收藏）**](https://blog.51cto.com/u_15274949/4896287)

f0ca08d1e1d0f10c0c7afe422fea7

第三部分是uuencode加密

c55192c992036ef623372601ff3a}

所以flag{dazzling_encoding#4e0ad4f0ca08d1e1d0f10c0c7afe422fea7c55192c992036ef623372601ff3a}







## [RSA常用解密代码块（网上收集）](http://note.shenghuo2.top/01RSA%E5%9F%BA%E7%A1%80%E7%AF%87/P18.RSA%E5%B8%B8%E7%94%A8%E8%A7%A3%E5%AF%86%E4%BB%A3%E7%A0%81%E5%9D%97%EF%BC%88%E7%BD%91%E4%B8%8A%E6%94%B6%E9%9B%86%EF%BC%89/)

## RSA了解

密码学题目中经常会遇到RSA的题目

在RSA算法中用到了许多的参数，具体可以分为以下参数：

N：整数N，模数

p 和 q ：N的两个因子（素数）


$$
e 和 d：互为模反数的两个指数(互为逆元，mod\varphi(N))
$$


c 和 m：密文和明文，求解d时用到函数（d=gmpy2.invert(e,n的欧拉函数)）

![rsa1.PNG](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e27651f9fa52466c93fc9031dcb59818~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)



**加密算法**

![RSA2.PNG](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9980f94d48f643adb91925e30eb9bde6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

上面式子展示的就是将明文m（m<n是一个整数）加密成密文c，具体分析一下公式也就是密文等于**m的e次方对N求余**，可能涉及参数比较多有点混乱，我们重新整理一下，公式中C为密文，M为明文，而n,e组成公钥。

**解密算法**

![rsa3.PNG](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a5ea689977845cba4fd764866e995f2~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

可以发现与上面式子是类似的，只不过将E改成了D，RSA的明文是对代表密文的数字 c 的 d 次方对 N 求余的结果。仿照上面公钥的组成方式，我们便可以推断出私钥的组成方式。

图解

![rsa4.PNG](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9bf40c9b1e74073bd2dcd770b33ef73~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

后面还有一些，感觉非常好的一篇文章--来自稀土掘金    [你了解RSA加密算法吗？](https://juejin.cn/post/7150092791400366094)







### 第六题 babyrsa

![image-20240409225605891](../typora-user-images/image-20240409225605891.png)

题目提示说是很容易分解的n

所以我们可以直接尝试一下在线分解(懒得用yafu了)

我们需要分解大素数n为pq，然后得到欧拉函数，ed互为逆元的话就可以得到d了

然后m=pow(c,d,n)

<img src="../typora-user-images/image-20240409232744850.png" alt="image-20240409232744850" style="zoom:50%;" />

yafu会找到多个素数因子

<img src="../typora-user-images/image-20240409232816843.png" alt="image-20240409232816843" style="zoom: 67%;" />

网上找到的多因子应该也是可以算欧拉值的，但是我这里一通计算好像不太对，原来还是yafu没有分解完全C39还不是素数，需要继续分解(不熟悉yafu的使用)

<img src="../typora-user-images/image-20240409235304152.png" alt="image-20240409235304152" style="zoom:50%;" />

最后得到15个素数因子，然后计算出phi，然后解密即可

```python
from Crypto.Util.number import *

import gmpy2

n=17290066070594979571009663381214201320459569851358502368651245514213538229969915658064992558167323586895088933922835353804055772638980251328261

c=14322038433761655404678393568158537849783589481463521075694802654611048898878605144663750410655734675423328256213114422929994037240752995363595

e = 65537

p11 = 3831680819

p12=4278428893

p13=2794985117

p14= 4093178561

p15=2970591037

P1 = 2804303069

p2 = 2217990919

p3 = 3207148519

p4 = 3939901243

p5 = 2923072267

p6 = 3654864131

p7 = 2463878387

p8 = 2338725373

p9 = 2370292207

p10 = 2706073949



phi=(p15-1)*(p12-1)*(p13-1)*(p14-1)*(p11-1)*(P1-1)*(p2-1)*(p3-1)*(p4-1)*(p5-1)*(p6-1)*(p7-1)*(p8-1)*(p9-1)*(p10-1)



d=gmpy2.invert(e,phi)

flag=long_to_bytes(pow(c,d,n)).decode()

print(flag)
```

得到flag{us4_s1ge_t0_cal_phI}



可以在线分解http://www.factordb.com/

有关CTF中RSA题目的总结有许多前人了已经 [【中秋第二弹】手把手教你攻克CTF密码学！](https://juejin.cn/post/6844903683956670478)



官方题解是使用sage

![image-20240410134558566](../typora-user-images/image-20240410134558566.png)

省了点力

脚本中需要 from sage.all import *,但最后还是要在sage下吧，为什么用python导入？

![image-20240410135019610](../typora-user-images/image-20240410135019610.png)









### yafu的使用

输入 yafu-x64 进入命令行

最常用的命令是 factor (n)，将 n 值分解

使用 yafu 的时候遇到 mismatched parens

这是因为在命令行里不支持过长的位数，所以我们只要把 n 的值从文件中去读取即可。

新建一个文件 pcat.txt，内容里写上 n 的值(直接写)

注意：最后面一定要换行，不然会出现 eof; done processing batchfile

然后运行命令为

yafu-x64 “factor(@)” -batchfile pcat.txt

[来自yafu 使用方法](https://seamiloak.github.io/2020/10/16/yafu%E4%BD%BF%E7%94%A8%E6%96%B9%E6%B3%95/)

### Sage的使用

**SageMath 是在GPL协议下发布的开源数学软件，并且整合了许多已有的开源软件包到一个基于Python的统一界面下。其目标是创造一个Magma，Maple，Mathematica和Matlab的开源替代品。**

**SageMath 包含了从线性代数、微积分，到密码学、数值计算、组合数学、群论、图论、数论等各种初高等数学的计算功能。**

SageMath 的功能十分强大，现在很多CTF比赛的密码学题目都是能用Sage去求解的。建议大家花一部分时间去学习下。

如果觉得安装麻烦的话，可以使用在线的sage：https://sagecell.sagemath.org/

Windows版的话，官方给出的安装方法是通过WSL子系统的方式进行安装。

那我还是在kali上安装吧，kali上发现找不到安装包，于是就到pwn环境的虚拟机上(ubuntu)安装了





### 第七题 Small d

题目描述提到了维纳这个人，而且看加密代码我们也可以看到指数e很大而d很小，针对这种加密指数情况我们可以采取维纳攻击 https://github.com/pablocelayes/rsa-wiener-attack

[Wiener's Attack Ride(维纳攻击法驾驭) 证明](https://zhuanlan.zhihu.com/p/400818185)



直接使用解密脚本嗦了

但这里我遇到的问题是，我在终端运行脚本时会报错，说是n赋值存在语法错误，但是我使用vscode的python调试器运行脚本就可以正常获取结果......

flag{learn_some_continued_fraction_technique#dc16885c}

<img src="../typora-user-images/image-20240410235541648.png" alt="image-20240410235541648" style="zoom: 67%;" />

同样的脚本，刚才跑不通，现在又跑通了。离谱





### 第八题 babyxor

<img src="../typora-user-images/image-20240411130902886.png" alt="image-20240411130902886" style="zoom:80%;" />

刚看到被吓一跳，实际上仔细一想直接明文攻击即可，已知flag{肯定是开头的明文，直接和密文异或即可得到key

<img src="../typora-user-images/image-20240411132055780.png" alt="image-20240411132055780" style="zoom: 67%;" />

第一个十六进制字节e9是字母f异或密钥后的结果，所以运行结果中143就是key值

据此编写脚本解密

```python
key=143

cipher="e9e3eee8f4f7bffdd0bebad0fcf6e2e2bcfbfdf6d0eee1ebd0eabbf5f6aeaeaeaeaeaef2"

cipher1=bytes.fromhex(cipher)

for data in cipher1:

  print(chr(data^key),end="")
```

得到flag{x0r_15_symm3try_and_e4zy!!!!!!}

官方的题解

```python
from pwn import xor 

ciphertext= bytes.fromhex('e9e3eee8f4f7bffdd0bebad0fcf6e2e2bcfbfdf6d0eee1e bd0eabbf5f6aeaeaeaeaeaef2') 

for b in range(256): 

	if b'flag' in xor(bytes([b]), ciphertext):

		print(xor(bytes([b]),ciphertext))	
```







### 第九题 Affine

​		题目描述说Caesar with multiplication  凯撒带着乘法？而affine是放射变化的意思，可能是想说仿射变换和凯撒有些像，看了加密算法确实是，凯撒加密就是移位，而仿射变换就是在移位之前进行了乘法，所以就说是凯撒带着乘法。

在网上随便找了一篇博客比葫芦画瓢写了个爆破脚本，256*256还是很快的

https://blog.soreatu.com/posts/writeup-for-crypto-problems-in-cybrics-ctf-2019/

```python
# De=key[0]^-1(C-key[1])%256

a=17

b=23

c="dd4388ee428bdddd5865cc66aa5887ffcca966109c66edcca920667a88312064"

inv_a=inverse(a,256)

cipher=bytes.fromhex(c)

for data in cipher:

  plain=(inv_a*(data-b))%256

  print(chr(plain),end="")
```

得到flag{4ff1ne_c1pher_i5_very_3azy}





### 第十题 babyaes

AES加密显然的

def pad(data):

  return data + b"".join([b'\x00' for _ in range(0, 16 - len(data))])

根据填充函数可以猜测明文不会太长，应该是在16字节内，所以AES加密也就是进行了一轮而已

![image-20240411143609871](../typora-user-images/image-20240411143609871.png)

对key和iv进行了异或处理，但是key有256位，iv只有128位

key^iv ，那么只有 iv 与 key 的低128位相异或，所以 key 的高128位是固定不变的。所以 xor 的高128bits,就是 key 的高128bits,进而可以得到 key 的所有值256bits。

噢噢，key构造的时候是乘2构造的，所以有了前一半就有后一半了。看题不仔细了

```python
# AES

\# print(bytes_to_long(key) ^ bytes_to_long(iv) ^ 1)

from Crypto.Cipher import AES

from Crypto.Util.number import *

key_iv=3657491768215750635844958060963805125333761387746954618540958489914964573229

enc=b'>]\xc1\xe5\x82/\x02\x7ft\xf1B\x8d\n\xc1\x95i'

t=long_to_bytes(key_iv)

key=t[:16]*2

print(key)

iv=bytes_to_long(key[16:])^bytes_to_long(t[16:])

iv_byte=long_to_bytes(iv)

aes=AES.new(key,AES.MODE_CBC,iv_byte)

plain=aes.decrypt(enc)

print(plain)
```

得到flag{firsT_cry_Aes}





## Week2 

### 第一题 滴啤

题目说不分解也能求

根据提供代码，我们已知d%(p-1),n以及c，稍微搜索一看我们知道题目滴啤就是已知dp，dp泄露的一类题型

在网上找个脚本跑一下，记录一下

得到flag{cd5ff82d-989c-4fbf-9543-3f98ab567546}



### 第二题 不止一个pi

题目说都给我了,也是一种题型，简单推导

```
欧拉函数φ(n)的定义是小于n的自然数中与n互质的数的个数。

p^r 质因数就只有p，那么不与它互质的数都含有p，

和它不互质的数的个数：p^r/p=p^r-1/p=p^{r-1} 

剩下就是互质的数，其个数就是：p^r-p^{r-1} 

则有 φ(n)=p^r-p^{r-1}

对q同理
```

写出脚本运行得到

flag{bu_zhi_yige_p1dsaf}



### 第三题 halfcandecode

flag分成了两半，另一半不可解密，看加密代码可知

flag的前一半转换成大整数后又加了8位的随机数字作为m1然后RSA加密得到c1

相关的参数n和c1写入out.txt

**后一半的flag**

flag的后一半对每一个字符做hash写入out.txt，那显然是hash这一半不能解密了，可能需要暴力碰撞

那后面的这一半只需要一行一行去碰撞就行了，总会在数字字母和符号之间的一个hash值



```python
from Crypto.Util.number import *

import gmpy2

from hashlib import md5

\# 可显示字符编号范围是32-126，共95个字符

with open('E:\CTF\Task\halfcandeocde\out.txt') as f:

  for line in f:

​    if line.startswith('43054766'):

​      for line in f:

​        for i in range(32,126):

​          if (md5(chr(i).encode()).hexdigest())==line.strip():

​            print(chr(i),end="")
```

得到 cse_t0_fact0r}

**前一半的flag**

直接在线网站就可以发现n是可分解的，那就十分简单了

常规求解得到flag{two_cloab

所以最终flag{two_cloabcse_t0_fact0r}



### 第四题 Rotate Xor

旋转异或我闭着眼，flag我看不见

看题目代码，首先k1，k2是选取了两个64位的素数，然后将k1转换为字节形式与flag异或得到密文ciphertext

round_rotate_left函数主要是进行一个循环左移操作，对k1加密，最后打印出了密文，加密后的k1，原k2

而我们主要是要解密k1，需要根据加密过程逆向写出解密脚本

\

```python
# 先写逆向解密脚本

from pwn import xor

from Crypto.Util.number import *

round=12

ciphertext = b'\x8dSyy\xd2\xce\xe2\xd2\x98\x0fth\x9a\xc6\x8e\xbc\xde`zl\xc0\x85\xe0\xe4\xdfQlc'

enc_k1 = 7318833940520128665

k2 = 9982833494309156947



def round_rotate_right(num,step):

  return ((num>>step)|(num<<(64-step)))& 0xffffffffffffffff

def decrypt_key(enc_key):

  for _ in range(round):

​    enc_key=round_rotate_right(enc_key^k2,3)

  return enc_key

k1=decrypt_key(enc_k1)

flag=xor(ciphertext,long_to_bytes(k1))

print(flag) # \x对应的是UTF-8编码的数据
```

flag{z3_s0lv3r_15_bri11i4nt}



官方题解使用了Z3求解器

![image-20240414135357578](../typora-user-images/image-20240414135357578.png)

什么玩意儿





### 第五题 partial decrypt

解密，但是只解密一半

完全不知道什么意思

<img src="../typora-user-images/image-20240414140957025.png" alt="image-20240414140957025" style="zoom:67%;" />



![使用中国剩余定理对RSA算法进行解密](https://img-blog.csdnimg.cn/4a04ec90e5e44db6bf8e412bb5c93a27.png)

https://blog.csdn.net/qq_43589852/article/details/127691919

将题目给的代码补全了就好，真就只解密了一半

最后一步

```python
m=m2+h*q

print(long_to_bytes(m))
```

得到flag{rsa_with_crt#b12a3a020c9cc5f1a6df4618256f7c88c83fdd95aab1a2b2656d760475bd0bf1}





### 第六题 broadcast

我宣布个flag，但是加密的

和远程服务器交互，nc一下，可以得到的信息

n = 8139950558320960666395459970028100705795800751694224660318211578098359732077010548819381031461729760336640400102610331282850509234118385384603955831885521

c = 702541123719442947517620655896686738928249636633608096105155825955777175916762318315431080321558368811341901455465574704007064447948227332363855536193099

e = 17

e是非常小的

应该属于**低加密指数广播攻击**

​		如果选取的加密指数较低，并且使用了相同的加密指数给一个接受者的群发送相同的信息，那么可以进行广播攻击得到明文。

​		在CTF中，n、c不同，明文m，e相同，其e比较小。使用中国剩余定理求解

​		直接选择官方的脚本

​		得到flag{d0_n0t_sh0ut_loud1y_1n_th3_d4rk_f0r3st}











