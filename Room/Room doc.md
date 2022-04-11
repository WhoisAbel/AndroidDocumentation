# Section 1

  **Room is a persistence library, part of the Android Jetpack.**

> Room provides an abstraction layer over SQLite to allow fluent database access while harnessing the full power of SQLite.

**Room is now considered as a better approach for data persistence than SQLiteDatabase. It makes it easier to work with SQLiteDatabase objects in your app, decreasing the amount of boilerplate code and verifying SQL queries at compile time.**

#### Why use Room?

- **Compile-time verification of SQL queries. each `@Query` and `@Entity` is checked at the compile time, that preserves your app from crash issues at runtime and not only it checks the only syntax, but also missing tables.**

- **Boilerplate code**

- **Easily integrated with other Architecture components (like LiveData)**

- **Streamlined database migration paths.**

#### Major problems with SQLite usage are

- **There is no compile-time verification of raw SQL queries. For example, if you write a SQL query with a wrong column name that does not exist in real database then it will give exception during run time and you can not capture this issue during compile time.**

- **As your schema changes, you need to update the affected SQL queries manually. This process can be time-consuming and error-prone.**

- **You need to use lots of boilerplate code to convert between SQL queries and Java data objects (POJO).**
  
  #### Room vs SQLite

- **In the case of SQLite, There is `no compile-time verification` of raw SQLite queries. But in Room, there is SQL `validation at compile time`.**

- **You need to use lots of `boilerplate` code to convert between SQL queries and Java data objects. But, Room maps our database objects to Java Object without boilerplate code.**

- **As your schema changes, you need to update the affected SQL queries manually. Room solves this problem.**

- **Room is built to work with `LiveData` and `RxJava` for data observation, while SQLite does not.**

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

![](/home/abolfazl/Documents/GitHub/AndroidDocumentation/resources/Primary%20components.png)

### Sample implementation

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

# Section 2

#### Defining data using Room entities

**When you use the Room persistence library to store your app's data, you define entities to represent the objects that you want to store. Each entity corresponds to a table in the associated Room database, and each instance of an entity represents a row of data in the corresponding table.**

**`@Entity` — every model class with this annotation will have a mapping table in DB.**

```kotlin
@Entity(tableName = "users")
data class User(
    @PrimaryKey val id: Int,

    val firstName: String?,
    val lastName: String?
)
```

**Note:** By default, Room uses the class name as the database table name. If you want the table to have a different name, set the `tableName` property of the `@Entity` annotation.

- **`foreignKeys` — names of foreign keys**
- **`indices` — list of indicates on the table**
- **`primaryKeys` — names of entity primary keys**
- **`tableName`**

**`@PrimaryKey` — this annotation points the primary key of the entity.**

**Note:** If you need Room to assign automatic IDs to entity instances, set the `autoGenerate` property of `@PrimaryKey` to `true`.

```kotlin
@PrimaryKey(autoGenerate = true) val id: Int
```

**Note:** If you need instances of an entity to be uniquely identified by a combination of multiple columns, you can define a *composite primary key* by listing those columns in the `primaryKeys` property of `@Entity`:

```kotlin
@Entity(primaryKeys = ["firstName", "lastName"])
data class User(
    val firstName: String?,
    val lastName: String?
)
```

**`@ColumnInfo` — allows specifying custom information about column.**

**Note:** Room uses the field names as column names in the database by default. If you want a column to have a different name, add the `@ColumnInfo` annotation to the field and set the `name`property.

```kotlin
@Entity(tableName = "users")
data class User (
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

**`@Ignore` — field will not be persisted by Room.**

```kotlin
@Entity
data class User(
    @PrimaryKey val id: Int,
    val firstName: String?,
    val lastName: String?,
    @Ignore val picture: Bitmap?
)
```

**Note:** In cases where an entity inherits fields from a parent entity, it's usually easier to use the `ignoredColumns` property of the `@Entity` attribute:

```kotlin
open class User {
    var picture: Bitmap? = null
}

@Entity(ignoredColumns = ["picture"])
data class RemoteUser(
    @PrimaryKey val id: Int,
    val hasVpn: Boolean
) : User()
```

**`@Embeded` — nested fields can be referenced directly in the SQL queries.**

```kotlin
data class Coordinates(
    val latitude: Double,
    val longitude: Double
)

@Entity
data class Address(
    @PrimaryKey(autoGenerate = true)
    val id: Long,
    val street: String,
    @Embedded
    val coordinates: Coordinates  
)
```

**Its database table will have 3 columns: `id, street, latitude, longitude`**

**So if you have a query that returns `id, street, latitude, longitude`, Room will properly construct an `Address` class.**

**`@Fts3,@Fts4`**

**If your app requires very quick access to database information through full-text search (FTS), have your entities backed by a virtual table that uses either the FTS3 or FTS4.If your app requires very quick access to database information through full-text search (FTS), have your entities backed by a virtual table that uses either the FTS3 or FTS4.**

```kotlin
// Use `@Fts3` only if your app has strict disk space requirements or if you
// require compatibility with an older SQLite version.
@Fts4
@Entity(tableName = "users")
data class User(
    /* Specifying a primary key for an FTS-table-backed entity is optional, but
       if you include one, it must use this type and column name. */
    @PrimaryKey @ColumnInfo(name = "rowid") val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?
)
```

**Note:** FTS-enabled tables always use a primary key of type `INTEGER` and with the column name "rowid". If your FTS-table-backed entity defines a primary key, it **must** use that type and column name.

**`@Index`**

**If your app must support SDK versions that don't allow for using FTS3- or FTS4-table-backed entities, you can still index certain columns in the database to speed up your queries. To add indices to an entity, include the `indices` property within the `@Entity` annotation, listing the names of the columns that you want to include in the index or composite index.**

```kotlin
@Entity(indices = [Index(value = ["last_name", "address"])])
data class User(
    @PrimaryKey val id: Int,
    val firstName: String?,
    val address: String?,
    @ColumnInfo(name = "last_name") val lastName: String?,
    @Ignore val picture: Bitmap?
)
```

Sometimes, certain fields or groups of fields in a database must be unique. You can enforce this uniqueness property by setting the `unique` property of an `@Index` annotation to `true`. The following code sample prevents a table from having two rows that contain the same set of values for the `firstName` and `lastName` columns:

```kotlin
@Entity(indices = [Index(value = ["first_name", "last_name"],
        unique = true)])
data class User(
    @PrimaryKey val id: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?,
    @Ignore var picture: Bitmap?
)
```
