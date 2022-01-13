---
title: iOS使用AFNetworking如何实现同步请求
author: huhansome
date: 2021-12-28 17:38:00 +0800
categories: [流弊技能]
tags: [iOS, 信号量, 锁, dispatch_semaphore, 锁, 线程安全, 同步]
---

iOS开发中，很重要的一个环节就是网络请求，AFNetworking作为目前最流行的异步网络请求库，可能没有iOS开发者不熟悉这个框架。AFNetworking2.0是基于NSURLConnection实现，NSURLConnection3.0是基于NSURLSession。平时开发的时候，基本上都是异步请求的需求大于同步，一般网络请求都是异步的，然而有时候我们也需要同步的网络请求，所以如何实现同步的网络请求也是我们要考虑的问题。

在iOS9.0之前，我们的网络请求主要是依赖NSURLConnection，我们可以使用系统的API就能实现同步请求。
```
+ (nullable NSData *)sendSynchronousRequest:(NSURLRequest *)request returningResponse:(NSURLResponse * _Nullable * _Nullable)response error:(NSError **)error API_DEPRECATED("Use [NSURLSession dataTaskWithRequest:completionHandler:] (see NSURLSession.h", macos(10.3,10.11), ios(2.0,9.0), tvos(9.0,9.0)) __WATCHOS_PROHIBITED;
```

但是这个方法已经过期了，我们现在基于NSURLSession，我们应该怎么实现呢？或者说我们现在的项目基于AFNetworking3.0，应该怎么样发送同步网络请求？

我们可以通过信号量来控制，达到同步的需求。
```
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];
    manager.responseSerializer = [AFHTTPResponseSerializer serializer];
    __block id _responseObject = nil;
    
    dispatch_semaphore_t semaphore =  dispatch_semaphore_create(0);
    
    [manager GET:url parameters:params progress:^(NSProgress * _Nonnull downloadProgress) {
        
    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        _responseObject = responseObject;
        dispatch_semaphore_signal(semaphore);
        
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        _responseObject = nil;
        dispatch_semaphore_signal(semaphore);
        
    }];
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    NSLog(@"获取到结果:%@", _responseObject);
```
如果你就像上面一样实现了，就以为成功了，那就错了，运行会发现出现了死锁现象。这是因为AFNetworking默认的回调是在主线程，导致waite和signal相互等待，从而导致死锁。所以我们只需要将AFNetworking的回调线程设置为其他线程即可解决。

```
manager.completionQueue = dispatch_get_global_queue(0, DISPATCH_QUEUE_PRIORITY_DEFAULT);
```
除了以上使用信号量来实现以外，我们还可以用GCD group实现（其实还是信号量），以及operation或者NSCondition等。
