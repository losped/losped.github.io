---
title: Android官方架构Jetpack-Navigation
date: 2020.06.24 14:13:59
tags: Android
categories: Android
---

分享两篇好文章：
https://www.jianshu.com/p/ad040aab0e66
https://www.jianshu.com/p/729375b932fe

##### 联动
Navigation可以和ActionBar、NavigationView形成联动，实现动态变化.

activity.xml
```
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <androidx.drawerlayout.widget.DrawerLayout
        android:id="@+id/drawer_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:fitsSystemWindows="true">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="vertical">

            <com.google.android.material.appbar.AppBarLayout
                android:id="@+id/appbar"
                style="@style/AppTheme.AppBarOverlay"
                android:layout_width="match_parent"
                android:layout_height="wrap_content">

                <androidx.appcompat.widget.Toolbar
                    android:id="@+id/toolbar"
                    android:layout_width="match_parent"
                    android:layout_height="?attr/actionBarSize"
                    app:popupTheme="@style/AppTheme.PopupOverlay"/>

            </com.google.android.material.appbar.AppBarLayout>

<!--            1. NavHostFragment是导航界面的容器-->
<!--               app:navGraph 声明导航结构图-->
<!--               app:defaultNavHost 拦截系统Back键的点击事件-->
            <fragment
                android:id="@+id/actor_nav_fragment"
                android:name="androidx.navigation.fragment.NavHostFragment"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                app:defaultNavHost="true"
                app:navGraph="@navigation/nav_actor"/>
        </LinearLayout>

        <!-- Container for contents of drawer - use NavigationView to make configuration easier -->
        <com.google.android.material.navigation.NavigationView
            android:id="@+id/navigationView"
            style="@style/Widget.MaterialComponents.NavigationView"
            android:layout_width="wrap_content"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            app:headerLayout="@layout/nav_header"
            app:menu="@menu/menu_navigation"/>

    </androidx.drawerlayout.widget.DrawerLayout>
</layout>
```

MainActivity.kt
```
class MainActivity : AppCompatActivity() {
    private lateinit var drawerLayout: DrawerLayout
    private lateinit var appBarConfiguration: AppBarConfiguration
    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val binding: ActivityMainBinding = DataBindingUtil.setContentView(this,
                R.layout.activity_main)
        drawerLayout = binding.drawerLayout

        // NavHostFragment：Fragment新一代管理库
        navController = Navigation.findNavController(this, R.id.actor_nav_fragment)

        // AppBar联动效果必须使用DrawerLayout+AppBarLayout的组合
        appBarConfiguration = AppBarConfiguration(navController.graph, drawerLayout)
        // Set up ActionBar
        setSupportActionBar(binding.toolbar)
        // 绑定App bar与NavController，让Appbar随Fragment页面的变化动态变化
        setupActionBarWithNavController(navController, appBarConfiguration)

        // 绑定NavigationView与NavController
        binding.navigationView.setupWithNavController(navController)
    }


    // 重写onSupportNavigateUp()方法，将它的 back键点击事件的委托出去
    override fun onSupportNavigateUp(): Boolean {

        // 返回上一个Fragment
        return navController.navigateUp(appBarConfiguration) || super.onSupportNavigateUp()
    }

    override fun onBackPressed() {
        if (drawerLayout.isDrawerOpen(GravityCompat.START)) {
            drawerLayout.closeDrawer(GravityCompat.START)
        } else {
            super.onBackPressed()
        }
    }
}
```

##### action标签

nav_actor.xml
```
<!--app:startDestination 定义默认进入的Fragment需要用navigation标签-->
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools"
            android:id="@+id/nav_actor"
            app:startDestination="@id/actorCollectionFragment">

    <fragment
        android:id="@+id/actorCollectionFragment"
        android:name="me.liangfei.databinding.fragments.ActorCollectionFragment"
        android:label="@string/title_actor_collection">
        <action
            android:id="@+id/showActorDetail"
            app:destination="@id/actorDetailFragment"/>
    </fragment>
    <fragment
        android:id="@+id/actorDetailFragment"
        android:name="me.liangfei.databinding.fragments.ActorDetailFragment"
        android:label="@string/title_actor_detail">
        <argument
            android:name="actorId"
            app:argType="integer"/>
    </fragment>
</navigation>
```

DataBinding会根据<action标签>和目标Framgent的情况，自动生成fragment的id+Directions.java（这里是ActorCollectionFragmentDirections）
这个类用于路由操作，并负责传递目标Fragment的参数.
并且会根据<argument>标签生成fragment的id+Args.java（这里是ActorDetailFragmentArgs）
**Losped：什么原理？** 这样子可用这种方式简单取得传递值 ```val params by navArgs<ActorDetailFragmentArgs>()```

ActorListAdapter.kt
```
// 页面跳转
private fun createOnClickListener(actorId: Int): View.OnClickListener {
    return View.OnClickListener {
        // ActorCollectionFragmentDirections是根据<action>和<argument>自动生成的
        val direction = ActorCollectionFragmentDirections.showActorDetail(actorId)
        // 1. 可以利用<argument>将值传递给目标Fragment
        // 2. 可以利用第二个参数Bundle传递参数给目标Fragment
        it.findNavController().navigate(direction)
    }
}
```

目标页面 ActorDetailFragment.kt
```
class ActorDetailFragment : Fragment() {
    // 委托给navArgs. navArgs是一个扩展函数
    // 实际上ActorDetailFragmentArgs.java是根据<argument>自动生成的,这里可以获取源页面下<argument>的参数
    private val params by navArgs<ActorDetailFragmentArgs>()

    override fun onCreateView(
            inflater: LayoutInflater,
            container: ViewGroup?,
            savedInstanceState: Bundle?
    ): View? {
        val binding = FragmentActorDetailBinding.inflate(inflater, container, false)
        val context = context ?: return binding.root

        // init view pager
        binding.tabLayout.setupWithViewPager(binding.viewPager)
        binding.viewPager.adapter = ActorDetailPagerAdapter(
                requireFragmentManager(), params.actorId)
                ...
        return binding.root
    }
}
```


自动生成的ActorCollectionFragmentDirections.java
```
public class ActorCollectionFragmentDirections {
  private ActorCollectionFragmentDirections() {
  }

  // "actorId"为目标Fragment的<argrument>参数
  @NonNull
  public static ShowActorDetail showActorDetail(int actorId) {
    return new ShowActorDetail(actorId);
  }

  public static class ShowActorDetail implements NavDirections {
    private final HashMap arguments = new HashMap();

    private ShowActorDetail(int actorId) {
      this.arguments.put("actorId", actorId);
    }

    @NonNull
    public ShowActorDetail setActorId(int actorId) {
      this.arguments.put("actorId", actorId);
      return this;
    }

    @Override
    @SuppressWarnings("unchecked")
    @NonNull
    public Bundle getArguments() {
      Bundle __result = new Bundle();
      if (arguments.containsKey("actorId")) {
        int actorId = (int) arguments.get("actorId");
        __result.putInt("actorId", actorId);
      }
      return __result;
    }

    @Override
    public int getActionId() {
      return R.id.showActorDetail;
    }

    @SuppressWarnings("unchecked")
    public int getActorId() {
      return (int) arguments.get("actorId");
    }

    @Override
    public boolean equals(Object object) {
      if (this == object) {
          return true;
      }
      if (object == null || getClass() != object.getClass()) {
          return false;
      }
      ShowActorDetail that = (ShowActorDetail) object;
      if (arguments.containsKey("actorId") != that.arguments.containsKey("actorId")) {
        return false;
      }
      if (getActorId() != that.getActorId()) {
        return false;
      }
      if (getActionId() != that.getActionId()) {
        return false;
      }
      return true;
    }

    @Override
    public int hashCode() {
      int result = 1;
      result = 31 * result + getActorId();
      result = 31 * result + getActionId();
      return result;
    }

    @Override
    public String toString() {
      return "ShowActorDetail(actionId=" + getActionId() + "){"
          + "actorId=" + getActorId()
          + "}";
    }
  }
}
```

自动生成的ActorDetailFragmentArgs.java
```
import android.os.Bundle;
import androidx.annotation.NonNull;
import androidx.navigation.NavArgs;
import java.lang.IllegalArgumentException;
import java.lang.Object;
import java.lang.Override;
import java.lang.String;
import java.lang.SuppressWarnings;
import java.util.HashMap;

public class ActorDetailFragmentArgs implements NavArgs {
  private final HashMap arguments = new HashMap();

  private ActorDetailFragmentArgs() {
  }

  private ActorDetailFragmentArgs(HashMap argumentsMap) {
    this.arguments.putAll(argumentsMap);
  }

  @NonNull
  @SuppressWarnings("unchecked")
  public static ActorDetailFragmentArgs fromBundle(@NonNull Bundle bundle) {
    ActorDetailFragmentArgs __result = new ActorDetailFragmentArgs();
    bundle.setClassLoader(ActorDetailFragmentArgs.class.getClassLoader());
    if (bundle.containsKey("actorId")) {
      int actorId;
      actorId = bundle.getInt("actorId");
      __result.arguments.put("actorId", actorId);
    } else {
      throw new IllegalArgumentException("Required argument \"actorId\" is missing and does not have an android:defaultValue");
    }
    return __result;
  }

  @SuppressWarnings("unchecked")
  public int getActorId() {
    return (int) arguments.get("actorId");
  }

  @SuppressWarnings("unchecked")
  @NonNull
  public Bundle toBundle() {
    Bundle __result = new Bundle();
    if (arguments.containsKey("actorId")) {
      int actorId = (int) arguments.get("actorId");
      __result.putInt("actorId", actorId);
    }
    return __result;
  }

  @Override
  public boolean equals(Object object) {
    if (this == object) {
        return true;
    }
    if (object == null || getClass() != object.getClass()) {
        return false;
    }
    ActorDetailFragmentArgs that = (ActorDetailFragmentArgs) object;
    if (arguments.containsKey("actorId") != that.arguments.containsKey("actorId")) {
      return false;
    }
    if (getActorId() != that.getActorId()) {
      return false;
    }
    return true;
  }

  @Override
  public int hashCode() {
    int result = 1;
    result = 31 * result + getActorId();
    return result;
  }

  @Override
  public String toString() {
    return "ActorDetailFragmentArgs{"
        + "actorId=" + getActorId()
        + "}";
  }

  public static class Builder {
    private final HashMap arguments = new HashMap();

    public Builder(ActorDetailFragmentArgs original) {
      this.arguments.putAll(original.arguments);
    }

    public Builder(int actorId) {
      this.arguments.put("actorId", actorId);
    }

    @NonNull
    public ActorDetailFragmentArgs build() {
      ActorDetailFragmentArgs result = new ActorDetailFragmentArgs(arguments);
      return result;
    }

    @NonNull
    public Builder setActorId(int actorId) {
      this.arguments.put("actorId", actorId);
      return this;
    }

    @SuppressWarnings("unchecked")
    public int getActorId() {
      return (int) arguments.get("actorId");
    }
  }
}
```


##### argument标签
































再见findViewByID.

## 介绍
Data Binding库不仅灵活而且广泛兼容- 它是一个support库，因此你可以在所有的Android平台最低能到Android 2.1（API等级7+）上使用它。
需求：Android Plugin for Gradle 1.5.0-alpha1 或 更高版本。

## 环境
Gradle版本需求：Android Plugin for Gradle 1.5.0-alpha1 或 更高版本。
Android Studio版本需求：请确保您使用的是Android Studio的兼容版本。Android Studio的Data Binding插件需要Android Studio 1.3.0 或 更高版本

添加dataBinding到build.gradle(:project)
```
android {
    ....
    dataBinding {
        enabled = true    
    }    
}
```

## Data Binding Layout文件

##### Data Binding基本用法
用法示例
main_activity.xml
```
<?xml version="1.0" encoding="utf-8"?>

<!--layout标签, 被layout包围的标签才能使用Data Binding-->
<layout xmlns:android="http://schemas.android.com/apk/res/android">

   <!--data标签,type为POJO类-->
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">

       <!--引用data标签的内容填充-->
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

User.java
```
public class User {
   private final String firstName;
   private final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
   public String getFirstName() {
       return this.firstName;
   }
   public String getLastName() {
       return this.lastName;
   }
}
```

Binding数据
默认情况下，一个Binding类会基于layout文件的名称而产生，将其转换为Pascal case（译注：首字母大写的命名规范）并且添加“Binding”后缀。上述的layout文件是main_activity.xml，因此生成的类名是MainActivityBinding。
创建bindings的方式有以下几种：

MainActivity.java
```
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   //第一种
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);

   //第二种
   MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());

   //第三种，如果你在ListView或者RecyclerView adapter使用Data Binding时，可以使用
   ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup,
   false);
   ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);

}
```

##### Data Binding深入使用
```
<!--alias可以重命名-->
<!--class可用来重命名或放置在不同的包中-->
<data class="com.example.ContactItem>
    <import type="android.view.View"
    alias="Vista"/>
</data>

<!--import(导入类型)-->
<!--variable表示定义某种类型的属性-->
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>

<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>

<!--include引用并传入参数-->
<LinearLayout
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include layout="@layout/name"
        bind:user="@{user}"/>
    <include layout="@layout/contact"
        bind:user="@{user}"/>
</LinearLayout>
```

##### Data 对象
修改POJO不会导致UI更新。Data Binding的真正能力是当数据变化时，可以通知给你的Data对象。有三种不同的数据变化通知机制：Observable对象、ObservableFields以及observable collections。
当这些可观察Data对象​​绑定到UI，Data对象属性的更改后，UI也将自动更新。

- 实现Observable 接口
实现android.databinding.Observable接口的类可以允许附加一个监听器到Bound对象以便监听对象上的所有属性的变化。通过指定一个Bindable注解给getter以及setter内通知来完成的。
User.java
```
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }

   // 通过 Bindable注解 监听进行UI更新
   @Bindable
   public String getFirstName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```

- 使用 Observable 类作属性
ObservableFields是自包含具有单个字段的observable对象。它有所有基本类型和一个是引用类型。要使用它需要在data对象中创建public final字段
User.java
```
private static class User {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
}
```

MainActivity.java
```
user.firstName.set("Google");
int age = user.age.get();
```

- 使用 Observable 集合类作属性
MainActivity.java
```
//使用Map
ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);

//使用List
ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);
```

MainActivity.xml
```
//使用Map
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
…
<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>

//使用List。注意需要导入索引文件Fields
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
</data>
…
<TextView
   android:text='@{user[Fields.LAST_NAME]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```


动态Variables的使用
```
// 要强制执行，使用executePendingBindings()方法。
public void onBindViewHolder(BindingHolder holder, int position) {
   final T item = mItems.get(position);
   holder.getBinding().setVariable(BR.item, item);
   holder.getBinding().executePendingBindings();
}
```

##### Binding生成、属性Setters等进阶内容比较少用，反而会使Adapt复杂不少，待需要时再补充

参考文章：https://blog.gokit.info/post/android-data-binding/






































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
1. Build-Make Project
2. gradlew clean assembleDebug publish
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
1. gradlew assembleDebug：打包初始版本的apk和基线包ap(同步更新到maven，补丁生成时会根据maven的历史文件来生成差异补丁)
2. 代码内容更新，注意要修改"versionName"（举例versionName由初始1.1.0更新到1.1.1再更新到1.1.2）
3. gradlew assembleDebug -DapVersion=1.1.0 -DversionName=1.1.2。
**注意该命令会生成两个补丁文件1.1.0-1.1.2和1.1.1-1.1.2，修复补丁的时候，服务端需要根据用户当前的版本号来下发相应的补丁。**
4. 补丁包放入手机指定路径，加载补丁包重启更新。

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
