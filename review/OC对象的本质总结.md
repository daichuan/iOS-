# OC对象的本质总结
#iOS笔记/复习笔记
---

## NSObjcect
NSObjcect对象的OC代码
```objectivec
@interface NSObject <NSObject> {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wobjc-interface-ivars"
    Class isa  OBJC_ISA_AVAILABILITY;
#pragma clang diagnostic pop
}
@end
```

转换成c语言其实就是一个**结构体**
```c
struct NSObject_IMPL {
    Class isa;
};
```

**我们发现这个结构体只有一个成员，isa指针，而指针在64位架构中占8个字节。也就是说一个NSObjec对象所占用的内存是8个字节。**

一个NSObject对象占用多少内存？
* 系统分配了16个字节给NSObject对象（通过malloc_size函数获得）（一个对象最少分配16个字节）
* 但NSObject对象内部只使用了8个字节的空间（64bit环境下，可以通过class_getInstanceSize函数获得）

内存对齐原则
**原则 1. 前面的地址必须是后面的地址正数倍,不是就补齐。**
**原则 2. 整个Struct的地址必须是最大字节的整数倍。**

## OC对象
OC对象分为三种:
1. **instance对象（实例对象）**:instance对象就是通过类alloc出来的对象;
2. **class对象（类对象）**
3. **meta-class对象（元类对象）**

**instance对象**
* isa指针
* 其他成员变量

**class对象**
* isa指针
* superclass指针
* 类的属性信息（property）
* 类的对象方法信息（instance method）
* 类的协议信息（protocol）
* 类的成员变量信息（ivar）
	* 成员变量信息：存储成员变量的类型，名字等。

**成员变量的值是存储在实例对象中的，因为只有当我们创建实例对象的时候才为成员变赋值。但是成员变量叫什么名字，是什么类型，只需要有一份就可以了。所以存储在class对象中。**

**元类对象 meta-class**
* isa指针
* superclass指针
* 类的类方法信息（class method）

**meta-class对象和class对象的内存结构是一样的，所以meta-class中也有类的属性信息，类的对象方法信息等成员变量，但是其中的值可能是空的。**


![EF376435-D133-427A-99CE-90A3F8F36DF7.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd8275def87f431aa61e42a31e15119a~tplv-k3u1fbpfcp-watermark.image?)

**isa指针指向**

instance.isa -> class -> meta-class，如下图

![1434508-e71cf3850379fe21.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81cfc1b0630c4ccab1964dfbd88b7e22~tplv-k3u1fbpfcp-watermark.image?)
superclass指针指向,如下图

![1434508-c424291af118ebad.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/09b2b88ebf374420be59837ee27dee33~tplv-k3u1fbpfcp-watermark.image?)

**经典isa指向图**

![1434508-49ba7d6446b3ded2.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/544f1e1b86cc474b825b7798abc951a3~tplv-k3u1fbpfcp-watermark.image?)

**对isa、superclass总结**
* **instance.isa** -> **class**
* **class.isa** -> **meta-class**
* **meta-class.isa** -> **基类的meta-class**，
* **基类.isa** -> **基类自己**

* **instance调用对象方法的轨迹，isa找到class，方法不存在，就通过superclass找父类**
* **class调用类方法的轨迹，isa找meta-class，方法不存在，就通过superclass找父类**



参考链接
[iOS底层原理总结 - 探寻OC对象的本质 - 简书](https://www.jianshu.com/p/aa7ccadeca88)
[iOS底层原理总结—instance、class、meta-calss对象的isa和superclass](https://juejin.cn/post/6844903744165904398)