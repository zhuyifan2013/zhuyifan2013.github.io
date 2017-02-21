title: React Native  集成 unity3D 场景 - iOS
date: 2017-02-21 20:27:50
tags: 
- react native
- unity
- ios
---

# 需求
最近开始啦一个新的项目，需求是：通过 Unity 3D 制作场景等，同时 iOS 上也会有自己的 2D 界面，两者之间可以相互跳转。
通过两天的学习和尝试，最终成功的在 React Native 项目（以下简称 RN 项目）中集成了 Unity3D 的导出项目（称作 Native 或者 Unity 项目），现记录一下踩坑过程。

# 思路
一开始便有两种思路：
1. 拷贝 Unity 项目的文件到 RN 项目中，修改 RN 项目的配置。（以 RN 项目为主）
2. 在 Unity 项目的基础上，新建 RN 项目所需的文件。（以 Unity 项目为主）

# 实现
首先选择的是方案一，主要考虑到我想以 2D 界面为主，Unity 项目为辅，基于这个想法才想将各种文件都拷贝至已经生成的 RN 项目中，然后开始修改 Build Setting，添加各种依赖，但是由于我的 Unity3D 项目比较复杂，里边集成了 Google Cardboard SDK，还有很多第三方SDK，所以导致最终一直有几个方法找不到，参见我在 Unity 社区提出的[问题](http://answers.unity3d.com/questions/1315885/integrate-vr-project-with-xcode-project-some-metho.html)，尝试了各种办法之后放弃了，如果你有解决办法，还请告知。

最终选择的是方案二，这里我描述一下集成过程：

* 通过 Unity 导出 iOS 项目
* 新建 项目文件夹，按如下架构组织，ios 文件夹中放的是 Unity导出的所有文件，package.json 为初始化文件，index.ios.js 为 RN 项目的文件。此部分可以参考 RN 的[官方文档](https://facebook.github.io/react-native/docs/integration-with-existing-apps.html), 根据文档一直操作到 `Create a index.ios.js file` 部分，后面的可以暂时不用管。
![项目文件组织](/img/2017-02-21/project_tree.png)

> 在这个过程中 cocoaPod 有一个坑，那就是下载速度特别的慢，经过查阅各种资料，所谓的换源什么的方法对我都没有什么用，我是直接用 `git clone https://github.com/CocoaPods/Specs.git >~/.cocoapods/repos/master` 的方式看着一点点下载完然后更新再 `pod install`的，前后大约耗时 30 分钟，请耐心等待

* `pod install` 执行成功后，切记打开`.xcworkspace` 文件，而不是原来的 .xcodeproj  文件，此时应该可以build成功

> 这里有另外一个坑：由于我之前自己有改过 Build Setting 的 `Other C Flags` 设置，所以pod install的时候，pod需要添加的项没有添加进来，而其实 pod 是有 warning 的，却被我忽略了，解决办法是在这个设置上添加 `${inherited}`即可，之后执行各种命令，warning 也是不能忽略的。

集成成功后，下一步要解决的问题是跳转问题，由于项目刚刚开始做，我先做了一个简单的页面模拟跳转过程，尝试是否能跳转成功。这里边涉及的功能点是：

1. RN 页面调用 Native 代码
2. Native 页面跳转 Unity 页面，之所以一定要实现这样的功能，是考虑到项目以后UI部分全部都会用 RN 去写，而Unity项目生成的为 Native 的页面，所以找到其桥梁势是很有必要的，而不是只把RN页面作为Native的简单嵌套（尽管RN的实质是这样的）

#### 1. RN 页面如何调用 Native 代码？
首先请参考[官方文档参考](https://facebook.github.io/react-native/docs/native-modules-ios.html)
这里的重点是要实现 `RCTBridgeModule` ，将方法暴露给RN端。
在`ios/`下新建 `NavigateBridge.m` 与 `NavigateBridge.h` , 内容如下：

```objc
// NavigateBridge.m 
#import "NavigateBridge.h"
#import "UnityAppController.h"

@implementation NavigateBridge

RCT_EXPORT_MODULE();

RCT_EXPORT_METHOD(startUnity)
{
    RCTLogInfo(@"I will start Unity!");
    dispatch_async(dispatch_get_main_queue(), ^{
    [[NSNotificationCenter defaultCenter]postNotificationName:@"openUnity" object:nil];
    });

}

@end
```

```objc
// NavigateBridge.h 
#import <React/RCTBridgeModule.h>
#import <React/RCTLog.h>

@interface NavigateBridge : NSObject<RCTBridgeModule>

@end
```

根据官方文档的说明，`startUnity` 为暴露给 RN 端的方法，所以在 index.ios.js 的代码如下：重点是 NavigateManager 的使用

```javascript
import React, {Component} from 'react';
import {
    AppRegistry,
    StyleSheet,
    Text,
    View,
    Alert,
    Button,
    NativeModules
} from 'react-native';


var NavigateManager = NativeModules.NavigateBridge;

const onButtonPress = () => {
    NavigateManager.startUnity();
};


export default class untitled extends Component {
    render() {
        return (
            <View style={styles.container}>
                <Text style={styles.welcome}>
                    Welcome to React Native!哈哈哈哈哈·12345！
                </Text>
                <Text style={styles.instructions}>
                    To get started, edit index.ios.js 不错的
                </Text>
                <Button onPress={onButtonPress}
                        title="Press me!"
                />
                <Text style={styles.instructions}>
                    Press Cmd+R to reload,{'\n'}
                    Cmd+D or shake for dev menu
                </Text>
            </View>
        );
    }
}


const styles = StyleSheet.create({
    container: {
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#F5FCFF',
    },
    welcome: {
        fontSize: 20,
        textAlign: 'center',
        margin: 10,
    },
    instructions: {
        textAlign: 'center',
        color: '#333333',
        marginBottom: 5,
    },
});

AppRegistry.registerComponent('moonplayer', () => untitled);
```

至此，可以实现 RN 页面调用 Native 的方法，不过此时运行起来还是直接进入了Unity页面，请继续往下看。

#### 2. Native 如何调起 Unity 界面？
我的解决方案是不修改程序的启动入口，而直接干预 unity 的启动过程。
ios/Classes 文件夹中，`main.mm` 为整个App的入口，可以看到直接启动了 UnityAppController ，在该目录下找到 `UnityAppController.mm`, 首先我们要注意到这是一个 UIApplicationDelegate 的对象（参见头文件定义），所以其实它承担的就是平时我们熟悉的 AppDelegate的角色。
在头文件中可以看到很多重要的定义，比如 UIWindow，UIViewController 等，有些目前用不到，但是说不定以后需要用，所以需要注意。

我们注意到在mm文件中，`applicationDidBecomeActive` 方法里有调用 startUnity 方法，根据网上资料可以得知，startUnity 是打开 Unity界面的 函数，所以我们要干预的就是调用此方法的地方。
找到

```objc
          _startUnityScheduled = true;
          [self performSelector:@selector(startUnity) withObject:application afterDelay:0];
```

将 startUnity 替换为 `startMainPage`，`withObject:application` 也记得替换 该方法定义如下：

```objc
- (void)startMainPage {
    NSURL *jsCodeLocation;

    jsCodeLocation = [NSURL URLWithString:@"http://10.1.20.47:8081/index.ios.bundle?platform=ios"];
    RCTRootView *rootView = [[RCTRootView alloc] initWithBundleURL:jsCodeLocation
                                                        moduleName:@"moonplayer"
                                                 initialProperties:nil
                                                     launchOptions:nil];
    rootView.backgroundColor = [[UIColor alloc] initWithRed:1.0f green:1.0f blue:1.0f alpha:1];

    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
    UIViewController *rootViewController = [UIViewController new];
    rootViewController.view = rootView;
    self.window.rootViewController = rootViewController;
    [self.window makeKeyAndVisible];
}
```

http://10.1.20.47:8081  这个地址根据你使用的是模拟器还是真机而更改为 localhost 或者是你电脑的 ip地址，我用的是真机，所以用的是自己电脑的ip地址，同时我也修改了 `RCTWebSocketExecutor.m`，请参考[官方文档](https://reactnative.cn/docs/0.41/running-on-device-ios.html)
根据文档很好理解代码所做的事情，注意的是 modeleName 需要与 index.ios.js 中注册的 module name 一致才可以，至此，程序启动后是先跳转到了我们的 RN 首页。

上面代码的  NavigateBridge.m  已经写好了点击按钮后触发的事件，即发出了一个通知，我们在 UnityAppController.mm 中监听此通知：

在 `didFinishLaunchingWithOptions` 方法的最后添加：

```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(openUnity:) name:@"openUnity" object:nil]; 
```

同时 openUnity 方法的定义如下：

```objc
-(void) openUnity:(NSNotification *)notification{
    NSLog(@"receive notification");
    [self startUnity:[UIApplication sharedApplication]];
}
```

至此即可实现 Native 界面跳转至 Unity3D 的场景，而且加载速度很快









