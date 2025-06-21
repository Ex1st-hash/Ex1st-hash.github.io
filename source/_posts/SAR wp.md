## 扫描结果

target ip:10.10.10.134

![image-20250621215905091](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621215905091.png)

只有一个80端口，看一看TCP详细扫描

![image-20250621215942825](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621215942825.png)

nmap脚本扫描

![image-20250621220007300](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220007300.png)

扫出了robots.txt

## 初始权限

访问http服务，看到是一个默认的阿帕奇页面，查看源码发现无隐藏信息

直接访问扫出的robots.txt

![image-20250621220024804](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220024804.png)

得到一个字符串，推测它可能是一个目录，一个凭据，或者一种CMS名称，用户名等，由于是在robots.txt中发现的，是目录的可能性比较高，目录拼接后访问

![image-20250621220345684](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220345684.png)

sar2html Ver 3.2.1，可能是一个cms的版本，在searchsploit或者网页上查找是否有相关漏洞

![image-20250621220411295](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220411295.png)

有远程代码执行漏洞，先拷下来看看

先看txt文件，似乎是先需要一个用户认证才能利用![image-20250621220432579](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220432579.png)

再看看py,先直接运行看一下回显

![image-20250621220459092](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220459092.png)

不需要凭据似乎可以直接利用

查看它的源码，大致原理是

服务器会执行`plot=;[攻击者命令]`这种拼接后的命令

不妨手工验证一下，可以正确利用：

<img src="C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220545049.png" alt="image-20250621220545049" style="zoom:33%;" />



先尝试反弹shell，开启攻击机的1234端口监听

![image-20250621220604714](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220604714.png)

这里开始反弹shell没有反弹成功，回想脚本原理，给payload加一个urlencode,反弹成功

## 提权

先优化一下tty

![image-20250621220621127](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220621127.png)

开始枚举

sudo -l需要密码，不可行

suid，查看内核，查看用户信息

suid乍一看没什么可以利用的，内核版本很高，应该不可行

/home/love目录没有看到可利用的隐藏文件

查看定时任务：![image-20250621220700843](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220700843.png)

发现了一个root的sh脚本，每隔五分钟执行，查看其权限

![image-20250621220804869](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220804869.png)

可读可执行，cat看一下

![image-20250621220818531](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220818531.png)

发现该脚本执行一个名为write.sh的脚本

而且根据脚本的写法，判断应该是在同级目录下，进去看看

![image-20250621220837687](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220837687.png)

是一个权限很低的可写文件，所以这里的提权思路就是，通过在write.sh写入指定命令，让定时任务自动以root权限去执行，应该可以写一个反弹shell，时间一到，root执行反弹shell，攻击机监听成功反弹root shell 

这里让它以root身份打开一个shell应该是不行的，按AI的说法，因为**cron 是后台服务**，没有连接到任何终端（TTY），即使让它运行 `/bin/bash`，这个 bash 也无法弹出交互界面，因为它没有终端可以依附，所以直接反弹shell试试![image-20250621220856111](C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220856111.png)

等几分钟回来看看

提权成功

<img src="C:\Users\YJY\AppData\Roaming\Typora\typora-user-images\image-20250621220927964.png" alt="image-20250621220927964" style="zoom:67%;" />

## 总结感想

这台靶机资产很少，不需要判断攻击链优先级权重，比较线性的打法，但是总体来说思路很经典标准，值得练习巩固