### ReactNative
#### 参考
- [【React Native】从源码一步一步解析它的实现原理](https://www.jianshu.com/p/5cc61ec04b39)
- [React Native通信机制详解](http://blog.cnbang.net/tech/2698/)
- [React Native：从入门到原理](http://www.cocoachina.com/ios/20160612/16654.html)

##### 前提条件
```
动态或者脚本语言要跟本地语言互通要具备如下几点：
1.本地语言有一个runtime机制来对对象的方法调用进行动态解析。
2.本地语言一定有一个脚本的解析引擎（所以采用是JavaScriptCore）
3.建立一个脚本语言到本地语言的映射表，KEY是脚本语言认识的符号，VALUE是本地语言认识的符号。通过这个映射表来构建从脚本到本地的调用。


从 Server 获取配置 --> 解析 --> 执行native代码
利用 JSON 文件实现动态配置的效果，它的核心流程是：
通过 HTTP 请求获取 JSON 格式的配置文件。
配置文件中标记了每一个元素的属性，比如位置，颜色，图片 URL 等。
解析完 JSON 后，我们调用 Objective-C 的代码，完成 UI 控件的渲染。
```

#### 原理

```
React Native 能够运行起来，全靠 Objective-C 和 JavaScript 的交互

```
![image](http://cc.cocimg.com/api/uploads/20160612/1465703949535385.jpg)


```
初始化 React Native

RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                moduleName:@"PropertyFinder"
                initialProperties:nil
                 launchOptions:launchOptions];
                 
                 读取 JavaScript 源码

- 初始化模块信息
- 初始化 JavaScript 代码的执行器，即 RCTJSCExecutor 对象
- 生成模块列表并写入 JavaScript 端
- 执行 JavaScript 源码
```

- 初始化模块信息

```
这一步在方法 initModulesWithDispatchGroup: 中实现，主要任务是找到所有需要暴露给 JavaScript 的类。每一个需要暴露给 JavaScript 的类(也成为 Module，以下不作区分)都会标记一个宏：RCT_EXPORT_MODULE，这个宏的具体实现并不复杂：

#define RCT_EXPORT_MODULE(js_name) \
RCT_EXTERN void RCTRegisterModule(Class); \
+ (NSString *)moduleName { return @#js_name; } \
+ (void)load { RCTRegisterModule(self); }
这样，这个类在 load 方法中就会调用 RCTRegisterModule 方法注册自己：

void RCTRegisterModule(Class moduleClass)
{
  static dispatch_once_t onceToken;
  dispatch_once(&onceToken, ^{
    RCTModuleClasses = [NSMutableArray new];
  });
  [RCTModuleClasses addObject:moduleClass];
}
因此，React Native 可以通过 RCTModuleClasses 拿到所有暴露给 JavaScript 的类。下一步操作是遍历这个数组，然后生成 RCTModuleData 对象：
for (Class moduleClass in RCTGetModuleClasses()) {
    RCTModuleData *moduleData = [[RCTModuleData alloc]initWithModuleClass:moduleClass                               bridge:self];
    [moduleClassesByID addObject:moduleClass];
    [moduleDataByID addObject:moduleData];
}
可以想见，RCTModuleData 对象是模块配置表的主要组成部分。如果把模块配置表想象成一个数组，那么每一个元素就是一个 RCTModuleData 对象。

这个对象保存了 Module 的名字，常量等基本信息，最重要的属性是一个数组，保存了所有需要暴露给 JavaScript 的方法。

暴露给 JavaScript 的方法需要用 RCT_EXPORT_METHOD 这个宏来标记，它的实现原理比较复杂，有兴趣的读者可以自行阅读。简单来说，它为函数名加上了 __rct_export__ 前缀，再通过 runtime 获取类的函数列表，找出其中带有指定前缀的方法并放入数组中:

- (NSArray<id> *)methods{
    unsigned int methodCount;
    Method *methods = class_copyMethodList(object_getClass(_moduleClass), &methodCount); // 获取方法列表
    for (unsigned int i = 0; i < methodCount; i++) {
        RCTModuleMethod *moduleMethod = /* 创建 method */
        [_methods addObject:moduleMethod];
      }
    }
    return _methods;
}
因此 Objective-C 管理模块配置表的逻辑是：Bridge 持有一个数组，数组中保存了所有的模块的 RCTModuleData 对象。只要给定 ModuleId 和 MethodId 就可以唯一确定要调用的方法。

```

- 初始化 JavaScript 代码的执行器，即 RCTJSCExecutor 对象


```
通过查看源码可以看到，初始化 JavaScript 执行器的时候，addSynchronousHookWithName 这个方法被调用了多次，它其实向 JavaScript 上下文中添加了一些 Block 作为全局变量：
- (void)addSynchronousHookWithName:(NSString *)name usingBlock:(id)block {
    self.context.context[name] = block;
}

这里我们需要重点注意的是名为 nativeRequireModuleConfig 的 Block，它在 JavaScript 注册新的模块时调用：

get: () => {
    let module = RemoteModules[moduleName];
    const json = global.nativeRequireModuleConfig(moduleName); // 调用 OC 的 Block
    const config = JSON.parse(json); // 解析 json
    module = BatchedBridge.processModuleConfig(config, module.moduleID); // 注册 config
    return module;
},
**这就是模块配置表能够加载到 JavaScript 中的原理**。


另一个值得关注的 Block 叫做 nativeFlushQueueImmediate。实际上，JavaScript 除了把调用信息放到 MessageQueue 中等待 Objective-C 来取以外，也可以主动调用 Objective-C 的方法：
if (global.nativeFlushQueueImmediate &&
    now - this._lastFlush >= MIN_TIME_BETWEEN_FLUSHES_MS) {
    global.nativeFlushQueueImmediate(this._queue); // 调用 OC 的代码
}
目前，React Native 的逻辑是，如果消息队列中有等待 Objective-C 处理的逻辑，而且 Objective-C 超过 5ms 都没有来取走，那么 JavaScript 就会主动调用 Objective-C 的方法：


[self addSynchronousHookWithName:@"nativeFlushQueueImmediate" usingBlock:^(NSArray*calls){
    [self->_bridge handleBuffer:calls batchEnded:NO];
}];
这个 handleBuffer 方法是 JavaScript 调用 Objective-C 方法的关键，在下一节——方法调用中，我会详细分析它的实现原理。

一般情况下，Objective-C 会定时、主动的调用 handleBuffer 方法，这有点类似于轮询机制：


// 每个一段时间发生一次：
Objective-C：嘿，JavaScript，有没有要调用我的方法呀？
JavaScript：有的，你从 MessageQueue 里面取出来。
然而由于卡顿或某些特殊原因，Objective-C 并不能总是保证能够准时的清空 MessageQueue，这就是为什么 JavaScript 也会在一定时间后主动的调用 Objective-C 的方法。查看上面 JavaScript 的代码可以发现，这个等待时间是 5ms。


```

- 生成模块配置表并写入 JavaScript 端

```
复习一下 nativeRequireModuleConfig 这个 Block，它可以接受 ModuleName 并且生成详细的模块信息，但在前文中我们没有提到 JavaScript 是如何知道 Objective-C 要暴露哪些类的(目前只是 Objective-C 自己知道)。

这一步的操作就是为了让 JavaScript 获取所有模块的名字：

- (NSString *)moduleConfig{
    NSMutableArray*config = [NSMutableArray new];
    for (RCTModuleData *moduleData in _moduleDataByID) {
      [config addObject:@[moduleData.name]];
    }
}
查看源码可以发现，Objective-C 把 config 字符串设置成 JavaScript 的一个全局变量，名字叫做：__fbBatchedBridgeConfig。

```

- 执行 JavaScript 源码

```
这一步也没什么技术难度可以，代码已经加载进了内存，该做的配置也已经完成，只要把 JavaScript 代码运行一遍即可。

运行代码时，第三步中所说的那些 Block 就会被执行，从而向 JavaScript 端写入配置信息。

至此，JavaScript 和 Objective-C 都具备了向对方交互的能力
```
时序图以供参考

![image](http://cc.cocimg.com/api/uploads/20160612/1465706696208798.png)







