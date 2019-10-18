---
title: Accessibility机制
date: 2019.10.12 13:02:29
tags: Android
categories: Android
---


##### 何为Accessibility机制
对于那些由于视力、听力或其它身体原因导致不能方便使用Android智能手机的用户，Android提供了Accessibility功能和服务帮助这些用户更加简单地操作设备，包括文字转语音、触觉反馈、手势操作、轨迹球和手柄操作。开发者可以搭建自己的Accessibility服务，这可以加强应用的可用性，例如声音提示，物理反馈，和其他可选的操作模式。
随着Android系统版本的迭代，Accessibility功能也越来越强大。**AccessibilityService运行在后台, 它能实时地获取当前操作应用的窗口元素信息，也能对窗口元素进行操作并双向交互;能获取用户的输入，比如点击按钮;接收焦点改变等等**
**使用条件：**
- 需要经过用户授权. 设置——辅助功能找到对应的应用名称,然后进行开启
- 此机制是免Root的,并且需要API14以上

常用的方法:

方法 | 作用
-|-
disableSelf()	| 禁用当前服务,也就是在服务可以通过该方法停止运行	|
findFoucs(int falg)	| 查找拥有特定焦点类型的控件	|
getRootInActiveWindow()	| 如果配置能够获取窗口内容,则会返回当前活动窗口的根结点	|
getSeviceInfo()	| 获取当前服务的配置信息	|
onAccessibilityEvent(AccessibilityEvent event)	| 有关AccessibilityEvent事件的回调函数.系统通过sendAccessibiliyEvent()不断的发送AccessibilityEvent到此处	|
performGlobalAction(int action)	| 执行全局操作,比如返回,回到主页,打开最近等操作	|
setServiceInfo(AccessibilityServiceInfo info)	| 设置当前服务的配置信息	|
getSystemService(String name)	| 获取系统服务	|
onKeyEvent(KeyEvent event)	| 如果允许服务监听按键操作,该方法是按键事件的回调,需要注意,这个过程发生了系统处理按键事件之前	|
onServiceConnected()	| 系统成功绑定该服务时被触发,也就是当你在设置中开启相应的服务,系统成功的绑定了该服务时会触发,通常我们可以在这里做一些初始化操作	|
onInterrupt() | 服务中断时的回调 |

##### 示例
1. 继承AccessibilityService类
```
package krelve.demo.rob;

import java.sql.Date;
import java.text.SimpleDateFormat;
import java.util.List;

import com.example.bean.DBInfo;
import com.example.db.DatabaseUtil;
import com.example.dbdemo.Add_Date;

import android.accessibilityservice.AccessibilityService;
import android.annotation.SuppressLint;
import android.content.Intent;
import android.provider.Settings;
import android.util.Log;
import android.view.accessibility.AccessibilityEvent;
import android.view.accessibility.AccessibilityNodeInfo;

public class RobMoney extends AccessibilityService {

    private DatabaseUtil mDBUtil;

    @Override
    protected void onServiceConnected() {
        super.onServiceConnected();

    }

    @SuppressLint("NewApi")
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {

             String strClickData = event.getText().get(0).toString();
             System.out.println("输入内容" + strClickData);
             AccessibilityNodeInfo rootNode = getRootInActiveWindow();
             // findEditText(rootNode);
             // 获取包名
             String str_package = rootNode.getPackageName().toString();
             String app_name = StringUtil.pkgTransform(RobMoney.this, str_package);  
    }

}
```

2. AndroidManifest.xml配置Service
```
<service
            android:name="krelve.demo.rob.RobMoney"
            android:enabled="true"
            android:exported="true"
            android:label="@string/app_name"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE" >
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>

            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility" />
        </service>
```
其中，<action android:name="android.accessibilityservice.AccessibilityService" />是需要申请的权限

3. **核心配置** xml/accessibility是做了初始化的工作，具体实现如下：
```
<?xml version="1.0" encoding="utf-8"?>
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeViewFocused|typeViewTextChanged"
    android:accessibilityFeedbackType="feedbackVisual"
    android:canRetrieveWindowContent="true"
    android:description="@string/stenographer_service_description"
    android:notificationTimeout="100" />
```
这里我们看到有很多选项,我们看一下常用的几个属性：

- android:accessibilityEventTypes="xxxxxxxx"  
设置响应事件的类型, 属性如下：
typeAllMask：接收所有事件。
-------窗口事件相关（常用）---------
typeWindowStateChanged：监听窗口状态变化，比如打开一个popupWindow，dialog，Activity切换等等。
typeWindowContentChanged：监听窗口内容改变，比如根布局子view的变化。
typeWindowsChanged：监听屏幕上显示的系统窗口中的事件更改。 此事件类型只应由系统分派。
typeNotificationStateChanged：监听通知变化，比如notifacation和toast。
-----------View事件相关--------------
typeViewClicked：监听view点击事件。
typeViewLongClicked：监听view长按事件。
typeViewFocused：监听view焦点事件。
typeViewSelected：监听AdapterView中的上下文选择事件。
typeViewTextChanged：监听EditText的文本改变事件。
typeViewHoverEnter、typeViewHoverExit：监听view的视图悬停进入和退出事件。
typeViewScrolled：监听view滚动，此类事件通常不直接发送。
typeViewTextSelectionChanged：监听EditText选择改变事件。
typeViewAccessibilityFocused：监听view获得可访问性焦点事件。
typeViewAccessibilityFocusCleared：监听view清除可访问性焦点事件。
------------手势事件相关---------------
typeGestureDetectionStart、typeGestureDetectionEnd：监听手势开始和结束事件。
typeTouchInteractionStart、typeTouchInteractionEnd：监听用户触摸屏幕事件的开始和结束。
typeTouchExplorationGestureStart、typeTouchExplorationGestureEnd：监听触摸探索手势的开始和结束。

- android:accessibilityFeedbackType="xxxxxxxx"
设置回馈给用户的方式，有语音播出和振动。可以配置一些TTS引擎，让它实现发音。属性如下：
feedbackAllMask、feedbackGeneric、feedbackAudible、feedbackSpoken、feedbackHaptic、feedbackVisual

- canRetrieveWindowContent
是否希望能够检索活动窗口内容。此设置无法在运行时更改。属性如下：
true or false

- android:notificationTimeout="100"
两个相同类型的可访问性事件之间的最短间隔时间（以毫秒为单位）

- canPerformGestures 是否可以执行手势（api 24新增）属性如下：
true or false

- android:packageNames="com.xxx.xxx"
可以指定响应某个应用的事件，这里因为要响应所有应用的事件，所以不填，默认就是响应所有应用的事件。比如我们写一个微信抢红包的辅助程序，就可以在这里填写微信的包名，便可以监听微信产生的事件了。我们这些配置信息除了在xml中定义，同样也可以在代码中定义，我们一般都是在onServiceConnected()方法里进行

另外也可使用代码配置
```
override fun onServiceConnected() {
    val serviceInfo = AccessibilityServiceInfo().apply {
        eventTypes = AccessibilityEvent.TYPES_ALL_MASK
        feedbackType = AccessibilityServiceInfo.FEEDBACK_ALL_MASK
        packageNames = arrayOf("com.eg.android.AlipayGphone")//支付宝包名，可以多个
        notificationTimeout = 10
    }
    setServiceInfo(serviceInfo)
}
```

4. 指引用户去手动打开该服务
首先判断该服务是否为开启状态：
```
public static boolean isAccessibilitySettingsOn(Context mContext, Class<? extends AccessibilityService> clazz) {
    int accessibilityEnabled = 0;
    final String service = mContext.getPackageName() + "/" + clazz.getCanonicalName();
    try {
        accessibilityEnabled = Settings.Secure.getInt(mContext.getApplicationContext().getContentResolver(),
                Settings.Secure.ACCESSIBILITY_ENABLED);
    } catch (Settings.SettingNotFoundException e) {
        e.printStackTrace();
    }
    TextUtils.SimpleStringSplitter mStringColonSplitter = new TextUtils.SimpleStringSplitter(':');
    if (accessibilityEnabled == 1) {
        String settingValue = Settings.Secure.getString(mContext.getApplicationContext().getContentResolver(),
                Settings.Secure.ENABLED_ACCESSIBILITY_SERVICES);
        if (settingValue != null) {
            mStringColonSplitter.setString(settingValue);
            while (mStringColonSplitter.hasNext()) {
                String accessibilityService = mStringColonSplitter.next();
                if (accessibilityService.equalsIgnoreCase(service)) {
                    return true;
                }
            }
        }
    }
    return false;
}
```

没有开启的话则跳转到该服务的开启页面，由用户手动开启
```
startActivity(Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS))
```


##### 检索窗口内容（获取节点）
允许服务检索窗口内容
```
android:canRetrieveWindowContent="true"
```
通过getWindows获取屏幕上的可以看见的窗口，它返回一个列表，按降序的方式，排在第一个的就是最顶层的窗口。
我们可以通过以下方法来检索当前活动窗口其中的控件
```
val root = rootInActiveWindow //获取当前活动窗口的根节点
val node = root.findAccessibilityNodeInfosByViewId("com.alipay.mobile.payee:id/payee_NextBtn") //通过控件id来获取某个控件
val node = root.findAccessibilityNodeInfosByText("确定") //通过text来获取某个控件
val node = root.findFoucs(int falg) //寻找拥有特殊焦点的控件（FOCUS_INPUT 或 FOCUS_ACCESSIBILITY）
```

viewId的获取，可以通过android Device Monitor工具来查看，3.0之后的android studio，可通过命令行进入sdk的tools目录，运行下面命令：
```
monitor
```

##### 事件交互
拿到AccessibilityNodeInfo对象后，我们可以进行一些列的操作，包括getChild()、getParent()、getBoundsInScreen()、isClickable等等一些列获取属性的操作，当然也可以进行交互性的操作，比如点击（当然前提是这个控件的clickable为true）：
```
node[0].performAction(AccessibilityNodeInfo.ACTION_CLICK)
```
除了操作界面内控件之外，我们还可以通过performGlobalAction(int action)执行一些全局操作，比如点击back键、home键等等。
```
performGlobalAction(GLOBAL_ACTION_BACK)
performGlobalAction(GLOBAL_ACTION_HOME)
performGlobalAction(GLOBAL_ACTION_NOTIFICATIONS)
performGlobalAction(GLOBAL_ACTION_RECENTS)
```

##### 手势交互
除了Action交互之外，我们还可以模拟人的手势进行操作，这是在Android 24中新加的一个api：
```
dispatchGesture(gesture, callback, handler)
```
它接收一个GestureDescription（手势描述）、一个GestureResultCallback（结果回调）和一个Handler。简单封装一下大概是这样的：
```
/**
 * 通过AccessibilityService在屏幕上模拟手势
 * @param path 手势路径
 */
@RequiresApi(Build.VERSION_CODES.N)
fun AccessibilityService.gestureOnScreen(
        path: Path,
        startTime:Long = 0,
        duration:Long = 100,
        callback:AccessibilityService.GestureResultCallback,
        handler: Handler? = null
){
    val builder = GestureDescription.Builder()
    builder.addStroke(GestureDescription.StrokeDescription(path, startTime, duration))
    val gesture = builder.build()
    dispatchGesture(gesture, callback, handler)
}

/**
 * 通过AccessibilityService在屏幕上某个位置单击
 */
@RequiresApi(Build.VERSION_CODES.N)
fun AccessibilityService.clickOnScreen(
        x:Float,
        y:Float,
        callback:AccessibilityService.GestureResultCallback,
        handler: Handler? = null
){
    val p = Path()
    p.moveTo(x,y)
    gestureOnScreen(p,callback = callback,handler = handler)
}
```

————————————————
整理修改自：
https://blog.csdn.net/chaozhung_no_l/article/details/53556100
https://www.jianshu.com/p/7b91e3702328
