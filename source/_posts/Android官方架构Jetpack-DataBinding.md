---
title: Android官方架构Jetpack-DataBinding
date: 2020.06.22 16:13:59
tags: Android
categories: Android
---

再见findViewByID.

## 介绍
Data Binding库不仅灵活而且广泛兼容- 它是一个support库，因此你可以在所有的Android平台最低能到Android 2.1（API等级7+）上使用它。
需求：Android Plugin for Gradle 1.5.0-alpha1 或 更高版本。

## 环境
Gradle版本需求：Android Plugin for Gradle 1.5.0-alpha1 或 更高版本。
Android Studio版本需求：请确保您使用的是Android Studio的兼容版本。Android Studio的Data Binding插件需要Android Studio 1.3.0 或 更高版本

添加dataBinding到build.gradle(:project)
```
android {
    ....
    dataBinding {
        enabled = true    
    }    
}
```

## Data Binding Layout文件

##### Data Binding基本用法
用法示例
main_activity.xml
```
<?xml version="1.0" encoding="utf-8"?>

<!--layout标签, 被layout包围的标签才能使用Data Binding-->
<layout xmlns:android="http://schemas.android.com/apk/res/android">

   <!--data标签,type为POJO类-->
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">

       <!--引用data标签的内容填充-->
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

User.java
```
public class User {
   private final String firstName;
   private final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
   public String getFirstName() {
       return this.firstName;
   }
   public String getLastName() {
       return this.lastName;
   }
}
```

**Binding数据**
默认情况下，一个Binding类会基于layout文件的名称而产生，将其转换为Pascal case（译注：首字母大写的命名规范）并且添加“Binding”后缀。上述的layout文件是main_activity.xml，因此生成的类名是MainActivityBinding。
绑定之后，可以调用<variable>的get/set方法
创建bindings的方式有以下几种：

ActorCollectionFragment.java
```
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   //第一种
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.fragment_actor_list);
   User user = new User("Test", "User");
   binding.setUser(user);

   //第二种，FragmentActorListBinding是由fragment_actor_list.xml自动生成的
   FragmentActorListBinding binding = FragmentActorListBinding.inflate(inflater, container, false)
   User user = new User("Test", "User");
   binding.setUser(user);
   // this.user = user  //Kotlin写法

   //第三种，如果你在ListView或者RecyclerView adapter使用Data Binding时，ListItemBinding是由list_item.xml自动生成的。可以使用
   ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup,
   false);
   ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);

}
```

fragment_actor_list.xml
```
<layout xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable
            name="user"
            type="me.liangfei.databinding.data.entities.User"/>
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/actorListView"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_marginStart="8dp"
            android:layout_marginLeft="8dp"
            android:layout_marginTop="8dp"
            android:layout_marginEnd="8dp"
            android:layout_marginRight="8dp"
            android:layout_marginBottom="8dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"/>
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

##### Data Binding深入使用
```
<!--alias可以重命名-->
<!--class可用来重命名或放置在不同的包中-->
<data class="com.example.ContactItem>
    <import type="android.view.View"
    alias="Vista"/>
</data>

<!--import(导入类型)-->
<!--variable表示定义某种类型的属性-->
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>

<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>

<!--include引用并传入参数-->
<LinearLayout
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include layout="@layout/name"
        bind:user="@{user}"/>
    <include layout="@layout/contact"
        bind:user="@{user}"/>
</LinearLayout>
```

- **ViewStubs标签**
xml 中的 ViewStub 经过 binding 之后会转换成 ViewStubProxy。
可为 ViewStubProy 注册 ViewStub.OnInflateListener 事件做到渲染监听和延迟渲染。

ViewStubActivity.java
```
public class ViewStubActivity extends BaseActivity {
    private ActivityViewStubBinding mBinding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mBinding = DataBindingUtil.setContentView(this, R.layout.activity_view_stub);
        // ViewStub标签默认隐藏,可以做到延迟渲染
        mBinding.viewStub.setOnInflateListener(new ViewStub.OnInflateListener() {
            @Override
            public void onInflate(ViewStub stub, View inflated) {
                ViewStubBinding binding = DataBindingUtil.bind(inflated);
                User user = new User("liang", "fei");
                binding.setUser(user);
            }
        });

    }


    /**
     * Don't panic for red error reporting. Just ignore it and run the app. Surprise never ends.
     */
    public void inflateViewStub(View view) {
        // 手动渲染
        if (!mBinding.viewStub.isInflated()) {
            mBinding.viewStub.getViewStub().inflate();
        }
    }
}
```

activity_view_stub.xml
```
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin" >

        <Button
            android:text="Inflate the ViewStub"
            android:onClick="inflateViewStub"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <ViewStub
            android:id="@+id/view_stub"
            android:layout="@layout/view_stub"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

    </LinearLayout>
</layout>
```

view_stub.xml
```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>
        <variable name="user" type="me.liangfei.databinding.model.User" />
    </data>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="horizontal">

        <TextView
            android:id="@+id/firstName"
            android:text="@{user.firstName}"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <TextView
            android:id="@+id/lastName"
            android:text="@{user.lastName}"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
    </LinearLayout>
</layout>
```

- **Attribute setters**
自定义控件时，属性只要有set方法。就能在xml的标签中直接使用

activity_attribute_setters.xml
```
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <import type="me.liangfei.databinding.sample.attributesetter.AttributeSettersActivity"/>

        <variable
            name="activity"
            type="me.liangfei.databinding.sample.attributesetter.AttributeSettersActivity"/>

        <variable
            name="imageUrl"
            type="String"/>
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

<!--        即使属性没有在 declare-styleable 中定义，只要有 setter 方法，我们也可以直接使用
            app:firstName,app:lastName 进行赋值操作-->
<!--        如下面代码的firstName和lastName-->
        <me.liangfei.databinding.view.NameCard
            android:layout_width="match_parent"
            android:layout_height="200dp"
            android:layout_marginEnd="@dimen/largePadding"
            android:layout_marginLeft="@dimen/largePadding"
            android:layout_marginRight="@dimen/largePadding"
            android:layout_marginStart="@dimen/largePadding"
            android:gravity="center"
            app:age="27"
            app:firstName="@{@string/firstName}"
            app:lastName="@{@string/lastName}"/>

        <com.liangfeizc.avatarview.AvatarView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:error="@{@drawable/error}"
            app:imageUrl="@{imageUrl}"
            app:onClickListener="@{activity.avatarClickListener}"/>

    </LinearLayout>
</layout>
```

NameCard.java
```
public class NameCard extends LinearLayout {
    private int mAge;

    private TextView mFirstName;
    private TextView mLastName;

    public NameCard(Context context) {
        this(context, null);
    }

    public NameCard(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public NameCard(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);

        TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.NameCard);

        try {
            mAge = a.getInteger(R.styleable.NameCard_age, 0);
        } finally {
            a.recycle();
        }

        init();
    }

    private void init() {
        inflate(getContext(), R.layout.name_card, this);
        mFirstName = (TextView) findViewById(R.id.first_name);
        mLastName = (TextView) findViewById(R.id.last_name);
    }

    public void setFirstName(@NonNull final String firstName) {
        mFirstName.setText(firstName);
    }

    public void setLastName(@NonNull final String lastName) {
        mLastName.setText(lastName);
    }

    public void setAge(@IntRange(from=1) int age) {
        mAge = age;
    }
}
```

- @BindingAdapter
被 @BindingAdapter 注解的函数，表示设置一个自定义控件属性。会在生成的Bilding文件的executeBindings()时被调用。
我们只要 **用app命名空间+参数名** 就可以在xml中设定这个属性,当这个属性被赋值时，该函数会得到调用
一般用于对数据做一个加工处理。非常适合ImageView的使用环境

activity_attribute_setters.xml
```
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <import type="me.liangfei.databinding.sample.attributesetter.AttributeSettersActivity"/>

        <variable
            name="activity"
            type="me.liangfei.databinding.sample.attributesetter.AttributeSettersActivity"/>

        <variable
            name="imageUrl"
            type="String"/>
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <ImageView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:error="@{@drawable/error}"
            app:imageUrl="@{imageUrl}"
            app:onClickListener="@{activity.avatarClickListener}"/>

<!--        me.liangfei.databinding.sample.attributesetter.AttributeSettersActivity.loadImage(this.mboundView2, imageUrl, getDrawableFromResource(mboundView2, R.drawable.error));-->
    </LinearLayout>
</layout>
```

AttributeSettersActivity.java
```
public class AttributeSettersActivity extends BaseActivity {
    private ActivityAttributeSettersBinding mBinding;

    public View.OnClickListener avatarClickListener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Toast.makeText(AttributeSettersActivity.this, "Come on", Toast.LENGTH_SHORT).show();
            mBinding.setImageUrl(Randoms.nextImgUrl());
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mBinding = DataBindingUtil.setContentView(this, R.layout.activity_attribute_setters);
        mBinding.setActivity(this);
        mBinding.setImageUrl(Randoms.nextImgUrl());
    }

    // @BindingAdapter 绑定 app:imageUrl 和 app:error后
    // 当这两个属性被赋值时，该函数会得到调用(在Bilding文件执行executeBindings()时被调用)
    @BindingAdapter({"imageUrl", "error"})
    public static void loadImage(ImageView view, String url, Drawable error) {
        Log.d(App.Companion.getTAG(), "load image");
        Picasso.with(view.getContext()).load(url).error(error).into(view);
    }
}
```

- @BindingConversion
被 @BindingConversion 注解的函数，会将<data>中所有某类型的值转换为另一类型
**注意是符合该类型的全部转换，需要谨慎使用。**

activity_conversions.xml
```
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>
        <variable name="isError" type="androidx.databinding.ObservableBoolean" />
        <variable name="height" type="float" />
    </data>
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingBottom="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        android:paddingTop="@dimen/activity_vertical_margin">

        <!--android:layout_width="@{isError.get() ? @string/large : @string/small}"-->
        <View
            android:background="@{isError ? @color/red : @color/white}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:layout_height="@{height}" />

        <Button
            android:onClick="toggleIsError"
            android:text="@{isError ? @string/red : @string/white}"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    </LinearLayout>
</layout>
```

ConversionsActivity.java
```
public class ConversionsActivity extends BaseActivity {

    private ObservableBoolean mIsError = new ObservableBoolean();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        ActivityConversionsBinding binding =
                DataBindingUtil.setContentView(this, R.layout.activity_conversions);

        mIsError.set(true);

        binding.setIsError(mIsError);
        binding.setHeight(ScreenUtils.dp2px(this, 200));

    }

    public void toggleIsError(View view) {
        mIsError.set(!mIsError.get());
    }

    // @BindingConversion 类型转换，将<data>中所有INT类型赋值转为ColorDrawable类型
    @BindingConversion
    public static ColorDrawable convertColorToDrawable(int color) {
        return new ColorDrawable(color);
    }

    // @BindingAdapter 绑定app:layout_height,可对数据做进一度处理
    // 动态更新
    @BindingAdapter("layout_height")
    public static void setLayoutHeight(View view, float height) {
        ViewGroup.LayoutParams params = view.getLayoutParams();
        params.height = (int) height;
        view.setLayoutParams(params);
    }
    /** !!! Binding conversion should be forbidden, otherwise it will conflict with
     *  {@code android:visiblity} attribute.
     */
    /*
    @BindingConversion
    public static int convertColorToString(int color) {
        switch (color) {
            case Color.RED:
                return R.string.red;
            case Color.WHITE:
                return R.string.white;
        }
        return R.string.app_name;
    }*/
}
```


##### Data 对象
修改POJO不会导致UI更新。Data Binding的真正能力是当数据变化时，可以通知给你的Data对象。有三种不同的数据变化通知机制：Observable对象、ObservableFields以及observable collections。
当这些可观察Data对象​​绑定到UI，Data对象属性的更改后，UI也将自动更新。

- 实现Observable 接口
实现android.databinding.Observable接口的类可以允许附加一个监听器到Bound对象以便监听对象上的所有属性的变化。通过指定一个Bindable注解给getter以及setter内通知来完成的。
User.java
```
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }

   // 通过 Bindable注解 监听进行UI更新
   @Bindable
   public String getFirstName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```
BR 是编译阶段生成的一个类，功能与 R.java 类似，用 @Bindable 标记过 getter 方法会在 BR 中生成一个 entry。
实际上还是通过notifyPropertyChanged(BR.XXX)手动发送通知，更新 UI。

- 使用 Observable 类作属性
ObservableFields是自包含具有单个字段的observable对象。它有所有基本类型和一个是引用类型。要使用它需要在data对象中创建public final字段
User.java
```
private static class User {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
}
```

MainActivity.java
```
user.firstName.set("Google");
int age = user.age.get();
```

- 使用 Observable 集合类作属性
MainActivity.java
```
//使用Map
ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);

//使用List
ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);
```

MainActivity.xml
```
//使用Map
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
…
<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user["age"])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>

//使用List。注意需要导入索引文件Fields
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
</data>
…
<TextView
   android:text='@{user[Fields.LAST_NAME]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
<TextView
   android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```


动态Variables的使用
```
// 要强制执行，使用executePendingBindings()方法。
public void onBindViewHolder(BindingHolder holder, int position) {
   final T item = mItems.get(position);
   holder.getBinding().setVariable(BR.item, item);
   holder.getBinding().executePendingBindings();
}
```

##### Binding生成、属性Setters等进阶内容比较少用，反而会使Adapt复杂不少，待需要时再补充

参考文章：
https://blog.gokit.info/post/android-data-binding/
https://github.com/liangfeidotme/MasteringAndroidDataBinding
