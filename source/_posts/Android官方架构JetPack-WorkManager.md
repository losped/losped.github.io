---
title: Android官方架构JetPack-WorkManager
date: 2020.07.01 14:22:59
tags: Android
categories: Android
---

分享好文章：
https://www.jianshu.com/p/e495ee6e84de
关于Kotlin协程：
https://blog.csdn.net/suyimin2010/article/details/92387717
https://www.jianshu.com/p/76d2f47b900d


Kotlin尚有许多需要深入的地方，留待下次学习.

- 创建WorkRequest新的方式
```
private fun buildDatabase(context: Context): AppDatabase {
    return Room.databaseBuilder(context, AppDatabase::class.java, DATABASE_NAME)
            .addCallback(object : RoomDatabase.Callback() {
                override fun onCreate(db: SupportSQLiteDatabase) {
                    super.onCreate(db)
                    // 直接以泛型方式传入Worker创建OneTimeWorkRequestBuilder
                    val request = OneTimeWorkRequestBuilder<SeedDatabaseWorker>().build()
                    // 加入任务队列
                    WorkManager.getInstance().enqueue(request)
                }
            })
            .build()
}
```

- CorotineWorker
内部会使用coroutineScope.launch创建一个协程任务
Corotine就是以任务为主体，将不同部分的任务通过挂起和恢复的方式按顺序给不同的线程执行（也可能是在同一线程）

CoroutineWorker.kt
```
final override fun startWork(): ListenableFuture<Result> {

    val coroutineScope = CoroutineScope(coroutineContext + job)
    coroutineScope.launch {
        try {
            val result = doWork()
            future.set(result)
        } catch (t: Throwable) {
            future.setException(t)
        }
    }

    return future
}
```

SeedDatabaseWorker.kt
```
class SeedDatabaseWorker(
        context: Context,
        workerParams: WorkerParameters
) : CoroutineWorker(context, workerParams) {
    private val TAG by lazy { SeedDatabaseWorker::class.java.simpleName }
    // Coroutine作用域
    override val coroutineContext = Dispatchers.IO

    // suspend 表示此协程可挂起，注意挂起的是整个协程，而非该方法
    // 调用 coroutineScope {}将代码块放入协程中
    override suspend fun doWork(): Result = coroutineScope {

        // 类似Rxjava的callback
        try {
            applicationContext.assets.open(ACTOR_DATA_FILENAME).use { inputStream ->
                JsonReader(inputStream.reader()).use { jsonReader ->
                    val actorType = object : TypeToken<List<Actor>>() {}.type
                    val actorList: List<Actor> = Gson().fromJson(jsonReader, actorType)

                    val database = AppDatabase.getInstance(applicationContext)
                    database.actorDao().insertAll(actorList)

                    Result.success()
                }
            }
        } catch (ex: Exception) {
            Log.e(TAG, "Error seeding database", ex)
            Result.failure()
        }
    }
}
```
