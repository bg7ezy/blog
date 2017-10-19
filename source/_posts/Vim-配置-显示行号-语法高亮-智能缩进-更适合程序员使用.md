---
title: 'Vim 配置 显示行号 语法高亮 智能缩进 更适合程序员使用 '
copyright: true
date: 2017-09-02 10:55:58
tags: [Vim,语法高亮]
categories: [Linux,编程工具]
zhuanzai: "http://blog.csdn.net/sun_shine_/article/details/8449520"
---

在终端下使用 vim 进行编辑时，默认情况下，编辑的界面上是没有显示行号、语法高亮度显示、智能缩进等功能的。为了更好的在 vim 下进行工作，需要手动设置一个配置文件：.vimrc。在启动 vim 时，当前用户根目录下的. vimrc 文件会被自动读取，该文件可以包含一些设置甚至脚本，所以，一般情况下把 `.vimrc` 文件创建在当前用户的根目录下比较方便，即创建的命令为：`$vi ~/.vimrc` ,设置完后 `$:x 或者 $wq` 进行保存退出即可。有些 linux 里 vi 即调用 vim，可以用 which vi 或 alias 查看。
<!-- more -->
下面给出一个例子，其中列出了经常用到的设置，详细的设置信息请参照参考资料：
```shell
" 双引号开始的行为注释行，下同
" 去掉讨厌的有关 vi 一致性模式，避免以前版本的一些 bug 和局限 
set nocompatible

"显示行号
set number

"检测文件的类型
filetype on 

"记录历史的行数
set history=1000 

"背景使用夜晚模式 // 你会很爽的
color eveing

"语法高亮度显示
syntax on 

"下面两行在进行编写代码时，在格式对起上很有用；
"第一行，vim 使用自动对起，也就是把当前行的对起格式应用到下一行；
"第二行，依据上面的对起格式，智能的选择对起方式，对于类似 C 语言编
"写上很有用
set autoindent
set smartindent

"第一行设置 tab 键为 4 个空格，第二行设置当行之间交错时使用 4 个空格

set tabstop=4
set shiftwidth=4

"设置匹配模式，类似当输入一个左括号时会匹配相应的那个右括号
set showmatch

"去除 vim 的 GUI 版本中的 toolbar
set guioptions=T

"当 vim 进行编辑时，如果命令错误，会发出一个响声，该设置去掉响声
set vb t_vb=

"在编辑过程中，在右下角显示光标位置的状态行
set ruler

"默认情况下，寻找匹配是高亮度显示的，该设置关闭高亮显示
set nohls

"查询时非常方便，如要查找 book 单词，当输入到 / b 时，会自动找到第一
"个 b 开头的单词，当输入到 / bo 时，会自动找到第一个 bo 开头的单词，依
"次类推，进行查找时，使用此设置会快速找到答案，当你找要匹配的单词
"时，别忘记回车
set incsearch

"修改一个文件后，自动进行备份，备份的文件名为原文件名加 “~“后缀
if has(“vms”) //注意双引号要用半角的引号"　"
 set nobackup
else
 set backup
endif
```
如果去除注释后，一个完整的. vimrc 配置信息如下所示：
```shell
set nocompatible  
set number  
filetype on   
set history=1000   
color eveing  
syntax on   
set autoindent  
set smartindent  
set tabstop=4  
set shiftwidth=4  
set showmatch  
set guioptions-=T  
set vb t_vb=  
set ruler  
set nohls  
set incsearch  
if has("vms")  
 set nobackup  
else  
 set backup  
endif  
```

效果图：
![](http://img.my.csdn.net/uploads/201212/29/1356726365_8884.JPG)
