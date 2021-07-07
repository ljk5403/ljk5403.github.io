---
title: 避坑：Arch系Linux上nc的安装
date: 2020-12-18 23:15:50
---

# 避坑：Arch系Linux上nc的安装

在为vscode配置ssh及其代理的过程中，遇到如下坑：

`nc`是`netcat`的缩写。在仓库中有两个包提供这一命令，分别是`gnu-netcat`和`openbsd-netcat`。它们提供的二进制文件都叫`nc`，但其参数格式不同。网上大多数用`nc`作为`vscode`的`ssh`代理的文章，用的都是后者。如果安装的是前者就会报错。

幸甚至哉，书以避坑。

by 一只不愿意透露姓名的猫猫