---
title: 把系统同步进网盘，能否实现多电脑共用一个系统？
date: 2026-2-27
updated: 2026-6-19
img: images/pic/cover_Windowsononedrive.png
top: false
coverImg: images/pic/cover_Windowsononedrive.png
tags:
  -  探究实验
  -  系统整活
  -  Windows
  -  网盘
---

# 把系统同步进网盘，能否实现多电脑共用一个系统？

**Author:** GaltonLai

**Date:** 2026-06-19

**Link:** https://zhuanlan.zhihu.com/p/2051271877268714435

## **提前说明：此题材只是一个实验探究类题材，请勿轻易在自己电脑上面模仿。（除非你爱折腾）**

在这期实验中，我使用的是微软的OneDrive网盘。这是一台登录了微软账号（小号）的设备，并且我已经提前安装并配置好了OneDrive网盘。

![](https://pic2.zhimg.com/v2-5a3a98f2ed5c2502494c3d48d0017eff_1440w.jpg)

  

实验开始之前，我先测试一下OneDrive能不能正常同步上传。我这里准备了一个文本文件来测试。

  

![](https://picx.zhimg.com/v2-a2b0a6af63dcf9489413746a2a8389f3_1440w.jpg)

  

上传成功，网页上也可以成功读取文件。

  

![](https://pic2.zhimg.com/v2-14e537ea48a03300d169d665b2346423_1440w.jpg)

![](https://pica.zhimg.com/v2-72d8748d1f21d490dc91e9af9ff468a6_1440w.jpg)

  

  

所以我要做到系统装完之后，系统总空间不能超过5个G，所以现在我要找一些极度精简的系统（能开机就行了，管它能不能用呢）

![](https://pica.zhimg.com/v2-1dff14a44a864a68eeeee094fa478898_1440w.jpg)

  

好，实验开始。

### 方法一：把Windows文件夹直接安装到OneDrive同步文件夹，构造启动项再等待OneDrive同步

我在很早期的时候找到了239MB的[Win7](https://zhida.zhihu.com/search?content_id=277253933&content_type=Article&match_order=1&q=Win7&zhida_source=entity)镜像，现在我直接把系统文件解压到OneDrive的文件夹上（然后等它同步）。

![](https://pic3.zhimg.com/v2-0481e9ccf3db63626da5ed3d21de988e_1440w.jpg)

  

顺着目录找到OneDrive存放的文件夹。

![](https://pic4.zhimg.com/v2-7a7814ce8f6455975688ed48eb7f8f5b_1440w.jpg)

  

等待解压。

![](https://pica.zhimg.com/v2-8ac83ce75906dfa9d27a88950f8be000_1440w.jpg)

  

这里可以看到，系统文件都解压出来了。文件名旁边的状态表明了文件“正在同步”。

![](https://pic3.zhimg.com/v2-b59381b63f06cfb26c3a3900f391fd2a_1440w.jpg)

  

OneDrive网页端这边也看到了Win7的系统文件了。

![](https://picx.zhimg.com/v2-33bcfe8357cafae4a9f0002dc7ece8c7_1440w.jpg)

好，现在改注册表，我是通过[RegWorkshop](https://zhida.zhihu.com/search?content_id=277253933&content_type=Article&match_order=1&q=RegWorkshop&zhida_source=entity)这个软件挂载精简版Win7的注册表，分别挂载DEFAULT、SECURITY、SOFTWARE、SYSTEM这些注册表文件（目录在C:\\Windows\\System32\\config），然后构造系统路径，把原本键值C:\\Windows\\改成C:\\Users\\你的用户\\OneDrive\\Windows\\ （长难句部分）

![](https://pic2.zhimg.com/v2-53bda35ea8c4d0fce7f1942c49262c05_1440w.jpg)

![](https://pic4.zhimg.com/v2-05ba7434a94ade661d02d0f5183669e7_1440w.jpg)

![](https://pic3.zhimg.com/v2-8419fff91fa6f24cb605cc5adc4432c8_1440w.jpg)

这个软件只能查找1000条，所以要一直重复替换。

![](https://picx.zhimg.com/v2-e42415710e386c393969aa072fe00ae5_1440w.jpg)

![](https://pic4.zhimg.com/v2-5c31e43030054c1ab56fa99bab306e2b_1440w.jpg)

最后，添加Win7的引导，我用的是[bootice](https://zhida.zhihu.com/search?content_id=277253933&content_type=Article&match_order=1&q=bootice&zhida_source=entity)这个软件，编辑现有BCD引导文件，并添加一条启动项，这里要调整的是启动文件和启动路径。

![](https://pic4.zhimg.com/v2-5dd34da9921a030d23f126d595d5c997_1440w.jpg)

  

重启电脑看看效果。

![](https://pic1.zhimg.com/v2-7386febc637aaa13f7bd5e002b903eac_1440w.jpg)

  

效果不行，蓝屏了。

![](https://pic2.zhimg.com/v2-af3c9e9498a1ef0589ae00e2599534c1_1440w.jpg)

  

之后尝试了各个系统，都不行，所以只能换一种方式。

![](https://pic1.zhimg.com/v2-02ac9a85078d3a61bba04da09c2f739e_1440w.jpg)

  

### 方法二：新建一个[VHD](https://zhida.zhihu.com/search?content_id=277253933&content_type=Article&match_order=1&q=VHD&zhida_source=entity)，挂载成分区之后在分区里面装系统，OneDrive同步整个VHD文件

这个方法就是创建虚拟硬盘（VHD），然后把系统直接安装到VHD里面，再把VHD文件存放在OneDrive同步的文件夹里面。

因为免费用户的OneDrive存储才5个G，新用户“入门指南”那些文件又占了10几MB。所以我打算建4.5GB大小的VHD文件。

![](https://picx.zhimg.com/v2-e4f999dbe126a4715c14a2ed609cdffd_1440w.jpg)

  

按了“确定”之后，等待一会，这个VHD文件就建好了。

![](https://pica.zhimg.com/v2-efae681b8069b00ee6df3d5d0b246762_1440w.jpg)

  

点开VHD文件，现在VHD文件挂载的分区也出来了，现在可以在里面装系统了。

![](https://picx.zhimg.com/v2-2f5e10e1ebc081e7906bb796c2151dc7_1440w.jpg)

  

之后经过了几次失败的测试，我成功在VHD上安装了Win10的精简版…

![](https://pica.zhimg.com/v2-21216f55040fc43c35c879128b248acc_1440w.jpg)

  

照样新建一个引导项，设备类型选择VHD，设备文件也选择VHD文件。

![](https://pic1.zhimg.com/v2-2fb1278f12ab331ae38b3f5d3d8045ba_1440w.jpg)

重启，选择第二个系统查看效果。

![](https://picx.zhimg.com/v2-113168d3f511a2b3a6c12f77816db62f_1440w.jpg)

成功进入，但是没有字体。

![](https://pic2.zhimg.com/v2-4bb0ec3d72eaeb38f23691caa5de962f_1440w.jpg)

  

之后我回原系统，尝试去恢复字体。

![](https://pic4.zhimg.com/v2-16f2535dd150c2c25626a6c0dedf2f55_1440w.jpg)

  

现在字体是恢复了，但是我装完之后，开始菜单、搜索功能等UWP有关的组件打不开了。 （可能系统文件还是没处理好吧，我看这个镜像有人测评过不会有这个问题的啊）

![](https://pic1.zhimg.com/v2-cf2b41c5fa1a7d36938165efe2f41d06_1440w.jpg)

  

算了，我这里也不太打算去折腾这个系统。我现在回去看一下这个VHD系统 OneDrive 能否同步。

![](https://picx.zhimg.com/v2-ba57c76053cc429c49123d57624af7c3_1440w.jpg)

  

等了半小时，成功上传。

![](https://pic1.zhimg.com/v2-dda2f5bc5ac4c5e07e849550d946e3ac_1440w.jpg)

  

OneDrive网页端也看到了VHD文件。

![](https://pic4.zhimg.com/v2-2adfc6d27d3589cd10e88cb1f8e16f47_1440w.jpg)

  

突然想起我脚下有一台很旧、很低配置的实体机，已经吃灰没用了。 刚好这次可以拿来做实验。可以看到，我把那台实体机通过采集卡连接到我的电脑。

**（为了便于区分，那虚拟机就叫做A电脑吧，等会儿实验用的实体机叫做B电脑）**

  

![](https://pic2.zhimg.com/v2-e47f474dc09434ae0ea7747d97ea44b5_1440w.jpg)

  

电脑开机，**B电脑**装的是Win11。

![](https://pic3.zhimg.com/v2-c6290530b94a04dce302edabd6f742d6_1440w.jpg)

  

B电脑设置好OneDrive。

![](https://pic1.zhimg.com/v2-a0f1aba03a3ca6154a57afde38738976_1440w.jpg)

  

可以看到，文件同步过来了**（但是下载又等了半个小时）**。

![](https://pic2.zhimg.com/v2-2234a5055812585f6d663111b76df9bd_1440w.jpg)

  

下载完之后，添加引导，尝试查看效果。

![](https://pic1.zhimg.com/v2-4ceb575620111988950849e5cd6141de_1440w.jpg)

![](https://pic3.zhimg.com/v2-90ce8c8237ff79b9f231ce340e3527de_1440w.jpg)

确认一下引导，确保没有写错。

  

重启电脑

![](https://pic4.zhimg.com/v2-04da6b4cd281e624968423ccc813088b_1440w.jpg)

  

存在OneDrive的VHD里的Win10黑屏了， 我觉得是注册表盘符挂载这一块出现了问题，导致开不了机。

![](https://pic4.zhimg.com/v2-ea84bd4281db4d109f09dc9b3d671c8b_1440w.jpg)

  

之后我回到Win11，挂载VHD系统的注册表，把记录盘符的值全删了，让VHD里面的Win10重新记录。 **（系统记录盘符对应的注册表：HKLM\\SYSTEM\\MountedDevices）**

![](https://pic2.zhimg.com/v2-b2d234bbe529c2a34070b917853a2103_1440w.jpg)

  

重启之后，现在能进欢迎界面了，但还是报错了。

![](https://pic1.zhimg.com/v2-0b7932c98749d375c8b8feb3a2e90150_1440w.jpg)

  

结果卡在欢迎界面动不了了，我尝试三键（Ctrl+Alt+Del）打开任务管理器，尝试重启 xplorer（资源管理器），但是这个时候**资源管理器闪退了**。

![](https://pic4.zhimg.com/v2-f279cd3291adc38bd46ad885fbb4dead_1440w.jpg)

explorer.exe一启动就闪退

基本组件是可以用的，这个**残缺的系统**可以看作是一个没有GUI的系统了。

![](https://pic4.zhimg.com/v2-b915b6178b4b4eb065bee01578928fe3_1440w.jpg)

  

这个时候回去看虚拟机，发现OneDrive中的VHD文件“正在上传”。

（毕竟进入的是VHD文件里面的系统，系统肯定会对硬盘进行读写操作的，所以说VHD原本的文件肯定是有改动的嘛）

![](https://picx.zhimg.com/v2-acd7b2748fe70d90cd9ebc06e0f04d03_1440w.jpg)

  

### 那可以说明

假设那台虚拟机是A电脑，这台实体机是B电脑。如果A电脑进入VHD系统，那B电脑就会显示“正在上传”；而如果B电脑进入VHD系统，那A电脑就会显示“正在上传”。

但是这样很容易导致文件混乱（如果两边文件同步失败，可能会导致**直接丢引导，进不了系统**）

而且系统第一次进入时，会记录盘符的对应信息。但是如果VHD系统中已经记录过的注册表信息同步过来，不同硬盘的电脑可能对应不上，就容易导致**黑屏、蓝屏、卡死、重启**等问题。

### 可能会造成的误解

**Q：你电脑系统不是从网盘运行的，而是得通过上传下载，说白了还是在本地运行。**

A：你说得对。A电脑、B电脑的系统都是在本地运行的，只不过网盘只是负责同步系统的文件而已。

### 如果说系统是纯网盘运行，会是什么效果？

（我的网速上传2MB/s，下载5MB/s，应该算挺快的吧）

我用的是[CloudDrive2](https://zhida.zhihu.com/search?content_id=277253933&content_type=Article&match_order=1&q=CloudDrive2&zhida_source=entity)这个软件，挂载OneDrive网盘成网络盘符。

![](https://pic4.zhimg.com/v2-b587e0f141dd591d01f42d5918ece20b_1440w.jpg)

挂载OneDrive网盘

  

![](https://pica.zhimg.com/v2-ccdea7f1e9ca19c174acd9ecc64f0f3a_1440w.jpg)

添加挂载点

  

![](https://picx.zhimg.com/v2-67a35412de2ada33285f885457da63e5_1440w.jpg)

网络盘符

  

**_测试效果：_**

直接点开vmx文件打开虚拟机 43秒47

![](https://pic2.zhimg.com/v2-4fe862e1eab77027206731f108744101_1440w.jpg)

  

里面是个**硬盘占用2GB不到的Win8.1极度精简版**，开机需要等足足**35分钟**。

![](https://pic3.zhimg.com/v2-0a4cffcd34acd1c923399e2e58cbb1ba_1440w.jpg)

  

在网盘上运行的虚拟机可以发现，只要做任何操作，系统就会当场卡给你看。

## 总结

装系统的流程：**把系统装到本地里面的和网盘同步的文件夹（所以A电脑当时还是在本地运行系统）**→然后**A电脑负责把系统上传到网盘**→**B电脑登录网盘，等待系统文件同步**（说白了就是下载）→**B电脑添加引导，然后进入系统**。

但是这种做法操作风险极大，如果没有处理好同步，很容易导致文件混乱（如果两边文件同步失败，可能会导致**直接掉引导，进不了系统**）

就算同步成功，有可能文件或者注册表产生冲突，就容易导致**黑屏、蓝屏、卡死、重启**等问题。