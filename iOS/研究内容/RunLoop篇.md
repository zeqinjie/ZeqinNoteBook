###  RunLoops
#### 参考
- [apple文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW5)
- [RunLoop入门](https://www.jianshu.com/p/2d3c8e084205)
- [深入理解RunLoop](http://www.cocoachina.com/ios/20150601/11970.html)

#### 简述
##### 1. 什么是RunLoops
- RunLoops 是 thread 线程的底层基础.它的本质是和它的意思一样运行着的循环,更加确切的说是线程中的循环.它用来接受循环事件和安排线程的工作,并且在线程中没有工作时,让线程进入睡眠状态
- 每条线程都有唯一的与之对应的RunLoop对象。
- 主线程的RunLoop已经默认系统帮我们创建好了，而子线程的需要手动创建。
（也就是说子线程的RunLoop默认是关闭的，因为有时候开了个线程但却没有必要开一个RunLoop，不然反而浪费了资源。 ）

```
** 注意：主线程默认开启runloop,子线程默认没有开启需手动创建
```


##### 2. RunLoop基本作用：

- 保持程序持续运行（程序默认开启主线程，主线程一开起来就会跑一个主线程对应的RunLoop,RunLoop保证主线程不会被销毁，也就保证了程序的持续运行）
- 处理App中的各种事件（比如：触摸事件，定时器事件，Selector事件等）
- 节省CPU资源，提高程序性能（当程序没做任何操作，runloop告诉程序现在没有事情做，可以休息。这时CPU释放资源）

##### 3.在 CoreFoundation 里面关于 RunLoop 有5个类:
- [x]     CFRunLoopRef
- [x]     CFRunLoopModeRef
- [x]     CFRunLoopSourceRef
- [x]     CFRunLoopTimerRef
- [x]     CFRunLoopObserverRef

![image](http://cc.cocimg.com/api/uploads/20150528/1432798883604537.png)

##### 4.由图中可以得出以下几点
-  CFRunLoopModeRef代表的是RunLoop的运行模式。
- 一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。
- 每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。
- 如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。


```

一.CFRunLoopModeRef 系统默认注册了5个mode
    kCFRunLoopDefaultMode //App的默认Mode，通常主线程是在这个Mode下运行
    UITrackingRunLoopMode //界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
    UIInitializationRunLoopMode // 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
    GSEventReceiveRunLoopMode // 接受系统事件的内部 Mode，通常用不到
    kCFRunLoopCommonModes //这是一个占位用的Mode，不是一种真正的Mode
    
二.Source/Timer/Observer
1.CFRunLoopSourceRef是事件产生的地方。Source有两个版本：Source0 和 Source1。
Source0 只包含了一个回调（函数指针），它并不能主动触发事件。使用时，你需要先调用 CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop，让其处理这个事件。
Source1 包含了一个 mach_port 和一个回调（函数指针），被用于通过内核和其他线程相互发送消息。这种 Source 能主动唤醒 RunLoop 的线程，其原理在下面会讲到。

2.CFRunLoopTimerRef 是基于时间的触发器，它和 NSTimer 是toll-free bridged 的，可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

3.CFRunLoopObserverRef 是观察者，每个 Observer 都包含了一个回调（函数指针），当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。可以观测的时间点有以下几个：
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即将进入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即将处理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即将处理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即将进入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 刚从休眠中唤醒
    kCFRunLoopExit          = (1UL << 7), // 即将退出Loop
};
```

#### RunLoop的事件队列
![image](https://upload-images.jianshu.io/upload_images/2312304-8e64b79a92d53567.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
```
每次运行run loop，你线程的run loop对会自动处理之前未处理的消息，并通知相关的观察者。具体的顺序如下
1. 通知观察者run loop已经启动
2. 通知观察者任何即将要开始的定时器
3. 通知观察者任何即将启动的非基于端口的源
4. 启动任何准备好的非基于端口的源
5. 如果基于端口的源准备好并处于等待状态，立即启动；并进入步骤9。
6. 通知观察者线程进入休眠
7. 将线程置于休眠直到任一下面的事件发生：
    * 某一事件到达基于端口的源
    * 定时器启动
    * Run loop设置的时间已经超时
    * run loop被显式唤醒
8. 通知观察者线程将被唤醒。
9. 处理未处理的事件
    * 如果用户定义的定时器启动，处理定时器事件并重启run loop。进入步骤2
    * 如果输入源启动，传递相应的消息
    * 如果run loop被显式唤醒而且时间还没超时，重启run loop。进入步骤2
10. 通知观察者run loop结束。
```


#### 实践
##### mian 函数

``` objc
//如上主程序运行启动默认主线程和对应的runloop 
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}

//调整1
int main(int argc, char * argv[]) {
    @autoreleasepool {
       int res = UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
        NSLog(@"-----");//不会输出
        return res;
    }
}

```

#####  timerWithTime方法
``` OBJC
- (IBAction)ButtonDidClick:(id)sender {
    //默认加入到kCFRunLoopDefaultMode 模式的runloop中
    [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timerTest) userInfo:nil repeats:YES];
    
    //下面需加入到runloop中，否则定时器不执行
    NSTimer *timer = [NSTimer timerWithTimeInterval:1.0 target:self selector:@selector(timerTest) userInfo:nil repeats:YES];
}


- (void)timerTest
{
    NSLog(@"timerTest----");
}

```

##### 有scrollView的情况下使用Timer

```
对于NSTimer 加入到NSDefaultRunLoopMode 下滚动scrollview时候timer的是事件是不被触发。原因是scrollview的模式UITrackingRunLoopMode
所以为了同时响应：需将timer加入到kNSRunLoopCommonModes
```

##### AFNetworking



```
AFURLConnectionOperation 这个类是基于 NSURLConnection 构建的，其希望能在后台线程接收 Delegate 回调。为此 AFNetworking 单独创建了一个线程，并在这个线程中启动了一个 RunLoop：
```

``` objc
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
  
+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}

```
```
RunLoop 启动前内部必须要有至少一个 Timer/Observer/Source，所以 AFNetworking 在 [runLoop run] 之前先创建了一个新的 NSMachPort 添加进去了。通常情况下，调用者需要持有这个 NSMachPort (mach_port) 并在外部线程通过这个 port 发送消息到 loop 内；但此处添加 port 只是为了让 RunLoop 不至于退出，并没有用于实际的发送消息。
```
``` OBJC
- (void)start {
    [self.lock lock];
    if ([self isCancelled]) {
        [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    } else if ([self isReady]) {
        self.state = AFOperationExecutingState;
        [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    }
    [self.lock unlock];
}
```

```
当需要这个后台线程执行任务时，AFNetworking 通过调用 [NSObject performSelector:onThread:..] 将这个任务扔到了后台线程的 RunLoop 中。
```

##### AsyncDisplayKit


```
AsyncDisplayKit 是 Facebook 推出的用于保持界面流畅性的框架，其原理大致如下：

UI 线程中一旦出现繁重的任务就会导致界面卡顿，这类任务通常分为3类：排版，绘制，UI对象操作。

排版通常包括计算视图大小、计算文本高度、重新计算子式图的排版等操作。

绘制一般有文本绘制 (例如 CoreText)、图片绘制 (例如预先解压)、元素绘制 (Quartz)等操作。

UI对象操作通常包括 UIView/CALayer 等 UI 对象的创建、设置属性和销毁。

其中前两类操作可以通过各种方法扔到后台线程执行，而最后一类操作只能在主线程完成，并且有时后面的操作需要依赖前面操作的结果 （例如TextView创建时可能需要提前计算出文本的大小）。ASDK 所做的，就是尽量将能放入后台的任务放入后台，不能的则尽量推迟 (例如视图的创建、属性的调整)。

为此，ASDK 创建了一个名为 ASDisplayNode 的对象，并在内部封装了 UIView/CALayer，它具有和 UIView/CALayer 相似的属性，例如 frame、backgroundColor等。所有这些属性都可以在后台线程更改，开发者可以只通过 Node 来操作其内部的 UIView/CALayer，这样就可以将排版和绘制放入了后台线程。但是无论怎么操作，这些属性总需要在某个时刻同步到主线程的 UIView/CALayer 去。

ASDK 仿照 QuartzCore/UIKit 框架的模式，实现了一套类似的界面更新的机制：即在主线程的 RunLoop 中添加一个 Observer，监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit 事件，在收到回调时，遍历所有之前放入队列的待处理的任务，然后一一执行。
```
