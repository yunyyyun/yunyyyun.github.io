## Runloop

一个线程一次只能执行一个任务，执行完成后线程就会退出。RunLoop 机制能让线程随时处理事件但并不退出。这里说的随时是指：程序需要运行时就保持程序的持续运行，不需要的时候就进入休眠状态。

#### RunLoop 的结构

和 RunLoop 相关的主要涉及五个类：

- CFRunLoopRef：RunLoop对象
- CFRunLoopModeRef：运行模式
- CFRunLoopSourceRef：输入源/事件源
- CFRunLoopTimerRef：定时源
- CFRunLoopObserverRef：观察者

一个RunLoop 对象中可以包含多个 Mode，每个 Mode 又包含多个个 Source、Timer、Observer。

#### RunLoop 中的 Mode

关于Mode首先要知道一个RunLoop 对象中可能包含多个Mode，且每次调用 RunLoop 的主函数时，只能指定其中一个 Mode(CurrentMode)。切换 Mode，需要重新指定一个 Mode 。主要是为了分隔开不同的 Source、Timer、Observer，让它们之间互不影响。

总共是有五种Mode:

* `kCFRunLoopDefaultMode`：默认模式，主线程是在这个运行模式下运行
* `UITrackingRunLoopMode`：跟踪用户交互事件（用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他Mode影响
* `UIInitializationRunLoopMode`：在刚启动App时第进入的第一个 Mode，启动完成后就不再使用
* `GSEventReceiveRunLoopMode`：接受系统内部事件，通常用不到
* `kCFRunLoopCommonModes`：伪模式，不是一种真正的运行模式，实际是`kCFRunLoopDefaultMode` 和 `UITrackingRunLoopMode`的结合。

#### Runloop运行流程

Runloop 具体来说主要执行逻辑是这样的：

1. 通知观察者 RunLoop 已经启动。
2. 通知观察者即将要开始定时器
3. 通知观察者任何即将启动的非基于端口的源。
4. 启动任何准备好的非基于端口的源(Source0)。
5. 如果基于端口的源(Source1)准备好并处于等待状态，进入步骤9。
6. 通知观察者线程进入休眠状态。
7. 将线程置于休眠状态，知道下面的任一事件发生才唤醒线程。
   . 某一事件到达基于端口的源
   . 定时器启动。
   . RunLoop 设置的时间已经超时。
   . RunLoop 被唤醒。
8. 通知观察者线程将被唤醒。
9. 处理未处理的事件。
   .如果用户定义的定时器启动，处理定时器事件并重启RunLoop。进入步骤2。
   .如果输入源启动，传递相应的消息。
   .如果RunLoop被显示唤醒而且时间还没超时，重启RunLoop。进入步骤2
10. 通知观察者RunLoop结束。

#### 常驻线程

借助RunLoop可以实现线程后台常驻的功能：

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    self.thread = [[NSThread alloc] initWithTarget:self selector:@selector(runOne) object:nil];
    [self.thread start];
}
```

```objective-c
- (void) runOne{
    NSLog(@"----任务1-----");
    // 下面两句代码可以实现线程保活
    [[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
    [[NSRunLoop currentRunLoop] run];
    // 测试是否开启了RunLoop，如果开启RunLoop，则来不了这里，因为RunLoop开启了循环。
    NSLog(@"未开启RunLoop");
}
```

```objective-c
- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    // 利用performSelector，在self.thread的线程中调用run2方法执行任务
    [self performSelector:@selector(runTwo) onThread:self.thread withObject:nil waitUntilDone:NO];
}

- (void) runTwo{
    NSLog(@"----任务2------");
}
```

## AutoreleasePool

应用程序一旦启动，主线程 RunLoop 里注册了两个 Observer。

一个 Observer 监听即将进入Loop事件，回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池，并保证创建释放池发生在其他所有回调之前。_

_另外一个 Observer 监视了两个事件(RunLoop即将进入休眠和即将退出 RunLoop 事件) ，前者会调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；后者会调用 _objc_autoreleasePoolPop() 来释放自动释放池，并保证释放自动释放池事件发生在其它回调之后。

#### 实现原理

- 一个线程的自动释放池是一个指针堆栈
- 每一个指针或者指向被释放的对象，或者是自动释放池的POOL_BOUNDARY，POOL_BOUNDARY 是自动释放池的边界。
- 一个池子的 token 是指向池子 POOL_BOUNDARY 的指针。当池子被出栈的时候，每一个高于标准的对象都会被释放掉。
- 堆栈被分成一个页面的双向链表。页面按照需要添加或者删除。
- 本地线程存放着指向当前页的指针，在这里存放着新创建的自动释放的对象。

自动释放池是由 AutoreleasePoolPage 以双向链表的方式实现的，当对象调用 autorelease 方法时，会将对象加入 AutoreleasePoolPage 的栈中，调用 AutoreleasePoolPage::pop 方法会向栈中的对象发送 release 消息.

#### 使用场景

通常情况下，我们是不需要手动添加 autoreleasepool 的，使用线程自动维护的 autoreleasepool 就好了。仅在下列三种情况下需要我们手动添加 autoreleasepool：

1. 如果你编写的程序不是基于 UI 框架的，比如说命令行工具；
2. 如果你编写的循环中创建了大量的临时对象；
3. 如果你创建了一个辅助线程。

例子：

```objective-c
int lagerNum = 1024 * 1024 * 2 ;
    for(int i = 0 ; i < lagerNum; i++)
    {
        @autoreleasepool{
            NSString *str = [NSString stringWithFormat:@"Hello"];
            str = [str uppercaseString];
            str = [NSString stringWithFormat:@"%@ - %@   %d",str, @"World!", i];
            // NSLog(@"%@", str);
        }
    }
```

在我的钢琴app：[klaver](https://gitlab.com/yunyyyun/Klavier#)中也加入了常驻线程来处理琴谱解析的问题，由于对于这个app琴谱解析是常用的功能，为了避免每次切换下一曲时重新创建线程的开销而使用常驻线程。