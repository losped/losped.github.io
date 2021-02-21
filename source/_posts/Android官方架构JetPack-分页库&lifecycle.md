---
title: Android官方架构Jetpack-分页库&lifecycle
date: 2020.07.06 11:58:59
tags: Android
categories: Android
---

官方文档：
https://developer.android.google.cn/topic/libraries/architecture/paging#kotlin
https://developer.android.google.cn/topic/libraries/architecture/lifecycle

##### lifecycle
- LifecycleOwner
- LifecycleObserver
- 处理 ON_STOP 事件

##### 分页库
- DataSource和DataSource.Factory
1. Room的DataSource
2. 自定义DataSource

- AsyncListUtil
...

使用示例
```
@Dao
    interface ConcertDao {
        // The Int type parameter tells Room to use a PositionalDataSource
        // object, with position-based loading under the hood.
        @Query("SELECT * FROM concerts ORDER BY date DESC")
        fun concertsByDate(): DataSource.Factory<Int, Concert>
    }

    class ConcertViewModel(concertDao: ConcertDao) : ViewModel() {
        val concertList: LiveData<PagedList<Concert>> =
                concertDao.concertsByDate().toLiveData(pageSize = 50)
    }

    class ConcertActivity : AppCompatActivity() {
        public override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            // Use the 'by viewModels()' Kotlin property delegate
            // from the activity-ktx artifact
            val viewModel: ConcertViewModel by viewModels()
            val recyclerView = findViewById(R.id.concert_list)
            val adapter = ConcertAdapter()
            viewModel.concertList.observe(this, PagedList(adapter::submitList))
            recyclerView.setAdapter(adapter)
        }
    }

    class ConcertAdapter() :
            PagedListAdapter<Concert, ConcertViewHolder>(DIFF_CALLBACK) {
        fun onBindViewHolder(holder: ConcertViewHolder, position: Int) {
            val concert: Concert? = getItem(position)

            // Note that "concert" is a placeholder if it's null.
            holder.bindTo(concert)
        }

        companion object {
            private val DIFF_CALLBACK = object :
                    DiffUtil.ItemCallback<Concert>() {
                // Concert details may have changed if reloaded from the database,
                // but ID is fixed.
                override fun areItemsTheSame(oldConcert: Concert,
                        newConcert: Concert) = oldConcert.id == newConcert.id

                override fun areContentsTheSame(oldConcert: Concert,
                        newConcert: Concert) = oldConcert == newConcert
            }
        }
    }
```
