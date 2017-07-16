title: Gradle 学习
date: 2016-04-07 18:36
tags: 
- tools
- android
category:
- tools
---

gradle wrapper 最主要的目的就是方便其他人更方便的快速编译整个项目，不用担心各种版本的不同

gradle wrapper 生成的文件有

> *   gradlew (Unix Shell script)
> *   gradlew.bat (Windows batch file)
> *   gradle/wrapper/gradle-wrapper.jar (Wrapper JAR)
> *   gradle/wrapper/gradle-wrapper.properties (Wrapper properties)

**这些文件一定要提交到VCS中**

生成的方法是使用 `gradle wrapper --gradle-version 2.0` 命令，自己也可以个性化这个task，更多个性化的内容参见[Wrapper](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.wrapper.Wrapper.html)的API文档

``` java
task wrapper(type: Wrapper) {
    gradleVersion = '2.0'
}
```

此后可以直接使用 `./gradlew XXX` 进行编译，可以将`gradlew`看成一个包有gradle命令的壳

需要注意的是，如果你在使用其他人生成好的gradle项目的时候，初次运行`./gradlew build` 会下载对应版本的gradle，下载的文件会存放在` ~/.gradle/wrapper/dists `下

<!-- more -->