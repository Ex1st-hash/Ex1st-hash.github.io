## 扫描结果
target ip:10.10.10.134

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749112492613-c6c4d76f-b653-46c5-88e3-bea517c3a60e.png)

只有一个80端口，看一看TCP详细扫描

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749112608640-b3f25cda-3b29-43b9-a653-0032e8364caa.png)

nmap脚本扫描

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749112724661-166a9d25-1ff3-4e39-8d58-4909263e17a9.png)

扫出了robots.txt

## 初始权限
访问http服务，看到是一个默认的阿帕奇页面，查看源码发现无隐藏信息

直接访问扫出的robots.txt

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749112956093-086f6ff7-ba7d-4b67-990a-7db1acfbfb98.png)

得到一个字符串，推测它可能是一个目录，一个凭据，或者一种CMS名称，用户名等，由于是在robots.txt中发现的，是目录的可能性比较高，目录拼接后访问

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749113065660-02a11367-d413-4d08-8693-63553f22a9ee.png)

sar2html Ver 3.2.1，可能是一个cms的版本，在searchsploit或者网页上查找是否有相关漏洞

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749113171997-fffdc95e-681a-47f0-8bc0-d206b5817df1.png)

有远程代码执行漏洞，先拷下来看看

先看txt文件，似乎是先需要一个用户认证才能利用![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749113321174-c596e692-416d-47ee-8950-3510006fbb84.png)

再看看py,先直接运行看一下回显

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749113550582-5613dd9b-833f-461a-951f-ea7a4c9a3931.png)

不需要凭据似乎可以直接利用

查看它的源码，大致原理是

服务器会执行`plot=;[攻击者命令]`这种拼接后的命令

不妨手工验证一下，可以正确利用：

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749115148094-fe168017-0ed5-4dd2-89cc-6d4838ce12ed.png)



先尝试反弹shell，开启攻击机的1234端口监听

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749114238444-46262250-62a0-4f3c-aacc-c7912784cfd0.png)

这里开始反弹shell没有反弹成功，回想脚本原理，给payload加一个urlencode,反弹成功

## 提权
先优化一下tty

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749114357881-72f255ab-aa8b-4501-9fd2-825be4d8372b.png)

开始枚举

sudo -l需要密码，不可行

suid，查看内核，查看用户信息

suid乍一看没什么可以利用的，内核版本很高，应该不可行

/home/love目录没有看到可利用的隐藏文件

查看定时任务：![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749114620406-7fe506ee-7ab6-4e90-9ee7-e1e99af34603.png)

发现了一个root的sh脚本，每隔五分钟执行，查看其权限

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749114700280-5c44be74-1981-469d-b7f5-0e5a070799f2.png)

可读可执行，cat看一下

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749114752990-30db7871-a5a0-4731-ab95-b8c5fbe690b2.png)

发现该脚本执行一个名为write.sh的脚本

而且根据脚本的写法，判断应该是在同级目录下，进去看看

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749114839086-90dc7a86-ec5f-40de-9378-455588a6926e.png)

是一个权限很低的可写文件，所以这里的提权思路就是，通过在write.sh写入指定命令，让定时任务自动以root权限去执行，应该可以写一个反弹shell，时间一到，root执行反弹shell，攻击机监听成功反弹root shell 

这里让它以root身份打开一个shell应该是不行的，按AI的说法，因为**cron 是后台服务**，没有连接到任何终端（TTY），即使让它运行 `/bin/bash`，这个 bash 也无法弹出交互界面，因为它没有终端可以依附，所以直接反弹shell试试![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749116188444-fc7c8221-6fa6-4e27-9f26-7879aeea61c6.png)

等几分钟回来看看

提权成功

![](https://cdn.nlark.com/yuque/0/2025/png/48966044/1749116202728-0249613e-d402-462f-af38-425d41976532.png)

## 总结感想
这台靶机资产很少，不需要判断攻击链优先级权重，比较线性的打法，但是总体来说思路很经典标准，值得练习巩固



```plain
hexo init Ex1st
git config --global user.name "Ex1st-hash"
git config --global user.email "camoaurora@gmail.com"

git remote add origin git@github.com:ex1st-hash/ex1st-hash.github.io.git
```















































