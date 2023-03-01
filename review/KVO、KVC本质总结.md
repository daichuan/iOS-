#iOS笔记/复习笔记
---

# KVO
**KVO的全称是Key-Value Observing，俗称”键值监听”，可以用于监听摸个对象属性值得改变。**

## KVO原理
如果对象没有添加KVO监听那么它的isa指向的就是自己原来的类对象，如下图

![1434508-af5d48461719e8b4.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6889322abaf1433ea27c4d92986f14aa~tplv-k3u1fbpfcp-watermark.image?)

当一个对象添加了KVO的监听时，当前对象的isa指针指向的就不是你原来的类，指向的是另外一个类对象，如下图

![1434508-40e66a75b2c8cc8a.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ca7b2c2844e448ba23fa9820599f529~tplv-k3u1fbpfcp-watermark.image?)

**NSKVONotifying_DLPerson**
* 是 Runtime动态创建的一个类，在程序运行的过程中产生的一个新的类；
* 是DLPerson的一个子类,superclass指向了DLPerson；
* 存在自己的 setAge:、class、dealloc、isKVOA...方法；
* 扩展，isa指针指向自己的元类（NSKVONotifying_DLPerson应该有自己的元类）；

1. 添加了KVO监听的对象，runtime动态创建一个新类**NSKVONotifying_DLPerson**，**NSKVONotifying_DLPerson** 是**DLPerson**的子类，DLPerson的实例对象的isa指针指向新类；
2. 当我们**DLperson的实例对象**调用**setAge**方法时，实例对象的isa指针找到类对象，然后在类类对象中寻找相应的对象方法，也就找到了**NSKVONotifying_DLPerson**类的**setAge**方法；
3. **NSKVONotifying_DLPerson**在**setAge**中添加了一些操作
```objectivec
[self willChangeValueForKey:@"age"];
[super setAge:age];
[self didChangeValueForKey:@"age"];
```

>  总结：`didChangeValueForKey:` 内部会调用 `observer的observeValueForKeyPath:ofObject:change:context:`方法

---

# KVC
**KVC的全称key - value - coding，俗称”键值编码”,可以通过key来访问某个属性**

## setValue:forKey:的原理

![167ea605f5f2c483.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9120d4a491042a8a5338d6da75c636a~tplv-k3u1fbpfcp-watermark.image?)

`accessInstanceVariablesDirectly`用来判断是否可以直接访问成员变量
```objectivec
 + (BOOL)accessInstanceVariablesDirectly{
      return YES;   ///> 可以直接访问成员变量
  //    return NO;  ///>  不可以直接访问成员变量,  
  ///> 直接访问会报NSUnkonwKeyException错误  
  }
```

1. 当我们设置`setValue:forKey:`时
2. 首先会查找`setKey:、_setKey:` (按顺序查找)
3. 如果有直接调用
4. 如果没有，先查看`accessInstanceVariablesDirectly`方法
5. 如果可以访问会按照 `_key、_isKey、key、iskey`的顺序查找成员变量
6. 找到直接赋值
7. 未找到报错`NSUnkonwKeyException`错误

## valueForKey:的原理

![167ea61203a823f5.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/665ff2209d6a486eb269aa7830a301c5~tplv-k3u1fbpfcp-watermark.image?)

1. kvc取值按照 `getKey、key、iskey、_key` 顺序查找方法
2. 存在直接调用
3. 没找到同样，先查看`accessInstanceVariablesDirectly`方法
4. 如果可以访问会按照 `_key、_isKey、key、iskey`的顺序查找成员变量
5. 找到直接赋值
6. 未找到报错NSUnkonwKeyException错误

# 面试题
1. iOS用什么方式实现对一个对象的KVO？(KVO的本质是什么？)
> * 利用RuntimeAPI动态生成一个子类，并且让instance对象的isa指向这个全新的子类.
> * 当修改instance对象的属性时，会调用Foundation的_NSSetXXXValueAndNotify函数
>  ```
>  willChangeValueForKey:
>  父类原来的setter
>  didChangeValueForKey:
>  ```
>  * 内部会触发监听器（Oberser）的监听方法`(observeValueForKeyPath:ofObject:change:context:）`
>
2. 如何手动触发KVO？
>  手动调用`willChangeValueForKey:`和`didChangeValueForKey:`

3. 直接修改成员变量会触发KVO么？
> 不会触发KVO，因为直接修改成员变量并没有走set方法。

4. 通过KVC会触发KVO么？
> *  都会触发;
> *  KVC修改属性，属性都有set方法，因此会触发KVO；
> *  KVC修改成员变量，因为成员变量没有set方法，因此KVC是直接赋值，没有调用set方法，但是当这个成员变量被KVO监听后，通过KVC修改成员变量时会调用**willChangeValueForKey**，**didChangeValueForKey**，因此也会触发KVO;（可以在.m文件中添加这个两个方法进行验证，kvo不监听该成员变量时不会调用这两个方法，只有监听才会调用，ps必须是同一个成员变量）





参考链接
[iOS底层原理总结篇— 深入理解 KVC\KVO 实现机制](https://juejin.cn/post/6844903747680731150)
[KVC实现原理 | NeroXie的个人博客](https://www.neroxie.com/2019/07/12/KVC%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/)