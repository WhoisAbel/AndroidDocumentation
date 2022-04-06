# Section 1

  
  **Room is a persistence library, part of the Android Jetpack.**

> Room provides an abstraction layer over SQLite to allow fluent database access while harnessing the full power of SQLite.

**Room is now considered as a better approach for data persistence than SQLiteDatabase. It makes it easier to work with SQLiteDatabase objects in your app, decreasing the amount of boilerplate code and verifying SQL queries at compile time.**

#### Why use Room?

- **Compile-time verification of SQL queries. each `@Query` and `@Entity` is checked at the compile time, that preserves your app from crash issues at runtime and not only it checks the only syntax, but also missing tables.**
- **Boilerplate code**
- **Easily integrated with other Architecture components (like LiveData)**

- **Streamlined database migration paths.**

#### [Setup](https://developer.android.com/training/data-storage/room#setup)

**To use Room in your app, add the following dependencies to your app's `build.gradle` file:**

```kotlin
    // Room
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"
    implementation "androidx.room:room-ktx:$room_version"
    testImplementation "androidx.room:room-testing:$room_version"
```

#### Primary components

- The **database** class that holds the database and serves as the main access point for the underlying connection to your app's persisted data.
- **Data entities** that represent tables in your app's database.
- **Data access objects DAOs** that provide methods that your app can use to query, update, insert, and delete data in the database.

![](/home/abolfazl/.config/marktext/images/2022-04-06-15-34-36-image.png)

#### Sample implementation

##### 1. Data entity

```kotlin
@Entity
data class User(
    @PrimaryKey val uid: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

##### 2. DAO

**`Dao` provides the methods that the rest of the app uses to interact with data in tables.**

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): List<User>

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    fun loadAllByIds(userIds: IntArray): List<User>

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
           "last_name LIKE :last LIMIT 1")
    fun findByName(first: String, last: String): User

    @Insert
    fun insertAll(vararg users: User)

    @Delete
    fun delete(user: User)
} 
```

##### 3. Database

- **The class must be annotated with a `@Database` annotation that includes an `entities` array that lists all of the data entities associated with the database.**
- **The class must be an abstract class that extends `RoomDatabase`.**
- **For each DAO class that is associated with the database, the database class must define an abstract method that has zero arguments and returns an instance of the DAO class.**

```kotlin
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

# ***Use***

```kotlin
val db = Room.databaseBuilder(
            applicationContext,
            AppDatabase::class.java, "database-name"
        ).build()
val userDao = db.userDao()
val users: List<User> = userDao.getAll()
```


