---
title: 踩坑——Hex编码的C++实现
date: 2019-02-23 21:00:00
categories:
  学习
tag: 
  C++
---


# 蜜汁Bug：macOS下C++ open语句无法正确地打开文件
<!--more-->
## ·使用finder
直接使用当前路径（“sample”），不行；  
增加当前路径标识符（“./sample”）,不行；  
使用用户相对路径（“~/workshop/Lab/sample”）,不行；  
使用完整路径（“/Users/xxxx/workshop/Lab/sample”），可以。  
## ·使用terminal
直接使用当前路径（“sample”），可以；  
增加当前路径标识符（“./sample”）,可以；  
使用用户相对路径（“~/workshop/Lab/sample”）,不行；  
使用完整路径（“/Users/xxxx/workshop/Lab/sample”），可以。  

所以，难道是说，在finder中点击执行会使用非当前用户来执行？？？  
卒  


# 问题：移位运算符高位补充元素不定？  
移位时，移出的位数全部丢弃，移出的空位补入的数与左移还是右移有关。  
如果是左移，则规定补入的数全部是0；如果是右移，还与被移位的数据是否带符号有关。若是不带符号数，则补入的数全部为0；若是带符号数，则补入的数全部等于原数的最左端位上的原数(即原符号位)。  

而**char 分为有符号性（signed）和无符号型（unsigned）**两种：
若是signed型，就意味着取值范围为[-128,127]；  
若是unsigned型，就意味着取值范围为[0,255]；  

C语言中我们通常直接用类型char，但是**它究竟是被当做signed型还是unsigned型，由编译器决定**。

