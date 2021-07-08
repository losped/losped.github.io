---
title: 自动化测试-UiAutoMator
date: 2019.12.16 13:39:59
tags: Android
categories: Android
---

UiAutomator是Google提供的用来做安卓自动化测试的一个Java库，基于Accessibility服务，可以获取屏幕任意APP的页面控件信息，并对控件进行操作。
图形化界面：sdk\\tools\\bin\\uiautomatorviewer.bat
脚本形式如下：
1.创建java脚本项目，导入android.jar、uiautomator.jar。（在SDK/platforms/可找到对应版本的jar包）
2.创建类并编写脚本
```
package com.test;
public class Test extends UiAutomatorTestCase {
    private int x;
    private int y;
    //必须包含一个test字样的public void方法
    public void testDemo() {
        x = Integer.valueOf(getParamsVal("pointX"));
        y = Integer.valueOf(getParamsVal("pointY"));
        UiDevice uiDevice = UiDevice.getInstance();  // 获取UiDevice
        uiDevice.click(x, y); //点击界面上坐标为(50,80)的坐标点
    }

    private Bundle params = null;
    //key：cmd命令-e 后面的字段
    public String getParamsVal(String key) {
        params = getParams();
        String val = null;
        if (params.containsKey(key)) {
            val = params.getString(key);
        }
        return val;
    }
}
```
3.将项目打包成jar包
(1)使用CMD打包。javac编译java文件得到class文件，新建manifest文件，jar -cvfm main.jar manifest -C test .
(2)使用IDE打包
4.push并执行.
adb push ./test.jar /data/local/tmp/
adb shell uiautomator runtest /data/local/tmp/test.jar -c com.test.TestClass -e x 50 -e y 18


##### uiautomator API

```
public class Test extends UiAutomatorTestCase {
    public void testDemo() {
        UiDevice uiDevice = UiDevice.getInstance();  // 获取UiDevice
        uiDevice.pressBack(); //模拟点击返回键
        try {
            uiDevice.wakeUp(); //唤醒屏幕，如果屏幕是处于唤醒状态，则无效
        } catch (RemoteException e) {
            e.printStackTrace();
        }
        uiDevice.pressHome(); //点击home键
        uiDevice.click(50, 80); //点击界面上坐标为(50,80)的坐标点
        int height = uiDevice.getDisplayHeight(); //获取屏幕高度
        int width = uiDevice.getDisplayWidth(); //获取屏幕宽度
        uiDevice.pressEnter(); //模拟短按回车键
        uiDevice.pressMenu(); //模拟短按menu键
        uiDevice.pressKeyCode(KeyEvent.KEYCODE_NUMPAD_ENTER); //模拟按回车键
        uiDevice.swipe(10, 10, 100, 100, 50);   //从坐标点(10,10) 滑动到 坐标点(100,100) ，步长为50

        String path = DroidCmd.PICCACHE_PATH + System.currentTimeMillis() + ".png";
        File file = new File(path);   //根据文件名创建文件
        uiDevice.takeScreenshot(file);    //截图

        final UiObject check = new UiObject(CheckBox);
        final UiObject btn = new UiObject(Button);
        uiDevice.registerWatcher("test", new UiWatcher() {
            //这里重写checkForCondition方法
            public boolean checkForCondition() {
                try {
                    if (check.exists()) {
                        btn.click();
                        Thread.sleep(1000);
                        return true;
                    }
                    else{
                        Thread.sleep(1000);
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }

                return false;
            }
        });

        //通过resource-id查找控件
           UiObject uiObject = new UiObject(new UiSelector().resourceId("通过 uiautomatorviewer 查看到的id"));
           //通过class查找控件
           UiObject uiObject1 = new UiObject(new UiSelector().className("通过 uiautomatorviewer 查看到的 class"));
           //通过content-desc查找控件
           UiObject uiObject2 = new UiObject(new UiSelector().description("通过 uiautomatorviewer 查看到的 class"));
           //通过text查找控件
           UiObject uiObject3 = new UiObject(new UiSelector().text("通过 uiautomatorviewer 查看到的text"));
           //通过id和text 组合查找控件
           UiObject uiObject4 = new UiObject(new UiSelector().resourceId(id).text(text));
    }

}
```

IClipboard输入框实例
```
public class xxx extends UiAutomatorTestCase {
	public void testDemo() throws UiObjectNotFoundException {
		//先根据id，找到文本输入框，然后调用setMsgToEdit方法
		UiObject uiEdit = new UiObject(new UiSelector().resourceId("jp.naver.line.android:id/chathistory_message_edit"));
		if(uiEdit.exists()) {
			setMsgToEdit("你好");
		}
	}

	public void setMsgToEdit(String string) {
		try {
			string = URLDecoder.decode(string, "UTF-8");
		} catch (Exception e) {
		}
		try {
			Thread.sleep(500);
		} catch (Exception e) {
		}

		IClipboard iClipboard = IClipboard.Stub
				.asInterface((IBinder) ServiceManager.getService("clipboard"));
		// IInputManager.Stub.asInterface((IBinder)ServiceManager.getService("input"));
		try {
			iClipboard.setPrimaryClip(
					ClipData.newPlainText("NonASCII", string), string);
		} catch (RemoteException var1_2) {
			var1_2.printStackTrace();
		}
		paste();
		try {
			iClipboard
					.setPrimaryClip(ClipData.newPlainText("NonASCII", ""), "");
		} catch (Exception e) {
		}
	}

	private void paste() {
		UiDevice.getInstance().pressKeyCode(50, 4096);
	}
}
————————————————
```

版权声明：本文参考了CSDN博主「JianXiongx」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/JianXiongx/article/details/100882993
