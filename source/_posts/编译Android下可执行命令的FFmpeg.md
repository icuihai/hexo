---
title: 编译Android下可执行命令的FFmpeg
date: 2018-04-04 17:22:59
tags:
---

### 概述

上篇文章我们在Mac端交叉编译出来了so文件，但是这个so文件现在还不能直接在Android中使用的，所以说如果想在Android端使用命令执行FFmpeg剪辑音视频文件等，还需要在编译出适合Android的so文件。

##### 说明

在Android端编译so文件有两种方式，一种是比较传统的ndk-build方式，另外一种是AS2.2后比较推荐的CMake,当然这两种只是使用方式稍微有点不一样，但是结果是一样的。本文使用第一种方式，以下内容默认你已经了解NDK开发步骤并且交叉编译出了so文件，如果没有请先看上篇文章 [交叉编译-Mac环境使用NDK编译FFmpeg](http://icuihai.com/2018/03/30/交叉编译-Mac环境使用NDK编译FFmpeg/)

##### 我的编译环境：

- FFmpeg 3.0.11 (之前我用最新版3.3.4编译失败)
- macOS 10.13.2
- NDK android-ndk-r14b
- Android Studio 3.0

#### 编译

大致分为以下几个步骤：

##### 1. 编写native方法

~~~ java
public static native int run(String[] commands);
~~~

##### 2. 加载静态代码块

~~~ java
static {
        System.loadLibrary("avutil-55");
        System.loadLibrary("avformat-57");
        System.loadLibrary("swresample-2");
        System.loadLibrary("swscale-4");
        System.loadLibrary("avcodec-57");
        System.loadLibrary("avfilter-6");
        System.loadLibrary("avdevice-57");
        System.loadLibrary("ffmpeg");
    }
~~~

##### 3. 在main文件夹下新建jni文件夹和jniLibs文件夹b并且导入之前编译好的so文件和include文件

* cd到native路径下利用javah生成.h头文件并拷贝到jni目录下
* 编写相同名字的.c文件并且#include 上一步生成的.h文件

##### 4. 修改源文件（以下文件都在jni目录下）

从ffmpeg源码中拷贝ffmpeg.h、ffmpeg.c、ffmpeg_opt.c、ffmpeg_filter.c、cmdutils.c、cmdutils.h 以及 cmdutils_common_opts.h 共 7 个文件到 jni 目录下，为了将日志输出函数简化为简洁的 “LOGD”、 “LOGE”，需要在jni目录西下新建android_log.h 文件：

~~~ c
#ifdef ANDROID
#include <android/log.h>
#ifndef LOG_TAG
#define  MY_TAG   "MYTAG"
#define  AV_TAG   "AVLOG"
#endif
#define LOGE(format, ...)  __android_log_print(ANDROID_LOG_ERROR, MY_TAG, format, ##__VA_ARGS__)
#define LOGD(format, ...)  __android_log_print(ANDROID_LOG_DEBUG,  MY_TAG, format, ##__VA_ARGS__)
#define  XLOGD(...)  __android_log_print(ANDROID_LOG_INFO,AV_TAG,__VA_ARGS__)
#define  XLOGE(...)  __android_log_print(ANDROID_LOG_ERROR,AV_TAG,__VA_ARGS__)
#else
#define LOGE(format, ...)  printf(MY_TAG format "\n", ##__VA_ARGS__)
#define LOGD(format, ...)  printf(MY_TAG format "\n", ##__VA_ARGS__)
#define XLOGE(format, ...)  fprintf(stdout, AV_TAG ": " format "\n", ##__VA_ARGS__)
#define XLOGI(format, ...)  fprintf(stderr, AV_TAG ": " format "\n", ##__VA_ARGS__)
#endif
~~~

先贴下我的目录吧：

![](http://p6nix480q.bkt.clouddn.com/18-4-4/80850158.jpg)

* 首先修改 ffmpeg.c

1. 导入include "android_log.h"
2. 修改 log_callback_null 方法为下：（原方法为空）

~~~ c
static void log_callback_null(void *ptr, int level, const char *fmt, va_list vl)
{
    static int print_prefix = 1;
    static int count;
    static char prev[1024];
    char line[1024];
    static int is_atty;
    av_log_format_line(ptr, level, fmt, vl, line, sizeof(line), &print_prefix);
    strcpy(prev, line);
    if (level <= AV_LOG_WARNING){
        XLOGE("%s", line);
    }else{
        XLOGD("%s", line);
    }
}
~~~

3. 设置日志回调方法为 log_callback_null：（main 函数开始处）

~~~ c
int main(int argc, char **argv)
{
    av_log_set_callback(log_callback_null);
    int i, ret
~~~

4. 找到 ffmpeg.c 的 ffmpeg_cleanup 方法，在该方法的末尾添加以下代码：

~~~ c
nb_filtergraphs = 0;
nb_output_files = 0;
nb_output_streams = 0;
nb_input_files = 0;
nb_input_streams = 0;
~~~

5. 最后在 main 函数的最后执行 ffmpeg_cleanup 方法，如下：

~~~ 

  	 ffmpeg_cleanup(0);
    return main_return_code;
}
~~~

* 执行结束后不结束进程（修改 cmdutils.c、cmdutils.h）

  由于FFmpeg在执行结束或者遇到异常就会结束进程，但是我们还要Android接着用啊，怎么办呢？那就不让它销毁进程只正常返回就行了，就要需要修改 cmdutils.c 中的 exit_program 方法

~~~ c
void exit_program(int ret)
{
    if (program_exit)
        program_exit(ret);
    exit(ret);
}
~~~

为

~~~ c
int exit_program(int ret)
{
   return ret;
}
~~~

当然.h方法声明也要改哦，即修改cmdutils.h 中的：

~~~ 
void exit_program(int ret) av_noreturn;
~~~

为

~~~ 
int exit_program(int ret);
~~~

到这里我们就把源码修改完了（当然这部分网上一搜一大堆），然后就是编写Android.mk和Applicaton.mk文件了，在这里我贴上我的Android.mk

~~~ c
LOCAL_PATH:= $(call my-dir)
#编译好的 FFmpeg 头文件目录
INCLUDE_PATH:=/Users/CH/Work/FFmpeg/app/src/main/jniLibs/include
#编译好的 FFmpeg 动态库目录
FFMPEG_LIB_PATH:=/Users/CH/Work/FFmpeg/app/src/main/jniLibs/armeabi-v7a

include $(CLEAR_VARS)
LOCAL_MODULE:= libavcodec
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavcodec-57.so
LOCAL_EXPORT_C_INCLUDES := $(INCLUDE_PATH)
include $(PREBUILT_SHARED_LIBRARY)
 
include $(CLEAR_VARS)
LOCAL_MODULE:= libavformat
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavformat-57.so
LOCAL_EXPORT_C_INCLUDES := $(INCLUDE_PATH)
include $(PREBUILT_SHARED_LIBRARY)
 
include $(CLEAR_VARS)
LOCAL_MODULE:= libswscale
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libswscale-4.so
LOCAL_EXPORT_C_INCLUDES := $(INCLUDE_PATH)
include $(PREBUILT_SHARED_LIBRARY)
 
include $(CLEAR_VARS)
LOCAL_MODULE:= libavutil
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavutil-55.so
LOCAL_EXPORT_C_INCLUDES := $(INCLUDE_PATH)
include $(PREBUILT_SHARED_LIBRARY)
 
include $(CLEAR_VARS)
LOCAL_MODULE:= libavfilter
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavfilter-6.so
LOCAL_EXPORT_C_INCLUDES := $(INCLUDE_PATH)
include $(PREBUILT_SHARED_LIBRARY)
 
include $(CLEAR_VARS)
LOCAL_MODULE:= libswresample
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libswresample-2.so
LOCAL_EXPORT_C_INCLUDES := $(INCLUDE_PATH)
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE:= libavdevice
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavdevice-57.so
LOCAL_EXPORT_C_INCLUDES := $(INCLUDE_PATH)
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
#要生成的so文件名字  
LOCAL_MODULE := ffmpeg
LOCAL_SRC_FILES := com_example_ch_ffmpeg_FFmpeg.c \
                  cmdutils.c \
                  ffmpeg.c \
                  ffmpeg_opt.c \
                  ffmpeg_filter.c   
#源码文件             
LOCAL_C_INCLUDES := /Users/CH/Learn/ffmpeg-3.0.11
LOCAL_LDLIBS := -lm -llog
LOCAL_SHARED_LIBRARIES := libavcodec libavfilter libavformat libavutil libswresample libswscale libavdevice
include $(BUILD_SHARED_LIBRARY)

~~~

Application.mk

~~~ c
APP_ABI := armeabi-v7a
APP_PLATFORM=android-14
NDK_TOOLCHAIN_VERSION=4.9
~~~

到这里准备工作已经完成,cd到jni路径下执行

~~~
ndk-build
~~~

然后就可以在java文件下下生成了两个文件夹libs和obj

#### 在应用中加载so文件

我们拷贝上一步生成的libs到app目录下的libs，并且在应用的 build.gradle 文件中 android 节点下添加动态库加载路径，

~~~ java
sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
            jni.srcDirs = []
        }
    }
~~~

和defaultConfig节点下(我这里引入的v7)

```java
ndk {
    abiFilters  "armeabi-v7a"
}
```

到这里全部工作已经完成，接下来就到了就像Mac端使用命令操作音视频的步骤了，比如音频截取，拼接，等。这里我写了一个音频截取的Demo（当然是我工作中需要用到才写的了~~~）

~~~ java
public void run() {
        ...
        soundFile1 = new File(soundFileDir.getPath() + "/" + "V7ExT5s88_13" + ".aac");
        soundFile2 = new File(soundFileDir.getPath() + "/" + "V7ExT5s88_14" + ".aac");
        String[] commands = new String[10];
        commands[0] = "ffmpeg";
        commands[1] = "-i";
        commands[2] = soundFile1.getAbsolutePath();
        commands[3] = "-ss";
        commands[4] = "00:00:10";
        commands[5] = "-t";
        commands[6] = "00:00:20";
        commands[7] = "-acodec";
        commands[8] = "copy";
        commands[9] = soundFile2.getAbsolutePath();
        int result = FFmpeg.run(commands);
        if (result == 0) {
            Toast.makeText(MainActivity.this, "命令行执行完成", Toast.LENGTH_SHORT).show();
        }
    }
~~~

#### 问题

* 首先在编写Android.mk时候路径要要一定要写对，不能对不上
* 修改 cmdutils.c 中的 exit_program 方法时一定要注意void和int（这个问题困扰了我一上午，硬是找不到哪里错了~~~）
* armeabi-v7a和armeabi不一样，无论是在Android.mk或者libs都要注意要对应起来
* 还有好多问题想起来再说吧