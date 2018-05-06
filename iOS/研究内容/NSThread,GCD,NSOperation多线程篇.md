### 多线程
#### 参考
- [iOS多线程——你要知道的NSThread都在这里](https://www.jianshu.com/p/973f0a5e0ec3)
- [iOS沉思录】NSThread、GCD、NSOperation多线程编程总结](https://blog.csdn.net/cordova/article/details/69060085)
- [iOS多线程NSThread/GCD/NSOperation区别和使用](https://www.cnblogs.com/zhaoyunboy/p/how-to-use-gcd-nsoperation.html)
- [多线程死锁](https://www.jianshu.com/p/201ccb40a3f8)

#### 简述
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCE9928680abc3cae9e9a7f0a832cb60fa6/2466)

#### 三者比较：
1. NSThread优点：NSThread 比其他两个轻量级。缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销。
2. Cocoa NSOperation优点:不需要关心线程管理， 数据同步的事情，可以把精力放在自己需要执行的操作上。Cocoa operation相关的类是NSOperation, NSOperationQueue.NSOperation是个抽象类,使用它必须用它的子类，可以实现它或者使用它定义好的两个子类: NSInvocationOperation和NSBlockOperation.创建NSOperation子类的对象，把对象添加到NSOperationQueue队列里执行。
3.  GCD(全优点)Grand Central dispatch(GCD) 利用自身CPU 内核发挥内核优势，自动管理线程的生命周期，无需管理线程。
4. pthread是一套通用的多线程API，适用于Linux\Windows\Unix,跨平台，可移植，使用C语言，生命周期需要程序员管理，IOS开发中使用很少。


#### NSThread  

优点：NSThread 比其他两个轻量级 缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销

```
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(run) object:nil]; // 初始化线程
thread.threadPriority = 1; // 设置线程的优先级(0.0 - 1.0，1.0最高级)
[thread start]; // 开启线程
[thread cancel] //删除线程
isExcuteting //是否正在执行
isFinished  //是否完成
//注意NSThread  子类化 需重写main 方法
```



```
//NSThread的类方法
+detachNewThreadSelector:toTarget: withObject   //隐式的开启线程无需start默认自动启动线程
[NSThread currentThread] //当前线程
[NSThread mainThread]  //主线程
[NSThread sleepForTimeInterval:1]	//线程睡眠
+sleepUntilDate   //睡眠线程执行某个日期

NSObject对象的实例方法
-performSelectorInBackground:withObject  //隐式创建线程的方法
-performSelectorOnMainThread:withObject:waitUntilDone:modes // 主线程操作
// 参数waitUntilDone：是否当前线程被阻塞，在子线程发中有效

```

#### NSOperation

```
优点:不需要关心线程管理， 数据同步的事情，可以把精力放在自己需要执行的操作上。 Cocoa operation相关的类是NSOperation, NSOperationQueue.  (NSInvocationOperation、NSBlockOperation) NSOperation是个抽象类,使用它必须用它的子类，可以实现它或者使用它定义好的两个子类: NSInvocationOperation和NSBlockOperation. 创建NSOperation子类的对象，把对象添加到NSOperationQueue队列里执行。 异步操作的过程需要更多的被交互和UI呈现出来，NSOperationQueue会是一个更好的选择
实现方式需NSOperation子类  NSInvocationOperation，NSBlockOperation，自定义子类继承NSOperation，实现内部相应的方法
```


```
1. NSInvocationOperation：
NSInvocationOperation *operation = [[[NSInvocationOperation alloc] initWithTarget:self selector:@selector(run:) object:@"mj"]；
[operation start]；
//注意：默认情况下是同步操作，调用了start方法后并不会开一条新线程去执行操作，而是在当前线程同步执行操作。只有将operation放到一个NSOperationQueue中，才会异步执行操作，
2. NSBlockOperation
NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^(){
         NSLog(@"执行了一个新的操作");
}];
[operation addExecutionBlock:^() {
  　　NSLog(@"又执行了1个新的操作，线程：%@", [NSThread currentThread]);
 }]; //添加多一个操作才是异步调用
 // 开始执行任务
[operation start];

注意上面操作数目只有一个时候是同步操作，只有开启多个操作才是异步操作
3. 自定义NSOperation
首先继承NSOperation,重写main 入口函数
- (void)main {
     // 新建一个自动释放池，如果是异步执行操作，那么将无法访问到主线程的自动释放池
@autoreleasepool { 
 //如果这个是在异步线程中执行操作，也就是说main方法在异步线程调用，
//那么将无法访问主线程的自动释放池，所以在这创建了一个属于当前线程的自动释放池
         // ....
     }
 }
cancel //取消
addDependency //依赖关系
setMaxConcurrentOperationCount//设置最大线程数
注意：当重写start 会执行start方法 不会执行main 方法 同时配合KVO 对属性状态控制，同时必须放在NSOperationQueue中才是异步操作否则直接start 是同步操作
需要管理以下状态
isFinished//是否完成
isExecuting//是否执行
isCancelled //是否取消
isReady //是否准备
isPause // 是否暂停



//例如手动管理isCancelled属性注意  线程锁 NSRecursiveLock
[self willChangeValueForKey:@"isCancelled"];
_isCancelled = YES;
[self didChangeValueForKey:@"isCancelled"];
 
/*注意：想拥有更多的控制权，或者想在一个操作中可以执行异步任务，那么就重写 start方法, 但是注意：这种情况下，你必须手动管理操作的状态，
只有当发送 isFinished的 KVO 消息时，才认为是 operation 结束*/
 
 
4.NSOperationQueue
- (void)addOperation:(NSOperation *)op; - (void)addOperationWithBlock:(void (^)(void))block; 
/*
NSOperationQueue的作⽤： 
NSOperation可以调⽤start⽅法来执⾏任务,但默认是同步执行的
除非将NSOperation添加到NSOperationQueue(操作队列)中,系统会自动异步执行NSOperation中的操作
添加操作到NSOperationQueue中，自动执行操作，自动开启线程 
*/

```


#### GCD


```
GCD更接近底层，而NSOperationQueue则更高级抽象，所以GCD在追求性能的底层操作来说，是速度最快的, 
从异步操作之间的事务性，顺序行，依赖关系。GCD需要自己写更多的代码来实现，而NSOperationQueue已经内建了这些支持
任务之间不太互相依赖，而需要更高的并发能力 GCD 最好

dispatch_queue_create("serial", DISPATCH_QUEUE_SERIAL) //创建串行
dispatch_queue_create("concurrent", DISPATCH_QUEUE_CONCURRENT)  //创建并行

dispatch_get_main_queue 主队列是串行队列 ; dispatch_get_global_queue 全局队列是并行队列
//异步全局队列(并行队列)  每次获取不相同的线程对象
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
});
//异步主队列(串行队列)
dispatch_async(dispatch_get_main_queue(), ^{
});
//执行一次
static dispatch_once_t once;
dispatch_once(&once, ^ {
});

dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        //  放一些极其耗时间的任务在此执行
        dispatch_async(dispatch_get_main_queue(), ^{
        /*
             耗时任务完成，拿到资源，更新UI
             更新UI只可以在主线程中更新
             */ 
        });
         
    });
dispatch_async() //往队列中添加任务，任务会排队执行
dispatch_after() //往队列中添加任务，任务排队且延迟执行
dispatch_apply() //往队列中添加任务，任务重复执行
dispatch_group_async() //往队列中添加任务，并添加分组标记
dispatch_group_notify() //往队列中添加任务，当某个分组的所有任务执行完之后，此任务才执行
dispatch_barrier_async() //往队列中添加任务，此任务执行的时候，其他任务停止(注意只能搭配自定义的并行队列，不能用dispatch_get_global_queue)

//dispatch_get_current_queue被废弃因为会造成死锁

//1创建队列
 dispatch_queue_t queue =dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT,0);//全局队列是并行队列
//2创建祖
dispatch_group_t group = dispatch_group_create();
//3notify之前最起码需要一个任务，否则notify不会等其他执行完才执行
dispatch_group_async(group, queue, ^{
NSLog(@"线程是：%@， 是否主线程：%d", 
[NSThread currentThread], [[NSThread currentThread] isMainThread]); 
});
dispatch_group_notify(group, queue, ^{
NSLog(@"我是最后一个任务，其他执行完我才执行");
});


创建群

GCD 定时器使用  
dispatch_queue_t queue = dispatch_get_main_queue();
// 新建定时器
dispatch_source_t timer  = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER,0, 0, queue);
//开始时间
dispatch_time_t start =dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
//间隔
uint64_t interval = (uint64_t)(1.0 *NSEC_PER_SEC);
//定时器开启
dispatch_source_set_timer(timer, start, interval,0);
dispatch_source_set_event_handler(timer, ^{
	//回调事件
NSLog(@"------------%@", [NSThreadcurrentThread]);
});
//启动定时器
dispatch_resume(timer);
```




#### 补充 多线程的理解
```
串行按顺序的执行每个任务，一次只能处理一个任务，只存一个堆 处理一个调用栈
并行就是同一时间执行多件任务的能力（多个人同时处理多个任务）
并发就是同一时间内应对多件任务的能力(一个人同时处理多个任务)
注意：并行是CPU的多核芯同时执行多个任务  并发是单核CPU交替执行两个任务

那么，如果他们相互串起来，会怎么样呢？
并行+异步：就是真正的并发，新开有有多个线程处理任务，任务并发执行（不按顺序执行）
串行+异步：新开一个线程，任务一个接一个执行，上一个任务处理完毕，下一个任务才可以被执行
并行+同步：不新开线程，任务一个接一个执行
串行+同步：不新开线程，任务一个接一个执行

串行：队列里面的任务一个接着一个执行，要等前一个任务结束，下一个任务才可以执行
异步：具有新开线程的能力
同步：不具有新开线程的能力，只能在当前线程执行任务
串行与并行针对的是队列，而同步与异步，针对的则是线程


注意：
1.无论串行队列和并行队列都是遵守FIFO(先进先出)原则 ，并发队列中的任务会按入队的顺序执行任务，但是
哪个任务先完成是不确定的。所以并发队列无需等待上个任务执行完在执行下一个，但是串行是一个一个执行。
2.主线程是特殊的串行队列 ， 全局队列是并行队列。
3.重点！同步和异步最大的区别在于，同步线程要阻塞当前线程（也就任务所在的队列），必须要等待同步线程中的任务执行完，
返回以后，才能继续执行下一任务；而异步线程则是不用等待。
4.所以如果是串行队列遇到同步任务就容易造成死锁，因为串行队列是一个一个执行，而且同步任务是会阻碍当前线程的
```

#### 多线程死锁
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCE7ddce76f6d295cdd961796355dd7a883/2497)
```
例子一
NSLog(@"1"); // 任务1 在主线程执行
dispatch_sync(dispatch_get_main_queue(), ^{
    NSLog(@"2"); // 任务2 造成死锁
});
NSLog(@"3"); // 任务3  在主线程执行
GCD中在主线程中用同步函数分派任务到串行队列中会产生死锁是什么原因？   
1.首先.dispatch_sync表示是一个同步线程；dispatch_get_main_queue表示运行在主线程中的主队列；
任务2是同步线程的任务。任务3需要等待任务2结束之后再执行.
2.然后首先执行任务1，这是肯定没问题的，只是接下来，程序遇到了同步线程，那么它会进入等待，等待任务2执行完，然后执行任务3。
但这是主队列，是一个特殊的串行队列,有任务来，当然会将任务加到队尾，然后遵循FIFO（先进先出）原则执行任务。那么，现在任务2就会被加到最后，
任务3排在了任务2前面，意味着任务2要在任务3执行完才能执行，所以他们进入了互相等待的局面。
3.注意但是自己创建的串行对列  执行同步操作任务 不会死锁 因为 不是在同一个串行队列中

```

![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCE573e77e0f41d105b14b648ed0a7f02f3/2499)


```
例子二
dispatch_queue_t queue = dispatch_queue_create("com.demo.serialQueue", DISPATCH_QUEUE_SERIAL);
NSLog(@"1"); // 任务1
dispatch_async(queue, ^{
    NSLog(@"2"); // 任务2
    dispatch_sync(queue, ^{  
        NSLog(@"3"); // 任务3
    });
    NSLog(@"4"); // 任务4
});
NSLog(@"5"); // 任务5  输出1 5 2（5 和2  顺序不一定）
分析：分析： 这个案例没有使用系统提供的串行或并行队列，而是自己通过dispatch_queue_create函数创建了一个DISPATCH_QUEUE_SERIAL的串行队列。
注意任务1和任务5 都是在主线程中执行（串行队列），默认执行都是在主线程中   1.执行任务1； 2.遇到异步线程，将【任务2、同步线程、任务4】加入串行队列中。因为是异步线程，所以在主线程中的任务5不必等待异步线程中的所有任务完成； 3.因为任务5不必等待，所以2和5的输出顺序不能确定； 4.任务2执行完以后，遇到同步线程，这时，将任务3加入串行队列； 5.又因为任务4比任务3早加入串行队列，所以，任务3要等待任务4完成以后，才能执行。但是任务3所在的同步线程会阻塞，所以任务4必须等任务3执行完以后再执行。这就又陷入了无限的等待中，造成死锁。

```

![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCEb3ebfc92dc579a61108e8c1aaaaa053c/2498)


```
案例 三
NSLog(@"1"); // 任务1
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    NSLog(@"2"); // 任务2
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"3"); // 任务3
    });
    NSLog(@"4"); // 任务4
});
NSLog(@"5"); // 任务5   输出1，2，5，3，4  //2 和 5 的顺序不一定
分析：异步加载数据,回调主线程更新UI
首先，将【任务1、异步线程、任务5】加入Main Queue中，异步线程中的任务是：【任务2、同步线程、任务4】。
所以，先执行任务1，然后将异步线程中的任务加入到Global Queue中，因为异步线程，所以任务5不用等待，结果就是2和5的输出顺序不一定。
然后再看异步线程中的任务执行顺序。任务2执行完以后，遇到同步线程。将同步线程中的任务又回调加入到Main Queue中，这时加入的任务3在任务5的后面。
当任务3执行完以后，没有了阻塞(因为是不同线程 且不是串行队列)，程序继续执行任务4。
从以上的分析来看，得到的几个结果：1最先执行；2和5顺序不一定；4一定在3后面。

```
