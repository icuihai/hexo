---
title: 交叉编译-Mac环境使用NDK编译FFmpeg
date: 2018-03-30 10:38:43
tags:
---

### 概述

FFmpeg是一套非常强大的音视频处理工具，在音视频领域绝对是一个元老级的存在，围绕FFmpeh可以进行音视频编解码，裁剪，拼接等操作。
今天的主题就是使用NDK进行教交叉编译，生成so文件在Android上使用

##### 我的编译环境：

- FFmpeg v3.0.11 (之前测试最新版3.3.4编译失败)
- macOS 
- NDK android-ndk-r14b
- Android Studio 3.1

### 下载FFmpeg源码

FFmpeg官网下载：https://www.ffmpeg.org/download.html

也可以Git下载 git clone https://git.ffmpeg.org/ffmpeg.git ffmpeg 

###  配置脚本

默认编译出来的so库版本号是在.so之后，Android识别不了，所以要修改configure文件

使用sublime打开configure，大概在3305行

~~~ SLIBNAME_WITH_MAJOR=&#39;$(SLIBNAME).$(LIBMAJOR)&#39;
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
~~~

把上面几行直接改成

~~~SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)$(LIBMAJOR)$(SLIBSUF)' 
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'  
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'  
SLIB_INSTALL_LINKS='$(SLIBNAME)'
~~~

### 配置build_android.sh

这个文件需要自己在FFmpeg根目录下手动创建，直接在sublime新建一个.sh文件。此脚本网上很多，大部分可以直接拿过来使用，但是要注意修改NDK目录。下面我提供一个修改之后的脚本以供参考

~~~ python
# ndk环境   
export NDK=/Users/CH/Learn/android-ndk-r14b
export SYSROOT=$NDK/platforms/android-14/arch-arm
export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/darwin-x86_64
CPU=armv7-a

# 要保存动态库的目录，这里保存在源码根目录下的android/armv7-a
export PREFIX=$(pwd)/android/$CPU
ADDI_CFLAGS="-marm"
function build_android
{
./configure 
	--target-os=linux 
	--prefix=$PREFIX \
    --enable-cross-compile \
    --enable-shared \
    --disable-static \
    --disable-doc \
    --disable-ffmpeg \
    --disable-ffplay \
    --disable-ffprobe \
    --disable-ffserver \
    --enable-avdevice \
    --disable-doc \
    --disable-symver \
    --cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
    --arch=arm \
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $ADDI_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    $ADDITIONAL_CONFIGURE_FLAG
make clean
# 不确定自己上面的目录或者环境有没有错误时
# 可以先注释一下下面两个命令
# make
# make install
}
build_android
~~~

### 执行脚本

进入到脚本根目录，如果你是第一次执行可能会提示权限不足,如下：

*Permission denied*

授予权限即可：

~~~ python
chmod 777 build_android.sh
~~~

然后重新执行

~~~python
./build_android.sh
~~~

此处如果脚本文件环境配置正常会提示一个WARNING，不用管它继续执行

~~~python
make
~~~

大概15分钟之后执行结束根目录下你配置的输出目录下看到.so文件和头文件两个文件夹,如下图：

![](http://p6nix480q.bkt.clouddn.com/18-4-4/90143872.jpg)

#### 提示

尽量先执行./build_android 确认配置无问题在执行  make 和 makeinstall 因为编译一次大概十几分钟。所以说一定先确认环境，目录无问题在执行 make

### 问题

##### 1 .在执行./build_android.sh 时报错如下

_nasm/yasm not found or too old. Use --disable-x86asm for a crippled build_

那就说明你没有安装汇编工具yasm

直接在终端执行

~~~python
brew install yasm
~~~

即可安装

##### 2.在输出目录时报错大概如下

_No such file or directory make: Error 127_

这说明你的输出目录找不到，尽量不要把输出目录建在系统下。还有一点最好手动把输出目录设置为可读写（虽然可能已经是可读写了~）

##### 3.新版本FFpmeg编译失败

目前在我编译过程中最新版本3.4.2是不行的，我目前使用的是3.0.11

##### 4.最好不要使用androidstudio下载的ndk

到官网手动 下载ndk（我使用的是android-ndk-r14b）

https://developer.android.google.cn/ndk/downloads/older_releases.html#ndk-14b-downloads

#### 总结

编译过程中一定要耐心再耐心~特别像我这种不会C不会Linux的，简直是在看天书各种google，下一篇会讲述把so包集成到Android中。

