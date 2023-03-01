# autoreleasepool
#iOS笔记/复习笔记
- - - -
## 使用形式
```objectivec
// OC
@autoreleasepool {
    // 生成自动释放对象
}

// swift
autoreleasepool {
    // 生成自动释放对象
}
```

## AutoreleasePoolPage 结构
AutoreleasePoolPage 是一个 C++ 中的类，在NSObject.mm 中的定义是这样的：

```cpp
class AutoreleasePoolPage {
    // 对当前AutoreleasePoolPage 完整性的校验
    magic_t const magic;

    // 指向下一个即将产生的autoreleased对象的存放位置（当next == begin()时，表示AutoreleasePoolPage为空；当next == end()时，表示AutoreleasePoolPage已满
    id *next;

    // 当前线程，表明与线程有对应关系
    pthread_t const thread;

    // 指向父节点，第一个节点的 parent 值为 nil；
    AutoreleasePoolPage * const parent;

    // 指向子节点，最后一个节点的 child 值为 nil；
    AutoreleasePoolPage *child;

    // 代表深度，第一个page的depth为0，往后每递增一个page，depth会加1；
    uint32_t const depth;

    // 表示high water mark（最高水位标记）
    uint32_t hiwat;
};
```

从上述的结构可以知道，其实每一个`AutoreleasePool`都是以`AutoreleasePoolPage`
为节点用**双向链表**的形式连接起来的。
每个`AutoreleasePoolPage`对象有`4096`字节的存储空间, 除了存放它自己的成员变量（56 个字节，每个占 8 个字节）外, 剩下的空间用来存储后面加入的`autorelease`对象。

![](autoreleasepool/6311050e74084023854d562afe4ff775~tplv-k3u1fbpfcp-zoom-in-crop-mark-3024-0-0-0.image.png)

### 大致流程

* 当进入`@autoreleasepool`作用域时，`objc_autoreleasePoolPush`方法被调用，runtime会向当前的`AutoreleasePoolPage`中添加一个nil对象作为哨兵对象，并返回该哨兵对象的地址；
* 对象调用`autorelease`方法，会被加入到对应的`AutoreleasePoolPage`中，`next`指针类似一个游标，不断变化，记录位置，如果加入的对象超出一页大小，便会自动添加一个新页面；
* 当离开`@autoreleasepool`作用域时，`objc_autoreleasePoolPop:(哨兵对象地址)`方法被调用，其会从当前page的next指标的上一个元素开始查找，一直找到最近的一个哨兵对象，依次向这个范围中的对象发送release消息；

因为哨兵对象的存在，自动释放池的嵌套也是满足的，不管是嵌套，还是被嵌套的自动释放池，找到自己对应的哨兵对象就行了；



[iOS探究 - autorelease 和 autoreleasepool - 简书](https://www.jianshu.com/p/97dd0ae27108)
[AutoreleasePool - 掘金](https://juejin.cn/post/7066426604892717093)
