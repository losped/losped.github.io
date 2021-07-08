---
title: 借Kotlin探索MVP、RxJava（1）
date: 2018.10.30 21:16:54
tags: Android
categories: Android
---


最近通过学习一个短视频类的小项目，开始踏入**Kotlin**的领域，也借机想形成合适自己的**MVP**实现规范，并加深对**RxJava2**的理解。

# MVP

MVP模式实际就是为了解耦合增加扩展性而存在的。在我的理解就是，原本Activity混杂了逻辑层和视图层，耦合程度比较头疼。

MVP的全称是Model、View、Presenter，将整个应用分为三层。

**View层**：视图层，**包含界面相关的功能**，例如Activity，Fragment，View，Adapter等，该层专注于用户交互，实现设计师给出的界面，动画等交互效果。View层一般会**持有Presenter层的引用**，或者依赖注入（如dagger）的方式获得presenter实例，并将非UI的逻辑操作委托给Presenter。

**Presenter层**：逻辑控制层，充当中间人的角色，用来隔离View层和model层，该层是通过从View层剥离控制逻辑部分而形成，主要负责View层和model层的控制和交互。例如**接受View层的网络数据的加载请求**，并分发给model处理，同时监听model层的处理结果，最终将其反馈给View层从而实现界面的刷新

**model层** 封装各种数据来源，例如**远程网络数据，本地数据库数据**等，对Presenter层提供简单易用的接口

![Contract](/images/借Kotlin探索MVP、RxJava/Contract.jpg)

Presenter层和View层以及model层的交互都是基于接口实现的，这有助于对Presenter进行单元测试，同时由于是面向接口编程，只需要事先定义好接口，每一层的实现都可以交由不同的开发人员并行实现，最终再一起连调，都能明显的加快某一功能的开发进度。虽然使用MVP模式会加大代码量，但分层更清晰合理，利于维护和扩展。

```

interface HomeContract {

//由Activity/Fragment实现

interface View : IBaseView {

/**

        * 设置第一次请求的数据

        */

        fun setHomeData(homeBean: HomeBean)

/**

        * 设置加载更多的数据

        */

        fun setMoreData(itemList:ArrayList)

/**

        * 显示错误信息

        */

        fun showError(msg: String,errorCode:Int)

}

//由Presenter实现

interface Presenter : IPresenter {

/**

        * 获取首页精选数据

        */

        fun requestHomeData(num: Int)

/**

        * 加载更多数据

        */

        fun loadMoreData()

}

```

![Presenter](/images/借Kotlin探索MVP、RxJava/Presenter.jpg)

## MVP运用

```

class HomeFragment : BaseFragment(), HomeContract.View {

//Losped：延迟委托，类似懒加载，第一次调用 get() 会执行已传递给 lazy() 的 lamda 表达式并记录结果， 后续调用 get() 只是返回已得到的结果。

private val mPresenter by lazy { HomePresenter()}

override fun initView() {

mPresenter.attachView(this)

}

override fun lazyLoad() {

mPresenter.requestHomeData(num)//请求数据

}

override fun onDestroy() {

super.onDestroy()

mPresenter.detachView()

}

}

```

```

class HomePresenter : BasePresenter(), HomeContract.Presenter {

private var bannerHomeBean: HomeBean? =null

    private var nextPageUrl:String?=null    //加载首页的Banner 数据+一页数据合并后，nextPageUrl没add

    private val homeModel: HomeModelby lazy {

        HomeModel()

}

    /**

    * 获取首页精选数据 banner 加 一页数据

    */

    override fun requestHomeData(num: Int) {

// 检测是否绑定View

        checkViewAttached()

mRootView?.showLoading()

val disposable =homeModel.requestHomeData(num)

.flatMap({ homeBean->

                    Log.i("TAG","HomeBean格式2：" + homeBean.toString())

//过滤掉 Banner2(包含广告,等不需要的 Type), 具体查看接口分析

                    val bannerItemList = homeBean.issueList[0].itemList

                    //Losped :filter过滤集合

                    bannerItemList.filter { item->

                        item.type=="banner2"|| item.type=="horizontalScrollCard"

                    }.forEach{ item->

                        //移除item

                        bannerItemList.remove(item)

}

                    Log.i("TAG","HomeBean格式3：" + homeBean.issueList[0].itemList.size)

Log.i("TAG","HomeBean格式3：" + homeBean.issueList[0].itemList.toString())

bannerHomeBean = homeBean//记录第一页是当做 banner 数据

                    //根据 nextPageUrl 请求下一页数据

                    homeModel.loadMoreData(homeBean.nextPageUrl)//Losped默认最后一行return

                })

//Losped：

                // 匿名内部类，参数→返回值，(new Consumer() { @Override public void accept(@NonNull MobileAddress data) throws Exception {}}

                // mRootView?.apply：调用某对象的apply函数，在函数块内可以通过 this 指代该对象。返回值为该对象自己。

                .subscribe({ homeBean->

                    mRootView?.apply {

                        dismissLoading()

nextPageUrl = homeBean.nextPageUrl

                        //过滤掉 Banner2(包含广告,等不需要的 Type), 具体查看接口分析

                        val newBannerItemList = homeBean.issueList[0].itemList

                        newBannerItemList.filter { item->

                            item.type=="banner2"||item.type=="horizontalScrollCard"

                        }.forEach{ item->

                            //移除item

                            newBannerItemList.remove(item)

}

                        // 重新赋值 Banner 长度

                        bannerHomeBean!!.issueList[0].count =bannerHomeBean!!.issueList[0].itemList.size

                        //赋值过滤后的数据 + banner 数据

                        bannerHomeBean?.issueList!![0].itemList.addAll(newBannerItemList)

Log.i("TAG","HomeBean格式4：" +bannerHomeBean!!.issueList[0].itemList.size)

Log.i("TAG","HomeBean格式4：" +bannerHomeBean!!.issueList[0].itemList.toString())

setHomeData(bannerHomeBean!!)

}

                },{ t->

                    mRootView?.apply {

                        dismissLoading()

showError(ExceptionHandle.handleException(t),ExceptionHandle.errorCode)

}

                })

addSubscription(disposable)///用于取消所有正在执行的订阅

    }

/**

    * 加载更多

    */

    override fun loadMoreData() {

val disposable =nextPageUrl?.let {

            homeModel.loadMoreData(it)

.subscribe({ homeBean->

                        mRootView?.apply {

                            //过滤掉 Banner2(包含广告,等不需要的 Type), 具体查看接口分析

                            val newItemList = homeBean.issueList[0].itemList

                            newItemList.filter { item->

                                item.type=="banner2"||item.type=="horizontalScrollCard"

                            }.forEach{ item->

                                //移除item

                                newItemList.remove(item)

}

                            nextPageUrl = homeBean.nextPageUrl

                            setMoreData(newItemList)

}

                    },{ t->

                        mRootView?.apply {

                            showError(ExceptionHandle.handleException(t),ExceptionHandle.errorCode)

}

                    })

}

        if (disposable !=null) {

addSubscription(disposable)

}

}

}

```

```

class HomeModel{

/**

    * 获取首页 Banner 数据

    */

    fun requestHomeData(num:Int):Observable{

return RetrofitManager.service.getFirstHomeData(num)

.compose(SchedulerUtils.ioToMain())//重写 Transformer类，貌似用于线程变换

    }

/**

    * 加载更多

    */

    fun loadMoreData(url:String):Observable{

return RetrofitManager.service.getMoreHomeData(url)

.compose(SchedulerUtils.ioToMain())

}

}

```

```

data class HomeBean(val issueList: ArrayList,val nextPageUrl: String,val nextPublishTime: Long,val newestIssueType: String,val dialog: Any){

data class Issue(val releaseTime:Long,val type:String,val date:Long,val total:Int,val publishTime:Long,val itemList:ArrayList,var count:Int,val nextPageUrl:String){

data class Item(val type: String,val data: Data?,val tag: String) : Serializable {

data class Data(val dataType: String,

val text: String,

val videoTitle: String,

val id: Long,

val title: String,

val slogan: String?,

val description: String,

val actionUrl: String,

val provider: Provider,

val category: String,

val parentReply: ParentReply,

val author: Author,

val cover: Cover,

val likeCount:Int,

val playUrl: String,

val thumbPlayUrl: String,

val duration: Long,

val message: String,

val createTime:Long,

val webUrl: WebUrl,

val library: String,

val user: User,

val playInfo: ArrayList?,

val consumption: Consumption,

val campaign: Any,

val waterMarks: Any,

val adTrack: Any,

val tags: ArrayList,

val type: String,

val titlePgc: Any,

val descriptionPgc: Any,

val remark: String,

val idx: Int,

val shareAdTrack: Any,

val favoriteAdTrack: Any,

val webAdTrack: Any,

val date: Long,

val promotion: Any,

val label: Any,

val labelList: Any,

val descriptionEditor: String,

val collected: Boolean,

val played: Boolean,

val subtitles: Any,

val lastViewTime: Any,

val playlists: Any,

val header: Header,

val itemList:ArrayList

) : Serializable {

data class Tag(val id: Int,val name: String,val actionUrl: String,val adTrack: Any) : Serializable

data class Author(val icon: String,val name: String,val description: String) : Serializable

data class Provider(val name: String,val alias: String,val icon: String) : Serializable

data class Cover(val feed: String,val detail: String,

val blurred: String,val sharing: String,val homepage: String) : Serializable

data class WebUrl(val raw: String,val forWeibo: String) : Serializable

data class PlayInfo(val name: String,val url: String,val type: String,val urlList:ArrayList) : Serializable

data class Consumption(val collectionCount: Int,val shareCount: Int,val replyCount: Int) : Serializable

data class User(val uid: Long,val nickname: String,val avatar: String,val userType: String,val ifPgc: Boolean) : Serializable

data class ParentReply(val user: User,val message: String) : Serializable

data class Url(val size: Long) : Serializable

data class Header(val id: Int,val icon: String,val iconType: String,val description: String,val title: String,val font: String,val cover: String,val label: Label,

val actionUrl: String ,val subtitle:String,val labelList: ArrayList): Serializable{

data class Label(val text: String,val card: String,val detial: Any,val actionUrl: Any)

}

}

}

}

}

```

# Kotlin

Kotlin 是一种在 Java 虚拟机上运行的静态类型编程语言，被称之为 Android 世界的Swift，由 JetBrains 设计开发并开源。

Kotlin 可以编译成Java字节码，也可以编译成 JavaScript，方便在没有 JVM 的设备上运行。

在Google I/O 2017中，Google 宣布 Kotlin 成为 Android 官方开发语言。Kotlin更像是对Java的包装。

很多时候Kotlin优雅的语法糖大大减少了Java繁杂冗长的表达，顺应响应式编程的趋势，相信未来会成为开发者喜欢的语言之一。

## Kotlin与Java相互转换

AS为例：

Kotlin 转 Java

Tools>Kotlin>Show Kotlin Bytecode

Decompile

Java 转 Kotlin

Code - Convert Java File To Kotlin File 

## package打包与import导包

如果没有任何包声明的话，则当中的代码都属于默认包，导入时包名即为函数名！
比如：import shortToast

## Kotlin的函数fun

这里只写几种特殊的写法：

表达式作为函数体，返回类型自动推断：

```

fun sum(a: Int, b: Int) = a + b

public fun sum(a: Int, b: Int): Int = a + b // public 方法则必须明确写出返回类型

```

函数的变长参数可以用 vararg 关键字进行标识：

```

fun vars(vararg v:Int){

  for(vt in v){

     print(vt) 

   }

}

```

lambda(匿名函数)：

val sumLambda: (Int, Int) -> Int = {x,y -> x+y}

## Kotlin的常量与变量

可变变量定义：var 关键字

不可变变量定义：val 关键字，只能赋值一次的变量(类似Java中final修饰的变量)

编译器支持自动类型判断,即声明时可以不指定类型,由编译器判断。

```

val a: Int = 1

val b = 1

```

## Kotlin特有的NULL检查机制

```

//类型后面加?表示可为空

var age: String? = "23"

 //抛出空指针异常

val ages = age!!.toInt()

//不做处理返回 null

val ages1 = age?.toInt()

//age为空返回-1

val ages2 = age?.toInt() ?: -1

```

当一个引用可能为 null 值时, 对应的类型声明必须明确地标记为可为 null。

```

un parseInt(str: String): Int? { // ...}

```

## 语法-区间

区间表达式由具有操作符形式 .. 的 rangeTo 函数辅以 in 和 !in 形成。

```

// 使用 step 指定步长

for (i in 1..4 step 2) print(i)  // 输出“13”

for (i in 4 downTo 1 step 2) print(i)  // 输出“42”

// 使用 until 函数排除结束元素

for (i in 1 until 10) { // i in [1, 10) 排除了 10 println(i)}

```

## 数据类型-类型转换

toByte(): Byte

toShort(): Short

toInt(): Int

toLong(): Long

toFloat(): Float

toDouble(): Double

toChar(): Char

## 数据类型-位操作符

shl(bits) – 左移位 (Java’s <<)

shr(bits) – 右移位 (Java’s >>)

ushr(bits) – 无符号右移位 (Java’s >>>)

and(bits) – 与

or(bits) – 或

xor(bits) – 异或

inv() – 反向

## 数据类型-数组

数组的创建两种方式：一种是使用函数arrayOf()；另外一种是使用工厂函数。

```

fun main(args: Array<String>) 

{

 //[1,2,3]

 val a = arrayOf(1, 2, 3)

 //[0,2,4]

 val b = Array(3, { i -> (i * 2) }) 

 //读取数组内容 

 println(a[0]) // 输出结果：1 

 println(b[1]) // 输出结果：2}

```

## 数据类型-字符串

otlin 支持三个引号 """ 扩起来的字符串，支持多行字符串；String 可以通过 trimMargin() 方法来删除多余的空白。

```

fun main(args: Array<String>) { val text = """

    |多行字符串

    |菜鸟教程

    |多行字符串

    |Runoob

    """.trimMargin()    println(text)    // 前置空格删除了}

```

**字符串模板**，字符串可以包含模板表达式 ，即一些小段代码，会求值并把结果合并到字符串中。 

```

fun main(args: Array<String>) {

 val s = "runoob"

 val str = "$s.length is ${s.length}"   // 求值结果为 "runoob.length is 6" 

 println(str)

}

```

## 语法-条件控制

可以把 **IF 表达式**的结果赋值给一个变量

```

val max = if (a > b) {

 print("Choose a") a

} else { 

 print("Choose b") b

}

```

**When表达式**

```

when (x) { 

 0, 1 -> print("x == 0 or x == 1")

 in 1..10 -> print("x is in the range")

 is String -> x.startsWith("prefix")

 else -> print("otherwise")

}

```

## 语法-循环控制

**For 循环**

```

for (i in array.indices) {

 print(array[i])

}

```

while 与 do...while 循环

**标签处返回**

这个 return 表达式从最直接包围它的函数即 foo 中返回。 （注意，这种非局部的返回只支持传给内联函数的 lambda 表达式。） 如果我们需要从 lambda 表达式中返回，我们必须给它加标签并用以限制 return。

```

fun foo() {

    ints.forEach lit@ {

        if (it == 0) return@lit

        print(it)    }

}

```

现在，它只会从 lambda 表达式中返回。通常情况下使用隐式标签更方便。 该标签与接受该 lambda 的函数同名。

```

fun foo() {

    ints.forEach {

        if (it == 0) return@forEach

        print(it)    }

}

```

或者，我们用一个匿名函数替代 lambda 表达式。 匿名函数内部的 return 语句将从该匿名函数自身返回

```

fun foo() {

    ints.forEach(fun(value: Int) { 

       if (value == 0) return

        print(value)   

 })

}

```

## 类和对象

kotlin的类和属性默认为final，若不需要可用open修饰

### 类的构造函数

创建类实例

```

val site = Runoob() // Kotlin 中没有 new 关键字

```

##### 正常构造函数
**主构造器**

Koltin 中的类可以有一个 **主构造器**，以及一个或多个次构造器，主构造器是类头部的一部分，位于类名称之后，使用init代码块进行初始化：
```
class Person constructor(firstName: String) {
  init {
        this.name = firstName
    }
}

```
如果主构造器没有任何注解，也没有任何可见度修饰符，那么constructor关键字**可以省略**。
```

class Person(firstName: String) {}
```

**次构造函数**

如果类有主构造函数，每个次构造函数都要，或直接或间接通过另一个次构造函数代理主构造函数。在同一个类中代理另一个构造函数使用 this 关键字：

```

class Person(val name: String) {    constructor (name: String, age:Int) : this(name) {        // 初始化...    }}

```

如果一个非抽象类没有声明构造函数(主构造函数或次构造函数)，它会产生一个没有参数的构造函数。构造函数是 public 。如果你不想你的类有公共的构造函数，你就得声明一个空的主构造函数：

```

class DontCreateMe private constructor () {}

```

##### 多重构造函数
```
class MyBean {
    var name: String = ""

    var age: Int = 0
    //缺省
    var sex = false

    /**
     * 1. 多重构造函数需要有一个主函数，和N个次函数
     * 2. 次函数将委托给主函数
     * 3. 委托关系用this关键词表示
     */
    //主函数
    constructor()
    //次函数
    constructor(name:String):this(){
        this.name = name
    }
    //次函数
    constructor(name:String,age:Int,sex:Boolean):this(){
        this.name = name
        this.age = age
        this.sex = sex
    }

}
```

### 类的属性

getter 和 setter，field代表该变量本身

```

class Person {

 var lastName: String = "zhang"

 get() = field.toUpperCase() // 将变量赋值后转换为大写

 set 

 var no: Int = 100 

 get() = field // 后端变量 

 set(value) { 

 if (value < 10) {             // 如果传入的值小于 10 返回该值 field = value

            } else {

                field = -1        // 如果传入的值大于等于 10 返回 -1            

        }        

}    

var heiht: Float = 145.4f

        private set

}

```

### 类的类型

##### 抽象类

类本身，或类中的部分成员，都可以声明为abstract。抽象成员在类中不存在具体的实现（纯虚函数）。注意：无需对抽象类或抽象成员标注open注解。

```

open class Base {

 open fun f() {}

}

abstract class Derived : Base() {

 override abstract fun f()

}

```

![abtract](/images/借Kotlin探索MVP、RxJava/abtract.jpg)

##### 接口

Kotlin的接口允许有方法有默认实现：

```

interface MyInterface {

fun bar()// 未实现

    fun foo() {//已实现

        // 可选的方法体

        println("foo")

}

}

```

**接口中的属性**

接口中的属性只能是抽象的，不允许初始化值，接口不会保存属性值，实现接口时，必须重写属性：

```

interface MyInterface{

var name:String//name 属性, 抽象的

}

class MyImpl:MyInterface{

override var name: String ="runoob" //重载属性

}

```

**函数重写**

```

interface A {

fun foo() {print("A") }// 已实现

    fun bar()// 未实现，没有方法体，是抽象的

}

interface B {

fun foo() {print("B") }// 已实现

    fun bar() {print("bar") }// 已实现

}

class C : A {

override fun bar() {print("bar") }// 重写

}

class D : A, B {

override fun foo() {

super.foo()

super**.foo()**

}

override fun bar() {

super<B>.bar()

}

}

fun main(args: Array) {

val d =  D()

d.foo();

d.bar();

}

```

##### 嵌套类

我们可以把类嵌套在其他类中，看以下实例：

```

class Outer {                  // 外部类

    private val bar: Int = 1

    class Nested {            // 嵌套类

        fun foo() = 2

    }

}

fun main(args: Array) {

    val demo = Outer.Nested().foo() // 调用格式：外部类.嵌套类.嵌套类方法/属性

    println(demo)    // == 2

}

```

##### 内部类

内部类使用**inner** 关键字来表示。

**内部类与嵌套类的区别**在于，内部类会带有一个对外部类的对象的引用，并可访问外部类，所以内部类可以访问外部类成员属性和成员函数。

```

class Outer {

    private val bar: Int = 1

    var v = "成员属性"

    /**嵌套内部类**/

    inner class Inner {

        fun foo() = bar  // 访问外部类成员

        fun innerTest() {

            var o = this@Outer //获取外部类的成员变量

            println("内部类可以引用外部类的成员，例如：" + o.v)

}

}

}

fun main(args: Array) {

    val demo = Outer().Inner().foo()

    println(demo) //  1

    val demo2 = Outer().Inner().innerTest()

    println(demo2)  // 内部类可以引用外部类的成员，例如：成员属性

}

```

为了消除歧义，要访问来自外部作用域的this，我们使用this@label，其中@label 是一个 代指this 来源的标签。

##### 匿名内部类

使用对象表达式来创建匿名内部类：

```

class Test {

    var v = "成员属性"

    fun setInterFace(test: TestInterFace) {

test.test()

}

}

/**

* 定义接口

*/

interface TestInterFace {

    fun test()

}

fun main(args: Array) {

    var test = Test()

    /**

    * 采用对象表达式来创建接口对象，即匿名内部类的实例。

    */

    test.setInterFace(object : TestInterFace {

        override fun test() {

            println("对象表达式创建匿名内部类的实例")

}

})

}

```

##### 数据类

Kotlin 可以创建一个只包含数据的类，关键字为 data。

为了保证生成代码的一致性以及有意义，数据类需要满足以下条件：

主构造函数至少包含一个参数。

所有的主构造函数的参数必须标识为val 或者 var ;

数据类不可以声明为 abstract, open, sealed 或者 inner;

数据类不能继承其他类 (但是可以实现接口)。

**复制**

```

data class User(val name: String,val age: Int)

fun main(args: Array) {

val jack = User(name = "Jack",age = 1)

val olderJack = jack.copy(age = 2)

println(jack)

println(olderJack)

}

```

**解构声明**

```

val jane = User("Jane",35)

val (name, age) = jane

println("$name, $age years of age")// prints "Jane, 35 years of age"

```

##### **密封类**

密封类用来表示受限的类继承结构：当一个值为有限几种的类型, 而不能有任何其他类型时。在某种意义上，他们是枚举类的扩展：枚举类型的值集合 也是受限的，**但每个枚举常量只存在一个实例，而密封类 的一个子类可以有可包含状态的多个实例**。

声明一个密封类，使用 **sealed **修饰类，密封类可以有子类，但是所有的子类都必须要内嵌在密封类中。

sealed 不能修饰 interface ,abstract class(会报 warning,但是不会出现编译错误)

**使用密封类的关键好处在于使用when 表达式 的时候，如果能够 验证语句覆盖了所有情况，就不需要为该语句再添加一个else 子句了。**

```

sealed class UiOp {

object Show: UiOp()

object Hide: UiOp()

class TranslateX(val px: Float): UiOp()

class TranslateY(val px: Float): UiOp()

}

fun execute(view: View, op: UiOp) =when (op) {

UiOp.Show -> view.visibility = View.VISIBLE

UiOp.Hide -> view.visibility = View.GONE

is UiOp.TranslateX -> view.translationX = op.px// 这个 when 语句分支不仅告诉 view 要水平移动，还告诉 view 需要移动多少距离，这是枚举等 Java 传统思想不容易实现的

    is UiOp.TranslateY -> view.translationY = op.px

}

```

# Kotlin 扩展

## 扩展函数

扩展函数可以在已有类中添加新的方法，不会对原类做修改：

```

class User(var name:String)

/**扩展函数**/

fun User.Print(){

print("用户名 $name")

}

fun main(arg:Array){

var user = User("Runoob")

user.Print()

}

```

**扩展函数是静态解析的**

具体被调用的的是哪一个函数，由**调用函数的的对象表达式**来决定的，而不是动态的类型决定的:

```

open class C

class D: C()

fun C.foo() ="c"  // 扩展函数foo

fun D.foo() ="d"  // 扩展函数foo

fun printFoo(c: C) {

println(c.foo())// 类型是 C 类

}

fun main(arg:Array){

printFoo(D())

}

```

**除了函数，Kotlin 也支持属性对属性进行扩展，扩展属性只能被声明为val**

```

val <T> List<T>.lastIndex: Int

    get() = size - 1

```

**如果一个类定义有一个伴生对象 ，你也可以为伴生对象定义扩展函数和属性。**

**伴生对象通过"类名."形式调用伴生对象**，伴生对象声明的扩展函数，通过用类名限定符来调用：

```

class MyClass {

    companion object { }  // 将被称为"Companion"

}

fun MyClass.Companion.foo() {

    println("伴随对象的扩展函数")

}

val MyClass.Companion.no: Int

    get() = 10

fun main(args: Array) {

    println("no:${MyClass.no}")

    MyClass.foo()

}

```

**在一个类内部你可以为另一个类声明扩展。**

**以成员的形式定义的扩展函数, 可以声明为 open , 而且可以在子类中覆盖. 也就是说, 在这类扩展函数的派 发过程中, 针对分发接受者(扩展者老大)是虚拟的(virtual), 但针对扩展接受者（被扩展者）仍然是静态的。**

```

open class D{

}

class D1 : D(){

}

open class C {

open fun D.foo() {

println("D.foo in C")

}

open fun D1.foo() {

println("D1.foo in C")

}

fun caller(d: D) {

d.foo()// 调用扩展函数，由D决定调用D的扩展函数

    }

}

class C1 : C() {

override fun D.foo() {

println("D.foo in C1")

}

override fun D1.foo() {

println("D1.foo in C1")

}

}

fun main(args: Array) {

C().caller(D())// 输出"D.foo in C"

    C1().caller(D())// 输出 "D.foo in C1" —— 分发接收者虚拟解析

    C().caller(D1())// 输出 "D.foo in C" —— 扩展接收者静态解析

}

```

**要使用所定义包之外的一个扩展, 通过import导入扩展的函数名进行使用:**

```

package com.example.usage

import foo.bar.goo // 导入所有名为 goo 的扩展

// 或者

import foo.bar.*  // 从 foo.bar 导入一切

fun usage(baz: Baz) {

baz.goo()

}

```

## 泛型约束

最常见的约束是上界(upper bound)：

```

fun **<T : Comparable<T>>** sort(list: List<T>) {

    // ……

}

```

Comparable 的子类型可以替代T。 例如:

```

sort(listOf(1, 2, 3)) // OK。Int 是 Comparable<Int> 的子类型

sort(listOf(HashMap<Int, String>())) // 错误：HashMap<Int, String> 不是 Comparable<HashMap<Int, String>> 的子类型

```

对于多个上界约束条件，可以用 where 子句：

```

fun copyWhenGreater(list: List, threshold:T): List

**where T : CharSequence,**

**T : Comparable {**

**return list.filter { it > threshold}.map { it.toString()}**

**}**

```

##### 型变约束

**声明处型变**

**使用out 使得一个类型参数协变，协变类型参数只能用作输出，可以作为返回值类型但是无法作为入参的类型：**

```

// 定义一个支持协变的类

class Runoob<out A>(val a: A) {

    fun foo(): A {

        return a

}

}

fun main(args: Array) {

    var strCo: Runoob<String> = Runoob("a")

    var anyCo: Runoob<Any> = Runoob<Any>("b")

anyCo = strCo

    println(anyCo.foo())  // 输出a

}

in 使得一个类型参数逆变，逆变类型参数只能用作输入，可以作为入参的类型但是无法作为返回值的类型：

// 定义一个支持逆变的类

class Runoob<in A>(a: A) {

    fun foo(a: A) {

}

}

fun main(args: Array) {

    var strDCo = Runoob("a")

    var anyDCo = Runoob<Any>("b")

strDCo = anyDCo

}

```

##### 星号投射

暂时还未接触

## 对象

Kotlin中，对象与类的使用还是有区别的。(对象声明相当于静态类)

##### 对象表达式

**使用类的定义建立对象**

```

fun main(args: Array) {

val site =object {

var name: String ="菜鸟教程"

        var url: String ="www.runoob.com"

    }

println(site.name)

println(site.url)

}

```

**匿名内部类，在对象表达中可以方便的访问到作用域中的其他变量:**

```

fun countClicks(window: JComponent) {

var clickCount =0

    var enterCount =0

    window.addMouseListener(object : MouseAdapter() {

override fun mouseClicked(e: MouseEvent) {

clickCount++

}

override fun mouseEntered(e: MouseEvent) {

enterCount++

}

})

// ……

}

```

请注意，匿名对象可以用作只在本地和私有作用域中声明的类型。如果你使用匿名对象作为公有函数的 返回类型或者用作公有属性的类型，那么该函数或属性的实际类型 会是匿名对象声明的超类型，如果你没有声明任何超类型，就会是 Any。在匿名对象 中添加的成员将无法访问。

```

class C {

// 私有函数，所以其返回类型是匿名对象类型

    private fun foo() =object {

val x: String ="x"

    }

// 公有函数，所以其返回类型是Any

    fun publicFoo() =object {

val x: String ="x"

    }

fun bar() {

        val x1 = foo().x        // 没问题

        val x2 = publicFoo().x// 错误：未能解析的引用“x”

    }

}

```

##### 对象声明

Kotlin使用 object 关键字来声明一个**对象**，Kotlin中通过这种对象声明来获得一个**单例**。
object声明的单例对象不能声明构造函数，因为单例对象只有一个实例，无需我们手动将它创建出来，因此自然不需要构造函数 

```

object DataProviderManager {

fun registerDataProvider(provider: DataProvider) {

// ……

    }

val allDataProviders: Collection

get() =// ……

}

```

引用该对象，我们直接使用其名称即可（类似Java中的静态）：

```

DataProviderManager.registerDataProvider(……)

```

与对象表达式不同，也区别于内部类和嵌套类，当对象声明在另一个类的内部时，这个对象并**不能通过外部类的实例访问到该对象**，而只能通过类名来访问，同样该对象也**不能直接访问到外部类的方法和变量**。

```

class Site {

var name ="菜鸟教程"

    object DeskTop{

var url ="www.runoob.com"

        fun showName(){

print{"desk legs $name"} // 错误，不能访问到外部类的方法和变量

        }

}

}

fun main(args: Array) {

var site = Site()

site.DeskTop.url // 错误，不能通过外部类的实例访问到该对象

    Site.DeskTop.url // 正确

}

```

##### 伴身对象

类内部的对象声明可以用 companion 关键字标记，这样它就与外部类关联在一起，我们就可以直接通过外部类访问到对象的内部元素。**可用这种方式实现类中的静态方法。而object的单例可实现彻底的静态类**

我们可以**省略掉该对象的对象名**，然后使用Companion 替代需要声明的对象名：

```

class MyClass {

    companion object {

}

}

val x = MyClass.Companion

```

注意：一个类里面只能声明一个内部关联对象，即关键字companion 只能使用一次。

请伴生对象的成员看起来像其他语言的静态成员，但在运行时他们仍然是真实对象的实例成员。例如还可以实现接口：

```

interface Factory<T> {

    fun create(): T

}

class MyClass {

    companion object : Factory {

        override fun create(): MyClass = MyClass()

}

}

```

## kotlin 委托

##### 类委托

类的委托即一个类中定义的方法实际是调用另一个类的对象的方法来实现的。

以下实例中派生类Derived 继承了接口Base 所有方法，并且委托一个传入的Base 类的对象来执行这些方法。

```

// 创建接口

interface Base {

    fun print()

}

// 实现此接口的被委托的类

class BaseImpl(val x: Int) : Base {

    override fun print() { print(x) }

}

// 通过关键字 by 建立委托类

class Derived(b: Base) : Base by b

fun main(args: Array) {

    val b = BaseImpl(10)

    Derived(b).print() // 输出10

}

```

在Derived 声明中，by 子句表示，将b 保存在Derived 的对象实例内部，而且编译器将会生成继承自Base 接口的所有方法, 并将调用转发给b。

##### 属性委托

定义一个被委托的类

该类需要包含getValue() 方法和setValue() 方法，且参数thisRef 为进行委托的类的对象，prop 为进行委托的属性的对象。

```

import kotlin.reflect.KProperty

// 定义包含属性委托的类

class Example {

    var p: String by Delegate()

}

// 委托的类

class Delegate {

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {

        return "$thisRef, 这里委托了 ${property.name} 属性"

    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {

        println("$thisRef 的 ${property.name} 属性赋值为 $value")

}

}

fun main(args: Array) {

    val e = Example()

    println(e.p)    // 访问该属性，调用 getValue() 函数

    e.p = "Runoob"  // 调用 setValue() 函数

    println(e.p)

}

输出结果为：

Example@433c675d, 这里委托了p 属性

Example@433c675d 的p 属性赋值为Runoob

Example@433c675d, 这里委托了p 属性

```

##### 标准委托

**延迟属性 Lazy**

lazy()是一个函数,接受一个 Lambda表达式作为参数,返回一个 Lazy 实例的函数，返回的实例可以作为实现延迟属性的委托： 第一次调用 get()会执行已传递给 lazy()的 lamda表达式并记录结果， 后续调用 get()只是返回记录的结果。**相当于惰性执行，第一次执行第二次直接调用。**

```

val lazyValue: Stringby lazy {

    println("computed!")// 第一次调用输出，第二次调用不执行

    "Hello"

}

fun main(args: Array) {

println(lazyValue)// 第一次执行，执行两次输出表达式

    println(lazyValue)// 第二次执行，只输出返回值

}

```

**可观察属性 Observable**

observable可以用于实现观察者模式。

Delegates.observable()函数接受两个参数:第一个是初始化值,第二个是属性值变化事件的响应器(handler)。

在属性赋值后会执行事件的响应器(handler)，它有三个参数：被赋值的属性、旧值和新值：

```

import kotlin.properties.Delegates

class User {

var name: Stringby Delegates.observable("初始值"){

        prop, old, new->

        println("旧值：$old -> 新值：$new")

}

}

fun main(args: Array) {

val user = User()

user.name ="第一次赋值"

    user.name ="第二次赋值"

}

执行输出结果：

旧值：初始值 ->新值：第一次赋值

旧值：第一次赋值 ->新值：第二次赋值

```

**把属性储存在映射中**

一个常见的用例是在一个映射（map）里存储属性的值。 这经常出现在像解析JSON 或者做其他"动态"事情的应用中。 在这种情况下，你可以使用映射实例自身作为委托来实现委托属性。

```

class Site(val map: Map) {

    val name: String by map

    val url: String  by map

}

fun main(args: Array) {

    // 构造函数接受一个映射参数

    val site = Site(mapOf(

            "name" to "菜鸟教程",

            "url"  to "www.runoob.com"

    ))

    // 读取映射值

    println(site.name)

    println(site.url)

}

执行输出结果：

菜鸟教程

www.runoob.com

```

**Not Null**

notNull适用于那些无法在初始化阶段就确定属性值的场合。

```

class Foo {

var notNullBar: Stringby Delegates.notNull<String>()

}

foo.notNullBar ="bar"

println(foo.notNullBar)

```

## 收尾

21点钟，疲倦，写不完了，上面的总结大多是参考自菜鸟联盟和一个github项目。之后有一些运用到的总结和知识整理；接下来还有 RxJava2的运用，下回待续O(∩

参考小视频项目：https://github.com/git-xuhao/KotlinMvp
