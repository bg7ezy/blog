---
title: 开发者教你简单插桩适配MIUI8-ROM包
copyright: true
date: 2017-09-04 11:23:45
tags: [Android,MIUI,移植]
categories: [Android,移植]
zhuanzai: "http://bbs.zhiyoo.com/thread-12856293-1-1.html"
---

##### 开发环境
推荐使用Ubuntu14.04及其以上系统
自用镜像：https://pan.baidu.com/s/1b17o0u
需要安装的依赖：curl、git、openjdk-7、android-tools-adb
```shell
sudo apt-get install openjdk-7-jdk git curl android-tools-adb
```
当然miui适配工具还是支持macOS平台、mac用户也可以下载玩耍

<!-- more -->

##### 同步代码
###### 下载repo
魔趣的repo不需要挂ss下载，推荐使用！
```shell
mkdir -p ~/bin
curl http://download.mokeedev.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
echo "export PATH=~/bin:$PATH" >> ~/.bashrc
```
###### 下载适配代码
miui开源地址在这里 `https://github.com/MiCode`
下载miui代码命令：
```shell
mkdir patchrom
cd patchrom
repo init -u git://github.com/MiCode/patchrom.git -b marshmallow
```
整个代码同步大概需要一小时左右，请喝杯茶等待,如果发现速度过慢，请自带梯子，传送门: [http://www.jianshu.com/p/b085b4832fd0](http://www.jianshu.com/p/b085b4832fd0)

##### 选择合适的底包
由于miui开发的代码是针对android6.0也就是marshmallow、所以你的底包也应该是基于Android 6.0 的推荐开发者在有实际手机的情况下进行适配,一方面,在 Android 6.0 上,有一些必要的文件信息需要从手机上获取;另一方面,很多适配出现的问题,是需要在真机上调试才能解决的。

##### 拉取底包vendor
###### 首先新建机型目录
```shell
. build/envsetup.sh
mkdir xblade
cd xblade
```
###### 然后将手机重启到rec模式、
```shell
adb reboot recovery
```
###### 接下来运行如下命令来拉取vendor
```shell
    ../tools/releasetools/ota_target_from_phone -r
```
“-r”的意思是在rec模式下拉包

##### 拷贝makefile并进行配置
从angler文件夹拷贝makefile、参考它进行相关配置

##### 反编译framework
在 makefile 准备完毕后,便可以开始构建新的机型工程。以下命令会自动反编译
```shell
    make workspace
```
##### 首次插桩
在新机型工程生成完毕之后,执行以下命令会完成自动插桩:
```shell
    make firstpatch
```
此时工具将会自动进行插桩、
由于原厂rom和miui官方逻辑问题、会导致部分patch失败、从而产生冲突、产生的冲突在temp/rej下。

##### 解决冲突
有些smali冲突容易解决,甚至可以瞬间解决。一些难以解决的冲突依赖于冲突位置处的上下文,很多时候都是由于 board 和 vendor 在 smali 寄存器变量的使用差异导致的,我们需要从上下文中判断出寄存器变量的语义。这考察开发者着耐心。

没有相关基础的同学、朕赐你一本秘籍: [https://pan.baidu.com/s/1dFHqyvZ](https://pan.baidu.com/s/1dFHqyvZ)

##### 打包
运行如下命令即可在out目录下面生成fullota.zip刷机包
```shell
make fullota
```
如果你需要重新编译rom、则需要运行
```shell
make clean
```
clean完成了再次fullota即可

##### 调试
对新手来讲，适配miui是个相对很难的过程，仅仅解决了冲突并不一定能直接开机。那就需要运用好logcat来抓取错误log、进行相关的分析
```shell
    adb logcat
```
Github上面有很多优秀开发者的开源miui项目，可以参考他们的开源项目进行冲突的修改以及bug的修复。
