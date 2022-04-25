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

![](../resources/Primary%20components.png)

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

# Section 3

#### Accessing data using Room DAOs - Part 1

**When you use the Room persistence library to store your app's data, you interact with the stored data by defining *data access objects*, or DAOs. Each DAO includes methods that offer abstract access to your app's database. At compile time, Room automatically generates implementations of the DAOs that you define.**

**You can define each DAO as either an interface or an abstract class. For basic use cases, you should usually use an interface. In either case, you must always annotate your DAOs with `@Dao`. DAOs don't have properties, but they do define one or more methods for interacting with the data in your app's database.**

```kotlin
@Dao
interface UserDao {
    @Insert
    fun insertAll(vararg users: User)

    @Delete
    fun delete(user: User)

    @Query("SELECT * FROM user")
    fun getAll(): List<User>
}
```

**There are two types of DAO methods that define database interactions:**

- **`Convenience methods` that let you insert, update, and delete rows in your database without writing any SQL code.**
- **`Query methods` that let you write your own SQL query to interact with the database.**

#### Convenience methods

**Room provides convenience annotations for defining methods that perform simple `inserts`, `updates`, and `deletes` without requiring you to write a SQL statement.**

**If you need to define more complex inserts, updates, or deletes, or if you need to query the data in the database, use a query method instead.**

#### `@Insert`

**The `@Insert` annotation allows you to define methods that insert their parameters into the appropriate table in the database.**

```kotlin
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    fun insertUsers(vararg users: User)

    @Insert
    fun insertBothUsers(user1: User, user2: User)

    @Insert
    fun insertUsersAndFriends(user: User, friends: List<User>)
}
```

**If the `@Insert` method receives a single parameter, it can return a `long` value, which is the new `rowId` for the inserted item. If the parameter is an array or a collection, then the method should return an array or a collection of `long` values instead, with each value as the `rowId` for one of the inserted items.**

- **When an `@Insert` method is called, Room inserts each passed entity instance into the corresponding database table.**

- **If the `@Insert` method receives a single parameter, it can return a `long` value, which is the new `rowId` for the inserted item. If the parameter is an array or a collection, then the method should return an array or a collection of `long` values instead, with each value as the `rowId` for one of the inserted items.**

#### `@Update`

**The `@Update` annotation allows you to define methods that update specific rows in a database table. Similarly to `@Insert` methods, `@Update` methods accept data entity instances as parameters.**

```kotlin
@Dao
interface UserDao {
    @Update
    fun updateUsers(vararg users: User)
}
```

- **Room uses the primary key to match passed entity instances to rows in the database. If there is no row with the same primary key, Room makes no changes.**

- **An `@Update` method can optionally return an `int` value indicating the number of rows that were updated successfully.**

#### `@Delete`

**The `@Delete` annotation allows you to define methods that delete specific rows from a database table. Similarly to `@Insert` methods, `@Delete` methods accept data entity instances as parameters.**

```kotlin
@Dao
interface UserDao {
    @Delete
    fun deleteUsers(vararg users: User)
}
```

- **Room uses the primary key  to match passed entity instances to rows in the database. If there is no row with the same primary key, Room makes no changes.**

- **A `@Delete` method can optionally return an `int` value indicating the number of rows that were deleted successfully.**

#### Query methods

- **The `@Query` annotation allows you to write SQL statements and expose them as DAO methods. Use these query methods to query data from your app's database, or when you need to perform more complex inserts, updates, and deletes.**

- **Room validates SQL queries at compile time. This means that if there's a problem with your query, a compilation error occurs instead of a runtime failure.**

#### Simple queries

**The following code defines a method that uses a simple `SELECT` query to return all of the `User` objects in the database:**

```kotlin
@Query("SELECT * FROM user")
fun loadAllUsers(): Array<User>
```

# Section 4

- **Most of the time, you only need to return a subset of the columns from the table that you are querying. For example, your UI might display just the first and last name for a user instead of every detail about that user. In order to save resources and streamline your query's execution, you should only query the fields that you need. Room allows you to return a simple object from any of your queries as long as you can map the set of result columns onto the returned object.**

```kotlin
data class NameTuple(
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

```kotlin
@Query("SELECT first_name, last_name FROM user")
fun loadFullName(): List<NameTuple>
```

- **Most of the time, your DAO methods need to accept parameters so that they can perform filtering operations. Room supports using method parameters as bind parameters in your queries.**

```kotlin
@Query("SELECT * FROM user WHERE age > :minAge")
fun loadAllUsersOlderThan(minAge: Int): Array<User>
```

- **You can also pass multiple parameters or reference the same parameter multiple times in a query.**

```kotlin
@Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
fun loadAllUsersBetweenAges(minAge: Int, maxAge: Int): Array<User>

@Query("SELECT * FROM user WHERE first_name LIKE :search " +
       "OR last_name LIKE :search")
fun findUserWithName(search: String): List<User>
```

- **Some of your DAO methods might require you to pass in a variable number of parameters that is not known until runtime. Room understands when a parameter represents a collection and automatically expands it at runtime based on the number of parameters provided.**

```kotlin
@Query("SELECT * FROM user WHERE region IN (:regions)")
fun loadUsersFromRegions(regions: List<String>): List<User>
```

#### Query multiple tables

- **Some of your queries might require access to multiple tables to calculate the result. You can use `JOIN` clauses in your SQL queries to reference more than one table.**

```kotlin
@Query(
    "SELECT * FROM book " +
    "INNER JOIN loan ON loan.book_id = book.id " +
    "INNER JOIN user ON user.id = loan.user_id " +
    "WHERE user.name LIKE :userName"
)
fun findBooksBorrowedByNameSync(userName: String): List<Book>
```

- **You can also define simple objects to return a subset of columns from multiple joined tables as discussed in Return a subset of a table's columns.**

```kotlin
interface UserBookDao {
    @Query(
        "SELECT user.name AS userName, book.name AS bookName " +
        "FROM user, book " +
        "WHERE user.id = book.user_id"
    )
    fun loadUserAndBookNames(): LiveData<List<UserBook>>

    // You can also define this class in a separate file.
    data class UserBook(val userName: String?, val bookName: String?)
}
```

#### Return a multimap

**In Room 2.4 and higher, you can also query columns from multiple tables without defining an additional data class by writing query methods that return a multimap.Consider the example from the Query multiple tables  section. Instead of returning a list of instances of a custom data class that holds pairings of `User` and `Book` instances, you can return a mapping of `User` and `Book`directly from your query method:**

```kotlin
@Query(
    "SELECT * FROM user" +
    "JOIN book ON user.id = book.user_id"
)
fun loadUserAndBookNames(): Map<User, List<Book>>
```

**When your query method returns a multimap, you can write queries that use `GROUP BY` clauses, allowing you to take advantage of SQL's capabilities for advanced calculations and filtering. For example, you can modify your `loadUserAndBookNames()` method to only return users with three or more books checked out:**

```kotlin
@Query(
    "SELECT * FROM user" +
    "JOIN book ON user.id = book.user_id" +
    "GROUP BY user.name WHERE COUNT(book.id) >= 3"
)
fun loadUserAndBookNames(): Map<User, List<Book>>
```

**If you don't need to map entire objects, you can also return mappings between specific columns in your query by setting the `keyColumn` and `valueColumn` attributes in a `@MapInfo` annotation on your query method:**

```kotlin
@MapInfo(keyColumn = "userName", valueColumn = "bookName")
@Query(
    "SELECT user.name AS username, book.name AS bookname FROM user" +
    "JOIN book ON user.id = book.user_id"
)
fun loadUserAndBookNames(): Map<String, List<String>
```

#### 

#### Direct cursor access

**If your app's logic requires direct access to the return rows, you can write your DAO methods to return a `Cursor` object as shown in the following example:**

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
    fun loadRawUsersOlderThan(minAge: Int): Cursor
}
```

#### 

#### Paginated queries with the Paging library

**Room supports paginated queries through integration with the Paging library. In `Room 2.3.0-alpha01` and higher, DAOs can return `PagingSource` objects for use with Paging 3.**

```kotlin
@Dao
interface UserDao {
  @Query("SELECT * FROM users WHERE label LIKE :query")
  fun pagingSource(query: String): PagingSource<Int, User>
}
```

# Section 5
