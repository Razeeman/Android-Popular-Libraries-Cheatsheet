# MISC

- [GSON](#gson---)
- [Moshi](#moshi---)
- [Butter Knife](#butter-knife---)
- [Exoplayer](#exoplayer---)
- [Timber](#timber---)
- [Icepick](#icepick---)
- [LeakCanary](#leakcanary---)
- [Cicerone](#cicerone---)
- [Litho](#litho---)
- [Sources](#sources)



# GSON [![Maven][gson-mavenbadge]][gson-maven] [![Source][gson-sourcebadge]][gson-source] ![gson-starsbadge]

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

[Content](#misc)
# Moshi [![Maven][moshi-mavenbadge]][moshi-maven] [![Source][moshi-sourcebadge]][moshi-source] ![moshi-starsbadge]

Moshi is a modern JSON library for Android and Java. It makes it easy to parse JSON into Java objects. Uses Okio for I/O. Works with Kotlin. Supposed to be GSON3.0.

```gradle
dependencies {
    implementation 'com.squareup.moshi:moshi:1.8.0'
}
```

### Moshi. Basic usage

Parse JSON into Java objects:

```java
class User {
    String username;
    @Json(name = "pwd") String password;
    private transient int total;

    ...
}
```

```java
Moshi moshi = new Moshi.Builder().build();

String json = ...;
JsonAdapter<User> jsonAdapter = moshi.adapter(User.class);
User user = jsonAdapter.fromJson(json);

String json = ...;
Type type = Types.newParameterizedType(List.class, User.class);
JsonAdapter<List<User>> adapter = moshi.adapter(type);
List<User> users = adapter.fromJson(json);
```

Serialize Java objects as JSON:

```java
Moshi moshi = new Moshi.Builder().build();

User user = new User(...);
JsonAdapter<User> jsonAdapter = moshi.adapter(User.class);
String json = jsonAdapter.toJson(user);
```

### Moshi. Adapters

Moshi has built-in support for reading and writing Java’s core data types:

- Primitives (int, float, char...) and their boxed counterparts (Integer, Float, Character...).

- Arrays, Collections, Lists, Sets, and Maps

- Strings

- Enums

### Moshi. Custom adapter

A type adapter is any class that has methods annotated **@ToJson** and **@FromJson**.

```java
class CardAdapter {
    @ToJson String toJson(Card card) {
        return card.rank + card.suit.name().substring(0, 1);
    }

    @FromJson Card fromJson(String card) {
        if (card.length() != 2) throw new JsonDataException("Unknown card: " + card);

        char rank = card.charAt(0);
        switch (card.charAt(1)) {
            case 'C': return new Card(rank, Suit.CLUBS);
            case 'D': return new Card(rank, Suit.DIAMONDS);
            case 'H': return new Card(rank, Suit.HEARTS);
            case 'S': return new Card(rank, Suit.SPADES);
            default: throw new JsonDataException("unknown suit: " + card);
        }
    }
}
```

```java
Moshi moshi = new Moshi.Builder()
    .add(new CardAdapter())
    .build();
```

### Moshi. Kotlin

May use reflection, codegen, or both.

**Reflection**

The reflection adapter uses Kotlin’s reflection library to convert Kotlin classes to and from JSON.

```gradle
dependencies {
    implementation 'com.squareup.moshi:moshi-kotlin:1.8.0'
}
```

```kotlin
val moshi = Moshi.Builder()
    // Added after custom factories and adapters.
    .add(KotlinJsonAdapterFactory())
    .build()
```

**Codegen**

Moshi’s Kotlin codegen support is an annotation processor. It generates a small and fast adapter for each of Kotlin classes at compile time.

```gradle
dependencies {
    kapt 'com.squareup.moshi:moshi-kotlin-codegen:1.8.0'
}
```

```kotlin
@JsonClass(generateAdapter = true)
data class User(
        val username: String,
        ...
)
```

[Content](#misc)
# Butter Knife [![Maven][butterknife-mavenbadge]][butterknife-maven] [![Source][butterknife-sourcebadge]][butterknife-source] ![butterknife-starsbadge]

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

[Content](#misc)
# Exoplayer [![Maven][exoplayer-mavenbadge]][exoplayer-maven] [![Source][exoplayer-sourcebadge]][exoplayer-source] ![exoplayer-starsbadge]

ExoPlayer is an application level media player for Android. It provides an alternative to Android’s MediaPlayer API for playing audio and video both locally and over the Internet. ExoPlayer supports features not currently supported by Android’s MediaPlayer API, including DASH and SmoothStreaming adaptive playbacks. Unlike the MediaPlayer API, ExoPlayer is easy to customize and extend, and can be updated through Play Store application updates.

```gradle
dependencies {
    implementation 'com.google.android.exoplayer:exoplayer:2.10.1'
    
    // Modularized alternative.
    implementation 'com.google.android.exoplayer:exoplayer-core:2.10.1'
    implementation 'com.google.android.exoplayer:exoplayer-dash:2.10.1'
    implementation 'com.google.android.exoplayer:exoplayer-hls:2.10.1'
    implementation 'com.google.android.exoplayer:exoplayer-smoothstreaming:2.10.1'
    implementation 'com.google.android.exoplayer:exoplayer-ui:2.10.1'
}

compileOptions {
    targetCompatibility JavaVersion.VERSION_1_8
}
```

### Exoplayer. Basic usage

```java
SimpleExoPlayer player = ExoPlayerFactory.newSimpleInstance(context);

// Bind the player to the view.
playerView.setPlayer(player);

// Produces DataSource instances through which media data is loaded.
DataSource.Factory dataSourceFactory = new DefaultDataSourceFactory(context,
    Util.getUserAgent(context, "yourApplicationName"));

// This is the MediaSource representing the media to be played.
MediaSource videoSource = new ProgressiveMediaSource.Factory(dataSourceFactory)
    .createMediaSource(mp4VideoUri);
    
// Prepare the player with the source.
player.prepare(videoSource);

// Add a listener to receive events from the player.
player.addListener(eventListener);

// Control the player.
player.setPlayWhenReady(true);

ExoPlayer.release;
```

### Exoplayer. Media sources

In ExoPlayer every piece of media is represented by MediaSource. The ExoPlayer library provides MediaSource implementations for several stream types:

- **DashMediaSource** for DASH.

- **SsMediaSource** for SmoothStreaming.

- **HlsMediaSource** for HLS.

- **ProgressiveMediaSource** for regular media files.

### Exoplayer. UI components

An app playing media requires user interface components for displaying media and controlling playback. The ExoPlayer library includes a UI module that contains a number of UI components.

```gradle
implementation 'com.google.android.exoplayer:exoplayer-ui:2.10.1'
```

The most important components are PlayerControlView and PlayerView.

- **PlayerControlView** is a view for controlling playbacks. It displays standard playback controls including a play/pause button, fast-forward and rewind buttons, and a seek bar.

- **PlayerView** is a high level view for playbacks. It displays video, subtitles and album art during playback, as well as playback controls using a PlayerControlView.

```xml
<com.google.android.exoplayer2.ui.PlayerView
    android:id="@+id/player_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:show_buffering="when_playing"
    app:show_shuffle_button="true"/>
```

[Content](#misc)
# Timber [![Maven][timber-mavenbadge]][timber-maven] [![Source][timber-sourcebadge]][timber-source] ![timber-starsbadge]

A logger with a small, extensible API which provides utility on top of Android's normal Log class.

```gradle
dependencies {
    implementation 'com.jakewharton.timber:timber:4.7.1'
}
```

### Timber. Basic usage

Behavior is added through Tree instances. You can install an instance by calling ```Timber.plant```. Installation of Trees should be done as early as possible. The onCreate of your application is the most logical choice. The DebugTree implementation will automatically figure out from which class it's being called and use that class name as its tag.

- Install any Tree instances you want in the onCreate of your application class.

- Call Timber's static methods everywhere throughout your app.

```java
public class ExampleApp extends Application {
    @Override public void onCreate() {
        super.onCreate();

        if (BuildConfig.DEBUG) {
            Timber.plant(new Timber.DebugTree());
        }
    }
}
```

```java
Timber.e("Error Message");
Timber.d("Debug Message");

Timber.tag("LifeCycles");
Timber.i("A button with ID %s was clicked to say '%s'.", button.getId(), button.getText());

Timber.tag("Some Different tag").e("Another error message");
```

[Content](#misc)
# Icepick [![Maven][icepick-mavenbadge]][icepick-maven] [![Source][icepick-sourcebadge]][icepick-source] ![icepick-starsbadge]

Icepick is an Android library that eliminates the boilerplate of saving and restoring instance state. It uses annotation processing to generate code that does bundle manipulation and key generation, so that you don't have to write it yourself.

```gradle
repositories {
    maven {url "https://clojars.org/repo/"}
}

dependencies {
    implementation 'frankiesardo:icepick:3.2.0'
    annotationProcessor 'frankiesardo:icepick-processor:3.2.0'
}
```

### Icepick. Basic usage


It works for Activities, Fragments or any object that needs to serialize its state on a Bundle. Can also generate the instance state code for custom Views. You can put the calls to Icepick into a BaseActivity. All Activities extending BaseActivity automatically have state saved/restored.

```java
class ExampleActivity extends Activity {
    // This will be automatically saved and restored.
    @State String username;

    @Override public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        Icepick.restoreInstanceState(this, savedInstanceState);
    }

    @Override public void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        Icepick.saveInstanceState(this, outState);
    }
}
```

### Icepick. Kotlin

```kotlin
class ExampleActivity : Activity() {
    @State @JvmField
    var username: String? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        Icepick.restoreInstanceState(this, savedInstanceState)
    }

    override fun onSaveInstanceState(outState: Bundle?) {
        super.onSaveInstanceState(outState)
        Icepick.saveInstanceState(this, outState)
    }
}
```

[Content](#misc)
# LeakCanary [![Maven][leakcanary-mavenbadge]][leakcanary-maven] [![Source][leakcanary-sourcebadge]][leakcanary-source] ![leakcanary-starsbadge]

A memory leak detection library for Android.

```gradle
dependencies {
    // debugImplementation because LeakCanary should only run in debug builds.
    debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.0-alpha-2'
}
```

### LeakCanary. Basic usage

No code change needed. LeakCanary will autoinstall iteself as a separate app and automatically show a notification when a memory leak is detected in debug builds.

- The library automatically watches destroyed activities and destroyed fragments using weak references. You can also watch any instance that is no longer needed, e.g. a detached view.

- If the weak references aren't cleared, after waiting 5 seconds and running the GC, the watched instances are considered retained, and potentially leaking.

- When the number of retained instances reaches a threshold, LeakCanary dumps the Java heap into a .hprof file stored on the file system. The default threshold is 5 retained instances when the app is visible, 1 otherwise.

- LeakCanary parses the .hprof file and finds the chain of references that prevents retained instances from being garbage collected (leak trace). A leak trace is technically the shortest strong reference path from GC Roots to retained instances, but that's a mouthful.

- Once the leak trace is determined, LeakCanary uses its built in knowledge of the Android framework to deduct which instances in the leak trace are leaking. You can help LeakCanary by providing Reachability inspectors tailored to your own app.

- Using the reachability information, LeakCanary narrows down the reference chain to a sub chain of possible leak causes, and displays the result. Leaks are grouped by identical sub chain.

### LeakCanary. Memory leak fundamentals

In a Java based runtime, a memory leak is a programming error that causes an application to keep a reference to an object that is no longer needed. As a result, the memory allocated for that object cannot be reclaimed, eventually leading to an OutOfMemoryError crash. For example, an Android activity instance is no longer needed after its onDestroy() method is called, and storing a reference to that activity in a static field would prevent it from being garbage collected.

### LeakCanary. LeakSentry

Can be used to watch instances that should be garbage collected.

```kotlin
class MyService : Service {

    ...

    override fun onDestroy() {
        super.onDestroy()
        LeakSentry.refWatcher.watch(this)
    }
}
```

Suitable for release builds to track and count retained instances.

```gradle
dependencies {
    implementation 'com.squareup.leakcanary:leaksentry:2.0-alpha-2'
}
```

```kotlin
val retainedInstanceCount = LeakSentry.refWatcher.retainedKeys.size
```

[Content](#misc)
# Cicerone [![Maven][cicerone-mavenbadge]][cicerone-maven] [![Source][cicerone-sourcebadge]][cicerone-source] ![cicerone-starsbadge]

Lightweight library that makes the navigation in an Android app easy. Designed to be used with the MVP, but will work great with any architecture.

```gradle
dependencies {
    implementation 'ru.terrakok.cicerone:cicerone:5.0.0'
}
```

**Main advantages:**

- is not tied to Fragments
- not a framework
- short navigation calls (no builders)
- lifecycle-safe!
- functionality is simple to extend
- suitable for Unit Testing

**Additional features:**

- opening several screens inside single call (for example: deeplink)
- implementation of parallel navigation (Instagram like)
- predefined navigator ready for Single-Activity apps
- predefined navigator ready for setup transition animation

### Cicerone. Initialization

```java
public class SampleApplication extends Application {
    public static SampleApplication INSTANCE;
    private Cicerone<Router> cicerone;

    @Override
    public void onCreate() {
        super.onCreate();
        INSTANCE = this;

        initCicerone();
    }

    private void initCicerone() {
        cicerone = Cicerone.create();
    }

    public NavigatorHolder getNavigatorHolder() {
        return cicerone.getNavigatorHolder();
    }

    public Router getRouter() {
        return cicerone.getRouter();
    }
}
```

### Cicerone. Basic usage

**Presenter** calls router.navigateTo(). **Router** converts high-level navigation calls to set of special commands. **CommandBuffer** stores commands if navigator doesn't ready for navigation (life-cycle). **Navigator** starts execution of simple navigation commands grouped into batches.

**Presenter** calls navigation method of **Router**.

```java
public class SamplePresenter extends Presenter<SampleView> {
    private Router router;

    public SamplePresenter() {
        router = SampleApplication.INSTANCE.getRouter();
    }

    public void onBackCommandClick() {
        router.exit();
    }

    public void onForwardCommandClick() {
        router.navigateTo(new SomeScreen());
    }
}
```

**Router** converts the navigation call to the set of **Commands** and sends them to **CommandBuffer**. **CommandBuffer** checks whether there are "active" **Navigator**:

- If yes, it passes the commands to the Navigator. Navigator will process them to achive the desired transition.
- If no, then CommandBuffer saves the commands in a queue, and will apply them as soon as new "active" Navigator will appear.

```java
void executeCommands(Command[] commands) {
        if (navigator != null) {
            navigator.applyCommands(commands);
        } else {
            pendingCommands.add(commands);
        }
    }
```

**Navigator** processes the navigation commands. Usually it is an anonymous class inside the Activity. Activity provides **Navigator** to the **CommandBuffer** in onResume and removes it in onPause.

```java
@Override
protected void onResume() {
    super.onResume();
    SampleApplication.INSTANCE.getNavigatorHolder().setNavigator(navigator);
}

@Override
protected void onPause() {
    super.onPause();
    SampleApplication.INSTANCE.getNavigatorHolder().removeNavigator();
}

private Navigator navigator = new Navigator() {
    @Override
    public void applyCommands(Command[] commands) {
        //implement commands logic (apply command batch to navigation container)
    }
};
```

### Cicerone. Navigation commands

- **Forward** - Opens new screen.

- **Back** - Rolls back the last transition.

- **BackTo** - Rolls back to the needed screen in the screens chain.

- **Replace** - Replaces the current screen.

[Content](#misc)
# Litho [![Maven][litho-mavenbadge]][litho-maven] [![Source][litho-sourcebadge]][litho-source] ![litho-starsbadge]

Litho is a declarative framework for building efficient UIs on Android.

```gradle
dependencies {
    implementation 'com.facebook.litho:litho-core:0.36.0'
    implementation 'com.facebook.litho:litho-widget:0.36.0'

    annotationProcessor 'com.facebook.litho:litho-processor:0.36.0'
    
    // SoLoader
    implementation 'com.facebook.soloader:soloader:0.9.0'
}
```

- **Declarative**: Litho uses a declarative API to define UI components. You simply describe the layout for your UI based on a set of immutable inputs and the framework takes care of the rest.
- **Asynchronous layout**: Litho can measure and layout your UI ahead of time without blocking the UI thread.
- **View flattening**: Litho uses Yoga for layout and automatically reduces the number of ViewGroups that your UI contains.
- **Fine-grained recycling**: Any component such as a text or image can be recycled and reused anywhere in the UI.

### Litho. Basic usage

Initialize SoLoader in Application class. Behind the scenes, Litho uses Yoga for layout. Yoga has native dependencies and SoLoader is brought in to take care of loading those. Initializing SoLoader here ensures that you’re not referencing unloaded libraries later on.

```java
public class SampleApplication extends Application {
  @Override
  public void onCreate() {
    super.onCreate();
    SoLoader.init(this, false);
  }
}
```

Create and display a component in your Activity. **LithoView** is an Android ViewGroup that can render components; it is the bridge between Litho components and Android Views.

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    final ComponentContext c = new ComponentContext(this);

    final Component component = Text.create(c)
        .text("Hello World")
        .textSizeDip(50)
        .build();

    setContentView(LithoView.create(c, component));
}
```

### Litho. Custom components

Write **Spec** classes to declare the layout for your components. A component spec class will be processed to generate a ComponentLifecycle subclass with the same name as the spec but without the Spec suffix. For example, a MyComponentSpec spec generates a MyComponent class. At runtime, all component instances of a certain type share the same ComponentLifecycle reference. This means that there will only be one spec instance per component type, not per component instance.

There are two types of component specs:

- **Layout spec**: combines other components into a specific layout. This is the equivalent of a ViewGroup on Android.
- **Mount spec**: a component that can render a view or a drawable.

Example component called ListItem and displays a title with a smaller subtitle underneath:

```java
@LayoutSpec
public class ListItemSpec {

  @OnCreateLayout
  static Component onCreateLayout(ComponentContext c) {
  
    return Column.create(c)
        .paddingDip(ALL, 16)
        .backgroundColor(Color.WHITE)
        .child(
            Text.create(c)
                .text("Hello world")
                .textSizeSp(40))
        .child(
            Text.create(c)
                .text("Litho tutorial")
                .textSizeSp(20))
        .build();
  }
}
```

```java
final Component component = ListItem.create(c).build();
```

### Litho. Lists

You can create lists by using the **RecyclerCollectionComponent** and the **Sections** library. **RecyclerCollectionComponent** is used for creating scrollable units in Litho and it hides some of the complexity of having to work directly with Android’s RecyclerView and Adapter concepts. With the **Sections** API, you group the items in your list into sections and write **GroupSectionSpec** classes to declare what each section renders and what data it uses.

```java
@GroupSectionSpec
public class ListSectionSpec {

  @OnCreateChildren
  static Children onCreateChildren(final SectionContext c) {
    Children.Builder builder = Children.create();

    for (int i = 0; i < 32; i++) {
      builder.child(
          SingleComponentSection.create(c)
              .key(String.valueOf(i))
              .component(ListItem.create(c).build()));
    }
    return builder.build();
  }
}
```

**SingleComponentSection** is a core section that renders a single component. **ListSectionSpec** describes a section that has 32 child sections, each of which is responsible for rendering a ListItem. We can use this section with **RecyclerCollectionComponent** to render our list. RecyclerCollectionComponent takes a section as a prop and renders a RecyclerView containing whatever UI the section outputs. It also manages updates and changes from the section such as refreshing data and performing tail fetches.

```java
final Component component =
    RecyclerCollectionComponent.create(c)
        .section(ListSection.create(new SectionContext(c)).build())
        .build();
```

### Litho. Component properties

Props are parameters to methods of the component specification, annotated with the @Prop annotation. The processor generates methods on the component builder that correspond to the props.

```java
@OnCreateLayout
static Component onCreateLayout(
    ComponentContext c,
    @Prop int color,
    @Prop String title,
    @Prop String subtitle) {

  return Column.create(c)
        .paddingDip(ALL, 16)
        .backgroundColor(color)
        .child(
            Text.create(c)
                .text(title)
                .textSizeSp(40))
        .child(
            Text.create(c)
                .text(subtitle)
                .textSizeSp(20))
        .build();
}
```

```java
ListItem.create(c)
    .color(Color.WHITE)
    .title("Hello, world!")
    .subtitle("Litho tutorial")
    .build()
```

You can specify more options to the @Prop annotation. This tells the annotation processor to construct a number of functions, such as shadowRadiusPx, shadowRadiusDip, shadowRadiusSp as well as shadowRadiusRes:

```java
@Prop(optional = true, resType = ResType.DIMEN_OFFSET) int shadowRadius
```

### Litho. Mount specs

A mount spec defines a component that can render views or drawables. Mount here refers to the operation performed by all components in a layout tree to extract their rendered state (a View or a Drawable) to be displayed. Mount spec classes should be annotated with @MountSpec and implement at least an @OnCreateMountContent method. The other methods listed below are optional.

The lifecycle of mount spec components is as follows:
BG = occurs on BG thread when possible – do not modify the view hierarchy,
UI = can occur on UI thread,
PC = Performance critical – put as little work in it as possible, use BG methods instead

- Run **@OnPrepare** once, before layout calculation. BG/UI
- Run **@OnMeasure** optionally during layout calculation. This will not be called if Yoga has already determined your component’s bounds (e.g. a static width/height was set on the component). BG/UI
- Run **@OnBoundsDefined** once, after layout calculation. This will be called whether or not @OnMeasure was called. BG/UI
- Run **@OnCreateMountContent** before the component is attached to a hosting view. This content may be reused for other instances of this component. UI
- Run **@OnMount** before the component is attached to a hosting view. This will happen when the component is about to become visible when incremental mount is enabled (it is enabled by default). UI/PC
- Run **@OnBind** after the component is attached to a hosting view. UI/PC
- Run **@OnUnbind** before the component is detached from a hosting view. UI/PC
- Run **@OnUnmount** optionally after the component is detached from a hosting view. See incremental mount notes on @OnMount: they apply in reverse here. UI/PC

[Content](#misc)
# Sources

**Moshi**

https://github.com/square/moshi

**Exoplayer**

https://github.com/google/ExoPlayer
<br>
https://exoplayer.dev/

**Timber**

https://github.com/JakeWharton/timber
<br>
https://medium.com/mindorks/better-logging-in-android-using-timber-72e40cc2293d

**Icepick**

https://github.com/frankiesardo/icepick
<br>
https://github.com/frankiesardo/icepick/issues/47

**LeakCanary**

https://github.com/square/leakcanary

**Litho**

https://github.com/facebook/litho
<br>
https://fblitho.com/docs/intro



[gson-maven]: https://search.maven.org/artifact/com.google.code.gson/gson
[gson-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.google.code.gson/gson/badge.svg
[gson-source]: https://github.com/google/gson
[gson-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[gson-starsbadge]: https://img.shields.io/github/stars/google/gson

[moshi-maven]: https://search.maven.org/artifact/com.squareup.moshi/moshi
[moshi-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.squareup.moshi/moshi/badge.svg
[moshi-source]: https://github.com/square/moshi
[moshi-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[moshi-starsbadge]: https://img.shields.io/github/stars/square/moshi

[butterknife-maven]: https://search.maven.org/artifact/com.jakewharton/butterknife
[butterknife-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.jakewharton/butterknife/badge.svg
[butterknife-source]: https://github.com/JakeWharton/butterknife
[butterknife-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[butterknife-starsbadge]: https://img.shields.io/github/stars/JakeWharton/butterknife

[exoplayer-maven]: https://bintray.com/google/exoplayer/exoplayer/_latestVersion
[exoplayer-mavenbadge]: https://api.bintray.com/packages/google/exoplayer/exoplayer/images/download.svg
[exoplayer-source]: https://github.com/google/ExoPlayer
[exoplayer-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[exoplayer-starsbadge]: https://img.shields.io/github/stars/google/ExoPlayer

[timber-maven]: https://search.maven.org/artifact/com.jakewharton.timber/timber
[timber-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.jakewharton.timber/timber/badge.svg
[timber-source]: https://github.com/JakeWharton/timber
[timber-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[timber-starsbadge]: https://img.shields.io/github/stars/JakeWharton/timber

[icepick-maven]: https://clojars.org/frankiesardo/icepick
[icepick-mavenbadge]: https://img.shields.io/clojars/v/frankiesardo/icepick.svg
[icepick-source]: https://github.com/frankiesardo/icepick
[icepick-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[icepick-starsbadge]: https://img.shields.io/github/stars/frankiesardo/icepick

[leakcanary-maven]: https://search.maven.org/artifact/com.squareup.leakcanary/leakcanary-android
[leakcanary-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.squareup.leakcanary/leakcanary-android/badge.svg
[leakcanary-source]: https://github.com/square/leakcanary
[leakcanary-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[leakcanary-starsbadge]: https://img.shields.io/github/stars/square/leakcanary

[cicerone-maven]: https://bintray.com/terrakok/terramaven/cicerone/_latestVersion
[cicerone-mavenbadge]: https://api.bintray.com/packages/terrakok/terramaven/cicerone/images/download.svg
[cicerone-source]: https://github.com/terrakok/Cicerone
[cicerone-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[cicerone-starsbadge]: https://img.shields.io/github/stars/terrakok/Cicerone

[litho-maven]: https://bintray.com/facebook/maven/com.facebook.litho:litho-core/_latestVersion
[litho-mavenbadge]: https://api.bintray.com/packages/facebook/maven/com.facebook.litho:litho-core/images/download.svg
[litho-source]: https://github.com/facebook/litho
[litho-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[litho-starsbadge]: https://img.shields.io/github/stars/facebook/litho
