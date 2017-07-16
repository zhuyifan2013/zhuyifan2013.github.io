title: java的volatile关键字
date: 2016-04-09 17:22:32
tags:
- java
category:
- java
---

volatile的引入是为了解决java并发编程中的可见性与有序性的问题。
java并发编程要注意以下三个问题

* 原子性
* 可见性
* 有序性

同时满足以上三个性质才能解决并发的问题

### java内存模型

Java内存模型规定所有的变量都是存在主存当中（物理内存），每个线程都有自己的工作内存（高速缓存）。线程对变量的所有操作都必须在工作内存中进行，而不能直接对主存进行操作。并且每个线程不能访问其他线程的工作内存。

#### Happens-before原则

#### 原子性
原子性即某个操作CPU可以用一条指令执行完成的，要么执行，要么不执行，不存在中途停下的问题。
如下：

``` java
    x = 4
    x = y
    x++

```

只有第一句为原子操作，通常Java内存模型只保证了基本读取和赋值是原子性操作，变量之间的复制也不是原子操作
#### 可见性
#### 有序性

### volatile做了什么事情？
经过volatile修饰的变量保证了如下两件事：

* 可见性：不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的
* 有序性：禁止进行指令重排序

```
为什么会有指令重排？
因为处理器为了有更高的效率，编译器可能会将单线程下不影响执行效果的代码重排序

```

<!--more-->

需要注意的是，volatile并不能保证原子性，见这个例子：

``` java
public class Test {
    public volatile int inc = 0;
     
    public void increase() {
        inc++;
    }
     
    public static void main(String[] args) {
        final Test test = new Test();
        for(int i=0;i<10;i++){
            new Thread(){
                public void run() {
                    for(int j=0;j<1000;j++)
                        test.increase();
                };
            }.start();
        }
         
        while(Thread.activeCount()>1)  //保证前面的线程都执行完
            Thread.yield();
        System.out.println(test.inc);
    }
}
```

test.inc的值并不是期望中的10000

### volatile是怎样保证可见性与原子性的？

加入volatile关键字和没有加入volatile关键字时所生成的汇编代码发现，加入volatile关键字时，会多出一个lock前缀指令
lock前缀指令实际上相当于一个内存屏障（也成内存栅栏），内存屏障会提供3个功能：
1）它确保指令重排序时不会把其后面的指令排到内存屏障之前的位置，也不会把前面的指令排到内存屏障的后面；即在执行到内存屏障这句指令时，在它前面的操作已经全部完成；
2）它会强制将对缓存的修改操作立即写入主存；
3）如果是写操作，它会导致其他CPU中对应的缓存行无效。

### 参考资料
[Java并发编程：volatile关键字解析](http://www.cnblogs.com/dolphin0520/p/3920373.html)