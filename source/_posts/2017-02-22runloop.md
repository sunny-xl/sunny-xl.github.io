---
title: RunLoop 简介
date: 2017-02-22 15:36:41
categories: iOS
tags: [iOS]
comments: false
---

## RunLoop基本概念

一般来讲，一个线程一次只能执行一个任务，执行完成后线程就会退出。如果我们需要一个机制，让线程能随时处理事件但并不退出，通常的代码逻辑是这样的：

```
function loop() {
    initialize();
    do {
        var message = get_next_message();
        process_message(message);
    } while (message != quit);
}
```

这种模型通常被称作 [Event Loop](https://en.wikipedia.org/wiki/Event_loop)。 Event Loop 在很多系统和框架里都有实现，比如 Node.js 的事件处理，比如 Windows 程序的消息循环，再比如 OSX/iOS 里的 RunLoop。实现这种模型的关键点在于：**如何管理事件/消息，如何让线程在没有处理消息时休眠以避免资源占用、在有消息到来时立刻被唤醒**。

所以，RunLoop 实际上就是一个对象，这个对象管理了其需要处理的事件和消息，并提供了一个入口函数来执行上面 Event Loop 的逻辑。线程执行了这个函数后，就会一直处于这个函数内部 "接受消息->等待->处理" 的循环中，直到这个循环结束（比如传入 quit 的消息），函数返回。

<!--more-->

OSX/iOS 系统中，提供了两个这样的对象：NSRunLoop 和 CFRunLoopRef。
CFRunLoopRef 是在 CoreFoundation 框架内的，它提供了纯 C 函数的 API，所有这些 API 都是线程安全的。
NSRunLoop 是基于 CFRunLoopRef 的封装，提供了面向对象的 API，但是这些 API 不是线程安全的。

一张苹果官方文档的图，大体说明了 Runloop 的工作模式：

![RunLoop](/blogImages/runloop.png)

## RunLoop基本作用

- 保持程序的持续运行,保持线程的持续运行，并接受用户输入
- 处理App中的各种事件(比如触摸事件,定时器事件,Selector事件)
- 调用解耦（Message Queue）
- 节省CPU资源,提高程序性能:该做事时做事,该休息时休息

## RunLoop与线程

- 每个线程（包括主线程）都有一个对应的 RunLoop 对象
- 我们并不能自己创建 RunLoop 对象，但是可以获取到系统提供的 RunLoop 对象。
- 主线程的RunLoop默认是启动的，用于接收各种输入sources;其他线程的RunLoop默认是没有启动的，如果你需要更多的线程交互则可以手动配置和启动
- RunLoop在第一次获取时由系统自动创建,在线程结束时销毁

## RunLoop使用场景

- 使用ports 或 input sources 和其他线程通信
- 在线程中使用timer
- 在`Cocoa`应用中使用`performSelector...`方法   // 应该是performSelector...这种方法会启动一个线程并启动run loop吧
- 让线程执行一个周期性的任务   // 如果不启动run loop， 线程跑完就可能被系统释放了

来源[RunLoop官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)

## RunLoop运行模式

- 一个 RunLoop包含若干个Mode，每个Mode又包含若干个Source/Timer/Observer
- 每次RunLoop启动时，只能指定其中一个Mode，这个Mode被称作 CurrentMode
- 如果需要切换Mode，只能退出Loop，再重新指定一个Mode进入系统默认模式

系统默认注册了5个Mode:

- NSDefaultRunLoopMode：App的默认Mode，通常主线程是在这个Mode下运行
- UITrackingRunLoopMode：界面跟踪Mode，用于ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
- UIInitializationRunLoopMode:在刚启动App时第进入的第一个 Mode，启动完成后就不再使用
- GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
- NSRunLoopCommonModes:这是一个占位用的Mode，不是一种真正的Mode。可以看成模式组,默认情况下包括了NSDefaultRunLoopMode,UITrackingRunLoopMode)两种模式.

## RunLoop应用

- NSTimer
  * 需指定模式，若GCD则不用

- ImageView显示
 * 在特定模式下执行某些操作，图片设置与拖拽分别在不同模式

- PerformSelector

- 常驻线程
 * 某些操作，需要重复开辟子线程，重复开辟内存过于消耗性能，可以设定子线程常驻

- 自动释放池
 * 创建和释放
 1.第一次创建, 是在runloop进入的时候创建，对应的状态 = kCFRunLoopEntry
 2.最后一次释放, 是在runloop退出的时候 对应的状态 = kCFRunLoopExit
 3.其它创建和释放

 每次睡觉的时候都会释放前自动释放池,然后再创建一个新的

- 可以添加Observer监听RunLoop的状态
 * 比如监听点击事件的处理（在所有点击事件之前做一些事情）

### 注意点

- 子线程RunLoop常驻

```
// 1.子线程的NSRunLoop需要手动创建
  // 2.子线程的NSRunLoop需要手动开启
  // 3.如果子线程的NSRunLoop没有设置source or timer, 那么子线程的NSRunLoop会立刻关闭
  // 无含义，设置子线程为常住线程，让子线程不关闭
  // [[NSRunLoop currentRunLoop] addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];

  NSTimer *timer = [NSTimer timerWithTimeInterval:5.0 target:self selector:@selector(test) userInfo:nil repeats:YES];
  // 会添加到当前子线程
  [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
  [[NSRunLoop currentRunLoop] run];
```

注意：
1. NSRunLoop只会检查有没有source和timer, 没有就关闭, 不会检查observer
2. 主线程没有到期时间，子线程有




