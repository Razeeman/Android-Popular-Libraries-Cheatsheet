# Database

- [ORMLite](#ormlite---)
- [GreenDAO](#greendao---)
- [ObjectBox](#objectbox---)
- [Realm](#realm---)
- [Room](#room---)
- [Sources](#sources)



# ORMLite [![Maven][ormlite-mavenbadge]][ormlite-maven] [![Source][ormlite-sourcebadge]][ormlite-source] ![ormlite-starsbadge]

Object Relational Mapping Lite (ORM Lite) provides some simple, lightweight functionality for persisting Java objects to SQL databases while avoiding the complexity and overhead of more standard ORM packages. Old and proven ORM. Somewhat outdated.

```gradle
dependencies {
    implementation 'com.j256.ormlite:ormlite-android:5.1'
}
```

### ORMLite. Performance Comparison

ObjectBox < Realm < GreenDAO < Room < SQLite < **ORMLite**.

Very questionable, depends on complexity of queries, read/write preferences, app architecture etc.

Realm and ObjectBox are very fast, comparable to clean SQLite or even faster, but have some downsides.

### ORMLite. Usage

If you have worked with SQLite on Android you have worked with the SQLiteOpenHelper class. When working with ORMLite you need to extend the OrmLiteSqliteOpenHelper instead, but they are similar in the way that they have an onCreate, an onUpgrade ect. And you do the same kind of operations in them but with the help of ORMLite.

```java
@DatabaseTable(tableName = "accounts")
public class Account {
    
    @DatabaseField(columnName = "id", generatedId = true)
    private String name;
    
    @DatabaseField(columnName = "password")
    private String password;
    
    public Account() {
        // ORMLite needs a no-arg constructor 
    }
    
    /** Getters & Setters **/
    
}
```

```java
public class DatabaseHelper extends OrmLiteSqliteOpenHelper {

    private static final String DATABASE_NAME = "ormlite.db";
    private static final int    DATABASE_VERSION = 1;

    private Dao<Account, Integer> accountDao = null;

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db, ConnectionSource connectionSource) {
        try {
            TableUtils.createTable(connectionSource, Account.class);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, ConnectionSource connectionSource, 
                int oldVersion, int newVersion) {
        try {
            TableUtils.dropTable(connectionSource, Account.class, true);
            onCreate(db, connectionSource);
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    public Dao<Account, Integer> getAccountDao() throws SQLException {
        if (accountDao == null) {
            accountDao = getDao(Account.class);
        }
        return accountDao;
    }

    @Override
    public void close() {
        accountDao = null;
        super.close();
    }
}
```

```java
DatabaseHelper helper = new DatabaseHelper(this);
Dao<Account, Integer> accountDao = helper.getAccountDao();

Account account = new Account("name", "password");
accountDao.create(account);
```

### ORMLite. DAO methods

- create(Collection<T> datas)
    
- create(T data)

- createOrUpdate(T data)

- queryForAll()

- queryForId(ID id)

- update(T data)

- delete(T data)

- deleteById(ID id)

- executeRaw(String statement, String... arguments) - Run a raw execute SQL statement to the database.

- refresh(T data) - Does a query for the data's id and copies values from the database to refresh the data parameter.


### ORMLite. Custom DAO

Extend BaseDaoImpl.

```java
public class CustomDao<T, ID> extends BaseDaoImpl<T, ID> {
    protected CustomDao(final Class<T> dataClass) throws SQLException {
        super(dataClass);
    }

    @Override
    public int create(final T data) throws SQLException {
        int result = super.create(data);
        // Send an event with EventBus or Otto
        return result;
    }
}
```

```java
@DatabaseTable(tableName = "accounts", daoClass = CustomDao.class)
public class Account {
    ...
}
```

[Content](#database)
# GreenDAO [![Maven][greendao-mavenbadge]][greendao-maven] [![Source][greendao-sourcebadge]][greendao-source] ![greendao-starsbadge]

GreenDAO is a light & fast ORM solution for Android that maps objects to SQLite databases.

```gradle
buildscript {
    repositories {
        mavenCentral() // add repository
    }
    dependencies {
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2' // add plugin
    }
}
```

```gradle
apply plugin: 'org.greenrobot.greendao' // apply plugin
 
dependencies {
    implementation 'org.greenrobot:greendao:3.2.2' // add library
}
```

- **Maximum performance** (probably the fastest ORM for Android, was the fastest).

- **Easy to use** powerful APIs covering relations and joins.

- **Minimal memory consumption**.

- **Small library size** (<100KB) to keep build times low and to avoid the 65k method limit.

- **Database encryption**: greenDAO supports SQLCipher to keep user’s data safe.

- **Strong community**: More than 10.000 GitHub stars show there is a strong and active community.

### GreenDAO. Performance Comparison

ObjectBox < Realm < **GreenDAO** < Room < SQLite < ORMLite.

Very questionable, depends on complexity of queries, read/write preferences, app architecture etc.

Realm and ObjectBox are very fast, comparable to clean SQLite or even faster, but have some downsides.

### GreenDAO. Core classes

- **DaoMaster**

  The entry point for using greenDAO. Holds the database object (SQLiteDatabase) and manages DAO classes (not objects) for a specific schema. It has static methods to create the tables or drop them. Its inner classes OpenHelper and DevOpenHelper are SQLiteOpenHelper implementations that create the schema in the SQLite database.

- **DaoSession**

  Manages all available DAO objects for a specific schema, which you can acquire using one of the getter methods. Provides also some generic persistence methods like insert, load, update, refresh and delete for entities. Keeps track of an identity scope.

- **DAOs**

  Persists and queries for entities. For each entity, greenDAO generates a DAO. It has more persistence methods than DaoSession, for example: count, loadAll, and insertInTx.

- **Entities**

  Persistable objects. Usually, entities are objects representing a database row using standard Java properties (like a POJO).

### GreenDAO. Basic usage

```java
@Entity(nameInDb = "note")
public class Note {

    @Id(autoincrement = true)
    private Long id;

    @Property(nameInDb = "text")
    private String text;
}
```

Initialization in App class. DevOpenHelper is for dev only. DevOpenHelper will drop all tables on schema changes (in onUpgrade()). So it is recommended to create and use a subclass of DaoMaster.OpenHelper instead. DAO classes generated after build.

```java
public class App extends Application {
    private DaoSession daoSession;

    @Override
    public void onCreate() {
        super.onCreate();
        
        DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this, "notes-db");
        Database db = helper.getWritableDb();
        // Database db = helper.getEncryptedWritableDb("encryption-key");
        daoSession = new DaoMaster(db).newSession();
    }
    
    public DaoSession getDaoSession() {
        return daoSession;
    }
}
```

```java
Note note = new Note();

DaoSession daoSession = ((App) getApplication()).getDaoSession();
noteDao = daoSession.getNoteDao();

noteDao.insert(note);
noteDao.deleteByKey(id);
note.changeNote();
noteDao.update(note);

List<Note> notes = noteDao.queryBuilder()
    .where(Properties.PropertyName.eq("some property value"))
    .orderAsc(Properties.OtherPropertyName)
    .list();
```

### GreenDAO. TypeConverters

Example mapping an enum to an Integer.

```java
@Entity
public class User {
    @Id
    private Long id;

    @Convert(converter = RoleConverter.class, columnType = Integer.class)
    private Role role;

    public enum Role {
        DEFAULT(0), AUTHOR(1), ADMIN(2);
        
        final int id;
        
        Role(int id) {
            this.id = id;
        }
    }

    public static class RoleConverter implements PropertyConverter<Role, Integer> {
        @Override
        public Role convertToEntityProperty(Integer databaseValue) {
            if (databaseValue == null) {
                return null;
            }
            for (Role role : Role.values()) {
                if (role.id == databaseValue) {
                    return role;
                }
            }
            return Role.DEFAULT;
        }

        @Override
        public Integer convertToDatabaseValue(Role entityProperty) {
            return entityProperty == null ? null : entityProperty.id;
        }
    }
}
```

### GreenDAO. Kotlin

- Put greendao gradle plugin before kotlin plugin when applying plugins in app gradle file.

```
apply plugin: 'org.greenrobot.greendao'
apply plugin: 'kotlin-android'
```

- If you configure greenDao in gradle (e.g. set schema or generate location), then also put this sript into app gradle file, which allows greendao gradle task to run before kotlin debug and generate necessary classes.

```
tasks.whenTaskAdded { task ->
    if (task.name == 'kaptDebugKotlin') {
        task.dependsOn tasks.getByName('greendao')
    }
}
```

- GreenDAO entities should be written in java.

[Content](#database)
# ObjectBox [![Maven][objectbox-mavenbadge]][objectbox-maven] [![Source][objectbox-sourcebadge]][objectbox-source] ![objectbox-starsbadge]

ObjectBox is a super fast object-oriented NoSQL database, built uniquely for IoT and Mobile devices. From developers of GreenDAO.

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "io.objectbox:objectbox-gradle-plugin:2.3.4"
    }
}
```

```java
apply plugin: 'io.objectbox' // apply last
```

### ObjectBox. Performance Comparison

**ObjectBox** < Realm < GreenDAO < Room < SQLite < ORMLite.

Very questionable, depends on complexity of queries, read/write preferences, app architecture etc.

Realm and ObjectBox are very fast, comparable to clean SQLite or even faster, but have some downsides.

### ObjectBox. Entity Classes

Entities must have one @Id property of type long. By default IDs for new objects are assigned by ObjectBox.

```java
@Entity
public class User {
    @Id public long id;
    
    @Index
    @NameInDb("username")
    public String name;
    
    @Transient
    private int tempUsageCount; // not persisted
    
    // getters and setters ...
}
```

### ObjectBox. Core Classes

- **MyObjectBox**: Generated based on your entity classes, MyObjectBox supplies a builder to set up a BoxStore for your app.

- **BoxStore**: The entry point for using ObjectBox. BoxStore is your direct interface to the database and manages Boxes.

- **Box**: A box persists and queries for entities. For each entity, there is a Box (supplied by BoxStore).

### ObjectBox. Initialization

Probaby in application subclass.

```java
BoxStore boxStore = MyObjectBox.builder()
    .androidContext(context.getApplicationContext())
    .build();
```

### ObjectBox. Basic Box operations

The Box class is likely the class you interact with most. You get Box instances via **BoxStore.boxFor()**. A Box instance gives you access to objects of a particular type.

```java
Box<User> userBox = boxStore.boxFor(User.class);

User user = new User();
userBox.put(user);

user.setName("some name");
userBox.put(user);

List<User> users = userBox.query().equal(User_.name, "some name").build().find();

// Explicit transaction.
boxStore.runInTx(() -> {
   for(User user: allUsers) {
     modify(user);
     box.put(user);
   }
});

userBox.remove(user);
```

- **put**: Inserts a new or updates an existing object with the same ID. When inserting and put returns, an ID will be assigned to the just inserted object. put also supports putting multiple objects, which is more efficient. Runs an implicit transaction.

- **get** and **getAll**: Given an object’s ID reads it back from its box.

- **remove** and **removeAll**: Remove a previously put object from its box (deletes it). remove also supports removing multiple objects, which is more efficient.

- **count**: Returns the number of objects stored in this box.

- **query**: Starts building a query to return objects from the box that match certain conditions.

### ObjectBox. Queries

The **QueryBuilder** class lets you build custom queries for your entities. Create an instance via **Box.query()**. QueryBuilder offers several methods to define query conditions for properties of an entity. To specify a property ObjectBox does not use Strings but meta information "underscore" classes (like User_) that are generated during build time. The meta information classes have a static field for each property (like User_.name). This allows to reference properties safely with compile time checks to prevent runtime errors, for example because of typos.

- **equal()**, **notEqual()**, **greater()** and **less()**.

- **isNull()** and **notNull()**.

- **between()** to filter for values that are between the given two.

- **in()** and **notIn()** to filter for values that match any in the given array.

- **startsWith()**, **endsWith()** and **contains()** for extended String filtering.

- **and()** and **or()** to build more complex combinations of conditions.

- **order()** to order the returned results.

- **find()**, **findFirst()**, **findUnique()** to retrieve objects matching the query.

[Content](#database)
# Realm [![Maven][realm-mavenbadge]][realm-maven] [![Source][realm-sourcebadge]][realm-source] ![realm-starsbadge]

Realm is a mobile NoSQL database, a replacement for SQLite & ORMs.

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath "io.realm:realm-gradle-plugin:5.11.0"
    }
}
```

```gradle
apply plugin: 'realm-android'
```

- **Mobile-first**: Realm is the first database built from the ground up to run directly inside phones, tablets, and wearables.

- **Simple**: Data is directly exposed as objects and queryable by code, removing the need for ORM's riddled with performance & maintenance issues. Plus, we've worked hard to keep our API down to very few classes: most of our users pick it up intuitively, getting simple apps up & running in minutes.

- **Modern**: Realm supports easy thread-safety, relationships & encryption.

- **Fast**: Realm is faster than even raw SQLite on common operations while maintaining an extremely rich feature set.

### Realm. Performance Comparison

ObjectBox < **Realm** < GreenDAO < Room < SQLite < ORMLite.

Very questionable, depends on complexity of queries, read/write preferences, app architecture etc.

Realm and ObjectBox are very fast, comparable to clean SQLite or even faster, but have some downsides.

### Realm. Model class

RealmObjects are live, auto-updating views into the underlying data; you never have to refresh objects. Changes to objects are instantly reflected in query results.

```java
public class Dog extends RealmObject {
    private String name;
    private int age;

    // ... Generated getters and setters ...
}

public class Person extends RealmObject {
    @PrimaryKey
    private long id;
    @Index
    private String name;
    private RealmList<Dog> dogs; // Declare one-to-many relationships
    
    @Ignore
    private int sessionId;

    // ... Generated getters and setters ...
}
```

### Realm. Basic usage

Realm only manages the returned object, not the object originally created. To make changes to the object in the database, make changes to the returned copy, not the original.

```java
// Initialize Realm (just once per application). Probaby in application subclass.
Realm.init(context);

// Get a Realm instance for this thread.
Realm realm = Realm.getDefaultInstance();

// No changes in the database yet. Object is unmanaged.
Dog dog = new Dog();
dog.setName("Rex");

// All writes are wrapped in a transaction to facilitate safe multi threading.
realm.beginTransaction();

final Dog managedDog = realm.copyToRealm(dog);    // Persist unmanaged objects.
Person person = realm.createObject(Person.class); // Create managed objects directly.
person.setName("name");                           // Database will be changed, because managed object changed.
person.getDogs().add(managedDog);
managedDog.deleteFromRealm();

realm.commitTransaction();

RealmResults<User> result = realm
    .where(User.class)
    .greaterThan("age", 10)
    .beginGroup()
        .equalTo("name", "Peter")
        .or()
        .contains("name", "Jo")
    .endGroup()
    .findAll();
```

### Realm. Alternative execution

```java
// Automatically handles begin/commit, and cancel if an error happens.
realm.executeTransaction(new Realm.Transaction() {
    @Override
    public void execute(Realm realm) {
        Person person = realm.createObject(Person.class);
        person.setName("name");
    }
});
```

```java
//Asynchronous transaction to avoid blocking UI.
realm.executeTransactionAsync(new Realm.Transaction() {
            @Override
            public void execute(Realm bgRealm) {
                User user = bgRealm.createObject(User.class);
                user.setName("John");
                user.setEmail("john@corporation.com");
            }
        }, new Realm.Transaction.OnSuccess() {
            @Override
            public void onSuccess() {
                // Transaction was a success.
            }
        }, new Realm.Transaction.OnError() {
            @Override
            public void onError(Throwable error) {
                // Transaction failed and was automatically canceled.
            }
        });
```

### Realm. Queries

All fetches (including queries) are lazy in Realm, and the data is never copied.

```java
// Build the query looking at all users.
RealmQuery<User> query = realm.where(User.class);

// Add query conditions.
query.equalTo("name", "John");
query.or().equalTo("name", "Peter");

// Execute the query. Method findAll executes the query.
RealmResults<User> result1 = query.findAll();
```

- **where** method starts a RealmQuery by specifying a model.

- **findAll, findAllAsync, findFirst** methods executes the query.

- filter criteria is specified with predicate methods, most of which have self-explanatory names:
  **equalTo, notEqualTo, in, between, greaterThan, lessThan, greaterThanOrEqualTo, lessThanOrEqualTo, contains, beginsWith, endsWith, like, isEmpty, isNotEmpty, isNull, isNotNull**.
  
- **beginGroup** and **endGroup** methods group conditions to specify order of evaluation.

- **not** method negates conditions.

- **sort** method specifies how the results should be sorted.

- **limit** method limits query results.

- **distinct** method limits results to only unique values.

### Realm. Listeners

UI or other looper threads can get notified of changes in a Realm by adding a listener, which is executed whenever the Realm is changed.

- Realm notifications.

```java
realm = Realm.getDefaultInstance();
realmListener = new RealmChangeListener<Realm>() {
    @Override
    public void onChange(Realm realm) {
        // ... do something with the updates (UI, etc.) ...
    }
};
realm.addChangeListener(realmListener);
```

- Collection notifications.

```java
persons.addChangeListener(new OrderedRealmCollectionChangeListener<RealmResults<Person>>() {
    @Override
    public void onChange(RealmResults<Person> results, OrderedCollectionChangeSet changeSet) {
        // ... do something with the updates (UI, etc.) ...
    }
});
```

- Object notifications.

```java
persons.addChangeListener(new RealmObjectChangeListener<Person>() {
    @Override
    public void onChange(Person person, ObjectChangeSet changeSet) {
        // ... do something with the updates (UI, etc.) ...
    }
};
```

### Realm. Kotlin

- The realm-android plugin has to be applied after kotlin-android and kotlin-kapt.

- Model classes should be open.

- A limitation of the Kotlin annotation processor indicates that adding the annotation @RealmClass is required in some cases.

- Storing enums in Realm Model classes in Kotlin should be done using a specific pattern.

- In Kotlin Long::class.java actually returns a Class reference to long not Long. The same is true for other primitive types like Integer, Float, Double and Boolean. Choosing the correct class has implications during a migration.

- If Kotlin is used in a project, Realm automatically detects this and adds a number of extension methods that makes working with Kotlin easier.

[Content](#database)
# Room [![Maven][room-mavenbadge]][room-maven] [![Source][room-sourcebadge]][room-source]

The Room persistence library provides an abstraction layer over SQLite to allow for more robust database access. Provides compile time verification of SQL queries. Can work with LiveData and RxJava. Part of the AAC - android architecture components.

```gradle
dependencies {
    implementation "androidx.room:room-runtime:2.1.0-beta01"
    annotationProcessor "androidx.room:room-compiler:2.1.0-beta01"
    
    // Old namespace.
    // implementation "android.arch.persistence.room:runtime:1.1.1"
}
```

```gradle
apply plugin: 'kotlin-kapt'

dependencies {
    implementation "androidx.room:room-runtime:2.1.0-beta01"
    kapt "androidx.room:room-compiler:2.1.0-beta01"
}
```

There are 3 major components in Room:

- **Database**: Contains the database holder and serves as the main access point for the underlying connection to your app's persisted, relational data.

  Abstract class that extends RoomDatabase. Annotated with @Database. Includes the list of entities within the annotation. Contains an   abstract method that has 0 arguments and returns the class that is annotated with @Dao.
  
- **Entity**: Represents a table within the database.

  To persist a field, Room must have access to it. You can make a field public, or you can provide a getter and setter for it. 

- **DAO**: Contains the methods used for accessing the database.

  Allow abstraction of the the database communication. Easier to mock in tests (compared to running direct sql queries). Automatically does the conversion from Cursor to application classes. All queries in Dao are verified while compile. Can be either an interface or an abstract class.
  
### Room. Performance Comparison

ObjectBox < Realm < GreenDAO < **Room** < SQLite < ORMLite.

Very questionable, depends on complexity of queries, read/write preferences, app architecture etc.

Realm and ObjectBox are very fast, comparable to clean SQLite or even faster, but have some downsides.
  
### Room. Entity

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

### Room. Dao

```java
@Dao
public interface UserDao {
    @Query("SELECT * FROM users")
    List<User> getAll();
    
    @Query("SELECT * FROM users")
    LiveData<List<User>> getAll();
    
    @Query("SELECT * FROM users")
    Observable<List<User>> getAll();

    @Query("SELECT * FROM users WHERE uid IN (:userIds)")
    List<User> loadAllByIds(int[] userIds);

    @Query("SELECT * FROM users WHERE first_name LIKE :first AND " +
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

### Room. Database

```java
@Database(entities = {User.class, Book.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
    public abstract BookDao bookDao();
}
```

### Room. Instantiation

Expensive, so probably singleton. Outside of the main thread. Use Room.inMemoryDatabaseBuilder() for in memory DB.

```java
AppDatabase db = Room.databaseBuilder(getApplicationContext(),
    AppDatabase.class, "database-name").build();
db.userDao().insert(user);
```

### Room. TypeConverters

Specifies additional type converters that Room can use. The TypeConverter is added to the scope of the element so if you put it on a class / interface, all methods / fields in that class will be able to use the converters.

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

### Room. Migration

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



[Content](#database)
# Sources

**ObjectBox**

https://docs.objectbox.io/
<br>
https://github.com/objectbox/objectbox-java

**Realm**

https://github.com/realm/realm-java
<br>
https://realm.io/docs/java/latest/
<br>
https://realm.io/blog/realm-for-android/

**DBMS comparisons**

https://github.com/Rexee/AndroidDatabaseLibraryComparison
<br>
https://github.com/AlexeyZatsepin/Android-ORM-benchmark
<br>
https://proandroiddev.com/android-databases-performance-crud-a963dd7bb0eb (recent)



[ormlite-maven]: https://search.maven.org/artifact/com.j256.ormlite/ormlite-android
[ormlite-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.j256.ormlite/ormlite-android/badge.svg
[ormlite-source]: https://github.com/j256/ormlite-android
[ormlite-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[ormlite-starsbadge]: https://img.shields.io/github/stars/j256/ormlite-android

[greendao-maven]: https://search.maven.org/artifact/org.greenrobot/greendao
[greendao-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.greenrobot/greendao/badge.svg
[greendao-source]: https://github.com/greenrobot/greenDAO
[greendao-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[greendao-starsbadge]: https://img.shields.io/github/stars/greenrobot/greenDAO

[objectbox-maven]: https://bintray.com/objectbox/objectbox/io.objectbox:objectbox-gradle-plugin/_latestVersion
[objectbox-mavenbadge]: https://api.bintray.com/packages/objectbox/objectbox/io.objectbox:objectbox-gradle-plugin/images/download.svg
[objectbox-source]: https://github.com/objectbox/objectbox-java
[objectbox-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[objectbox-starsbadge]: https://img.shields.io/github/stars/objectbox/objectbox-java

[realm-maven]: https://bintray.com/realm/maven/realm-gradle-plugin/_latestVersion
[realm-mavenbadge]: https://api.bintray.com/packages/realm/maven/realm-gradle-plugin/images/download.svg
[realm-source]: https://github.com/realm/realm-java
[realm-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[realm-starsbadge]: https://img.shields.io/github/stars/realm/realm-java

[room-maven]: https://mvnrepository.com/artifact/androidx.room/room-runtime
[room-mavenbadge]: https://img.shields.io/badge/maven%20google--brightgreen.svg
[room-source]: https://android.googlesource.com/platform/frameworks/support/+/androidx-master-dev/room/
[room-sourcebadge]: https://img.shields.io/badge/source-google-orange.svg
