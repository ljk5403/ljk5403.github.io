---
title: 记一次迷幻的找 Bug：Android Studio 和 vim 的“小”冲突
date: 2020-06-28 06:55:35
tag: [vim,Android]
---


# 记一次迷幻的找 Bug：Android Studio 和 vim 的“小”冲突

这是一个迷幻的 Bug 的发现过程记录，如果想看解决方案可以直接拉到问题发现 的末尾～

<!-- more -->


## 问题发现

问题最开始是这样出现的：因为没有及时和 GitHub repo 保持同步，我本地代码和远端代码的 merge 遇到了 conflicts，于是只好一个一个文件去改，小心翼翼地修改了近十个文件后，在 Android Studio 中 build 报错：

```
Error:Content is not allowed in prolog.
```

去 Google 上查，[第一个搜索结果](https://stackoverflow.com/questions/3030903/content-is-not-allowed-in-prolog-when-parsing-perfectly-valid-xml-on-gae) 说是 XML header 的问题，但辣么多 XML 叫我怎么找啊，于是添加 ` --info` 或是 `--debug` 再 build，虽说没太多有用的信息，但有一句

```
Task ':app:extractDeepLinksDebug' is not up-to-date because:
  Input property 'navFilesFolders' file /Users/xxxxx/AndroidStudioProjects/Xebird/app/src/main/res/navigation/nav_graph.xml has changed.
```

好，起码知道 `nav_graph.xml` 是有问题的——毕竟确实对它做了merge，我天真地想到。

于是先暴力验证一下：上 GitHub 把最新版的 `nav_graph.xml` 弄下来覆盖本地——

依旧报错。

我直接懵了，反复检查，还去查看了其他的几个修改过的 XML 文件，均没有 header上的问题，于是又去 Google 和 Stack Overflow 上反复搜索…… 大家给的解决方案无外乎 clean project 或者删除某些自动生成的文件夹之类的，均失败。在几番尝试未果的情况下，最终我只得另开一个文件夹，pull下来最新的代码，再把自己的修改相应地补上去……

本以为这事就这么结束了。

结果在又一次 merge conflict 时，这个问题又一次出现了！这次的 conflicts 只出现三个文件中，且 `nav_graph.xml` 是其中唯一一个 XML 文件…… 我不禁陷入沉思。

总这样也不是个法子，于是就再仔细看了下错误代码的搜索结果和错误报告，注意到[有一篇文章](https://stackoverflow.com/questions/25145539/android-studio-compile-error-content-is-not-allowed-in-prolog) 讲到了不能在res文件夹下乱放东西，而错误报告里面也只讲到了没能成功完成一项任务，其原因为 `nav_graph.xml` 没能成功更新。

所以，难道我一不小心添加了什么文件？

可我记得完全没有做过文件移动到操作啊，重命名都没有……

我在 Android Studio中找了一遍，没有。

用 ` git status` 看了看，没有。

这岂不是见鬼了……

所以那个鬼文件在哪？难不成真在res文件夹下？于是我直奔res文件夹下，没看到——突然想起，在文件浏览器中有一种文件是看不到的——开头为 `.` 的文件！

于是果断打开 iTerm，`ll`，没有……

不对，记得错误还是和 `nav_graph.xml`  有关的，直接跑去 `res/navigation` 文件夹，`ll`，出来了！

```
total 16
drwxr-xr-x   4 xxxxx  staff   128B xxxxxxxx .
drwxr-xr-x  21 xxxxx  staff   672B xxxxxxxx ..
-rw-r--r--   1 xxxxx  staff     0B xxxxxxxx .nav_graph.xml.un~
-rw-r--r--   1 xxxxx  staff   5.8K xxxxxxxx nav_graph.xml
```

原来是 vim 自动生成的 `.un~`文件！

我预感这基本上就是病因了，于是果断去项目根目录下运行了如下命令：

```shell
# 这个 alias 其实是我写在 .zshrc 里的，以前犯洁癖的时候写着用来清理 removeable devices 上的 vim 隐藏文件
alias clean_vim='find . -name '\''*.un~'\'' -type f -delete'
clean_vim
```
然后再 build，大功告成！

## 总结

熟悉 vim 的朋友应该知道， `.un~` 文件是 vim 为了记录文件历史而设计的，默认是放在与被编辑的文件相同的文件夹下，如果被编辑文件名为 `file` ，则生成的文件名为 `.file.un~`。这个设计其实挺不错的，在确保文件移动（如果是针对文件夹级别的操作）的时候不影响恢复操作的同时，对用户隐藏了具体实现细节。

但这次问题就出在这个文件：出于尚不清楚的原因，Android Studio 没有显示这个文件，同时又在build的时候没有排除掉它。至于为何 ` git status` 没有显示，是因为我在项目之初就把 `*.un~` 加入到了 `.gitignore` 中。

之后复现也基本成功了，不过也很有意思：

在 上次build 通过，未修改 `nav_graph.xml` 的并添加  `.nav_graph.xml.un~` 情况下，直接build，问题不出现——应该是检查发现文件未修改所以跳过了。

在修改 `nav_graph.xml` 的并添加  `.nav_graph.xml.un~` 情况下（顺序无所谓），build 不能通过，且在我使用 `touch .nav_graph.xml.un~` 创建时，报错为 `Premature end of file.` 看起来有些无厘头，不过再 `echo "a" >> ..nav_graph.xml.un~`，马上就回到 `Content is not allowed in prolog.` 这个报错啦。

最后，为了彻底解决这个问题，我参考了一下[这篇文章](https://medium.com/@Aenon/vim-swap-backup-undo-git-2bf353caa02f)，把那些 vim 生成的隐藏文件单独拿出来放着：

在 .vimrc 中添加了

```
set undodir=~/.vim/.undo//
set backupdir=~/.vim/.backup//
set directory=~/.vim/.swp//
```

并建立相应文件夹，再添加 crontab 定期删除：

```
@daily find ~/.vim/.undo -type f -name '*;*' -not \( -atime 0 -or -atime 1 -or -atime 2 -or -atime 3 -or -atime 4 -or -atime 5 -or -atime 6 \) -delete
@daily find ~/.vim/.backup -type f -name '*;*' -not \( -atime 0 -or -atime 1 -or -atime 2 -or -atime 3 -or -atime 4 -or -atime 5 -or -atime 6 \) -delete
@daily find ~/.vim/.swp -type f -name '*;*' -not \( -atime 0 -or -atime 1 -or -atime 2 -or -atime 3 -or -atime 4 -or -atime 5 -or -atime 6 \) -delete
```

OK，问题解决！