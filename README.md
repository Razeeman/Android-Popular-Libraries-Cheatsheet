# Android Essential Libraries Cheatsheet

## Room

The Room persistence library provides an abstraction layer over SQLite to allow for more robust database access. Provides compile time verification of SQL queries.

```java
dependencies {
    def room_version = "2.1.0-alpha04"
    
    implementation "androidx.room:room-runtime:$room_version"
    annotationProcessor "androidx.room:room-compiler:$room_version"
    
    // android.arch.persistence.room:runtime:1.1.0
    // android.arch.persistence.room:compiler:1.1.0
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

```java
@Entity(tableName = "users")
public class User {
    @PrimaryKey(autoGenerate = true)
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

```java
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
    
    @Insert
    long insert(User user);
    
    @Update
    public void updateUsers(User... users);

    @Delete
    void delete(User user);
}
```

**AppDatabase**

```java
@Database(entities = {User.class, Book.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
    public abstract BookDao bookDao();
}
```

**Instantiation**. Expensive, so probably singleton. Outside of the main thread. Use Room.inMemoryDatabaseBuilder() for in memory DB.

```java
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name").build();
db.userDao().insert(user);
```

**TypeConverters**. Specifies additional type converters that Room can use. The TypeConverter is added to the scope of the element so if you put it on a class / interface, all methods / fields in that class will be able to use the converters.

```java
 // example converter for java.util.Date
 public static class DateConverter {
    @TypeConverter
    public Date fromLong(Long value) { return value == null ? null : new Date(value); }

    @TypeConverter
    public Long fromDate(Date date) { return date == null ? null : date.getTime() }
}
```
```java
@Entity()
public class Employee {
   @PrimaryKey
   public long id;
 
   @TypeConverters({DateConverter.class})
   public Date date;
}
```

**Migration**.

```java
public static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE users "
                + " ADD COLUMN age INTEGER");
    }
};

AppDatabase db = Room.databaseBuilder(getApplicationContext(),
        AppDatabase.class, "database-name")
        .addMigrations(MyDatabase.MIGRATION_1_2)
        .build();
```

## Dagger2

A fast dependency injector for Android and Java. Compile-time evolution. Uses code generation and is based on annotations.

```java
dependencies {
  compile 'com.google.dagger:dagger:2.21'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.21'
}
```

Dagger 2 dependency injection process:

- **Dependency provider**: Classes annotated with **@Module** are responsible for providing objects which can be injected. Such classes define methods annotated with **@Provides**. The returned objects from these methods are available for dependency injection.

- **Dependency consumer**: The **@Inject** annotation is used to define a dependency. Can be used on a constructor, a field, or a method.

- **Connecting consumer and producer**: A **@Component** annotated interface defines the connection between the provider of objects (modules) and the objects which express a dependency. The class for this connection is generated by the Dagger.

**Dependency injection** is a technique whereby one object supplies the dependencies of another object. A dependency is an object that can be used. An injection is the passing of a dependency to a dependent object that would use it. The intent is to decouple objects so that no client has to be changed simply because an object it depends on needs to be changed to a different one. This permits following the Open / Closed principle and Inversion of Control principle. The most important advantage is that it increases the possibility of reusing the class and to be able to test them independent of other classes.

**Example without dependency injection**

```java
public class MyPresenter {
    // Internal reference to the service used by this client
    private MyDatabase database;

    MyPresenter() {
        // Specify a specific implementation in the constructor instead of using dependency injection
        database = new MyDatabase(context);
    }
}
```

**Constructor injection**

```java
// Constructor
MyPresenter(MyDatabase database) {
    // Save the reference to the passed-in service inside this client
    this.database = database;
}
```

**Setter injection**

```java
// Setter method
public void setDatabase(MyDatabase database) {
    // Save the reference to the passed-in service inside this client.
    this.database = database;
}
```

**Interface injection**

This is simply the client publishing a role interface to the setter methods of the client's dependencies. It can be used to establish how the injector should talk to the client when injecting dependencies.

```java
public interface DatabaseSetter {
    public void setDatabase(MyDatabase database);
}

public class MyPresenter implements DatabaseSetter {
    private MyDatabase database;

    // Set the service that this client is to use.
    @Override
    public void setDatabase(MyDatabase database) {
        this.database = database;
    }
}
```

**Constructor injection assembly**

```java
class MyActivity {
    @Override
    public void onCreate() {
        Context context = this.getApplicationContext();
        MyDatabase db = new MyDatabase(context);
        MyPresenter presenter = new MyPresenter(db);
    }
}

class MyDatabase {
    MyDatabase(Context context) {
    }
}

class MyPresenter {
  	MyPresenter(MyDatabase db) {
    }
}
```

**With dagger 2**

```java
@Module
class ContextModule {
    Context context;
    
    ContextModule(Context context) {
    	this.context = context;
    }
    
    @Provides
    @Named("ApplicationContext")
    Context getContext() {
    	return context;
    }
}
```
```java
@Component (modules={ContextModule.class})
interface MyComponent {
    void inject(MyPresenter presenter);
}
```
```java

class MyApplication extends Application {
    private static MyComponent component;
    
    @Override
    void onCreate() {
        component = DaggerMyComponent.builder()
            .contextModule(new ContextModule(getApplicationContext()))
            .build();
    }
    
    public static MyComponent getMyComponent() {
        return component;
    }
}
```
```java
@Singleton
class MyDatabase {
    @Inject
    MyDatabase(@Named("ApplicationContext") Context context) {
    }
}
```
```java
class MyPresenter {

    @Inject 
    MyDatabase db;
    
    MyPresenter() {
        MyApplication.getMyComponent().inject(this);
    }
}
```

## Retrofit

## Glide

## Picasso

## RxJava
