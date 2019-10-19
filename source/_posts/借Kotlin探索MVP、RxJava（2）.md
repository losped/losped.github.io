---
title: 借Kotlin探索MVP、RxJava（2）
date: 2018.10.31 14:48:30
tags: Android
categories: Android
---


# Kotlin补充

## 异常
“**Kotlin中没有检验异常**！”

而抛出异常和try-catch-finally和Java中的类似！但是Kotlin中**throw和try都是表达式**，
意味着他们**可以赋值给某个变量**，这一点在处理边界问题的时候很有用！

**比如**：

![Exception](/images/借Kotlin探索MVP、RxJava/Exception.jpg)

，输出结果：

![Exception1](/images/借Kotlin探索MVP、RxJava/Exception1.jpg)

## 高阶函数
#### 局部函数

```
fun outter(): Unit {
    val a = 12
    fun inner(): Unit {
        print("inner" + a )
    }
    //invoke
    inner()
}
```
那么inner函数就是局部函数，对于在outter声明的变量，在inner中可以使用外部函数的局部变量(闭包)

#### 高阶函数

高阶函数就是有一个函数作为参数，或者是返回值是一个函数
(parameterType) -> T , 函数类型就是这种格式。在kotlin中所有东西都可以看作是对象。
```
// 函数参数为函数类型，此函数类型为：没有参数，返回值为Unit
fun funType(a: () -> Unit): Unit {

    //声明一个函数类型的变量，此函数类型为：参数为Int，返回值为Int
    val b: (Int) -> Int
}
```
使用**::**然后加上函数名就可以将函数以参数的形式传递过去
```
fun funType(a: () -> Unit): Unit {
    //invoke 1
    a()
    //invoke 2
    a.invoke()
}

fun main(args: Array<String>) {

    //局部函数
    fun inner() = println("hello world")//单表达式函数
    funType(::inner)//调用高阶函数

}
```
**::**然后加上函数名就可以将函数以参数的形式传递过去，而作为**函数类型的参数**，调用这个参数也有有如上两种方法

```
fun funType(a: () -> Unit): Unit {
}

fun funType1(b: Int, a: (Int) -> Unit): Unit {
}

fun funType2(a: (String) -> Unit): Unit {
}

fun main(args: Array<String>) {

    val a = 12
    funType({ print("hello") })//Lambda
    funType1(1) { it -> print(it) }//将花括号提到外面
    funType2 { it -> print(it) }//参数只有一个函数类型的时候可以将()省略掉
    funType2 { print(it) }//单一函数类型参数的隐含名称
}
```
使用时，使用invoke传入作为参数的函数里面的参数
```
/**VideoDetailActivity
        mAdapter.setOnItemDetailClick { mPresenter.loadVideoInfo(it) }
```

```
/**VideoDetailAdapter
    private var mOnItemClickRelatedVideo: ((item: HomeBean.Issue.Item) -> Unit)? = null

    fun setOnItemDetailClick(mItemRelatedVideo: (item: HomeBean.Issue.Item) -> Unit) {
        this.mOnItemClickRelatedVideo = mItemRelatedVideo
    }

      holder.itemView.setOnClickListener { mOnItemClickRelatedVideo?.invoke(data) }  //Losped?： invoke传入函数里面的参数
```
#### 内联函数

```
inline fun funType(): Unit {
    println("inline，不是高阶函数")
}

fun funType1(): Boolean {
    funType()
    funType2 { return true }//直接退出funType1()
    return false //不会执行
}

inline fun funType2(a: () -> Unit): Unit {
    println("inline 高阶函数")
    a.invoke()
}   
fun main(args: Array<String>): Unit {
    println(funType1())
}

```
inline函数funType1相当于下面的代码
```
fun funType1(): Boolean {
    println("inline，不是高阶函数")
    println("inline 高阶函数")
    return true
    return false
}
```
有inline当然也会有noinline，noinline函数只能使用在inline高阶函数中修饰函数类型的参数
```
inline fun funType2(a: () -> Unit, noinline b: () -> String): Unit {
    println("inline 高阶函数")
    b.invoke()
    a.invoke()
}

//在funType1中调用
funType2({
    println("return true 之前")
    return true//直接退出funType1()
}, {
    if (true) "df"
    else "df"

})
//相当于
fun funType1(): Boolean {
    b()//调用函数
    println("return true 之前")
    return true//直接退出funType1()
}
```
所以对于函数类型参数b，它就是不会插入到调用者的函数体中，会存在着函数调用

#### 五个函数
**run**
功能：调用run函数块。返回值为函数块最后一行，或者指定return表达式。
```
val a = run {
    println("run")
    return@run 3
}
println(a)
运行结果：

run
3
```

**apply**
功能：调用某对象的apply函数，在函数块内可以通过 this 指代该对象。返回值为该对象自己。

**let**
功能：调用某对象的let函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为函数块的最后一行或指定return表达式。

**also**
功能：调用某对象的also函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为该对象自己。

**with**
功能：with函数和前面的几个函数使用方式略有不同，因为它不是以扩展的形式存在的。它是将某对象作为函数的参数，在函数块内可以通过 this 指代该对象。返回值为函数块的最后一行或指定return表达式。
```
val a = with("string") {
    println(this)   \\输出string
    3
}
println(a)
```
#### mapTo函数
将现有数据转化为其他数据然后加入到目的集合中
```
(0 until mTitles.size).mapTo(mTabEntities){ TabEntity(mTitles[it], mIconSelectIds[it], mIconUnSelectIds[it]) }
```
#### kotlin 中::class 、class.java、javaClass、javaClass.kotlin区别

![class](/images/借Kotlin探索MVP、RxJava/class.jpg)

![class1](/images/借Kotlin探索MVP、RxJava/class1.jpg)

#### 集合函数
**filter和forEach**
```
                        newBannerItemList.filter { item ->
                            item.type=="banner2"||item.type=="horizontalScrollCard"
                        }.forEach{ item ->
                            //移除 item
                            newBannerItemList.remove(item)
                        }
```


##### 参考资料
https://www.cnblogs.com/duduhuo/p/6934137.html
https://www.jianshu.com/p/8d2987c343b5
https://www.jianshu.com/p/29c8b5775663
