---
title: 借Kotlin探索MVP、RxJava（3）
date: 2018.10.31 18:02:29
tags: Android
categories: Android
---


# RxJava2
本文仅作个人笔记总结使用。目前接触到的程度限于基本使用，更丰富的类型还需查阅文档运用。

**发射者：Observable，Flowable**
**接收者：Observer，Subscriber**
**发射参数：ObservableEmitter等**
**接收参数：Disposable，Consumer等**

**CompositeDisposable：将Disposable统一管理**
添加订阅：CompositeDisposable().add(Disposable)
清空订阅：CompositeDisposable().clear()


### Observable的简单示例
```
Observable.create(new ObservableOnSubscribe<Integer>() { // 第一步：初始化Observable
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                Log.e(TAG, "Observable emit 1" + "\n");
                e.onNext(1);
                Log.e(TAG, "Observable emit 2" + "\n");
                e.onNext(2);
                Log.e(TAG, "Observable emit 3" + "\n");
                e.onNext(3);
                e.onComplete();
                Log.e(TAG, "Observable emit 4" + "\n" );
                e.onNext(4);
            }
        }).subscribe(new Observer<Integer>() { // 第三步：订阅

            // 第二步：初始化Observer
            private int i;
            private Disposable mDisposable;

            @Override
            public void onSubscribe(@NonNull Disposable d) {      
                mDisposable = d;
            }

            @Override
            public void onNext(@NonNull Integer integer) {
                i++;
                if (i == 2) {
                    // 在RxJava 2.x 中，新增的Disposable可以做到切断的操作，让Observer观察者不再接收上游事件
                    mDisposable.dispose();
                }
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.e(TAG, "onError : value : " + e.getMessage() + "\n" );
            }

            @Override
            public void onComplete() {
                Log.e(TAG, "onComplete" + "\n" );
            }
        });
```
进行相应的简化订阅，只不过传入对象改为了 **Consumer**。Consumer 即消费者，用于接收单个值，BiConsumer 则是接收两个值，Function 用于变换对象，Predicate 用于判断
```
        val disposable = homeModel.requestHomeData(num)
                .flatMap({ homeBean ->

                    Log.i("TAG","HomeBean格式2：" + homeBean.toString())

                    //过滤掉 Banner2(包含广告,等不需要的 Type), 具体查看接口分析
                    val bannerItemList = homeBean.issueList[0].itemList

                    //Losped :filter过滤集合
                    bannerItemList.filter { item ->
                        item.type=="banner2"|| item.type=="horizontalScrollCard"
                    }.forEach{ item ->
                        //移除 item
                        bannerItemList.remove(item)
                    }

                    Log.i("TAG","HomeBean格式3：" + homeBean.issueList[0].itemList.size)
                    Log.i("TAG","HomeBean格式3：" + homeBean.issueList[0].itemList.toString())

                    bannerHomeBean = homeBean //记录第一页是当做 banner 数据


                    //根据 nextPageUrl 请求下一页数据
                    homeModel.loadMoreData(homeBean.nextPageUrl)  //Losped默认最后一行 return
                })

                //Losped：
                // 匿名内部类，参数→返回值，(new Consumer<MobileAddress>() { @Override public void accept(@NonNull MobileAddress data) throws Exception {}}
                // mRootView?.apply：调用某对象的apply函数，在函数块内可以通过 this 指代该对象。返回值为该对象自己。
                .subscribe({ homeBean->
                    mRootView?.apply {
                        dismissLoading()

                        nextPageUrl = homeBean.nextPageUrl
                        //过滤掉 Banner2(包含广告,等不需要的 Type), 具体查看接口分析
                        val newBannerItemList = homeBean.issueList[0].itemList

                        newBannerItemList.filter { item ->
                            item.type=="banner2"||item.type=="horizontalScrollCard"
                        }.forEach{ item ->
                            //移除 item
                            newBannerItemList.remove(item)
                        }
                        // 重新赋值 Banner 长度
                        bannerHomeBean!!.issueList[0].count = bannerHomeBean!!.issueList[0].itemList.size

                        //赋值过滤后的数据 + banner 数据
                        bannerHomeBean?.issueList!![0].itemList.addAll(newBannerItemList)
                        Log.i("TAG","HomeBean格式4：" + bannerHomeBean!!.issueList[0].itemList.size)
                        Log.i("TAG","HomeBean格式4：" + bannerHomeBean!!.issueList[0].itemList.toString())

                        setHomeData(bannerHomeBean!!)

                    }

                }, { t ->
                    mRootView?.apply {
                        dismissLoading()
                        showError(ExceptionHandle.handleException(t),ExceptionHandle.errorCode)
                    }
                })
```
这里Kotlin匿名实现了Consumer

### 线程调度
subScribeOn： 用于指定 subscribe() 时所发生的线程，发射事件的线程
observeOn：用于指定下游 Observer 回调发生的线程，订阅者接收事件的线程

**内置线程选择**
Schedulers.io() 代表io操作的线程, 通常用于网络,读写文件等io密集型的操作；
Schedulers.computation() 代表CPU计算密集型的操作, 例如需要大量计算的操作；
Schedulers.newThread() 代表一个常规的新线程；
AndroidSchedulers.mainThread() 代表Android的主线程

### 操作符

##### map

map函数：对数据进行加工
map(new Function<A, B>{}),A表示输入类型，B表示返回类型。
**网络请求**
```
Observable.create(new ObservableOnSubscribe<Response>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Response> e) throws Exception {
                Builder builder = new Builder()
                        .url("http://api.avatardata.cn/MobilePlace/LookUp?key=ec47b85086be4dc8b5d941f5abd37a4e&mobileNumber=13021671512")
                        .get();
                Request request = builder.build();
                Call call = new OkHttpClient().newCall(request);
                Response response = call.execute();
                e.onNext(response);
            }
        }).map(new Function<Response, MobileAddress>() {
                    @Override
                    public MobileAddress apply(@NonNull Response response) throws Exception {
                        if (response.isSuccessful()) {
                            ResponseBody body = response.body();
                            if (body != null) {
                                Log.e(TAG, "map:转换前:" + response.body());
                                return new Gson().fromJson(body.string(), MobileAddress.class);
                            }
                        }
                        return null;
                    }
                }).observeOn(AndroidSchedulers.mainThread())
                .doOnNext(new Consumer<MobileAddress>() {
                    @Override
                    public void accept(@NonNull MobileAddress s) throws Exception {
                        Log.e(TAG, "doOnNext: 保存成功：" + s.toString() + "\n");
                    }
                }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<MobileAddress>() {
                    @Override
                    public void accept(@NonNull MobileAddress data) throws Exception {
                        Log.e(TAG, "成功:" + data.toString() + "\n");
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "失败：" + throwable.getMessage() + "\n");
                    }
                });
```

##### concat
concat函数：按顺序发射Observable，调用 onComplete 后才能订阅下一个 Observable
**先读取缓存再通过网络请求获取数据**
```
Observable<FoodList> cache = Observable.create(new ObservableOnSubscribe<FoodList>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<FoodList> e) throws Exception {
                Log.e(TAG, "create当前线程:"+Thread.currentThread().getName() );
                FoodList data = CacheManager.getInstance().getFoodListData();

                // 在操作符 concat 中，只有调用 onComplete 之后才会执行下一个 Observable
                if (data != null){ // 如果缓存数据不为空，则直接读取缓存数据，而不读取网络数据
                    isFromNet = false;
                    Log.e(TAG, "\nsubscribe: 读取缓存数据:" );
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            mRxOperatorsText.append("\nsubscribe: 读取缓存数据:\n");
                        }
                    });

                    e.onNext(data);
                }else {
                    isFromNet = true;
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            mRxOperatorsText.append("\nsubscribe: 读取网络数据:\n");
                        }
                    });
                    Log.e(TAG, "\nsubscribe: 读取网络数据:" );
                    e.onComplete();
                }


            }
        });

        Observable<FoodList> network = Rx2AndroidNetworking.get("http://www.tngou.net/api/food/list")
                .addQueryParameter("rows",10+"")
                .build()
                .getObjectObservable(FoodList.class);


        // 两个 Observable 的泛型应当保持一致

        Observable.concat(cache,network)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<FoodList>() {
                    @Override
                    public void accept(@NonNull FoodList tngouBeen) throws Exception {
                        Log.e(TAG, "subscribe 成功:"+Thread.currentThread().getName() );
                        if (isFromNet){
                            mRxOperatorsText.append("accept : 网络获取数据设置缓存: \n");
                            Log.e(TAG, "accept : 网络获取数据设置缓存: \n"+tngouBeen.toString() );
                            CacheManager.getInstance().setFoodListData(tngouBeen);
                        }

                        mRxOperatorsText.append("accept: 读取数据成功:" + tngouBeen.toString()+"\n");
                        Log.e(TAG, "accept: 读取数据成功:" + tngouBeen.toString());
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "subscribe 失败:"+Thread.currentThread().getName() );
                        Log.e(TAG, "accept: 读取数据失败："+throwable.getMessage() );
                        mRxOperatorsText.append("accept: 读取数据失败："+throwable.getMessage()+"\n");
                    }
                });
```
##### flatMap（懵）
flatMap函数：可以将一个发射数据的 Observable 变换为多个 Observables ，然后将它们发射的数据合并后放到一个单独的 Observable
```
Rx2AndroidNetworking.get("http://www.tngou.net/api/food/list")
                .addQueryParameter("rows", 1 + "")
                .build()
                .getObjectObservable(FoodList.class) // 发起获取食品列表的请求，并解析到FootList
                .subscribeOn(Schedulers.io())        // 在io线程进行网络请求
                .observeOn(AndroidSchedulers.mainThread()) // 在主线程处理获取食品列表的请求结果
                .doOnNext(new Consumer<FoodList>() {
                    @Override
                    public void accept(@NonNull FoodList foodList) throws Exception {
                        // 先根据获取食品列表的响应结果做一些操作
                        Log.e(TAG, "accept: doOnNext :" + foodList.toString());
                        mRxOperatorsText.append("accept: doOnNext :" + foodList.toString()+"\n");
                    }
                })
                .observeOn(Schedulers.io()) // 回到 io 线程去处理获取食品详情的请求
                .flatMap(new Function<FoodList, ObservableSource<FoodDetail>>() {
                    @Override
                    public ObservableSource<FoodDetail> apply(@NonNull FoodList foodList) throws Exception {
                        if (foodList != null && foodList.getTngou() != null && foodList.getTngou().size() > 0) {
                            return Rx2AndroidNetworking.post("http://www.tngou.net/api/food/show")
                                    .addBodyParameter("id", foodList.getTngou().get(0).getId() + "")
                                    .build()
                                    .getObjectObservable(FoodDetail.class);
                        }
                        return null;

                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<FoodDetail>() {
                    @Override
                    public void accept(@NonNull FoodDetail foodDetail) throws Exception {
                        Log.e(TAG, "accept: success ：" + foodDetail.toString());
                        mRxOperatorsText.append("accept: success ：" + foodDetail.toString()+"\n");
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "accept: error :" + throwable.getMessage());
                        mRxOperatorsText.append("accept: error :" + throwable.getMessage()+"\n");
                    }
                });

```
##### zip
zip函数：可以将多个来源于不同接口的 Observable 的数据结合为一个数据源再发射出去

```
Observable<MobileAddress> observable1 = Rx2AndroidNetworking.get("http://api.avatardata.cn/MobilePlace/LookUp?key=ec47b85086be4dc8b5d941f5abd37a4e&mobileNumber=13021671512")
                .build()
                .getObjectObservable(MobileAddress.class);

        Observable<CategoryResult> observable2 = Network.getGankApi()
                .getCategoryData("Android",1,1);

        Observable.zip(observable1, observable2, new BiFunction<MobileAddress, CategoryResult, String>() {
            @Override
            public String apply(@NonNull MobileAddress mobileAddress, @NonNull CategoryResult categoryResult) throws Exception {
                return "合并后的数据为：手机归属地："+mobileAddress.getResult().getMobilearea()+"人名："+categoryResult.results.get(0).who;
            }
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<String>() {
                    @Override
                    public void accept(@NonNull String s) throws Exception {
                        Log.e(TAG, "accept: 成功：" + s+"\n");
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        Log.e(TAG, "accept: 失败：" + throwable+"\n");
                    }
                });
```
##### interval
interval函数：轮训，常用于心跳间隔任务
```
private Disposable mDisposable;
    @Override
    protected void doSomething() {
        mDisposable = Flowable.interval(1, TimeUnit.SECONDS)
                .doOnNext(new Consumer<Long>() {   //简单订阅，直接调用Consumer.doOnNext发射
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        Log.e(TAG, "accept: doOnNext : "+aLong );
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<Long>() {
                    @Override
                    public void accept(@NonNull Long aLong) throws Exception {
                        Log.e(TAG, "accept: 设置文本 ："+aLong );
                        mRxOperatorsText.append("accept: 设置文本 ："+aLong +"\n");
                    }
                });
    }

    /**
     * 销毁时停止心跳
     */
    @Override
    protected void onDestroy() {
        super.onDestroy();
        if (mDisposable != null){
            mDisposable.dispose();
        }
    }
```
##### 参考资料
链接：https://www.jianshu.com/p/0cd258eecf60
