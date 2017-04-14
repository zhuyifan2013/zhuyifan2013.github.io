title: OpenGL ES 3.0 绘制三角形
date: 2017-04-14 17:43:02
tags:
- opengl
- ios
---
本文实现 在 iOS 平台上 使用 OpenGL ES 绘制一个简单的三角形 且 不使用 GLKit，原因是方便跨平台共享代码，也易于理解更底层的知识.

源代码参见 [Github OpenGLES3](https://github.com/zhuyifan2013/OpenGLES3)

使用的各组件信息如下：
>OpenGL ES : 3.0
iOS : 10.2
Device : iPhone 6s 模拟器
xCode : 8.3 
除了EGL的版本信息之外，其他的即使使用其他版本应该无太大关系，需要注意 从 iOS 7.0 之后才支持 3.0 版本。

### 重点代码分析

* 创建EAGL的 layer ，EAGL 可以理解为 Apple 的 EGL(Embedded GL) 实现

```objc
+(Class)layerClass {
    return [CAEAGLLayer class];
}
```

* 创建 EAGL 的 context

```objc
context = [[EAGLContext alloc] initWithAPI:kEAGLRenderingAPIOpenGLES3]; 
```

* 初始化 layer 

```objc
CAEAGLLayer *layer = (CAEAGLLayer *)self.layer;
layer.opaque = YES;
layer.contentsScale = [UIScreen mainScreen].scale;
```

* 接下来是 初始化 renderBuffer 和 frameBuffer，关于 frameBuffer 的信息参考 [苹果官方文档](https://developer.apple.com/library/content/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/WorkingwithEAGLContexts/WorkingwithEAGLContexts.html#//apple_ref/doc/uid/TP40008793-CH103-SW1)

* 下一步绑定(attach) vertextShader 与 fragmentShader 的信息，入门的时候可以先不用理解shader代码的含义

* 最终交换前后端帧缓冲区

```objc
[context presentRenderbuffer:GL_RENDERBUFFER];
```

### 参考资料：

1. [OpenGL ES 3.0 数据可视化](http://www.jianshu.com/p/9ece99f1adda)

2. [Apple OpenGL ES Programming Guide](https://developer.apple.com/library/content/documentation/3DDrawing/Conceptual/OpenGLES_ProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008793-CH1-SW1)

3. OpenGL ES 3.0 Programming Guide (2nd Edition)



