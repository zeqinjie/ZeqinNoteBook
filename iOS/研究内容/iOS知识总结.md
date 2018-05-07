# iOS知识总结

## 基础知识篇
####  1.为什么都要在主线程更新UI？


```
1. UI操作并不是线程安全的，多线程更新UI会造成死锁，竞争条件等多种问题，因为子线程代码执行完毕了，又自动进入到了主线程，执行了子线程中的UI更新的函数栈，这中间的时间非常的短，就让大家误以为分线程可以更新UI。==如果子线程一直在运行，则子线程中的UI更新的函数栈主线程无法获知，即无法更新==。
2. 只有极少数的UI能，因为开辟线程时会获取当前环境，如点击某个按钮，这个按钮响应的方法是开辟一个子线程，在子线程中对该按钮进行UI更新是能及时的(时间很短)，如换标题，换背景图，但这没有任何意义。
```

#### 2.为什么iOS开发中，控件一般为weak而不是strong？

```
IBOutlet的属性一般可以设为weak是因为它已经被view引用了，除非view被释放，否则IBOutlet的属性也不会被释放，另外IBOutlet属性的生命周期和view应该是一致的，所以IBOutlet属性一般设为weak
```

#### 3.数据增量 

```
增量更新的原理是在数据库中，每条数据都必须有update_time这个值，记录数据最后更新的时间，当app从服务器获取了一次数据后（返回的数据必须按时间排序，update_time最近的在第一条），记录下第一条数据的update_time，当再次获取数据就只需要获取上个时间点到访问服务器这刻为止所更新的数据即可
```


#### 4.<AVKit/AVKit.h>
```
iOS 8 之后<AVKit/AVKit.h> 替代 <MediaPlayer/MediaPlayer.h>   可以使用 <AVFoundation/AVFoundation.h>
//AVAsset   ALAsset(图片)  被 PHAsset
```


#### 5.HTTP,TCP/IP,UDP,HTTPS,GET,POST

```
TCP/IP是个协议组，可分为三个层次：网络层、传输层和应用层：
网络层：IP协议、ICMP协议、ARP协议、RARP协议和BOOTP协议
传输层：TCP协议（传输控制协议，三次握手 可靠 例如 网页（http）、邮件（SMTP）、远程连接(Telnet)、文件(FTP 文件传输协议传送）
第一次握手：客户端发送syn包(syn=j)到服务器，并进入SYN_SEND状态，等待服务器确认；
第二次握手：服务器收到syn包，必须确认客户的SYN（ack=j+1），同时自己也发送一个SYN包（syn=k），即SYN+ACK包，此时服务器进入SYN_RECV状态；
第三次握手：客户端收到服务器的SYN＋ACK包，向服务器发送确认包ACK(ack=k+1)，此包发送完毕，客户端和服务器进入ESTABLISHED状态，完成三次握手。
UDP协议（用户数据报协议，不可靠 例如  语音广播、视频、QQ、TFTP(简单文件传送）、SNMP（简单网络管理协议）、RTP（实时传送协议）RIP（路由信息协议，如报告股票市场，航空信息）、DNS(域名解释)
）
应用层：FTP、HTTP、TELNET、SMTP、DNS等协议
HTTP是应用层协议，其传输层都是被包装成TCP协议传输，在网络层使用IP 协议。可以用SOCKET实现HTTP。
HTTP 的长连接和短连接  实际上是TCP协议的长连接（响应头加上 Connection:keep-alive）和短连接。
HTTP 响应头有
	Cache-Control ：是否缓存
	Connection：keep-alive 使用保持长连接
	Content-Encoding: 告诉客户端服务端发送资源采用何种编码
	Content-Type：text/html;charset=UTF-8 告诉客户端使用何种编码
      Date：服务器发送资源的日期
	Content-leng：资源长度
Cookie：标识客户端，存储客户端的资源，用户行为


长短连接 的与否字于（客户端和服务端建立连接 后 完成一次数据读写之后 客户端是否向服务器发送关闭请求，如长连接则）
SOCKET是实现传输层协议的一种编程API，可以是TCP，也可以是UDP。
SOCKET套接字 是通信的基石，是支持TCP/IP协议的网络通信的基本操作单元。它是网络通信过程中端点的抽象表示，
包含进行网络通信必须的五种信息：连接使用的协议，本地主机的IP地址，本地进程的协议端口，远地主机的IP地址，远地进程的协议端口。
应用层可以和传输层通过Socket接口，区分来自不同应用程序进程或网络连接的通信，实现数据传输的并发服务
建立socket连接 需要一对套接字 客户端 和服务端
SOCKET 支持不同的传输协议  如果是TCP 则SOCKET连接就是TCP 连接  所以才会说 HTTP 连接 – TCP连接 – SOCKET连接 
```
HTTPS
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCE17356661b8133b310a274eeb98dfb480/2738)
```
HTTPS是在应用层和传输层之间加入了SSL(Secure Socket Layer)安全套接层和TLS(Transport Layer Security)安全传输层协议

HTTP + SSL加密 需要ca 证书  
端口443 不是 HTTP 的80
SSL 使用40位RC4 ，MD5,RSA等加密算法，
HTTP 是超文本传输协议，是明文传输，HTTPS是具有安全性的SSL加密传输协议
具体讲，客户端有一个对称的密钥，通过服务器的证书交换密钥
HTTP/1.0 使用默认使用短连接，1.1使用长链接
GET  和 POST
GET:
请求的数据会附在URL之后（就是把数据放置在HTTP协议头中）
,理论上是没有限制大小的（HTTP协议规范没有对URL长度进行限制），
因为URL不存在参数上限的问题。这个限制是特定的浏览器及服务器对它的限制。
POST:
是没有大小限制的，HTTP协议规范也没有进行大小限制,起限制作用的是服务器的处理程序的处理能力。
所以存在 着80K/100K的大小限制。
POST的安全性要比GET的安全性高 ，因为通过GET提交数据，用户名和密码将明文出现在URL上，
有的缓存在浏览器缓存中所以，但是POST提交的数据是表单中。
```

#### 6.bitcode 

```
bitcode是被编译程序的一种中间形式的代码。包含bitcode配置的程序将会在App store上被编译和链接。bitcode允许苹果在后期重新优化我们程序的二进制文件，而不需要我们重新提交一个新的版本到App store上。
```

#### 7.load和initialize方法

```
load 和 initialize都只会执行一次，无需[super load] （如果子类重写且super load则会重新调用），类别都会调用
load 在应用加载所有的类的时候就执行了，先于main 方法执行
load:  加载类时候被调用，启动App时候 ，父类优先子类调用load ，主类优先 类别调用
initialize :第一次调用当前类的方法时调用（第一次alloc时调用），线程安全, 父类优先子类调用initialize，如果有类别 最后的类别会调用，覆盖掉主类的，父类还是会调用
initialize 的调用发生在 +init 方法之前
load是只要类所在文件被引用就会被调用，而initialize是在类或者其子类的第一个方法被调用前调用。所以如果类没有被引用进项目，就不会有load调用；但即使类文件被引用进来，但是没有使用，那么initialize也不会被调用

在Method Swizzling 中写在load 中且 用单例写法，因为有可能重写且super load
```


#### 8．swift  oc  混编

```
项目名称-Bridging-Header.h
项目名称-Swift.h

附上arc  marc  : 
对于ARC工程中，如果要混编MRC文件，需要在工程的Compiler Flags添加-fno-bojc-arc；对于MRC工程中，如果要混编ARC文件，需要设置为-fobjc-arc。
```


#### 9. @synthesize  @dynamic  

```
@synthesize: 自动生成属性的setter和getter方法 没有属性自动生成属性
1.在协议（类别）中定义属性（只是声明一个getter setter 方法而已 需自己实现该setter getter 方法），有时候对于实现该协议，直接使用@synthesize var;会自动帮我们实现setter  和 getter 方法 同时生成一个var的属性（而不是_ var属性）
2.现在定义属性默认不写@synthesize，相当于以前@syntheszie var = _var;
@dynamic: getter和setter方法会在程序运行的时候或者用其他方式动态绑定
1.就是告诉编译器getter和setter方法无需自动生成，我们自己实现。注意如果自己没有实现调用.语法时会crash
```


#### 10.runtime

```
常用函数
objc_getAssociatedObject()
objc_setAssociatedObject()
method_getImplementation
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types)
//修改已经存在的方法
IMP class_replaceMethod(Class cls, SEL name, IMP imp,  const char *types) 

Method class_getInstanceMethod(Class cls, SEL name)
//交换方法
void method_exchangeImplementations(Method m1, Method m2)


Swizzled的使用
在load 方法中调用
+ （void）load{
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken,^{
     	SEL	 originalSelector = @selector(originalMethod:);
     	SEL  changeSelector = @selector(changeMethod:);
     	Method originalMethod = class_getInstanceMethod(self, originalSelector);
     	Method overrideMethod = class_getInstanceMethod(self, overrideSelector);
     	if (class_addMethod(self, originalSelector, method_getImplementation(overrideMethod), method_getTypeEncoding(overrideMethod))) {
         	 class_replaceMethod(self, overrideSelector, method_getImplementation(originalMethod), method_getTypeEncoding(originalMethod));
         } else {
    	      	 method_exchangeImplementations(originalMethod, overrideMethod);
    		}
}

});
}
+ (void)exchangeInstanceMethod1:(SEL)method1 method2:(SEL)method2
{
    method_exchangeImplementations(class_getInstanceMethod(self, method1), class_getInstanceMethod(self, method2));
}

+ (void)exchangeClassMethod1:(SEL)method1 method2:(SEL)method2
{
    method_exchangeImplementations(class_getClassMethod(self, method1), class_getClassMethod(self, method2));
}

我们可以利用 method_exchangeImplementations 来交换2个方法中的IMP，
我们可以利用 class_replaceMethod 来修改类，
我们可以利用 method_setImplementation 来直接设置某个方法的IMP，……

三者区别：
runtime如何实现weak属性?
通过关联属性
// 声明一个weak属性，这里假设delegate，其实weak关键字可以不使用，
// 因为我们重写了getter/setter方法
@property (nonatomic, weak) id delegate;
 
- (id)delegate {
  return objc_getAssociatedObject(self, @"__delegate__key");
}
 
// 指定使用OBJC_ASSOCIATION_ASSIGN，官方注释是：
// Specifies a weak reference to the associated object.
// 也就是说对于对象类型，就是weak了
- (void)setDelegate:(id)delegate {
  objc_setAssociatedObject(self, @"__delegate__key", delegate, OBJC_ASSOCIATION_ASSIGN);
}
```

#### 11. Instrument  可以检测 电池的耗电量、和内存的消耗。的用法。

#### 12. NSRunLoopCommonModes  NSDefaultRunLoopMode UITrackingRunLoopMode

```
当滚动UIScrolView,UITablView，UICollection 会主线程的(mainLoop  默认是Defalut模式)RunLoop,会切换成UITrackingRunLoopMode，使NSTimer ，NSURLCollection,NSStream(加入的mainLoop中)  停止，所以需改成NSRunLoopCommonModes.
```

#### 13. 线程与进程的区别和联系

```
一个进程就代表着一个应用程序，而操作系统为了更好的利用资源，提供了线程用于处理并发。
线程之间没有有单独的地址空间，处理完成之后还得回到主线程，所以，一个线程死掉就等于整个进程死掉。
进程和线程都是操作系统的基本单元，只是分工不同，是两种不同的资源管理方式。
线程所需要的资源都来自于进程，它没有自己独立的资源，也就没有自己的堆栈和局部变量。
```


#### 14. OC 动态特性：动态类型，动态绑定，动态加载

```
动态类型（id）运行时决定对象类型
动态类型，简单点说就是id 类型，可以理解为通用对象类型，一旦被赋值，可以被强制转化为其它类型。可以通过[obj isKindOfClass:aClass]，来判断其具体类型，作相应操作。这一点在委托（delegate）中体现得比较充分
动态绑定 运行时决定方法调用
动态绑定是基于动态类型的，某个实例被确定后，其类型也是确定的，其对应的属性和方法将会因为类型的确定而确定，这就是动态绑定。
动态加载  运行时加载新模块
动态加载是程序启动时动态加载可执行代码和资源。例如：多国语言的程序，会在程序启动时只加载设定为某一种语言的资源，而不是全部加载。基于Utility Application的程序，分别在iPhone和iPad上运行的时候，只会加载对应的代码和资源。当然兼容视网膜技术的@2x 图片加载也是这样的
```


#### 15．面向对象的三特性：封装，继承，多态      《泛型》NO

#### 16．KVO KVC 


```
KVC即是指NSKeyValueCoding，是一个非正式的Protocol，提供一种机制来间接访问对象的属性。KVO 就是基于KVC实现的关键技术之一。
KVO即Key-Value Observing，是建立在KVC之上，它能够观察一个对象的KVC key path值的变化。 当keypath对应的值发生变化时，会回调observeValueForKeyPath:ofObject:change:context:方法，我们可以在这里处理。
```


```
KVO 的原理(Apple 使用了 isa 混写（isa-swizzling）来实现 KVO 。)
当你观察一个对象时，一个新的类会被动态创建。这个类继承自该对象的原本的类，并重写了被观察属性的 setter 方法。重写的 setter 方法会负责在调用原 setter 方法之前和之后，通知所有观察对象：值的更改。最后通过 isa 混写（isa-swizzling） 把这个对象的 isa 指针 ( isa 指针告诉 Runtime 系统这个对象的类是什么 ) 指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例。
借来的一张示意图，如下所示：
```
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCE297dd8c6b0a9a9aa1395529bb4fea0e8/2712)
```
KVO 确实有点黑魔法：
下面做下详细解释：
键值观察通知依赖于 NSObject 的两个方法: willChangeValueForKey: 和 didChangevlueForKey: 。在一个被观察属性发生改变之前， willChangeValueForKey: 一定会被调用，这就会记录旧的值。而当改变发生后， didChangeValueForKey: 会被调用，继而 observeValueForKey:ofObject:change:context: 也会被调用。可以手动实现这些调用，但很少有人这么做。一般我们只在希望能控制回调的调用时机时才会这么做。大部分情况下，改变通知会自动调用。
比如调用 setNow: 时，系统还会以某种方式在中间插入 wilChangeValueForKey: 、 didChangeValueForKey: 和 observeValueForKeyPath:ofObject:change:context: 的调用。大家可能以为这是因为 setNow: 是合成方法，有时候我们也能看到人们这么写代码:
- (void)setNow:(NSDate *)aDate {
    [self willChangeValueForKey:@"now"]; // 没有必要
    _now = aDate;
    [self didChangeValueForKey:@"now"];// 没有必要
}
 
这是完全没有必要的代码，不要这么做，这样的话，KVO代码会被调用两次。KVO在调用存取方法之前总是调用 willChangeValueForKey: ，之后总是调用 didChangeValueForkey: 。怎么做到的呢?答案是通过 isa 混写（isa-swizzling）。第一次对一个对象调用 addObserver:forKeyPath:options:context: 时，框架会创建这个类的新的 KVO 子类，并将被观察对象转换为新子类的对象。在这个 KVO 特殊子类中， Cocoa 创建观察属性的 setter ，大致工作原理如下:

- (void)setNow:(NSDate *)aDate {
    [self willChangeValueForKey:@"now"];
    [super setValue:aDate forKey:@"now"];
    [self didChangeValueForKey:@"now"];
}
 
这种继承和方法注入是在运行时而不是编译时实现的。这就是正确命名如此重要的原因。只有在使用KVC命名约定时，KVO才能做到这一点。
KVO 在实现中通过 isa 混写（isa-swizzling） 把这个对象的 isa 指针 ( isa 指针告诉 Runtime 系统这个对象的类是什么 ) 指向这个新创建的子类，对象就神奇的变成了新创建的子类的实例。这在Apple 的文档可以得到印证：
Automatic key-value observing is implemented using a technique called isa-swizzling… When an observer is registered for an attribute of an object the isa pointer of the observed object is modified, pointing to an intermediate class rather than at the true class …
然而 KVO 在实现中使用了 isa 混写（ isa-swizzling） ，这个的确不是很容易发现：Apple 还重写、覆盖了 -class 方法并返回原来的类。 企图欺骗我们：这个类没有变，就是原本那个类。。。
但是，假设“被监听的对象”的类对象是 MYClass ，有时候我们能看到对 NSKVONotifying_MYClass 的引用而不是对 MYClass 的引用。借此我们得以知道 Apple 使用了 isa 混写（isa-swizzling）
```


#### 17.响应者链
```
命中测试找到响应对象
```
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCEca6d507a84b0711eff098de945ddf365/2713)
```
hitTest:withEvent:事件操作点所在的view是否能响应事件，如果能返回该视图，如果不能返回nil
pointInside:withEvent: 判断事件的触发是否在该区域内从而寻找触发视图

hitTest:withEvent:方法的处理流程:
1.首先调用当前视图的pointInside:withEvent:方法判断触摸点是否在当前视图内；
2.若返回NO,则hitTest:withEvent:返回nil;
3.若返回YES,则向当前视图的所有子视图(subviews)发送hitTest:withEvent:消息，
   所有子视图的遍历顺序是从top到bottom，即从subviews数组的末尾向前遍历,
   直到有子视图返回非空对象或者全部子视图遍历完毕；
4.若第一次有子视图返回非空对象,则hitTest:withEvent:方法返回此对象，处理结束；
5.如所有子视图都返回NO，则hitTest:withEvent:方法返回自身(self)。

以下视图的hitTest:withEvent:方法会返回nil，导致自身和其所有子视图不能被hit-testing发现，无法响应触摸事件： 1.隐藏(hidden=YES)的视图 2.禁止用户操作(userInteractionEnabled=NO)的视图 3.alpha<0.01的视图 4.视图超出父视图的区域

```
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCEd9898f99df9e2a173008f16a60cb0f01/2715)
```

触摸点在view A中，所以要先检查子view B和C。
触摸点不在view B中，但在C中，所以检查C的子view D和E。
触摸点不在D中，但在E中。View E是这个层级上处于lowest的view的边界范围包含触摸点，所以它成为了hit-test view

过程：（上图点击View E） 
UIWindow的根视图发送 hitTest:withEvent:消息,返回 A
A 发送pointInside:withEvent方法，如果返回YES 则说明触摸点是在这个范围内（这里是E 是在A 范围内），返回YES
那么遍历A的子视图，向所有A的子视图（B,C，（top-bottom）优先从最后addSubview 视图发起） 发送hitTest方法，然后去发送pointInside方法判断是否在该子视图上，如上B 返回NO ，
则向C 发送hisTest方法-> pointInside方法 判断是C ,继续向C的子视图（D,E）发送hitTest方法。
D hitTest ->pointInside 返回NO，
则向E 发送hitTest->pointInside方法返回YES 且没有子视图则hitTest对象就是E视图。

假如第一次所有子视图返回NO 则就是A本身，假如
事件的响应者链
```
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCEa207904d4b8fec4ab8e8228e7f18ffb1/2716)
```
左半图:
initial view若不能处理事件，则传到其父视图view
view若不能处理，则传到其父视图，因为它还不是最上层视图
这里view的父视图是view controller的view，因为这个view也不能处理事件，因此传给view controller
若view controller也不能处理此事件，则传到window
若window也不能处理此事件，则传到app单例对象Application
若UIApplication单例对象也不能处理，则表示无效事件

右半图：
initial view一直传递直到最上层view(原话：A view passes an event up its view controller’s view hierarchy until it reaches the topmost view.)
topmost view传递事件到它所在的控制器（原话：The topmost view passes the event to its view controller.）
view controller传递事件到topmost view的父视图，重复前三步，走到到达root controller（原话：passes the event to its topmost view’s superview. Steps 1-3 repeat until the event reaches the root view controller.）
由root控制器传递事件到window（原话：The root view controller passes the event to the window object.）
若window也不能处理此事件，则传到app单例对象Application
若UIApplication单例对象也不能处理，则表示无效事件


事件的传递和响应分两个链：
事件传递链：    由系统向离用户最近的view传递。UIKit –> active app’s event queue –> window –> root view –>……–>lowest view
事件响应者链：  由离用户最近的view向系统传递。initial view –> super view –> …..–> view controller –> window –> Application

事件传递的完整过程
先将事件对象同一层级由上往下（这里意思是top 到bottom 就是最后addSubview 的是top）传递(由父控件传递给子控件)，找到最合适的控件来处理这个事件。 调用最合适控件的touches….方法 如果调用了[super touches….];就会将事件顺着响应者链条往上传递，传递给上一个响应者 接着就会调用上一个响应者的touches….方法

响应者链条的事件传递过程
如果view是控制器的view，就传递给控制器；如不是，则将其传递给它的父视图 在视图层次结构的最顶级视图，如果也不能处理收到的事件或消息，则其将事件或消息传递给window对象进行处理 如果window对象也不处理，则其将事件或消息传递给UIApplication对象 如果UIApplication也不能处理该事件或消息，则将其丢弃

补充：
B 是 A 控件的子View 怎么让 点B 时候A 也被调用呢？
那么只要在自定义BView中 重写
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    NSLog(@"B-----touchesBegan");
    //这里就是super 就是A ，self.nextResponder 也就是A
    [super touchesBegan:touches withEvent:event];
}

响应者 链是从下到上 最终找到事件源view  作为第一响应者
然后从上到下 形成一条响应者链，
[self nextResponder] 是找到该view 的父view
First Responser -- > The Window -- >The Application -- > App Delegate (下层次)
所有的子视图 调用 hitTest （返回YES 则该子视图的子视图 调用hitTest方法）直到所有的子视图的子视图不存在子视图则该视图为第一响应者。
```

#### 18.__bridge, __bridge_transfer, __bridge_retained
```
1.__bridge:CF和OC对象转化时只涉及对象类型不涉及对象所有权的转化

OC -> CF
NSString *ocStr =  [NSString stringWithFormat:@"bridge"];
CFStringRef cfStr = (__bridge CFStringRef)ocStr;
系统为我们自动添加了__bridge,因为是OC创建的对象并且在转换时没有涉及对象所有权的转换，所以上面的代码不需要加CFRelease,因为OC（ARC环境下）还是在管理着ocStr 
//所以无需CFRelease(cfStr)

CF -> OC  使用__bridge:CF 则需要释放CF
CFStringRef cfStr = CFStringCreateWithCString(NULL, "bridge", kCFStringEncodingUTF8);
NSString *ocStr = (__bridge NSString *)(cfStr);
//注意由于__bridge 不会转换管理权  所以CF 函数是需要释放的
CFRelease(cfStr);

2.__bridge_transfer:常用在讲CF对象转换成OC对象时，将CF对象的所有权交给OC对象，此时ARC就能自动管理该内存；（作用同CFBridgingRelease()），如果OC 释放掉则 CF 无法使用
CF -> OC
CFStringRef cfStr = CFStringCreateWithCString(NULL, "__bridge_transfer", kCFStringEncodingUTF8);
NSString *ocStr = (__bridge_transfer NSString *)(cfStr);
//无需CFRelease(cfStr)，这时已经把权限交给了 ARC 了,如果ARC已经释放了内存,那么CF对象还是无法读取内存

3.__bridge_retained:（与__bridge_transfer相反）常用在将OC对象转换成CF对象时，将OC对象的所有权交给CF对象来管理；(作用同CFBridgingRetain())
OC -> CF
NSString *ocStr =  [NSString stringWithFormat:@"__bridge_retained"];
CFStringRef cfStr = (__bridge_retained CFStringRef)ocStr;
//这里将管理权交给CF，所以需要释放
CFRelease(cfStr);
```


#### 19.野指针
```
就是僵尸对象，被系统回收内存的对象，对象指针地址指向一块被回收的地址。调用该指针的方法会崩溃，XCODE zombie enable 打开。
当时nil 发送方法不会崩溃，因为objc_megSend(idself,SEL,ivar…) 返回的也是self  当idself 是nil 时候返回调用的地方继续执行

1.objc_msgSend方法根据对象的isa指针找到对象的类，然后在类的调度表（dispatch table）中查找selector。如果无法找到selector，objc_msgSend通过指向父类的指针找到父类，并在父类的调度表（dispatch table）中查找selector，以此类推直到NSObject类。一旦查找到selector，objc_msgSend方法根据调度表的内存地址调用该实现。通过这种方式，message与方法的真正实现在执行阶段才绑定(动态绑定)。
2.为了保证消息发送与执行的效率，系统会将全部selector和使用过的方法的内存地址缓存起来。每个类都有一个独立的缓存，缓存包含有当前类自己的 selector以及继承自父类的selector。查找调度表（dispatch table）前，消息发送系统首先检查receiver对象的缓存。缓存命中的情况下，消息发送（messaging）比直接调用方法（function call）只慢一点点点点。
```

#### 20 nil、Nil、NULL和NSNull区别
```
NULL、nil、Nil这三者对于Objective-C中值是一样的，都是(void *)0，那么为什么要区分呢？又与NSNull之间有什么区别：
NULL是宏，是对于C语言指针而使用的，表示空指针
nil是宏，是对于Objective-C中的对象而使用的，表示对象为空
Nil是宏，是对于Objective-C中的类而使用的，表示类指向空
NSNull是类类型，是用于表示空的占位对象，与JS或者服务端的null类似的含意
```

#### 21.NSStringFromClass([self class])和NSStringFromClass([super class])输出都是self的类名。原因如下？

```
调用[self class]的时候先调用objc_msgSend，发现self没有class这个方法，然后用objc_msgSendSuper就去父类找，还是没有，继续用objc_msgSendSuper到NSObject里找，结果找到了，查找NSObject中class方法的runtime源码会发现它会返回self，所以[self class]会返回self本身。同理[super class]相对前者就是少了objc_msgSend这一步，最后也会找到NSObject根类里的class方法，自然结果也是返回了self
```

#### 22.什么是RunLoop？（可以使线程不被销毁 继续等待使用，通过等待 – 执行 – 睡眠 方式）

```
NSRunLoop 是基于 CFRunLoopRef 的封装
从字面意思看就是运行循环，其实内部就是do－while循环，这个循环内部不断地处理各种任务（比 如Source，Timer，Observer）
它是用来调度和协调接收到的事件处理。使用Run Loop的目的，就是使得线程有工作需要做时可以忙碌起来，
而当没有事可做时，又可以使得线程睡眠
一个线程对应一个RunLoop（懒加载），主线程的RunLoop默认已经启动，子线程的RunLoop得手动启动（run方法）
RunLoop只能选择一个Mode启动，如果当前Mode中没有任何Source，Timer，Observer，那么就直接退出RunLoop 
 
二 你在开发过程中怎么使用RunLoop？什么应用场景？
开启一个常驻线程（让一个子线程不进入消亡状态，等待其他线程发来的消息，处理其他事件）
在子线程中开启一个定时器
在子线程中进行一些长期监控
可以控制定时器在特定模式下运行
可以让某些事件（行为，任务）在特定模式下执行
可以添加observer监听RunLoop的状态，比如监听点击事件的处理（比如在所有点击事件前做一些处理）
 
 三 自动释放池什么时候释放？
在RunLoop睡眠或者销毁之前释放（kCFRunLoopBeforeWaiting）


补：
1.主线的Runloop默认开启
作用是使程序一直运行接受用户输入
决定程序在何时应该处理哪些Event
调用解耦
节省CPU时间
2.其他线程开启runloop是否在于
使用端口或自定义输入源来和其他线程通信
使用线程的定时器
Cocoa中使用任何performSelector…的方法
使线程周期性工作
```

#### 23.performSelector
```
- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay inModes:(NSArray *)modes; - (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay; 这两个方法为异步执行，即使delay传参为0，仍为异步执行。只能在主线程中执行，在子线程中不会调到aSelector方法。可用于当点击UI中一个按钮会触发一个消耗系统性能的事件，在事件执行期间按钮会一直处于高亮状态，此时可以调用该方法去异步的处理该事件，就能避免上面的问题。 在方法未到执行时间之前，取消方法为： + (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget selector:(SEL)aSelector object:(id)anArgument; + (void)cancelPreviousPerformRequestsWithTarget:(id)aTarget; 注意：调用该方法之前或在该方法所在的viewController生命周期结束的时候去调用取消函数，以确保不会引起内存泄露。

为什么- (void)performSelector:(SEL)aSelector withObject:(id)anArgument afterDelay:(NSTimeInterval)delay;引起内存泄露
首先会将anArgument 的引用计数+1，执行完这个方法才会减1 但是由于delay导致没有及时-1，测试发现需VC需等待aSelector 方法执行完VC 才会delloc 所以延迟1个小时那么该VC 就会在1小时后才会dellco
//使用下面的延迟如果是self 必须改成__weak typeof(self)weakSelf = self;  weakSelf 否也会等待delay 后才会dealloc
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(20 *NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
      [self changeTitleState];
});
```

#### 24. 有很多请求  用多线程怎么让其中只要一个接口调用失败 就重新发请求?

#### 25 runloop 怎么控制线程? runMode：beforeDate 通过while 设置当前线程runloop 不执行，阻碍当前线程

```
1.在子线程中使用[NSRunloop currentRunLoop]  获取当前子线程的runloop （默认子线程会分配对应的runloop 但是不会启动runloop，需要添加对应的输入源(Timer源或者 输入源)）同时使用run 启动当前runloop 。

2.NSTimer ：
在主线程中  scheduledTimerWithTimeInterval 会自动加入主线程的runloop 同时自动启动runloop 从而启动定时器
但是在子线程中发起的需要收收动添加到当前子线程的runloop中才会启动定时器。
对于timerWithTimeInterval 无论是主线程还是子线程都必须手动添加到当前的runloop中 同时启动runloop 的run方法，才会启动timer 打印回去当前的runloop 和 当前的thread

runloop 阻碍当前线程 等待其他线程执行后再执行从而控制线程操作
//下面是阻碍主线程的操作
	[NSThread detachNewThreadSelector:@selector(runOnNewThread) toTarget:self withObject:nil];//开启子线程
    	while (!end) {//默认end NO 直到别的线程将end设置成YES
       	 NSLog(@"runloop…");
		// distantFuture遥远的，就是当前runloop在遥远的未来在执行。到达阻碍当前线程的效果
       	 [[NSRunLoop currentRunLoop] runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];//这句代码是使当前线程被阻碍
        	 NSLog(@"runloop end.");
 	}
NSLog(@"ok.");

-(void)runOnNewThread{
    NSLog(@"run for new thread …");
    sleep(1);
    [self performSelectorOnMainThread:@selector(setEnd) withObject:nil waitUntilDone:NO];
    NSLog(@"end");
}

-(void)setEnd{
    end=YES;//线程执行完毕
}
```


#### 26. __weak , __block, __strong__weak 
```
typeof(self)weakSelf = self //避免强引用，导致内存泄露
__strong typeof(weakSelf)strongSelf = weakSelf; //在__weak前提下使block强引用防止提前释放
1.__block不管是ARC（会被强引用，现在使用__weak）还是MRC（不会,以前使用它）模式下都可以使用，主要修饰对象或者基本数据类型 用于更改变量值。  2.__weak只能在ARC模式下使用，也只能修饰对象（NSString），不能修饰基本数据类型（int）。用于避免block 的强引用  3.同时在block在使用__strong 强引用下避免block提前释放（注意是对weakSelf）
4.__weak是_unsafe_unretained修饰符替代品
27.私有变量
@private声明变量， 放在m文件中定义 ,objc 没有绝对私有变量和私有方法可以使用KVO 或则使用如下runtime方法 
Ivar userNameIvar = class_getInstanceVariable([model class], "_userName");
NSString *userName = object_getIvar(model, userNameIvar);

@public  声明的属性可以被获取  
@protected 不可以被获取
```

#### 28.strong, retain,copy,weak,assign, unsafe_unretained 修饰属性的不同点

```
strong: 在arc 环境下使用，相当于retain 但是 在使用block的情况下arc环境下会相当于copy，而mrc 下相当于assgin
retain: mrc 用的多，和arc 下 用strong一样  强引用下对象
copy:拷贝对象，但是对于（NSString,NSArray,NSDirctionary是浅拷贝，mutableCopy 才是深拷贝// NSCopying, NSMutableCopying），一般是NSString 和 block使用
weak:弱引用，指针地址被释放会自动置nil，deletegate
unsafe_unretained:同上但是不会置nil
assign:基础类型使用,不会置nil,MRC 下deletegate 使用

ARC下不显式指定任何属性关键字时，默认的关键字都有哪些？
对于基本数据类型默认关键字是：atomic,readwrite,assign
对于普通的Objective-C对象：atomic,readwrite,strong
```

#### 29.autorelease的对象何时被释放

```
如果了解一点点Run Loop的知道，应该了解到：Run Loop在每个事件循环结束后会去自动释放池将所有自动释放对象的引用计数减一，若引用计数变成了0，则会将对象真正销毁掉，回收内存。
所以，autorelease的对象是在每个事件循环结束后，自动释放池才会对所有自动释放的对象的引用计数减一，若引用计数变成了0，则释放对象，回收内存。因此，若想要早一点释放掉auto release对象，那么我们可以在对象外加一个自动释放池。比如，在循环处理数据时，临时变量要快速释放，就应该采用这种方式
for (int i = 0; i < 10000000; ++i) {
   @autoreleasepool {
      HYBTestModel *tempModel = [[HYBTestModel alloc] init];
      // 临时处理
      // ...
   } // 出了这里，就会去遍历该自动释放池了
}
```

#### 30.drawRect，layoutSubviews，updateConstraints

```
在调用setNeedsDisplay后，会调用drawRect方法，我们通过在此方法中可以获取到context（设置上下文），就可以实现绘图
在调用setNeedsLayout后，会调用layoutSubviews方法，我们可以通过在此方法去调整UI。当然能引起layoutSubviews调用的方式有很多种的,初始化时(init)不会触发，除非设置(initWithFame)fame，比如addSubviews、滚动scrollview、修改视图的frame等。旋转screen 改变UIView 的frame 会触发父UIView的layoutSubviews.
补充： 如果你想要在外部设置subviews的位置，就不要重写layoutSubviews。 
刷新子对象布局 （内部代码修改重写layoutSubviews 方法 修改Views的frame，或者需要数据内容）
layoutSubviews对subviews重新布局,方法调用先于drawRect
-setNeedsLayout方法： 标记为需要重新布局，异步调用layoutIfNeeded刷新布局，不立即刷新，但layoutSubviews一定会被调用, 在系统runloop的下一个周期自动调用layoutSubviews链 -layoutIfNeeded方法：如果，有需要刷新的标记，立即调用layoutSubviews进行布局（如果没有标记，不会调用layoutSubviews） 如果要立即刷新，要先调用[view setNeedsLayout]，把标记设为需要布局，然后马上调用[view layoutIfNeeded]，实现布局后，可以直接调用[view layoutIfNeeded] , UIKit会判断是否需要layout，遍历的不是superview链，应该是subviews
重绘  （内部代码重写drawRect方法 使用CGContextRef 绘画）UIGraphisGetCurrentContext
-setNeedsDisplay方法：标记为需要重绘，异步调用drawRect（ 系统runloop下一个周期1/60秒重绘 ） -setNeedsDisplayInRect:(CGRect)invalidRect方法：标记为需要局部重绘
 sizeToFit可以被手动直接调用,会自动调用sizeThatFits方法；不应该在子类中被重写，应该重写sizeThatFits sizeThatFits传入的参数是receiver当前的size，返回一个适合的size sizeToFit和sizeThatFits方法都没有递归，对subviews也不负责，只负责自己
  此外layer 的绘画  使用drawInContext 类似drawRect

- (void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx


修改约束 （重写updateConstraints方法 修改约束  然后[super updateConstraints]）
基于约束的AutoLayer的方法：
1、setNeedsUpdateConstraints
当一个自定义view的某个属性发生改变，并且可能影响到constraint时，需要调用此方法去标记constraints需要在未来的某个点更新，系统然后调用updateConstraints.
2、needsUpdateConstraints
constraint-based layout system使用此返回值去决定是否需要调用updateConstraints作为正常布局过程的一部分。
3、updateConstraintsIfNeeded
立即触发约束更新，自动更新布局。
4、updateConstraints
自定义view应该重写此方法在其中建立constraints. 注意：要在实现在最后调用[super updateConstraints]
一般先[self setNeedsUpdateConstraints]  然后[self updateConstraintsIfNeeded]  注意进入updateConstraints 需先修改约束 最好调super
31.自定义NSOperation，需要实现哪些方法？
对于自定义非并发NSOperation，只需要实现main方法就可以了。
对于自定义并发NSOperation，需要重写main、start、isFinished、isExecuting，还要注意在相关地方加上kvo的代码，通知其它线程，否则当任务完成时，若没有设置isFinished=YES，isExecuting=NO，任务是不会退队的。
```


#### 32.单例
```
+ (instancetype)sharedInstance {
  static id s_manager = nil;
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    s_manager = [[[self class]alloc] init];
  });
  return s_manager;
}
```

#### 33.重用机制

```
UITableView提供了一个属性：visibleCells，它是记录当前在屏幕可见的cell，要想重用cell，我们需要明确指定重用标识（identifier）。当cell滚动出tableview可视范围之外时，就会被放到可重用数组中。当有一个cell滚动出tableview可视范围之外时，同样也会有新的cell要显示到tableview可视区，因此这个新显示出来的cell就会先从可重用数组中通过所指定的identifier来获取，如果能够获取到，则直接使用之，否则创建一个新的cell。
```


#### 34.堆和栈的

```
栈，由编译器管理的，不是我们手动控制，但是栈所能分配的内存是比较少（1M）的，如果要处理大数据，则需要在堆上分配，因此在栈上比较容易出现Memory Leak；
堆，需要我们自己申请内存，同时也需要我们自己手动释放，否则会造成内存泄露；如果过多地申请内存空间，会导致内存空间不连接，从而造成内存碎片，使程序效率降低。
创建对象(NSObject *)是在堆上
临时变量是在栈上
```

#### 35.断点续传需要在请求头中添加的控制续传最重要的关键字

```
Range实体头，指定第一个字节的位置和最后一个字节的位置，而与之对应的响应头有Content-Range，
指示了整个实体的长度及部分插入位置，如Content-Range: bytes 0-500/801，指定了范围为当前范围与文件总大小。
//Length ,type ,size
```



#### 36.有a、b、c、d 
```
4个异步请求，如何判断a、b、c、d都完成执行？如果需要a、b、c、d顺序执行，该如何实现？
// GCD的group
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, queue, ^{ /*任务a */ });
dispatch_group_async(group, queue, ^{ /*任务b */ });
dispatch_group_async(group, queue, ^{ /*任务c */ }); 
dispatch_group_async(group, queue, ^{ /*任务d */ }); 
 
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    // 在a、b、c、d异步执行完成后，会回调这里
});
//要求顺序执行，那么可以将任务放到串行队列中，自然就是按顺序来异步执行了
```




```
假如GCD实现1，2并行和3串行和45串行，4，5是并行。即3依赖1，2的执行，45依赖3的执行。
- (void) methodone{
dispatch_group_t group = dispatch_group_create();
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"%d",1);
});
dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"%d",2);
});
dispatch_group_notify(group, dispatch_get_main_queue(), ^{
    NSLog(@"3");
    dispatch_group_t group1 = dispatch_group_create();
    dispatch_group_async(group1, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"%d",4);
    });
    dispatch_group_async(group1, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"%d",5);
    });
});
}
```


#### 37.webView 的代理方法使用

```
// 服务端
function getComments (name,where)//这里会传参数给第二种方法
{
      //window.location.assign("come")第二种方法回调参数
      alert(name+" "+where)
 }

/*注意
	JS 代码 
function callCamera() {
      window.location.href = 'toyun://callCamera';//使用href 和 assign 都可以拦截在shouldStartLoadWithRequest方法中
   		 }

*/
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
       NSString *url = request.URL.absoluteString;
 // NSString *str=[url stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
	if ([url hasSuffix:@"come"]) {
        DLog(@"getComments");
        return NO;
 }
     if ([url rangeOfString:@"toyun://"].location != NSNotFound) { 
       	 // url的协议头是Toyun
       	 NSLog(@"callCamera");
        return NO;
     }
   	 return YES;
}

//javaScriptContext 一定在DidFinishLoad中使用
- (void)webViewDidFinishLoad:(UIWebView *)webView{
	if (!self.jsContext) {
        self.jsContext = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    }
    __weak __typeof(&*self)weakSelf = self;
    self.jsContext[@"getComments"] = ^(id param,id param2) {
        if (param) {
            dispatch_async(dispatch_get_main_queue(), ^{
                DLog(@"getComments");
            });
        }
};

}

stringByEvaluatingJavaScriptFromString 使用
//注意一定要做didFinishLoad上获取值
//获取高度
   CGFloat webH = [[webView stringByEvaluatingJavaScriptFromString:@"document.body.offsetHeight"] floatValue];
//其实stringByEvaluatingJavaScriptFromString 就是可以执行JS代码
document.location.href //获取当前页的URL
document.title//获取title
document.getElementsByName('q')[0].value='侯文富专栏';//获取元素
document.forms[0].submit();//表单提交

//如下注入一段js 代码 注意script.type=’text/javascript’
[webView stringByEvaluatingJavaScriptFromString:@"var script = document.createElement('script');"
     "script.type = 'text/javascript';"
     "script.text = \"function myFunction() { "
     "var field = document.getElementsByName('word')[0];"
     "field.value='测试';"
     "document.forms[0].submit();"
     "}\";"
     "document.getElementsByTagName('head')[0].appendChild(script);"];
    [webView stringByEvaluatingJavaScriptFromString:@"myFunction();"];
```


#### 38.BAD_ACCESS在什么情况下出现？

```
原因是访问了野指针，比如访问已经释放对象的成员变量或者发消息、死循环等。
举个例子，播放视频时，设置了代理，当快速进入播放再快速返回时，就有可能出现这种bug。
```


#### 39.苹果是如何实现autoreleasepool的（栈）
```
autoreleasepool以一个队列数组的形式实现,主要通过下列三个函数完成.
objc_autoreleasepoolPush
objc_autoreleasepoolPop
objc_autorelease
看函数名就可以知道，对autorelease分别执行push、pop操作。销毁对象时执行release操作。

Autorelease对象是在当前的runloop迭代结束时释放的，而它能够释放的原因是系统在每个runloop迭代中都加入了自动释放池Push和Pop
ARC下，我们使用@autoreleasepool{}来使用一个AutoreleasePool，随后编译器将其改写成下面的样子：
void *context = objc_autoreleasePoolPush();//加入新的AutoreleasedPool 调用 // {}中的代码 objc_autoreleasePoolPop(context);
```
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCE4ddef105f19ac2bd62f6b7067986c559/2719)
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCEf386783af923ab7a59cada97d0f36db6/2718) 
```
objc_autoreleasePoolPush的返回值正是这个哨兵对象的地址，被objc_autoreleasePoolPop(哨兵对象)作为入参，于是：
根据传入的哨兵对象地址找到哨兵对象所处的page
在当前page中，将晚于哨兵对象插入的所有autorelease对象都发送一次- release消息，并向回移动next指针到正确位置
补充2：从最新加入的对象一直向前清理，可以向前跨越若干个page，直到哨兵所在的page
刚才的objc_autoreleasePoolPop执行后


总结：
1.Pool是一个链表连接。开始分配一个Pool,时候会调用objc_autoreleasePoolPush（哨兵对象的地址），从而加入objc_autoreleasePoolPop（哨兵对象），
当autoreleased对象加入过多，满的时候会向最新加入的哨兵对象之前所有对象发released消息。
2.当前线程有多个autoreleasedPool  当前线程的runloop 结束时释放所有的pool里的对象，发release消息。
3. 向一个对象发送- autorelease消息，就是将这个对象加入到当前AutoreleasePoolPage的栈顶next指针指向的位置
```


#### 40.系统的动画block是否考虑循环引用
```
系统的blokc动画
不考虑，例如这些不考虑：
[UIView animateWithDuration:duration animations:^{ [self.superview layoutIfNeeded]; }]; 
[[NSOperationQueue mainQueue] addOperationWithBlock:^{ self.someProperty = xyz; }]; 
[[NSNotificationCenter defaultCenter] addObserverForName:@"someNotification" 
                                                  object:nil 
                           queue:[NSOperationQueue mainQueue]
                                              usingBlock:^(NSNotification * notification) {
                                                    self.someProperty = xyz; 
                                                    }];
但如果你使用一些参数中可能含有成员变量的系统api，如GCD、NSNotificationCenter就要小心一点。比如GCD内部如果引用了 self，而且GCD的其他参数是成员变量，则要考虑到循环引用：

__weak __typeof(self) weakSelf = self;
dispatch_group_async(_operationsGroup, _operationsQueue, ^{
    __typeof __(self) strongSelf = weakSelf;
    [strongSelf doSomething];
    [strongSelf doSomethingElse];
});

__weak __typeof(self) weakSelf = self;
_observer = [[NSNotificationCenter defaultCenter] addObserverForName:@"testKey"
                                                            object:nil
                                                             queue:nil
                                                        usingBlock:^(NSNotification *note) {
  __typeof__(self) strongSelf = weakSelf;
  [strongSelf dismissModalViewControllerAnimated:YES];
}];
```

#### 41.lldb命令

```
breakpoint 设置断点定位到某一个函数
n 断点指针下一步，猜测是next的缩写
po打印对象，笔者猜测po是print object的缩写
p与po类似，只是打印出来的是地址
bt 打印堆栈信息
expr 执行某行代码，比如@import UIKit
```

#### 42.常见的object-c的数据类型有那些， 和C的基本数据类型有什么区别?如：NSInteger和int
```
object-c的数据类型有NSString，NSNumber，NSArray，NSMutableArray，NSData等等，这些都是class，创建后便是对象，而C语言的基本数据类型int，只是一定字节的内存空间，用于存放数值;NSInteger是基本数据类型，并不是NSNumber的子类，当然也不是NSObject的子类。NSInteger是基本数据类型Int或者Long的别名(NSInteger的定义typedef long NSInteger)，它的区别在于，NSInteger会根据系统是32位还是64位来决定是本身是int还是Long。
```

#### 43.Instruments
```
1.time profiler CPU和进程：监视系统活动、采样、负载图表和线程
2.leaks     内存：跟踪垃圾回收、对象分配和泄露
 用户事件：追踪用户交互动作的精确事件，如鼠标点击等。
3.Network   网络活动：衡量并记录网络流量
4. file activity  文件活动：监视磁盘活动，读写和文件锁。
5.opengl   图形：解释OpenGL驱动的内在工作。
6.gpu driver   gpu 帧率
7 engry log  电量
44.UITableView的调优
重用cell，设置好cellIdentifier
重用header、footer view，设置好identifier
若高度固定，直接使用rowHight；若不固定则使用heightForRowAtIndexPath代理方法
缓存cell的高度、header/footer view的高度
不要修改view的opaque，默认就是YES,表示不透明度
不要动态添加子view到cell上，直接在初始时创建，然后做显示与隐藏操作
尽量不要直接使用cornerRadius，采用镂空图或者Core Graphics API来绘制
将I/O操作、复杂运算放到子线程中处理，再回到主线程更新UI
```


#### 45.性能优化

```
1. 用ARC管理内存  2. 在正确的地方使用reuseIdentifier
3. 尽可能使Views不透明  尽量opaque = YES
4. 避免庞大的XIB    5. 不要阻碍主线程   
6. 在Image Views中调整图片大小
7. 选择正确的Collection
8. 打开gzip压缩
9. 重用和延迟加载Views
10. Cache, Cache, 还是Cache！
11. 权衡渲染方法
12. 处理内存警告
13. 重用大开销的对象
14. 使用Sprite Sheets
15. 避免反复处理数据
16. 选择正确的数据格式
17. 正确地设定Background Images
18. 减少使用Web特性
19. 设定ShadowPath  view.layer.shadowOffset 改成 view.layer.shadowPath = [[UIBezierPath bezierPathWithRect:view.bounds] CGPath]
20. 优化你的Table View
21. 选择正确的数据存储选项
22. 加速启动时间
23. 使用Autorelease Pool
24. 选择是否缓存图片
25. 尽量避免日期格式转换
```


#### 46.什么是运行时？运行时的原理？
![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCEa202b0bc42de3d369ea65ad7c88bf92b/2721)
```
1.oc拥有动态特性：包括动态类型，动态加载，动态绑定
2.runtime一套基于C的API ，OC 代码在程序运行过程最终都是转成runtime 的C代码。
```



#### 47.cocopod的使用？

```
编辑profile 文件 
pod init,pod install ,pod update，pod search
深入理解
```


#### 48.什么是block， block的原理？

```
三种block 写法
typedef void(^ Myblock)();
property (nonatomic,copy) Myblock block;
property (nonatomic,copy) void(^block)();
-(void)myblock:(void(^)())block

block的结构
使用终端，转到mian.m文件下，使用如下代码 clang -rewrite-objc main.m 将其编译生成 main.cpp文件 这时候，我们打开mian.cpp便知 在文件的最底下main函数中
block 就是一个闭包，是一个指向函数的指针
block的数据结构定义
struct Block_descriptor {     unsigned long int reserved;     unsigned long int size;     void (*copy)(void *dst, void *src);     void (*dispose)(void *); };  struct Block_layout {     void *isa;     int flags;     int reserved;     void (*invoke)(void *, ...);     struct Block_descriptor *descriptor;     /* Imported variables. */ };
isa 指针，所有对象都有该指针，用于实现对象相关的功能。
flags，用于按 bit 位表示一些 block 的附加信息，本文后面介绍 block copy 的实现代码可以看到对该变量的使用。
reserved，保留变量。
invoke，函数指针，指向具体的 block 实现的函数调用地址。
descriptor， 表示该 block 的附加描述信息，主要是 size 大小，以及 copy 和 dispose 函数的指针。
variables，capture 过来的变量，block 能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。


在 Objective-C 语言中，一共有 3 种类型的 block：
_NSConcreteGlobalBlock 全局的静态 block，不会访问任何外部变量。
//在MRC 环境下存在，在ARC 环境下会替换成下面的堆block 所以在MRC 下需使用copy定义block,ARC下可以使用copy(通常)或者strong
_NSConcreteStackBlock 保存在栈中的 block，当函数返回时会被销毁。
_NSConcreteMallocBlock 保存在堆中的 block，当引用计数为 0 时会被销毁。
```

#### 49.代理的作用？

```
代理的目的是改变或传递控制链,代理传值。
允许一个类在某些特定时刻通知到其他类。
代理一般和协议一起使用，代理实习协议里的方法，达到反向传值，获取数据源，
有时候，父类公开代理给子类，子类实现父类的数据源方法，到达父类获取子类的数据，从而统一在父类中执行某些方法

//
题外话  协议：有时候，定义一个协议，代理对象使用@synthesize 可以 直接生成属性
```

#### 50.static const

```
//静态常量
static  NSString *const name = @”zhengzeqin” 
static  const  CGFloat  f = 12.0;
static  在静态区，所有的实例对象共用一份。
const  常量   修饰 *前 ：表示name 不能修改，*后   name 可以修改 但是*name 不能修改 


注意 不使用static（静态 只属于该类）标记  默认是extern（外部 属于整个项目） 那如果不同文件出现同名的定义会报错
```

#### 51.@class的作用是什么？

```
使用@class来声明这个名称是类的名称。 而在实现类里面，因为会用到这个引用类的内部的实体变量和方法，所以需要使用#import来包含这个被引用类的头文件。
•       @class的作用是告诉编译器，有这么一个类的声明
•       @class还可以解决循环依赖的问题，例如A.h导入了B.h，而B.h导入了A.h，每一个头文件的编译都要让对象先编译成功才行
```


#### 52.Object-C中创建线程的方法是什么？如果在主线程中执行代码，方法是什么？如果想延时执行代码、方法又是什么？
```
线程创建有三种方法：使用NSThread创建、使用 GCD的dispatch、使用子类化的NSOperation,然后将其加入NSOperationQueue;在主线程执行代码，方法是 performSelectorOnMainThread，
如果想延时执行代码可以用
performSelector:onThread:withObject:afterDelay: 
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{ // 2秒后异步执行这里的代码... });
```
#### 53.URL缓存

![image](https://note.youdao.com/yws/public/resource/3885b57559fa21e13d008c690c9d564b/xmlnote/WEBRESOURCEec792f09de291e94865d1650784b5b56/2723)
```
AF图片缓存分析

//默认会话模式（default）：工作模式类似于原来的NSURLConnection，使用的是基于磁盘缓存的持久化策略，使用用户keychain中保存的证书进行认证授权。
+ (NSURLSessionConfiguration *)defaultSessionConfiguration;  
//瞬时会话模式（ephemeral）：该模式不使用磁盘保存任何数据。所有和会话相关的caches，证书，cookies等都被保存在RAM中，因此当程序使会话无效，这些缓存的数据就会被自动清空。
+ (NSURLSessionConfiguration *)ephemeralSessionConfiguration;  
//后台会话模式（background）：该模式在后台完成上传和下载，在创建Configuration对象的时候需要提供一个NSString类型的ID用于标识完成工作的后台会话。
+ (NSURLSessionConfiguration *)backgroundSessionConfiguration:(NSString *)identifier;

//对特定的 URL 请求使用网络协议中实现的缓存逻辑。这是默认的策略。
NSURLRequestUseProtocolCachePolicy： 
//数据需要从原始地址加载。不使用现有缓存。
NSURLRequestReloadIgnoringLocalCacheData： 
//不仅忽略本地缓存，同时也忽略代理服务器或其他中间介质目前已有的、协议允许的缓存。
NSURLRequestReloadIgnoringLocalAndRemoteCacheData：
无论缓存是否过期，先使用本地缓存数据。如果缓存中没有请求所对应的数据，那么从原始地址加载数据。
NSURLRequestReturnCacheDataElseLoad：
//无论缓存是否过期，先使用本地缓存数据。如果缓存中没有请求所对应的数据，那么放弃从原始地址加载数据,请求视为失败（即：“离线”模式）。
NSURLRequestReturnCacheDataDontLoad：
//从原始地址确认缓存数据的合法性后，缓存数据就可以使用，否则从原始地址加载。
NSURLRequestReloadRevalidatingCacheData：
```


#### 54.如何追踪App崩溃，如何解决闪退

```
当前iOS设备的App应用闪退时，操作系统会生成一个crash日志,保存在设备上。
内存报警闪退：通过Instruments工具的Allocations和Leaks模块库来发现内存分配问题和内存泄露
响应超时：当程序对特定时间响应不及时，苹果的Watchdog机制会把应用程序干掉
用户强制退出：
SEGV：（Segmentation Violation，段违例），无效内存地址，比如空指针，未初始化指针，栈溢出等；
SIGABRT：收到Abort信号，可能自身调用abort()或者收到外部发送过来的信号；
SIGBUS：总线错误。与SIGSEGV不同的是，SIGSEGV访问的是无效地址（比如虚存映射不到物理内存），而SIGBUS访问的是有效地址，但总线访问异常（比如地址对齐问题）；
SIGILL：尝试执行非法的指令，可能不被识别或者没有权限；
SIGFPE：Floating Point Error，数学计算相关问题（可能不限于浮点计算），比如除零操作；
SIGPIPE：管道另一端没有进程接手数据；
```

#### 54.APP的生命函数

```
1、application:didFinishLaunchingWithOptions：当应用程序启动时执行，应用程序启动入口，只在应用程序启动时执行一次。若用户直接启动，lauchOptions内无数据,若通过其他方式启动应用，lauchOptions包含对应方式的内容。
2、applicationWillResignActive：在应用程序将要由活动状态切换到非活动状态时候，要执行的委托调用，如 按下 home 按钮，返回主屏幕，或全屏之间切换应用程序等。
3、applicationDidEnterBackground：在应用程序已进入后台程序时，要执行的委托调用。
4、applicationWillEnterForeground：在应用程序将要进入前台时(被激活)，要执行的委托调用，刚好与applicationWillResignActive 方法相对应。
5、applicationDidBecomeActive：在应用程序已被激活后，要执行的委托调用，刚好与applicationDidEnterBackground 方法相对应。
6、applicationWillTerminate：在应用程序要完全推出的时候，要执行的委托调用，这个需要要设置UIApplicationExitsOnSuspend的键值。
初次启动：
1
2
iOS_didFinishLaunchingWithOptions
iOS_applicationDidBecomeActive
按下home键：
1
2
iOS_applicationWillResignActive
iOS_applicationDidEnterBackground
点击程序图标进入：
1
2
iOS_applicationWillEnterForeground
iOS_applicationDidBecomeActive
```


#### 55.能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

```
不能向编译后得到的类中增加实例变量；
能向运行时创建的类中添加实例变量；
因为编译后的类已经注册在 runtime 中，类结构体中的 objc_ivar_list 实例变量的链表 和 instance_size 实例变量的内存大小已经确定，同时runtime 会调用 class_setIvarLayout 或 class_setWeakIvarLayout 来处理 strong weak 引用。所以不能向存在的类中添加实例变量；
运行时创建的类是可以添加实例变量，调用 class_addIvar 函数。但是得在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前，原因同上。
```

#### 57.静态库 与 动态库

```
静态库多个项目的依赖
静态库与动态库的制作
静态库和动态库详解
Framework&&Bundle打包&&iOS SDK
```


#### 58.loadView  

```
loadView:创建View 的视图。注意需创建View或者调用[super loadView]创建view 否则在viewDidLoad 里使用view会死循环
```


#### 59.如何给imageView 添加圆角 高性能
 
```
- (UIImage*)cropImageWithRect:(CGRect)cropRect{ 
    CGRect drawRect = CGRectMake(-cropRect.origin.x , -cropRect.origin.y, self.size.width * self.scale, self.size.height * self.scale);
    UIGraphicsBeginImageContext(cropRect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextClearRect(context, CGRectMake(0, 0, cropRect.size.width, cropRect.size.height));
    [self drawInRect:drawRect];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return image;
}

@end

滚动圆角卡顿刨根问底离屏幕渲染

问UIView  CALayer的关系
这是常见的 UIView 和 CALayer 的关系：View 持有 Layer 用于显示，View 中大部分显示属性实际是从 Layer 映射而来；Layer 的 delegate 在这里是 View，当其属性改变、动画产生时，View 能够得到通知。UIView 和 CALayer 不是线程安全的，并且只能在主线程创建、访问和销毁。
```

#### 60.__block 在 ARC 和非 ARC 下含义一样吗

```
__block 在 ARC 下捕获的变量会被 block retain , 这样可能导致循环引用, 所以必须要使用弱引用才能解决该问题. 而在非 ARC 下, 可以直接使用 __block 说明符修饰变量, 因为在非 ARC 下, block 不会 retain 捕获的变量.
```






