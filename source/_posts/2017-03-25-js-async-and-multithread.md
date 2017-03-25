title: React Native 异步与多线程
date: 2017-03-25 15:51:12
tags:
- react native
- javascript
---

# JS thread 和 Main thread
React Native 中有两个线程值得注意：
一个是 `JavaScript thread`， 一个是 `Main Thread`。
JavaScript Thread 上会运行大部分的逻辑，也是 React 应用运行的地方，会处理各种API调用，事件反馈等等。
Main Thread 则是 Native 端 （iOS，Android）通常意义上的 UI Thread。
我在这里将 JavaScript Thread 理解为 Native 端意义上的一个普通的非UI线程。
关于 React Native 线程的更多介绍，请参见[官方文档](https://facebook.github.io/react-native/docs/performance.html#js-frame-rate-javascript-thread)

# 异步的问题
首先异步不代表多线程，简单的理解异步其实就是改变了代码的执行顺序，但是并不意味改变了代码的执行环境。
Promise就是异步的一种方式，这里不详细介绍Promise的使用，具体使用请参见：[Promise迷你书](http://liubin.org/promises-book/)
我在初识JS与React Native的时候，潜意识的认为异步执行的代码就在非JS线程中（后台线程），因此做了很多无用功去排查问题，具体的Case是我进入一个页面，需要获取一些数据，然后将数据填充在ListView上，获取数据的过程比较长，十几秒的时间，因此造成了每次进入页面就会卡顿一会儿。一开始很纳闷为什么我已经使用Promise去做了这件事，还是阻碍了JS线程的页面绘制，直到最后意识到Proimise只是实现了异步，并非多线程。
最终的解决办法是在iOS端使用`GCD`，新建后台线程，运行完毕后再通知给 JS端，这样才真正用非JS线程去做一些heavy的工作。

# 其他参考
[JS是单线程的深入分析](http://www.cnblogs.com/Mainz/p/3552717.html)