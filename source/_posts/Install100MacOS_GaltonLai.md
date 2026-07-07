---
title:  一台电脑，我又装了100个MacOS系统！
date: 2021-7-13
updated: 2026-7-2
img: images/pic/cover_Install100MacOS.png
top: false
coverImg: images/pic/cover_Install100MacOS.png
tags:
  -  实验探究
  -  多系统
---

# 一台电脑，我又装了100个MacOS系统！

**Author:** GaltonLai

**Date:** 2026-07-02

**Link:** https://zhuanlan.zhihu.com/p/2054931932241212080

**_（此实验完成于2021.7.13）_**

这个想法其实源于我之前的一个视频——我曾把100个Windows系统装进一台电脑，并成功运行。那次实验反响不错，于是我就在想：既然Windows能做到，那macOS呢？毕竟macOS对硬件兼容性要求更高，分区、引导、克隆都更复杂。但挑战越大，乐趣也越大，所以我决定试试看。

首先，我需要一个足够大的存储空间。我使用了一块1.31TB的虚拟硬盘（[VMware SATA](https://zhida.zhihu.com/search?content_id=277918975&content_type=Article&match_order=1&q=VMware+SATA&zhida_source=entity)），并提前创建了100个独立的10GB分区。这里有个关键点：macOS自带的磁盘工具最多只能创建16个分区。

![](https://pic3.zhimg.com/v2-aec9e2968cc75fc4aa1e046370bcc05c_1440w.jpg)

所以必须借助第三方工具或手动创建多个虚拟磁盘来绕过限制。我选择的是在VMware中预先配置好100个独立的虚拟磁盘文件，每个10GB，这样既规避了系统限制，又便于管理。

![](https://pic4.zhimg.com/v2-2de39e635ecdb2210f7243181ff8e539_1440w.jpg)

接下来是核心步骤——系统安装与克隆。我先手动安装第一个macOS 10.12（Sierra）系统作为“母版”。安装过程和普通[黑苹果](https://zhida.zhihu.com/search?content_id=277918975&content_type=Article&match_order=1&q=%E9%BB%91%E8%8B%B9%E6%9E%9C&zhida_source=entity)无异：从U盘启动、格式化目标盘、同意许可协议、设置账户信息……一切顺利完成后，我得到了一个干净、可启动的macOS系统。

  

真正体现效率的地方来了：我使用了一款叫Carbon Copy Cloner（CCC）的工具进行批量克隆。CCC本质上是一个高级磁盘备份/克隆软件，支持增量复制、任务计划、多任务并发等特性。我将第一个系统设为“源”，然后依次选择第2到第100个分区作为“目标”，创建了100个独立的克隆任务。由于每个系统只占10GB，且CCC能高效利用空闲带宽，整个克隆过程耗时约1小时——对于100个系统来说，这已经非常惊人了。

![](https://pic2.zhimg.com/v2-e0027ecb1554efc4e0bee010403419a7_1440w.jpg)

分盘，从1到100

![](https://pic4.zhimg.com/v2-5ea03d238e97c2f3e80c0a06607f250b_1440w.jpg)

克隆完成后，最激动人心的环节来了：验证。我打开“启动磁盘”设置，赫然发现里面列出了100个系统卷标，从“1”到“100”，全部命名为“macOS, 10.12.1”。由于一次性展示所有系统太耗时，我采用了随机抽查的方式：分别重启进入第1号、第15号、第43号、第57号、第73号和第100号系统。每一次，屏幕都会经历经典的Apple Logo → 进度条 → 登录界面 → 桌面的流程，而且每个系统都能正常进入桌面、打开“关于本机”查看信息、甚至运行Safari浏览网页——它们彼此完全独立，互不干扰。

![](https://pic3.zhimg.com/v2-0ed55e141d7f84d037c1d7ec3c9e2376_1440w.jpg)

很卡

![](https://pica.zhimg.com/v2-eac5137c3d84594e2bef1f5870692ab2_1440w.jpg)

你可能会问：这么做的意义是什么？坦白说，实用价值有限——没人会真用100个一模一样的系统日常办公。但它的价值在于技术探索与边界突破：它证明了macOS在非官方硬件上的极致可塑性，也展示了现代工具链（如VMware + CCC）如何让看似不可能的任务变得可行。

![](https://pica.zhimg.com/v2-bada84a1e7e8ccd36102415fcebc1504_1440w.jpg)

15号

![](https://pic2.zhimg.com/v2-aa5957a6985c47f36e299b88c9d47725_1440w.jpg)

43号

![](https://pic3.zhimg.com/v2-f204ceee01a0f9b83f7bdaa0691ffa0a_1440w.jpg)

73号

![](https://pic1.zhimg.com/v2-25d538ac699529a5b4d57db50663449a_1440w.jpg)

100号

最后，我想强调一点：这个实验的前提是“黑苹果”环境，即在非Apple硬件上运行macOS。它依赖于特定的引导器（如[Clover](https://zhida.zhihu.com/search?content_id=277918975&content_type=Article&match_order=1&q=Clover&zhida_source=entity)或OpenCore）、定制内核扩展（[kexts](https://zhida.zhihu.com/search?content_id=277918975&content_type=Article&match_order=1&q=kexts&zhida_source=entity)）以及对硬件的精细调校。如果你是新手，不建议盲目模仿；但如果你对操作系统底层、虚拟化技术或自动化运维感兴趣，这类实验可能有用吧。

（依旧不会升华，就这样了）