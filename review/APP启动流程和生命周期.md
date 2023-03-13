# APP启动流程和生命周期
#iOS笔记/复习笔记
- - - -
## APP的启动流程
1. 加载解析APP的**info.plist**文件；
2. 创建沙盒（iOS8后，每次启动APP都会生成一个新的沙盒路径）
3. 根据info.plist的配置检查相应的权限状态；
4. 加载Mach-O文件，读取dyld路径，并运行dyld动态连接器
	1. 首先dyld会寻找合适的CPU运行环境
	2. 加载程序依赖的库和我们自己的.h.m文件编译成的.o可执行文件，并对这些库进行链接；
	3. 加载所有方法（runtime就在这个时候被初始化并完成OC的内存布局）
	4. 加载C函数
	5. 加载category的扩展（runtime会对所有类结构进行初始化）
	6. 加载C++静态函数，加载OC+load
	7. 最后dyld返回main函数地址，main函数被调用；

### app启动优化
* 减少系统的依赖库
* 减少自己需要加入的各种三方库
* 尽量用静态库，少用动态库，动态库加载比较慢，如果必须依赖动态库，则把多个非系统的动态库合并成一个动态库。
* 将不必须在`+load`方法中做的事情延迟到`+initalize`中；
* 减少项目文件中的Category，静态变量等的使用数量；
* 让UI大佬尽量把资源压缩到最小；启动加载时会加载资源图片进行IO操作，

### 冷启动、热启动
如果程序刚被运行过一次，那么程序的代码会被dyld缓存起来，因此即使杀掉进程再次重启，加载的时间也会相对快一点，如果长时间没有启动或者当前的dyld的缓存已经被其他应用占据，这次启动就会慢一些；这就是热启动和冷启动；

## APP初始化流程
1. main函数执行
2. 执行UIApplicationMain
	1. 创建UIApplication对象和delegate对象
	2. 创建MainRunLoop
	3. delegate对象开始监听系统事件
3. 根据 info.plist获取最主要的storyboard的文件名，加载storyboard；
4. 程序启动完毕的时候，调用代理`application:didFinishLaunchingWithOptions:`；
5. 显示第一个窗口；

### APP初始化流程优化
* 尽量使用纯代码而不是xib和storyboard；因为xib和storyboard还是要解析成代码来渲染页面；
* 尽量减少在`application:didFinishLaunchingWithOptions:`中代码的执行时间。能多线程多线程，能后台执行就后台执行，不要阻塞主线程；

## ViewController的生命周期
1. initWithCoder(使用storyboard) ；initWithNibName（xib或纯代码）
2. awakeFromNib（xib会调用，纯代码不会）
3. loadView
4. viewDidLoad
5. viewWillAppear
6. viewWillLayoutSubviews
7. viewDidLayoutSubviews
8. viewDidAppear
9. viewWillDisappear
10. viewDidDisappear
11. dealloc
12. didReceiveMemoryWarning（内存警告）
```objectivec
#pragma mark --- sb相关的life circle
//执行顺序1
// 当使用storyBoard时走的第一个方法。这个方法而不走initWithNibName方法。
- (instancetype)initWithCoder:(NSCoder *)aDecoder {
     NSLog(@"%s", __func__);
    if (self = [super initWithCoder:aDecoder])
     {
          //这里仅仅是创建self，还没有创建self.view所以不要在这里设置self.view相关操作
     }
    return self;
}
#pragma mark --- life circle
//执行顺序1
// 当控制器不是SB时，都走这个方法。(xib或纯代码都会走这个方法)
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil {
    NSLog(@"%s", __func__);
    if (self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil]) 
    {
        //这里仅仅是创建self，还没有创建self.view所以不要在这里设置self.view相关操作
    }
    return self;
}

//执行顺序2
// xib加载完成时调用，纯代码不会调用。系统自行调用
- (void)awakeFromNib {
    [super awakeFromNib];
     //当awakeFromNib方法被调用时，所有视图的outlet和action已经连接，但还没有被确定。
     NSLog(@"%s", __func__);
}

//执行顺序3
// 加载控制器的self.view视图。(默认从nib)
- (void)loadView {
    //该方法一般开发者不主动调用，应该由系统自行调用。
    //系统会在self.view为nil的时候调用。当控制器生命周期到达需要调用self.view的时候会自行调用。
    //或者当我们设置self.view=nil后，下次需要用到self.view时，系统发现self.view为nil，则会调用该方法。
    //该方法一般会首先根据nibName去找对应的nib文件然后加载。
    //如果nibName为空或找不到对应的nib文件，则会创建一个空视图(这种情况一般是纯代码)
    NSLog(@"%s", __func__);
    //该方法比较特殊，如果重写不能调用父类的方法[super loadView];
    self.view = [[UIView alloc] initWithFrame:[UIScreen mainScreen].bounds];
}

//执行顺序4
//视图控制器中的视图加载完成，viewController自带的view加载完成后会第一个调用的方法
- (void)viewDidLoad {
    //当self.view被创建后，会立即调用该方法。一般用于完成各种初始化操作
    NSLog(@"%s", __func__);
    [super viewDidLoad];
}

//执行顺序5
//视图将要出现
- (void)viewWillAppear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewWillAppear:animated];
}

//执行顺序6
// view 即将布局其 Subviews
- (void)viewWillLayoutSubviews {
    //view即将布局它的Subviews子视图。 当view的的属性发生了改变。
    //需要要调整view的Subviews子视图的位置，在调整之前要做的工作都可以放在该方法中实现
    NSLog(@"%s", __func__);
    [super viewWillLayoutSubviews];
}

//执行顺序7
// view 已经布局其 Subviews
- (void)viewDidLayoutSubviews {
    //view已经布局其Subviews，这里可以放置调整完成之后需要做的工作
    NSLog(@"%s", __func__);
    [super viewDidLayoutSubviews];
}

//执行顺序8
//视图已经出现
- (void)viewDidAppear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewDidAppear:animated];
}

//执行顺序9
//视图将要消失
- (void)viewWillDisappear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewWillDisappear:animated];
}

//执行顺序10
//视图已经消失
- (void)viewDidDisappear:(BOOL)animated {
    NSLog(@"%s", __func__);
    [super viewDidDisappear:animated];
}

//执行顺序11
// 视图被销毁
- (void)dealloc {
    //系统会在此时释放掉init与viewDidLoad中创建的对象
    NSLog(@"%s", __func__);
}

//执行顺序12
//出现内存警告  //模拟内存警告:点击模拟器->hardware-> Simulate Memory Warning
- (void)didReceiveMemoryWarning {
    //在内存足够的情况下，app的视图通常会一直保存在内存中，但是如果内存不够，一些没有正在显示的viewController就会收到内存不足的警告。
    //然后就会释放自己拥有的视图，以达到释放内存的目的。但是系统只会释放内存，并不会释放对象的所有权，所以通常我们需要在这里将不需要显示在内存中保留的对象释放它的所有权，将其指针置nil。
    NSLog(@"%s", __func__);
    [super didReceiveMemoryWarning];
}
```

