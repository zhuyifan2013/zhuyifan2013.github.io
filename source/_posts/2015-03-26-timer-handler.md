title: Handler和Timer作为计时器的优劣
date: 2015-03-26 20:56:44
tags: 
-  android
categories: android

---

Android中可以采用`Timer＋Handler`或者单纯使用`Handler`的方法来实现定时通知的功能，但是采用Handler的方法会更好，优点不仅仅在于代码量会少很多，更主要在使用于Timer会产生bug。

### 使用Timer＋Handler的方式
这种方式在TimerTask的`run`方法中sendMessage给Handler，实现定时操作的目的，但是使用Timer本身会出现一种问题，看下面的实验代码：

``` java
	TimerTask timerTask = new TimerTask() {
		@Override
		public void run() {
			print("first");
			try {
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			print("second");
		}
	};
	
	Timer timer = new Timer();
	timer.schedule(timerTask, 0, 1000);
```

愿意是想立即执行一次这样的任务：先输出`first`，隔2秒再输出`second`，然后再隔1秒，再执行一次同样的任务，但是当执行过后就会发现，`second`输出完之后会马上输出`first`，并没有延迟1秒，问题就在于timer的时间延迟是从任务开始执行时就已经计算了，并不是上一次任务执行完成后才计算的，因此这个任务执行了2秒，其实第二次任务已经开始执行了，但是是单队列，所以并没有马上出现，但是下一次已经不会再如我们所愿的延迟1秒了(但是它其实也按照它自己的方式延迟了1秒，也就是它不在乎任务有没有执行完，我自己发出命令就开始计算1秒，然后再发出命令)，因此这种方式可能会出现问题。

### 只使用Handler的方式
这种方式既简单也没有风险:

	handler.sendEmptyMessageDelayed(MSG_TIME_COUNT, 1000);
	
一句代码即可搞定，会延迟1秒发送消息给`handleMessage`来处理，而且每次是等任务执行完才会开始延迟。