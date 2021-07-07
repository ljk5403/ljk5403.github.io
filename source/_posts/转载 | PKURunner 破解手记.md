---
title: 转载 | PKURunner 破解手记
date: 2019-07-09 23:43:48
tag: 
  转载
---
 

# PKURunner 破解手记

by [HamiltonHuaji](https://github.com/HamiltonHuaji/PKUWalker)  

<!--more-->

## 0x00 工欲善其事, 必先利其器

工具集: apktool, Android Studio, jd-gui, dex2jar, Root Explorer

## 0x01 掀起你的头盖骨, 让我看看你的脸

`apktool d pkurunner.apk -o pkurunner`可得到PKU Runner的smali代码.

`jd-gui`可以反编译获得部分Java源码以帮助人类识别关键代码.据观察`smali/cn/edu/pku/pkurunner`是其主要的代码.

由RE浏览器观察PKU Runner目录可知其数据藏在`files/data.db`里.

因此改造这些数据即可伪造跑步记录.为了让非root玩家也能玩, 直接插入代码是不可行的, 因为太长的代码容易导致各种问题, 同时也难插入apk中.

因此我们选择了导出数据再伪造, 最后导入回来.

## 0x02 把PKU Runner关进虚拟机的笼子

偶然间发现某个函数对vbox进行了检测, 以防止有同学用虚拟机作弊. 不如先改掉它练练手.

经查看Java代码, 其逻辑是检测`/dev`目录下是否有`vbox*`的文件. 但是我们直接让它无论如何都返回true就行了.

因此找到返回值. 它是代码中的一个常量, 那么直接改掉就行了. 经手动创建`vbox`文件测试, 上述改动确实有效.

## 0x03 XML YES

为了导出数据, 我决定直接在它的UI上加个按钮.

因此找到设置界面的XML, 模仿其他条目, 在上面加了两个按钮.

回编译, 果然多了两个暂时没有卵用的按钮.

## 0x04 名副其实

为了让两个按钮名副其实, 我写了另一个app, 它拥有几乎一毛一样的界面, 但是多了给按钮附加`onClickListener`的功能.

当然, `onClickListener`里的东西, 分别就是分享`data.db`和向文件管理器要一个文件并写入`data.db`的代码. (没错, 真的就是用File类什么的暴力写进去.)

为了防止还有其他代码在读写这个文件, 我设计了写入`data.db`后退出整个app的功能.

随后, 将这个app也反编译, 找到其中有用的部分smali并植入PKU Runner的smali代码中.

回编译, 不出所料, 该有的功能都有了.

## EOF

没啥好说的, 喵.
