---
title: MIUI12.5 Toast问题
date: 2021.04.25 16:27:37
tags: Android
categories: Android
---

### 题记
前几天小米推送了新版本系统MIUI12.5，在Toast显示上出现了一个奇怪的错误。问题背景：连续调用系统相机拍照，拍一张传一张。每次拍完发送一个Toast消息"上传照片： XXX"；当上传完毕通过handler发送一个Toast状态消息"xxx"。
期间有跳转到系统相机Activity的操作，错误恰恰出现在跳转回应用Activity的操作后。下面先贴出相关代码，再从相关知识和源码入手，来找出原因。

```
// code DataEntry.java
	protected void onActivityResult(int requestCode, int resultCode, Intent data) {
		if (RESULT_OK == resultCode) {
	        switch (requestCode) {
                // PhotosView拍照(调用系统)
                case REQUEST_CODE_MULTIPLE_CAMERA:
                    if(photosView != null){
                        getPrompt().showToast("上传照片： " + upFileName, Toast.LENGTH_LONG); //第一个Toast
                        prompt.uploadBitmap(map, upFileName + ".jpg", bitmap,
                        null, new Prompt.WebsiteCallback() {
                            @Override
                            public void visitWebsite(String text) {
                                if (text != null) {
                                    ...
                                    prompt.showToast(sb.toString(), Toast.LENGTH_LONG); //第二个Toast，上传完毕后通过handler发送
                                } else {
                                    prompt.showToast("网络错误，请重试", Toast.LENGTH_LONG);
                                }
                            }
                        }, phoNum * 1000);
                    }
                    else {
                        getPrompt().showToast("题目错误：" + photosView);
                    }
                    break;
	        }
		}
        ...
		super.onActivityResult(requestCode, resultCode, data);
	}
```

```
// code Prompt.java

private Toast toast;

    public Prompt(MyApplication application) {
        this.application = application;
        toast = Toast.makeText(application, "", Toast.LENGTH_SHORT);
        toast.setGravity(Gravity.TOP | Gravity.CENTER, 0, screenHeight / 4);
    }

    public void showToast(String message, int duration) {
        toast.setDuration(duration);
        View view1 = toast.getView();
        toast.setText(message);
        View view2 = toast.getView();
        toast.show();
    }
```

```
// 错误提示
java.lang.IllegalArgumentException: View=android.widget.LinearLayout{61841dd V.E...... ......ID 0,42-605,358} not attached to window manager
	at android.view.WindowManagerGlobal.findViewLocked(WindowManagerGlobal.java:572)
	at android.view.WindowManagerGlobal.removeView(WindowManagerGlobal.java:476)
	at android.view.WindowManagerImpl.removeView(WindowManagerImpl.java:139)
	at android.widget.ToastPresenter.show(ToastPresenter.java:207)
	at android.widget.Toast$TN.handleShow(Toast.java:810)
	at android.widget.Toast$TN$1.handleMessage(Toast.java:742)
	at android.os.Handler.dispatchMessage(Handler.java:106)
	at android.os.Looper.loop(Looper.java:236)
	at android.app.ActivityThread.main(ActivityThread.java:8067)
	at java.lang.reflect.Method.invoke(Native Method)
	at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:656)
	at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:967)
```

### Window结构
先复习下Windows的结构，再来做分析。
##### 1.Window、PhoneWindow、DecorView 之间的关系
所有 Android 进程的视图显示都需要依赖于一个窗口Window。每一个 Activity/Dialog/Toast 持有一个 PhoneWindow 的对象，而一个 PhoneWindow 对象持有一个 DecorView 的实例，
所以视图中 View 相关的操作其实大都是通过 DecorView 来完成。
![关系图](/images/MIUI12.5 Toast问题/1.jpeg)

##### 2.关于ViewRootImpl
ViewRootImpl是View中的最高层级，属于所有View的根（但ViewRootImpl不是View，只是实现了ViewParent接口），实现了View和WindowManager之间的通信协议，实现的具体细节在WindowManagerGlobal这个类当中。
是Window和View沟通的桥梁，由WindowManagerGlobal创建；而 ViewRootImpl 就是对 DecorView 进行操作的，包括onMeasure(),onLayout(),onDraw()的调用。
- 关于ViewRootImpl的设计：因为对于Activity来说是需要用 DecorView 管理View的，而对于系统级别比如Toast来说也需要用 WindowManager 自身管理View；而每个窗口是与Context绑定的，如果都使用DecorView，那么所有View的生命周期都只能是同步的了。因此需要
ViewRootImpl做View根进行统一管理。
![关系图](/images/MIUI12.5 Toast问题/2.jpeg)

##### 3.关于WindowManager
WindowManager继承ViewManger，用于对View进行管理，包含添加、更新、删除View等方法。WindowManagerImpl为WindowManager的实现类。WindowManagerImpl内部方法实现都是由代理类WindowManagerGlobal完成，
而WindowManagerGlobal是一个单例，也就是一个进程中只有一个WindowManagerGlobal对象服务于所有页面的View。

```
public final class WindowManagerImpl implements WindowManager {
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mDisplay, mParentWindow);
    }

    @Override
    public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.updateViewLayout(view, params);
    }

    @Override
    public void removeView(View view) {
        mGlobal.removeView(view, false);
    }
    ...
}
```

### Toast源码

##### Toast的初始化

```
// 源码版本 SDK 30
public class Toast {

    /**
     * Make a standard toast that just contains text.
     *
     * @param context  The context to use.  Usually your {@link android.app.Application}
     *                 or {@link android.app.Activity} object.
     * @param text     The text to show.  Can be formatted text.
     * @param duration How long to display the message.  Either {@link #LENGTH_SHORT} or
     *                 {@link #LENGTH_LONG}
     *
     */
    public static Toast makeText(Context context, CharSequence text, @Duration int duration) {
        return makeText(context, null, text, duration);
    }

    /**
     * Make a standard toast to display using the specified looper.
     * If looper is null, Looper.myLooper() is used.
     *
     * @hide
     */
    public static Toast makeText(@NonNull Context context, @Nullable Looper looper,
            @NonNull CharSequence text, @Duration int duration) {
        // 如果是系统应用
        if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
            Toast result = new Toast(context, looper);
            result.mText = text;
            result.mDuration = duration;
            return result;
        } else {
            Toast result = new Toast(context, looper);
            // SDK30 使用了MVP的模式，将耦合业务交给Presenter
            View v = ToastPresenter.getTextToastView(context, text);
            result.mNextView = v;
            result.mDuration = duration;
            return result;
        }
    }
}
```

```
// 源码版本 SDK 30
public class ToastPresenter {
    
    public static View getTextToastView(Context context, CharSequence text) {
        View view = LayoutInflater.from(context).inflate(TEXT_TOAST_LAYOUT, null);
        TextView textView = view.findViewById(com.android.internal.R.id.message);
        textView.setText(text);
        return view;
    }
}
```
主要进行mNextView和mDuration的初始化

##### Toast的展示

将 Toast 内部的 TN ( ITransientNotification 客户端对象)加入到 INotificationManager 服务端的 Binder 实现的 mToastQueue 队列中。再由服务端遍历处理 mToastQueue 队列中ToastRecord对象
```
public class Toast {
    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P)
    private static INotificationManager sService;

    @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P)
    static private INotificationManager getService() {
        if (sService != null) {
            return sService;
        }
        // 获取INotificationManager的客户端的Binder对象
        sService = INotificationManager.Stub.asInterface(
                ServiceManager.getService(Context.NOTIFICATION_SERVICE));
        return sService;
    }

    /**
     * Show the view for the specified duration.
     */
    public void show() {
        if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
            checkState(mNextView != null || mText != null, "You must either set a text or a view");
        } else {
            if (mNextView == null) {
                throw new RuntimeException("setView must have been called");
            }
        }

        // 初始化Service
        INotificationManager service = getService();
        // 获取包名
        String pkg = mContext.getOpPackageName();
        TN tn = mTN;
        tn.mNextView = mNextView;
        final int displayId = mContext.getDisplayId();

        try {
            if (Compatibility.isChangeEnabled(CHANGE_TEXT_TOASTS_IN_THE_SYSTEM)) {
                if (mNextView != null) {
                    // It's a custom toast
                    service.enqueueToast(pkg, mToken, tn, mDuration, displayId);
                } else {
                    // It's a text toast
                    ITransientNotificationCallback callback =
                            new CallbackBinder(mCallbacks, mHandler);
                    service.enqueueTextToast(pkg, mToken, mText, mDuration, displayId, callback);
                }
            } else {
                // 将TN加入INotificationManager中的mToastQueue队列
                service.enqueueToast(pkg, mToken, tn, mDuration, displayId);
            }
        } catch (RemoteException e) {
            // Empty
        }
    }
}
```

NotificationManagerService在服务端处理ITransientNotification客户端传过来的enqueueToast事件
```
public class NotificationManagerService extends SystemService {
// 源码版本 SDK 28

    // 是否是系统调用
    private static boolean isCallerSystem() {
        return isUidSystem(Binder.getCallingUid());
    }

    private final IBinder mService = new INotificationManager.Stub() {
        @Override
        public void enqueueToast(String pkg, ITransientNotification callback, int duration)
        {
            if (DBG) {
                Slog.i(TAG, "enqueueToast pkg=" + pkg + " callback=" + callback
                        + " duration=" + duration);
            }

            if (pkg == null || callback == null) {
                Slog.e(TAG, "Not doing toast. pkg=" + pkg + " callback=" + callback);
                return ;
            }
            // 是否系统调动
            final boolean isSystemToast = isCallerSystemOrPhone() || ("android".equals(pkg));
            final boolean isPackageSuspended =
                    isPackageSuspendedForUser(pkg, Binder.getCallingUid());

            // Toast或者通知权限被禁用
            if (ENABLE_BLOCKED_TOASTS && !isSystemToast &&
                    (!areNotificationsEnabledForPackage(pkg, Binder.getCallingUid())
                            || isPackageSuspended)) {
                Slog.e(TAG, "Suppressing toast from package " + pkg
                        + (isPackageSuspended
                                ? " due to package suspended by administrator."
                                : " by user request."));
                return;
            }

            // mToastQueue同步锁
            synchronized (mToastQueue) {
                int callingPid = Binder.getCallingPid();
                long callingId = Binder.clearCallingIdentity();
                try {
                    ToastRecord record;
                    int index;
                    // All packages aside from the android package can enqueue one toast at a time
                    if (!isSystemToast) {
                        // 寻找当前callback在mToastQueue中的索引，没找到则返回-1
                        index = indexOfToastPackageLocked(pkg);
                    } else {
                        index = indexOfToastLocked(pkg, callback);
                    }

                    // If the package already has a toast, we update its toast
                    // in the queue, we don't move it to the end of the queue.
                    // callback的索引在mToastQueue中存在，则更新时长,并重置callback
                    if (index >= 0) {
                        record = mToastQueue.get(index);
                        record.update(duration);
                        try {
                            // 隐藏callback
                            record.callback.hide();
                        } catch (RemoteException e) {
                        }
                        // 再更新callback
                        record.update(callback);
                    } else {
                        Binder token = new Binder();
                        // 生成一个Toast窗口(使用token)，由NotificationManager 系统服务生成
                        mWindowManagerInternal.addWindowToken(token, TYPE_TOAST, DEFAULT_DISPLAY);
                        record = new ToastRecord(callingPid, pkg, callback, duration, token);
                        // 将新的ToastRecord对象加入队列，索引为size - 1
                        mToastQueue.add(record);
                        index = mToastQueue.size() - 1;
                    }
                    keepProcessAliveIfNeededLocked(callingPid);
                    // If it's at index 0, it's the current toast.  It doesn't matter if it's
                    // new or just been updated.  Call back and tell it to show itself.
                    // If the callback fails, this will remove it from the list, so don't
                    // assume that it's valid after this.
                    // 如果队列中没有正在处理的消息，处理当前这个ToastRecord
                    if (index == 0) {
                        showNextToastLocked();
                    }
                } finally {
                    Binder.restoreCallingIdentity(callingId);
                }
            }
        }
    }
}
```
当 Android 进程需要构建一个窗口的时候，必须指定这个窗口的类型。 Toast指定的类型是:
public static final int TYPE_TOAST = FIRST_SYSTEM_WINDOW+5;//系统窗口
可以看出， Toast 是一个系统窗口，这就保证了 Toast 可以在 Activity 所在的窗口之上显示，并可以在其他的应用上层显示。

NotificationManagerService使用先进先出（FIFO）的方式处理 mToastQueue 队列中的消息
- 服务端处理
```
public class NotificationManagerService extends SystemService {

    @GuardedBy("mToastQueue")
    void showNextToastLocked() {
        // 获取队列第一个ToastRecord
        ToastRecord record = mToastQueue.get(0);
        while (record != null) {
            if (DBG) Slog.d(TAG, "Show pkg=" + record.pkg + " callback=" + record.callback);
            try {
                // 调用客户端Binder对应的TN.show方法，传递token
                record.callback.show(record.token);
                // 超时监听消息
                scheduleDurationReachedLocked(record);
                return;
            } catch (RemoteException e) {
                Slog.w(TAG, "Object died trying to show notification " + record.callback
                        + " in package " + record.pkg);
                // remove it from the list and let the process die
                int index = mToastQueue.indexOf(record);
                if (index >= 0) {
                    mToastQueue.remove(index);
                }
                // 切换当前ToastRecord进程
                keepProcessAliveIfNeededLocked(record.pid);
                if (mToastQueue.size() > 0) {
                    record = mToastQueue.get(0);
                } else {
                    record = null;
                }
            }
        }
    }
}
```
这里，showNextToastLocked 函数将调用 ToastRecord的 callback 成员的 show 方法通知进程显示，那么 callback 是什么呢？
```
final ITransientNotification callback;//TN的Binder代理对象
```
我们看到 callback 的声明，可以知道它是一个 ITransientNotification 类型的对象，而这个对象实际上就是我们刚才所说的 TN 类型对象的代理对象:
```
private static class TN extends ITransientNotification.Stub {
    ...
}
```

- 客户端的处理
回调TN的show()方法
```
public class Toast {

    private static class TN extends ITransientNotification.Stub {
        final Handler mHandler;
        View mView;
        View mNextView;
        private final ToastPresenter mPresenter;
        /**
         * Creates a {@link ITransientNotification} object.
         *
         * The parameter {@code callbacks} is not copied and is accessed with itself as its own
         * lock.
         */
        TN(Context context, String packageName, Binder token, List<Callback> callbacks,
                @Nullable Looper looper) {
            IAccessibilityManager accessibilityManager = IAccessibilityManager.Stub.asInterface(
                    ServiceManager.getService(Context.ACCESSIBILITY_SERVICE));
            ...
            mHandler = new Handler(looper, null) {
                @Override
                public void handleMessage(Message msg) {
                    switch (msg.what) {
                        case SHOW: {
                            IBinder token = (IBinder) msg.obj;
                            // 处理界面显示
                            handleShow(token);
                            break;
                        }
                        ...
                    }
                }
            };
        }

        /**
         * schedule handleShow into the right thread
         */
        @Override
        @UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P, trackingBug = 115609023)
        public void show(IBinder windowToken) {
            if (localLOGV) Log.v(TAG, "SHOW: " + this);
            // 得到服务端传过来的token
            mHandler.obtainMessage(SHOW, windowToken).sendToTarget();
        }

        public void handleShow(IBinder windowToken) {
            if (localLOGV) Log.v(TAG, "HANDLE SHOW: " + this + " mView=" + mView
                    + " mNextView=" + mNextView);
            // If a cancel/hide is pending - no need to show - at this point
            // the window token is already invalid and no need to do any work.
            if (mHandler.hasMessages(CANCEL) || mHandler.hasMessages(HIDE)) {
                return;
            }
            // 判断mNextView是否展示过
            if (mView != mNextView) {
                // remove the old view if necessary
                // 移除当前展示的Toast
                handleHide();
                mView = mNextView;
                // 调用Presenter处理展示
                mPresenter.show(mView, mToken, windowToken, mDuration, mGravity, mX, mY,
                        mHorizontalMargin, mVerticalMargin,
                        new CallbackBinder(getCallbacks(), mHandler));
            }
        }
    }
}
```

使用了MVP设计模式，由Presenter处理Window相关业务
```
public class ToastPresenter {
    @Nullable private View mView;
    @Nullable private IBinder mToken;
    private final String mPackageName;
    private final WindowManager.LayoutParams mParams;
    private final WindowManager mWindowManager;

    /**
     * Shows the toast in {@code view} with the parameters passed and callback {@code callback}.
     */
    public void show(View view, IBinder token, IBinder windowToken, int duration, int gravity,
            int xOffset, int yOffset, float horizontalMargin, float verticalMargin,
            @Nullable ITransientNotificationCallback callback) {
        checkState(mView == null, "Only one toast at a time is allowed, call hide() first.");
        mView = view;
        mToken = token;

        // 将token等赋值给窗口属性对象 mParams
        adjustLayoutParams(mParams, windowToken, duration, gravity, xOffset, yOffset,
                horizontalMargin, verticalMargin);
        // 如果mView添加过，那么先把mView从WindowManager中移除
        if (mView.getParent() != null) {
            mWindowManager.removeView(mView);
        }
        try {
            // 把需要展示的View添加在WindowManager中
            mWindowManager.addView(mView, mParams);
        } catch (WindowManager.BadTokenException e) {
            // Since the notification manager service cancels the token right after it notifies us
            // to cancel the toast there is an inherent race and we may attempt to add a window
            // after the token has been invalidated. Let us hedge against that.
            // UI线程长时间阻塞时token可能被回收，抛出异常
            Log.w(TAG, "Error while attempting to show toast from " + mPackageName, e);
            return;
        }
        trySendAccessibilityEvent(mView, mPackageName);
        if (callback != null) {
            try {
                callback.onToastShown();
            } catch (RemoteException e) {
                Log.w(TAG, "Error calling back " + mPackageName + " to notify onToastShow()", e);
            }
        }
    }
}
```
可以看出，NotificationManager 不仅掌管着 Toast 的生成，也管理着 Toast 的时序控制。

##### Toast的时序管理
在Toast的的展示中，服务端的处理调用了scheduleDurationReachedLocked(ToastRecord r)方法。用于管理 Toast 时序，定时删除Toast。

- 服务端处理
```
public class NotificationManagerService extends SystemService {
    // 让当前Toast展示一段时间后消失
    @GuardedBy("mToastQueue")
    private void scheduleDurationReachedLocked(ToastRecord r)
    {
        mHandler.removeCallbacksAndMessages(r);
        Message m = Message.obtain(mHandler, MESSAGE_DURATION_REACHED, r);
        long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
        mHandler.sendMessageDelayed(m, delay);
    }

    protected class WorkerHandler extends Handler
    {
        @Override
        public void handleMessage(Message msg)
        {
            switch (msg.what)
            {
                case MESSAGE_DURATION_REACHED:
                    handleDurationReached((ToastRecord)msg.obj);
                    break;
                    ...
            }
        }
    }

    private void handleDurationReached(ToastRecord record)
    {
        if (DBG) Slog.d(TAG, "Timeout pkg=" + record.pkg + " callback=" + record.callback);
        synchronized (mToastQueue) {
            // 找当前ToastRecord在mToastQueue队列中的索引
            int index = indexOfToastLocked(record.pkg, record.callback);
            if (index >= 0) {
                cancelToastLocked(index);
            }
        }
    }

    @GuardedBy("mToastQueue")
    void cancelToastLocked(int index) {
        ToastRecord record = mToastQueue.get(index);
        try {
            // 调用客户端Binder对应的TN.hide方法
            record.callback.hide();
        } catch (RemoteException e) {
            Slog.w(TAG, "Object died trying to hide notification " + record.callback
                    + " in package " + record.pkg);
            // don't worry about this, we're about to remove it from
            // the list anyway
        }

        // 移除处理完的ToastRecord
        ToastRecord lastToast = mToastQueue.remove(index);

        mWindowManagerInternal.removeWindowToken(lastToast.token, false /* removeWindows */,
                DEFAULT_DISPLAY);
        // We passed 'false' for 'removeWindows' so that the client has time to stop
        // rendering (as hide above is a one-way message), otherwise we could crash
        // a client which was actively using a surface made from the token. However
        // we need to schedule a timeout to make sure the token is eventually killed
        // one way or another.
        // 移除窗口的token，可能发生BadToken异常
        scheduleKillTokenTimeout(lastToast.token);

        keepProcessAliveIfNeededLocked(record.pid);
        if (mToastQueue.size() > 0) {
            // Show the next one. If the callback fails, this will remove
            // it from the list, so don't assume that the list hasn't changed
            // after this point.
            // 处理队列中的下一个ToastRecord
            showNextToastLocked();
        }
    }
}
```
NotificationManagerService 内部通过调用 Handler 的 sendMessageDelayed 函数来实现定时调用

- 客户端处理
```
public class Toast {

    private static class TN extends ITransientNotification.Stub {
        final Handler mHandler;
        View mView;
        View mNextView;
        private final ToastPresenter mPresenter;

        /**
         * Creates a {@link ITransientNotification} object.
         *
         * The parameter {@code callbacks} is not copied and is accessed with itself as its own
         * lock.
         */
        TN(Context context, String packageName, Binder token, List<Callback> callbacks,
                @Nullable Looper looper) {
            IAccessibilityManager accessibilityManager = IAccessibilityManager.Stub.asInterface(
                    ServiceManager.getService(Context.ACCESSIBILITY_SERVICE));
            ...
            mHandler = new Handler(looper, null) {
                @Override
                public void handleMessage(Message msg) {
                    switch (msg.what) {
                        case HIDE: {
                            handleHide();
                            // Don't do this in handleHide() because it is also invoked by
                            // handleShow()
                            mNextView = null;
                            break;
                        }
                        ...
                    }
                }
            };
        }

        /**
         * schedule handleHide into the right thread
         */
        @Override
        public void hide() {
            if (localLOGV) Log.v(TAG, "HIDE: " + this);
            mHandler.obtainMessage(HIDE).sendToTarget();
        }

        @UnsupportedAppUsage
        public void handleHide() {
            if (localLOGV) Log.v(TAG, "HANDLE HIDE: " + this + " mView=" + mView);
            if (mView != null) {
                checkState(mView == mPresenter.getView(),
                        "Trying to hide toast view different than the last one displayed");
                // 调用Presenter移除mView
                mPresenter.hide(new CallbackBinder(getCallbacks(), mHandler));
                mView = null;
            }
        }
    }
}
```

```
public class ToastPresenter {
    @Nullable private View mView;
    @Nullable private IBinder mToken;
    private final String mPackageName;
    private final WindowManager.LayoutParams mParams;
    private final WindowManager mWindowManager;

    /**
     * Hides toast that was shown using {@link #show(View, IBinder, IBinder, int,
     * int, int, int, float, float, ITransientNotificationCallback)}.
     *
     * <p>This method has to be called on the same thread on which {@link #show(View, IBinder,
     * IBinder, int, int, int, int, float, float, ITransientNotificationCallback)} was called.
     */
    public void hide(@Nullable ITransientNotificationCallback callback) {
        checkState(mView != null, "No toast to hide.");

        if (mView.getParent() != null) {
            // 调用WindowManager的removeView移除mView
            mWindowManager.removeViewImmediate(mView);
        }
        try {
             // 使用完毕，清除token
            mNotificationManager.finishToken(mPackageName, mToken);
        } catch (RemoteException e) {
            Log.w(TAG, "Error finishing toast window token from package " + mPackageName, e);
        }
        if (callback != null) {
            try {
                callback.onToastHidden();
            } catch (RemoteException e) {
                Log.w(TAG, "Error calling back " + mPackageName + " to notify onToastHide()", e);
            }
        }
        mView = null;
        mToken = null;
    }
}
```
最终做了两件事:
1. 远程调用 ITransientNotification.hide 方法，通知客户端隐藏窗口
2. 将给 Toast 生成的窗口 Token 从 WMS 服务中删除

Toast 的显示和隐藏大致可分成以下核心步骤:
Toast 调用 show 方法的时候 ，实际上是将自己纳入到 NotificationManager 的 Toast 管理中去，期间传递了一个本地的 TN 类型或者是 ITransientNotification.Stub 的 Binder 对象
NotificationManager 收到 Toast 的显示请求后，将生成一个 Binder 对象，将它作为一个窗口的 token 添加到 WMS 对象，并且类型是 TOAST
NotificationManager 将这个窗口 token 通过 ITransientNotification 的 show 方法传递给远程的 TN 对象，并且抛出一个超时监听消息 scheduleTimeoutLocked
TN 对象收到消息以后将往 Handler 对象中 post 显示消息，然后调用显示处理函数将 Toast 中的 View 添加到了 WMS 管理中， Toast 窗口显示
NotificationManager 的 WorkerHandler 收到 MESSAGE_TIMEOUT 消息， NotificationManager 远程调用进程隐藏 Toast 窗口，然后将窗口 token 从 WMS 中删除

### 回到问题
定位到报错的对应行
```
    ...
    public void show(View view, IBinder token, IBinder windowToken, int duration, int gravity,
            int xOffset, int yOffset, float horizontalMargin, float verticalMargin,
            @Nullable ITransientNotificationCallback callback) {
        checkState(mView == null, "Only one toast at a time is allowed, call hide() first.");
        mView = view;
        mToken = token;

        adjustLayoutParams(mParams, windowToken, duration, gravity, xOffset, yOffset,
                horizontalMargin, verticalMargin);
        if (mView.getParent() != null) {
            mWindowManager.removeView(mView);
        }
        try {
            mWindowManager.addView(mView, mParams);
        } catch (WindowManager.BadTokenException e) {
            // Since the notification manager service cancels the token right after it notifies us
            // to cancel the toast there is an inherent race and we may attempt to add a window
            // after the token has been invalidated. Let us hedge against that.
            Log.w(TAG, "Error while attempting to show toast from " + mPackageName, e);
            return;
        }
        trySendAccessibilityEvent(mView, mPackageName);
        if (callback != null) {
            try {
                callback.onToastShown();
            } catch (RemoteException e) {
                Log.w(TAG, "Error calling back " + mPackageName + " to notify onToastShow()", e);
            }
        }
    }
```
mView.getParent()不为空，表示mView仍在ViewRootImpl的View集合中；但在mWindowManager(WindowManagerGlobal)的View集合中却不存在。

看看WindowManagerGlobal的removeView()方法
```
public final class WindowManagerGlobal {
    public void removeView(View view, boolean immediate) {
        if (view == null) {
            throw new IllegalArgumentException("view must not be null");
        }

        synchronized (mLock) {
            // 查找该view在集合中的索引
            int index = findViewLocked(view, true);
            View curView = mRoots.get(index).getView();
            removeViewLocked(index, immediate);
            if (curView == view) {
                return;
            }

            throw new IllegalStateException("Calling with view " + view
                    + " but the ViewAncestor is attached to " + curView);
        }
    }

    private int findViewLocked(View view, boolean required) {
        final int index = mViews.indexOf(view);
        if (required && index < 0) {
            throw new IllegalArgumentException("View=" + view + " not attached to window manager");
        }
        return index;
    }

    // 如正常执行过，应在该方法中已将mView.getParent()置为空
    private void removeViewLocked(int index, boolean immediate) {
        ViewRootImpl root = mRoots.get(index);
        View view = root.getView();

        if (view != null) {
            InputMethodManager imm = InputMethodManager.getInstance();
            if (imm != null) {
                imm.windowDismissed(mViews.get(index).getWindowToken());
            }
        }
        boolean deferred = root.die(immediate);
        if (view != null) {
            // 置mView.getParent()为空
            view.assignParent(null);
            if (deferred) {
                mDyingViews.add(view);
            }
        }
    }
}
```
上下文使用ApplicationContext，如之前执行过WindowManagerGlobal.removeView()。 mView.getParent()应该是空的，纳闷...


1.怀疑是共用Toast问题。只是修改了setText()，再调用show()，是否算同一个callback？做如下测试: 
```
        // 三个相同的TN(callback)，只是setText不同。如callback在队列中已存在，不再加入队列。
        entry.getPrompt().showToast("测试" + (i++), Toast.LENGTH_LONG);
        entry.getPrompt().showToast("测试" + (i++), Toast.LENGTH_LONG);
        entry.getPrompt().showToast("测试" + (i++), Toast.LENGTH_LONG);
        // 三个不同的TN(callback)。轮询展示。
        Toast.makeText(entry.getApplicationContext(), "测试" + (i++), Toast.LENGTH_LONG).show();
        Toast.makeText(entry.getApplicationContext(), "测试" + (i++), Toast.LENGTH_LONG).show();
        Toast.makeText(entry.getApplicationContext(), "测试" + (i++), Toast.LENGTH_LONG).show();
```
结果发现，仅调用setText()和show()，属于同一个callback。如callback在队列中已存在，不再加入队列。与出现的异常无关。

2.发生过Activity跳转，小米是否上下文产生影响？待解决..
3.MIUI内部调用了setView(View view)？mView是一直被复用，但都是Toast(applicationContext)调用且是有时序的，遵循show()-hide()-show()...的周期，mView不会同时被使用


 








参考链接：
https://blog.csdn.net/stven_king/article/details/78775166
https://www.androidos.net.cn/android/9.0.0_r8/xref/frameworks/base/services/core/java/com/android/server/notification/NotificationManagerService.java
https://www.cnblogs.com/qcloud1001/p/8421356.html
https://stackoverflow.com/questions/13647750/java-lang-illegalargumentexception-view-not-attached-to-window-manager-when-c
