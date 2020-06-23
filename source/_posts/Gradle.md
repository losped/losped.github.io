---
title: Gradle
date: 2019.05.13 10:59:59
tags: IDE
categories: IDE
---


Gradle跟Ant类似 是一个构建工具，它是用来帮助我们构建app的，构建包括编译、打包等过程。在Gradle中，构建一个Project需要执行一系列Task，一个apk文件的构建包含以下Task：**Java源码编译、资源文件编译、Lint检查、打包以生成最终的apk文件等等**。
IDEA创建的Gradle项目与AS创建的略有不同，前者将脚本及项目的Gradle配置写在同一个gradle.build中，后者则是分开成一个脚本gradle.build和多个模块gradle.build。
![as](/images/Gradle/as.jpg)
![idea](/images/Gradle/idea.jpg)

导入的插件核心工作有两个：一是定义Task；而是执行Task。这些插件中定义了构建Project中的一系列Task，并且负责执行相应的Task。一般"com.android.application"整个插件中定义了如下4个顶级任务：
assemble: 构建项目的输出（apk）
check: 进行校验工作
build: 执行assemble任务与check任务
clean: 清除项目的输出
当我们执行一个任务时，会自动执行它所依赖的任务。比如，执行assemble任务会执行assembleDebug任务和assembleRelease任务，这是因为一个Android项目至少要有debug和release这两个版本的输出。

下面举个基本的实例，具体的lint代码检查、dexOptions构建速度、productFlavors分支切换等参数以后再慢慢研究。
build.gradle(Project)
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.
apply from: "config.gradle"
buildscript {  //脚本设置，脚本需要的依赖库等
    ext.kotlin_version = '1.2.60'
    repositories {   //构建脚本中所依赖的库都在jcenter，google()，mavenCentral()仓库下载
        google()
        mavenCentral()
        jcenter()
    }
    dependencies {  //指定了gradle插件的版本
        classpath 'com.android.tools.build:gradle:3.1.4'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {   //当前项目所有模块所依赖的库都在jcenter，google()，mavenCentral()仓库下载
        google()
        jcenter()
        mavenCentral()
        maven {
            url "https://jitpack.io"
        }
    }

}

task clean(type: Delete) {  //定义task命令
    delete rootProject.buildDir
}
```

build.gradle(:app)
```
//加载用于构建Android项目的插件
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'  //扩展插件

// 配置全局变量，调用时为rootProject.ext.configOne
ext{
    configOne = [isRealy:true]
}
def configOne = rootProject.ext.configOne
if(configOne.isRealy){}

android {
//加载用于构建Android项目的插件
    signingConfigs {  //通过将签名配置集成到构建脚本中，我们就不必每次构建发行版本时都手动设置了
        config {
            keyAlias 'losped'
            keyPassword 'losped123456'
            storeFile file('../porcelain.jks')
            storePassword 'losped123456'
        }
    }

    compileSdkVersion rootProject.ext.android.compileSdkVersion
    defaultConfig {  //构建Android项目使用的配置
        applicationId "com.porcelainsky"
        minSdkVersion rootProject.ext.android.minSdkVersion
        targetSdkVersion rootProject.ext.android.targetSdkVersion
        versionCode rootProject.ext.android.versionCode
        versionName rootProject.ext.android.versionName
        //指定一个TestInstrumentationRunner用来跑我们所写的所有的测试用例的,
        //需添加com.android.support.test.依赖
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        signingConfig signingConfigs.config

        // 声明需要使用注解功能。使用注解编译库，需要显示的声明，而butterknife是含有注解编译功能的，但是并没有声明。
        javaCompileOptions {
            annotationProcessorOptions {
                includeCompileClasspath true
            }
        }

        // 实现毛玻璃那种透明的效果需要添加的库
        renderscriptTargetApi 19
        renderscriptSupportModeEnabled true    // Enable RS support

        ndk {

            //APP的build.gradle设置支持的SO库架构
            abiFilters 'armeabi', 'armeabi-v7a', 'x86'
        }
    }

    buildTypes {
        debug {  //对debug版本进行的设置
            minifyEnabled false  //是否开启混淆
            debuggable true
            signingConfig signingConfigs.config  //签名配置集成到构建脚本中
        }

        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            debuggable false
            signingConfig signingConfigs.config  //签名配置集成到构建脚本中
            zipAlignEnabled true  //Zipalign优化（混淆相关）
            shrinkResources false  // 移除无用的resource文件（混淆相关）
        }
    }

    // 自定义输出apk配置
    android.applicationVariants.all { variant ->
        variant.outputs.all {
            outputFileName = "procelainsky_v${variant.versionName}_${variant.name}.apk"
        }
    }

    // 使用gradle支持java8，Java 8的新特性Lambda 结合 RxJava 在一起使用可以简化大量的代码
    compileOptions {
        targetCompatibility JavaVersion.VERSION_1_8
        sourceCompatibility JavaVersion.VERSION_1_8
    }

    //  productFlavors不用切换项目分支就可以编译调试不同项目版本的APK，并且可以快速打包所有项目版本的APK
    productFlavors {

    }

    // 设置dex相关参数提高构建速度
    dexOptions {
        jumboMode true
    }

    // lint代码检查， 用于检查gradle构建过程中的bug
    lintOptions {
        abortOnError false
    }
}

dependencies {  //指定当前模块的依赖（implementation为Gradle3.x版本导入命令，Gradle2.x版本为compile）
    implementation fileTree(include: ['*.jar'], dir: 'libs')  //导入本地jar
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    // exclude group和module：意为在该包的support-annotations模块排除掉com.android.support这个包，解决重复依赖问题
    androidTestImplementation('com.android.support.test.espresso:espresso-core:3.0.1') {
        exclude group: 'com.android.support', module: 'support-annotations'
    }
    // Support库
    implementation rootProject.ext.supportLibs
    // 网络请求库
    implementation rootProject.ext.networkLibs
    // RxJava2
    implementation rootProject.ext.rxJavaLibs
    implementation rootProject.ext.otherLibs
    // APT dependencies(Kotlin内置的注解处理器)
    kapt rootProject.ext.annotationProcessorLibs

    // 底部菜单
    implementation('com.flyco.tablayout:FlycoTabLayout_Lib:2.1.0@aar') {
        exclude group: 'com.android.support', module: 'support-v4'
    }
    //kotlin 支持库
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    //GlideOkHttp
    implementation(rootProject.ext.glideOkhttp) {
        exclude group: 'glide-parent'
    }
    //smartRefreshLayout 下拉刷新
    implementation 'com.scwang.smartrefresh:SmartRefreshLayout:1.0.3'
    implementation 'com.scwang.smartrefresh:SmartRefreshHeader:1.0.3'
    //Banner
    implementation 'cn.bingoogolapple:bga-banner:2.2.4@aar'
    // 视屏播放器
    implementation 'com.shuyu:GSYVideoPlayer:2.1.1'
    //Logger
    implementation 'com.orhanobut:logger:2.1.1'
    //Google开源的一个布局控件
    implementation 'com.google.android:flexbox:0.3.1'
    implementation project(':multiple-status-view')
    //模糊透明 View
    implementation 'com.github.mmin18:realtimeblurview:1.1.0'
    //leakCanary
    debugImplementation 'com.squareup.leakcanary:leakcanary-android:1.5.4'
    releaseImplementation 'com.squareup.leakcanary:leakcanary-android-no-op:1.5.4'
    //腾讯 bugly
    implementation 'com.tencent.bugly:crashreport:2.6.6.1'
    //运行时权限
    implementation'pub.devrel:easypermissions:1.2.0'

}
```

##### gradle版本和gradle插件版本
gradle插件版本：build.gradle文件
```
buildscript {
    repositories {
        jcenter()
        google()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.0'
        //classpath 'com.android.tools.build:gradle:2.3.3'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```
gradle版本：gradle-wrapper.properties文件
先搜索本地的.gradle\wrapper\dists\目录，找不到才从[distributionUrl]下载
```
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-4.6-all.zip
```

版本不对应会时常出现报错、sync失败
gradle插件版本和gradle版本对应可参照官网：https://developer.android.google.cn/studio/releases/gradle-plugin.html#updating-plugin
