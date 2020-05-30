# Network

- [OkHttp](#okhttp---)
- [Retrofit](#retrofit---)
- [Sources](#sources)  


# OkHttp [![Maven][okhttp-mavenbadge]][okhttp-maven] [![Source][okhttp-sourcebadge]][okhttp-source] ![okhttp-starsbadge]

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

[Content](#network)
# Retrofit [![Maven][retrofit-mavenbadge]][retrofit-maven] [![Source][retrofit-sourcebadge]][retrofit-source] ![retrofit-starsbadge]

Type-safe HTTP client. Provides a powerful framework for authenticating and interacting with APIs and sending network requests with OkHttp. Automatically serialises the JSON response using a POJO. For conversion uses GSON or several other converters.

```gradle
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.6.0'
    
    // Or any other available converter.
    implementation 'com.squareup.retrofit2:converter-gson:2.6.0'    
    implementation 'com.squareup.retrofit2:converter-moshi:2.6.0'
    
    // To use with RxJava.
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.6.0'
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
    .addConverterFactory(MoshiConverterFactory.create())       // Alternative.
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



[Content](#network)
# Sources



[retrofit-maven]: https://search.maven.org/artifact/com.squareup.retrofit2/retrofit
[retrofit-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.squareup.retrofit2/retrofit/badge.svg
[retrofit-source]: https://github.com/square/retrofit
[retrofit-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[retrofit-starsbadge]: https://img.shields.io/github/stars/square/retrofit

[okhttp-maven]: https://search.maven.org/artifact/com.squareup.okhttp3/okhttp
[okhttp-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/com.squareup.okhttp3/okhttp/badge.svg
[okhttp-source]: https://github.com/square/okhttp
[okhttp-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[okhttp-starsbadge]: https://img.shields.io/github/stars/square/okhttp
