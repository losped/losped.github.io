---
title: 安卓插件化和热修复（2） - Atlas
date: 2020.06.22 13:51:59
tags: Android
categories: Android
---

## Atlas
Atlas是一个阿里开源的Android客户端容器框架，主要提供了组件化、动态性、解耦化的支持，支持在编码期、Apk运行期以及后续运维修复期修复各种问题。
可用于热更新跟组件化开发。gradle集成了Atlas插件，使用更加方便。Atlas基本使用可见这篇文章：https://blog.csdn.net/xiangzhihong8/article/details/80275201
Atlas的用法有三种：本地bundle、远程bundle和差异补丁

##### gradle配置

build.gradle(:app)配置示例
```
// 需要放最上面初始化
group = "com.android.dzatlas"
// app版本
version = getEnvValue("versionName", "1.1.0");
// ap版本
def apVersion = getEnvValue("apVersion", "");

apply plugin: 'com.android.application'
apply plugin: 'com.taobao.atlas'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.android.dzatlas"
        minSdkVersion 15
        targetSdkVersion 25
        versionCode 1
        versionName version
    }

    signingConfigs {
        debug {
            storeFile file("atlasdemo.jks")
            storePassword "123456"
            keyAlias "key"
            keyPassword "123456"
        }
    }

    buildTypes {
        debug {
            minifyEnabled false
            zipAlignEnabled true
            signingConfig signingConfigs.debug
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.0-alpha8'
    compile 'com.alibaba:fastjson:1.1.45.android@jar'

    //atlas的依赖
    compile('com.taobao.android:atlas_core:5.0.7@aar') {
        transitive = true
    }
    compile 'com.taobao.android:atlasupdate:1.1.4.7@aar'

    // 初次加载组件 打包进apk
    bundleCompile project(':homebundle')
    // 远程加载组件 不打包进apk
    bundleCompile project(':loadingbundle')
    // 公共方法库 宿主与组件使用
    compile project(':middlewarelibrary')
}

atlas {
    atlasEnabled true
    tBuildConfig {
        autoStartBundles = ['com.android.homebundle'] //自启动bundle配置，可赋予多个bundle
        outOfApkBundles = ['loadingbundle']           //不把bundle编译进APK，在客户端使用时下载后加载（会生成.so文件）
        preLaunch = 'com.android.dzatlas.DzPreLaunch' //AppApplication启动之前调用
    }
    patchConfigs {
        debug {
            // createTPatch true 代表我们使用 Atlas 的动态部署补丁修复方案，这里也可以设置为阿里另一款热修复方案 Andfix
            createTPatch true
        }
    }
    buildTypes {
        debug {
            // 对应着本地maven仓库地址 .m2/repository/com/android/dzatlas/AP-debug/1.1.4/AP-debug-1.1.4.ap
            // 这里的 apVersion 代表的是基线版本号，在平时的开发工程中，这个版本号往往是我们通过渠道上线的大版本
            if (apVersion) {
                baseApDependency "com.android.dzatlas:AP-debug:${apVersion}@ap"
                patchConfig patchConfigs.debug
            }
        }
    }
    // 差异补丁生成步骤
    // 1.gradlew assembleDebug 打包初始版本的apk和基线包ap(同步更新到maven，补丁生成时会用到)
    // 更新动作：内容更新+versionName更新（比如versionName从1.1.0改为1.1.1）
    // 2.输入 gradlew assembleDebug -DapVersion=1.1.0 -DversionName=1.1.1
    // 生成补丁差异包 patch-1.1.1@1.1.0-tpatch 、更新说明 update.json
    // 3.补丁包放入手机指定路径，安装补丁更新
}

String getEnvValue(key, defValue) {
    def val = System.getProperty(key);
    if (null != val) {
        return val;
    }
    val = System.getenv(key);
    if (null != val) {
        return val;
    }
    return defValue;
}

apply plugin: 'maven'
apply plugin: 'maven-publish'

publishing {
    // 指定仓库位置
    repositories {
        mavenLocal()
    }
    publications {
        // 默认本地仓库地址  用户目录/.m2/repository/
        maven(MavenPublication) {
            //ap的路径，将上传到maven本地仓库
            artifact "${project.buildDir}/outputs/apk/${project.name}-debug.ap"
            //生成本地maven目录
            groupId group
            artifactId "AP-debug"
        }
    }
}

// 1.远程bundle(即.so文件)，需放入Android/data/mmc.atlastest/cache路径，可直接调用
// 2.gradlew assembleDebug打补丁差异包，用于更新
// 3.本地bundle，直接打包进apk
```

##### 本地bundle
本地bundle将打包进apk，可直接调用

本地bundle build.gradle(:homebundle)配置示例
```
apply plugin: 'com.taobao.atlas.library'

// atlas 本地bundle配置
// 如果要修改资源文件动态更新Bundle需要添加
atlas {
    bundleConfig{
        awbBundle true
    }
    buildTypes {
        debug {
            baseApFile project.rootProject.file('app/build/outputs/apk/app-debug.ap')
        }
    }
}

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"

    defaultConfig {
        minSdkVersion 15
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    providedCompile 'com.android.support:appcompat-v7:25.3.1'
    providedCompile project(':middlewarelibrary')
}
```

##### 远程bundle
远程bundle不会打包进apk。可编译生成.so文件，放入指定目录即可调用。
编译.so文件
1.Build-Make Project
2.gradlew clean assembleDebug publish
.so文件读取路径为 Android/data/mmc.atlastest/cache

远程bundle build.gradle(:loadingbundle)配置示例
```
// atlas 远程bundle配置
// 如果要修改资源文件动态更新Bundle需要添加
// .so文件读取路径为 Android/data/mmc.atlastest/cache
atlas {
    bundleConfig{
        awbBundle true
    }
    buildTypes {
        debug {
            baseApFile project.rootProject.file('app/build/outputs/apk/app-debug.ap')
        }
    }
}

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"

    defaultConfig {
        minSdkVersion 15
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    providedCompile project(':middlewarelibrary')
    providedCompile 'com.android.support.constraint:constraint-layout:1.0.0-alpha8'
    providedCompile 'com.android.support:appcompat-v7:25.3.1'
    providedCompile 'com.android.support.constraint:constraint-layout:1.0.0-alpha8'
}
```

##### 差异补丁
打差异补丁用于版本更新
1.gradlew assembleDebug：打包初始版本的apk和基线包ap(同步更新到maven，补丁生成时会根据maven的历史文件来生成差异补丁)
2.代码内容更新，注意要修改"versionName"（举例versionName由初始1.1.0更新到1.1.1再更新到1.1.2）
3.gradlew assembleDebug -DapVersion=1.1.0 -DversionName=1.1.2。
**注意该命令会生成两个补丁文件1.1.0-1.1.2和1.1.1-1.1.2，修复补丁的时候，服务端需要根据用户当前的版本号来下发相应的补丁。**
4.补丁包放入手机指定路径，加载补丁包重启更新。

加载补丁包代码
MainActivity.java
```
// 插件更新
findViewById(R.id.click2).setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... voids) {
                Updater.update(getBaseContext());
                return null;
            }

            @Override
            protected void onPostExecute(Void aVoid) {
                android.os.Process.killProcess(android.os.Process.myPid());
            }
        }.execute();
    }
});
```

Updater.java
```
package com.android.dzatlas;

import android.content.Context;
import android.os.Handler;
import android.os.Looper;
import android.util.Log;
import android.widget.Toast;

import com.alibaba.fastjson.JSON;
import com.taobao.atlas.dex.util.FileUtils;
import com.taobao.atlas.update.AtlasUpdater;
import com.taobao.atlas.update.model.UpdateInfo;

import java.io.File;

/**
 * Created by wuzhong on 2016/12/20.
 */

public class Updater {

    public static void update(Context context) {

        File updateInfo = new File(context.getExternalCacheDir(), "update.json");

        if (!updateInfo.exists()) {
            Log.e("update", "更新信息不存在，请先 执行 buildTpatch.sh");
            toast("更新信息不存在，请先 执行 buildTpatch.sh", context);
            return;
        }

        String jsonStr = new String(FileUtils.readFile(updateInfo));
        UpdateInfo info = JSON.parseObject(jsonStr, UpdateInfo.class);

        File patchFile = new File(context.getExternalCacheDir(), "patch-" + info.updateVersion + "@" + info.baseVersion + ".tpatch");
        try {
            AtlasUpdater.update(info, patchFile);
            Log.e("update", "update success");
            toast("更新成功，请重启app", context);
        } catch (Throwable e) {
            e.printStackTrace();
            toast("更新失败, " + e.getMessage(), context);
        }

    }

    private static void toast(final String msg, final Context context) {
        new Handler(Looper.getMainLooper()).post(new Runnable() {
            @Override
            public void run() {
                Toast.makeText(context, msg, Toast.LENGTH_LONG).show();
            }
        });
    }


}
```
