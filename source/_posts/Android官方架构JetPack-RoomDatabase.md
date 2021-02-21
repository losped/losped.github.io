---
title: Android官方架构Jetpack-RoomDatabase
date: 2020.06.30 15:43:59
tags: Android
categories: Android
---

分享两篇好文章：
https://blog.csdn.net/singwhatiwanna/article/details/104890202
https://www.jianshu.com/p/ffcbfcea0ad2


Room是SQLite之上的一个抽象层，通过@DataBase、@Entity和@Dao三个标签自动生成DataBae_Impl.java、Dao_Impl.java。
帮助我们包装了DataBase，数据表创建，增删改查接口的实现等工作，类似于Mybatis。
可以支持DAO直接返回LiveData、RxJava和Cursor类型的数据，十分强大。

- addCallback
提供了一个回调接口。可以在DataBase初始化完毕后，进行一些操作。

AppDatabase.kt
```
private fun buildDatabase(context: Context): AppDatabase {
    return Room.databaseBuilder(context, AppDatabase::class.java, DATABASE_NAME)
            .addCallback(object : RoomDatabase.Callback() {
                override fun onCreate(db: SupportSQLiteDatabase) {
                    super.onCreate(db)
                    // 初始化完毕，从json获取数据
                    val request = OneTimeWorkRequestBuilder<SeedDatabaseWorker>().build()
                    WorkManager.getInstance().enqueue(request)
                }
            })
            .build()
}
```
