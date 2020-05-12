---
title: NKD-Cmake与Android.mk
date: 2019.12.05 11:09:00
tags: Android
categories: Android
---


  利用NDK生成.so主流的有两种方法，传统的基于 Android.mk、Application.mk 来构建项目的方式和从 AS 2.2 之后便开始采用的 CMake 这种方式。

  ##### 一. Android.mk 方式
可以通过Android Studio，也可以通过命令ndk-build来自己手动生成。
1. 下载NDK并配置环境变量
[Android官方NDK](https://developer.android.com/ndk/downloads/index.html)
2. 在需要编译的文件目录中创建**Android.mk**配置文件
Android.mk
```
# 设置工作目录，而my-dir则会返回Android.mk文件所在的目录
LOCAL_PATH := $(call my-dir)
# CLEAR_VARS: 清除几乎所有以LOCAL_PATH开头的变量（不包括LOCAL_PATH）
include $(CLEAR_VARS)
# 定义变量
LAME_LIBMP3_DIR := lame-3.99.5_libmp3lame
# 设置模块的名称。
LOCAL_MODULE := mp3lame
# 指定参与模块编译的C/C++源文件名。
LOCAL_SRC_FILES := $(LAME_LIBMP3_DIR)/bitstream.c $(LAME_LIBMP3_DIR)/fft.c $(LAME_LIBMP3_DIR)/id3tag.c $(LAME_LIBMP3_DIR)/mpglib_interface.c $(LAME_LIBMP3_DIR)/presets.c $(LAME_LIBMP3_DIR)/quantize.c $(LAME_LIBMP3_DIR)/reservoir.c $(LAME_LIBMP3_DIR)/tables.c $(LAME_LIBMP3_DIR)/util.c $(LAME_LIBMP3_DIR)/VbrTag.c $(LAME_LIBMP3_DIR)/encoder.c $(LAME_LIBMP3_DIR)/gain_analysis.c $(LAME_LIBMP3_DIR)/lame.c $(LAME_LIBMP3_DIR)/newmdct.c $(LAME_LIBMP3_DIR)/psymodel.c $(LAME_LIBMP3_DIR)/quantize_pvt.c $(LAME_LIBMP3_DIR)/set_get.c $(LAME_LIBMP3_DIR)/takehiro.c $(LAME_LIBMP3_DIR)/vbrquantize.c $(LAME_LIBMP3_DIR)/version.c com_dgchina_entry_camera_Mp3Lame.c
# 指定生成的静态库或者共享库在运行时依赖的共享库模块列表
include $(BUILD_SHARED_LIBRARY)
```
3. 在需要编译的文件目录中创建**Application.mk**配置文件。非必需，可以用来配置编译平台相关内容。
Application.mk
```
# 指定我们需要基于哪些CPU架构的.so文件,默认为all
APP_ABI := armeabi armeabi-v7a x86 mips
# 使用ndk-build命令编译时加上-DSTDC_HEADERS
APP_CFLAGS += -DSTDC_HEADERS
# 指明配置文件
APP_BUILD_SCRIPT := Android.mk
```

4. cmd进入该目录，执行命令
ndk-build: 直接使用根目录下的Android.mk和Application.mk编译
ndk-build NDK_PROJECT_PATH=. APP_BUILD_SCRIPT=Android.mk: 指定Android.mk编译，NDK_PROJECT_PATH参数为项目目录
ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk: 指定Application.mk编译，NDK_PROJECT_PATH参数为项目目录

编译成功后在同级目录的libs/obj文件夹中可找到你需要的.so文件.


  ##### 二. Cmake 方式
CMake 是一个跨平台的安装（编译）工具，相比与传统 Android.mk 方便了不少。AS跟IDEA均有CMake插件支持。
1. 同样需要NDK和配置环境变量
2. 一般在项目根目录下(AS-C++ Support项目同级于app的build.gradle)下创建CMakeLists.txt配置文件
```
# For more information about using CMake with Android Studio, read the
# documentation: https://d.android.com/studio/projects/add-native-code.html

# Sets the minimum version of CMake required to build the native library.

# cmake最低支持版本
cmake_minimum_required(VERSION 3.4.1)

# 库名
# project(FaceImageAndroid)

# 得到一个完整文件名中的特定部分赋予 SRC_DIR。ABSOLUTE 表示完整路径
# get_filename_component(SRC_DIR  ${CMAKE_SOURCE_DIR}/..  ABSOLUTE)
# get_filename_component(ANDROID_NDK_EXPECTED_PATH "d:/ahello.c"   ABSOLUTE) #原封不动取目录或包含路径的文件 如果不填写路径会自动 添加上自己项目路径,也就是CMAKE_CURRENT_LIST_DIR和它拼接
# get_filename_component(ANDROID_NDK_EXPECTED_PATH "d:/ahello.c"   NAME) #取文件名,不包含路径
# get_filename_component(ANDROID_NDK_EXPECTED_PATH "d:/ahello.c"   EXT) #取扩展名
# get_filename_component(ANDROID_NDK_EXPECTED_PATH "d:/ahello.c"   REALPATH) #和ABSOLUTE 一样，
# get_filename_component(ANDROID_NDK_EXPECTED_PATH "d:/ahello.c"   PATH) #只保留路径

# Creates and names a library, sets it as either STATIC
# or SHARED, and provides the relative paths to its source code.
# You can define multiple libraries, and CMake builds them for you.
# Gradle automatically packages shared libraries with your APK.

# 判断编译器类型,如果是gcc编译器,则在编译选项中加入c++11支持
# if(CMAKE_COMPILER_IS_GNUCXX)
#     set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
#     message(STATUS "optional:-std=c++11")
# endif(CMAKE_COMPILER_IS_GNUCXX)

# 添加Flag
# if (ANDROID_ABI MATCHES "^armeabi-v7a$")
#     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfloat-abi=softfp -mfpu=neon")
# elseif(ANDROID_ABI MATCHES "^arm64-v8a")
#     set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -ftree-vectorize")
# endif()
#
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DSTANDALONE_DEMO_LIB \
#                     -std=c++11 -fno-exceptions -fno-rtti -O2 -Wno-narrowing \
#                     -fPIE")
# set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} \
#                               -Wl,--allow-multiple-definition \
#                               -Wl,--whole-archive -fPIE -v")

# 把当前工程目录下的 src 目录下的所有 .cpp 和 .c 文件赋值给 SRC_LIST（遍历子文件夹）
# AUX_SOURCE_DIRECTORY(${PROJECT_SOURCE_DIR}/src SRC_LIST)
# FILE(GLOB SRC_LIST "${PROJECT_SOURCE_DIR}/src/*.cpp")
# 指定工程的名称
# PROJECT(MP3LAME)

# 指定本工程中静态库.so生成的位置，即 build/lib;（AS默认存储在app\build\intermediates\cmake下）
# ADD_SUBDIRECTORY(lib)
# 定义变量LIBRARY_OUTPUT_PATH，指定输出 .so 动态库的目录位置（AS默认存储在app\build\intermediates\cmake下）
# SET(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)


# 定义文件SOURCES
set(SOURCES)
file(GLOB_RECURSE SOURCES ${CMAKE_SOURCE_DIR}/*.cpp ${CMAKE_SOURCE_DIR}/*.c)

# 定义集合LIBS
# 链接NDK内部库 jnigraphics (包含bitmap处理)和log，下载并存储在本地NDK或SDK的NDK-bundle中（参考bitmap.h）
# AS引入NDK有以下两种方式：
# 1. 使用sdk中的ndk bundle构建
# 2. 使用本地的NDK构建，需输入路径
# list(APPEND LIBS
#         jnigraphics
#         log
#         )
# 添加态链接库
# target_link_libraries(
#         native-lib
#         ${LIBS})

# 指定生成动态库
add_library( # Sets the name of the library. cmake系统会自动为你生成 libXXX.so/libXXX.a
             # 对应cpp的文件名
             mp3Converter

             # Sets the library as a shared library.  SHARED生成动态库文件扩展名常为 "*.so", STATIC生成静态库文件扩展名常为 "*.a"
             SHARED

             # Provides a relative path to your source file(s). 包含的cpp文件

             # 对应cpp的文件名地址
             src/main/cpp/Mp3Lame.c
        )
# ADD_EXECUTABLE(hello hello.cpp) # 单个文件时

#添加共享库搜索路径
#LINK_DIRECTORIES(${PROJECT_SOURCE_DIR}/lib)

# 加载预编译好的库（外部动态库）
add_library( mp3lame
        SHARED
            IMPORTED )
# 为 mp3lame 添加属性 IMPORTED_LOCATION
set_target_properties( mp3lame
            PROPERTIES IMPORTED_LOCATION
            ../../../../libs/armeabi-v7a/libmp3lame.so )
# 指定生成版本号，VERSION指代动态库版本，SOVERSION指代API版本
# SET_TARGET_PROPERTIES(mp3lame PROPERTIES VERSION 1.2 SOVERSION 1)
# 在输出时以"hello"的名字显示
# SET_TARGET_PROPERTIES (mp3lame PROPERTIES OUTPUT_NAME "m3l")
# 默认构建一个库时，会尝试清理掉其他使用这个名字的库，设置关闭
# SET_TARGET_PROPERTIES (hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)


# 显式定义并赋值变量CMAKE_CXX_FLAGS
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11")

# 指定本地.h头文件目录
include_directories(src/main/cpp)
include_directories(src/main/cpp/include)

# 查找到指定的预编译库, 设置包含的目录
find_library( # Sets the name of the path variable.
              log-lib
              log )

## 为media-handle添加共享库链接(其他so文件)
target_link_libraries( # Specifies the target library.
                       mp3Converter
                       mp3lame
                       # Links the target library to the log library
                       # included in the NDK.
                       ${log-lib} )

# 输出打印构建目录
MESSAGE(STATUS "This is HELLO_BINARY_DIR " ${HELLO_BINARY_DIR})
# 输出打印资源目录
MESSAGE(STATUS "This is HELLO_SOURCE_DIR " ${HELLO_SOURCE_DIR})

# 将hello和hello_static安装到<prefix>/lib目录；静态库要使用ARCHIVE关键字
INSTALL (TARGETS hello hello_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
# 将hello.h安装到<prefix>/include/hello目录。
INSTALL (FILES hello.h DESTINATION include/hello)
```
3. 在CmakeList.txt同级目录下，建立一个build目录。在build目录中cmd执行cmake ..，可以自动生成Makefile等文件。cmd执行make命令,即可生成可执行文件。（AS-C++ Support项目编译即可根据CmakeList.txt生成可执行文件）
运用实例
![cmake](/images/NKD-Cmake与Android.mk/cmake.png)


附Gradle的NDK配置
```
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 28
    defaultConfig {
        applicationId "com.vegen.facedetection"
        minSdkVersion 15
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags "-std=c++11 -frtti -fexceptions"
                abiFilters 'armeabi-v7a'
                arguments "-DANDROID_STL=gnustl_static"
            }
            ndk {
                abiFilters 'armeabi-v7a'
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    externalNativeBuild {
        cmake {
            path "src/main/cpp/CMakeLists.txt"
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
}

```


参考链接：
https://www.jianshu.com/p/528eeb266f83?utm_source=desktop&utm_medium=timeline
https://www.jianshu.com/p/6c09a0717793
