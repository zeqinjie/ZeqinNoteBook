# iOS研发规范
## 参考
[一份走心的iOS开发规范](http://www.cocoachina.com/ios/20180206/22157.html)

---

### 前言：
> - 清晰明了，易读，易用，易维护
> - 一般性原则：可读性高(简洁且清晰)和防止命名冲突(通过加前缀后缀来保证)。
> - 一致性:尽可能与Cocoa编程接⼝命名保持一致.
> - 驼峰原则:大驼峰原则：每个单词的首字母大写(UserNameTextField) 小驼峰原则：第一个单词首字母小写，其余都大写(userNameTextField).编码规范

### 命名规范

#### 类名/分类

> - 类的命名都遵循大驼峰命名。一般是：前缀 + 功能 + 类型。例如：TW + Login + ViewController

#### 分类命名：类名+标识+扩展

```
UIView+ZZQMasonry,NSObject+MJProperty
```

#### Protocol
> - Protocol:后缀Delegate或者是Protocol如

```
UITableViewDelegate 、NSObjectProtocol
```

> - 用optional修饰可以不实现的方法，用required修饰必须实现的方法

####  变量
> - 小驼峰式命名：第一个单词以小写字母开始，后面的单词的首字母全部大写
> - 建议：property变量在不需要改变一个属性时，添加@readonly，需要时添加@readwrite

#### 宏和常量命名

> - 宏：小写k+大驼峰 即为：#define kAnimationDuration = 0.3
> - 全局常量：工程前缀全大写，下划线隔开 即为：extern const NSString MW_USER_AGE_KEY

> - 对于宏定义的常量
>  1. #define 预处理定义的常量全部大写，单词间用 _ 分隔
>  2. 宏定义中如果包含表达式或变量，表达式或变量必须用小括号括起来。
> - 对于类型常量
> 1. 对于局限于某编译单元(实现文件)的常量，以字符k开头，例如kAnimationDuration，且需要以static const修饰
> 2. 对于定义于类头文件的常量，外部可见，则以定义该常量所在类的类名开头，例如EOCViewClassAnimationDuration, 仿照苹果风格，在头文件中进行extern声明，在实现文件中定义其值

```
//宏定义的常量
#define ANIMATION_DURATION    0.3
#define MY_MIN(A, B)  ((A)>(B)?(B):(A))

//局部类型常量
static const NSTimeInterval kAnimationDuration = 0.3;

//外部可见类型常量
//EOCViewClass.h
extern const NSTimeInterval EOCViewClassAnimationDuration;
extern NSString *const EOCViewClassStringConstant;  //字符串类型

//EOCViewClass.m
const NSTimeInterval EOCViewClassAnimationDuration = 0.3;
NSString *const EOCViewClassStringConstant = @"EOCStringConstant";
```


#### 方法

> - 小驼峰式命名
> - 方法名使用动词短语，能具体表达出该方法的功能
```
- (void)viewWillAppear:(BOOL)animated
- (void)setupPostValue:(int)value
- (void)adjustFontWithMaxSize:(CGSize)maxSize
```
> - 当参数过长时，每个参数占用一行，以冒号对齐。如：

```
- (void)saveUserInfo:(NSMutableDictionary *)dict
           userName:(NSString *)name
           passWord:(NSString *)pwd{
    ...
}
```

#### 分类/类别 方法

> - 避免category中的方法覆盖系统方法。可以使用前缀来区分系统方法和category方法。但前缀不要仅仅使用下划线”_“

```
- (NSDictionary *)zq_dicForKey:(NSString *)key{
    
}
```

#### Delegate方法命名规范

> - 用optional修饰可以不实现的方法，用required修饰必须实现的方法
> - 当你的委托的方法过多, 可以拆分数据部分和其他逻辑部分, 数据部分用dataSource做后缀. 如<UITableViewDataSource>
> - 使用did和will通知Delegate已经发生的变化或将要发生的变化。
> - 类的实例必须为回调方法的参数之一
>   1. 回调方法的参数只有类自己的情况，方法名要符合实际含义
>   2. 回调方法存在两个以上参数的情况，以类的名字开头，以表明此方法是属于哪个类的

```
@protocol UITableViewDataSource<NSObject>
@required
//回调方法存在两个以上参数使用类的名字开头
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;
@optional
//回调方法的参数只有类自己
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;
```

####  Enum
> - Enum类型的命名与类的命名规则一致
> - Enum中枚举内容的命名需要以该Enum类型名称开头
> - NS_ENUM定义通用枚举，NS_OPTIONS定义位移枚举
```
typedef NS_ENUM(NSInteger, UIViewAnimationTransition) {
    //UIViewAnimationTran枚举名称开头
    UIViewAnimationTransitionNone,
    UIViewAnimationTransitionFlipFromLeft,
    UIViewAnimationTransitionFlipFromRight,
    UIViewAnimationTransitionCurlUp,
    UIViewAnimationTransitionCurlDown,
};

typedef NS_OPTIONS(NSUInteger, UIControlState) {
    UIControlStateNormal       = 0,
    UIControlStateHighlighted  = 1 << 0,
    UIControlStateDisabled     = 1 << 1,
};
```


### 编码规范
#### Initialize规范
> - 如果我们想要让initialize方法仅仅被调用一次，那么需要借助于GCD的dispatch_once()

```
+ (void)initialize { 
    static dispatch_once_t onceToken = 0; 
    dispatch_once(&onceToken, ^{ 
        // the initializing code 
    }
}
```
> - 如果我们想在继承体系的某个指定的类的initialize方法中执行一些初始化代码，可以使用类型检查和而非dispatch_once()

```
if (self == [NSFoo class]) {
     // the initializing code 
}
```
> 总结：任何子类都会调用父类的initialize方法，所以可能会导致某个父类的initialize方法会被调用多次，为了避免这种情况，我们可以使用类型判等或dispatch_once()这两种方式，以保证initialize中的代码不会被无辜调用


#### Init方法规范
> - Objective-C有designated Initializers和secondary Initializers的概念,designated Initializers叫做指定初始化方法.
> - 一个类可以有一个或者多个designated Initializers。但是要保证所有的其他secondary initializers都要调用designated Initializers
> 1. 所有secondary 初始化方法都应该调用designated 初始化方法。
> 2. 所有子类的designated初始化方法都要调用父类的designated初始化方法。使这种调用关系沿着类的继承体系形成一条链
> 3. 如果子类的designated初始化方法与超类的designated初始化方法不同，则子类应该覆写超类的designated初始化方法。
> 4. 如果超类的某个初始化方法不适用于子类，则子类应该覆写这个超类的方法，并在其中抛出异常。
> 5. 禁止子类的designated初始化方法调用父类的secondary初始化方法。否则容易陷入方法调用死循环。

```
// 超类
    @interface ParentObject : NSObject 
    @end 
    @implementation ParentObject 
    //designated initializer 
    - (instancetype)initWithURL:(NSString*)url title:(NSString*)title { 
        if (self = [super init]) {
            _url = [url copy];
            _title = [title copy];
        } 
        return self;
    } //secondary initializer 
    - (instancetype)initWithURL:(NSString*)url { 
        return [self initWithURL:url title:nil];
    } 
    @end 
// 子类 
    @interface ChildObject : ParentObject 
    @end 
    @implementation ChildObject 
    //designated initializer 
    - (instancetype)initWithURL:(NSString*)url title:(NSString*)title { 
        //在designated intializer中调用 secondary initializer，错误的 
        if (self = [super initWithURL:url]) {

        } return self;
    } 
    @end 
    @implementation ViewController 
    - (void)viewDidLoad {
        [super viewDidLoad]; 
        // 这里会死循环 
        ChildObject* child = [[ChildObject alloc] initWithURL:@"url" title:@"title"];
    } 
    @end 
```
#### dealloc规范
> - 不要忘记在dealloc方法中移除通知和KVO。
> - 和init方法一样，禁止在dealloc方法中使用self.xxx的方式访问属性。如果存在继承的情况下，很有可能导致崩溃
> - 在dealloc方法中，禁止将self作为参数传递出去，如果self被retain住，到下个runloop周期再释放，则会造成多次释放crash

```
- (void)dealloc{
    [self unsafeMethod:self];    
    // 因为当前已经在self这个指针所指向的对象的销毁阶段，销毁self所指向的对象已经在所难免。如果在unsafeMethod:中把self放到了autorelease poll中，那么self会被retain住，计划下个runloop周期在进行销毁。但是dealloc运行结束后，self所指向的对象的内存空间就直接被回收了，但是self这个指针还没有销毁(即没有被置为nil)，导致self变成了一个名副其实的野指针。
    // 到了下一个runloop周期，因为self所指向的对象已经被销毁，会因为非法访问而造成crash问题。
}
```
####  Block规范
> 调用block时需要对block判空。
> 注意block潜在的引用循环。

#### Notification规范
> - 频繁使用通知时，同样会让你崩溃到走投无路。所以，在每个应用中，我们应该时刻留意并控制通知的数量，避免通知满天飞的现象
> - post通知时，object通常是指发出notification的对象，如果在发送notification的同时要传递一些额外的信息，请使用userInfo，而不是object。
> - NSNotificationCenter在iOS8及更老的系统有一个多线程bug，selector执行到一半可能会因为self的销毁而引起crash，解决的方案是在selector中使用weak_strong_dance。
> 
> - 在多线程应用中，Notification在哪个线程中post，就在哪个线程中被转发，而不一定是在注册观察者的那个线程中。如果post消息不在主线程，而接受消息的回调里做了UI操作，需要让其在主线程执行。
> - 总结：每个进程都会创建一个NotificationCenter，这个center通过NSNotificationCenter defaultCenter获取，当然也可以自己创建一个center。
> NoticiationCenter是以同步（非异步，当前线程，会等待，会阻塞）的方式发送请求。即，当post通知时，center会一直等待所有的observer都收到并且处理了通知才会返回到poster。如果需要异步发送通知，请使用notificationQueue，在一个多线程的应用中，通知会发送到所有的线程中
```
- (void)onMultiThreadNotificationTrigged:(NSNotification *)notify {
    __weak typeof(self) wself = self; __strong typeof(self) sself = wself; 
    if (!sself) { return; }
    [self doSomething]; 
}
```

#### UI规范
> - 如果想要获取window，不要使用view.window获取。请使用[[UIApplication sharedApplication] keyWindow]

#### IO规范
> - 尽量少用NSUserDefaults。 说明：[[NSUserDefaults standardUserDefaults] synchronize] 会block住当前线程，知道所有的内容都写进磁盘，如果内容过多，重复调用的话会严重影响性能。
> - 一些经常被使用的文件建议做好缓存。避免重复的IO操作。建议只有在合适的时候再进行持久化操作。

#### Collection规范
- 不要用一个可能为nil的对象初始化集合对象，否则可能会导致crash。
```
// 可能崩溃 NSObject *obj = somOjbcetMaybeNil;
NSMutableArray *arrM = [NSMutableArray arrayWithObject:obj];
```
> - 对插入到集合对象里面的对象也要进行判空
> - 注意在多线程环境下访问可变集合对象的问题，必要时应该加锁保护。不可变集合(比如NSArray)类默认是线程安全的，而可变集合类(比如NSMutableArray)不是线程安全的
> - 禁止在多线程环境下直接访问可变集合对象中的元素。应该先对其进行copy，然后访问不可变集合对象内的元素


```
// 正确写法
- (void)checkAllValidItems{
    NSArray *array = [array copy];
    [array enumerateObjectsUsingBlock:^(id _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
    //do something using obj
    }];
}
// 错误写法
- (void)checkAllValidItems{
    [self.allItems enumerateObjectsUsingBlock:^(id _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
    //do something using obj
    // 如果在enumerate过程中，其他线程对allItems这个可变集合进行了变更操作，这里就有可能引发crash
    }];
}
```
> - 注意使用enumerateObjectsUsingBlock遍历集合对象中的对象时，关键字return的作用域。block中的return代表的是使当前的block返回，而非使当前的整个函数体返回。以下使用NSArray举例，其他集合类型同理。
> - 说明：其实block相当于一个匿名函数，在block中使用return返回，仅是让当前这个匿名函数返回。

```
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    NSArray *array = [NSArray arrayWithObject:@"1"];
    [array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        // excute some code...
        return;
    }];
    // 依然会执行到这里
    NSLog(@"fall through");
}
// 执行结果：
// fall through
```
#### 分支语句规范
> - if条件判断语句后面必须要加大括号{}。不然随着业务的发展和代码迭代，极有可能引起逻辑问题

```
// 建议
if (!error) {
    return success;
}
// 不建议
if (!error) 
    return success;
     
if (!error)  return success;
```
> - 多于3个逻辑表达式必须用参数分割成多个有意义的bool变量
> - 遵循gold path法则，不要把真正的逻辑写道括号内

```
// 不建议
- (void)someFuncWith:(NSString *)parameter {
    if (parameter) {
        // do something
        [self doSomething];
    }
}
// 建议
- (void)someFuncWith:(NSString *)parameter {
    if (!parameter) {
        return;
    }
    // do something
    [self doSomething];
}
```
> - 对于条件语句的真假，因为 nil 解析为 NO，所以没有必要在条件中与它进行比较。永远不要直接和 YES 和 NO进行比较，因为 YES 被定义为 1，而 BOOL 可以多达 8 位

```
// 建议
if (isAwesome)
if (![someObject boolValue])
// 禁止这样做
if ([someObject boolValue] == NO) { }
if (isAwesome == YES) { }
```
> - 使用switch...case...语句的时候，不要丢掉default:。除非switch枚举。
> - switch...case...语句的每个case都要添加break关键字，避免出现fall-through。
> 

#### 对象判等规范
> - 一个对象的创建依赖于其他对象。
> - 一个对象在整个app过程中，可能被使用，也可能不被使用。
> - 一个对象的创建需要经过大量的计算或者比较消耗性能。除以上三条之外，请不要使用懒加载。
> - 不要滥用懒加载，只对那些真正需要懒加载的对象采用懒加载。
> - 如果一个对象在懒加载后，某些场景下又被设置为nil。我们很难保证这个懒加载不被再次触发。

#### 多线程规范
> - 禁止使用GCD的dispatch_get_current_queue()函数获取当前线程信息。
> - 仅当必须保证顺序执行时才使用dispatch_sync，否则容易出现死锁，应避免使用，可使用dispatch_async。

```
- (void)viewDidLoad {
   [super viewDidLoad];
   // 错误。出现死锁，报错:EXC_BAD_INSTRUCTION。原因:在主队列中同步的添加一个block到主队列中
   dispatch_queue_t mainQueue = dispatch_get_main_queue();
   dispatch_block_t block = ^() {
       NSLog(@"%@", [NSThread currentThread]);
   };
   dispatch_sync(mainQueue, block);
}

- (void)viewDidLoad {
   [super viewDidLoad];
   // 正确。异步执行。虽然还是把任务加到了主队列由主线程来执行，但因为是异步，此时主队列后面的任务不依赖于前面的任务。
   dispatch_queue_t mainQueue = dispatch_get_main_queue();
   dispatch_block_t block = ^() {
       NSLog(@"%@", [NSThread currentThread]);
   };
   dispatch_async(mainQueue, block);
}
```
> - 禁止在非主线程中进行UI元素的操作。
> - 在主线程中禁止进行同步网络资源读取，使用NSURLSession进行异步获取。当然，你可以在子线程同步获取网络资源，但还是上面的那一条建议：避免使用dispatch_sync，尽量使用dispatch_async。因为死锁不一定只发生在主线程。
> - 如果需要进行大文件或者多文件的IO操作，禁止主线程使用，必须进行异步处理。
> - 对剪贴板的读取必须要放在异步线程处理，最新Mac和iOS里的剪贴板共享功能会导致有可能需要读取大量的内容，导致读取线程被长时间阻塞。

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
   UIPasteboard *pasteboard = [UIPasteboard generalPasteboard]; 
   if (pasteboard.string.length > 0) {//这个方法会阻塞线程
      NSString *text = [pasteboard.string copy];
      [pasteboard setValue:@"" forPasteboardType:UIPasteboardNameGeneral];
      if (text == nil || [text isEqualToString:@""]) {
          return ;
      }
      dispatch_async(dispatch_get_main_queue(), ^{
          [self processShareCode:text];
      });
   }
}); 
```
#### 内存管理规范
> - 函数体提前return时，要注意是否有对象没有被释放掉(常见于CF对象)，避免造成内存泄露。
> - 请慎重使用单例，避免产生不必要的常驻内存。
> - 单例初始化方法中尽量保证单一职责,尤其不要进行其他单例的调用。极端情况下，两个单例对象在各自的单例初始化方法中调用，会造成死锁。
> - 在dealloc方法中，禁止将self作为参数传递出去，如果self被retain住，到下个runloop周期再释放，则会造成多次释放crash。这一点在dealloc一节中有说明。
> - 除非你清除的知道自己在做什么。否则不建议将UIView类的对象加入到NSArray、NSDictionary、NSSet中。如有需要可以添加到NSMapTable 和 NSHashTable。因为NSArray、NSDictionary、NSSet会对加入的对象做strong引用（即使你把加入的对象进行了weak）。而NSMapTable、NSHashTable会对加入的对象做weak引用。
> 说明：简单的说，NSHashTable相当于weak的NSMutableArray；NSMapTable相当于weak的NSMutableDictionary.

#### 
> - performSelector:withObject:afterDelay:要在有Runloop的线程里调用，否则调用无法生效。
> 说明：异步线程默认是没有runloop的，除非手动创建；而主线程是系统会自动创建Runloop的。所以在异步线程调用是请先确保该线程是有Runloop的。
> - 使用performSelector:withObject:afterDelay:和cancelPreviousPerformRequestsWithTarget组合的时候要小心：
> afterDelay会增加引用计数，而cancel会对引用计数减一
> 如果receiver在引用计数器为1的时候，调用cancel会立即回收receiver。后续再次调用receiver的方法就会crash。所以我们需要使用weakSelf并判空。

```
    __weak typeof(self) weakSelf = self;
    [NSObject cancelPreviousPerformRequestsWithTarget:self];
        if (!weakSelf) { 
        // NSLog(@"self dealloc"); 
        return;
    }
```



### 注释规范

> 三种情况比较适合写注释
> - 公共接口（注释要告诉阅读代码的人，当前类能实现什么功能）。
> - 涉及到比较深层专业知识的代码（注释要体现出实现原理和思想）。
> - 容易产生歧义的代码（但是严格来说，容易让人产生歧义的代码是不允许存在的）。
> - 单行的用//+空格开头，多行的采用/* */注释
> - 使用//TODO:说明 标记一些未完成的或完成的不尽如人意的地方

#### 属性注释
> - 每个属性，都必须要写上相对应的注释，用///注释
```
/// 学生
@property (nonatomic, strong) Student *student;
```
#### 方法注释
> - 方法注释，方法外部统一用option + command + /，方法内部统一用//注释。

```
/**
 <#Description#>
 
 @param tableView <#tableView description#>
 @param section <#section description#>
 @return <#return value description#>
 */
- (CGFloat)tableView:(UITableView *)tableView heightForHeaderInSection:(NSInteger)section
{
    //...
}

```

### 类的设计规范

> - 尽量减少继承，类的继承关系不要超过3层。可以考虑使用category、protocol来代替继承。
> - 把一些稳定的、公共的变量或者方法抽取到父类中。子类尽量只维持父类所不具备的特性和功能。
> - .h文件中尽量不要声明成员变量。
> - .h文件中的属性尽量声明为只读。
> - .h文件中只暴露出一些必要的类、公开的方法、只读属性；私有类、私有方法和私有属性以及成员变量，尽量写在.m文件中。

### 方法的分组标签模板


```
#pragma mark - LifeCycle  // 生命周期/初始化方法
 
#pragma mark - Setter && Getter // setter getter 方法

#pragma mark - UI // 界面相关

#pragma mark - API  // api请求

#pragma mark - IBAction  // 响应事件

#pragma mark - Override Method  // 重写父类方法

#pragma mark - Private Method // 私有方法

#pragma mark - Public Method // 公用方法

#pragma mark - KVO  // 监听方法

#pragma mark - NSNotifaction // 通知

#pragma mark - <#XXX#>Delegate // 代理 注意放在最下面
```
