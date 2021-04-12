---
layout: post
title:  "SDWebimageView + UITableView 卡顿问题"
date:   2019-04-16 12:00:00 +0800--
categories: [iOS]
tags: [UITableView]  

---

## SDWebImageView + UITableView 的问题

当UITableView里cell使用SDWebimageView加载网络图片时，当UITableView不断翻页加载数据时，你会看到内存是不断涨的，原因有2个：

* decodedImageWithImage占用大量内存，通过Instruments的allocations可以看到，也可以在

[SDWebImage 的 issue](https://github.com/SDWebImage/SDWebImage/issues/538) 看到关于这个的详细讨论。对应解决方法则是在翻页的代码里加入：

```objective-c
	static int count = 0;
    NSLog(@" %d", count);
    ++count;
    if (count>3){ // 每翻2页清除一次缓存
        count = 0;
        [[SDImageCache sharedImageCache] setValue:nil forKey:@"memCache"];
    }
```

这里有个知识点，SD的内存缓存用的是 NSCache：

```objective-c
@property (strong, nonatomic, nonnull) NSCache *memCache;
```

* 图片过大，当处理太大的图片时， 会产生瞬间大内存， 解决方法则是对图片进行处理，要么服务端不要给大图，要么客户端对大图做等比的压缩：

```objective-c
if (data.length/(1024) > 128){
	NSLog(@"image_is_too_large %ld", data.length);
	image = [self compressImageWith:image];
}
```

```objective-c
+(UIImage *)compressImageWith:(UIImage *)image
{
    float imageWidth = image.size.width;
    float imageHeight = image.size.height;
    float width = 640;
    float height = image.size.height/(image.size.width/width);
    
    float widthScale = imageWidth /width;
    float heightScale = imageHeight /height;
    
    // 创建一个bitmap的context
    // 并把它设置成为当前正在使用的context
    UIGraphicsBeginImageContext(CGSizeMake(width, height));
    
    if (widthScale > heightScale) {
        [image drawInRect:CGRectMake(0, 0, imageWidth /heightScale , height)];
    }
    else {
        [image drawInRect:CGRectMake(0, 0, width , imageHeight /widthScale)];
    }
    
    // 从当前context中创建一个改变大小后的图片
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    // 使当前的context出堆栈
    UIGraphicsEndImageContext();
    
    return newImage;
}
```

并在在SDWebImageDownloaderOperation的connectionDidFinishLoading方法里加入：

```objective-c
NSData *data = UIImageJPEGRepresentation(image, 1);
self.imageData = [NSMutableData dataWithData:data];
```

另外一解决方案则是通过runloop优化，滑动时不加载数据

* VC 统一管理耗时操作的任务执行：

  ```
  [self addTasks:^{
    UIImageView *img2 = [[UIImageView alloc] initWithFrame:CGRectMake(x,
                        y,
                        w,
                        h)];
    img2.image = [UIImage imageNamed:@"cell.jpg"];
    [cell addSubview:img2];
   }];
  ```

  ```
  - (void)addTasks:(runloopBlock)task {
      // 保存新任务
      [self.tasksArr addObject:task];
      // 如果超出最大任务数 丢弃之前的任务
      if (self.tasksArr.count > _maxTaskCount) {
          [self.tasksArr removeObjectAtIndex:0];
      }
  }
  ```

* 添加runloop：

  ```
  - (void)viewDidLoad {
      [super viewDidLoad];
      
      // 可以自己设置最大任务数量(我这里是当前页面最多同时显示几张照片)
      self.maxTaskCount = 50;
      [self.view addSubview:self.tableView];
      
      // 创建定时器 (保证runloop回调函数一直在执行)
      CADisplayLink *displayLink = [CADisplayLink displayLinkWithTarget:self
  																	selector:@selector(notDoSomething)];
      [displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];
  
      //添加runloop观察者
      [self addRunloopObserver];
  
  }
  ```

  ```
  // 添加runloop观察者
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
      
      ViewController *vc = (__bridge ViewController *)info;
      
      // 无任务  退出
      if (vc.tasksArr.count == 0) return;
      
      // 从数组中取出任务
      runloopBlock block = [vc.tasksArr firstObject];
      
      // 执行任务
      if (block) {
          block();
      }
      // 执行完任务之后移除任务
      [vc.tasksArr removeObjectAtIndex:0];
      
  }
  ```

一般经过上面处理后，就不会有内存暴涨问题了，但是也都有自己的缺陷，如下：

* 清除缓存后的图得重新加载，还有何时清理缓存
* 图片压缩后会失真，图片压缩比例怎么设定
* 通过runloop判断在滑动结束时才加载图片，会导致图片有延迟加载的感觉，甚至有闪一下的感觉，

具体怎么处理还需要实际场景具体处理

## UITableView+3DTouch 滑动卡顿的一个场景

当给cell添加3d Touch重按预览时，需要调用 registerForPreviewingWithDelegate 注册viewcontroller， 然后在 viewcontroller中实现*UIViewControllerPreviewingDelegate*的2个方法即可。如果不注意的话，一般可能会直接在cellForRowAtIndexPath里面调用 registerForPreviewingWithDelegate，这样做的后果是，当viewcontroller分页加载更多数据时会发现明显的卡顿。原因是没有考虑cell的复用机制，导致每次使用cell时不断重复registerForPreviewingWithDelegate， 对应解决如下：

```objective-c
- (void)setupPreviewingDelegateWithController:(UIViewController<UIViewControllerPreviewingDelegate> *)controller {
    if (self.isAllreadySetupPreviewingDelegate == YES) {
        return;
    }
    if ([self respondsToSelector:@selector(traitCollection)]) {
        if ([self.traitCollection respondsToSelector:@selector(forceTouchCapability)]) {
            if (self.traitCollection.forceTouchCapability == UIForceTouchCapabilityAvailable) {
                [controller registerForPreviewingWithDelegate:controller sourceView:self];
                self.isAllreadySetupPreviewingDelegate = YES;
            } else {
                self.isAllreadySetupPreviewingDelegate = NO;
            }
        }
    }
}
```

cell 添加isAllreadySetupPreviewingDelegate属性来判断是否已经注册过3d Touch功能，当该属性为false，并且cell补响应forceTouchCapability方法时，才调用registerForPreviewingWithDelegate。

## SDWebimageView 与 UITableView复用

又一个常见的问题是，当list快速滑动时，SDWebimageView怎么处理重复下载的问题，**原来SDWebImage在下载图片时，第一件事就是关闭imageView当前的下载操作**：

```objective-c
NSString *validOperationKey = operationKey ?: NSStringFromClass([self class]);
[self sd_cancelImageLoadOperationWithKey:validOperationKey];
```

