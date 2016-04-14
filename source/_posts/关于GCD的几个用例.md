title: 关于GCD的几个用例
tags:
  - GCD
categories: []
date: 2016-03-11 23:13:00
---
GCD作为日常开发中的使用已经非常普遍，基于C的API为应用在多核硬件上高效运行提供了有力支持。本文分场景写了几个测试Demo，方便大家理解与应用。

####1，dispatch_get_global_queue与dispatch_get_main_queue交互

很多应用场景需要后台读写大量数据，通过dispatch_get_global_queue函数可以获取全局队列来并发执行后台任务，并再结束后更新UI，保证应用的流程，避免主线程阻塞。代码如下：

``` objc
- (void)test1
{
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_group_t group = dispatch_group_create();
    for (int i = 0; i < 10; i++) {
        dispatch_group_async(group, queue, ^{
            NSLog(@"___%i %@", i, [NSThread currentThread]);
        });
    }
    dispatch_group_notify(group, queue, ^{
        NSLog(@"currentThread1 %@",[NSThread currentThread]);
        NSLog(@"执行完毕");
        
        dispatch_async(dispatch_get_main_queue(), ^{
            NSLog(@"currentThread2 %@",[NSThread currentThread]);
            NSLog(@"main_queue 执行完毕");
        });        
    });
}
```

执行结果：

``` objc
 test1：
 2016-03-10 17:21:37.293 GCDSample[3892:2238505] ___0 <NSThread: 0x12e53e570>{number = 2, name = (null)}
 2016-03-10 17:21:37.293 GCDSample[3892:2238506] ___2 <NSThread: 0x12e67f1e0>{number = 4, name = (null)}
 2016-03-10 17:21:37.293 GCDSample[3892:2238504] ___3 <NSThread: 0x12e689760>{number = 3, name = (null)}
 2016-03-10 17:21:37.293 GCDSample[3892:2238507] ___1 <NSThread: 0x12e683590>{number = 5, name = (null)}
 2016-03-10 17:21:37.294 GCDSample[3892:2238505] ___4 <NSThread: 0x12e53e570>{number = 2, name = (null)}
 2016-03-10 17:21:37.295 GCDSample[3892:2238506] ___5 <NSThread: 0x12e67f1e0>{number = 4, name = (null)}
 2016-03-10 17:21:37.295 GCDSample[3892:2238504] ___6 <NSThread: 0x12e689760>{number = 3, name = (null)}
 2016-03-10 17:21:37.295 GCDSample[3892:2238507] ___7 <NSThread: 0x12e683590>{number = 5, name = (null)}
 2016-03-10 17:21:37.295 GCDSample[3892:2238505] ___8 <NSThread: 0x12e53e570>{number = 2, name = (null)}
 2016-03-10 17:21:37.298 GCDSample[3892:2238506] ___9 <NSThread: 0x12e67f1e0>{number = 4, name = (null)}
 2016-03-10 17:21:37.299 GCDSample[3892:2238506] currentThread1 <NSThread: 0x12e67f1e0>{number = 4, name = (null)}
 2016-03-10 17:21:37.299 GCDSample[3892:2238506] 执行完毕
 2016-03-10 17:21:37.302 GCDSample[3892:2238481] currentThread2 <NSThread: 0x12e50c390>{number = 1, name = main}
 2016-03-10 17:21:37.303 GCDSample[3892:2238481] main_queue 执行完毕

```

####2，异步线程中串行执行任务
test1中dispatch_group_async以并发的方式开启异步线程，不能保证执行顺序，如果想在并发线程中串行执行任务该如何做呢？只需要创建一个串行队列，加入group任务即可。

``` objc
- (void)test1
{
    NSLog(@"code begin");
    NSLog(@"currentThread0 %@",[NSThread currentThread]);
    dispatch_group_t group = dispatch_group_create();
    dispatch_queue_t serialQueue = dispatch_queue_create(NULL, DISPATCH_QUEUE_SERIAL);
    
    for (int i = 0; i < 10; i++) {
        dispatch_group_async(group, serialQueue, ^{
            [NSThread sleepForTimeInterval:.1f];
            NSLog(@"___%i %@", i, [NSThread currentThread]);
        });
    }
    
    dispatch_group_notify(group, serialQueue, ^{
        NSLog(@"currentThread1 %@",[NSThread currentThread]);
        NSLog(@"执行完毕");
    });
    
    NSLog(@"code end");
}
```

执行结果：

``` objc
test2:
2016-03-10 17:07:10.366 GCDSample[3879:2235420] code begin
 2016-03-10 17:07:10.367 GCDSample[3879:2235420] currentThread0 <NSThread: 0x14c50c350>{number = 1, name = main}
 2016-03-10 17:07:10.367 GCDSample[3879:2235420] code end
 2016-03-10 17:07:10.472 GCDSample[3879:2235438] ___0 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:10.577 GCDSample[3879:2235438] ___1 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:10.683 GCDSample[3879:2235438] ___2 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:10.784 GCDSample[3879:2235438] ___3 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:10.890 GCDSample[3879:2235438] ___4 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:10.995 GCDSample[3879:2235438] ___5 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:11.101 GCDSample[3879:2235438] ___6 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:11.206 GCDSample[3879:2235438] ___7 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:11.309 GCDSample[3879:2235438] ___8 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:11.411 GCDSample[3879:2235438] ___9 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:11.411 GCDSample[3879:2235438] currentThread1 <NSThread: 0x14c692ed0>{number = 2, name = (null)}
 2016-03-10 17:07:11.411 GCDSample[3879:2235438] 执行完毕

```

####3，dispatch_barrier_async，分割多任务线程

如果有10个并发任务，想要分成两组，比如指定前五个任务执行完之后，优先执行第六个任务，再执行剩下的操作，可以通过dispatch_barrier_async来分割开，示例：

``` objc
- (void)test3
{
    dispatch_queue_t concurrentQueue = ({
        dispatch_queue_t queue = dispatch_queue_create("concurrentQueue", DISPATCH_QUEUE_CONCURRENT);
        queue;
    });
    for (int i = 0; i < 10; i++) {
        if (i != 5) {
            dispatch_async(concurrentQueue, ^{
                [NSThread sleepForTimeInterval:arc4random_uniform(3)];
                NSLog(@"dispatch_async %i", i);
            });
        }else{
            dispatch_barrier_async(concurrentQueue, ^{
                NSLog(@"dispatch_barrier_async");
            });
        }
    }
}

```

执行结果：

``` objc
test3
2016-03-10 17:47:32.397 GCDSample[3949:2246479] dispatch_async 1
 2016-03-10 17:47:32.400 GCDSample[3949:2246481] dispatch_async 3
 2016-03-10 17:47:33.403 GCDSample[3949:2246480] dispatch_async 2
 2016-03-10 17:47:34.400 GCDSample[3949:2246477] dispatch_async 0
 2016-03-10 17:47:34.405 GCDSample[3949:2246479] dispatch_async 4
 2016-03-10 17:47:34.406 GCDSample[3949:2246477] dispatch_barrier_async
 2016-03-10 17:47:34.406 GCDSample[3949:2246479] dispatch_async 6
 2016-03-10 17:47:35.411 GCDSample[3949:2246479] dispatch_async 7
 2016-03-10 17:47:35.412 GCDSample[3949:2246480] dispatch_async 9
 2016-03-10 17:47:35.411 GCDSample[3949:2246477] dispatch_async 8

```

####4，串行队列死锁

``` objc
- (void)test4
{
    NSLog(@"1");
    dispatch_sync(dispatch_get_main_queue(), ^{
        NSLog(@"2");
    });
    NSLog(@"3");
}

- (void)test4_1 {
    NSLog(@"1");
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        NSLog(@"2 %@",[NSThread currentThread]);
        dispatch_sync(dispatch_get_main_queue(), ^{
            NSLog(@"3 %@",[NSThread currentThread]);
        });
        NSLog(@"4 %@",[NSThread currentThread]);
    });
    NSLog(@"5");
}
```

方法test4会造成死锁，因为main queue需要等待dispatch_sync函数中block返回才能继续执行，而通过dispatch_sync放入main queue的block按照FIFO的原则（先入先出）现在得不到执行，就造成了相互等待的局面，产生死锁。将同步的串行队列放到另外一个异步线程就能够解决（如方法test4_1所示）。所以在使用dispatch_sync的时候需要很谨慎，需要先执行`[NSThread isMainThread]`判断下当前任务是否在mian queue中调用，比如有一段代码在后台执行，而它需要从界面控制层获取一个值。那么你可以使用dispatch_sync简单办到。执行结果：
``` objc
test4
 2016-03-10 17:23:30.020 GCDSample[3896:2239119] 1

test4_1
 2016-03-11 17:30:11.900 GCDSample[1197:730552] 1
 2016-03-11 17:30:11.901 GCDSample[1197:730552] 5
 2016-03-11 17:30:11.903 GCDSample[1197:730564] 2 <NSThread: 0x170263080>{number = 2, name = (null)}
 2016-03-11 17:30:11.933 GCDSample[1197:730552] 3 <NSThread: 0x174075400>{number = 1, name = main}
 2016-03-11 17:30:11.933 GCDSample[1197:730564] 4 <NSThread: 0x170263080>{number = 2, name = (null)}
```


####5，dispatch_semaphore_t 控制并发线程数

有时在执行多任务时需要避免抢占资源以及性能过多消耗的情况，需要在特定时间内控制同时执行的任务数量，在NSOperationQueue可以通过maxConcurrentOperationCount来控制，在GCD中可以指定semaphore来控制了。先介绍3个函数，dispatch_semaphore_create创建一个semaphore；dispatch_semaphore_wait等待信号，当信号总量少于0的时候就会一直等待，否则就可以正常的执行，并让信号总量-1；dispatch_semaphore_signal是发送一个信号，自然会让信号总量加1。看一个小例子：

``` objc
- (void)test7
{
    _semaphore = dispatch_semaphore_create(2);				
    [self task7_1];
    [self task7_2];
    [self task7_3];
    [self task7_4];
}

- (void)task7_1
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"7_1_begin");
        sleep(4);
        NSLog(@"7_1_end");
        dispatch_semaphore_signal(_semaphore);
    });
}

- (void)task7_2
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"7_2_begin");
        sleep(4);
        NSLog(@"7_2_end");
        dispatch_semaphore_signal(_semaphore);
    });
}

- (void)task7_3
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"7_3_begin");
        sleep(4);
        NSLog(@"7_3_end");
        dispatch_semaphore_signal(_semaphore);
    });
}

- (void)task7_4
{
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        dispatch_semaphore_wait(_semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"7_4_begin");
        sleep(4);
        NSLog(@"7_4_end");
        dispatch_semaphore_signal(_semaphore);
    });
}
```

执行结果：

``` objc
 semaphore 为 1
 
 2016-03-11 16:33:11.957 GCDSample[22394:2943718] 7_1_begin
 2016-03-11 16:33:15.959 GCDSample[22394:2943718] 7_1_end
 2016-03-11 16:33:15.960 GCDSample[22394:2943717] 7_2_begin
 2016-03-11 16:33:19.964 GCDSample[22394:2943717] 7_2_end
 2016-03-11 16:33:19.965 GCDSample[22394:2943721] 7_3_begin
 2016-03-11 16:33:23.970 GCDSample[22394:2943721] 7_3_end
 2016-03-11 16:33:23.970 GCDSample[22394:2943727] 7_4_begin
 2016-03-11 16:33:27.975 GCDSample[22394:2943727] 7_4_end

 ------- ------- ------- ------- ------- ------- ------- ----
 
 semaphore 为 2
 
 2016-03-11 16:33:47.396 GCDSample[22406:2944975] 7_2_begin
 2016-03-11 16:33:47.396 GCDSample[22406:2944974] 7_1_begin
 2016-03-11 16:33:51.398 GCDSample[22406:2944975] 7_2_end
 2016-03-11 16:33:51.398 GCDSample[22406:2944974] 7_1_end
 2016-03-11 16:33:51.398 GCDSample[22406:2944976] 7_3_begin
 2016-03-11 16:33:51.398 GCDSample[22406:2944983] 7_4_begin
 2016-03-11 16:33:55.401 GCDSample[22406:2944976] 7_3_end
 2016-03-11 16:33:55.401 GCDSample[22406:2944983] 7_4_end
```

可以看到，并发执行任务的数量取决于`_semaphore = dispatch_semaphore_create(2);`传入的数字，当为1时，效果如同执行串行队列。

####6，dispatch_set_target_queue 设置队列优先级
如果有串行队列A和并行队列B，队列A中加入任务1，队列B中加入任务2、任务3，如果确保1、2、3顺序执行呢？可以通过dispatch_set_target_queue设置队列的优先级，将队列AB指派到队列C上，任务123将会在串行队列C中顺序执行。代码如下：

``` objc
- (void)test8
{
    dispatch_queue_t serialQueue = dispatch_queue_create("com.starming.gcddemo.serialqueue", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t firstQueue = dispatch_queue_create("com.starming.gcddemo.firstqueue", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t secondQueue = dispatch_queue_create("com.starming.gcddemo.secondqueue", DISPATCH_QUEUE_CONCURRENT);
    
    dispatch_set_target_queue(firstQueue, serialQueue);
    dispatch_set_target_queue(secondQueue, serialQueue);
    
    dispatch_async(firstQueue, ^{
        NSLog(@"1 %@",[NSThread currentThread]);
        [NSThread sleepForTimeInterval:3.f];
    });
    dispatch_async(secondQueue, ^{
        NSLog(@"2 %@",[NSThread currentThread]);
        [NSThread sleepForTimeInterval:2.f];
    });
    dispatch_async(secondQueue, ^{
        NSLog(@"3 %@",[NSThread currentThread]);
        [NSThread sleepForTimeInterval:1.f];
    });
}
```
执行结果：
``` objc
 2016-03-11 17:31:41.515 GCDSample[1202:730942] 1 <NSThread: 0x170078340>{number = 2, name = (null)}
 2016-03-11 17:31:44.518 GCDSample[1202:730942] 2 <NSThread: 0x170078340>{number = 2, name = (null)}
 2016-03-11 17:31:46.520 GCDSample[1202:730942] 3 <NSThread: 0x170078340>{number = 2, name = (null)}
```
未完待续，Have fun!

参考：
https://github.com/nixzhu/dev-blog/blob/master/2014-04-19-grand-central-dispatch-in-depth-part-1.md