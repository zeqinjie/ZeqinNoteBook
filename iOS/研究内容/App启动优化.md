## 参考
- [iOS App 启动性能优化](https://blog.csdn.net/Tencent_Bugly/article/details/77363817?locationNum=1&fps=1)
- [阿里数据iOS端启动速度优化的一些经验](https://www.jianshu.com/p/f29b59f4c2b9)
- [[iOS]一次立竿见影的启动时间优化](http://www.cocoachina.com/ios/20170816/20267.html)
- [今日头条iOS客户端启动速度优化](https://techblog.toutiao.com/2017/01/17/iosspeed/)

### 启动耗时的测量

> 1. pre-main阶段
> 对于pre-main阶段，Apple提供了一种测量方法，在 Xcode 中 Edit scheme -> Run -> Auguments 将环境变量DYLD_PRINT_STATISTICS 设为1 ：
![image](https://upload-images.jianshu.io/upload_images/1171135-8ce6a34b39620ae2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)
> 设置好后把程序跑起来，控制台会有如下输出，pre-main阶段各过程的耗时一览无余
> 2. main()阶段
> 对于main()阶段，主要是测量main()函数开始执行到didFinishLaunchingWithOptions执行结束的耗时，就需要自己插入代码到工程中了。先在main()函数里用变量StartTime记录当前时间：


```
CFAbsoluteTime StartTime;
int main(int argc, char * argv[]) {
    StartTime = CFAbsoluteTimeGetCurrent();
//再在AppDelegate.m文件中用extern声明全局变量StartTime
extern CFAbsoluteTime StartTime;
//最后在didFinishLaunchingWithOptions里，再获取一下当前时间，与StartTime的差值即是main()阶段运行耗时。
double launchTime = (CFAbsoluteTimeGetCurrent() - StartTime);
```


### 影响启动速度

####  main()函数之前耗时的影响因素
> - 动态库加载越多，启动越慢。
> - ObjC类越多，启动越慢
> - C的constructor函数越多，启动越慢
> - C++静态对象越多，启动越慢
> - ObjC的+load越多，启动越慢



---

####  pre-main阶段的优化

> ##### 1. Load dylibs
> 
> 这一阶段dyld会分析应用依赖的dylib，找到其mach-o文件，打开和读取这些文件并验证其有效性，接着会找到代码签名注册到内核，最后对dylib的每一个segment调用mmap()。
> 一般情况下，iOS应用会加载100-400个dylibs，其中大部分是系统库，这部分dylib的加载系统已经做了优化。
> 
>  所以，依赖的dylib越少越好。在这一步，我们可以做的优化有：
> 
> 尽量不使用内嵌（embedded）的dylib，加载内嵌dylib性能开销较大
> 合并已有的dylib和使用静态库（static archives），减少dylib的使用个数
> 懒加载dylib，但是要注意dlopen()可能造成一些问题，且实际上懒加载做的工作会更多
> ##### 2. Rebase/Bind
> 
> 在dylib的加载过程中，系统为了安全考虑，引入了ASLR（Address Space Layout Randomization）技术和代码签名。由于ASLR的存在，镜像（Image，包括可执行文件、dylib和bundle）会在随机的地址上加载，和之前指针指向的地址（preferred_address）会有一个偏差（slide），dyld需要修正这个偏差，来指向正确的地址。
> Rebase在前，Bind在后，Rebase做的是将镜像读入内存，修正镜像内部的指针，性能消耗主要在IO。Bind做的是查询符号表，设置指向镜像外部的指针，性能消耗主要在CPU计算。
> 
> 所以，指针数量越少越好。在这一步，我们可以做的优化有：
> 
> 减少ObjC类（class）、方法（selector）、分类（category）的数量
> 减少C++虚函数的的数量（创建虚函数表有开销）
> 使用Swift structs（内部做了优化，符号数量更少）
> ##### 3. Objc setup
>
> 大部分ObjC初始化工作已经在Rebase/Bind阶段做完了，这一步dyld会注册所有声明过的ObjC类，将分类插入到类的方法列表里，再检查每个selector的唯一性。
> 
> 在这一步倒没什么优化可做的，Rebase/Bind阶段优化好了，这一步的耗时也会减少。
> ##### 4. Initializers
> 到了这一阶段，dyld开始运行程序的初始化函数，调用每个Objc类和分类的+load方法，调用C/C++ 中的构造器函数（用attribute((constructor))修饰的函数），和创建非基本类型的C++静态全局变量。Initializers阶段执行完后，dyld开始调用main()函数。
> 
> 在这一步，我们可以做的优化有：
> 
> 少在类的+load方法里做事情，尽量把这些事情推迟到+initiailize
> 减少构造器函数个数，在构造器函数里少做些事情
> 减少C++静态全局变量的个数


---

####  main()函数之后耗时的影响因素
> - 执行main()函数的耗时
> - 执行applicationWillFinishLaunching的耗时
> - rootViewController及其childViewController的加载、view及其subviews的加载

####  main()阶段的优化
> 这一阶段的优化主要是减少didFinishLaunchingWithOptions方法里的工作，在didFinishLaunchingWithOptions方法里，我们会创建应用的window，指定其rootViewController，调用window的makeKeyAndVisible方法让其可见。由于业务需要，我们会初始化各个二方/三方库，设置系统UI风格，检查是否需要显示引导页、是否需要登录、是否有新版本等，由于历史原因，这里的代码容易变得比较庞大，启动耗时难以控制。
> 
> 所以，满足业务需要的前提下，didFinishLaunchingWithOptions在主线程里做的事情越少越好。在这一步，我们可以做的优化有：
> 1. 梳理各个二方/三方库，找到可以延迟加载的库，做延迟加载处理，比如放到首页控制器的viewDidAppear方法里。
> 1. 梳理业务逻辑，把可以延迟执行的逻辑，做延迟执行处理。比如检查新版本、注册推送通知等逻辑。
> 1. 避免复杂/多余的计算。
> 1. 避免在首页控制器的viewDidLoad和viewWillAppear做太多事情，这2个方法执行完，首页控制器才能显示，部分可以延迟创建的视图应做延迟创建/懒加载处理。
> 1. 采用性能更好的API。
> 1. 首页控制器用纯代码方式来构建。


