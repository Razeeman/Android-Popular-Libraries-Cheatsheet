# Android Essential Libraries Cheatsheet

## Room

The Room persistence library provides an abstraction layer over SQLite to allow for more robust database access. Provides compile time verification of SQL queries.

```
dependencies {
    def room_version = "2.1.0-alpha04"
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"
    // For Kotlin use kapt instead of annotationProcessor
}
```

There are 3 major components in Room:

- **Database**: Contains the database holder and serves as the main access point for the underlying connection to your app's persisted, relational data.

  Abstract class that extends RoomDatabase. Annotated with @Database. Includes the list of entities within the annotation. Contains an   abstract method that has 0 arguments and returns the class that is annotated with @Dao.
  
- **Entity**: Represents a table within the database.

  To persist a field, Room must have access to it. You can make a field public, or you can provide a getter and setter for it. 

- **DAO**: Contains the methods used for accessing the database.

  Allow abstraction of the the database communication. Easier to mock in tests (compared to running direct sql queries). Automatically does the conversion from Cursor to application classes. All queries in Dao are verified while compile. Can be either an interface or an abstract class.
  
### Code examples

**User**

```
@Entity(tableName = "users")
public class User {
    @PrimaryKey
    public int uid;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;
    
    @Ignore
    Bitmap picture;
}
```

**UserDao**

```
@Dao
public interface UserDao {
    @Query("SELECT * FROM user")
    List<User> getAll();

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
           "last_name LIKE :last LIMIT 1")
    User findByName(String first, String last);

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    void insertAll(User... users);
    
    @Update
    public void updateUsers(User... users);

    @Delete
    void delete(User user);
}
```

**AppDatabase**

```
@Database(entities = {User.class, Book.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
    public abstract BookDao bookDao();
}
```

**Instantiation**. Probably singleton. Use Room.inMemoryDatabaseBuilder() for in memory DB.

```
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name").build();
```

## Dagger2

## Retrofit

## Glide

## Picasso

## RxJava
