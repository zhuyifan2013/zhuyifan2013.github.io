title: 背景颜色随滑动而渐变的View
date: 2015-05-19 21:40:30
tags: 
- analysis
- android
category: Analysis
---
**名称**: GuideBackgroundColorAnimation

**分类**: `动画`

**项目地址**: https://github.com/TaurusXi/GuideBackgroundColorAnimation

**功能**: view的背景色能随着滑动而渐变, 适合引导页的设计,稍作改动也可用于栏目的切换.

**分析**:	

这个小功能没有特别复杂的地方, 主要是在ViewPager的监听函数`onPageScrolled()`这里实现了颜色的改变.分析下过程吧:
1.	自定义一个View---ColorAnimationView	
2.	为view设置一各Viewpager, 并重写`ViewPager.OnPageChangeListener`	
3.	重点分析`onPageScrolled()`:	

```java
int count = getViewPagerChildCount() - 1;
if (count != 0) {
    float length = (position + positionOffset) / count;
    int progress = (int) (length * DURATION);
        ColorAnimationView.this.seek(progress);
}
```

先得到页面数量`count`, 根据每一次的`position`和`positionOffset`计算得到`length`, `position`是从0开始计算, 最大值为`count - 1`, 如果有`n`个子页面, `position`的最大值就是`n-1`. 而`positionOffset`则是一个取值范围在[0,1]之间的值, 对于一个页面来说, 滑动到一半, `positionOffset`的值就是0.5, 全部滑过来的那一瞬间, 取值为1, 当然也是下一个新的`position`的0了. 因此length可以想象成一个取值范围为`[0,count]/count`(也就是`[0,1]`)之间的浮点数, 能反映全部过程中的某一个时间点.	
根据这个百分比,再乘以总的时间`DURATION`就知道目前完成了整个动画的百分比了, 也就是`progress`, 随后调用`seek`方法.	


```java
private void seek(long seekTime) {
    if (colorAnim == null) {
        createDefaultAnimation();
    }
    colorAnim.setCurrentPlayTime(seekTime);
}
  
private void createDefaultAnimation() {
    colorAnim = ObjectAnimator.ofInt(this,"backgroundColor", WHITE, RED, BLUE, GREEN, WHITE);
    colorAnim.setEvaluator(new ArgbEvaluator());
    colorAnim.setDuration(DURATION);
    colorAnim.addUpdateListener(this);
}
```

这里面如果自己没有传入色彩定义, 就用默认的渐变过程`WHITE, RED, BLUE, GREEN, WHITE`, 使用的`Evaluator`是`ArgbEvaluator`, 总的时间为`DURATION`.	
随后调用了`colorAnim`的方法`setCurrentPlayTime()`, 这个方法的作用是播放传入的这个时刻的动画.因为传入的是不同时刻的动画进度, 当然连在一起也就是一个完整的动画了, 由此就实现了最初的目的.	


**深入学习**
1.	`Evaluator`和`Interpolator`	
2.	`ValueAnimator`的原理	
3.	`setCurrentPlayTime()`的分析, 以及动画的播放原理	

**参考资料**
1.	[AnimationEasingFunctions](https://github.com/daimajia/AnimationEasingFunctions)
2.	[Property Anim详解](http://blog.csdn.net/xushuaic/article/details/40424379/)
3.	[Android属性动画源代码解析（超详细）](http://www.cnblogs.com/kissazi2/p/4249213.html)