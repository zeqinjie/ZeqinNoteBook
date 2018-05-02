## __bridge, __bridge_transfer, __bridge_retained 2015.6.5

```objc

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