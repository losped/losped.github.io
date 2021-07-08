---
title: Android-Framework层-总成（1）
date: 2020.09.15 10:37:39
tags: Android
categories: Android
---

##### 背景
之前framework层源码开了个小坑还没怎么去填它，现在定个目标有空的话要每天学习一点。一直点方法容易云里雾里，碰到未解明模块先了解功能后标记，下次学到再细看。自己总结了以下三个步骤，减少一些阻力
1.扫一遍先看接口结构-方法功能-实现类
2.先看视频或文章解析√
3.按目的性去读 比如: Act生命周期

![Android架构图](/images/Android-Framework层-总成（1）/Android架构图.jpg)

##### Frameworks目录解释
![Frameworks目录解释1](/images/Android-Framework层-总成（1）/Frameworks目录解释1.png)
![Frameworks目录解释2](/images/Android-Framework层-总成（1）/Frameworks目录解释2.png)


##### Binder结构
Frameworks源码模块间常用到多进程通信，一般使用Binder通信。结构与使用AIDL时系统生成的java文件实现相同, 以ApplicationThread为例,架构如下:

AIDL文件结构
```
Interface接口
{
抽象类Stub 继承Binder  实现Interface{
静态的asInterface方法{
       返回代理类（分两种情况：同进程，直接返回继承该Stub的Binder（）；不同进程，返回静态代理类）
      }
重写的onTransact方法
静态代理类Proxy  实现Interface{
    实现接口方法，主动调用transact方法
    }
  }
声明接口方法
}
```


IInterface.java
```
/**
 * Base class for Binder interfaces.  When defining a new interface,
 * you must derive it from IInterface.
 */
public interface IInterface
{
    /**
     * Retrieve the Binder object associated with this interface.
     * You must use this instead of a plain cast, so that proxy objects
     * can return the correct result.
     */
    public IBinder asBinder();
}
```

IApplicationThread.java
```
public interface IApplicationThread extends IInterface {
  int SCHEDULE_PAUSE_ACTIVITY_TRANSACTION = IBinder.FIRST_CALL_TRANSACTION;
...
}
```

ApplicationThreadNative.java
```
// L: 这里实际与使用AIDL系统生成的文件原理相同, ApplicationThreadNative等同Stub,ApplicationThreadProxy等同Proxy
public abstract class ApplicationThreadNative extends Binder
        implements IApplicationThread {
    /**
     * Cast a Binder object into an application thread interface, generating
     * a proxy if needed.
     */
     // L: 判断远程IBinder进程与当前进程是否相同
     // 相同则返回ApplicationThreadNative
     // 不同则返回ApplicationThreadProxy
    static public IApplicationThread asInterface(IBinder obj) {
        if (obj == null) {
            return null;
        }
        IApplicationThread in =
            (IApplicationThread)obj.queryLocalInterface(descriptor);
        if (in != null) {
            return in;
        }

        return new ApplicationThreadProxy(obj);
    }

    public ApplicationThreadNative() {
        attachInterface(this, descriptor);
    }

    // L: 不同进程间client调用Proxy中IBinder的transact方法向服务端发送消息
    // 服务端 执行Binder对象中的onTransact()函数
    @Override
    public boolean onTransact(int code, Parcel data, Parcel reply, int flags)
            throws RemoteException {
        switch (code) {
          case SCHEDULE_PAUSE_ACTIVITY_TRANSACTION:
          { do something }
          ...
        }

        return super.onTransact(code, data, reply, flags);
    }

    public IBinder asBinder()
    {
        return this;
    }
}

// ApplicationThread代理类,持有ApplicationThreadNative的引用
class ApplicationThreadProxy implements IApplicationThread {
    private final IBinder mRemote;

    public ApplicationThreadProxy(IBinder remote) {
        mRemote = remote;
    }

    public final IBinder asBinder() {
        return mRemote;
    }

    public final void schedulePauseActivity(IBinder token, boolean finished,
            boolean userLeaving, int configChanges, boolean dontReport) throws RemoteException {
        Parcel data = Parcel.obtain();
        data.writeInterfaceToken(IApplicationThread.descriptor);
        data.writeStrongBinder(token);
        data.writeInt(finished ? 1 : 0);
        data.writeInt(userLeaving ? 1 :0);
        data.writeInt(configChanges);
        data.writeInt(dontReport ? 1 : 0);
        // 调用 transact() 发送消息给服务端
        mRemote.transact(SCHEDULE_PAUSE_ACTIVITY_TRANSACTION, data, null,
                IBinder.FLAG_ONEWAY);
        data.recycle();
    }
    ...
  }
```

ActivityStack.java
```
final boolean startPausingLocked(boolean userLeaving, boolean uiSleeping,
                                 ActivityRecord resuming, boolean dontWait) {
    ...
    ActivityRecord prev = mResumedActivity;
    // 不同进程间,调用Proxy中的schedulePauseActivity
    prev.app.thread.schedulePauseActivity(prev.appToken, prev.finishing,
            userLeaving, prev.configChangeFlags, dontWait);
}
```

参考文章：
[https://blog.csdn.net/love_xsq/article/details/50992515](https://blog.csdn.net/love_xsq/article/details/50992515)
[https://www.jianshu.com/p/e97dd804d5fe](https://www.jianshu.com/p/e97dd804d5fe)
[https://www.jianshu.com/p/17525199d462](https://www.jianshu.com/p/17525199d462)
