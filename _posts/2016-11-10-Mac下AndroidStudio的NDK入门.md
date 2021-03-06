---
layout: post
title: Mac下AndroidStudio的NDK入门
date: 2016-11-10 17:20:14
tags: Android
categories: post
---

最近开始学习[NDK开发](https://developer.android.com/ndk/index.html)，Mac环境下的NDK开发，使用了Androidstudio开发工具进行了一个简单的demo开发。这个博客记录我NDK开发入门。
<!--more-->

打开Androidstudio中的SDK-Tools下载工具

![ndk下载](http://of77q1ocj.bkt.clouddn.com/NDK_download.png)

配置好NDK,在terminal中输入

`pico .bash_profile`

然后编辑为

`export NDK_ROOT=/Users/Jing/Library/Android/sdk/ndk-bundle`
`export PATH=$PATH:$NDK_ROOT`

然后测试一下是否可以使用

```shell
$ ndk-build
```

如果输出

```shell
$ Could not find application project directory !
```

那么说明配置成功



新建一个项目，新建类NdkUtil。

```java
public class NdkUtil {

    public static native String say();

    static {
        System.loadLibrary("HelloTest");
    }
}
```

然后在IDE中Rebuild project，然后可以在app-build-intermediates-classes-debug中发现生成了一个包com.knjin.testndk.NdkUtil.class

现在可以使用terminal查找到该.class的位置

```shell
$ javah -jni /Users/Jing/Android/TestNdk/app/build/intermediates/classes/debug
```

然后可以看到该目录下生成了C++头文件``com_knjin_testndk_NdkUtil.h``

或者：

```groovy
terminal处于
main
	-java
的坐标位置

```

```shell
javah -d ../jni -jni com.***.testjni.JniTest   //(JniTest是类名)
											//-d ../jni 是头文件生成的文件夹
```



copy 该文件到main下面的jni文件夹中

```cpp
/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class com_knjin_testndk_NdkUtil */

#ifndef _Included_com_knjin_testndk_NdkUtil
#define _Included_com_knjin_testndk_NdkUtil
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     com_knjin_testndk_NdkUtil
 * Method:    say
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_com_knjin_testndk_NdkUtil_say
  (JNIEnv *, jobject);

#ifdef __cplusplus
}
#endif
#endif
```

然后再jni中新建一个jnihello.c文件

```c
#include <jni.h>

JNIEXPORT jstring JNICALL Java_com_knjin_testndk_NdkUtil_say
        (JNIEnv *env, jobject obj){
    return (*env)->NewStringUTF(env,"这是一个NDK测试程序");
}
```

app的gradle要配置

```groovy
defaultConfig {
    ...
    ndk{
        moduleName "HelloTest"
        abiFilters "armeabi","armeabi-v7a","x86"
    }
}
```

在grade.properties中要输入

```tex
android.useDeprecatedNdk = true
```

现在在/Users/Jing/Android/TestNdk/app/build/intermediates/ndk中会生成一.so包与.mk文件

###使用 Android studio 中自带的 cmake 方法

![CMakeLists.txt](../img/cmakelist.png)

```cmake
cmake_minimum_required(VERSION 3.4.1)
#这里是设置导出的so库的名称
add_library(
            imagelib
            SHARED
            Yuv2Rgb.c)

find_library(
          log-lib
          log
)


# Include libraries needed for Yuv2Rgb.c lib
target_link_libraries(imagelib
                      android
                      ${log-lib})


```

.build中的文件

```groovy
android {
    compileSdkVersion 24
    buildToolsVersion '24.0.2'
    defaultConfig {
        applicationId 'com.example.hellojni'
        minSdkVersion 15
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
        externalNativeBuild {
            cmake {
                arguments '-DANDROID_TOOLCHAIN=clang'
                cppFlags "-frtti -fexceptions"
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
        }
    }
```

*Make Project*

然后在 `app/build/intermediates/cmake`中查找so库.