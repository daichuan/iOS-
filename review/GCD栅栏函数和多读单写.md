# GCD栅栏函数和多读单写
#iOS笔记/复习笔记
- - - -
## 栅栏函数
当有个需求，A,B异步请求完成之后才能请求C,D。A,B,C，D都是异步请求。这个用`dispatch_group`也可以实现，只不过比`dispatch_barrier`麻烦一点。大家可以尝试用`dispatch_group`实现一下，这样能更高的理解`dispatch_group`。还以使用`dispatch_barrie`r来实现多读单写的功能，下面来看看`dispatch_barrier`怎么来实现这个需求。

`dispatch_barrier` 作用就是相当于栅栏，栅栏前不管多少个异步都要执行完毕，才会执行栅栏后面的操作。
* `dispatch_barrier_sync`
* `dispatch_barrier_async`

### dispatch_barrier_async
```objc
    //barrier 之前的并行线程执行2s，线程执行完毕。
    //barrier 2s  barrier 后面的线程并行执行又是两秒 总共6s.
    dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        
        sleep(1);
        NSLog(@"休眠1s，执行栅栏前的任务");
    });
    
    dispatch_async(queue, ^{
       
        sleep(2);
        NSLog(@"休眠2s，执行栅栏前的任务");
    });
    
    //栅栏前所有任务执行完毕，才开始执行这个方法
    dispatch_barrier_async(queue, ^{
    
        sleep(2);
        NSLog(@"执行栅栏的任务");
    });
    NSLog(@"执行栅栏后不在一个队列方法");
    //执行完栅栏的方法，
    dispatch_async(queue, ^{
        
        sleep(1);
        NSLog(@"休眠1s，执行栅栏后的任务");
    });
  
    dispatch_async(queue, ^{
    
        sleep(2);
        NSLog(@"休眠2s，执行栅栏后的任务");
    });
```

打印结果
```
休眠1s，执行栅栏前的任务
休眠2s，执行栅栏前的任务
执行栅栏后不在一个队列方法
执行栅栏的方法
休眠1s，执行栅栏后的任务
休眠2s，执行栅栏后的任务
```

执行结果表明：
* 并行执行栅栏前的所有任务
* dispatch_barrier_async异步方法，不阻塞当前线程，所以先执行栅栏后不在一个队列方法
* 执行栅栏里任务
* 并行执行栅栏后的所有任务

### dispatch_barrier_async
```objc
    //barrier 之前的并行线程执行2s，线程执行完毕。
    //barrier 2s  barrier 后面的线程并行执行又是两秒 总共6s.
    dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);
    dispatch_async(queue, ^{
        
        sleep(1);
        NSLog(@"休眠1s，执行栅栏前的任务");
    });
    
    dispatch_async(queue, ^{
       
        sleep(2);
        NSLog(@"休眠2s，执行栅栏前的任务");
    });
    
    //栅栏前所有任务执行完毕，才开始执行这个方法
    dispatch_barrier_sync(queue, ^{
    
        sleep(2);
        NSLog(@"执行栅栏的任务");
    });
    NSLog(@"执行栅栏后不在一个队列方法");
    //执行完栅栏的方法，
    dispatch_async(queue, ^{
        
        sleep(1);
        NSLog(@"休眠1s，执行栅栏后的任务");
    });
  
    dispatch_async(queue, ^{
    
        sleep(2);
        NSLog(@"休眠2s，执行栅栏后的任务");
    });
```

执行结果
```
并行休眠1s，执行栅栏前的任务
并行休眠2s，执行栅栏前的任务
执行栅栏的方法
执行栅栏后不在一个队列方法
并行执行休眠1s，执行栅栏后的任务
并行执行休眠2s，执行栅栏后的任务
```

执行结果表明：
* 并行执行栅栏前的所有任务
* dispatch_barrier_sync同步，阻塞当前线程，所以先执行栅栏里任务
* 执行栅栏后不在一个队列方法
* 并行执行栅栏后的任务

## 多读单写
**什么是多读单写？**
* 可以同时有多个读操作，而在读操作的时候，不能有写操作。
* 在写操作的过程中，不能有其他写操作，并且在写操作之前，读操作都完成。
* 读操作是可以并发执行，写操作与（读操作、其他写操作）是互斥的。

用栅栏函数来实现，代码如下

```objc
@interface TKReadWhiteSafeDic() {
    // 定义一个并发队列
    dispatch_queue_t concurrent_queue;
    
    // 用户数据中心, 可能多个线程需要数据访问
    NSMutableDictionary *userCenterDic;
}

@end

// 多读单写模型
@implementation TKReadWhiteSafeDic

- (id)init {
    self = [super init];
    if (self) {
        // 通过宏定义 DISPATCH_QUEUE_CONCURRENT 创建一个并发队列
        concurrent_queue = dispatch_queue_create("read_write_queue", DISPATCH_QUEUE_CONCURRENT);
        // 创建数据容器
        userCenterDic = [NSMutableDictionary dictionary];
    }
    return self;
}

- (id)objectForKey:(NSString *)key {
    __block id obj;
    // 同步读取指定数据
    dispatch_sync(concurrent_queue, ^{
        obj = [userCenterDic objectForKey:key];
    });
    return obj;
}

- (void)setObject:(id)obj forKey:(NSString *)key {
    // 异步栅栏调用设置数据
    dispatch_barrier_async(concurrent_queue, ^{
        [userCenterDic setObject:obj forKey:key];
    });
}

```

多读单写的两个困惑点解释一下：
* 读操作为啥同步`dispatch_sync`
读的话通常都是直接想要结果，需要同步返回结果，如果是异步获取的话就根网络请求一样了。
* 写操作为啥异步`dispatch_barrier_async`
写操作是因为不需要等待写操作完成，所以用异步。

