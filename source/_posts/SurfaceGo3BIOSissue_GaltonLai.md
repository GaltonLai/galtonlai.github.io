---
title: 【修电脑Vlog】Surface Go3更新BIOS后开机慢，如何降级BIOS？
date: 2026-4-13
updated: 2026-4-23
img: images/pic/cover_SurfaceGo3BIOSissue.png
top: false
coverImg: images/pic/cover_SurfaceGo3BIOSissue.png
tags:
  - 修电脑
  - Surface
  - BIOS
  - 固件问题
  - 开机慢
---

# Surface Go3更新BIOS后开机慢，如何降级BIOS？

**Author:** GaltonLai

**Date:** 2026-04-23

**Link:** https://zhuanlan.zhihu.com/p/2030071541736666466

## 前言
\=========先说一下情况==========

各位大家好，2025年的时候我入手了一台Surface Go 3。

这台Surface Go3更新了微软的[UEFI BIOS](https://zhida.zhihu.com/search?content_id=273534913&content_type=Article&match_order=1&q=UEFI+BIOS&zhida_source=entity)固件，开机长达1分多钟，将近2分钟。

![](https://pic3.zhimg.com/v2-470c903c5fbbb68df187d2386d90a6f4_1440w.jpg)

开机将近2分钟

一开始我开始排除电脑问题。

我排除了外设问题，发现根本不是。

我也重装了几次系统，没有解决。

我尝试进入BIOS，发现BIOS界面也会卡死，可以确定大部分是微软推送的固件有问题。

![](https://pic4.zhimg.com/v2-6616365bd09717a3922fa5a91e0adfe5_1440w.jpg)

进入BIOS时会卡死

\=========背景==========

在2025年9月，微软推送了16.106.143.0这个版本的固件，电脑开机就开始变得特别慢。

在2025年12月，微软推送了16.110.143.0这个版本的固件，没有解决。

在2026年1月，微软推送了17.103.143.0这个版本的固件，同样也没有解决。

然后我就看到了微软论坛这位网友的方法，尝试降级，我这里也顺便把降级的经验分享出来。

![](https://pic3.zhimg.com/v2-7ac999193a574b215c70b02bc048ed7e_1440w.jpg)

感谢这位微软论坛的网友

\=========需要准备的文件==========

1.一台能用的电脑

2.一个U盘（和扩展坞）

3.降级要用到的文件，可以在这里下载：

[Github网页中的surface-uefi-firmware](https://link.zhihu.com/?target=https%3A//github.com/linux-surface/surface-uefi-firmware)

4.Surface Go3的旧版驱动msi包，可以自行在微软官网下载，下载19044这个版本。

[微软官网-Surface Go3驱动程序下载](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/download/details.aspx%3Fid%3D103504)

\=========做法==========

这是我的操作流程，我的流程是这样的。

![](https://pic2.zhimg.com/v2-777a093a2981e7a79b7224891a682be5_1440w.jpg)

操作流程

## 第一步

开始之前，先卸载UEFI相关的硬件驱动（这个驱动会自动更新UEFI固件，到时候又会变成开机慢的状态了）

“此电脑”右键→管理→设备管理器→展开“固件”→找到Surface UEFI（有时候会显示成“固件设备”）→右键卸载→勾选“删除相关软件”→确定

![](https://pica.zhimg.com/v2-5788e5b86b5a89ac352d5f7b46f363ca_1440w.jpg)

卸载Surface UEFI驱动

然后屏蔽掉Windows更新

我用的是[dism++](https://zhida.zhihu.com/search?content_id=273534913&content_type=Article&match_order=1&q=dism%2B%2B&zhida_source=entity)这个软件→系统优化→勾选“更新不包含驱动程序”→Windows自动更新调整为“从不检查更新”

![](https://pic4.zhimg.com/v2-743349d58c593869782837947a53bde1_1440w.jpg)

屏蔽掉Windows更新

补充：原厂系统会预装“Surface”软件，这个软件也会更新驱动程序，最好也卸载掉。

## 第二步

**提前说明：如果你选择的系统制作成U盘启动盘之后，进入livecd之后没有问题，就不用那么复杂了，准备好材料直接第3步即可。**

然后准备一个空U盘和一个Linux系统镜像，可以制作livecd也可以把整个Linux系统安装进U盘。（当然选择你熟悉的Linux系统就好了）

我用的是[ubuntukylin22.04](https://zhida.zhihu.com/search?content_id=273534913&content_type=Article&match_order=1&q=ubuntukylin22.04&zhida_source=entity)，使用livecd时有时候会出现卡死，所以我这边就把U盘挂载进虚拟机，把U盘当做物理硬盘，这样虚拟机装系统的时候也顺理成章地把系统装进U盘了。

![](https://pic4.zhimg.com/v2-5d52f44f3719c5d72d751662e3db25f9_1440w.jpg)

创建虚拟机时选择物理硬盘

![](https://pic3.zhimg.com/v2-2fb5bf68316a9df72c622de0d413ba98_1440w.jpg)

打开cmd，通过diskpart然后list disk，查看U盘是几号盘，虚拟机设备就选择几号盘就行了。

![](https://picx.zhimg.com/v2-9284cab96b643cbbdb4b6c7b545ce351_1440w.jpg)

通过虚拟机把系统装进U盘

## 第三步

制作好启动盘之后，通过扩展坞插入Surface Go3（因为机器就一个TypeC口），插好U盘之后同时按住电源键和音量上键（音量+）先进入BIOS

![](https://pic4.zhimg.com/v2-89fb2f96392477ebba079c7a280c2ee9_1440w.jpg)

插入U盘

关闭安全启动（否则Linux系统进不去）

Security→Secure Boot按下“Change configuration”

![](https://pica.zhimg.com/v2-71551c86cf046c23acd64a2b4c373bd6_1440w.jpg)

左侧导航栏Security

弹出的界面选择“None”。

![](https://picx.zhimg.com/v2-5c41d6ffc0919d7ef83ea3e4f79f6143_1440w.jpg)

关闭安全启动

然后左侧导航栏Exit重启电脑。

重启电脑的时候按下电源键和音量下键（音量-），让这个电脑引导U盘里面的Linux系统。

![](https://pic3.zhimg.com/v2-10c15eed19e4a189adb44d9b4e4acc94_1440w.jpg)

U盘引导系统

系统进来之后连接WIFI，然后更新软件包

sudo apt-get update -y && sudo apt install vim msitools gcab dos2unix -y

![](https://picx.zhimg.com/v2-06c72a7863d38e25af4c0cc6f42dae37_1440w.jpg)

安装软件包

## 第四步

然后想方设法把降级要用到的文件和旧版驱动包导入进Linux系统里面，可以使用另外一个U盘传过来也可以用远程软件发送过来。

![](https://picx.zhimg.com/v2-6a52e196931ae1b79b9b2753361ab551_1440w.jpg)

U盘有这2个文件

## 第五步

然后cd surface-uefi-firmware/

给里面的shell文件一个执行权限：chmod +x ./surface-uefi-fireware/repack.sh

然后通过脚本把驱动包解压下来sudo ./surface-uefi-fireware/repack.sh (你存放的msi文件) -o [fwupd](https://zhida.zhihu.com/search?content_id=273534913&content_type=Article&match_order=1&q=fwupd&zhida_source=entity)ates

（-o 表示解压时指定的文件夹）

![](https://pica.zhimg.com/v2-d6e36f4e2fae36ab202ac2334ed5eabc_1440w.jpg)

解压驱动包

解压下来的树形目录：tree fwupdates/

![](https://pic1.zhimg.com/v2-abe522269364880ac4436029bda64cc8_1440w.jpg)

解压下来的树形目录

然后关闭数字签名：

sudo vim /etc/fwupd/daemon.conf（有的系统是 fwupd.conf）

把OnlyTrusted=true修改成OnlyTrusted=false

![](https://pic3.zhimg.com/v2-2b24c0f1a5bb20fa7a1be73d0b683ccc_1440w.jpg)

关闭数字签名检验

然后是重头戏

for f in fwupdates/\*; do

sudo fwupdmgr install --allow-older --allow-reinstall --no-reboot-check "$f"

done

命令解析：这里是开始用fwupd来安装，我这里是写一条for循环结构，然后遍历安装每一个cab文件。

但是其实也可以只装UEFI相关的这个文件，只不过不知道会不会出现旧版固件不兼容新版驱动的情况，所以我觉得稳妥一点就是全部遍历装一遍。

![](https://pic4.zhimg.com/v2-e9012751647b176ddc3fc11663a71c4f_1440w.jpg)

安装旧版固件驱动

安装成功之后reboot重启电脑。

等电脑安装时会跑进度条，跑完之后进入BIOS，可以发现BIOS版本号降级成功。

![](https://picx.zhimg.com/v2-b8787c34748349de3e0f72960c6fbbf7_1440w.jpg)

BIOS版本号降级成功

之后电脑开机慢的问题解决了。

![](https://picx.zhimg.com/v2-4eed5db359082867c03836746248bd3f_1440w.jpg)

降级后的开机速度
