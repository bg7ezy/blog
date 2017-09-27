---
title: '[教程]如何利用patchrom来编译你自己的MIUI'
copyright: true
date: 2017-09-04 15:30:47
tags:
categories:
zhuanzai: http://www.miui.com/thread-842680-1-1.html
---

##### 前言
本教程基本属于原创，经验之谈，不过也有一些素材来自网络，本人对此不负任何责任。这个教程介于入门和进阶之间，也就是说比如简单一点的像：rom的大体结构啊、android的基本原理啊、linux的基本操作啊。这个教程都一概略过。而比较深入的像：apk的修改、开发，源代码的修改、编写，smali插桩，移植适配其他非官方机型。这也一概没有。

本教程仅介绍如何利用MIUI放出的patchrom资源完成编译MIUI的流程，适用于对自制rom以及linux有一定了解，喜欢miui，喜欢DIY，但不太熟悉如何使用patchrom以及其基本流程的发烧友
如果你对android、linux完全是一个门外汉的话，这个教程对你来说可能会有不少无法理解的地方。而如果你如果已经是大神了，希望多多指点，跟大家交流一下你的经验
<!-- more -->
##### 介绍一些基本概念
首先介绍几个比较基本教程的链接
1. MIUI官方提供的教程：http://www.miui.com/thread-402322-1-1.html
2. CM官方wiki上的CM编译教程：http://wiki.cyanogenmod.org/index.php?title=Build_Guides
3. android官方提供的AOSP编译教程：http://source.android.com/source/initializing.html

要学习利用开源源代码编译android rom的话，看完后面两个基本就够了，CM、PA这类的rom都可以根据CM的编译方法来编译，而AOKP这类的rom可以根据AOSP的编译方法来编译
大部分开源rom在他们的github上也会写上相关的编译流程以及主要的命令，比如PA除了用CM的编译方法手动输入每条命令来编译，也可以直接运行他们写好的build.sh来进行编译，基本上还是以实际文件和github上写明的流程为准
而MIUI当然跟这两种的编译方法都不一样，因为MIUI不是利用android源代码编译出来的，而是反编译现成的rom，修改相关smali来适配各种机型的，所以MIUI自己配置了一套编译平台，这也就是patchrom项目
都知道，android是基于linux内核的，而且谷歌也并没有提供适用于windows平台进行编译的相关代码和工具，所以要从源代码编译android是需要一个linux环境的

虽然谷歌同样也支持在苹果的mac系统上编译，但一方面mac的环境配置较为复杂，另一方面mac用户不如pc用户多，要装mac也远不如装linux方便，所以搞这个的，大部分人都是使用的linux
而基于linux内核的操作系统其实有很多，这些不同的linux系统也叫Linux发行版，最主流最常见最通用的一个版本，就是ubuntu(我个人是opensuse党，也在opensuse上编译过CM、MIUI，不过opensuse相对更复杂，而ubuntu现成的软件也更多更方便一点，所以这个教程里是使用的64位ubuntu 12.10来介绍的)，而因此也有很多基于ubuntu修改的Linux发行版，比如深度的Linux Deepin和雨林木风的StartOS，所以相对来说，不论是原版ubuntu还是Linux Deepin和StartOS都比较适合刚接触linux的新手使用

以下所有输入命令的行为全是在“终端”程序中进行的，下文可能会有多处省略，所以你看到输命令可别问输在哪

##### 编译环境的搭建
我本来是使用的12.04，64位桌面版，后来12.10出来了，就顺着更新上来了，因为我这里运营商强制上网得用他的拨号软件，所以我是在虚拟机里搭建的，其实我是推荐直接装在真机上，这样性能更好，编译速度也更快。ubuntu版本的选择、下载、安装、更换源、系统更新等等，我就不介绍了，网上教程一堆一堆的,在装好了ubuntu之后，我们就要开始android rom编译的环境配置了，大致的说一下，需要下载的文件总共有五步

###### 编译依赖
第一步就是编译依赖的各种软件包，这类东西要是打个比方的话大概就相当于是windows下的一些常用的运行库以及基本的编程软件，因为我是既有编译MIUI，也有编译CM这类的rom的需求，所以没单独挑出MIUI所需要的软件包，而是把我所有安装了的软件包都列出来了：

每个都得装的基本软件包(这个是我自己根据CM的教程和aosp的教程里提到的整理出来的)：git-core gnupg flex bison gperf libsdl1.2-dev libesd0-dev libwxgtk2.8-dev squashfs-tools build-essential zip curl libncurses5-dev zlib1g-dev pngcrush schedtool libc6-dev x11proto-core-dev libx11-dev libreadline6-dev libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown libxml2-utils xsltproc

libncurse5软件包，根据系统版本二选一，文件名跟你的系统版本正好相反：
lib32ncurses5-dev（如果你是64位的系统，就是安装这个）
lib64ncurses5-dev（如果你是32位的系统，就是安装这个）

libreadline6软件包，同上：
lib32readline6-dev
lib64readline6-dev

lib32z1-dev这个是64位装的，因为我没试过32位的系统，所以不知道32位要不要(有没有)lib64z1-dev这个软件包

你既可以使用
```shell
    sudo apt-get install 软件包1名称 软件包2名称 软件包3名称
```
这个命令在“终端”中批量下载、安装以上软件包，也可以在“ubuntu软件中心”中下载一个“新立得软件包管理器”，用它来选择安装

###### 安装JDK环境
第二步就是安装java/jdk了，我是偷懒，直接通过新立得安装“openjdk-6-jdk”这个openjdk6的软件包，如果你不怕麻烦，可以去下载、安装sun的原版jdk，这个网上也是一堆教程，我就不多说明了

###### 安装Android SDK platform-tools
第三步安装Android SDK，Android SDK是android官方提供的一套软件开发套件，里面有很多实用工具，比如adb、fastboot、zipalign，还有被很多人称为android模拟器的功能，先在http://developer.android.com/sdk/index.html页面根据系统版本下载压缩包，选linux版就行了，下载完成后，解压到“主文件夹”下面，这个“主文件夹”类似于windows的我的文档，不过更像是我的文档的上一级目录，就像是
```shell
    C:\Users\用户名\
```

这个目录，而“主文件夹”在linux下的路径是
```
    /home/用户名
```
或者也可以用
```shell
    ~
```
符号来表示

而我们在SDK网站上下载的zip并不包含全部的SDK文件，所以需要打开SDK的管理窗口进行下载，打开tools文件夹，双击android这个文件，选择第一项“在终端中运行”或者第四项“运行”都没有问题，两个选一个就是了(运行之前请事先装好jdk，不然点完之后的窗口就会打不开或者一闪而过)，在弹出的这个SDK管理器窗口中，如图，正在连接谷歌服务器获取文件列表(如果卡住不动，说明服务器被墙了，请自行翻墙)，待获取完毕后，默认是会勾上最新版本的安卓镜像，去掉前面的勾(想在电脑上玩玩安卓模拟器的可以留着)，我们只需要勾上Android SDK Tools和Android SDK Platform-tools，这两个就行了，然后点右下角那个Install packages...的按钮(图上那个按钮因为窗口太小，文字没显示全，你知道是那个就行了)，待下载、安装完毕后，一层层关闭Android SDK的所有窗口，回到“主文件夹”，在左上角菜单的“查看”菜单里，选中“显示隐藏文件”，也可以用Ctrl+H这个快捷键快速打开，找到
```
    .bashrc
```
这个文件，双击打开，拉到最后一行，回车，输入一行定义Android SDK工具的路径的环境变量的代码，
```
    export PATH=${PATH}:路径/tools:路径/platform-tools
```
比如图上，我的用户名是shiro，sdk我是解压在“主文件夹”下的，所以把sdk的路径填入代码，我输入的就是
```
    export PATH=${PATH}:/home/shiro/android-sdk-linux/tools:/home/shiro/android-sdk-linux/platform-tools
```
保存，退出，这行代码只对当前用户有效，注销系统，或者重启系统之后，代码就会生效，输这个代码的意义就在于，如果你没有定义sdk的路径，那以后你要运行adb命令，你就得在终端里切换到/home/shiro/android-sdk-linux/platform-tools目录下或者在adb前面加上这个路径，然后才能运行adb命令，而加入了这行代码之后，无论你的终端当前在哪个目录，只要输入
```
    adb
```
这个三个字母就相当于直接使用adb命令了

###### 创建工作目录
第四步创建工作目录，并安装repo工具，在终端中输入如下两个命令
```
    mkdir -p ~/bin
    mkdir -p ~/MIUI
```
这样就会在“主文件夹”下新建“bin”和“MIUI”文件夹了，然后再在终端中输入这两个命令
```
    curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > ~/bin/repo
    chmod a+x ~/bin/repo
```
这样就会把repo工具下载到“bin”文件夹下面，并且赋予repo执行命令的权限

解释一下，“bin”文件夹是为了方便不同用户单独设置不同的命令而创建的，是跟着那一个用户走的，这样你就可以直接在终端中输入repo命令，而不需要切换目录或者前面加上路径了，如果系统上有多个用户，那就意味着这个命令的设置就只对你一个用户有效，其他用户如果要运行就得切换到这个目录或者前面加上路径了，设置“bin”文件夹的目的就跟前面设置sdk的路径的环境变量是一样的，为了输命令的时候少打一段路径，对于“bin”文件夹内的文件编辑是立即生效的，但文件的增减，比如下载了repo工具进来之后，同样要注销系统或者重启系统之后，才会生效

而这个新建的“MIUI”文件夹，就是我们要把MIUI发布的资源文件下载下来，以后进行编译等操作的主目录了，你也可以把命令里的MIUI改成别的名字，只要上面输的命令和下面要输的命令都统一改成一样的就行了，patchrom的github上的说明是新建了一个叫“patchrom”的文件夹，跟我们这里新建的“MIUI”文件夹都是一样的，随后，在终端中使用下面的命令，进入MIUI文件夹，并使用repo工具初始化这个patchrom项目
```
    cd ~/MIUI
    repo init -u git://github.com/MiCode/patchrom.git -b ics
```
这时"MIUI"文件夹下面就会生成一个“.repo”文件夹，并且终端里会提示你配置用户名、邮箱和提示颜色，根据提示选择跳过不配置就好了，对了，https://github.com/MiCode/patchrom就是MIUI的patchrom项目的网址，而https://github.com/MiCode是整个MiCode的主页，里面就有很多MIUI的开源项目

###### 同步源码
第五步开始同步MIUI的patchrom项目，确保你在“MIUI”文件夹下，执行
```
    repo sync
```
这个命令，repo就会根据.repo/manifest.xml里的内容，下载这个项目所包含的各个子项目了
这里说一下，repo工具是一个基于git命令的android专门的项目管理工具，只要配置好了manifest的xml，就可以通过这个工具批量进行多个git项目的操作，比如我们用的patchrom官方的这个，就是一次性下载完build manifest android honor i9100 lt18i sensation onex p1 miui tools i9300 ones razr lt26i vivo x515m saga所有的这么多个项目，对于MIUI的patchrom来说，这个patchrom项目就是manifest的所在，而CM则是以android项目为manifest的所在，而AOSP则是platform/manifest了
而值得提醒的是，git不支持断点续传，也就是说如果你在用repo或者git下载某一个项目时，中断了，那以前下载的文件就没用了，会残留在那浪费硬盘空间，然后下次再执行命令就重新下载，所以我推荐尽量一次性下完项目的所有文件，或者挑需要的项目下载
而我们需要的几个项目只有build manifest android miui tools以及你需要编译的机型的那一个项目，例如我修改过的patchrom项目的github地址是https://github.com/ymdzq/patchrom，我专门给dhd移植，并需要ds和is的文件，所以我就把其他的机型都去掉，只要下ace saga vivo这三个机型的项目就行了，如果你想自己去掉没用的项目，比如不用的机型，那就可以编辑“.repo/manifest.xml”把里面相应的那一行给去掉，保存
以后每次要更新项目的时候，只需要在“MIUI”文件夹下，执行
```
    repo sync
```
这个命令就会把“MIUI”文件夹下面的所有项目同步到最新版本了

##### 编译的主体
这里先简单的介绍一下patchrom下面的各个子项目：
1. build - MIUI制作的用来负责指导整个编译过程的几个核心文件的所在，还有几个framework的apk，以及四个常用的签名文件
2. manifest - patchrom项目本身，负责告诉repo工具你有哪些个子项目
3. android - MIUI把AOSP的framework和经过他们修改过之后的framework源代码进行的对比，这个项目对编译本身用处不大，主要是为给其他机型移植MIUI的大神提供参考，知道源代码和MIUI做了哪些修改就可以从smali插桩中反着推应该怎么去移植，这个工作难度较高，所以给MIUI适配第三方机型的，我估计民间大神中除了华为的格诺大大以外几乎没有什么人会去自己这样适配其他机型，而还干这种事的像明大，就都是小米的工程师了
4. honor - 华为 荣耀1的机型项目
5. i9100 - 三星 Galaxy S II的机型项目
6. lt18i - 索尼 Xperia arc S的机型项目
7. sensation - HTC Sensation(G14/G18)的机型项目
8. onex - HTC One X的机型项目
9. p1 - 华为 Ascend P1的机型项目
10. miui - MIUI放出的所有资源文件，包括App的apk本身、Framework的jar和apk本身，framework和app的相关的src(但很可惜是资源文件而不是源代码，也就是说是用于反编译插桩的资源文件，而不是像CM、AOSP那样可以直接编译出程序的源代码)，以及data(将要预装在手机中的程序，比如支付宝、游戏中心、搜狗输入法、在线视频、语音助手，未来可能会有更多)
11. tools  - MIUI提供的各种工具，非常齐全，应有尽有，很多工具在编译过程中都会被脚本自动使用到，而且也有制作ota包的工具、apktool、baksmali、smali、mkbootimg、unpackbootimg、signapk等等rom修改者经常单独使用的工具
12. i9300 - 三星 Galaxy S III的机型项目
13. ones - HTC One S的机型项目
14. razr - Moto Razr的机型项目
15. lt26i - 索尼 Xperia S的机型项目
16. vivo - HTC Incredible S(G11)的机型项目
17. x515m - HTC Evo 3D(G17)的机型项目
18. saga - HTC Desire S(G12)的机型项目

MiCode的页面上还有很多后来的其他机型的项目，这里就不一一介绍了，反正很好认，根据需要同步下来就是了

这里说明一下编译MIUI大概的原理的流程：
1. 首先你输入编译命令
2. 根据build项目里的核心文件来针对机型项目里给各个机型适配所修改的文件、设定的规则
3. 把机型项目文件夹中的作为移植base的stockrom打包成zip
4. 并将zip中的app和framework单独解压出来
5. 在out文件夹中调用tools里面的工具和linux系统当中的各种工具进行反编译，应用机型项目中所修改的文件、设定的规则，回编译
6. 输出到target_files文件夹中的对应目录里，并加入recovery、radio、boot、meta等等的相应信息
7. 完成后，打包生成target_files.zip，这个zip就是用于制作ota增量升级包的原文件
8. 再从target_files文件夹中提取system、data、meta、boot等文件到ZIP文件夹中，针对system中的app和framework进行签名
9. 将完成签名之后的ZIP文件夹打包，这样就得到了fullota.zip，这个就是我们说的MIUI完整包，放进手机用recovery刷的zip刷机包

至此，MIUI编译的流程就结束了

了解了流程之后，咱们就开始具体的编译吧
首先，切换到MIUI文件夹下
```
    cd ~/MIUI
```
前面说过，我们进行编译等操作的主目录就是这了
```
    . build/envsetup.sh
```
用这个命令，在主目录下，运行build文件夹中的envsetup.sh脚本，以初始化编译环境，因为每个人的机器不同，MIUI主目录的位置也不尽相同，所以MIUI所有的工具、规则都是用的相对路径，以主目录为准来向上向下推导各个项目的目录，所以envsetup.sh脚本就是用来设定各种变量、确定主目录的路径用的，这个命令很重要，如果你要利用tools里的工具，比如签名apk、制作ota升级包等等，都要事先运行一下这个命令
```
    cd /home/shiro/MIUI/ace
```
我这里的机型项目是以HTC DHD为例，我上面这个命令就是切换进ace的目录，你需要把命令中的路径改成你需要编译的机型的项目路径

输入下面这个命令之后
```
    make fullota
```
复制代码
编译就开始了，经过上面我概述的编译流程，最后等到出现英文提示你编译完成之后，打开机型文件夹里面的out文件夹，比如我的是/home/shiro/MIUI/ace/out，里面就能找到fullota.zip和target_files.zip了

##### 其他的一些说明
从头开始说吧，首先一个,编译对电脑配置（CPU、内存、硬盘速度这三样）要求比较高，虽然patchrom是通过反编译插桩，而不是像CM、AOSP那样通过源代码编译生成目标文件，所以运算量相对较小，但运算量比其日常使用还是比较大的，MIUI开发组他们都是I7的台式机，所以你如果是笔记本的话CPU怎么也得有I5或者同类的产品，台式机起码也得是五年之内的相对中高配置
内存容量最低4G，推荐6G到8G，越高当然是越好，我是12G内存，所以给4G内存给虚拟机也一样游刃有余，如果只有4G内存的，建议还是不要用虚拟机，直接真机装linux比较好，因为MIUI的签名工具由配置文件设定的默认阀值是4G，所以如果你内存太小，一来很慢，二来很可能会中途失败
硬盘的话也不是谁都买得起固态硬盘，笔记本本来速度也不快，编译的时候尽量不要执行其他复制文件或者占用磁盘速度的操作，台式机相对好不少，倒是可以比较随意

搭环境时装的那些软件包不是每个都会派上用场，编译MIUI要用到的不多，不过如果你都装好了之后也没啥坏处，要是有兴趣编译CM、AOSP这类rom的话就可以不用再装了

jdk是选择sun的还是openjdk的这个看喜好，不过openjdk必须是1.6版，sun的原版也必须是jdk6（jdk1.6），不要去下openjdk1.7、jdk7（jdk1.7），这个不是越新就越好，安卓2.3到4.1就盯准了要jdk6（jdk1.6）或者openjdk1.6才能编译，所以顺着它就好了

adb命令在linux上需要设置udev才能使用，不然会提示没有权限的，不过在编译本身的过程中是不会用到adb命令的，所以也无所谓，如果你有使用adb命令的需要的话，这个百度一下就好了

说一件很重要的事，在编译MIUI之前，需要用MIUI/tools/aapt这个文件覆盖掉sdk里platform-tools文件夹下的同名文件，因为MIUI从新编译过了aapt这个文件，很多miui的资源都依赖这个新版的aapt，如果不覆盖的话，编译时就会默认使用sdk里的老版aapt而导致反编译、回编译失败，中断编译

如果你编译完了rom，想要清理机型目录下生成出来的新文件，只需把
```
    make fullota
```
换成
```
    make clean
```
这样下次执行编译命令的时候就相当于是完全重新编译了

关于ota升级包的制作，可以参考@TOMMY 的帖子[http://www.miui.com/thread-509838-1-1.html](http://www.miui.com/thread-509838-1-1.html)
