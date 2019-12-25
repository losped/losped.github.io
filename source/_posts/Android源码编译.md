---
title: Android源码编译
date: 2019.12.20 13:39:59
tags: Android
categories: Android
---

Android 源码是非常庞大的，而且每个模块都是用git来进行管理 ，整个Android源码是由很多个git项目构成，Google对Android代码的更新也是更新到相应模块的git项目上。
因此Google开发了repo工具用于管理各模块源码，repo就是封装了git命令的python脚本。
目前只能在MacOs和Linux系统环境下进行编译。

本次编译源码的环境为Ubuntu 16.04（14.04升级），4核i5 + 16G内存。
[ubuntu网易镜像地址](http://mirrors.163.com/ubuntu-releases/)
初始化包大约需要70G空间 + repo sync大约需要40G空间 + 编译建议需要额外的70G空间


##### 一. 工具下载
- 下载repo
```
mkdir ~/bin   # 在home下创建bin文件夹
PATH=~/bin:$PATH   # 把bin文件夹加入环境变量的
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo  > ~/bin/repo #下载repo脚本
# curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo #从官方下载
chmod a+x ~/bin/repo #添加权限
```

.repo/：用于子模块统一管理，有单独git版本管理
.repo/mainfest：用于维护子项目列表，同样有单独git版本管理. 在mainfest中，执行git branch -a 就可以看到所有的分支

- 安装Git
```
sudo apt-get install git
git config –global user.email “×××@××.com”
git config –global user.name “×××”
```

##### 二. 源码下载
(1)从清华镜像下载最新的初始化包，详见
[清华](https://mirrors.tuna.tsinghua.edu.cn/help/AOSP/)
[中科大](https://lug.ustc.edu.cn/wiki/mirrors/help/aosp)
```
wget -c https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar # 下载初始化包
tar xf aosp-latest.tar
cd AOSP   # 解压得到的 AOSP 工程目录
# 这时 ls 的话什么也看不到，因为只有一个隐藏的 .repo 目录
# repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-8.0.1_r1 # 指定版本，如果未指定则使用最新的版本；下载初始化包不切换分支不需再init
repo sync # 正常同步一遍即可得到完整目录
# 或 repo sync -l 仅checkout代码
```
[查看Android 版本](https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds)

(2)从Android官方下载源码，[详见](https://source.android.google.cn/setup/downloading)
```
mkdir WORKING_DIRECTORY
cd WORKING_DIRECTORY
repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
repo sync
```

##### 三. 环境准备
- openJDK8
```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
# sudo update-alternative --config java  # 可用于版本切换
# sudo update-alternative --config javac # 可用于版本切换
```

- 一堆软件包
官方文档没有介绍Ubuntu 16.04所需的软件包。网上普遍为以下清单
```
sudo apt-get install libx11-dev:i386 libreadline6-dev:i386 libgl1-mesa-dev g++-multilib
sudo apt-get install -y git flex bison gperf build-essential libncurses5-dev:i386
sudo apt-get install tofrodos python-markdown libxml2-utils xsltproc zlib1g-dev:i386
sudo apt-get install dpkg-dev libsdl1.2-dev libesd0-dev
sudo apt-get install git-core gnupg flex bison gperf build-essential  
sudo apt-get install zip curl zlib1g-dev gcc-multilib g++-multilib
sudo apt-get install libc6-dev-i386
sudo apt-get install lib32ncurses5-dev x11proto-core-dev libx11-dev
sudo apt-get install libgl1-mesa-dev libxml2-utils xsltproc unzip m4
sudo apt-get install lib32z-dev ccache
```

##### 四. 编译
使用 envsetup.sh 脚本初始化环境
```
. build/envsetup.sh
# 或者 source build/envsetup.sh
```

使用 lunch 选择要编译的目标
```
lunch aosp_arm-eng
```
直接运行 lunch （没有参数），会列出所有支持的类型，输入对应的序号来进行选择。
所有编译目标都采用 BUILD-BUILDTYPE 形式，其中 BUILD 是表示特定功能组合的代号。BUILDTYPE 是以下类型之一：

| 编译类型	 | 使用情况                                                         |
| --------  | ----------------------------------------------------------------|
| user	    | 权限受限；适用于生产环境                                          |
| userdebug	| 与“user”类似，但具有 root 权限和可调试性；是进行调试时的首选编译类型 |
| eng	      | 具有额外调试工具的开发配置                                        |

源码编译
```
# 一般最大支持为电脑CPU总线程的2倍。1CPU*2核心*2线程 = make -j4 到 make -j8。
make -j4
```

##### 五. 遇到问题
- Repo自身更新失败
```
# 修改 ~/bin/repo
# REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
修改 ~/.bashrc, Repo的自动更新路径
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'
```

- 更改本地仓库同步地址
修改 .repo/manifests.git/config
```
-url = https://android.googlesource.com/platform/manifest
+url = git://mirrors.ustc.edu.cn/aosp/platform/manifest # 科大源
# +url = http://mirrors.ustc.edu.cn/aosp/platform/manifest # 由于某种原因不能通过 git 协议同步，只能使用http了
# +url = http://aosp.tuna.tsinghua.edu.cn/platform/manifest # 清华源
```

修改 .repo/manifest.xml（清华源）
```
<manifest>
   <remote  name="aosp"
-           fetch="https://android.googlesource.com"
+           fetch="https://aosp.tuna.tsinghua.edu.cn"
            review="android-review.googlesource.com" />
   <remote  name="github"
```

- 编译错误: Out of memory error.
Androd6.0~Android8.0有两种解决方法
(1)修改./prebuilts/sdk/tools/jack-admin文件
JACK_SERVER_COMMAND="java -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -cp $LAUNCHER_JAR $LAUNCHER_NAME"
添加 -Xmx4096m
JACK_SERVER_COMMAND="java -Djava.io.tmpdir=$TMPDIR $JACK_SERVER_VM_ARGUMENTS -Xmx4096m -cp $LAUNCHER_JAR $LAUNCHER_NAME"

(2)cmd
```export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4096m"
out/host/linux-x86/bin/jack-admin kill-server
out/host/linux-x86/bin/jack-admin start-server
```
若不存在jack-admin, 可再repo sync尝试。注意Android8.0以上 Jack工具链已被弃用。

Android8.0 以上:
?


##### 六. 源码目录
!(目录结构)[make1.png]
- abi: applicationbinary interface, 生成libgabi++.so相关库文件
- art: google在4.4后加入的Dalvik进行时
- bionic: Android的C库文件
- bootable: 启动引导
- build: 存放系统编译规则及generic等基础开发配置包
- cts: Android兼容性测试套件
- dalvil: java虚拟机
- developers: 供开发者使用的几个例子
- development: 开发者的一些例子和工具,导入AS需要用到
- device: 设备相关代码, 各厂商需要配置和修改的代码
- docs: 介绍开源相关文档
- external: android使用的一些模块组件
- frameworks: 核心框架，java和C++编写
- hardware: 部分厂家开源的硬件适配层HAL代码
- kernel: 驱动内核相关代码
- libcore: 核心库相关代码
- libnativehepler: JNI依赖的库
- ndk: ndk相关代码
- out: 编译完成后的输出目录
- packages: 应用程序包
- pdk: google用于减少碎片化的东西
- prebuilt: x86和arm CPU架构下预编译的一些资源
- sdk: sdk及模拟器
- tools:
- system: 底层文件系统库、应用及组件
- vendor: 厂商定制代码




版权声明：本文部分转载自CSDN博主「薛瑄」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/xx326664162/article/details/86354616
