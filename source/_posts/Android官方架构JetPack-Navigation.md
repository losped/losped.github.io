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
**Losped：什么原理？** 这样子可用这种方式简单取得传递值
```
val params by navArgs<ActorDetailFragmentArgs>()
```

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
...
