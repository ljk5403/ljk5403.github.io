---
title: 参与志愿科学计算：boinc
date: 2021-10-28 19:21:39
categories:  
tags:
---

# 参与志愿科学计算：boinc

本文介绍一种最为常用的参与志愿计算的方法——使用boinc，即 **伯克利开放式网络计算平台(Berkeley Open Infrastructure for Network Computing)** ，来利用自己空闲的计算机/服务器给科学的发展贡献一些微末的力量。愿涓涓细流可汇成江河湖海。

本文作者仅在Ubuntu和Termux（Android上的终端模拟器）使用过boinc，如读者知晓其他平台上的使用方法和注意事项，欢迎在下方评论区交流。

 <!-- more -->

## 注册账号

为了参与不同的项目，我们首先需要注册项目的账号。下文以[www.worldcommunitygrid.org](https://www.worldcommunitygrid.org/) 和 [scienceunited.org](https://scienceunited.org/) 为例，直接上官网注册账号即可。更多的项目可以参考Boinc的官网：https://boinc.berkeley.edu/projects.php

> Science United 似乎是 Boinc 官方制作的一个平台（同为 UCB 设立），可以分配多个平台的计算任务，但作者个人使用中感觉任务分发的优先级似乎并不高：World Community Grid 也加入了 Science United，但我几乎没有在 Science United 的账号下被分配到 World Community Grid 中的任务，与此同时另一台机器上的 World Community Grid 账户满负荷运行。

## 安装

### Ubuntu

直接 `sudo apt install boinc-client `。之后 `sudo service boinc-client start` 即可

### Termux

1. 安装boinc：`apt install boinc`

2. （因为没有service所以只能这样确保后台运行）开一个screen，创建一个单独的文件夹`$boincDir`，之后运行`boinc -dir=$boincDir`

   - 第一次运行可能失败，需要在`$boincDir`下`chmod 644 gui_rpc_auth.cfg`
   - 此后运行`boinccmd`务必到`$boincDir`下运行，不然会报错


## 添加项目到计算机

使用boinccmd添加项目：

### www.worldcommunitygrid.org

1. `boinccmd --lookup_account http://www.worldcommunitygrid.org <username> <password>` 测试是否能连接到project并获取account key
2. `boinccmd --host localhost --project_attach http://www.worldcommunitygrid.org <Account Key>` 添加项目，将自动在后台下载文件、运行。

> 如果使用的termux，必须在`$boincDir`下执行上述命令，否则报错。

### scienceunited.org

1. 添加项目需使用 `boinccmd --join_acct_mgr https://scienceunited.org <username> <password>`
2. 本项目需要比较新的bionc，截止2021年10月20日，暂时不能直接用ubuntu上apt安装的bionc运行（7.9.3 x86_64-pc-linux-gnu），但可以用termux上的运行（7.16.16 aarch64-android-linux-gnu）
3. 似乎会覆盖上面www.worldcommunitygrid.org的项目，因此在链接这个项目后应当重新链接。



## 调整设定

根据个人的考虑，可以设置科学计算的运行强度与时间分配，确保不会影响正常占用。

注意：一般调整完后需要重启boinc，或者`boinccmd --read_global_prefs_override`。

### 方法一：在项目网站上设置

#### www.worldcommunitygrid.org

[Account>>Profile and settings>>Devices>>Device profiles](https://www.worldcommunitygrid.org/ms/device/viewProfiles.do) 可以选择修改默认profile或新建profile。之后到 [Account>>Profile and settings>>Devices>>Device management](https://www.worldcommunitygrid.org/ms/device/viewDevices.do) 选择具体的设备来修改配置即可。

#### scienceunited.org

在[Computing settings](https://scienceunited.org/su_compute_prefs.php)修改。

### 方法二：修改 boinc 本地设置

通过修改`/etc/boinc-client/global_prefs_override.xml`文件，可以本地覆盖项目网站设定的选项。这个设定是全局的，不会因为项目的改变而改变。

具体参数参考：<https://boinc.berkeley.edu/trac/wiki/Prefs2>

对于`global_prefs_override.xml`的具体描述：<https://boinc.berkeley.edu/wiki/Global_prefs_override.xml>

> 另有一个链接的描述比较老旧，不少已经不可用：<https://boinc.berkeley.edu/trac/wiki/PrefsOverride>

常用的配置如限制cpu使用量、使用核心：

```xml
<global_preferences>
  <cpu_usage_limit>60</cpu_usage_limit><!--cpu占用量（如: 每1s占用0.6s的cpu）-->
  <max_ncpus_pct>100</max_ncpus_pct> <!--使用最多多少个cpu（按百分比计算）; 注意<max_cpus>已被废弃！-->
  <!--将默认使用效率较高的核心-->
</global_preferences>
```

修改完参数后需要重启boinc，或者使用这个命令读取配置文件：

```bash
boinccmd --read_global_prefs_override
```

修改完配置后亦可进入top/htop/btop中检查一下进程是否正常地开始运行。
