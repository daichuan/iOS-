#iOS笔记/复习笔记
---
## block本质
**Block本质上也是一个OC对象，它内部也有个isa指针**
**Block是封装了函数调用以及函数调用环境的OC对象**

Block的结构如下


![20537322-7cfd1cb75df46376.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06a446ead5cc43aea38e24661f2a04a6~tplv-k3u1fbpfcp-watermark.image?)

## blcok变量截获


![20537322-dd500520690c1bd8.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1761b1960b344bbc986db44ba395e200~tplv-k3u1fbpfcp-watermark.image?)

**1.为什么auto变量是值传递，static变量是指针传递？**
因为static变量会一直保存在内存当中，所以通过指针可以取到最新的值，而auto变量在离开方法括号“{}”作用域后就被销毁了，所以去要捕获他并把值保存起来。

**2.为什么局部变量需要捕获，全局变量不需要？**
因为局部变量需要跨函数访问，而且随时可能会被销毁所以需要提前捕获。全局变量随时都可以访问，且一直存在，所以不需要捕获。

**3.成员变量与属性在block中调用时，需要被捕获吗？**
因为调用成员变量与属性时，需要通过self来调用，而**self对block来说是局部变量**，所以是需要捕获的。

代码分析看[2019 iOS面试题-----Block原理、Block变量截获、Block的三种形式、__block - 简书](https://www.jianshu.com/p/0e1a0e7e988d)

## block类型

Block有三种，分别为：全局Block，栈Block， 堆Block；他们的存储位置不同；


![20537322-e845bca512ab10ab.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/22e0e1a3abb746f5a4970fd69861b1e7~tplv-k3u1fbpfcp-watermark.image?)

Block类型的区别

![20537322-b7a900795d34d3fe.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffa9c63576974d40b2ce9955fcee2cda~tplv-k3u1fbpfcp-watermark.image?)

堆:动态分配内存，需要程序员申请申请，也需要程序员自己管理内存（调用all申请的）
栈：程序自动分配，不可控，随时可能被销毁 （局部变量）
数据段区域：（全局变量）

因为栈中的block可能会被销毁，因此需要copy到堆上（copy栈上的block还是存在的，只是在堆中也copy了一份）；ARC环境下会自动copy；
ARC下，block用strong和copy都可以修饰；因为之前MRC的习惯，习惯性用copy修饰；


![20537322-7d0dd3a5dc5a5bcb.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c9da38869bb44cf2bba32efac8f659e7~tplv-k3u1fbpfcp-watermark.image?)

## 循环引用
如何解决强引用
**1._weak**
不会产生强引用，指向的对象销毁时，会自动让指针置为nil；


![20537322-8efbce486a8f3040.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3afd4fe667904fa59faa651cf6f8dfd9~tplv-k3u1fbpfcp-watermark.image?)

** 2.用`__block`解决 **，不推荐，使用比较麻烦，如果block不调用的话，永远不会释放，还是存在内存泄漏


![20537322-f25f0842619583aa.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03361c922bdd4801b29efaa8e4a4d90f~tplv-k3u1fbpfcp-watermark.image?)

**4.__weak与__strong搭配使用**（weak- strong- dance 强弱共舞,能保证在整个block环境使用中不会被释放，如下延时操作案例中）

```objectivec
    Student *student = [[Student alloc]init];
    student.name = @"Hello World";
    __weak typeof(student) weakSelf = student;
    
    student.study = ^{
        __strong typeof(student) strongSelf = weakSelf;

        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            NSLog(@"my name is = %@",strongSelf.name);
        });
    };
```

weakSelf 是为了block不持有self，避免Retain Circle循环引用。在 Block 内如果需要访问 self 的方法、变量，建议使用 weakSelf。
strongSelf的目的是因为一旦进入block执行，假设不允许self在这个执行过程中释放，就需要加入strongSelf。block执行完后这个strongSelf 会自动释放，没有不会存在循环引用问题。如果在 Block 内需要多次 访问 self，则需要使用 strongSelf。

## 相关面试题
### 1.block的本质
Block是封装了函数调用以及函数调用环境的OC对象
### 2.block为什么使用copy修饰
Block访问auto变量时,Block的内存地址显示在栈区,栈区的特点就是创建的对象随时可能被销毁,一旦被销毁后续再次调用空对象就可能会造成程序崩溃,在对block进行copy后,block存放在堆区.所以在使用Block属性时使用Copy修饰,而在ARC模式下,系统也会默认对Block进行copy操作
**注意**：ARC环境下使用copy和strong修饰并没有区别，ARC环境下使用strong也会自动copy
### 3.block如何捕获外部变量
* block在访问auto变量（局部变量）时，block内部会捕获到外部变量的值，后面修改外部auto变量的值，block内部的值不会随着改变而改变
* block在访问static变量（局部变量）时，block内部会捕获到外部变量的地址值，所以后面修改外部static变量值的时候，通过地址访问到的是最新修改后的值。
### 4.__block修饰为什么能修改auto变量
编译器会将__block变量包装成一个对象，会捕获到block内部，并进行指针传递，所以能够修改其值。

进阶链接：[阿里、字节：一套高效的iOS面试题之Block | 迈腾大队长](https://www.sunyazhou.com/2020/09/Block/)

[深入研究 Block 用 weakSelf、strongSelf、@weakify、@strongify 解决循环引用](https://halfrost.com/ios_block_retain_circle/)

[iOS中的Block - 简书](https://www.jianshu.com/p/1957da2e92ca)
