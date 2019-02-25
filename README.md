# Android Essential Libraries Cheatsheet

## Contents
- [Room](#room)
- [Dagger 2](#dagger-2)
- [Retrofit](#retrofit)
- [Glide](#glide)
- [RxJava](#rxjava)

<a name="room"></a>
# Room [![Maven Google][room-mavenbadge-svg]][room-mavengoogle]

The Room persistence library provides an abstraction layer over SQLite to allow for more robust database access. Provides compile time verification of SQL queries.

```java
dependencies {   
    implementation "androidx.room:room-runtime:2.1.0-alpha04"
    annotationProcessor "androidx.room:room-compiler:2.1.0-alpha04"
    
    // android.arch.persistence.room:runtime:1.1.1
    // android.arch.persistence.room:compiler:1.1.1
}
```

There are 3 major components in Room:

- **Database**: Contains the database holder and serves as the main access point for the underlying connection to your app's persisted, relational data.

  Abstract class that extends RoomDatabase. Annotated with @Database. Includes the list of entities within the annotation. Contains an   abstract method that has 0 arguments and returns the class that is annotated with @Dao.
  
- **Entity**: Represents a table within the database.

  To persist a field, Room must have access to it. You can make a field public, or you can provide a getter and setter for it. 

- **DAO**: Contains the methods used for accessing the database.

  Allow abstraction of the the database communication. Easier to mock in tests (compared to running direct sql queries). Automatically does the conversion from Cursor to application classes. All queries in Dao are verified while compile. Can be either an interface or an abstract class.
  
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

<a name="dagger-2"></a>
# Dagger 2 [![Maven Central][dagger-mavenbadge-svg]][dagger-mavencentral]

A fast dependency injector for Android and Java. Compile-time evaluation. Uses code generation and is based on annotations.

```java
dependencies {
  implementation 'com.google.dagger:dagger:2.21'
  annotationProcessor 'com.google.dagger:dagger-compiler:2.21'
}
```

### Dependency injection

**Dependency injection** is a technique whereby one object supplies the dependencies of another object. A dependency is an object that can be used. An injection is the passing of a dependency to a dependent object that would use it. The intent is to decouple objects so that no client has to be changed simply because an object it depends on needs to be changed to a different one. This permits following the Open / Closed principle and Inversion of Control principle. The most important advantage is that it increases the possibility of reusing the class and to be able to test them independent of other classes.

### Dagger 2 dependency injection process

- **Dependency provider**: Classes annotated with **@Module** are responsible for providing objects which can be injected. Such classes define methods annotated with **@Provides**. The returned objects from these methods are available for dependency injection.

- **Dependency consumer**: The **@Inject** annotation is used to define a dependency. Can be used on a constructor, a field, or a method.

- **Connecting consumer and producer**: A **@Component** annotated interface defines the connection between the provider of objects (modules) and the objects which express a dependency. The class for this connection is generated by the Dagger.

### Without dependency injection

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

### Dependency injection without frameworks

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

### Dependency injection with dagger 2

Module provides dependencies.

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

Database asks for context dependency and provides itself as a dependency.

```java
@Singleton
class MyDatabase {
    @Inject
    MyDatabase(@Named("ApplicationContext") Context context) {
    }
}
```

Component connects dependants that ask for dependencies with modules that provides them.

```java
@Component (modules={ContextModule.class})
interface MyComponent {
    void inject(MyPresenter presenter);
}
```

Instantiation of the component.

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

Invocation of dependency injection. Presenter asks for database, component injects database by calling it's constructor, which in turn asks for context dependency, which is provided by the module.

```java
class MyPresenter {

    @Inject 
    MyDatabase db;
    
    MyPresenter() {
        MyApplication.getMyComponent().inject(this);
    }
}
```

<a name="retrofit"></a>
# Retrofit [![Maven Central][retrofit-mavenbadge-svg]][retrofit-mavencentral]

Type-safe HTTP client. Provides a powerful framework for authenticating and interacting with APIs and sending network requests with OkHttp. Automatically serialises the JSON response using a POJO. For conversion uses GSON or several other converters.

```java
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.5.0'
    
    // Or any other available converter.
    implementation 'com.google.code.gson:gson:2.8.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
}
```

### Retrofit. Data model to store parsed json

Class structure is mirroring json structure.

```json
[
    {
        "id": 1,
        "name": "John Jameson",
        "email": "johnjameson@example.com",
    },
    {
        "id": 2,
        "name": "James Johnson",
        "email": "jamesjohnson@example.com",
    },
    {
        "id": 3,
        "name": "Jenna Juliano",
        "email": "jennajuliano@example.com",
    }
]
```

```java
public class User {
    @SerializedName("id")
    int mId;

    @SerializedName("name")
    String mName;
    
    @SerializedName("email")
    String mEmail;

    // Constructor, getters, setters.
}
```

### Retrofit. Instantiation

Expensive. So probably singleton.

```java
public static final String BASE_URL = "http://api.myservice.com/";
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl(BASE_URL)
    .addConverterFactory(GsonConverterFactory.create())
    .build();
```

### Retrofit. Endpoints

The endpoints are defined inside of an interface using special retrofit annotations to encode details about the parameters and request method. The return value is always a parameterized Call<T>. This is just a callback given by the retrofit to perform the request synchronously or asynchronously.

```java
public interface ApiServiceInterface  {
    @GET("users/{username}")
    Call<User> getUser(@Path("username") String username);

    @GET("group/{id}/users")
    Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);

    @POST("users/new")
    Call<User> createUser(@Body User user);
}
```

- **@Path** variable substitution for the API endpoint (i.e. username will be swapped for {username} in the URL endpoint).

- **@Query** specifies the query key name with the value of the annotated parameter. 

- **@Body** payload for the POST call (serialized from a Java object to a JSON string)

- **@Header** specifies the header with the value of the annotated parameter

### Retrofit. Invocation

enqueue() asynchronously sends the request on the background thread and notifies your app with a callback when a response comes back.

```java
User user = new User(123, "John Doe");
ApiServiceInterface apiService = retrofit.create(ApiServiceInterface.class);

Call<User> call = apiService.createuser(user);
call.enqueue(new Callback<User>() {
    @Override
    public void onResponse(Call<User> call, Response<User> response) {
        // code
    }

    @Override
    public void onFailure(Call<User> call, Throwable t) {
        // Log error
    }
});

Call<User> call = apiService.getUser("John Doe");
call.enqueue(new Callback<User>() {
    @Override
    public void onResponse(Call<User> call, Response<User> response) {
        int statusCode = response.code();
        User newUser = response.body();
    }

    @Override
    public void onFailure(Call<User> call, Throwable t) {
        // Log error
    }
});
```

<a name="glide"></a>
# Glide [![Maven Central][glide-mavenbadge-svg]][glide-mavencentral]

Glide is a fast and efficient image loading library for Android focused on smooth scrolling. Glide offers an easy to use API, a performant and extensible resource decoding pipeline and automatic resource pooling. Glide supports fetching, decoding, and displaying video stills, images, and animated GIFs. Uses HttpUrlConnection.

```java
dependencies {
    compile 'com.github.bumptech.glide:glide:4.9.0'
    annotationProcessor 'com.github.bumptech.glide:compiler:4.9.0'
}
```

- Smart and automatic **downsampling** and **caching** minimize storage overhead and decode times.
- Aggressive **re-use of resources** like byte arrays and Bitmaps minimizes expensive garbage collections and heap fragmentation.
- Deep **lifecycle integration** ensures that only requests for active Fragments and Activities are prioritized and that Applications release resources when neccessary to avoid being killed when backgrounded.

By default, Glide checks multiple layers of caches before starting a new request for an image:

- Active resources - Is this image displayed in another View right now?
- Memory cache - Was this image recently loaded and still in memory?
- Resource - Has this image been decoded, transformed, and written to the disk cache before?
- Data - Was the data this image was obtained from written to the disk cache before?

### Glide. Usage

```java
Glide.with(fragment)
    .load(myUrl)
    .into(imageView);
```

```java
GlideApp.with(context)
    .load(myUrl)
    .override(300, 200)
    .placeholder(R.drawable.placeholder)
    .error(R.drawable.imagenotfound)
    .thumbnail(Glide.with(this)
        .load(fastLoadUrl)
        .apply(requestOption))
    .transition(DrawableTransitionOptions.withCrossFade())
    .centerCrop()
    .listener(MyImageRequestListener(this))
    .into(imageView);
```

### Glide. Background Threads

```java
FutureTarget<Bitmap> futureTarget =
    Glide.with(context)
        .asBitmap()
        .load(url)
        .submit(width, height);

Bitmap bitmap = futureTarget.get();

// Do something with the Bitmap and then when you're done with it:
Glide.with(context).clear(futureTarget);
```

### Glide. RequestOptions

Most options in Glide can be applied using the RequestOptions class and the apply() method. Options among others:

- Placeholders
- Transformations
- Caching Strategies
- Component specific options, like encode quality, or decode Bitmap configurations.

```java
RequestOptions cropOptions = new RequestOptions().centerCrop(context);
...
Glide.with(fragment)
    .load(url)
    .apply(cropOptions)
    .into(imageView);
```

### Glide. Generated API

Hand writing custom subclasses of RequestOptions is challenging and produces a less fluent API. Generate API allows access all options in RequestBuilder, RequestOptions and any included integration libraries in a single fluent API. Generated after rebuild.

- Integration libraries can extend Glide’s API with custom options.
- Applications can extend Glide’s API by adding methods that bundle commonly used options.

```java
@GlideModule
public final class MyAppGlideModule extends AppGlideModule {}
```

```java
GlideApp.with(fragment)
    .load(myUrl)
    .placeholder(R.drawable.placeholder)
    .centerCrop()
    .into(imageView);
```

Unlike Glide.with() options like centerCrop() and placeholder() are available directly on the builder and don’t need to be passed in as a separate RequestOptions object.

<a name="rxjava"></a>
# RxJava [![Maven Central][rxjava-mavenbadge-svg]][rxjava-mavencentral] RxAndroid [![Maven Central][rxandroid-mavenbadge-svg]][rxandroid-mavencentral]

RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences. It extends the observer pattern to support sequences of data/events and adds operators that allow you to compose sequences together declaratively while abstracting away concerns about things like low-level threading, synchronization, thread-safety and concurrent data structures.

```java
dependencies {
    implementation "io.reactivex.rxjava2:rxjava:2.2.7"
    implementation "io.reactivex.rxjava2:rxandroid:2.1.1"
}
```

- **Reactive Programming** is a programming paradigm oriented around data flows and the propagation of change i.e. it is all about responding to value changes. For example, let’s say we define x = y+z. When we change the value of y or z, the value of x automatically changes. This can be done by observing the values of y and z. Everything you see is an asynchronous data stream, which can be observed and an action will be taken place when it emits values. You can create data stream out of anything; variable changes, click events, http calls, data storage, errors and what not. When it says asynchronous, that means every code module runs on its own thread thus executing multiple code blocks simultaneously.

- **Reactive Extensions** is a library that follows Reactive Programming principles to compose asynchronous and event-based programs by using observable sequence.

- **RxJava** is a Java based implementation of Reactive Extensions. Basically it’s a library that composes asynchronous events by following Observer Pattern. You can create asynchronous data stream on any thread, transform the data and consumed it by an Observer on any thread. The library offers wide range of amazing operators like map, combine, merge, filter and lot more that can be applied onto data stream.

- **RxAndroid** is specific to Android platform which utilises some classes on top of the RxJava library. Providers a scheduler to run code in the main thread of Android. It also provides the ability to create a scheduler that runs on a Android handler class.

### Observer Pattern

The observer pattern is a software design pattern in which an object, called the **subject**, maintains a list of its dependents, called **observers**, and **notifies** them automatically of any state changes, usually by calling one of their methods. It is mainly used to implement distributed event handling systems, in "event driven" software. Addresses the following problems:

- A one-to-many dependency between objects should be defined without making the objects tightly coupled.
- It should be ensured that when one object changes state an open-ended number of dependent objects are updated automatically.
- It should be possible that one object can notify an open-ended number of other objects.

### RxJava. Core

Everything can be modeled as streams. A stream emits item(s) over time, and each emission can be consumed/observed. The stream abstraction is implemented through 3 core constructs: the Observable, Observer, and the Operator. The Observable emits items (the stream); and the Observer consumes those items. Emissions from Observable objects can further be modified, transformed, and manipulated by chaining Operator calls.

- **Observable**: Observable is a data stream that do some work and emits data.

- **Observer**: Observer is the counter part of Observable. It receives the data emitted by Observable.

- **Subscription**: The bonding between Observable and Observer is called as Subscription. There can be multiple Observers subscribed to a single Observable.

- **Operator / Transformation**: Operators modifies the data emitted by Observable before an observer receives them. Operators can be chained together to create complex data flows that filter event based on certain criteria.

- **Schedulers**: Schedulers decides the thread on which Observable should emit the data and on which Observer should receives the data i.e background thread, main thread etc.

### RxJava. Base usage

```java
Flowable.just("Hello world")
    .subscribe(new Consumer<String>() {
        @Override public void accept(String s) {
            System.out.println(s);
        }
    });
```

Consumer is a functional interface which allows to use java8 lambdas and method references.

```java
public class HelloWorld {
    public static void main(String[] args) {
        Flowable.just("Hello world").subscribe(System.out::println);
    }
}

Produces:
Hello world
```

```java
Observable<String> observable = Observable.just("A", "B", "C", "D", "E");
Observer<String> observer new Observer<String>() {
    @Override
    public void onNext(String s) { Log.d(TAG, s); }
    @Override
    public void onError(Throwable e) { Log.e(TAG, "onError: " + e.getMessage()); }
    @Override
    public void onComplete() { Log.d(TAG, "All items emitted"); }
    @Override
    public void onSubscribe(Disposable d) { Log.d(TAG, "onSubscribe"); }
};
observable
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(observer);
```

Same with lambas.

```java
Observable.just("A", "B", "C", "D", "E").
    .subscribeOn(Schedulers.io())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(
        s -> Log.d(TAG, s),                               //OnNext
        e -> Log.e(TAG, "onError: " + e.getMessage()),    //OnError
        () -> Log.d(TAG, "All items emitted"),            //OnComplete
        d -> Log.d(TAG, "onSubscribe")                    //onSubscribe
    );

Produces:
onSubscribe
A
B
C
D
E
All items emitted
```

```java
Observable.fromArray(letters)
    .map(String::toLowerCase)
    .filter(letter -> !letter.equals("C"))
    .subscribe(letter -> result += letter);

Produces:
a
b
d
e
```

### RxJava. Base observable classes

RxJava 2 features several base classes you can discover operators on:

- **io.reactivex.Flowable**: 0..N flows, supporting Reactive-Streams and backpressure, which controls how fast a source emits items.
- **io.reactivex.Observable**: 0..N flows, no backpressure.
- **io.reactivex.Single**: a flow of exactly 1 item or an error, the reactive version of a method call.
- **io.reactivex.Completable**: a flow without items but only a completion or error signal, the reactive version of a Runnable.
- **io.reactivex.Maybe**: a flow with no items, exactly one item or an error.

### RxJava. Terminology

**Upstream, downstream**. The dataflows in RxJava consist of a source, zero or more intermediate steps followed by a data consumer or combinator step (where the step is responsible to consume the dataflow by some means). For intermediate operator, looking to the left towards the source, is called the **upstream**. Looking to the right towards the subscriber/consumer, is called the **downstream**.

**Backpressure**. When the dataflow runs through asynchronous steps, each step may perform different things with different speed. To avoid overwhelming such steps, which usually would manifest itself as increased memory usage due to temporary buffering or the need for skipping/dropping data, a so-called backpressure is applied, which is a form of flow control where the steps can express how many items are they ready to process. This allows constraining the memory usage of the dataflows in situations where there is generally no way for a step to know how many items the upstream will send to it.

**Assembly time**. The preparation of dataflows by applying various intermediate operators happens in the so-called assembly time. At this point, the data is not flowing yet and no side-effects are happening.

**Subscription time**. This is a temporary state when subscribe() is called on a flow that establishes the chain of processing steps internally. This is when the subscription side-effects are triggered (see doOnSubscribe). Some sources block or start emitting items right away in this state.

**Runtime**. This is the state when the flows are actively emitting items, errors or completion signals. Practically, this is when the body of the given example above executes.

[room-mavengoogle]: https://mvnrepository.com/artifact/androidx.room/room-runtime
[room-mavenbadge-svg]: https://img.shields.io/badge/maven%20google--green.svg
[dagger-mavencentral]: https://search.maven.org/artifact/com.google.dagger/dagger
[dagger-mavenbadge-svg]: https://maven-badges.herokuapp.com/maven-central/com.google.dagger/dagger/badge.svg
[retrofit-mavencentral]: https://search.maven.org/artifact/com.squareup.retrofit2/retrofit
[retrofit-mavenbadge-svg]: https://maven-badges.herokuapp.com/maven-central/com.squareup.retrofit2/retrofit/badge.svg
[glide-mavencentral]: https://search.maven.org/artifact/com.github.bumptech.glide/glide
[glide-mavenbadge-svg]: https://maven-badges.herokuapp.com/maven-central/com.github.bumptech.glide/glide/badge.svg
[rxjava-mavencentral]: https://search.maven.org/artifact/io.reactivex.rxjava2/rxjava
[rxjava-mavenbadge-svg]: https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxjava/badge.svg
[rxandroid-mavencentral]: https://search.maven.org/artifact/io.reactivex.rxjava2/rxandroid
[rxandroid-mavenbadge-svg]: https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxandroid/badge.svg
