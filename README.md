# Android Essential Libraries Cheatsheet

## Contents
- **Misc**
  - [GSON](#gson-tag)
  - [Glide](#glide-tag)
  - [Butter Knife](#butterknife-tag)
- **DI**
  - [Dagger](#dagger-tag)
  - [Koin](#koin-tag)
- **Database**
  - [ORMLite](#ormlite-tag)
  - [GreenDAO](#greendao-tag)
  - [ObjectBox](#objectbox-tag)
  - [Realm](#realm-tag)
  - [Room](#room-tag)
- **Async**
  - [EventBus](#eventbus-tag)
  - [RxJava](#rxjava-tag)
- **Network**
  - [OkHttp](#okhttp-tag)
  - [Retrofit](#retrofit-tag)
- **Testing**
  - [JUnit](#junit-tag)
  - [Mockito](#mockito-tag)
  - [PowerMock](#powermock-tag)
  - [Espresso](#espresso-tag)
- [Sources](#sources-tag)

<br><br>

<a name="gson-tag"></a>
# GSON [![Maven][gson-mavenbadge]][gson-maven] [![Source][gson-sourcebadge]][gson-source]

A Java serialization/deserialization library to convert Java Objects into JSON and back. Can work with arbitrary Java objects including pre-existing objects that you do not have source-code of. Internally, utilizes a JsonReader class.

```gradle
dependencies {
    implementation 'com.google.code.gson:gson:2.8.5'
}
```

### GSON. Primitives Examples

```java
Gson gson = new Gson();

gson.toJson(1);              // ==> 1
gson.toJson(new Integer(1)); // ==> 1
int one = gson.fromJson("1", int.class);
Integer one = gson.fromJson("1", Integer.class);
Long one = gson.fromJson("1", Long.class);

gson.toJson("abcd");         // ==> "abcd"
String str = gson.fromJson("\"abcd\"", String.class);

int[] values = { 1 };
gson.toJson(values);         // ==> [1]
Integer[] newValues = gson.fromJson("[1]", Integer[].class);

Boolean false = gson.fromJson("false", Boolean.class);
```

### GSON. Object Examples

```java
public class UserNested {  
    String name;
    String email;
    boolean isDeveloper;
    int age;
	
    UserAddress userAddress;
}

public class UserAddress {  
    String street;
    String houseNumber;
    String city;
    String country;
}

UserAddress userAddress = new UserAddress("Main Street", "42A", "Magdeburg", "Germany");
UserNested userObject = new UserNested("Norman", "norman@futurestud.io", 26, true, userAddress);

Gson gson = new Gson();  
String userWithAddressJson = gson.toJson(userObject);
UserNested restoredUser = gson.fromJson(userWithAddressJson, UserNested.class);  
```

Produces:

```java
{
    "age": 26,
    "email": "norman@futurestud.io",
    "isDeveloper": true,
    "name": "Norman",

    "userAddress": {
        "city": "Magdeburg",
        "country": "Germany",
        "houseNumber": "42A",
        "street": "Main Street"
    }
}
```

### GSON. Builder

Use GsonBuilder in order to change certain settings and customized configuration.

```java
GsonBuilder gsonBuilder = new GsonBuilder();
gsonBuilder.setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES);
gsonBuilder.serializeNulls();
gsonBuilder.setExclusionStrategies(new ExclusionStrategy() { ... });
gsonBuilder.excludeFieldsWithModifiers(Modifier.STATIC, Modifier.FINAL);
gsonBuilder.setLenient();
Gson gson = gsonBuilder.create();
```

### GSON. Annotations

- **@Expose(serialize = false, deserialize = false)** to control exposure.

- **@SerializedName(value = "fullName", alternate = "username")** to change the automated matching to and from the JSON.

- **@Since(version_num) and @Until(version_num)** with **builder.setVersion(version_num)** to control exposure with versioning.

<a name="glide-tag"></a>
# Glide [![Maven][glide-mavenbadge]][glide-maven] [![Source][glide-sourcebadge]][glide-source]

Glide is a fast and efficient image loading library for Android focused on smooth scrolling. Glide offers an easy to use API, a performant and extensible resource decoding pipeline and automatic resource pooling. Glide supports fetching, decoding, and displaying video stills, images, and animated GIFs. Uses HttpUrlConnection.

```gradle
dependencies {
    implementation 'com.github.bumptech.glide:glide:4.9.0'
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

Hand writing custom subclasses of RequestOptions is challenging and produces a less fluent API. Generate API allows access all options in RequestBuilder, RequestOptions and any included integration libraries in a single fluent API. The API is generated after rebuild in the same package as the AppGlideModule implementation provided by the application and is named GlideApp by default.

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

<a name="butterknife-tag"></a>
# Butter Knife [![Maven][butterknife-mavenbadge]][butterknife-maven] [![Source][butterknife-sourcebadge]][butterknife-source]

Field and method binding for Android views which uses annotation processing to generate boilerplate code. Instead of slow reflection, code is generated to perform the view look-ups. Calling bind delegates to this generated code that you can see and debug.

```gradle
dependencies {
    implementation 'com.jakewharton:butterknife:10.1.0'
    annotationProcessor 'com.jakewharton:butterknife-compiler:10.1.0'
}
```

### Butter Knife. Usage

- Eliminate findViewById calls by using **@BindView** on fields.
- Group multiple views in a list or array. Operate on all of them at once with actions, setters, or properties.
- Eliminate anonymous inner-classes for listeners by annotating methods with **@OnClick** and others.
- Eliminate resource lookups by using resource annotations on fields **@BindBool, @BindColor, @BindDimen, @BindDrawable, @BindInt, @BindString**.

```java
class ExampleActivity extends Activity {
    @BindView(R.id.user) EditText username;
    @BindView(R.id.pass) EditText password;

    @BindString(R.string.login_error) String loginErrorMessage;

    @OnClick(R.id.submit) void submit() {
		// TODO call server...
    }

    @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.simple_activity);
    ButterKnife.bind(this);
        // TODO Use fields...
    }
}
```

### Butter Knife. View lists

You can group multiple views into a List or array.

```java
@BindViews({ R.id.first_name, R.id.middle_name, R.id.last_name })
List<EditText> nameViews;
```

The apply method allows you to act on all the views in a list at once.

```java
ButterKnife.apply(nameViews, DISABLE);
ButterKnife.apply(nameViews, ENABLED, false);
```

Action and Setter interfaces allow specifying simple behavior.

```java
static final ButterKnife.Action<View> DISABLE = new ButterKnife.Action<View>() {
    @Override public void apply(View view, int index) {
        view.setEnabled(false);
    }
};
static final ButterKnife.Setter<View, Boolean> ENABLED = new ButterKnife.Setter<View, Boolean>() {
    @Override public void set(View view, Boolean value, int index) {
        view.setEnabled(value);
    }
};
```

An Android Property can also be used with the apply method.

```java
ButterKnife.apply(nameViews, View.ALPHA, 0.0f);
```

<a name="dagger-tag"></a>
# Dagger [![Maven][dagger-mavenbadge]][dagger-maven] [![Source][dagger-sourcebadge]][dagger-source]

A fast dependency injector for Android and Java. Compile-time evaluation. Uses code generation and is based on annotations.

```gradle
dependencies {
    implementation 'com.google.dagger:dagger:2.22.1'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.22.1'
}
```

For kotlin.

```gradle
apply plugin: 'kotlin-kapt'

dependencies {
    implementation 'com.google.dagger:dagger:2.22.1'
    kapt "com.google.dagger:dagger-compiler:2.22.1"
}
```

### Dependency injection

**Dependency injection** is a technique whereby one object supplies the dependencies of another object. A dependency is an object that can be used. An injection is the passing of a dependency to a dependent object that would use it. The intent is to decouple objects so that no client has to be changed simply because an object it depends on needs to be changed to a different one. This permits following the Open / Closed principle and Inversion of Control principle. The most important advantage is that it increases the possibility of reusing the class and to be able to test them independent of other classes.

### Dagger dependency injection process

- **Dependency provider**: Classes annotated with **@Module** are responsible for providing objects which can be injected. Such classes define methods annotated with **@Provides**. The returned objects from these methods are available for dependency injection.

- **Dependency consumer**: The **@Inject** annotation is used to define a dependency. Can be used on a constructor, a field, or a method. @Inject annotated constructor provides itself as a dependency. @Inject annotated field asks for dependency to be injected in that field.

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

### Dependency injection with dagger

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
    Context anyName() {
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
@Component (modules={ContextModule.class, OtherModule.class})
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
            .otherModule(new OtherModule())
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

### Dagger. Scopes

Scope determines the lifetime of a variable. If we do not specify any Scope annotation, the Component will create a new instance every time the dependency is injected, whereas if we do specify a Scope, Component can retain the instance for future use.

The scope is defined with a given name, where the name could be any name e.g. @Singleton, @ActivityScope, @FragmentScope, @WhateverScope. 

```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface ScopeName {
}
```

Dagger provides @Singleton scope annotation. It is just a usual named scope defined the same way but with a name Singleton. @Singleton annotation doesn't provide singleton functionality by itself. It's developer's responsibility to make sure that the Component with @Singleton annotation created only once. Otherwise every Component will have it's own version of @Singleton object.

Lifecycle of scoped objects tied to the lifecycle of the Component. Components live as long as you want it to or as long as class that created component wasn't destroyed (like android activity or fragment). If Component built in application its instances of scoped objects will live as long as application or until manually cleared. This is useful for application level dependencies like ApplicationContext. If Component is built in activity, scoped instances will be cleared on destroy of activity.

### Dagger. Component dependencies

If at least one provide method in a module has a scope annotation the Component should have the same scope annotation. Which means that if you want named scope, you need a separate Component for it. Components can depend on each other.

- Two dependent components cannot have the same Scope.
- Parent component must explicitly declare objects which can be used in child components.
- Component can depend on other components.

```java
@Component (modules={ContextModule.class, OtherModule.class})
@Singleton
interface AppComponent {
    // Providing dependencies to children.
    @Named("ApplicationContext") Context anyName();
    MyDatabase anyOtherName();
    // injects
}
```

```java
@Component (dependencies={AppComponent.class}, modules={ActivityModule.class})
@ActivityScope
interface ActivityComponent {
    // Context and MyDatabase available from parent and can be passed futher.
    @Named("ApplicationContext") Context anyName();
    // injects
}
```

Instantiation.

```java
appComponent = DaggerAppComponent.builder()
    .contextModule(new ContextModule(getApplicationContext()))
    .otherModule(new OtherModule())
    .build();
activityComponent = DaggerActivityComponent.builder()
    .appComponent(appComponent)
    .build();
```

### Dagger. Subcomponents

Same goal as component dependencies, different approach.

- Parent component is obliged to declare Subcomponents getters inside its interface.
- Subcomponent has access to all parents objects.
- Subcomponent can only have one parent.

```java
@Component (modules={ContextModule.class, OtherModule.class})
@Singleton
interface AppComponent {
    ActivityComponent plusActivityComponent(ActivityModule activityModule);
    // injects
}
```

```java
@Subcomponent (modules={ActivityModule.class})
@ActivityScope
interface ActivityComponent {
    // injects
}
```

Instantiation.

```java
appComponent = DaggerAppComponent.builder()
    .contextModule(new ContextModule(getApplicationContext()))
    .otherModule(new OtherModule())
    .build();
activityComponent = appComponent.plusActivityComponent(new ActivityModule());
```

<a name="koin-tag"></a>
# Koin [![Maven][koin-mavenbadge]][koin-maven] [![Source][koin-sourcebadge]][koin-source]

A pragmatic lightweight dependency injection framework for Kotlin developers. Written in pure Kotlin, using functional resolution only: no proxy, no code generation, no reflection.

```gradle
dependencies {
    implementation "org.koin:koin-android:2.0.1"
    
    // Koin Android Scope features
    implementation "org.koin:koin-android-scope:2.0.1"
    // Koin Android ViewModel features
    implementation "org.koin:koin-android-viewmodel:2.0.1"
    // Koin Android Experimental features
    implementation "org.koin:koin-android-ext:2.0.1"
}
```

### Koin. Basic Usage

Declare modules.

```kotlin
// Given some classes
class Controller(val service : BusinessService)
class BusinessService()

val myModule = module {
    single { BusinessService() }                     // Create singleton.
    
    single(named("mock")) { MockBusinessService() }  // Named.
    
    single { HttpClient(getProperty("server_url")) } // getProperty() resolves from Koin properties.
    
    factory { Controller(get()) }                    // Create a new instance each time. get() resolves dependencies.
    
    viewModel { MyViewModel(get()) }                 // Create ViewModel.

}
```

Start Koin.

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidLogger()                    // Optional, for logging.
            androidContext(this@MyApplication)
            androidFileProperties()            // Optional to load properties from assets.
            modules(myModule, myOtherModule)
        }
    }
}
```

Inject.

```kotlin
class MyActivity() : AppCompatActivity() {

    val controller : Controller by inject() // lazy
    val myViewModel: MyViewModel by viewModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val service : BusinessService = get() // directly
    }
}
```

### Koin. Scopes

MyScopePresenter class declared as a scoped definition for MyScopeActivity. This will allows to bind a MyScopePresenter with a scope, and drop this instance with the scope closing.

```kotlin
val appModule = module {

    // Single instance of HelloRepository.
    single<HelloRepository> { HelloRepositoryImpl() }

    // Scoped MyScopePresenter instance.
    scope(named<MyScopeActivity>()) {
        scoped { MyScopePresenter(get()) }
    }
}
```

The currentScope allows to retrieve/create a Koin scope for given activity.

```kotlin
class MyScopeActivity : AppCompatActivity() {

    // inject MyScopePresenter from current scope 
    val scopePresenter: MyScopePresenter by currentScope.inject()

    ...
}
```

### Koin. Injecting into Java

```java
public class JavaActivity extends AppCompatActivity {

    private Lazy<MyJavaPresenter> javaPresenter = inject(MyJavaPresenter.class);

    ...
}
```

<a name="ormlite-tag"></a>
# ORMLite [![Maven][ormlite-mavenbadge]][ormlite-maven] [![Source][ormlite-sourcebadge]][ormlite-source]

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

<a name="greendao-tag"></a>
# GreenDAO [![Maven][greendao-mavenbadge]][greendao-maven] [![Source][greendao-sourcebadge]][greendao-source]

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

<a name="objectbox-tag"></a>
# ObjectBox [![Maven][objectbox-mavenbadge]][objectbox-maven] [![Source][objectbox-sourcebadge]][objectbox-source]

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

<a name="realm-tag"></a>
# Realm [![Maven][realm-mavenbadge]][realm-maven] [![Source][realm-sourcebadge]][realm-source]

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

<a name="room-tag"></a>
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

<a name="eventbus-tag"></a>
# EventBus [![Maven][eventbus-mavenbadge]][eventbus-maven] [![Source][eventbus-sourcebadge]][eventbus-source]

Event bus for Android and Java that simplifies communication between Activities, Fragments, Threads, Services, etc. Less code, better quality.

```gradle
dependencies {
    implementation 'org.greenrobot:eventbus:3.1.1'
}
```

- Simplifies the communication between components.

- Decouples event senders and receivers.

- Performs well with Activities, Fragments, and background threads.

- Avoids complex and error-prone dependencies and life cycle issues.

- Has advanced features like delivery threads, sticky events, subscriber priorities, event cancellation.

- Fast, small, proven in practice.

### EventBus. Usage

Define events. Events are POJOs without any specific requirements.

```java
public static class MessageEvent { /* Additional fields if needed */ }
```

Prepare subscribers. Subscribers implement event handling methods (also called “subscriber methods”) that will be called when an event is posted. Declare and annotate your subscribing method, optionally specify a thread mode.

```java
@Subscribe(threadMode = ThreadMode.MAIN)  
public void onMessageEvent(MessageEvent event) {/* Do something */};
```

Register and unregister your subscriber. For example on Android, activities and fragments should usually register according to their life cycle:

```java
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

@Override
public void onStop() {
    super.onStop();
    EventBus.getDefault().unregister(this);
}
```

Post events:

```java
EventBus.getDefault().post(new MessageEvent());
```

### EventBus. Delivery Threads

- **ThreadMode: POSTING**. Subscribers will be called in the same thread posting the event. Default. Synchronous. Implies the least overhead because it avoids thread switching completely. Recommended for simple fast tasks. Event handlers using this mode should return quickly to avoid blocking the posting thread, which may be the main thread.

- **ThreadMode: MAIN**. Subscribers will be called in Android’s main thread (UI thread). If the posting thread is the main thread, event handler methods will be called directly (synchronously). Event handlers using this mode must return quickly to avoid blocking the main thread.

- **ThreadMode: MAIN_ORDERED**. Subscribers will be called in Android’s main thread. The event is always enqueued for later delivery to subscribers, so the call to post will return immediately. This gives event processing a stricter and more consistent order (thus the name MAIN_ORDERED). For example if you post another event in an event handler with MAIN thread mode, the second event handler will finish before the first one (because it is called synchronously – compare it to a method call). With MAIN_ORDERED, the first event handler will finish, and then the second one will be invoked at a later point in time (as soon as the main thread has capacity). Event handlers using this mode must return quickly to avoid blocking the main thread.

- **ThreadMode: BACKGROUND**. Subscribers will be called in a background thread. If posting thread is not the main thread, event handler methods will be called directly in the posting thread. If the posting thread is the main thread, EventBus uses a single background thread that will deliver all its events sequentially. Event handlers using this mode should try to return quickly to avoid blocking the background thread.

- **ThreadMode: ASYNC**. Event handler methods are called in a separate thread. This is always independent from the posting thread and the main thread. Posting events never wait for event handler methods using this mode. Event handler methods should use this mode if their execution might take some time, e.g. for network access. Avoid triggering a large number of long running asynchronous handler methods at the same time to limit the number of concurrent threads. EventBus uses a thread pool to efficiently reuse threads from completed asynchronous event handler notifications.

<a name="rxjava-tag"></a>
# RxJava [![Maven][rxjava-mavenbadge]][rxjava-maven] [![Source][rxjava-sourcebadge]][rxjava-source] RxAndroid [![Maven][rxandroid-mavenbadge]][rxandroid-maven] [![Source][rxandroid-sourcebadge]][rxandroid-source]

RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and event-based programs by using observable sequences. It extends the observer pattern to support sequences of data/events and adds operators that allow you to compose sequences together declaratively while abstracting away concerns about things like low-level threading, synchronization, thread-safety and concurrent data structures.

```gradle
dependencies {
    implementation 'io.reactivex.rxjava2:rxjava:2.2.7'
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
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

### RxJava. Basic usage

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

Same with lambdas.

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

### RxJava. Terminology

**Upstream, downstream**. The dataflows in RxJava consist of a source, zero or more intermediate steps followed by a data consumer or combinator step (where the step is responsible to consume the dataflow by some means). For intermediate operator, looking to the left towards the source, is called the **upstream**. Looking to the right towards the subscriber/consumer, is called the **downstream**.

**Backpressure**. When the dataflow runs through asynchronous steps, each step may perform different things with different speed. To avoid overwhelming such steps, which usually would manifest itself as increased memory usage due to temporary buffering or the need for skipping/dropping data, a so-called backpressure is applied, which is a form of flow control where the steps can express how many items are they ready to process. This allows constraining the memory usage of the dataflows in situations where there is generally no way for a step to know how many items the upstream will send to it.

**Assembly time**. The preparation of dataflows by applying various intermediate operators happens in the so-called assembly time. At this point, the data is not flowing yet and no side-effects are happening.

**Subscription time**. This is a temporary state when subscribe() is called on a flow that establishes the chain of processing steps internally. This is when the subscription side-effects are triggered (see doOnSubscribe). Some sources block or start emitting items right away in this state.

**Runtime**. This is the state when the flows are actively emitting items, errors or completion signals. Practically, this is when the body of the code executes.

### RxJava. Base observable classes

RxJava 2 features several base classes you can discover operators on:

- **Flowable**: 0..N flows, supporting Reactive-Streams and backpressure, which controls how fast a source emits items.

- **Observable**: 0..N flows, no backpressure.

- **Single**: a flow of exactly 1 item or an error, the reactive version of a method call.

- **Completable**: a flow without items but only a completion or error signal, the reactive version of a Runnable.

- **Maybe**: a flow with no items, exactly one item or an error.

### RxJava. Operators

- **subscribeOn(Scheduler)**: By default, an Observable emits its data on the thread where the subscription was declared, i.e. where you called the .subscribe method. In Android, this is generally the main UI thread. You can use the subscribeOn() operator to define a different Scheduler where the Observable should execute and emit its data. The subscribeOn() operator will have the same effect no matter where you place it in the observable chain. You can't use multiple subscribeOn() operators in the same chain. The chain will only use the subscribeOn() that’s the closest to the source observable.

- **observeOn(Scheduler)**: You can use this operator to redirect your Observable’s emissions to a different Scheduler, effectively changing the thread where the Observable’s notifications are sent, and by extension the thread where its data is consumed. Unlike subscribeOn(), where you place observeOn() in your chain does matter, as this operator only changes the thread that’s used by the observables that appear downstream.

- **flatMap()**: Transform the items emitted by an Observable into Observables, then flatten the emissions from those into a single Observable.

- **zip()**: Combine the emissions of multiple Observables together via a specified function and emit single items for each combination based on the results of this function.

- **filter()**: Emit only those items from an Observable that pass a predicate test.

And many more: http://reactivex.io/documentation/operators.html

### RxJava. Schedulers types

- **Schedulers.immediate()**: Creates and returns a Scheduler that executes work immediately on the current thread.

- **Schedulers.trampoline()**: Creates and returns a Scheduler that queues work on the current thread to be executed after the current work completes.

- **Schedulers.newThread()**: Creates and returns a Scheduler that creates a new Thread for each unit of work.

- **Schedulers.computation()**: Creates and returns a Scheduler intended for computational work. This can be used for event-loops, processing callbacks and other computational work. Do not perform IO-bound work on this scheduler. Use Schedulers.io() instead.

- **Schedulers.io()**: Creates and returns a Scheduler intended for IO-bound work. The implementation is backed by an Executor thread-pool that will grow as needed. This can be used for asynchronously performing blocking IO. Do not perform computational work on this scheduler. Use Schedulers.computation() instead.

- **Schedulers.test()**: Creates and returns a TestScheduler, which is useful for debugging. It allows you to test schedules of events by manually advancing the clock at whatever pace you choose.

- **AndroidSchedulers.mainThread()**: This Scheduler is provided by rxAndroid library. This is used to bring back the execution to the main thread so that UI modification can be made. This is usually used in observeOn method.

<a name="okhttp-tag"></a>
# OkHttp [![Maven][okhttp-mavenbadge]][okhttp-maven] [![Source][okhttp-sourcebadge]][okhttp-source]

An HTTP+HTTP/2 client for Android and Java applications.

```gradle
dependencies {
    implementation 'com.squareup.okhttp3:okhttp:3.13.1'
}
```

- HTTP/2 support allows all requests to the same host to share a socket.
- Connection pooling reduces request latency (if HTTP/2 isn’t available).
- Transparent GZIP shrinks download sizes.
- Response caching avoids the network completely for repeat requests.

Each call OkHttp can rewrite your request, rewrite response, redirect, follow-up and retry. OkHttp uses Call to model the task of satisfying your request through however many intermediate requests and responses are necessary.

### OkHttp. Basic usage

```java
OkHttpClient client = new OkHttpClient();
Response response = client.newCall(request).execute()
```

Get.

```java
OkHttpClient client = new OkHttpClient();

String run(String url) throws IOException {
    Request request = new Request.Builder()
        .url(url)
        .build();

    try (Response response = client.newCall(request).execute()) {
        return response.body().string();
    }
}
```

Post.

```java
public static final MediaType JSON 
    = MediaType.get("application/json; charset=utf-8");

OkHttpClient client = new OkHttpClient();

String post(String url, String json) throws IOException {
    RequestBody body = RequestBody.create(JSON, json);
    Request request = new Request.Builder()
        .url(url)
        .post(body)
        .build();
        
    try (Response response = client.newCall(request).execute()) {
        return response.body().string();
    }
}
```

Asynchronous Get.

```java
private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    Request request = new Request.Builder()
    .url("http://publicobject.com/helloworld.txt")
    .build();

    client.newCall(request).enqueue(new Callback() {
        @Override public void onFailure(Call call, IOException e) {
            e.printStackTrace();
        }

        @Override public void onResponse(Call call, Response response) throws IOException {
            try (ResponseBody responseBody = response.body()) {
                if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

                Headers responseHeaders = response.headers();
                for (int i = 0, size = responseHeaders.size(); i < size; i++) {
                    System.out.println(responseHeaders.name(i) + ": " + responseHeaders.value(i));
                }

                System.out.println(responseBody.string());
            }
        }
    });
}
```

### OkHttp. Connections

When you request a URL with OkHttp, here's what it does:

- It uses the URL and configured OkHttpClient to create an address. This address specifies how we'll connect to the webserver. It specifies a webserver (like github.com) and all of the static configuration necessary to connect to that server: the port number, HTTPS settings, and preferred network protocols (like HTTP/2 or SPDY).
- It attempts to retrieve a connection with that address from the connection pool.
- If it doesn't find a connection in the pool, it selects a route to attempt. This usually means making a DNS request to get the server's IP addresses. It then selects a TLS version and proxy server if necessary.
- If it's a new route, it connects by building either a direct socket connection, a TLS tunnel (for HTTPS over an HTTP proxy), or a direct TLS connection. It does TLS handshakes as necessary.
- It sends the HTTP request and reads the response.

### OkHttp. Interceptors

Interceptors are a powerful mechanism that can monitor, rewrite, and retry calls. Examples are logging, compressing, checksumming interceptors etc. Interceptors can be chained.

```java
class LoggingInterceptor implements Interceptor {
    @Override public Response intercept(Interceptor.Chain chain) throws IOException {
        Request request = chain.request();

        long t1 = System.nanoTime();
        logger.info(String.format("Sending request %s on %s%n%s",
            request.url(), chain.connection(), request.headers()));

        Response response = chain.proceed(request);

        long t2 = System.nanoTime();
        logger.info(String.format("Received response for %s in %.1fms%n%s",
            response.request().url(), (t2 - t1) / 1e6d, response.headers()));

        return response;
    }
}
```

```java
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new LoggingInterceptor())
    .build();
```

<a name="retrofit-tag"></a>
# Retrofit [![Maven][retrofit-mavenbadge]][retrofit-maven] [![Source][retrofit-sourcebadge]][retrofit-source]

Type-safe HTTP client. Provides a powerful framework for authenticating and interacting with APIs and sending network requests with OkHttp. Automatically serialises the JSON response using a POJO. For conversion uses GSON or several other converters.

```gradle
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.5.0'
    
    // Or any other available converter.
    implementation 'com.google.code.gson:gson:2.8.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.4.0'
    
    // To use with RxJava.
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.4.0'
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
    .addCallAdapterFactory(RxJava2CallAdapterFactory.create()) // For RxJava support.
    .build();
```

### Retrofit. Endpoints

The endpoints are defined inside of an interface using special retrofit annotations to encode details about the parameters and request method. The return value is always a parameterized Call<T>. This is just a callback given by the retrofit to perform the request synchronously or asynchronously.

```java
public interface ApiServiceInterface  {
    @GET("users/{username}")
    Call<User> getUser(@Path("username") String username);
    
    @GET("users/{username}")
    Observable<User> getUser(@Path("username") String username); // RxJava Observable

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

Call<User> call = apiService.createUser(user);
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

<a name="junit-tag"></a>
# JUnit [![Maven][junit-mavenbadge]][junit-maven] [![Source][junit-sourcebadge]][junit-source]

Popular unit testing framework.

```gradle
dependencies {
    testImplementation 'junit:junit:4.12'
}
```

### Unit tests

Located at module-name/src/test/java/ .

Unit tests run on local machine only. These tests are compiled to run locally on the Java Virtual Machine (JVM) to minimize execution time.

Running unit tests against your code, you can easily verify that the logic of individual units is correct. Running unit tests after every build helps you to quickly catch and fix software regressions introduced by code changes to your app. A unit test generally exercises the functionality of the smallest possible unit of code (which could be a method, class, or component) in a repeatable way.

JUnit assumes that all test methods can be executed in an arbitrary order. Well-written test code should not assume any order, i.e., tests should not depend on other tests.

By default, the Android Plug-in for Gradle executes your local unit tests against a modified version of the android.jar library, which does not contain any actual code. Instead, method calls to Android classes from your unit test throw an exception. This is to make sure you test only your code and do not depend on any particular behavior of the Android platform (that you have not explicitly built or mocked).

### JUnit. Basic test

```java
public class CalculatorTest {
    @Test
    public void calculatorTest_multiplicationOfZeroShouldReturnZero() {
        MyCalculator calc = new MyCalculator();
        assertEquals(0, calc.multiply(10, 0), "10 x 0 must be 0");
    }
}
```

### JUnit. Annotations

- **@Test** Identifies a method as a test method.

- **@BeforeClass** Executed once, before the start of all tests. It is used to perform time intensive activities, for example, to connect to a database. Methods need to be static.

- **@Before** Executed before each test. It is used to prepare the test environment (e.g., read input data, initialize the class).

- **@After** Executed after each test. It is used to cleanup the test environment (e.g., delete temporary data, restore defaults). It can also save memory by cleaning up expensive memory structures.

- **@AfterClass** Executed once, after all tests have been finished. It is used to perform clean-up activities, for example, to disconnect from a database. Methods need to be static.

- **@Ignore or @Ignore("Why disabled")** Marks that the test should be disabled. This is useful when the underlying code has been changed and the test case has not yet been adapted. Or if the execution time of this test is too long to be included.

- **@Test (expected = Exception.class)** Fails if the method does not throw the named exception.

- **@Test(timeout = 100)** Fails if the method takes longer than 100 milliseconds.

### JUnit. Assert statements

- fail([message])
- assertTrue([message,] boolean condition)
- assertFalse([message,] boolean condition)
- assertEquals([message,] expected, actual)
- assertEquals([message,] expected, actual, tolerance)
- assertNull([message,] object)
- assertNotNull([message,] object)
- assertSame([message,] expected, actual)
- assertNotSame([message,] expected, actual)

<a name="mockito-tag"></a>
# Mockito [![Maven][mockito-mavenbadge]][mockito-maven] [![Source][mockito-sourcebadge]][mockito-source]

Most popular Mocking framework for unit tests written in Java. Cannot mock static methods and private methods.

```gradle
dependencies {
    testImplementation 'junit:junit:4.12'
    testImplementation 'org.mockito:mockito-core:2.24.5'
}
```

If you have minimal Android dependencies and need to test specific interactions between a component and its dependency within your app, use a mocking framework to stub out external dependencies in your code. That way, you can easily test that your component interacts with the dependency in an expected way. By substituting Android dependencies with mock objects, you can isolate your unit test from the rest of the Android system while verifying that the correct methods in those dependencies are called.

A mock object is a dummy implementation for an interface or a class in which you define the output of certain method calls. Mock objects are configured to perform a certain behavior during a test. They typically record the interaction with the system and tests can validate that.

### Mockito. Mock dependencies

- To create a mock object for an Android dependency, add the @Mock annotation before the field declaration or use mock() method.
- Initialize mocks with MockitoAnnotations.initMocks(this).
- To stub the behavior of the dependency, you can specify a condition and return value when the condition is met by using the when().thenReturn() or doReturn().when(). The latter doesn't call the real method and can be used to avoid excepcions.

```java
public class PreferencesHelperTest {
    @Mock private SharedPreferences mSharedPreferences;
    private PreferencesHelper mPreferencesHelper;

    @Before
    public void setUp() throws Exception {
        MockitoAnnotations.initMocks(this);
        mPreferencesHelper = new PreferencesHelper(mSharedPreferences);
    }

    @Test
    public void getPreference() {
        when(mSharedPreferences.getString(eq(SharedPreferencesHelper.KEY_NAME), anyString()))
			.thenReturn(TEST_STRING);

        assertEquals(mPreferencesHelper.getSetting(), TEST_STRING);
    }
}
```

### Mockito. Spy

**@Spy** or the **spy()** method can be used to wrap a real object. Every call, unless specified otherwise, is delegated to the object.

```java
@Test
public void testSpy() {
    List<String> list = new LinkedList<>();
    List<String> spy = spy(list);

    doReturn("foo").when(spy).get(0);

    assertEquals("foo", spy.get(0));
}
```

### Mockito. Behavior testing

You can use the verify() method on the mock object to verify that the specified conditions are met. For example, you can verify that a method has been called with certain parameters.

```java
@Test
public void testVerify()  {
    MyClass test = Mockito.mock(Someclass.class);

    test.someEntryMethod(12);

    verify(test).someMethod(ArgumentMatchers.eq(12));
    verify(test, times(2)).someMethod();
    verify(test, never()).someMethod("never called");
    verify(test, atLeastOnce()).someMethod("called at least once");
    verify(test, atLeast(2)).someMethod("called at least twice");
    verify(test, times(5)).someMethod("called five times");
    verify(test, atMost(3)).someMethod("called at most 3 times");
    verifyNoMoreInteractions(test);
}
```

<a name="powermock-tag"></a>
# PowerMock [![Maven][powermock-mavenbadge]][powermock-maven] [![Source][powermock-sourcebadge]][powermock-source]

PowerMock is a framework that extends other mock libraries such as Mockito and EasyMock, but adds more capacity. PowerMock uses custom classloader and bytecode manipulation to allow mocking of static methods, builders, final classes, private methods and more. So we can test our code without modifying it, thanks to the fact that the mock objects are hooked to the code to be tested in execution time. By using reflection this framework allows you to modify even private attributes of the class.

```gradle
dependencies {
    testImplementation 'org.powermock:powermock-core:2.0.0' 
}
```

### PowerMock. Test setup

```java
@RunWith(PowerMockRunner.class)
@PrepareForTest( { YourClassWithEgStaticMethod.class })
public class YourTestCase {
...
}
```

### PowerMock. Bypass encapsulation

**Whitebox** class provides a set of methods which could you help bypass encapsulation if it required. Usually, it's not a good idea to get/modify non-public fields, but sometimes it's only way to cover code by test for future refactoring.

- **Whitebox.setInternalState(..)** to set a non-public member of an instance or class.
- **Whitebox.getInternalState(..)** to get a non-public member of an instance or class.
- **Whitebox.invokeMethod(..)** to invoke a non-public method of an instance or class.
- **Whitebox.invokeConstructor(..)** to create an instance of a class with a private constructor.
- **Whitebox.invokeMethod(..)** to invoke a private method.
- **WhiteBox.invokeConstructor(..)** to instantiate a class with a private constructor.

```java
public class ServiceHolder {
    private final Set<Object> services = new HashSet<Object>();

    public void addService(Object service) { services.add(service); }
    public void removeService(Object service) { services.remove(service); }
}

@Test
public void testAddService() throws Exception {
    ServiceHolder tested = new ServiceHolder();
    final Object service = new Object();

    tested.addService(service);

    // This is how you get the private services set using PowerMock
    Set<String> services = Whitebox.getInternalState(tested, "services");
    // Or
    Set<String> services = Whitebox.getInternalState(tested, Set.class);

    assertEquals("Size of the \"services\" Set should be 1",
        1, services.size());
    assertSame("The services Set should didn't contain the expect service",
        service, services.iterator().next());
}
```

### PowerMock. Suppressing unwanted behavior

- **Whitebox.newInstance(ClassWithEvilConstructor.class)** to instantiate a class without invoking the constructor what so ever.
- **suppress(constructor(EvilParent.class))** to suppress all constructors for the EvilParent class.
- **suppress(method(ClassWithEvilMethod.class, "methodName"))** to suppress the method in the class.
- **suppress(field(ClassWithEvilField.class, "fieldName"))** to suppress the field in the class.
- **@SuppressStaticInitializationFor("org.mycompany.SomeClass")** annotation to remove the static initializer for the class.

<a name="espresso-tag"></a>
# Espresso [![Maven][espresso-mavenbadge]][espresso-maven] [![Source][espresso-sourcebadge]][espresso-source]

UI testing framework. Provides APIs for writing UI tests to simulate user interactions within an app. Provides automatic synchronization of test actions with the UI of the app you are testing. Espresso detects when the main thread is idle, so it is able to run test commands at the appropriate time, improving the reliability of tests. Uses a separate apk to run tests.

```gradle
dependencies {
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.2-alpha01'
    
    // old namespace 'com.android.support.test.espresso:espresso-core:3.0.2'
}
```

### Instrumented (or Instrumentation, or UI) tests

Located at module-name/src/androidTest/java/ .

You can run instrumented unit tests on a physical device or emulator, which doesn't involve any mocking or stubbing of the framework. Because this form of testing involves significantly slower execution times than local unit tests, however, it's best to rely on this method only when it's essential to evaluate your app's behavior against actual device hardware.

### Espresso. Tests structure

- Activity will be launched using the @Rule annotation. By default the rule will be initialised and the activity will be launched before running every @Before method. Destroyed after running the @After method.

- Use JUnit annotations: @Test, @BeforeClass, @Before, @After, @AfterClass.

- Find the UI component you want to test in an Activity (for example, a sign-in button in the app) by calling the onView() method, or the onData() method for AdapterView controls, with specified matcher parameter.

- Simulate a specific user interaction to perform on that UI component, by calling the ViewInteraction.perform() or DataInteraction.perform() method and passing in the user action (for example, click on the sign-in button). To sequence multiple actions on the same UI component, chain them using a comma-separated list in your method argument.

- Use the ViewAssertions methods with ViewInteraction.check() or DataInteraction.check() to check that the UI reflects the expected state or behavior, after these user interactions are performed.

### Espresso. Basic test

```java
onView(ViewMatcher)
    .perform(ViewAction)
    .check(ViewAssertion);
```

```java
@RunWith(AndroidJUnit4.class)
public class ActivityTest {

    @Rule
    public final ActivityTestRule<MyActivity> mActivityTestRule
		= new ActivityTestRule<>(MyActivity.class);

    @Test
    public void basicTest() {
        onView(withId(R.id.my_view)).perform(click()).check(matches(isDisplayed()));
    }
}
```

### Espresso. Test examples

```java
// Testing view visibility.
onView(withText(R.string.some_text)).check(matches(isDisplayed()));
onView(withId(R.id.some_view)).check(matches(not(isDisplayed())));
onView(withText(containsString("some_word"))).check(doesNotExist());
onView(allOf(
    hasSibling(withText(R.string.prefs_date)),
    withText(R.string.prefs_not_set)))
    .check(matches(isDisplayed()));

// Checking view positioning.
onView(withText(R.string.some_text)).check(isCompletelyAbove(withText(R.string.other_text)));

// Clicking view on a ViewPager.
onView(allOf(withText(R.string.some_text), isCompletelyDisplayed())).perform(click());

// Typing text in EditText.
onView(withId(android.R.id.edit)).perform(typeText(text));

// Picking date on DatePicker.
onView(withClassName(equalTo(DatePicker.class.getName())))
    .perform(PickerActions.setDate(year, month, day));

// Scrolling ScrollView.
onView(withId(R.id.some_view)).perform(scrollTo()).check(matches(isDisplayed()));

// Scrolling RecyclerView.
onView(withId(R.id.recycler_view))
    .perform(RecyclerViewActions.scrollTo(hasDescendant(withText(R.string.some_text))));
```

<br><br>

<a name="sources-tag"></a>
# Sources

### Koin

https://github.com/InsertKoinIO/koin
<br>
https://insert-koin.io/
<br>
https://insert-koin.io/docs/2.0/getting-started/introduction/

### ObjectBox

https://docs.objectbox.io/
<br>
https://github.com/objectbox/objectbox-java

### Realm

https://github.com/realm/realm-java
<br>
https://realm.io/docs/java/latest/
<br>
https://realm.io/blog/realm-for-android/

### DBMS comparisons

https://github.com/Rexee/AndroidDatabaseLibraryComparison
<br>
https://github.com/AlexeyZatsepin/Android-ORM-benchmark
<br>
https://proandroiddev.com/android-databases-performance-crud-a963dd7bb0eb (recent)



[gson-maven]: https://search.maven.org/artifact/com.google.code.gson/gson
[gson-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.google.code.gson/gson/badge.svg
[gson-source]: https://github.com/google/gson
[gson-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[glide-maven]: https://search.maven.org/artifact/com.github.bumptech.glide/glide
[glide-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.github.bumptech.glide/glide/badge.svg
[glide-source]: https://github.com/bumptech/glide
[glide-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[dagger-maven]: https://search.maven.org/artifact/com.google.dagger/dagger
[dagger-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.google.dagger/dagger/badge.svg
[dagger-source]: https://github.com/google/dagger
[dagger-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[koin-maven]: https://bintray.com/ekito/koin/koin-core/_latestVersion
[koin-mavenbadge]: https://api.bintray.com/packages/ekito/koin/koin-core/images/download.svg
[koin-source]: https://github.com/InsertKoinIO/koin
[koin-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[rxjava-maven]: https://search.maven.org/artifact/io.reactivex.rxjava2/rxjava
[rxjava-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxjava/badge.svg
[rxjava-source]: https://github.com/ReactiveX/RxJava
[rxjava-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[rxandroid-maven]: https://search.maven.org/artifact/io.reactivex.rxjava2/rxandroid
[rxandroid-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxandroid/badge.svg
[rxandroid-source]: https://github.com/ReactiveX/RxAndroid
[rxandroid-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[butterknife-maven]: https://search.maven.org/artifact/com.jakewharton/butterknife
[butterknife-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.jakewharton/butterknife/badge.svg
[butterknife-source]: https://github.com/JakeWharton/butterknife
[butterknife-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[ormlite-maven]: https://search.maven.org/artifact/com.j256.ormlite/ormlite-android
[ormlite-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.j256.ormlite/ormlite-android/badge.svg
[ormlite-source]: https://github.com/j256/ormlite-android
[ormlite-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[greendao-maven]: https://search.maven.org/artifact/org.greenrobot/greendao
[greendao-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.greenrobot/greendao/badge.svg
[greendao-source]: https://github.com/greenrobot/greenDAO
[greendao-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[objectbox-maven]: https://bintray.com/objectbox/objectbox/io.objectbox:objectbox-gradle-plugin/_latestVersion
[objectbox-mavenbadge]: https://api.bintray.com/packages/objectbox/objectbox/io.objectbox:objectbox-gradle-plugin/images/download.svg
[objectbox-source]: https://github.com/objectbox/objectbox-java
[objectbox-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[realm-maven]: https://bintray.com/realm/maven/realm-gradle-plugin/_latestVersion
[realm-mavenbadge]: https://api.bintray.com/packages/realm/maven/realm-gradle-plugin/images/download.svg
[realm-source]: https://github.com/realm/realm-java
[realm-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[room-maven]: https://mvnrepository.com/artifact/androidx.room/room-runtime
[room-mavenbadge]: https://img.shields.io/badge/maven%20google--brightgreen.svg
[room-source]: https://android.googlesource.com/platform/frameworks/support/+/androidx-master-dev/room/
[room-sourcebadge]: https://img.shields.io/badge/source-google-orange.svg

[eventbus-maven]: https://search.maven.org/artifact/org.greenrobot/eventbus
[eventbus-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.greenrobot/eventbus/badge.svg
[eventbus-source]: https://github.com/greenrobot/EventBus
[eventbus-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[retrofit-maven]: https://search.maven.org/artifact/com.squareup.retrofit2/retrofit
[retrofit-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.squareup.retrofit2/retrofit/badge.svg
[retrofit-source]: https://github.com/square/retrofit
[retrofit-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[okhttp-maven]: https://search.maven.org/artifact/com.squareup.okhttp3/okhttp
[okhttp-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.squareup.okhttp3/okhttp/badge.svg
[okhttp-source]: https://github.com/square/okhttp
[okhttp-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[junit-maven]: https://search.maven.org/artifact/junit/junit
[junit-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/junit/junit/badge.svg
[junit-source]: https://github.com/junit-team/junit4
[junit-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[mockito-maven]: https://search.maven.org/artifact/org.mockito/mockito-core
[mockito-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.mockito/mockito-core/badge.svg
[mockito-source]: https://github.com/mockito/mockito
[mockito-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[powermock-maven]: https://search.maven.org/artifact/org.powermock/powermock-core
[powermock-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.powermock/powermock-core/badge.svg
[powermock-source]: https://github.com/powermock/powermock
[powermock-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg

[espresso-maven]: https://mvnrepository.com/artifact/androidx.test.espresso/espresso-core
[espresso-mavenbadge]: https://img.shields.io/badge/maven%20google--brightgreen.svg
[espresso-source]: https://android.googlesource.com/platform/frameworks/testing/+/android-support-test/espresso
[espresso-sourcebadge]: https://img.shields.io/badge/source-google-orange.svg
