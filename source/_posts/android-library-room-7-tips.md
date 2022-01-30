---
title: Room数据库酷七条
date: 2022-01-30 22:01:39
tags:
categories:
description:
---


#### 一、通过`RoomDatabase#Callback`预填充数据

如果需要在数据库创建或数据库打开的时候添加一些默认数据，可以使用`RoomDatabase#Callback`接口并复写它的`onCreate`或`onOpen`方法。由于DAO对象只能在这两个方法返回后使用，所以我们创建一个新线程来插入数据：

```kotlin

Room.databaseBuilder(context.applicationContext,
        DataDatabase::class.java, "Sample.db")
        // prepopulate the database after onCreate was called
        .addCallback(object : Callback() {
            override fun onCreate(db: SupportSQLiteDatabase) {
                super.onCreate(db)
                // moving to a new thread
                ioThread {
                    getInstance(context).dataDao()
                                        .insert(PREPOPULATE_DATA)
                }
            }
        })
        .build()

```

**需要注意的是**：当app首次启动时，在create和insert之间崩溃的话，数据将永远不会被插入。

#### 二、使用DAO的继承能力

很多DAO里存在一样的`Insert`、`Update`、`Delete`方法。我们可以使用继承来避免这些重复的代码。

```kotlin
interface BaseDao<T> {
    @Insert
    fun insert(vararg obj: T)
}
@Dao
abstract class DataDao : BaseDao<Data>() {
    @Query("SELECT * FROM Data")
    abstract fun getData(): List<Data>
}
```


#### 三、通过`@Transaction`减少事务查询的模板代码

用`@Transaction`注解的方法会在一个事务里面执行。当这些方法执行异常时，事务也会失败。
```kotlin
@Dao
abstract class UserDao {
    
    @Transaction
    open fun updateData(users: List<User>) {
        deleteAllUsers()
        insertAll(users)
    }
    @Insert
    abstract fun insertAll(users: List<User>)
    @Query("DELETE FROM Users")
    abstract fun deleteAllUsers()
}
```

当`@Query`方法带有select语句时，在下列情况下你可能会使用`@Transation`注解

+ 当查询的结果集很大时，让查询在一次事务里完成能够保证：如果查询结果不满足单个游标窗口，它不会因为在游标窗口交换之间的数据库更改而中断。
+ 如果查询结果是一个带`@Relation`字段的POJO，此字段会单独查询，所以让他们运行在一个事务中可以确保结果一致。

`Delete`、`Update`、`Insert`方法包含多个参数时会自动在事务里执行。


#### 四、只读需要的数据

关注App的内存消耗，只加载需要使用的字段，这能提高查询速度减少IO消耗。
```kotlin
@Entity(tableName = "users")
data class User(@PrimaryKey
                val id: String,
                val userName: String,
                val firstName: String, 
                val lastName: String,
                val email: String,
                val dateOfBirth: Date, 
                val registrationDate: Date)
```

上面的类中，很多字段我们用不到，定义一个类，只包含我们需要的字段：
```kotlin
data class UserMinimal(val userId: String,
                       val firstName: String, 
                       val lastName: String)
```

在DAO中，定义方法只查询所需字段：
```kotlin
@Dao
interface UserDao {
    @Query(“SELECT userId, firstName, lastName FROM Users)
    fun getUsersMinimal(): List<UserMinimal>
}
```

#### 五、使用外键
即使Room不直接支持关系，但它允许你定义外键来约束实体之间的关系。

Room的`@ForeignKey`是`@Entity`注解的一部分，用来支持Sqlite的外键特性。它强制表之间的约束，当你修改表时，保证关系一致性。

看一下`User`和`Pet`类，`Pet`有主人，以userId关联。
```kotlin
@Entity(tableName = "pets",
        foreignKeys = arrayOf(
            ForeignKey(entity = User::class,
                       parentColumns = arrayOf("userId"),
                       childColumns = arrayOf("owner"))))
data class Pet(@PrimaryKey val petId: String,
              val name: String,
              val owner: String)
```

你可以定义一些行为，当例子中的`User`在数据库中被删除或修改。你可以做以下一些操作：`NO_ACTION`、`RESTRICT`、`SET_NULL`、`SET_DEFAULT`或`CASCADE`，这些操作和Sqlite保持一致。

**注意：** 在Room中`SET_DEFAULT`和`SET_NULL`效果一致，因为Room还不允许为数据列设置默认值。


#### 六、使用`@Relation`简化一对多查询

在前面`User-Pet`的例子中，我们有个一对多的关系，一个人可以有多个宠物。当我们想获取主人和宠物集合时：

```kotlin
data class UserAndAllPets (val user: User,
                           val pets: List<Pet> = ArrayList())
```

需要两个查询：
```kotlin
@Query(“SELECT * FROM Users”)
public List<User> getUsers();

@Query(“SELECT * FROM Pets where owner = :userId”)
public List<Pet> getPetsForUser(String userId);
```

我们需要遍历用户来查宠物。


为了简化此过程，Room的`@Relation`注解自动关联实体。此注解只能用于`List`和`Set`对象。

```kotlin
class UserAndAllPets {
   @Embedded
   var user: User? = null

   @Relation(parentColumn = “userId”,
             entityColumn = “owner”)
   var pets: List<Pet> = ArrayList()
}
```

在DAO中，我们定义一次查询：
```kotlin
@Transaction
@Query(“SELECT * FROM Users”)
List<UserAndAllPets> getUsers();
```


#### 七、避免可观察查询的假通知
可观察查询，只关心对应ID的对象：
```kotlin
    @Query(“SELECT * FROM Users WHERE userId = :id)
    fun getUserById(id: String): LiveData<User> 
    
    // or
    
    @Query(“SELECT * FROM Users WHERE userId = :id)
    fun getUserById(id: String): Flowable<User>
```

每次此用户更新的时候你都得到一个新通知。但是其他的一些不影响此ID的修改（delete、update、insert）会发送通知。

这种场景背后的原因是什么：
1、Sqlite支持触发器，`DELETE`、`UPDATE`、`INSERT`发生时或触发。
2、Room创建了一个`InvalidationTracker`来跟踪观察表的变化。
3、`LiveData`和`Flowable`查询依赖`InvalidationTracker.Observer#onInvalidated`通知，收到通知就做一次再查询操作。

Room只知道表被修改了，不知道为什么修改也不知道什么被修改。因此，再查询操作会重新通知。由于Room不在内存保存数据也不能保证object的equals方法所以他不知道对象是否变化了。

你需要自己保证DAO过滤收到的事件，只对关心的数据做出响应。

如果观察的是`Flowable`，使用`Flowable#distinctUntilChanged`。

```kotlin
@Dao
abstract class UserDao : BaseDao<User>() {
/**
* Get a user by id.
* @return the user from the table with a specific id.
*/
@Query(“SELECT * FROM Users WHERE userid = :id”)
protected abstract fun getUserById(id: String): Flowable<User>

    fun getDistinctUserById(id: String): 
    Flowable<User> = getUserById(id).distinctUntilChanged()
}
```

如果观察的是`LiveData`，可以用`MediatorLiveData`处理只让有变化的数据发送事件。
```kotlin
fun <T> LiveData<T>.getDistinct(): LiveData<T> {
    val distinctLiveData = MediatorLiveData<T>()
    distinctLiveData.addSource(this, object : Observer<T> {
        private var initialized = false
        private var lastObj: T? = null
        override fun onChanged(obj: T?) {
            if (!initialized) {
                initialized = true
                lastObj = obj
                distinctLiveData.postValue(lastObj)
            } else if ((obj == null && lastObj != null) 
                       || obj != lastObj) {
                lastObj = obj
                distinctLiveData.postValue(lastObj)
            }
        }
    })
    return distinctLiveData
}
```

在DAO中，让真正关心变化的方法使用`public`修饰，查询方法用`protected`修饰

```kotlin
@Dao
abstract class UserDao : BaseDao<User>() {
    @Query(“SELECT * FROM Users WHERE userid = :id”)
    protected abstract fun getUserById(id: String): LiveData<User>

    fun getDistinctUserById(id: String): LiveData<User> = getUserById(id).getDistinct()
}
```

**注意：** 如果你返回列表用于展示，考虑使用`Paging`库，它能帮你处理数据的变化。