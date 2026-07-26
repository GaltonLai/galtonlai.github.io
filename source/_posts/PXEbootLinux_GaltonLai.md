---
title: 通过实体机的网络引导网盘内的系统？怎么做到的
date: 2026-6-27
updated: 2026-6-28
img: images/pic/cover_PXEBootLinux.png
top: false
coverImg: images/pic/cover_PXEBootLinux.png
tags:
  -  探究实验
  -  系统整活
  -  Linux
  -  网盘
---

# 通过实体机的网络引导网盘内的系统？怎么做到的

**Author:** GaltonLai

**Date:** 2026-06-27

**Link:** https://zhuanlan.zhihu.com/p/2054241718854329478

大家好，近期我做了一个实验：就是尝试通过实体机的网络引导（[PXE](https://zhida.zhihu.com/search?content_id=277811434&content_type=Article&match_order=1&q=PXE&zhida_source=entity)）来加载一个运行在网盘上的Linux系统。

这个想法源于一位网友的启发——他提到，理论上可以将任何系统镜像上传到网盘，再通过网络引导启动。这听起来像是不是轻易实现的事情，但经过反复调试与踩坑，我最终成功实现了“从OneDrive启动Linux”的目标。整个过程充满挑战，也让我对PXE、initramfs、[JuiceFS](https://zhida.zhihu.com/search?content_id=277811434&content_type=Article&match_order=1&q=JuiceFS&zhida_source=entity)等底层技术有了更深刻的理解。

* * *

实验的核心目标是：**不依赖本地存储介质（如U盘或硬盘），仅通过网络和云存储服务，让一台裸机直接启动并运行一个完整的Linux系统**。我选择的方案分为两个阶段：

第一阶段尝试用JuiceFS挂载OneDrive作为根文件系统。

第二阶段则改用静态initramfs+[IPXE](https://zhida.zhihu.com/search?content_id=277811434&content_type=Article&match_order=1&q=IPXE&zhida_source=entity)的方式，绕过动态挂载的性能瓶颈，最终取得成功。

  

* * *

  

### 方案一：JuiceFS + [rclone](https://zhida.zhihu.com/search?content_id=277811434&content_type=Article&match_order=1&q=rclone&zhida_source=entity) + FUSE —— 理想很丰满，现实很骨感

我的第一套方案是这样的：

1.在一台Linux主机（A机）上配置rclone，将OneDrive挂载为本地目录；

![](https://picx.zhimg.com/v2-4ffa3687f683453fedf75c27719e79dd_1440w.jpg)

![](https://pica.zhimg.com/v2-6119d1088009a9d78e7048a61a254592_1440w.jpg)

![](https://pica.zhimg.com/v2-ef194bdf74f8430c69f25e5458861b22_1440w.jpg)

![](https://pic2.zhimg.com/v2-cd6147f713513e85b47f34a5fdd2b5b5_1440w.jpg)

![](https://pic1.zhimg.com/v2-020484b504fafacac76d0f4e56db20bc_1440w.jpg)

使用JuiceFS将该目录封装成一个FUSE文件系统；

![](https://pic1.zhimg.com/v2-b6ec656e77d0dbbdff46761d8894d10c_1440w.jpg)

![](https://pica.zhimg.com/v2-dc5176c0875de9512bafe7e0e7b96370_1440w.jpg)

用[Alpine Linux](https://zhida.zhihu.com/search?content_id=277811434&content_type=Article&match_order=1&q=Alpine+Linux&zhida_source=entity)最小化镜像作为基础，打包一个包含`rclone`、`juicefs`、`fuse`等工具的initramfs；

![](https://pic1.zhimg.com/v2-9d81833397e417c05499d004ab1ae084_1440w.jpg)

将整个系统通过HTTP服务器提供，并配置PXE引导链：`pxelinux → iPXE → HTTP下载initrd → 启动时自动挂载OneDrive为根文件系统`。

**_听起来逻辑清晰，但执行起来问题频出。_**

* * *

首先，我在A机上安装了`rclone`和`juicefs`，并通过`rclone config`完成了OneDrive的OAuth认证。认证过程需要浏览器跳转，我复制了授权链接，在Firefox中登录微软账号完成授权，返回后rclone成功生成了配置文件。

接着，我尝试用`rclone mount onedrive:/ /mnt/onedrive`挂载网盘。然而，由于OneDrive的API限制，每次读取小文件都会触发大量HTTP请求。当系统启动时尝试`ls /`，它会遍历整个根目录结构——而JuiceFS为了支持POSIX语义，会把每个目录项都当作独立对象去请求，导致成千上万次API调用。结果就是：**挂载成功了，但`ls`命令卡死数分钟，甚至超时失败**。

![](https://pica.zhimg.com/v2-467d3012e91a9fe0ce30c880bb942238_1440w.jpg)

更致命的是，PXE环境下的initramfs内存极其有限（通常只有几十MB），而JuiceFS+Fuse+rclone的组合体积庞大，且依赖glibc等动态库，在精简的Alpine环境中难以静态编译。

![](https://picx.zhimg.com/v2-46c08443041ebdba649c60ebae18b745_1440w.jpg)

我尝试用`busybox`替代部分工具，但仍无法解决核心问题：**网盘延迟太高，无法支撑系统正常启动所需的文件I/O**。

![](https://picx.zhimg.com/v2-54f10c3f13e492ed714f36336dc3ae89_1440w.jpg)

最终，我在日志中看到大量`InvalidResource: ObjectHandle is Invalid`错误，这是OneDrive API的限流响应。方案一宣告失败。

![](https://pic2.zhimg.com/v2-5b58909504d3578910998ff9f526b095_1440w.jpg)

![](https://pica.zhimg.com/v2-abff4e775b45222afe97c7f234cd4c62_1440w.jpg)

* * *

### 方案二：静态initramfs + IPXE —— 绕过动态挂载，直击本质

  

既然动态挂载行不通，我决定换一条路：**不挂载网盘，而是把整个系统“预打包”成一个静态initramfs镜像，上传到网盘，由PXE直接下载并解压运行**。

具体步骤如下：

**1.构建完整根文件系统**  

我在A机上创建`/tmp/initramfs/`目录，手动复制`/bin`、`/etc`、`/lib`、`/usr`等关键目录，并将`rclone`、`busybox`、`bash`等必要二进制文件静态编译或拷贝进去。特别注意：必须确保所有依赖库（如`libc.so`）都一并放入`/lib`，否则运行时会报错。

![](https://picx.zhimg.com/v2-786c11aa7aa3d0d7ff380f1eecf5124b_1440w.jpg)

**2.编写init脚本**  

在`/tmp/initramfs/init`中写入一个简单的shell脚本，功能是：

-   挂载`/proc`、`/sys`、`/dev`
-   挂载临时文件
-   为了测试，我还加了一个彩蛋：启动后先玩一个“猜数字”小游戏（代码只有几十行，用于验证系统是否真正进入用户空间）

![](https://pic3.zhimg.com/v2-a74248bcd9a8e3ec02abe912b9468b4c_1440w.jpg)

**3.打包镜像**  

使用`find . | cpio -ov --format=newc | gzip > initrd-from-cloud.img`生成压缩的initramfs镜像。这个镜像不大，完全可接受。

![](https://pica.zhimg.com/v2-063b5eed0d4ae57f9366d0c4b766dd20_1440w.jpg)

**4.上传至网盘**  

将`initrd.img`和内核`vmlinuz`以及各个引导文件上传到OneDrive，再检查OneDrive是否挂载到本地目录。

![](https://pica.zhimg.com/v2-995aa8b3254d10027ac58b815b21c702_1440w.jpg)

![](https://picx.zhimg.com/v2-51fc2449830d3cf7e5bad9645df3f869_1440w.jpg)

**5.配置PXE服务器，把引导文件放网盘里**  

A电脑（构建机）安装[dnsmasq](https://zhida.zhihu.com/search?content_id=277811434&content_type=Article&match_order=1&q=dnsmasq&zhida_source=entity)（提供DHCP+TFTP），并配置ipxe脚本：

```bash
#!ipxe
dhcp
kernel http://192.168.31.142:8000/vmlinuz initrd=initrd.img console=tty0
initrd http://192.168.31.142:8000/initrd.img
boot
```

  

注意：这里vmlinuz和initrd.img实际是从A电脑的HTTP服务（Python http.server）提供的，而真正的系统文件（即initramfs内部的内容）是从OneDrive下载的——如何实现？**靠rclone挂载，我在挂载的目录改文件就相当于直接改网盘文件。**

![](https://pica.zhimg.com/v2-c6735043e78a32b13794face8096e8c0_1440w.jpg)

**6.启动服务，测试效果**

我找来一台老款联想笔记本（B电脑），装的是Windows 7。通过采集卡将其屏幕投影到另一台电脑上，以便录制启动过程。

![](https://pic3.zhimg.com/v2-82ac16d9f917ab2170821d6d9bc4dca8_1440w.jpg)

首先，将B机网线接入路由器，确保与A机在同一局域网。然后修改B机BIOS，启用网络启动（PXE）。重启后，B机进入Intel UNDI PXE界面，自动获取IP（192.168.31.115），并从A机的TFTP服务器下载iPXE脚本。

![](https://picx.zhimg.com/v2-f1de4f7eaf040dbdf03d3c7cdca373eb_1440w.jpg)

B电脑（测试机）成功获取IP地址

![](https://pica.zhimg.com/v2-99d7b5a372dbcd854408dc3ed44d56a6_1440w.jpg)

开始获取网盘文件

为了验证数据来源，我抓包分析：Wireshark显示，B电脑（测试机）在PXE阶段从A电脑（192.168.31.142）下载了initrd，随后又向OneDrive的IP（52.x.x.x）发起了HTTP GET请求，下载了相关文件**。而该IP经查询正是微软[Azure数据中心](https://zhida.zhihu.com/search?content_id=277811434&content_type=Article&match_order=1&q=Azure%E6%95%B0%E6%8D%AE%E4%B8%AD%E5%BF%83&zhida_source=entity)节点**——铁证如山，系统确实来自网盘！

![](https://pic1.zhimg.com/v2-974d241376f4b59569b38b7b9df53e98_1440w.jpg)

  

![](https://pic4.zhimg.com/v2-8ab59e6e08d56997d58701828fcc9b4f_1440w.jpg)

紧接着, iPXE执行HTTP请求，下载`vmlinuz`和`initrd.img`。initramfs启动后，解压到内存。全程耗时约2-3分钟——虽然不算快，但**稳定可行**！

成功引导系统init脚本，开始_“猜数字”_。

![](https://pic3.zhimg.com/v2-4eed706d162d8909f6223d6eaad5d8c0_1440w.jpg)

![](https://pica.zhimg.com/v2-2c9909dc4733401647d364ea1ac9f934_1440w.jpg)

数字猜对了，进入bash

![](https://pic2.zhimg.com/v2-85725a72a7f43cb25ab7c875fc5dadc1_1440w.jpg)

虽然有点问题，但还是成功了。

* * *

### 总结与思考

  

这次实验让我深刻体会到：**网络引导的本质，不是“挂载远程磁盘”，而是“下载并执行远程代码”**. JuiceFS的思路方向没错，但选错了场景——它适合长期运行的服务端，不适合一次性启动的initramfs。

  

而静态initramfs+IPXE的组合，巧妙地将“大文件下载”与“系统启动”解耦，既规避了API限流，又充分利用了HTTP协议的高效性。即使网速一般，**只要能完整下载镜像，就能成功启动**。

  

当然，这套方案仍有局限：**依赖公网稳定性、启动速度受网速影响、无法热更新。**但对于应急恢复、无盘工作站、或者单纯的技术探索，它提供了一种极富创意的解决方案。

  

技术的魅力，正在于把看似不可能的事，变成一行行可执行的代码。这一次，我让Linux从云端“跃”到了物理机上——这可能也是一种成就吧。（无所谓了，就这样写吧）
