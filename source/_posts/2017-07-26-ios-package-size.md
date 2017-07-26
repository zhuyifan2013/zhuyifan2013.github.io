title: 缩减 iOS 包体积（Unity）
date: 2017-07-26 23:08:46
tags: 
- ios
- unity
---

本文提供查看 iOS 包体积的方法，分两部分进行研究，具体缩减包体积的方法会在以后的文章介绍：

### 检查包体积的大小
真正在 AppStore 中显示的下载体积可以在 `iTunes Connect -> 我的 App -> 活动 -> 构建版本` 中查看 , 点击 `App Store 文件大小` 即可看到各个机型的文件大小。

![appstore-size](/img/2017-07-26/appstore-size.png)
![appstore-size-2](/img/2017-07-26/appstore-size-2.png)

### 如何查看具体的体积信息
在编译完成后的 `Products` 文件夹中右键可以看到生成的应用程序文件，右键点击显示包文件内容，显示的即为所有的包文件内容，但是可以发现其中还有一个 Unix executable 的文件，名字与 product 的名字一样，我这里这个文件名为 moonvrplayer，我们需要对这个文件再做具体的分析，这时候可以用 一个 `LinkMap Parser `的工具帮助分析该文件的组成

![xcode-preview](/img/2017-07-26/xcode-preview.png)

1. XCode 开启编译选项 Write Link Map File
XCode -> Project -> Build Settings -> 搜map -> 把Write Link Map File选项设为yes，并指定好 linkMap 的存储位置
2. 编译后，到编译目录里找到该 txt 文件，文件名和路径就是上述的Path to Link Map File
位于~/Library/Developer/Xcode/DerivedData/XXX-eumsvrzbvgfofvbfsoqokmjprvuh/Build/Intermediates/XXX.build/Debug-iphoneos/XXX.build/, 这个LinkMap里展示了整个可执行文件的全貌，列出了编译后的每一个.o目标文件的信息（包括静态链接库.a里的），以及每一个目标文件的代码段，数据段存储详情。

分析此文件的工具我们可以使用 [LinkMap Parser](https://gist.github.com/zhuyifan2013/61a52f409f8f56b61eaf6770f613ff17)

命令如下：

``` node
node size.js /Users/rockvr/Desktop/moonvrplayer-LinkMap-normal-arm64.txt  -hl > ~/Desktop/size.txt
```

`size.js` 为 gist的这个文件，`moonvrplayer-LinkMap-normal-arm64.txt` 为 linkmap 文件，`~/Desktop/size.txt` 为输出的文件

可以看出所使用的库的体积，针对比较大的文件进行缩减
![size-txt](/img/2017-07-26/size-txt.png)

### Unity Assets 大小查看
针对 Unity 资源的大小，我们可以参考[官方文档的说明](https://docs.unity3d.com/Manual/ReducingFilesize.html)
打开 `Editor Log` ，查看所有 asset 的大小并进行针对性的缩减

![unity-asset-size](/img/2017-07-26/unity-asset-size.png)


