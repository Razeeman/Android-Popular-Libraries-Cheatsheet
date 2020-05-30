# ASYNC

- [EventBus](#eventbus---)
- [RxJava](#rxjava----rxandroid---)
- [Sources](#sources)
  


# EventBus [![Maven][eventbus-mavenbadge]][eventbus-maven] [![Source][eventbus-sourcebadge]][eventbus-source] ![eventbus-starsbadge]

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

[Content](#async)
# RxJava [![Maven][rxjava-mavenbadge]][rxjava-maven] [![Source][rxjava-sourcebadge]][rxjava-source] ![rxjava-starsbadge] RxAndroid [![Maven][rxandroid-mavenbadge]][rxandroid-maven] [![Source][rxandroid-sourcebadge]][rxandroid-source] ![rxandroid-starsbadge]

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



[Content](#async)
# Sources



[rxjava-maven]: https://search.maven.org/artifact/io.reactivex.rxjava2/rxjava
[rxjava-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxjava/badge.svg
[rxjava-source]: https://github.com/ReactiveX/RxJava
[rxjava-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[rxjava-starsbadge]: https://img.shields.io/github/stars/ReactiveX/RxJava

[rxandroid-maven]: https://search.maven.org/artifact/io.reactivex.rxjava2/rxandroid
[rxandroid-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/io.reactivex.rxjava2/rxandroid/badge.svg
[rxandroid-source]: https://github.com/ReactiveX/RxAndroid
[rxandroid-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[rxandroid-starsbadge]: https://img.shields.io/github/stars/ReactiveX/RxAndroid

[eventbus-maven]: https://search.maven.org/artifact/org.greenrobot/eventbus
[eventbus-mavenbadge]: https://maven-badges.herokuapp.com/maven-central/org.greenrobot/eventbus/badge.svg
[eventbus-source]: https://github.com/greenrobot/EventBus
[eventbus-sourcebadge]: https://img.shields.io/badge/source-github-orange.svg
[eventbus-starsbadge]: https://img.shields.io/github/stars/greenrobot/EventBus
