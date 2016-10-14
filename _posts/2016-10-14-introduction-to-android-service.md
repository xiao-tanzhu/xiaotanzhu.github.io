---
layout: post
title: 关于 Android Service 的介绍
tags:
- Android Service
- 安卓
categories: 安卓
description: 关于 Android Service 的介绍
---

> 转载自：http://www.jianshu.com/p/63792fc84d94#

Service 是 Android 的四大组件之一，它主要的作用是后台执行操作，Activity 属于带有 UI 界面跟用户进行交互，而 Service 则没有 UI 界面，所有的操作都是基于后台运行完成。并且 Service 跟 Activity 一样也是可以由其它的应用程序调用启动的，而且就算用户切换了应用程序，Service 依旧保持运行。一个组件如果与 Service 进行了绑定( bind ),就可以跟 Service 进行数据的交互，并且也可以跟不同的进程之间进行交互 (IPC)。通常会使用到 Service 的情况有进行网络请求，音乐的操控，文件的 I/O 操作等。

## 启动

Service 通常是通过以下两种方式进行启动

### startService

当组件(例如 activity)通过调用`startService()`来启动 Service 的时候。一旦启动后，Service 就会独立的在后台运行，即使调用的组件已经销毁了，Service 还是可以继续在后台运行。一般情况下，**只需要进行一次单独的操作，不需要将操作后的结果返回给调用者的时候，会使用该方式启动 Service**。例如，进行上传或者下载操作的时候，当操作完成后，Service 应该自行调用 `stopService()` 或 `stopSelf()` 来结束运行。

### bindService

当组件(例如 activity)通过调用 `bindService()` 来启动 Service 的时候。这种方式提供了 client - service 的接口，可以让调用组件跟 Service 进行发送请求及返回结果的操作，设置可以进行进程间的通信 (IPC)。只要有一个组件对该 Service 进行了绑定，那该 Service 就不会销毁。并且多个组件可以同时对一个 Service 进行绑定，只有在所有进行了绑定的组件都解绑的时候，Service 才会销毁。

尽管两种方式是分开讨论的，但是并不是互斥的关系，使用 startService 启动了 Service 后，也是可以进行绑定的。

> **注意：**虽然 Service 是在后台运行，但是其实还是在主线程里进行所有的操作的。Service 在启动时除非单独进行了定义，否则不会运行在单独的线程或者进程里，而都是在主线程里。所以这表示任何能堵塞主线程的操作（例如音乐的播放或者网络请求）都应该单独开辟新的线程来进行操作，否则很容易出现 ANR 。

----

> 如果某个组件是通过调用 startService() 的方式来启动了 Service，那这个 Service 就会一直在后台运行直到 Service 内部调用 stopSelf() 或某个组件调用 stopService() 来结束该 Service。

> 如果某个组件是通过调用 bindService() 的方式来启动了 Service，那这个 Service 就会一直在后台运行直到该组件与其解绑。Service 在没有任何组件绑定的时候，系统会将其销毁


## 方法

在创建一个 Service 时，必须要去继承 Service，并且要重写父类一些主要的方法来实现功能。以下是主要方法的介绍

### onStartCommand()

系统会调用这个函数当某个组件(例如 activity，fragment)通过调用 `startService()` 启动 Service 时。在该方法被调用后，Service 就会被启动并独立的在后台运行。如果重写了该方法，开发者需要在 Service 执行完操作后自行的调用 `stopSelf()` 或 `stopService()`来结束 Service。如果只是会通过绑定的方式 (bind) 的方式来启动 Service 则不需要重写该方法。

### onBind()

当某个组件(例如 activity, fragment)通过调用 `bindService()`绑定的方式来启动 Service 的时候，系统会调用这个函数。在实现这个函数的时候，必须要返回一个 IBinder 的继承类，来与 Service 进行通信。这个函数是默认必须要重写的，但是如果不想通过绑定的方式来启动 Service，则可以直接返回 null

### onCreate()

系统会调用此方法在第一次启动 Service 的时候，用于初始化一些一次性的变量。**如果 Service 已经启动了，则此方法就不会再别调用。**

### onDestroy()

系统在 Service 即将被销毁的时候会调用此方法。Service 中如有用到 thread、listeners、receivers 等的时候，应该将这些的清理方法写在此方法内。

如果某个组件是通过调用 startService() 的方式来启动了 Service，那这个 Service 就会一直在后台运行直到 Service 内部调用 stopSelf() 或某个组件调用 stopService() 来结束该 Service。

如果某个组件是通过调用 bindService() 的方式来启动了 Service，那这个 Service 就会一直在后台运行直到该组件与其解绑。Service 在没有任何组件绑定的时候，系统会将其销毁

## 使用Service

下面的环节，将介绍如果通过上面讲述的两种方式来创建 Service

### 在 Manifest 里声明 Service

类似于 Activity，所有的 Service 都要在 Manifest 里面进行声明，如下:

```xml
<manifest ... >
  ...
  <application ... >
      <service android:name=".ExampleService" />
      ...
  </application>
</manifest>
```

查看 [&lt;service&gt;](https://developer.android.com/guide/topics/manifest/service-element.html) 标签的官方文档来获取更多信息

通过在 &lt;service&gt; 标签里将 `android:exported` 设置为 false。可以防止其他的程序来启动你的 Service。

### 通过 started 方式来启动 Service

组件(例如 activity, fragment) 通过调用 `startService()` 方法，系统随之调用 `onStartCommand()` 方法来实现 started 方式启动 Service。

当 Service 以该形式启动后，Service 的整个生命周期是完全独立的，即便启动 Service 的组件已经被销毁了，Service 还是可以在后台无限的运行的。但开发者应该在 Service 中的操作执行完成后，调用 `stopSelf()` 或其它组件调用 `stopService()` 的方式来结束该 Service。

程序组件(例如 activity) 可以通过传递一个 Intent 给 `startService()`,来实现组件与 Service 之前的数据传递。Service 是通过系统调用的 `onStartCommand()` 方法接受传递的Intent ，完成整个数据传递过程。

> **注意:** Service 本身默认是运行在主线程里的，所以如果在 Service 要进行一些会堵塞线程的操作，一定要将这些操作放在一个新的线程里。

Android 的框架提供了 [IntentService](https://developer.android.com/reference/android/app/IntentService.html) 来满足后台运行异步线程的需求。

#### IntentService

IntentService 是 Service 的子类，并且所有的请求操作都是在异步线程里。**如果不需要 Service 来同时处理多个请求的话，IntentService 将会是最佳的选择。**只需要继承并重写 IntentService 中的 `onHandleIntent()` 方法，就可以对接受到的 Intent 做后台的异步线程操作了。

IntentService 提供了如下的几个功能:

- 会创建一个异步线程来处理所有从程序主线程发送到 `onStartCommand()` 的 intents 。
- 创建了一个队列池来保证每次只有一个 Intent 传递到 `onHandleIntent()` 方法中，避免了多线程问题。
- 提供默认 `onBind()` 方法的实现，返回 null
- 提供默认 `onStartCommand()` 方法的实现，将收到的 intent 放到队列池中，然后再交由 `onHandleIntent()` 做处理

所有开发者只需要实现 `onHandleIntent()` 关注于 Service 要进行的操作即可。关于更多的 IntentService 实用技巧，请查看此文章：https://github.com/codepath/android_guides/wiki/Starting-Background-Services。

如果开发者需要重写其他的一些方法，例如 `onCreate()`, `onStartCommand()`, 和 `onDestroy()`，请保证调用父类的实现，这样可以保证 IntentService 能够正确的来处理线程的生命周期。例如:

```java
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    Toast.makeText(this, "service starting", Toast.LENGTH_SHORT).show();
    return super.onStartCommand(intent,flags,startId);
}
```

#### 系统回收资源问题

当系统内存不足的时候，系统会强制回收一些 Activity 和 Service 来获取更多的资源给那些用户正在交互的程序或页面。这就要求当资源充足的时候 Service 能够自动重启。这个功能是通过 `onStartCommand()` 的返回值来实现的

##### START_NOT_STICKY

当系统因回收资源而销毁了 Service，当资源再次充足时不自动启动 Service。除非还有未处理的 Intent 准备发送。当你的程序可以很容易的重新开启未完成的操作时，这是最安全的避免 Service 在不必要的情况下启动的选项。

##### START_STICKY

当系统因回收资源而销毁了 Service，当资源再次充足时自动启动 Service，并且再次调用 `onStartCommand()` 方法，但是不会传递最后一次的 Intent，相反系统在回调`onStartCommand()` 的时候会传一个空 Intent 。除非还有未处理的 Intent 准备发送。

##### START_REDELIVER_INTENT

当系统因回收资源而销毁了 Service，当资源再次充足时自动启动 Service，并且再次调用 `onStartCommand()` 方法，并会把最后一次 Intent 再次传递给`onStartCommand()`，相应的在队列里的 Intent 也会按次序依次传递。此模式适用于下载等服务。

#### Start Service

该方式允许多个组件同时对相同的 Service 进行 startService 操作，但是如果只要有其中有一个组件调用了 `stopSelf()` 或 `stopService()`, 该 Service 就会被销毁。

```java
Intent intent = new Intent(this, HelloService.class);
startService(intent);
```

#### Stop Service

当有多个组件进行了 startService 操作时，不应该直接的去调用 `stopSelf()` 或 `stopService()` 来结束 Service， 因为这会对其他已经发起请求的操作产生影响，故在 `onStartCommand()` 方法中会接受一个 startId, 然后在结束 Service 时，调用 `stopService(int)` 方法来只是结束一个特定的请求，从而达到保护其他请求不受影响的目的。

### 通过 Bind 方式启动 Service

当应用程序中的 activity 或其它组件需要与服务进行交互，或者应用程序的某些功能需要暴露给其它应用程序时，你应该创建一个 Bind 服务，并通过进程间通信（IPC）来完成。

Service 只在为绑定的应用程序组件工作时才会存活，因此，只要没有组件绑定到服务，系统就会自动销毁服务

#### 获取 IBinder 实例

##### 扩展 Binder 类

如果服务是你的应用程序所私有的，并且与客户端运行于同一个进程中（通常都是如此），你应该通过扩展 Binder 类来创建你的接口，并从 `onBind()` 返回一个它的实例。客户端接收该 Binder 对象并用它来直接访问 Binder 甚至 Service中可用的公共方法。

**如果你的服务只是为你自己的应用程序执行一些后台工作，那这就是首选的技术方案。**不用这种方式来创建接口的理由只有一个，就是服务要被其它应用程序使用或者要跨多个进程使用。

##### 使用 Messenger

如果你需要接口跨越多个进程进行工作，可以通过 Messenger 来为服务创建接口。在这种方式下，服务定义一个响应各类消息对象 Message 的 Handler。此 Handler 是 Messenger 与客户端共享同一个 IBinder 的基础，它使得客户端可以用消息对象 Message向服务发送指令。此外，客户端还可以定义自己的 Message ，以便服务能够往回发送消息。

这是执行进程间通信（IPC）最为简便的方式，因为 Messenger 会把所有的请求放入一个独立进程中的队列，这样你就不一定非要把服务设计为线程安全的模式了。

##### 使用 AIDL

绝大多数应用程序都**不应该**用 AIDL 来创建 bind 服务，因为这可能需要多线程处理能力并且会让代码变得更为复杂。 因此，AIDL 对绝大多数应用程序都不适用，并且本文也不会讨论如何在服务中使用它的内容。如果你确信需要直接使用 AIDL，那请参阅 [AIDL 文档](https://developer.android.com/guide/components/aidl.html)。

#### 扩展 Binder 类

如果你的服务只用于本地应用程序并且不需要跨进程工作，那你只要实现自己的 Binder 类即可，这样你的客户端就能直接访问服务中的公共方法了。

> 注意：仅当客户端和服务位于同一个应用程序和进程中，这也是最常见的情况，这种方式才会有用。比如，一个音乐应用需要把一个 activity 绑定到它自己的后台音乐播放服务上，采用这种方式就会很不错。

以下是设置步骤:

1. 在你的服务中，创建一个 Binder 的实例，其中实现以下三者之一：
	- 包含了可供客户端调用的公共方法
	- 返回当前 Service 实例，其中包含了可供客户端调用的公共方法
	- 或者，返回内含 Service 类的其它类的一个实例，Service 中包含了可供客户端调用的公共方法
2. 从回调方法 `onBind()` 中返回 Binder 的该实例
3. 在客户端中，在回调方法 `onServiceConnected()` 中接收 Binder 并用所提供的方法对绑定的服务进行调用

> **注意：**服务和客户端之所以必须位于同一个应用程序中，是为了让客户端能够正确转换（cast）返回的对象并调用对象的 API。 服务和客户端也必须位于同一个进程中，因为这种方式不能执行任何跨进程的序列化（marshalling）操作。

具体的实用案例请查看在[ApiDemos](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/) 中的 [LocalService.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/LocalService.java) 类和 [LocalServiceActivities.java](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/LocalServiceActivities.java)。

## Service 的生命周期

![](/upload/images/13.png)
