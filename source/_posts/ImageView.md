---
title: ImageView
date: 2019.04.04 14:56:15
tags: Android
categories: Android
---


**Drawable获取方式不同使用dpi不同**
```
                            drawable1 = Drawable.createFromStream(fis, "src");  //默认是使用mdpi处理
                            drawable1 = Drawable.createFromResourceStream(DataEntry.this.getResources(), null, fis, "src", null);  //默认使用当前手机dpi处理
                            img.setImageDrawable(drawable1);
                            img.setImageResource(R.drawable.test);  //使用res中不同文件的资源drawable-hdpi/drawable-mdpi等做处理
```

**获取ImageView原图片大小**
```
                            //原图片大小（不随ImageView大小和ScaleType改变）
                            Rect bounds = img.getDrawable().getCurrent().getBounds();
                            Log.i(TAG, "ImageView图片left坐标：" + bounds.left);
                            Log.i(TAG, "ImageView图片top坐标：" + bounds.top);
                            Log.i(TAG, "ImageView图片right坐标：" + bounds.right);
                            Log.i(TAG, "ImageView图片bottom坐标：" + bounds.bottom);
                            int picHeight = bounds.bottom - bounds.top;
                            int picWidth = bounds.right - bounds.left;
                            Log.i(TAG, "ImageView图片高度：" + picHeight);
                            Log.i(TAG, "ImageView图片宽度：" + picWidth);
```

**获取各种系统组件高度**
```
                //屏幕高度（不包括虚拟按键）
                WindowManager manager = DataEntry.this.getWindowManager();
                DisplayMetrics outMetrics = new DisplayMetrics();
                manager.getDefaultDisplay().getMetrics(outMetrics);
                int width = outMetrics.widthPixels;
                int height = outMetrics.heightPixels;

                //ActionBar高度
                int actionBarHeight = 0;
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
                    actionBarHeight = getActionBar().getHeight();
                }

                //状态栏高度（最上方横条）
                Rect frame = new Rect();
                getWindow().getDecorView().getWindowVisibleDisplayFrame(frame);
                int statusBarHeight = frame.top;

```
