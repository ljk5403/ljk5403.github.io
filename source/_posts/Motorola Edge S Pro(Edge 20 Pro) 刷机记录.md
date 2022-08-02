---
title: Motorola Edge S Pro(Edge 20 Pro) 刷机记录
date: 2022-07-29 17:26:14
categories:
tags:
---

 <!-- more -->

# Motorola Edge S Pro(Edge 20 Pro) 刷机记录

Motorola Edge S Pro，大陆以外称为Edge 20 Pro，一款性价比优良的水桶机，海鲜市场整了一台看起来挺新的，Android 手机到手第一件事肯定是把ta里里外外大清扫一遍！插上数据线开搞！

## Step 1：解锁 bootloader

这个得给 Motorola 点个赞，不需要啥麻烦的连续7天登录之类的东西，直接去官网，按照[官方指南](https://motorola-global-portal.custhelp.com/app/standalone/bootloader/unlock-your-device-a) 操作就行。注意会丢掉保修，但也是没办法的事，自由和安稳总是不能得兼的。

## Step 2：Root，以及不得不的救砖

其实本来想直接用 boot twrp 的方法来做，但[找到的版本](https://unofficialtwrp.com/twrp-3-5-2-root-motorola-edge-s-pro/)boot不上去，试着刷的时候按着第一步做就刷出问题来了——bootloader无法识别到系统，于是找原厂包，先救砖再说！

找到了欧洲版：<https://mirrors.lolinet.com/firmware/moto/pstar/official/RETEU/>

中国版：<https://mirrors.lolinet.com/firmware/motorola/pstar_retcn/>

关于不同版本的缩写，这里有解释：<https://bbs.ixmoe.com/t/topic/22491>

直接用了欧洲版。

Motorola的原厂包似乎都不带 `flash_all.sh` 啥的工具，于是只能找解说，发现其实有个等效的 `flashfile.xml` ，但翻译出来执行比较麻烦，找一找果然又发现先辈写的自动化工具，大概就是模仿windows上的 Motorola 官方刷原厂包工具 RSD Lite，写了一个脚本来实现。链接如下：<https://rootjunky.com/rsd-lite-mac-linux/>

按照链接中的来，注意要进入 `bootloader / AP fastboot mode` 的状态，很顺利地刷完，开机，发现还是略有区别：连接wifi无法通过扫码了，直接试图连接 Google Service——当然是失败了，还好没有强求一定要连接上。

之后就是按照[ Magisk 的文档](https://github.com/topjohnwu/Magisk/blob/master/docs/install.md)来安装了：`adb install` 把 Magisk app 先整上，然后`adb push`把刚刚解压出的 `boot.img` 放上手机，用 Magisk app 来 patch 一下，再 `adb pull` 回来刷：

```bash
adb reboot fastboot # 重启到 fastboot 去
fastboot flash boot magisk_patched-xxxxxxxxxxx.img # 刷写patch后的boot.img
```

重启，打开 Magisk App 应该就可以看到root成功啦。

顺便一说，push的时候发现Android的文件安全机制还是有点影响的，adb 无法 push 到 `/mnt/sdcard/` ，不过可以 push 到 `/sdcard/` 。



这么做以后升级据说是无法ota了，只能再折腾，注意使用 `flashfile.xml` 会删除所有用户文件，但 `servicefile.xml` 不会，拿着新的原厂包去升级就好，记得也要重新 patch boot.img 来恢复 Magisk 哟。 



> 不知不觉一个下午就过去了，大致先整活到这。 2022/07/29



> 因为欧版不能改左侧一键触达的功能，还是决定换回国行了。2022/07/30
