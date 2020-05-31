# IMAGES

- [Glide](#glide---)
- [Coil](#coil---)
- [Sources](#sources)  



# Glide [![Maven][glide-mavenbadge]][glide-maven] [![Source][glide-sourcebadge]][glide-source] ![glide-starsbadge]

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

[Content](#images)
# Coil [![Maven][coil-mavenbadge]][coil-maven] [![Source][coil-sourcebadge]][coil-source] ![coil-starsbadge]

An image loading library for Android backed by Kotlin Coroutines.

```gradle
dependencies {
    implementation 'io.coil-kt:coil:0.11.0'
}
```

- Fast: Coil performs a number of optimizations including memory and disk caching, downsampling the image in memory, re-using Bitmaps, automatically pausing/cancelling requests, and more.
- Lightweight: Coil adds ~2000 methods to your APK (for apps that already use OkHttp and Coroutines), which is comparable to Picasso and significantly less than Glide and Fresco.
- Easy to use: Coil's API leverages Kotlin's language features for simplicity and minimal boilerplate.
- Modern: Coil is Kotlin-first and uses modern libraries including Coroutines, OkHttp, Okio, and AndroidX Lifecycles.

### Coil. Usage

To load an image into an ImageView, use the load extension function:

```kotlin
imageView.load("https://www.example.com/image.jpg")
imageView.load(R.drawable.image)
imageView.load(File("/path/to/image.jpg"))
```

Requests can be configured with an optional trailing lambda:

```kotlin
imageView.load("https://www.example.com/image.jpg") {
    crossfade(true)
    placeholder(R.drawable.image)
    transformations(CircleCropTransformation())
}
```

imageView.load uses the singleton ImageLoader to execute a LoadRequest. Execution suspends the current coroutine, non-blocking and thread safe.

### Coil. Requests

To load an image into a custom target, execute a LoadRequest:

```kotlin
val imageLoader = Coil.imageLoader(context)
val request = LoadRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    .target { drawable ->
        // Handle the result.
    }
    .build()
imageLoader.execute(request)
```

To get an image imperatively, execute a GetRequest:

```kotlin
val imageLoader = Coil.imageLoader(context)
val request = GetRequest.Builder(context)
    .data("https://www.example.com/image.jpg")
    .build()
val drawable = imageLoader.execute(request).drawable
```

### Coil. Image Sampling

Suppose you have an image that is 500x500 on disk, but you only need to load it into memory at 100x100 to be displayed in a view. Coil will load the image into memory, but what happens now if you need the image at 500x500? There's still more "quality" to read from disk, but the image is already loaded into memory at 100x100. Ideally, we would use the 100x100 image as a placeholder while we read the image from disk at 500x500.

This is exactly what Coil does and Coil handles this process automatically for all BitmapDrawables.

[Content](#images)
# Sources

**Coil**

https://coil-kt.github.io/coil/



[glide-maven]: https://search.maven.org/artifact/com.github.bumptech.glide/glide
[glide-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.github.bumptech.glide/glide/badge.svg
[glide-source]: https://github.com/bumptech/glide
[glide-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[glide-starsbadge]: https://img.shields.io/github/stars/bumptech/glide

[coil-maven]: https://search.maven.org/artifact/io.coil-kt/coil
[coil-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/io.coil-kt/coil/badge.svg
[coil-source]: https://github.com/coil-kt/coil
[coil-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[coil-starsbadge]: https://img.shields.io/github/stars/coil-kt/coil
