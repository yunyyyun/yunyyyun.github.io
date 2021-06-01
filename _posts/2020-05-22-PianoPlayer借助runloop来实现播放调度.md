---
layout: post
title:  "PianoPlayer借助runloop来实现播放"
date:   2020-05-22 12:38:00 +0800--
categories: [iOS，钢琴]
tags: [runloop, swift]  
---

PianoPlayer（Klavier）项目起源于 4 年前，当时突然想学习钢琴，买了把电子琴，花了 3 个月，发现连和弦都学不会，感慨钢琴真是太难了。然而终究还是不想放弃，于是决定写个自动弹钢琴的app。目前上线了钢琴的基本功能，另外可以解析 musicXml格式的五线谱，并自动弹奏出来，项目地址在 首页->代码仓库可以找到。此文想简单介绍一下播放功能的实现

## 钢琴播放功能介绍
钢琴的播放功能其实蛮简单：
1. 生产端产生音集合
2. 消费端拿到音播放

其中生产端可以使手指在键盘的输入，也可以使解析五线谱拿到的时序数组，其最终都完成一个功能，在指定的时刻，将指定的音或者音的集合放入数组 _queue，消费端则从数组 _queue 拿到每个音播放。

## 第一版播放逻辑

为了做到每次能实时播放，想到用常驻线程配合来处理，其基本逻辑如下：

```
// 初始化
if (_player == nil) {
    _player = [[SoundBankPlayer alloc] init];
    _player.thread = [[NSThread alloc]initWithTarget:self selector:@selector(run) object:nil];
    [_player.thread start];
    [_player runLoop];
}
```
```
+ (void) run{
    NSLog(@"----任务1-----");
    [[NSRunLoop currentRunLoop] addPort:[NSPort port] forMode:NSDefaultRunLoopMode];
    [[NSRunLoop currentRunLoop] run];
    NSLog(@"开启 RunLoop 失败");
}

- (void) runLoop{
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        while (true) {
            if (_needPlay){
                [self performSelector:@selector(doPlayNoteQueue) onThread:self.thread withObject:nil waitUntilDone:NO];
            }
        }
    });
}
```
其中 doPlayNoteQueue 就是每次播放的函数，每次调用都会播放 _queue 里的所有音。

## 第二版播放器

第一版播放逻辑是可以实现实时播放要求的，然而运行起来后问题也很明显，cpu 稳定 100%，原因也很显而易见，因为在线程里开启了一个死循环，所以一直在消耗 cpu 的调度，然而实际情况是，大部分情况我们是不需要这么频繁的调度的，即我们需要这样一个机制，当 _queue 里有东西时，会自动取音来播放，而其他时间则不需要任何处理。这很容易让人联想到 runloop，runloop正好是这样特性的一个东西，我们只需要将播放功能注册在runloop 的状态变化 kCFRunLoopBeforeWaiting 里就行了，代码如下：
```
- (void)addRunloopObserver {
    // 1.获取当前Runloop
    CFRunLoopRef runloop = CFRunLoopGetCurrent();
      
    // 2.创建观察者
      
    // 2.0 定义上下文
    CFRunLoopObserverContext context = {
        0,
        (__bridge void *)(self),
        &CFRetain,
        &CFRelease,
        NULL
    };
      
    // 2.1 定义观察者
    static CFRunLoopObserverRef defaultModeObserver;
    // 2.2 创建观察者
    defaultModeObserver = CFRunLoopObserverCreate(kCFAllocatorDefault,
                                                  kCFRunLoopBeforeWaiting,
                                                  YES,
                                                  0,
                                                  &callBack,
                                                  &context);
     
    // 3. 给当前Runloop添加观察者
    // CFRunLoopMode mode : 设置任务执行的模式
    CFRunLoopAddObserver(runloop, defaultModeObserver, kCFRunLoopCommonModes);
      
    // C中出现 copy,retain,Create等关键字,都需要release
    CFRelease(defaultModeObserver);
}

static void callBack(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info) {
    // printf("callback---");
    SoundBankPlayer *player = (__bridge SoundBankPlayer *)info;
    [player doPlayNoteQueue];
}
```
如此，大功告成