---
title: strong weak dance是什么意思？
author: huhansome
date: 2021-12-26 17:38:00 +0800
categories: [流弊技能]
tags: [iOS, block, 循环引用, strong, weak]
---

在说strong weak dance之前，我们先了解一下block, block在其他很多语言中也是存在的，只是可能不是叫这个名字，比如在js语言可能叫做匿名函数，在Java里面叫做lambda表达式。在iOS中，block作为非常常用的一种数据结构，使用的时候需要注意循环引用。


# 防止循环引用

一般情况，如果controller（self）引用了block,然后在block内部引用了self，那么这种情况就会导致循环引用。

```objectivec
ViewController *controller = [[ViewController alloc] init];
ViewController * __weak weakViewController = controller;
controller.completionHandler = ^(NSInteger result) {
   [weakViewController dismissViewControllerAnimated:YES completion:nil];
};
```
在大多数情况下，block的使用都是这样使用的，但是其实会发生问题，你知道为什么吗？因为在ARC环境下，weak变量的生命周期是不确定的，你不知道它会在什么时候被释放，所以像上面的使用可以说是不安全， 不严谨的使用。虽然平时这样使用没怎么出现问题，但是毕竟程序上来说是有bug的。那你可能会说，我在使用的时候判断一下是否被释放总行了吧。


```objectivec
ViewController *controller = [[ViewController alloc] init];
ViewController * __weak weakViewController = controller;
controller.completionHandler = ^(NSInteger result) {
    if (weakViewController != nil) {
        
    }  
    [weakViewController dismissViewControllerAnimated:YES completion:nil];
};
```

其实这样写如果是单线程编程的话，那就没问题，但是这是多线程编程，不是线程安全的，所以也是不行的

# strong weak dance

我们看看一些开源库里面，strong weak dance是如何使用的。


```objectivec
__weak __typeof(self)weakSelf = self;
    AFNetworkReachabilityStatusBlock callback = ^(AFNetworkReachabilityStatus status) {
    __strong __typeof(weakSelf)strongSelf = weakSelf;

    strongSelf.networkReachabilityStatus = status;
    if (strongSelf.networkReachabilityStatusBlock) {
        strongSelf.networkReachabilityStatusBlock(status);
    }

};
```

先是用weakSelf来弱引用self，打断循环引用，接着为了在使用的时候保证weakself还在，于是用strong来延长weakself的生命周期，然后用strongself执行，这样就可以保证strongself在block里执行的时候不会是空指针的情况。看起来是没问题的，但是如果weakself在赋值给strongself之前就已经被释放了呢？


# 苹果官方做法

看看苹果官方的代码，会发现如下的代码:



```objectivec
MyViewController *myController = [[MyViewController alloc] init…];
// ...
MyViewController * __weak weakMyController = myController;
myController.completionHandler = ^(NSInteger result) {
    MyViewController *strongMyController = weakMyController;
        if (strongMyController) {
            // ...
            [strongMyController dismissViewControllerAnimated:YES completion:nil];
            // ...
        }
        else {
            // Probably nothing...
        }
};
```

if (strongMyController) 是这段代码的亮点。之所以在Block的代码执行之前加上这样一个判断，就是为了防止在把 weakSelf 转换成 strongSelf 之前 weakSelf 就已经为 nil 了，这样才能确保万无一失。