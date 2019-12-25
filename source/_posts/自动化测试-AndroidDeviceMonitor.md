---
title: 自动化测试-AndroidDeviceMonitor
date: 2019.12.16 13:51:59
tags: Android
categories: Android
---

AndroidDeviceMonitor是一个独立的工具,可以对Android应用进行调试和分析。在各种主流IDE上都有其集成, Intellij家族在Tools-Android-Android Device Monitor打开。
![Android Device Monitor](/images/自动化测试-AndroidDeviceMonitor/ADM.png)

Device界面有以下小工具：

##### 1. Debug：Debug工具, 即一般是用的Debug模式。

##### 2. Heap
-Updated Heap：对指定进程执行栈观测。在每次调用GC后更新，Cause GC为强制GC。接着在DDMS的 **Heap板块** 性能分析。此工具与Android Studio自带的Dump Java Heap工具基本无二，只不过AS界面更酷炫一点。
![Updated Heap](/images/自动化测试-AndroidDeviceMonitor/ADM1.png)
一般情况下通过不断在手机上操作一个功能，观察GC后的TotalSize。如果该数据在不断地增加，则可判定该功能模块存在内存泄露问题。

-Dump HPROF file：导出hprof文件，将hprof文件拖动到编写代码的窗口，就出现下面的显示信息。(或可从View—Tool Windows—Captures打开)。
![Dump HPROF file](/images/自动化测试-AndroidDeviceMonitor/ADM2.png)
**Hprof Viewer主要分ABC三大区域：**
区域A：这个应用中所有类的名字。
区域B：左边类的所有实例。
区域C：在选择B中的实例后，这个实例的引用树。

**上图中A区域：**

左上角是Hprof Viewer查看方式的可选列表，分别是用来选择Heap区域和Class View的展示方式的。

Heap类型分为：
App Heap – 当前App使用的Heap。
Image Heap – 磁盘上当前App的内存映射拷贝。
Zygote Heap – Zygote进程Heap（每个App进程都是从Zygote孵化出来的，这部分基本是Framework中通用类的Heap）。

Class View类型分为：
Class List View – 类列表方式。
Package Tree View – 根据包结构的树状显示。

A区域左上角列名解释

Class Name 类名，Heap中的所有Class
Total Count 内存中该类这个对象总共的数量，有的在栈中，有的在堆中
Heap Count 堆内存中这个类对象的个数
Sizeof 每个该实例占用的内存大小
Shallow Size 所有该类的实例占用的内存大小
Retained Size 所有该类对象被释放掉，会释放多少内存

**B区域右上角列名解释**

Instance 该类的实例
Depth 深度，从任一GC Root点到该实例的最短跳数
Dominating Size 该实例可支配的内存大小

**C区域描述的是B中实例具体被引用的信息**

B区域右上角有个“三角”的按钮，点击会进入Hprof Analyzer的hprof分析界面，在这个界面中可以直接将可能导致内存泄露的类找出来。
![Dump HPROF file](/images/自动化测试-AndroidDeviceMonitor/ADM3.png)
我们来分析一下MainActivity的泄露情况：

一个Activity应该只有一个实例，但是从A区域来看Total Count的值为2，Heap Count的值也为2，这说明有一个实例是多余的。
在B区域中可以看到两个MainActivity的实例，点击一个查看它的引用树情况。
在C区域中可以看到MainActivity的实例Context被UserInfoManage的instance引用了，引用深度为1。
在Analyzer Tasks区域中可以看到Leaked Activities中包含了MainActivity。
以上分析的结果表明MainActivity发生了内存泄露。

##### 3. Profiling
-Udate Threads 可在DDMS的 **Thread板块** 查看断点的thread信息。
![TraceView](/images/自动化测试-AndroidDeviceMonitor/ADM6.png)
看看每个字段的含义：
(1) ID:虚拟机分配的唯一的线程ID,有的ID左上方有一个"\*"号，感觉像是标示这些事系统创建的线程，其他的是用户创建的。
(2)Tid:Tid：Linux的线程ID号
(3)Status:线程状态,有如下一些状态。
    runnable:  可执行，可能正在运行，也可能没有运行
    sleeping：执行了Thread.sleep()
    monitor：等待接受一个监听锁。
    wait:：Object.wait()，等待被其他线程唤醒
    native：正在执行native代码，
    vmwait：等待虚拟机
    zombie：线程在垂死的进程
    init：线程在初始化
    starting：线程正在启动
(4)utime：执行用户代码的累计时间
(5)stime：执行系统代码的累计时间
(6)Name：线程的名字, 创建thread的时候，传入的name就是现实在这里。根据不同名字，也可以看出线程的来源
  1. main
   这个就是主线程了。具体流程待细述。
  2. HeapWorker
   一个异步的工作线程，处理那些需要在单独线程里面做的避免同步问题的堆操作。其源代码在     dalvik/vm/alloc/HeapWorker.\*部分。
  3. Signal Catcher
    这个线程是用来捕获linux信号和做一些后续处理的。比如说，当一个SIGQUIT (Ctrl-\)信号到     达后，这个线程就会挂起虚拟机，并且将所有线程的状态信息输出到log。其源代码在              dalvik/vm/SignalCatcher.\*部分。
  4. JDWP
  这个线程是用来实现Java Debug Wire Protocol的。如果命令行调试器的参数为"suspend=y"，  这样会暂停虚拟机。这个估计和eclipse的调试和ddms等调试工具相关。其源代码在  dalvik/vm/jdwp/\*部分。
  5. Stdio Converter
  这个线程从标准输出和标准错误输出读取信息并将它们转换为log信息。其源代码在  dalvik/vm/StdioConverter.\*部分。
  6. Compiler
  Android's Jit独立于目标平台的部分。其源代码在dalvik/vm/compiler/Compiler.*
dalvik/vm/interp/Jit.\*等部分。
  7. Binder Thread
  使用binder进行通讯时用到的线程。其源代码在frameworks/base/libs/binder/\*等部分。
  以下的线程属于system_server和应用程序专有线程，视具体应用的需求而定。
  8. system_server专有
  android.server.ServerThread
  ActivityManager
  ProcessStats
  PackageManager
  FileObserver
  AccountManagerService
  SyncHandlerThread
  UEventObserver
  PowerManagerService
  AlarmManager
  WindowManager
  InputDeviceReader
  WindowManagerPolicy
  InputDispatcher
  ConnectivityThread
  WifiService
  WifiWatchdogThread
  LocationManagerService
  AudioService
  GpsEventThread
  GpsNetworkThread
  android.hardware.SensorManager

-Start Method Profiling
内置的TraceView工具，它可以加载 trace 文件，用图形的形式展示代码的执行时间、次数及调用栈。
方式一，点击"Start Method Profiling"按钮(等红色小点变成黑色以后就表示TraceView已经开始工作了)，此时进行小范围性能测试，操作最好不要超过5s。然后再按一下刚才按的按钮结束操作。就出现下面的显示信息。
方式二，使用android.os.Debug.startMethodTracing()和android.os.Debug.stopMethodTracing()方法，将生成一个trace文件在/sdcard目录。就出现下面的显示信息。
![TraceView](/images/自动化测试-AndroidDeviceMonitor/ADM4.png)
1、Sample based profiling以固定的频率像VM发送中断并搜集调用栈信息。低版本手机也是采用该方式来采集样本的默认是1毫秒采集一次。精确度和采集的频率有关间隔频率越小会越精确但运行也会相应的更慢。一般使用默认模式。
2、Trace based profiling不论多小的函数都会跟踪整个函数的执行过程所以开销也会很大。运行起来会非常的慢不适合检测滑动性能。

![TraceView](/images/自动化测试-AndroidDeviceMonitor/ADM5.png)
上部分是每条线程的执行情况和测试时间段内所涉及的函数调用信息，下部分是线程执行调用的函数占用资源信息。
常用字段的属性说明如下：
列名                           描述
Name                    该线程运行过程中所调用的函数名
Incle Cpu Time          某函数占用的CPU时间，包含内部调用其他函数的CPU时间
Excl Cpu  Time          某函数占用的CPU时间，但不含内部调用其他函数所占用的CPU时间
Incl Real Time          某函数运行的真实时间，含调用其他函数所占用的真实时间
Excl Real Time          某函数运行的真实时间，不含调用其他函数所占用的真实时间
Call +Recur Calls/Total 某函数被调用次数以及递归调用占总调用次数的百分比
Cpu Time/Call           某函数调用CPU时间与调用次数的比，相当于该函数平均执行时间
Real Time/Call          某函数调用CPU的真实时间；

##### 4. Screen Capture
截图工具

##### 5. Dump View Hierarchy for UI AutoMator
输出UI AutoMator用于布局分析。

##### 6. Capture system wide trace using Android Systrace
属于Android提供的Systrace工具，支持版本Android 4.1以上。一般可用于在解决应用程序中与性能相关的错误（如启动缓慢、转换缓慢或UI jank）。Systrace的原理是在系统的一些关键链路插入一些 **标记** ，通过这些标记收集系统关键路径的运行时间信息用于性能分析。
命令行操作: 使用SDK目录platform-tools/systrace/systrace.py。
![Capture](/images/自动化测试-AndroidDeviceMonitor/ADM7.png)

导出trace.html静态页面用浏览器打开分析。左边呈现是UI Frames的每个进程，选择你要观察的进程，右边是沿时间线指示呈现每个渲染的Frame。绿色圆表示： 16.6毫秒内渲染完成的Frames。黄色或红色圆表示： 渲染时间超过16.6毫秒的Frames。
![Capture](/images/自动化测试-AndroidDeviceMonitor/ADM8.png)
单击一个frame 圆将显示系统渲染该frame所做的相关信息,包括系统在呈现该帧时正在执行的方法(可调查导致ui-jank的方法)和系统提醒Alert。
屏幕右边的标签Alerts的集合，可查看调用次数和系统提醒信息
![Capture](/images/自动化测试-AndroidDeviceMonitor/ADM9.png)

 **定义自定义事件**
(1)在Android 4.3（API级别18）及更高版本中，使用代码中的Trace类在HTML报告中标记执行事件。需要使用-a或--app命令行选项运行Systrace，并指定应用程序的包名称。
```
python systrace.py -a com.lqr.wechat -b 16384 -o my_systrace_report.html sched freq idle am wm gfx view binder_driver hal dalvik camera input res
```
    下面的代码示例演示如何使用Trace类来标记方法的执行，包括该方法中的两个嵌套代码块；多次调用BeginAtomon（String）时，调用EndSection（）只结束最近调用的BeginAtomon（String）方法。因此，对于嵌套调用（如下面示例中的调用），需要确保每个对BeginAtomion（）的调用都与对EndSection（）的调用正确匹配。此外，不能在一个线程上调用BeginAtomon（）并从另一个线程结束它，必须从同一个线程调用EndSection（）。
```
public  class  MyAdapter  extends  RecyclerView.Adapter<MyViewHolder>  {  
    ...
    @Override  
    public  MyViewHolder onCreateViewHolder(ViewGroup parent,  int viewType)  {  
        Trace.beginSection("MyAdapter.onCreateViewHolder");  
        MyViewHolder myViewHolder;  
        try  {
           myViewHolder =  MyViewHolder.newInstance(parent);
        }  finally  {  
            // In 'try...catch' statements, always call `[endSection()](https://developer.android.google.cn/reference/android/os/Trace.html#endSection())`  
            // in a 'finally' block to ensure it is invoked even when an exception  // is thrown.  
            Trace.endSection();
        }  
        return myViewHolder;  
    }

    @Override  
    public  void onBindViewHolder(MyViewHolder holder,  int position)  {
        Trace.beginSection("MyAdapter.onBindViewHolder");  
        try  {  
            try  {  
                Trace.beginSection("MyAdapter.queryDatabase");  
                RowItem rowItem = queryDatabase(position);
                dataset.add(rowItem);  
            }  finally  {  
                Trace.endSection();
            }
            holder.bind(dataset.get(position));
        }  finally  {  
            Trace.endSection();  
        }
    }
    ...
 }
 ```

(2)Android 6.0（API级别23）及更高版本支持调用native层tracing
API 通过引入trace.h，将trace事件写入系统缓冲区，然后使用Systrace进行分析。Android 6.0到4.3可以尝试通过JNI调用。

为ATrace functions定义方法指针
```
#include <android/trace.h>
#include <dlfcn.h>

void *(*ATrace_beginSection) (const char* sectionName);
void *(*ATrace_endSection) (void);

typedef void *(*fp_ATrace_beginSection) (const char* sectionName);
typedef void *(*fp_ATrace_endSection) (void);
```
在运行时加载ATrace符号，通常在对象构造函数中执行这个过程,出于安全原因，仅在应用程序或游戏的调试版本中包含对dlopen（）的调用。
```
// Retrieve a handle to libandroid.
void *lib = dlopen("libandroid.so", RTLD_NOW || RTLD_LOCAL);

// Access the native tracing functions.
if (lib != NULL) {
   // Use dlsym() to prevent crashes on devices running Android 5.1
   // (API level 22) or lower.
    ATrace_beginSection = reinterpret_cast<fp_ATrace_beginSection>(dlsym(lib, "ATrace_beginSection"));
    ATrace_endSEction = reinterpret_cast<fp_ATrace_endSection>(dlsym(lib, "ATrace_endSection"));
}
```
在自定义事件的开始和结束调用atrace_begindomon() 和atrace_endsection()
```
#include <android/trace.h>

char *customEventName = new char[32];
sprintf(customEventName, "User tapped %s button", buttonName);

ATrace_beginSection(customEventName);
// Your app or game's response to the button being pressed.
ATrace_endSection();
```

##### 7. Start OpenGL Trace
可逐帧（实为逐函数）记录app使用OpenGL ES的绘制过程。提供OPENGL函数的消耗时长，可用于渲染性能分析。Trace开始跟踪，Stop Tracing结束跟踪。
![OpenGL](/images/自动化测试-AndroidDeviceMonitor/ADM10.png)
对生成的Trace log文件进行分析。
![OpenGL](/images/自动化测试-AndroidDeviceMonitor/ADM11.png)
左上角滚动条用于调节查看哪一帧的绘制过程。下面显示当前帧的所有绘制函数，绘制函数会标蓝显示。点击函数，能显示该函数执行前后的效果。





版权声明：本文部分转载自CSDN博主「w一花一世界w」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/my_rabbit/article/details/79831911
版权声明：本文部分转载自CSDN博主「六公子」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/ilove027/article/details/50295357
参考链接：https://www.jianshu.com/p/8fa300effa59, https://blog.csdn.net/google_huchun/article/details/79281915
Systrace参考链接：https://www.jianshu.com/p/73a186ff2f22
Traceview 参考链接：https://yq.aliyun.com/articles/20467
