---
title: swift消息派发机制是怎么样？
author: huhansome
date: 2021-12-29 17:38:00 +0800
categories: [流弊技能]
tags: [iOS, swift, 消息派发, 消息转发, runtime]
---

swift是苹果公司开发的编程语言，他的特点是快，精简，安全。如果你使用过swift，你会发现它和OC有着很大的差距，它和很多的脚本语言有一些相似，今天我们讨论的是swift语言和OC在消息派发的差距。

众所周知Objective-C是一门动态语言，之所以是动态语言，是因为强大的runtime的支撑，OC的消息转发是动态的，是在运行的时候去决议调用哪个方法，这样达到了灵活的好处，但是也会带来问题，比如性能问题，安全问题等。那么swift的消息派发有什么不通过呢？

swift是一门静态语言，所以他的速度会比OC块，swift消息派发比较复杂，会在考虑性能的同时考虑安全性。swift语言消息派发有如下方式：

-  值类型永远使用direct dispatch， 比如结构体 枚举
-  在protocol和class的定义中声明的方法使用table dispatch
-  在protocol和class的extension中定义的方法使用direct dispatch
-  继承nsobject的对象 定义里面申明的方法还是走函数表，extension里面的方法走消息转发

所以swift的消息机制还是比较复杂的，还有一些修饰可以改变swift消息派发的方式，比如`dynamic`修饰变成消息转发，使用`final / inline`可以是方法变为直接调用。另外`@objc` 和 `@nonobjc` 显示声明一个函数能否被 Objective-C 的运行时捕获到，不会修改方法的派发机制。所以你学会了吗？