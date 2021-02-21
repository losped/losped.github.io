---
title: Android官方架构Jetpack-LiveData&ViewModel
date: 2020.07.02 15:26:59
tags: Android
categories: Android
---

具体使用看官方文档已经写的很详细：
https://developer.android.google.cn/topic/libraries/architecture/viewmodel
https://developer.android.google.cn/topic/libraries/architecture/livedata
https://blog.csdn.net/wenyingzhi/article/details/97129224

LiveData的观察者模式似乎与RxJava有异曲同工之妙，ViewModel则与MVP模式中Present的职责类似。

- 创建ViewModel的方式

1. 直接创建
如果没有定义Factory，内部会默认使用AndroidViewModelFactory；
这个Factory只能创建构造参数为空的ViewModel和AndroidViewModel
因此只能创建构造参数为空的ViewModel

MasterFragment.kt
```
class MasterFragment : Fragment() {
    // Use the 'by activityViewModels()' Kotlin property delegate
    // from the fragment-ktx artifact

    // 跟随activity的生命周期
    private val model: SharedViewModel by activityViewModels()
    private val model = ViewModelProviders.of(requireActivity()).get(SharedViewModel::class.java)
    // 跟随本Fragment的生命周期
    private val model = ViewModelProviders.of(MasterFragment.this).get(SharedViewModel::class.java)
    val model: MyViewModel by viewModels()
}
```

2. 通过ViewModelFactory创建
通过ViewModelFactory，可以创建携带构造参数的ViewModel

ActorCollectionFragment.kt
```
class ActorCollectionFragment : Fragment() {
    private lateinit var viewModel: ActorViewModel

    override fun onCreateView(
            inflater: LayoutInflater,
            container: ViewGroup?,
            savedInstanceState: Bundle?
    ): View? {
        ...
        val context = context ?: return binding.root
        val factory = InjectorUtils.provideActorViewModelFactory(context)

        // 获取ViewModel
        viewModel = ViewModelProviders.of(requireActivity(), factory)
                .get(ActorViewModel::class.java)
        // 换种写法
        viewModel = ViewModelProviders.of(this)[ActorViewModel::class.java]

        ...
        return binding.root
    }

    private fun subscribeUi(adapter: ActorListAdapter) {
        // 绑定数据
        viewModel.actors.observe(viewLifecycleOwner, Observer {
            adapter.submitList(it)
        })
    }
}
```

InjectorUtils.kt
```
object InjectorUtils {
    private fun getActorRepository(context: Context): ActorRepository {
        return ActorRepository.getInstance(
                AppDatabase.getInstance(context.applicationContext).actorDao())
    }

    fun provideActorViewModelFactory(
            context: Context
    ): ActorViewModelFactory {
        val repository = getActorRepository(context)
        return ActorViewModelFactory(repository)
    }
}
```

ActorViewModelFactory.kt
```
@Suppress("UNCHECKED_CAST")
// 新建一个Factory类继承自ViewModelProvider.NewInstanceFactory(),并实现create()方法
class ActorViewModelFactory(
        private val repository: ActorRepository
) : ViewModelProvider.NewInstanceFactory() {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return ActorViewModel(repository) as T
    }
}
```

ActorViewModel.kt
```
class ActorViewModel internal constructor(private val actorRepository: ActorRepository)
    : ViewModel() {
    val actors = actorRepository.getActors()
    fun actorDetail(actorId: Int) = actorRepository.getActor(actorId)
}
```

ActorRepository.kt
```
// 将Dao抽离Model，统一放在Repository中.
class ActorRepository private constructor(private val actorDao: ActorDao) {
    fun getActors() = actorDao.getActors()

    fun getActor(actorId: Int) = actorDao.getActor(actorId)

    // 单例模式
    companion object {
        @Volatile private var instance: ActorRepository? = null

        fun getInstance(actorDao: ActorDao) =
                instance ?: synchronized(this) {
                    instance ?: ActorRepository(actorDao).also { instance = it}
                }
    }
}
```
