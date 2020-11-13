# RunLoop从源码到应用全面解析



## 写在前面

> 由于文章比较长，简书没有目录，读起来不方便。建议看有目录版[RunLoop从源码到应用全面解析——带目录版](https://link.jianshu.com/?t=http%3A%2F%2Fweslyxl.coding.me%2F2018%2F03%2F18%2F2018%2F3%2FRunLoop%E4%BB%8E%E6%BA%90%E7%A0%81%E5%88%B0%E5%BA%94%E7%94%A8%E5%85%A8%E9%9D%A2%E8%A7%A3%E6%9E%90%2F)

> 在开始之前有必要重点说明一下：现在开发中用到的所有的内容其实早在官方文档及相关API中已经介绍过，**不要认为大神有多牛逼，他只是比你先阅读官方文档及使用维基百科，StackOverFlow而已。**所以在你开始研究某个知识点之前看这些东西远比去读别人消化过得有用得多。一句话**学会如何学习才是核心竞争力的关键。**

[RunLoop官方介绍](https://link.jianshu.com/?t=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Fcontent%2Fdocumentation%2FCocoa%2FConceptual%2FMultithreading%2FRunLoopManagement%2FRunLoopManagement.html%23%2Fapple_ref%2Fdoc%2Fuid%2F10000057i-CH16-SW23)

## 理解RunLoop

[深入理解RunLoop](https://blog.ibireme.com/2015/05/18/runloop/)这是国内对runloop写得非常非常好的文章。本文除了源码分析部分，也很多地方参考了其中的内容。非常感谢作者！

### 从Event Loop开始谈起

RunLoop源码在CoreFoundation里面，下载地址如下：

- [CFRunLoopRef 源码](https://link.jianshu.com/?t=http%3A%2F%2Fopensource.apple.com%2Fsource%2FCF%2FCF-855.17%2FCFRunLoop.c)
- [CF 源码](https://link.jianshu.com/?t=http%3A%2F%2Fopensource.apple.com%2Ftarballs%2FCF%2F)
- [Swift-Corelibs-foundation](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fapple%2Fswift-corelibs-foundation%2F)

一般情况下，一个线程执行完之后就会停止。为了保证线程能随时处理事件并不退出，于是最简单的想到就是一个for循环。

```c
function loop() {
    initialize();
    do {
        var message = get_next_message();
        process_message(message);
    } while (message != quit);
}
```

上面这种模型叫做事件循环，**实现这种模型的关键点就是如何在没有消息到来的情况下休眠以避免系统资源的占有，消息一到来立刻恢复。**

Runloop就是用来处理上面提到的事件及消息，执行上面的Event Loop模型。线程执行了这个函数之后就会一直处于这个函数内部，**接受消息——》等待消息——》处理消息**的循环中。直到循环结束（传入quit），然后函数返回。

### Runloop内部数据结构

在进行RunLoop的执行过程分析之前，需要熟悉内部的各个数据结构。在 CoreFoundation 里面关于 RunLoop 有5个类，这几个类都是结构体指针，对应后面会讲到的结构体。

- CFRunLoopRef：对外部暴露的对象，外界通过CFRunLoopRef的接口来管理整个Runloop。
- CFRunLoopModeRef：
- CFRunLoopSourceRef
- CFRunLoopTimerRef
- CFRunLoopObserverRef

他们之间的关系（一对多模式）可以如下图（来源于网上）所示，后面会更加详细的说明这种关系。

![img](https://upload-images.jianshu.io/upload_images/664334-7821780c94518f6e..png?)



**接下来通过源码来了解其中数据结构，也是对上面这种图的进一步解释。**

#### CFRunLoop

对应CFRunLoopRef，其数据结构如下。

```c
struct __CFRunLoop {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;                /* locked for accessing mode list */
    __CFPort _wakeUpPort;                 // used for CFRunLoopWakeUp 
    Boolean _unused;
    volatile _per_run_data *_perRunData;  // reset for runs of the run loop
    pthread_t _pthread;                   //runloop对应的线程
    uint32_t _winthread;
    CFMutableSetRef _commonModes;         //存储的是字符串，记录所有标记为common的mode
    CFMutableSetRef _commonModeItems;     //存储所有commonMode的item(source、timer、observer)
    CFRunLoopModeRef _currentMode;        //当前运行的mode
    CFMutableSetRef _modes;               //存储的是CFRunLoopModeRef，
    struct _block_item *_blocks_head;
    struct _block_item *_blocks_tail;
    CFAbsoluteTime _runTime;
    CFAbsoluteTime _sleepTime;
    CFTypeRef _counterpart;
};
```

从上面可以看出一个RunLoop包含一个线程，**也就是和线程是一一对应的**；以及若干个Mode、若干个commonModeItem，还有一个当前运行的CurrentMode。**如果在RunLoop中需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。**

> 重点说一下modes，在主线程的runloop中存在很多model，但是runloop在一个时间点只能在其中一种model下。

下图是来至[维基百科中对runloop中的Model说明](https://link.jianshu.com/?t=http%3A%2F%2Fiphonedevwiki.net%2Findex.php%2FCFRunLoop)：

![img](https://upload-images.jianshu.io/upload_images/664334-d5544ebe487d47c9.jpg?)

> 但是我在模拟器中测试下只看到了UITrackingRunLoopMode、GSEventReceiveRunLoopMode、kCFRunLoopDefaultMode、kCFRunLoopCommonMode四种Model。

#### CFRunLoopMode

对应CFRunLoopModeRef，结构如下

```c
struct __CFRunLoopMode {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;        /* must have the run loop locked before locking this */
    CFStringRef _name;            //mode名称
    Boolean _stopped;             //mode是否被终止
    char _padding[3];
    
    //几种事件，下面这四个字段，在苹果官方文档里面称为Item。runloop中有个commomitems字段，里面就是保持的下面这些内容。
    CFMutableSetRef _sources0;    //sources0
    CFMutableSetRef _sources1;    //sources1
    CFMutableArrayRef _observers; //观察者
    CFMutableArrayRef _timers;    //定时器
    
    CFMutableDictionaryRef _portToV1SourceMap; //字典  key是mach_port_t，value是CFRunLoopSourceRef
    __CFPortSet _portSet; //保存所有需要监听的port，比如_wakeUpPort，_timerPort都保存在这个数组中
    
    CFIndex _observerMask;
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    dispatch_source_t _timerSource;
    dispatch_queue_t _queue;
    Boolean _timerFired; // set to true by the source when a timer has fired
    Boolean _dispatchTimerArmed;
#endif
#if USE_MK_TIMER_TOO
    mach_port_t _timerPort;
    Boolean _mkTimerArmed;
#endif
#if DEPLOYMENT_TARGET_WINDOWS
    DWORD _msgQMask;
    void (*_msgPump)(void);
#endif
    uint64_t _timerSoftDeadline; /* TSR */
    uint64_t _timerHardDeadline; /* TSR */
};
```

从上可以看出一个CFRunLoopMode对象有一个name，若干source0、source1、timer、observer和若干port，其中source，timer，observer 数据结构被统称为 mode item。**上面提到的那几种model（UITrackingRunLoopMode、GSEventReceiveRunLoopMode、kCFRunLoopDefaultMode、kCFRunLoopCommonMode），其实就是这里的name。**

只能通过 mode的name字段（**也就是字符串，前面提到的kCFRunLoopDefaultMode 和 UITrackingRunLoopMode**） 操作内部的 Mode，当你传入一个新的 mode name 但 RunLoop 内部没有对应 mode 时，RunLoop会自动帮你创建对应的 CFRunLoopModeRef。对于一个 RunLoop 来说，其内部的 mode 只能增加不能删除。

source、timer、observer可以在多个model中注册，**但是只有runloop当前的currentMode下的source、timer、observer才可以运行。**

Model暴露给外面管理model Item的接口：

```c
CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFStringRef modeName);
CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFStringRef modeName);
CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFStringRef mode);
```

#### CFRunLoopSource

对应CFRunLoopModeRef，其结构如下

```c
struct __CFRunLoopSource {
    CFRuntimeBase _base;
    uint32_t _bits;         //用于标记Signaled状态，source0只有在被标记为Signaled状态，才会被处理
    pthread_mutex_t _lock;
    CFIndex _order;         /* immutable */
    CFMutableBagRef _runLoops;
    union {
        CFRunLoopSourceContext version0;     /* immutable, except invalidation */
        CFRunLoopSourceContext1 version1;    /* immutable, except invalidation */
    } _context;
};
```

Source分为Source、Observer、Timer三种，他们统称为modeItem。

__CFRunLoopSource是事件产生的地方。Source有两个版本：Source0 和 Source1。

- source0 只包含了一个回调（函数指针），source0是需要手动触发的Source，**它并不能主动触发事件，必须要先把它标记为signal状态。使用时，你需要先调用 CFRunLoopSourceSignal(source)，将这个 Source 标记为待处理，也就是通过`uint32_t _bits`来实现的**，只有_bits标记Signaled状态才会被处理。然后手动调用 CFRunLoopWakeUp(runloop) 来唤醒 RunLoop，让其处理这个事件。
- source1 包含了一个 mach_port 和一个回调（函数指针），**被用于通过内核和其他线程相互发送消息。这种 Source 能主动唤醒 RunLoop 的线程**。简单来说就是更加偏向于底层。

以下是source0的结构体：

```c
typedef struct {
    CFIndex version;
    void *  info;
    const void *(*retain)(const void *info);
    void    (*release)(const void *info);
    CFStringRef (*copyDescription)(const void *info);
    Boolean (*equal)(const void *info1, const void *info2);
    CFHashCode  (*hash)(const void *info);
    void    (*schedule)(void *info, CFRunLoopRef rl, CFStringRef mode);//当source加入到model触发的回调
    void    (*cancel)(void *info, CFRunLoopRef rl, CFStringRef mode);//当source从runloop中移除时触发的回调
    void    (*perform)(void *info);//当source事件被触发时的回调，使用CFRunLoopSourceSignal方式触发。
} CFRunLoopSourceContext;
```

以下是source1的结构体：

```c
typedef struct {
    CFIndex version;
    void *  info;
    const void *(*retain)(const void *info);
    void    (*release)(const void *info);
    CFStringRef (*copyDescription)(const void *info);
    Boolean (*equal)(const void *info1, const void *info2);
    CFHashCode  (*hash)(const void *info);
#if (TARGET_OS_MAC && !(TARGET_OS_EMBEDDED || TARGET_OS_IPHONE)) || (TARGET_OS_EMBEDDED || TARGET_OS_IPHONE)
    mach_port_t (*getPort)(void *info);//当source被添加到mode中的时候，从这个函数中获得具体mach_port_t。
    void *  (*perform)(void *msg, CFIndex size, CFAllocatorRef allocator, void *info);
#else
    void *  (*getPort)(void *info);
    void    (*perform)(void *info);
#endif
} CFRunLoopSourceContext1;
```

> source1除了多个了getPort。其余的字段含义和source0相同。作用就是当source被添加到mode中的时候，从这个函数中获得具体mach_port_t。

#### CFRunLoopTimer

对应RunLoopTimerRef，结构如下：

```c
struct __CFRunLoopTimer {
    CFRuntimeBase _base;
    uint16_t _bits;                  //标记fire状态
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;           //添加该timer的runloop
    CFMutableSetRef _rlModes;        //存放所有包含该timer的 mode的 modeName，意味着一个timer可能会在多个mode中存在
    CFAbsoluteTime _nextFireDate;
    CFTimeInterval _interval;        //理想时间间隔  /* immutable */
    CFTimeInterval _tolerance;       //时间偏差      /* mutable */
    uint64_t _fireTSR;               /* TSR units */
    CFIndex _order;                  /* immutable */
    CFRunLoopTimerCallBack _callout; /* immutable */
    CFRunLoopTimerContext _context;  /* immutable, except invalidation */
};
```

它和 NSTimer 是toll-free bridged 的（[资料可以看这里](https://link.jianshu.com/?t=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Fmac%2Fdocumentation%2FCoreFoundation%2FConceptual%2FCFDesignConcepts%2FArticles%2FtollFreeBridgedTypes.html%23%2F%2Fapple_ref%2Fdoc%2Fuid%2FTP40010677)），可以混用。其包含一个时间长度和一个回调（函数指针）。当其加入到 RunLoop 时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

> 根据上面的分析t一个timer可能会在多个mode中存在。

#### CFRunLoopObserver

对应CFRunLoopObserverRef，结构如下

```c
struct __CFRunLoopObserver {
    CFRuntimeBase _base;
    pthread_mutex_t _lock;
    CFRunLoopRef _runLoop;
    CFIndex _rlCount;
    CFOptionFlags _activities;           /* immutable */
    CFIndex _order;                      /* immutable */
    CFRunLoopObserverCallBack _callout;  /* immutable  设置回调函数*/
    CFRunLoopObserverContext _context;   /* immutable, except invalidation */
};
```

CFRunLoopObserver是观察者，可以观察RunLoop的各种状态，每个 Observer 都包含了一个回调（也就是上面的CFRunLoopObserverCallBack函数指针），当 RunLoop 的状态发生变化时，观察者就能通过回调接受到这个变化。状态定义在_CF_OPTIONS：

```c
/* Run Loop Observer Activities */
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),         //即将进入run loop
    kCFRunLoopBeforeTimers = (1UL << 1),  //即将处理timer
    kCFRunLoopBeforeSources = (1UL << 2), //即将处理source
    kCFRunLoopBeforeWaiting = (1UL << 5), //即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),  //被唤醒但是还没开始处理事件
    kCFRunLoopExit = (1UL << 7),          //run loop已经退出
    kCFRunLoopAllActivities = 0x0FFFFFFFU
};
```

下面是回调函数的原型:

```c
typedef void (*CFRunLoopObserverCallBack)(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info);
```

#### 小结

根据上面的数据结构，总结出如下内容。

一个model中有多个item，这些item由source、observe、timer组成。对于我们来讲用的最多的应该是observe和timer，常常通过回调来得知当前runloop的状态，进行来优化应用程序（比如监控在waiting状态下，这个时候做一些优化的事情）。其次设置定时器执行定时任务也是很常见的。

一个runloop包含了多个model，但是runloop在一个时间点只会处于一种model（kCFRunLoopDefaultMode、UITrackingRunLoopMode、）状态下也即是currentModel，**如果该当前应用状态在另一种mode下，则该model下的item（source、observe、timer）就不会工作**。

runloop其中有一个commomModes的数组，里面保存的是被标记为common的model。这种标记为common的model有种特性，那就是当 RunLoop 的内容发生变化时，RunLoop 都会自动将 commonModeItems 里的 Source/Observer/Timer 同步到具有 “Common” 标记的所有Model里。**可以这样理解，runloop中的_commonModeItems由被标记为common的model下得各个item（source、observe、timer）组成。**

通过上面三点来理解为什么NSTimer在添加到runloop中的时候要用NSRunLoopCommonModes作为参数了。这里简单说一下。

> **首先Runloop初始化的时候的会把名字为kCFRunLoopDefaultMode、UITrackingRunLoopMode的model加入到common modesls数组里面，标记为common mode**，上面提到过common mode的特性。NSTimer对应于上面提到的model中的item中的timer，并且NSTimer创建好之后默认是加入名为kCFRunLoopDefaultMode的model中，所以只有应用程序在kCFRunLoopDefaultMode下，Timer才会工作。

> 如果这个时候滑动了屏幕，那么应用的mode就从名字为kCFRunLoopDefaultMode的mode中退出，进入到名称为UITrackingRunLoopMode的model中。因为NSTimer没有添加到名字为UITrackingRunLoopMode的item中，所以只要等待不再滑动回到kCFRunLoopDefaultMode的时候才再次开始工作。

> 怎么解决呢？也就是把Timer加到了UITrackingRunLoopMode的Item。怎么样加到UITrackingRunLoopMode中呢？通过NSRunLoopCommonModes标记之后，Runloop会把NSTimer加入到Runloop中的commonModeItems中。上面讲过RunLoop 都会自动将 commonModeItems 里的 Source/Observer/Timer 同步到具有 “Common” 标记的所有Model里。

### RunLoop内部逻辑

上面的介绍Source/Timer/Observer 数据结构被统称为 mode item，一个 item 可以被同时加入多个 mode。但一个 item 被重复加入同一个 mode 时是不会有效果的。如果一个 mode 中一个 item 都没有，则 RunLoop 会直接退出，不进入循环。

根据上面讲的数据结构可以得出下面这张图：
[图片上传失败...(image-4539dd-1521365090788)]

NSRunloop暴露给外界的接口

![img](https://upload-images.jianshu.io/upload_images/664334-67e2afcf399dc4b1.jpg?)

### Runloop执行过程

#### RunLoopRun的启动

首先总的来说逻辑如下图所示：



![img](https://upload-images.jianshu.io/upload_images/664334-7fa0bbc318371433..png?)



> 下面提供了两套代码，简化版与非简化版，如果觉得非简化版代码太长可以直接看简化版的代码，他们所表达的含义是一样的。





##### 简化版代码（来源于网上）

```c
/// 用DefaultMode启动
void CFRunLoopRun(void) {
    CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
}
 
/// 用指定的Mode启动，允许设置RunLoop超时时间
int CFRunLoopRunInMode(CFStringRef modeName, CFTimeInterval seconds, Boolean stopAfterHandle) {
    return CFRunLoopRunSpecific(CFRunLoopGetCurrent(), modeName, seconds, returnAfterSourceHandled);
}
 
/// RunLoop的实现
int CFRunLoopRunSpecific(runloop, modeName, seconds, stopAfterHandle) {
    
    /// 首先根据modeName找到对应mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(runloop, modeName, false);
    /// 如果mode里没有source/timer/observer, 直接返回。
    if (__CFRunLoopModeIsEmpty(currentMode)) return;
    
    /// 1. 通知 Observers: RunLoop 即将进入 loop。
    __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopEntry);
    
    /// 内部函数，进入loop
    __CFRunLoopRun(runloop, currentMode, seconds, returnAfterSourceHandled) {
        
        Boolean sourceHandledThisLoop = NO;
        int retVal = 0;
        do {
 
            /// 2. 通知 Observers: RunLoop 即将触发 Timer 回调。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeTimers);
            /// 3. 通知 Observers: RunLoop 即将触发 Source0 (非port) 回调。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeSources);
            /// 执行被加入的block
            __CFRunLoopDoBlocks(runloop, currentMode);
            
            /// 4. RunLoop 触发 Source0 (非port) 回调。
            sourceHandledThisLoop = __CFRunLoopDoSources0(runloop, currentMode, stopAfterHandle);
            /// 执行被加入的block
            __CFRunLoopDoBlocks(runloop, currentMode);
 
            /// 5. 如果有 Source1 (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
            if (__Source0DidDispatchPortLastTime) {
                Boolean hasMsg = __CFRunLoopServiceMachPort(dispatchPort, &msg)
                if (hasMsg) goto handle_msg;
            }
            
            /// 通知 Observers: RunLoop 的线程即将进入休眠(sleep)。
            if (!sourceHandledThisLoop) {
                __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopBeforeWaiting);
            }
            
            /// 7. 调用 mach_msg 等待接受 mach_port 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
            /// • 一个基于 port 的Source 的事件。
            /// • 一个 Timer 到时间了
            /// • RunLoop 自身的超时时间到了
            /// • 被其他什么调用者手动唤醒
            __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort) {
                mach_msg(msg, MACH_RCV_MSG, port); // thread wait for receive msg
            }
 
            /// 8. 通知 Observers: RunLoop 的线程刚刚被唤醒了。
            __CFRunLoopDoObservers(runloop, currentMode, kCFRunLoopAfterWaiting);
            
            /// 收到消息，处理消息。
            handle_msg:
 
            /// 9.1 如果一个 Timer 到时间了，触发这个Timer的回调。
            if (msg_is_timer) {
                __CFRunLoopDoTimers(runloop, currentMode, mach_absolute_time())
            } 
 
            /// 9.2 如果有dispatch到main_queue的block，执行block。
            else if (msg_is_dispatch) {
                __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
            } 
 
            /// 9.3 如果一个 Source1 (基于port) 发出事件了，处理这个事件
            else {
                CFRunLoopSourceRef source1 = __CFRunLoopModeFindSourceForMachPort(runloop, currentMode, livePort);
                sourceHandledThisLoop = __CFRunLoopDoSource1(runloop, currentMode, source1, msg);
                if (sourceHandledThisLoop) {
                    mach_msg(reply, MACH_SEND_MSG, reply);
                }
            }
            
            /// 执行加入到Loop的block
            __CFRunLoopDoBlocks(runloop, currentMode);
            
 
            if (sourceHandledThisLoop && stopAfterHandle) {
                /// 进入loop时参数说处理完事件就返回。
                retVal = kCFRunLoopRunHandledSource;
            } else if (timeout) {
                /// 超出传入参数标记的超时时间了
                retVal = kCFRunLoopRunTimedOut;
            } else if (__CFRunLoopIsStopped(runloop)) {
                /// 被外部调用者强制停止了
                retVal = kCFRunLoopRunStopped;
            } else if (__CFRunLoopModeIsEmpty(runloop, currentMode)) {
                /// source/timer/observer一个都没有了
                retVal = kCFRunLoopRunFinished;
            }
            
            /// 如果没超时，mode里没空，loop也没被停止，那继续loop。
        } while (retVal == 0);
    }
    
    /// 10. 通知 Observers: RunLoop 即将退出。
    __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
}
```

上面的过程是对刚才那种图的具体说明。在整个循环中，通过observer向外界告知当前runloop的状态，事件触发由source(source0，source1）及timer发起，并且通过之前设置的函数进行回调处理，在处理各个回调的时候也触发了block的处理。



##### 非简化版代码（于CF-855.17）

下面的源码来至于CF-855.17中

非简化版代码非常长，要非常有耐心才能看下去。

###### CFRunLoopRun（入口函数）

```c
//默认运行runloop的kCFRunLoopDefaultMode
void CFRunLoopRun(void) {   /* DOES CALLOUT */
    int32_t result;
    do {
        //默认在kCFRunLoopDefaultMode下运行runloop
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}
```

###### CFRunLoopRunInMode

```c
SInt32 CFRunLoopRunInMode(CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) {     /* DOES CALLOUT */
    CHECK_FOR_FORK();
    return CFRunLoopRunSpecific(CFRunLoopGetCurrent(), modeName, seconds, returnAfterSourceHandled);
}
```

###### CFRunLoopRunSpecific

```c
/*
 * 指定mode运行runloop
 * @param rl 当前运行的runloop
 * @param modeName 需要运行的mode的name
 * @param seconds  runloop的超时时间
 * @param returnAfterSourceHandled 是否处理完事件就返回
 */
SInt32 CFRunLoopRunSpecific(CFRunLoopRef rl, CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) {     /* DOES CALLOUT */
    CHECK_FOR_FORK();
    if (__CFRunLoopIsDeallocating(rl)) return kCFRunLoopRunFinished;
    __CFRunLoopLock(rl);
    //根据modeName找到本次运行的mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(rl, modeName, false);
    //如果没找到 || mode中没有注册任何事件，则就此停止，不进入循环
    if (NULL == currentMode || __CFRunLoopModeIsEmpty(rl, currentMode, rl->_currentMode)) {
        Boolean did = false;
        if (currentMode) __CFRunLoopModeUnlock(currentMode);
        __CFRunLoopUnlock(rl);
        return did ? kCFRunLoopRunHandledSource : kCFRunLoopRunFinished;
    }
    volatile _per_run_data *previousPerRun = __CFRunLoopPushPerRunData(rl);
    //取上一次运行的mode
    CFRunLoopModeRef previousMode = rl->_currentMode;
    //如果本次mode和上次的mode一致
    rl->_currentMode = currentMode;
    //初始化一个result为kCFRunLoopRunFinished
    int32_t result = kCFRunLoopRunFinished;
    
    // 1.通知observer即将进入runloop
    if (currentMode->_observerMask & kCFRunLoopEntry ) __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopEntry);
    result = __CFRunLoopRun(rl, currentMode, seconds, returnAfterSourceHandled, previousMode);
    //10.通知observer已退出runloop
    if (currentMode->_observerMask & kCFRunLoopExit ) __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
    
    __CFRunLoopModeUnlock(currentMode);
    __CFRunLoopPopPerRunData(rl, previousPerRun);
    rl->_currentMode = previousMode;
    __CFRunLoopUnlock(rl);
    return result;
}
```

- 如果指定了一个不存在的mode来运行RunLoop，那么会失败，mode不会被创建，所以这里传入的mode必须是存在的。
- 如果指定了一个mode，但是这个mode中不包含任何modeItem，那么RunLoop也不会运行，所以必须要传入至少包含一个modeItem的mode。
- 在进入run loop之前通知observer，状态为kCFRunLoopEntry。
- 在退出run loop之后通知observer，状态为kCFRunLoopExit。

###### __CFRunloopRun（核心！！）

__CFRunloopRun才是最为重要的一步，runloop启动操作最终会执行这个方法。上面提到的简化版主要是对下面这个方法简化。

```c
/**
 *  运行run loop
 *
 *  @param rl              运行的RunLoop对象
 *  @param rlm             运行的mode
 *  @param seconds         run loop超时时间
 *  @param stopAfterHandle true:run loop处理完事件就退出  false:一直运行直到超时或者被手动终止
 *  @param previousMode    上一次运行的mode
 *
 *  @return 返回4种状态
 */
static int32_t __CFRunLoopRun(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFTimeInterval seconds, Boolean stopAfterHandle, CFRunLoopModeRef previousMode) {
    //获取系统启动后的CPU运行时间，用于控制超时时间
    uint64_t startTSR = mach_absolute_time();
    
    //如果RunLoop或者mode是stop状态，则直接return，不进入循环
    if (__CFRunLoopIsStopped(rl)) {
        __CFRunLoopUnsetStopped(rl);
        return kCFRunLoopRunStopped;
    } else if (rlm->_stopped) {
        rlm->_stopped = false;
        return kCFRunLoopRunStopped;
    }
    
    //mach端口，在内核中，消息在端口之间传递。 初始为0
    mach_port_name_t dispatchPort = MACH_PORT_NULL;
    //判断是否为主线程
    Boolean libdispatchQSafe = pthread_main_np() && ((HANDLE_DISPATCH_ON_BASE_INVOCATION_ONLY && NULL == previousMode) || (!HANDLE_DISPATCH_ON_BASE_INVOCATION_ONLY && 0 == _CFGetTSD(__CFTSDKeyIsInGCDMainQ)));
    //如果在主线程 && runloop是主线程的runloop && 该mode是commonMode，则给mach端口赋值为主线程收发消息的端口
    if (libdispatchQSafe && (CFRunLoopGetMain() == rl) && CFSetContainsValue(rl->_commonModes, rlm->_name)) dispatchPort = _dispatch_get_main_queue_port_4CF();
    
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    mach_port_name_t modeQueuePort = MACH_PORT_NULL;
    if (rlm->_queue) {
        //mode赋值为dispatch端口_dispatch_runloop_root_queue_perform_4CF
        modeQueuePort = _dispatch_runloop_root_queue_get_port_4CF(rlm->_queue);
        if (!modeQueuePort) {
            CRASH("Unable to get port for run loop mode queue (%d)", -1);
        }
    }
#endif
    
    //GCD管理的定时器，用于实现runloop超时机制
    dispatch_source_t timeout_timer = NULL;
    struct __timeout_context *timeout_context = (struct __timeout_context *)malloc(sizeof(*timeout_context));
    
    //立即超时
    if (seconds <= 0.0) { // instant timeout
        seconds = 0.0;
        timeout_context->termTSR = 0ULL;
    }
    //seconds为超时时间，超时时执行__CFRunLoopTimeout函数
    else if (seconds <= TIMER_INTERVAL_LIMIT) {
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, DISPATCH_QUEUE_OVERCOMMIT);
        timeout_timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
        dispatch_retain(timeout_timer);
        timeout_context->ds = timeout_timer;
        timeout_context->rl = (CFRunLoopRef)CFRetain(rl);
        timeout_context->termTSR = startTSR + __CFTimeIntervalToTSR(seconds);
        dispatch_set_context(timeout_timer, timeout_context); // source gets ownership of context
        dispatch_source_set_event_handler_f(timeout_timer, __CFRunLoopTimeout);
        dispatch_source_set_cancel_handler_f(timeout_timer, __CFRunLoopTimeoutCancel);
        uint64_t ns_at = (uint64_t)((__CFTSRToTimeInterval(startTSR) + seconds) * 1000000000ULL);
        dispatch_source_set_timer(timeout_timer, dispatch_time(1, ns_at), DISPATCH_TIME_FOREVER, 1000ULL);
        dispatch_resume(timeout_timer);
    }
    //永不超时
    else { // infinite timeout
        seconds = 9999999999.0;
        timeout_context->termTSR = UINT64_MAX;
    }
    
    //标志位默认为true
    Boolean didDispatchPortLastTime = true;
    //记录最后runloop状态，用于return
    int32_t retVal = 0;
    do {
        //初始化一个存放内核消息的缓冲池
        uint8_t msg_buffer[3 * 1024];
#if DEPLOYMENT_TARGET_MACOSX || DEPLOYMENT_TARGET_EMBEDDED || DEPLOYMENT_TARGET_EMBEDDED_MINI
        mach_msg_header_t *msg = NULL;
        mach_port_t livePort = MACH_PORT_NULL;
#elif DEPLOYMENT_TARGET_WINDOWS
        HANDLE livePort = NULL;
        Boolean windowsMessageReceived = false;
#endif
        //取所有需要监听的port
        __CFPortSet waitSet = rlm->_portSet;
        
        //设置RunLoop为可以被唤醒状态
        __CFRunLoopUnsetIgnoreWakeUps(rl);
        
        //2.通知observer，即将触发timer回调，处理timer事件
        if (rlm->_observerMask & kCFRunLoopBeforeTimers) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeTimers);
        //3.通知observer，即将触发Source0回调
        if (rlm->_observerMask & kCFRunLoopBeforeSources) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeSources);
        
        //执行加入当前runloop的block
        __CFRunLoopDoBlocks(rl, rlm);
        
        //4.处理source0事件
        //有事件处理返回true，没有事件返回false
        Boolean sourceHandledThisLoop = __CFRunLoopDoSources0(rl, rlm, stopAfterHandle);
        if (sourceHandledThisLoop) {
            //执行加入当前runloop的block
            __CFRunLoopDoBlocks(rl, rlm);
        }
        
        //如果没有Sources0事件处理 并且 没有超时，poll为false
        //如果有Sources0事件处理 或者 超时，poll都为true
        Boolean poll = sourceHandledThisLoop || (0ULL == timeout_context->termTSR);
        
        //第一次do..whil循环不会走该分支，因为didDispatchPortLastTime初始化是true
        if (MACH_PORT_NULL != dispatchPort && !didDispatchPortLastTime) {
#if DEPLOYMENT_TARGET_MACOSX || DEPLOYMENT_TARGET_EMBEDDED || DEPLOYMENT_TARGET_EMBEDDED_MINI
            //从缓冲区读取消息
            msg = (mach_msg_header_t *)msg_buffer;
            //5.接收dispatchPort端口的消息，（接收source1事件）
            if (__CFRunLoopServiceMachPort(dispatchPort, &msg, sizeof(msg_buffer), &livePort, 0)) {
                //如果接收到了消息的话，前往第9步开始处理msg
                goto handle_msg;
            }
#elif DEPLOYMENT_TARGET_WINDOWS
            if (__CFRunLoopWaitForMultipleObjects(NULL, &dispatchPort, 0, 0, &livePort, NULL)) {
                goto handle_msg;
            }
#endif
        }
        
        didDispatchPortLastTime = false;
        
        //6.通知观察者RunLoop即将进入休眠
        if (!poll && (rlm->_observerMask & kCFRunLoopBeforeWaiting)) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeWaiting);
        //设置RunLoop为休眠状态
        __CFRunLoopSetSleeping(rl);
        // do not do any user callouts after this point (after notifying of sleeping)
        
        // Must push the local-to-this-activation ports in on every loop
        // iteration, as this mode could be run re-entrantly and we don't
        // want these ports to get serviced.
        
        __CFPortSetInsert(dispatchPort, waitSet);
        
        __CFRunLoopModeUnlock(rlm);
        __CFRunLoopUnlock(rl);
        
#if DEPLOYMENT_TARGET_MACOSX || DEPLOYMENT_TARGET_EMBEDDED || DEPLOYMENT_TARGET_EMBEDDED_MINI
#if USE_DISPATCH_SOURCE_FOR_TIMERS
        //这里有个内循环，用于接收等待端口的消息
        //进入此循环后，线程进入休眠，直到收到新消息才跳出该循环，继续执行run loop
        do {
            if (kCFUseCollectableAllocator) {
                objc_clear_stack(0);
                memset(msg_buffer, 0, sizeof(msg_buffer));
            }
            msg = (mach_msg_header_t *)msg_buffer;
            //7.接收waitSet端口的消息
            __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY);
            //收到消息之后，livePort的值为msg->msgh_local_port，
            if (modeQueuePort != MACH_PORT_NULL && livePort == modeQueuePort) {
                // Drain the internal queue. If one of the callout blocks sets the timerFired flag, break out and service the timer.
                while (_dispatch_runloop_root_queue_perform_4CF(rlm->_queue));
                if (rlm->_timerFired) {
                    // Leave livePort as the queue port, and service timers below
                    rlm->_timerFired = false;
                    break;
                } else {
                    if (msg && msg != (mach_msg_header_t *)msg_buffer) free(msg);
                }
            } else {
                // Go ahead and leave the inner loop.
                break;
            }
        } while (1);
#else
        if (kCFUseCollectableAllocator) {
            objc_clear_stack(0);
            memset(msg_buffer, 0, sizeof(msg_buffer));
        }
        msg = (mach_msg_header_t *)msg_buffer;
        __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY);
#endif
        
        
#elif DEPLOYMENT_TARGET_WINDOWS
        // Here, use the app-supplied message queue mask. They will set this if they are interested in having this run loop receive windows messages.
        __CFRunLoopWaitForMultipleObjects(waitSet, NULL, poll ? 0 : TIMEOUT_INFINITY, rlm->_msgQMask, &livePort, &windowsMessageReceived);
#endif
        
        __CFRunLoopLock(rl);
        __CFRunLoopModeLock(rlm);
        
        // Must remove the local-to-this-activation ports in on every loop
        // iteration, as this mode could be run re-entrantly and we don't
        // want these ports to get serviced. Also, we don't want them left
        // in there if this function returns.
        
        __CFPortSetRemove(dispatchPort, waitSet);
        
 
        __CFRunLoopSetIgnoreWakeUps(rl);
        
        // user callouts now OK again
        //取消runloop的休眠状态
        __CFRunLoopUnsetSleeping(rl);
        //8.通知观察者runloop被唤醒
        if (!poll && (rlm->_observerMask & kCFRunLoopAfterWaiting)) __CFRunLoopDoObservers(rl, rlm, kCFRunLoopAfterWaiting);
      
        //9.处理收到的消息
    handle_msg:;
        __CFRunLoopSetIgnoreWakeUps(rl);
        
#if DEPLOYMENT_TARGET_WINDOWS
        if (windowsMessageReceived) {
            // These Win32 APIs cause a callout, so make sure we're unlocked first and relocked after
            __CFRunLoopModeUnlock(rlm);
            __CFRunLoopUnlock(rl);
            
            if (rlm->_msgPump) {
                rlm->_msgPump();
            } else {
                MSG msg;
                if (PeekMessage(&msg, NULL, 0, 0, PM_REMOVE | PM_NOYIELD)) {
                    TranslateMessage(&msg);
                    DispatchMessage(&msg);
                }
            }
            
            __CFRunLoopLock(rl);
            __CFRunLoopModeLock(rlm);
            sourceHandledThisLoop = true;
            
            // To prevent starvation of sources other than the message queue, we check again to see if any other sources need to be serviced
            // Use 0 for the mask so windows messages are ignored this time. Also use 0 for the timeout, because we're just checking to see if the things are signalled right now -- we will wait on them again later.
            // NOTE: Ignore the dispatch source (it's not in the wait set anymore) and also don't run the observers here since we are polling.
            __CFRunLoopSetSleeping(rl);
            __CFRunLoopModeUnlock(rlm);
            __CFRunLoopUnlock(rl);
            
            __CFRunLoopWaitForMultipleObjects(waitSet, NULL, 0, 0, &livePort, NULL);
            
            __CFRunLoopLock(rl);
            __CFRunLoopModeLock(rlm);
            __CFRunLoopUnsetSleeping(rl);
            // If we have a new live port then it will be handled below as normal
        }
        
        
#endif
        if (MACH_PORT_NULL == livePort) {
            CFRUNLOOP_WAKEUP_FOR_NOTHING();
            // handle nothing
            //通过CFRunloopWake唤醒
        } else if (livePort == rl->_wakeUpPort) {
            CFRUNLOOP_WAKEUP_FOR_WAKEUP();
            //什么都不干，跳回2重新循环
            // do nothing on Mac OS
#if DEPLOYMENT_TARGET_WINDOWS
            // Always reset the wake up port, or risk spinning forever
            ResetEvent(rl->_wakeUpPort);
#endif
        }
#if USE_DISPATCH_SOURCE_FOR_TIMERS
        //如果是定时器事件
        else if (modeQueuePort != MACH_PORT_NULL && livePort == modeQueuePort) {
            CFRUNLOOP_WAKEUP_FOR_TIMER();
            //9.1 处理timer事件
            if (!__CFRunLoopDoTimers(rl, rlm, mach_absolute_time())) {
                // Re-arm the next timer, because we apparently fired early
                __CFArmNextTimerInMode(rlm, rl);
            }
        }
#endif
#if USE_MK_TIMER_TOO
        //如果是定时器事件
        else if (rlm->_timerPort != MACH_PORT_NULL && livePort == rlm->_timerPort) {
            CFRUNLOOP_WAKEUP_FOR_TIMER();
            // On Windows, we have observed an issue where the timer port is set before the time which we requested it to be set. For example, we set the fire time to be TSR 167646765860, but it is actually observed firing at TSR 167646764145, which is 1715 ticks early. The result is that, when __CFRunLoopDoTimers checks to see if any of the run loop timers should be firing, it appears to be 'too early' for the next timer, and no timers are handled.
            // In this case, the timer port has been automatically reset (since it was returned from MsgWaitForMultipleObjectsEx), and if we do not re-arm it, then no timers will ever be serviced again unless something adjusts the timer list (e.g. adding or removing timers). The fix for the issue is to reset the timer here if CFRunLoopDoTimers did not handle a timer itself. 9308754
           //9.1处理timer事件
            if (!__CFRunLoopDoTimers(rl, rlm, mach_absolute_time())) {
                // Re-arm the next timer
                __CFArmNextTimerInMode(rlm, rl);
            }
        }
#endif
        //如果是dispatch到main queue的block
        else if (livePort == dispatchPort) {
            CFRUNLOOP_WAKEUP_FOR_DISPATCH();
            __CFRunLoopModeUnlock(rlm);
            __CFRunLoopUnlock(rl);
            _CFSetTSD(__CFTSDKeyIsInGCDMainQ, (void *)6, NULL);
#if DEPLOYMENT_TARGET_WINDOWS
            void *msg = 0;
#endif
            //9.2执行block
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
            _CFSetTSD(__CFTSDKeyIsInGCDMainQ, (void *)0, NULL);
            __CFRunLoopLock(rl);
            __CFRunLoopModeLock(rlm);
            sourceHandledThisLoop = true;
            didDispatchPortLastTime = true;
        } else {
            CFRUNLOOP_WAKEUP_FOR_SOURCE();
            // Despite the name, this works for windows handles as well
            CFRunLoopSourceRef rls = __CFRunLoopModeFindSourceForMachPort(rl, rlm, livePort);
            // 有source1事件待处理
            if (rls) {
#if DEPLOYMENT_TARGET_MACOSX || DEPLOYMENT_TARGET_EMBEDDED || DEPLOYMENT_TARGET_EMBEDDED_MINI
                mach_msg_header_t *reply = NULL;
                //9.2 处理source1事件
                sourceHandledThisLoop = __CFRunLoopDoSource1(rl, rlm, rls, msg, msg->msgh_size, &reply) || sourceHandledThisLoop;
                if (NULL != reply) {
                    (void)mach_msg(reply, MACH_SEND_MSG, reply->msgh_size, 0, MACH_PORT_NULL, 0, MACH_PORT_NULL);
                    CFAllocatorDeallocate(kCFAllocatorSystemDefault, reply);
                }
#elif DEPLOYMENT_TARGET_WINDOWS
                sourceHandledThisLoop = __CFRunLoopDoSource1(rl, rlm, rls) || sourceHandledThisLoop;
#endif
            }
        }
#if DEPLOYMENT_TARGET_MACOSX || DEPLOYMENT_TARGET_EMBEDDED || DEPLOYMENT_TARGET_EMBEDDED_MINI
        if (msg && msg != (mach_msg_header_t *)msg_buffer) free(msg);
#endif
        
        __CFRunLoopDoBlocks(rl, rlm);
        
        if (sourceHandledThisLoop && stopAfterHandle) {
            //进入run loop时传入的参数，处理完事件就返回
            retVal = kCFRunLoopRunHandledSource;
        }else if (timeout_context->termTSR < mach_absolute_time()) {
            //run loop超时
            retVal = kCFRunLoopRunTimedOut;
        }else if (__CFRunLoopIsStopped(rl)) {
            //run loop被手动终止
            __CFRunLoopUnsetStopped(rl);
            retVal = kCFRunLoopRunStopped;
        }else if (rlm->_stopped) {
            //mode被终止
            rlm->_stopped = false;
            retVal = kCFRunLoopRunStopped;
        }else if (__CFRunLoopModeIsEmpty(rl, rlm, previousMode)) {
            //mode中没有要处理的事件
            retVal = kCFRunLoopRunFinished;
        }
        //除了上面这几种情况，都继续循环
    } while (0 == retVal);
    
    if (timeout_timer) {
        dispatch_source_cancel(timeout_timer);
        dispatch_release(timeout_timer);
    } else {
        free(timeout_context);
    }
    
    return retVal;
}
```

- **给源代码添加注释是一件非常需要耐心的事情。**

> 可以看到，实际上 RunLoop 就是这样一个函数，其内部是一个 do-while 循环。当你调用 CFRunLoopRun() 时，线程就会一直停留在这个循环里；直到超时或被手动停止，该函数才会返回。同时RunLoop有很多个mode，但是RunLoop在run的时候必须只能指定其中一个mode，运行起来之后，被指定的mode即为currentMode。

> 这里有个细节RunLoop 的超时时间就是使用 GCD 中的 dispatch_source_t来实现的对应到上面的代码可以看一看。

#### Runloop相关操作

CFRunLoop是基于pthread来管理。iOS中不能直接创建Runloop，只能从系统中获取CFRunLoopGetMain() 和 CFRunLoopGetCurrent()。

系统一共提供了如下几种方式，区别在于面向的框架不一样：



```objectivec
CFRunLoopRef CFRunLoopGetCurrent(void);//获取当前线程的RunLoop对象
CFRunLoopRef CFRunLoopGetMain(void);//获取主线程的RunLoop对象
+(NSRunLoop *)currentRunLoop
+(NSRunLoop *)mainRunLoop
```

##### 获取当前线程



```c
//取当前所在线程的RunLoop
CFRunLoopRef CFRunLoopGetCurrent(void) {
    CHECK_FOR_FORK();
    CFRunLoopRef rl = (CFRunLoopRef)_CFGetTSD(__CFTSDKeyRunLoop);
    if (rl) return rl;
    //传入当前线程
    return _CFRunLoopGet0(pthread_self());
}
```

在CFRunLoopGetCurrent函数内部调用了_CFRunLoopGet0()，传入的参数是当前线程（`eturn _CFRunLoopGet0(pthread_self());`）。这里可以看出，CFRunLoopGetCurrent函数必须要在线程内部调用，才能获取当前线程的RunLoop。也就是说子线程的RunLoop必须要在子线程内部获取。

##### CFRunLoopGetMain



```objectivec
//取主线程的RunLoop
CFRunLoopRef CFRunLoopGetMain(void) {
    CHECK_FOR_FORK();
    static CFRunLoopRef __main = NULL; // no retain needed
    //传入主线程
    if (!__main) __main = _CFRunLoopGet0(pthread_main_thread_np()); // no CAS needed
    return __main;
}
```

在CFRunLoopGetMain函数内部也调用了_CFRunLoopGet0()，传入的参数是主线程。可以看出，CFRunLoopGetMain()不管在主线程还是子线程中调用，都可以获取到主线程的RunLoop。

##### CFRunLoopGet0

获取当前的及主线程的runloop最终都是调用CFRunLoopGet0来实现的。



```c
//全局的Dictionary，key 是 pthread_t， value 是 CFRunLoopRef
static CFMutableDictionaryRef __CFRunLoops = NULL;
/// 访问 loopsDic 时的锁, 可以知道__CFRunLoops是线程不安全的
static CFSpinLock_t loopsLock = CFSpinLockInit;

// should only be called by Foundation
// t==0 is a synonym for "main thread" that always works
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
// 如果传入线程为空，默认则为主线程
    if (pthread_equal(t, kNilPthreadT)) {
    t = pthread_main_thread_np();
    }
    //加锁访问字典
    __CFSpinLock(&loopsLock);
    if (!__CFRunLoops) {
    // 第一次进入时，初始化全局__CFSpinUnlock字典，并先为主线程创建一个 RunLoop。
        __CFSpinUnlock(&loopsLock);
    CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
    CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
    CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
    
    //释放临时创建的变量
    if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
        CFRelease(dict);
    }
    CFRelease(mainLoop);
    //解锁
        __CFSpinLock(&loopsLock);
    }
    //如果不是第一次，则直接从字典里面取
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFSpinUnlock(&loopsLock);
    
    if (!loop) {
    //如果取不到则创建一个新的CFRunLoopRef，然后存在全局的字典里面，传入的参数（线程指针）作为key
    CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFSpinLock(&loopsLock);
    loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    if (!loop) {
       //保存到字典
        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
        loop = newLoop;
    }
    //释放资源
        // don't release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFSpinUnlock(&loopsLock);
    CFRelease(newLoop);
    }
    
    //如果传入的线程是当前线程
    if (pthread_equal(t, pthread_self())) {
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
        // 注册一个回调，当线程销毁时，顺便也销毁其对应的 RunLoop。
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```

通过上面代码可以总结出:

- 线程和 RunLoop 之间是一一对应的，其关系是保存在一个全局的 Dictionary 里。
- 线程刚创建时并没有 RunLoop（没有加到对应的runloop字典中），如果你不主动获取，那它一直都不会有。
- RunLoop 的创建是发生在第一次获取时。一般是获取主线程的时候。
- RunLoop 的销毁是发生在线程结束时。
- 只能在一个线程的内部获取其 RunLoop（主线程除外），否则就这个Runloop就没有注册销毁回调。这一点是根据`pthread_equal(t, pthread_self())`后面的代码，如果是当前线程后面才会注册销毁回调。**因为上面讲过Runlopp暴露给外部的创建方式只有CFRunLoopGetMain() 和 CFRunLoopGetCurrent()两种，所以这种情况不用考虑。**下面是CFRunloop.h的头文件暴露接口，可以看到获取方式只有两种。

![img](https://upload-images.jianshu.io/upload_images/664334-f35be006cb828f58.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/439/format/webp)

#### Mode相关操作

在Core Foundation中，针对Mode的操作，苹果只开放了如下API:



```objectivec
CFRunLoopAddCommonMode(CFRunLoopRef rl, CFStringRef mode)//向当前RunLoop的common modes中添加一个mode。
CFStringRef CFRunLoopCopyCurrentMode(CFRunLoopRef rl)//返回当前运行的mode的name
CFArrayRef CFRunLoopCopyAllModes(CFRunLoopRef rl)//返回当前RunLoop的所有mode
```

> 我们没有办法直接创建一个CFRunLoopMode对象，但是我们可以调用CFRunLoopAddCommonMode传入一个字符串向RunLoop中添加Mode，传入的字符串即为Mode的名字，Mode对象应该是此时在RunLoop内部创建的。**特别注意只能通过CFRunLoopAddCommonMode，是CommonMode。**

##### CFRunLoopAddCommonMode



```c
void CFRunLoopAddCommonMode(CFRunLoopRef rl, CFStringRef modeName) {
    CHECK_FOR_FORK();
    if (__CFRunLoopIsDeallocating(rl)) return;
    __CFRunLoopLock(rl);
    //看rl中是否已经有这个mode，如果有就什么都不做
    if (!CFSetContainsValue(rl->_commonModes, modeName)) {
        CFSetRef set = rl->_commonModeItems ? CFSetCreateCopy(kCFAllocatorSystemDefault, rl->_commonModeItems) : NULL;
        //把modeName添加到RunLoop的_commonModes中
        CFSetAddValue(rl->_commonModes, modeName);
        if (NULL != set) {
            CFTypeRef context[2] = {rl, modeName};
            /* add all common-modes items to new mode */
            //__CFRunLoopAddItemsToCommonMode是一个方法，里面会调用CFRunLoopAddSource/CFRunLoopAddObserver/CFRunLoopAddTimer
            //__CFRunLoopFindMode(rl, modeName, true)，CFRunLoopMode对象在这个时候被创建
            CFSetApplyFunction(set, (__CFRunLoopAddItemsToCommonMode), (void *)context);
            CFRelease(set);
        }
    } else {
    }
    __CFRunLoopUnlock(rl);
}
```

可以总结出如下几点：

- modeName不能重复，modeName是mode的唯一标识符
- 添加commonMode会把commonModeItems数组中的所有item(source，observe，timer,)同步到新添加的mode中。

CFRunLoopCopyCurrentMode/CFRunLoopCopyAllModes实现比较简单，直接返回对应的model。

#### Source相关操作（ModeItem）

系统提供了如下几个函数来操作Item



```c
CF_EXPORT Boolean CFRunLoopContainsSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFRunLoopMode mode);
CF_EXPORT void CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFRunLoopMode mode);
CF_EXPORT void CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef source, CFRunLoopMode mode);

CF_EXPORT Boolean CFRunLoopContainsObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFRunLoopMode mode);
CF_EXPORT void CFRunLoopAddObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFRunLoopMode mode);
CF_EXPORT void CFRunLoopRemoveObserver(CFRunLoopRef rl, CFRunLoopObserverRef observer, CFRunLoopMode mode);

CF_EXPORT Boolean CFRunLoopContainsTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFRunLoopMode mode);
CF_EXPORT void CFRunLoopAddTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFRunLoopMode mode);
CF_EXPORT void CFRunLoopRemoveTimer(CFRunLoopRef rl, CFRunLoopTimerRef timer, CFRunLoopMode mode);
```

可以看到总体来讲提供了对item的判断是否已经包含item，添加，删除的功能。

##### CFRunLoopAddSource

作用：将一个CFRunLoopAddSource对象添加到一个指定的Runloop Model中。



```c
//添加source事件
void CFRunLoopAddSource(CFRunLoopRef rl, CFRunLoopSourceRef rls, CFStringRef modeName) {    /* DOES CALLOUT */
    CHECK_FOR_FORK();
    if (__CFRunLoopIsDeallocating(rl)) return;
    if (!__CFIsValid(rls)) return;
    Boolean doVer0Callout = false;
    __CFRunLoopLock(rl);
    //如果是kCFRunLoopCommonModes
    if (modeName == kCFRunLoopCommonModes) {
        //如果runloop的_commonModes存在，则copy一个新的复制给set
        CFSetRef set = rl->_commonModes ? CFSetCreateCopy(kCFAllocatorSystemDefault, rl->_commonModes) : NULL;
       //如果runl _commonModeItems为空
        if (NULL == rl->_commonModeItems) {
            //先初始化
            rl->_commonModeItems = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
        }
        //把传入的CFRunLoopSourceRef加入_commonModeItems
        CFSetAddValue(rl->_commonModeItems, rls);
        //如果刚才set copy到的数组里有数据
        if (NULL != set) {
            CFTypeRef context[2] = {rl, rls};
            /* add new item to all common-modes */
            //则把set里的所有mode都执行一遍__CFRunLoopAddItemToCommonModes函数
            CFSetApplyFunction(set, (__CFRunLoopAddItemToCommonModes), (void *)context);
            CFRelease(set);
        }
        //以上分支的逻辑就是，如果你往kCFRunLoopCommonModes里面添加一个source，那么所有_commonModes里的mode都会添加这个source
    } else {
        //根据modeName查找mode
        CFRunLoopModeRef rlm = __CFRunLoopFindMode(rl, modeName, true);
        //如果_sources0不存在，则初始化_sources0，_sources0和_portToV1SourceMap
        if (NULL != rlm && NULL == rlm->_sources0) {
            rlm->_sources0 = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
            rlm->_sources1 = CFSetCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeSetCallBacks);
            rlm->_portToV1SourceMap = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, NULL);
        }
        //如果_sources0和_sources1中都不包含传入的source
        if (NULL != rlm && !CFSetContainsValue(rlm->_sources0, rls) && !CFSetContainsValue(rlm->_sources1, rls)) {
            //如果version是0，则加到_sources0
            if (0 == rls->_context.version0.version) {
                CFSetAddValue(rlm->_sources0, rls);
                //如果version是1，则加到_sources1
            } else if (1 == rls->_context.version0.version) {
                CFSetAddValue(rlm->_sources1, rls);
                __CFPort src_port = rls->_context.version1.getPort(rls->_context.version1.info);
                if (CFPORT_NULL != src_port) {
                    //此处只有在加到source1的时候才会把souce和一个mach_port_t对应起来
                    //可以理解为，source1可以通过内核向其端口发送消息来主动唤醒runloop
                    CFDictionarySetValue(rlm->_portToV1SourceMap, (const void *)(uintptr_t)src_port, rls);
                    __CFPortSetInsert(src_port, rlm->_portSet);
                }
            }
            __CFRunLoopSourceLock(rls);
            //把runloop加入到source的_runLoops中
            if (NULL == rls->_runLoops) {
                rls->_runLoops = CFBagCreateMutable(kCFAllocatorSystemDefault, 0, &kCFTypeBagCallBacks); // sources retain run loops!
            }
            CFBagAddValue(rls->_runLoops, rl);
            __CFRunLoopSourceUnlock(rls);
            if (0 == rls->_context.version0.version) {
                if (NULL != rls->_context.version0.schedule) {
                    doVer0Callout = true;
                }
            }
        }
        if (NULL != rlm) {
            __CFRunLoopModeUnlock(rlm);
        }
    }
    __CFRunLoopUnlock(rl);
    if (doVer0Callout) {
        // although it looses some protection for the source, we have no choice but
        // to do this after unlocking the run loop and mode locks, to avoid deadlocks
        // where the source wants to take a lock which is already held in another
        // thread which is itself waiting for a run loop/mode lock
        rls->_context.version0.schedule(rls->_context.version0.info, rl, modeName); /* CALLOUT */
    }
}
```

可以总结如下：

- source会被runloop持有。
- 如果modeName传入kCFRunLoopCommonModes，则该source会被保存到RunLoop的commonModeItems中，然后添加到common models每个mode下面，进而被所有的common models监控。
- 如果modeName传入的不是kCFRunLoopCommonModes，则会先查找该Mode，如果没有，会创建一个。
- 同一个source在一个mode中只能被添加一次。
- 只有在加到source1的时候才会把souce和一个mach_port_t对应起来，这个mach_port_t由source传入。

##### CFRunLoopRemoveSource

remove操作和add操作的逻辑基本一致，很容易理解



```c
//移除source
void CFRunLoopRemoveSource(CFRunLoopRef rl, CFRunLoopSourceRef rls, CFStringRef modeName) { /* DOES CALLOUT */
    CHECK_FOR_FORK();
    Boolean doVer0Callout = false, doRLSRelease = false;
    __CFRunLoopLock(rl);
    //如果是kCFRunLoopCommonModes，则从_commonModes的所有mode中移除该source
    if (modeName == kCFRunLoopCommonModes) {
        if (NULL != rl->_commonModeItems && CFSetContainsValue(rl->_commonModeItems, rls)) {
            CFSetRef set = rl->_commonModes ? CFSetCreateCopy(kCFAllocatorSystemDefault, rl->_commonModes) : NULL;
            CFSetRemoveValue(rl->_commonModeItems, rls);
            if (NULL != set) {
                CFTypeRef context[2] = {rl, rls};
                /* remove new item from all common-modes */
                CFSetApplyFunction(set, (__CFRunLoopRemoveItemFromCommonModes), (void *)context);
                CFRelease(set);
            }
        } else {
        }
    } else {
        //根据modeName查找mode，如果不存在，返回NULL
        CFRunLoopModeRef rlm = __CFRunLoopFindMode(rl, modeName, false);
        if (NULL != rlm && ((NULL != rlm->_sources0 && CFSetContainsValue(rlm->_sources0, rls)) || (NULL != rlm->_sources1 && CFSetContainsValue(rlm->_sources1, rls)))) {
            CFRetain(rls);
            //根据source版本做对应的remove操作
            if (1 == rls->_context.version0.version) {
                __CFPort src_port = rls->_context.version1.getPort(rls->_context.version1.info);
                if (CFPORT_NULL != src_port) {
                    CFDictionaryRemoveValue(rlm->_portToV1SourceMap, (const void *)(uintptr_t)src_port);
                    __CFPortSetRemove(src_port, rlm->_portSet);
                }
            }
            CFSetRemoveValue(rlm->_sources0, rls);
            CFSetRemoveValue(rlm->_sources1, rls);
            __CFRunLoopSourceLock(rls);
            if (NULL != rls->_runLoops) {
                CFBagRemoveValue(rls->_runLoops, rl);
            }
            __CFRunLoopSourceUnlock(rls);
            if (0 == rls->_context.version0.version) {
                if (NULL != rls->_context.version0.cancel) {
                    doVer0Callout = true;
                }
            }
            doRLSRelease = true;
        }
        if (NULL != rlm) {
            __CFRunLoopModeUnlock(rlm);
        }
    }
    __CFRunLoopUnlock(rl);
    if (doVer0Callout) {
        // although it looses some protection for the source, we have no choice but
        // to do this after unlocking the run loop and mode locks, to avoid deadlocks
        // where the source wants to take a lock which is already held in another
        // thread which is itself waiting for a run loop/mode lock
        rls->_context.version0.cancel(rls->_context.version0.info, rl, modeName);   /* CALLOUT */
    }
    if (doRLSRelease) CFRelease(rls);
}
```

##### CFRunLoopAddItemToCommonModes



```c
static void __CFRunLoopAddItemToCommonModes(const void *value, void *ctx) {
    CFStringRef modeName = (CFStringRef)value;
    CFRunLoopRef rl = (CFRunLoopRef)(((CFTypeRef *)ctx)[0]);//取出runloop
    CFTypeRef item = (CFTypeRef)(((CFTypeRef *)ctx)[1]);//取出添加到item(可能是source、observer、timer)
    //根据类型添加到对应的model中。
    if (CFGetTypeID(item) == CFRunLoopSourceGetTypeID()) {
    CFRunLoopAddSource(rl, (CFRunLoopSourceRef)item, modeName);
    } else if (CFGetTypeID(item) == CFRunLoopObserverGetTypeID()) {
    CFRunLoopAddObserver(rl, (CFRunLoopObserverRef)item, modeName);
    } else if (CFGetTypeID(item) == CFRunLoopTimerGetTypeID()) {
    CFRunLoopAddTimer(rl, (CFRunLoopTimerRef)item, modeName);
    }
}
```

#### Observer/Timer的相关操作

添加observer和timer的内部逻辑和添加source大体类似。

> 区别在于observer和timer只能被添加到一个RunLoop的一个或者多个mode中，比如一个timer被添加到主线程的RunLoop中，则不能再把该timer添加到子线程的RunLoop，而source没有这个限制，不管是哪个RunLoop，只要mode中没有，就可以添加。

上面的记录可以从CFRunLoopSource结构体可以明确的知道。**CFRunLoopSource中有保存RunLoop对象的数组，而CFRunLoopObserver和CFRunLoopTimer只有单个RunLoop对象。**

### 小结

以上内容是对runloop从源码角度的理解过程。由于代码比较多，看起来也费事，可以直接选择重点内容看。下面的内容主要来至于[深入理解RunLoop](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)这篇文章。毕竟这篇文章在国内来讲，是研究runloop文章中，国内写得非常有参考价值的文章。

## Runloop在系统中的应用（下面内容大部分来源于[深入理解RunLoop](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)）

首先把之前的runloop执行过程中的函数使用长函数名改一下，这样便于在实际调试中便于分析。因为在真实debug时不会出现上面的函数名，而是通过长函数名代替，在源码中也能看到之前的函数名和这里的长函数名是同一个函数。



```c
{
    /// 1. 通知Observers，即将进入RunLoop
    /// 此处有Observer会创建AutoreleasePool: _objc_autoreleasePoolPush();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopEntry);
    do {
 
        /// 2. 通知 Observers: 即将触发 Timer 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeTimers);
        /// 3. 通知 Observers: 即将触发 Source (非基于port的,Source0) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeSources);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
 
        /// 4. 触发 Source0 (非基于port的) 回调。
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__(source0);
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
 
        /// 6. 通知Observers，即将进入休眠
        /// 此处有Observer释放并新建AutoreleasePool: _objc_autoreleasePoolPop(); _objc_autoreleasePoolPush();
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopBeforeWaiting);
 
        /// 7. sleep to wait msg.
        mach_msg() -> mach_msg_trap();
        
 
        /// 8. 通知Observers，线程被唤醒
        __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopAfterWaiting);
 
        /// 9. 如果是被Timer唤醒的，回调Timer
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_TIMER_CALLBACK_FUNCTION__(timer);
 
        /// 9. 如果是被dispatch唤醒的，执行所有调用 dispatch_async 等方法放入main queue 的 block
        __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(dispatched_block);
 
        /// 9. 如果如果Runloop是被 Source1 (基于port的) 的事件唤醒了，处理这个事件
        __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE1_PERFORM_FUNCTION__(source1);
 
 
    } while (...);
 
    /// 10. 通知Observers，即将退出RunLoop
    /// 此处有Observer释放AutoreleasePool: _objc_autoreleasePoolPop();
    __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__(kCFRunLoopExit);
}
```

这里举个例子，如下debug信息：



![img](https://upload-images.jianshu.io/upload_images/664334-94979e984762bb5e.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/570/format/webp)

对应到上面的过程就是：**触发 Source0 (非基于port的) 回调。**

下图是对主线程的runloop的中在kCFRunLoopDefaultMode模式下所有的observer的日志。



![img](https://upload-images.jianshu.io/upload_images/664334-ddac29c5cbd56ccc.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/896/format/webp)

### Block

在最开始介绍CFRunloop的时候就简单提了一下其中关于block的两个字段blocks_head，blocks_tail。并且也提到在runloop周期中会对此调用__CFRunLoopDoBlocks来执行加入到这个runloop的block。下面从源码来说明一下block如何与runloop结合的。

先来看看最基本的block_item 数据结构，特别注意这里保存了runloop的model，决定了block是否应该执行。



```c
struct _block_item {
    struct _block_item *_next;
    CFTypeRef _mode;    // CFString or CFSet
    void (^_block)(void);
};
```

在执行block的时候会传入



```c
/**
 执行block

 @param rl runloop
 @param rlm 当前的model
 @return 是否执行
 */
static Boolean __CFRunLoopDoBlocks(CFRunLoopRef rl, CFRunLoopModeRef rlm) { // Call with rl and rlm locked
   //如果头结点没有、或者model不存在则强制返回，什么也不做
    if (!rl->_blocks_head) return false;
    if (!rlm || !rlm->_name) return false;
    Boolean did = false;//记录其中一个block结点是否被执行过
    //取出头尾结点，并且将当前runloop保存的头尾节点置位NULL
    struct _block_item *head = rl->_blocks_head;
    struct _block_item *tail = rl->_blocks_tail;
    rl->_blocks_head = NULL;
    rl->_blocks_tail = NULL;
    //取出被标记为common的所有mode、及当前model的name
    CFSetRef commonModes = rl->_commonModes;
    CFStringRef curMode = rlm->_name;
    __CFRunLoopModeUnlock(rlm);
    __CFRunLoopUnlock(rl);
    
    //定义两个临时变量，用于对保存block链表的遍历
    struct _block_item *prev = NULL;
    struct _block_item *item = head;//记录头指针，从头部开始遍历
    //开始遍历block链表
    while (item) {
        struct _block_item *curr = item;
        item = item->_next;
    Boolean doit = false；//表示是否应该执行这个block,注意和前面的did区分开
    
    //从blockitem结构体就知道,其中的_mode只能是CFString 或者CFSet
    //如果block结点保存的model是CFString类型
    if (CFStringGetTypeID() == CFGetTypeID(curr->_mode)) {
       //是否执行block只需要满足下面三个条件中的一个
       //1. blockitem 中保存的model是当前的model
       //2. blockitem 中保存的model是标记为kCFRunLoopCommonModes的model
       //3. 当前model保存在commonModes数组
        doit = CFEqual(curr->_mode, curMode) || (CFEqual(curr->_mode, kCFRunLoopCommonModes) && CFSetContainsValue(commonModes, curMode));
        } else {
       //如果block结点保存的model是CFSet类型，步骤和上面一样，等于换成了包含。
        doit = CFSetContainsValue((CFSetRef)curr->_mode, curMode) || (CFSetContainsValue((CFSetRef)curr->_mode, kCFRunLoopCommonModes) && CFSetContainsValue(commonModes, curMode));
    }
    
    //如果不执行block,则直接移动当前结点，进行下一个blockitem的判断
    if (!doit) prev = curr;
    if (doit) {
    //如果执行block,则先移动结点。
        if (prev) prev->_next = item;
        if (curr == head) head = item;
        if (curr == tail) tail = prev;
       
        void (^block)(void) = curr->_block;
            CFRelease(curr->_mode);
            free(curr);
        if (doit) {
        //最终在这里执行block，__CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__的函数原型就是调用block
                __CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__(block);
            did = true;
        }
            Block_release(block); // do this before relocking to prevent deadlocks where some yahoo wants to run the run loop reentrantly from their dealloc
    }
    }
    __CFRunLoopLock(rl);
    __CFRunLoopModeLock(rlm);
    //重建循环链表
    if (head) {
    tail->_next = rl->_blocks_head;
    rl->_blocks_head = head;
        if (!rl->_blocks_tail) rl->_blocks_tail = tail;
    }
    return did;
}
```

通过上面分析可以知道：

1. block其实在runloop中通过循环链表保存的
2. 如果block可以加入到多个model下面，但是执行block只有在加入的那个model下才能之后，或者加入modle用common标记
3. 每次调用__CFRunLoopDoBlocks，会把加入的block遍历执行，然后重置循环链表。

### AutoreleasePool

App启动后，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 _wrapRunLoopWithAutoreleasePoolHandler()。

第一个 Observer 监视的事件是 Entry(即将进入Loop)，其回调内会调用 _objc_autoreleasePoolPush() 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

第二个 Observer 监视了两个事件： BeforeWaiting(准备进入休眠) 时调用_objc_autoreleasePoolPop() 和 _objc_autoreleasePoolPush() 释放旧的池并创建新池；Exit(即将退出Loop) 时调用 _objc_autoreleasePoolPop() 来释放自动释放池。**这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。**

在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 AutoreleasePool 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。

**关于Autorelease后面需要一篇源码分析来说明问题。**

### 手势识别

上面可以看到第二个observe就是_UIGestureRecognizerUpdateObserver，关于手势识别的。

当上面的 _UIApplicationHandleEventQueue() 识别了一个手势时，其首先会调用 Cancel 将当前的 touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。

苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，这个Observer的回调函数是 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。

当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。

### 界面更新

上面可以看到第三和四个observe分别是_beforeCACommitHandler与_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv，是关于动画及界面更新的。

当在操作 UI 时，比如改变了 Frame、更新了 UIView/CALayer 的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理，并被提交到一个全局的容器去。

苹果注册了一个 Observer 监听 BeforeWaiting(即将进入休眠) 和 Exit (即将退出Loop) 事件，回调去执行一个很长的函数：
_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()。这个函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

### 定时器

上面截图中还有个timer

NSTimer 其实就是 CFRunLoopTimerRef，他们之间是 toll-free bridged 的。一个 NSTimer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件。例如 10:00, 10:10, 10:20 这几个时间点。RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer。Timer 有个属性叫做 Tolerance (宽容度)，标示了当时间点到后，容许有多少最大误差。

如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。

#### PerformSelecter

当调用 NSObject 的 performSelecter:afterDelay: 来实现延迟执行，实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中。**所以如果当前线程没有 RunLoop，则这个方法会失效。**

当调用 performSelector:onThread: 时，实际上其会创建一个 Timer 加到对应的线程去，**同样的，如果对应线程没有 RunLoop 该方法也会失效。**

### GCD

> 在看runloop执行过程的源码中，可以知道RunLoop 的超时时间就是使用 GCD 中的 dispatch_source_t来实现的。

当调用 dispatch_async(dispatch_get_main_queue(), block) 时，libDispatch 会向主线程的 RunLoop 发送消息，RunLoop会被唤醒，并从消息中取得这个 block，并在回调 **CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE**() 里执行这个 block。但这个逻辑仅限于 dispatch 到主线程，dispatch 到其他线程仍然是由 libDispatch 处理的。

### 网络请求

iOS 中，关于网络请求的接口自下至上有如下几层:



```c
CFSocket
CFNetwork       ->ASIHttpRequest
NSURLConnection ->AFNetworking
NSURLSession    ->AFNetworking2, Alamofire
```

CFSocket 是最底层的接口，只负责 socket 通信。
• CFNetwork 是基于 CFSocket 等接口的上层封装，ASIHttpRequest 工作于这一层。
• NSURLConnection 是基于 CFNetwork 的更高层的封装，提供面向对象的接口，AFNetworking 工作于这一层。
• NSURLSession 是 iOS7 中新增的接口，表面上是和 NSURLConnection 并列的，但底层仍然用到了 NSURLConnection 的部分功能 (比如 com.apple.NSURLConnectionLoader 线程)，AFNetworking2 和 Alamofire 工作于这一层。

#### 工作原理

通常使用 NSURLConnection 时，你会传入一个 Delegate，当调用了 [connection start] 后，这个 Delegate 就会不停收到事件回调。实际上，start 这个函数的内部会会获取 CurrentRunLoop，然后在其中的 DefaultMode 添加了4个 Source0 (即需要手动触发的Source)。CFMultiplexerSource 是负责各种 Delegate 回调的，CFHTTPCookieStorage 是处理各种 Cookie 的。

当开始网络传输时，我们可以看到 NSURLConnection 创建了两个新线程：com.apple.NSURLConnectionLoader 和 com.apple.CFSocket.private。其中 CFSocket 线程是处理底层 socket 连接的。NSURLConnectionLoader 这个线程内部会使用 RunLoop 来接收底层 socket 的事件，并通过之前添加的 Source0 通知到上层的 Delegate。



![img](https://upload-images.jianshu.io/upload_images/664334-4d5ca6982a692bc0..png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

image

NSURLConnectionLoader 中的 RunLoop 通过一些基于 mach port 的 Source 接收来自底层 CFSocket 的通知。当收到通知后，其会在合适的时机向 CFMultiplexerSource 等 Source0 发送通知，同时唤醒 Delegate 线程的 RunLoop 来让其处理这些通知。CFMultiplexerSource 会在 Delegate 线程的 RunLoop 对 Delegate 执行实际的回调。

## Runloop在平时开发中的应用（[深入理解RunLoop](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)）

### AFNetworking

为了线程保活。
代码：



```objective-c
- (void)start {
    [self.lock lock];
    if ([self isCancelled]) {
        [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    } else if ([self isReady]) {
        self.state = AFOperationExecutingState;
        [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    }
    [self.lock unlock];
}

+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}

+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}
```

上面函数的调用关系由上到下调用。

AF希望能在后台线程接收 Delegate 回调。为此 AFNetworking 单独创建了一个线程，并在这个线程中启动了一个 RunLoop。过程在networkRequestThreadEntryPoint中，因为RunLoop 启动前内部必须要有至少一个 Timer/Observer/Source，所以 AFNetworking 在 [runLoop run] 之前先创建了一个新的 NSMachPort 添加进去了。通常情况下，调用者需要持有这个 NSMachPort (mach_port) 并在外部线程通过这个 port 发送消息到 loop 内；但此处添加 port 只是为了让 RunLoop 不至于退出，并没有用于实际的发送消息。

当需要这个后台线程执行任务时，AFNetworking 通过调用 [NSObject performSelector:onThread:..] 将这个任务扔到了后台线程的 RunLoop 中，具体内容对应start方法。

### UITableView+FDTemplateLayoutCell

利用runloop的空闲状态计算高度达到预缓存的功能。具体分析见这里[优化UITableViewCell高度计算的那些事](https://link.jianshu.com/?t=http%3A%2F%2Fblog.sunnyxx.com%2F2015%2F05%2F17%2Fcell-height-calculation%2F)

### AsyncDisplayKit

ASDK 仿照 QuartzCore/UIKit 框架的模式，实现了一套类似的界面更新的机制：即在主线程的 RunLoop 中添加一个 Observer，监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit 事件，在收到回调时，遍历所有之前放入队列的待处理的任务，然后一一执行。

可以直接看源码进行分析[AsyncDisplayKit](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Ffacebookarchive%2FAsyncDisplayKit)，但是现在更名为[Texture](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Ftexturegroup%2Ftexture%2F)

# 扩展阅读

[RunLoop官方介绍](https://link.jianshu.com/?t=https%3A%2F%2Fdeveloper.apple.com%2Flibrary%2Fcontent%2Fdocumentation%2FCocoa%2FConceptual%2FMultithreading%2FRunLoopManagement%2FRunLoopManagement.html%23%2Fapple_ref%2Fdoc%2Fuid%2F10000057i-CH16-SW23)
[深入理解RunLoop](https://link.jianshu.com/?t=https%3A%2F%2Fblog.ibireme.com%2F2015%2F05%2F18%2Frunloop%2F)



[客户端技术](https://www.jianshu.com/nb/3488959)