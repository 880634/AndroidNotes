---
title: Android动画之视图动画
date: 2016-7-12 18:59:25
categories: Android
tags: Android
---

# 概述
> 
• 操作某个控件让其展现出旋转、渐变、移动、缩放的这么一种转换过程。
• 只能应用于View对象，而且只支持一部分属性，如支持缩放旋转而不支持背景颜色的改变。
• 它只是改变了View对象绘制的位置，而没有改变View对象本身。
• 可以以XML形式定义动画，也可以编码实现。不过建议用XML，更具可读性、可重用性。

<!--more-->

# 相关类
![](http://oeiu2t0ur.bkt.clouddn.com/20150613182703583.png)

| java类名 | XML | 描述 |
|------|------|------|
| AlphaAnimation | <alpha> 放置在res/anim/目录下 | 渐变透明度动画效果 |
| RotateAnimation | <rotate> 放置在res/anim/目录下 | 画面转移旋转动画效果 |
| ScaleAnimation | <scale> 放置在res/anim/目录下 | 渐变尺寸伸缩动画效果 |
| TranslateAnimation | <translate> 放置在res/anim/目录下 | 画面转换位置移动动画效果 |
| AnimationSet | <set> 放置在res/anim/目录下 | 一个持有其它动画元素alpha、scale、translate、rotate或者其它set元素的容器
 |

接下来我们就详细一个一个的介绍一下各种补间动画。

`Animation`抽象类是所有补间动画类的基类，所以基类会提供一些通用的动画属性方法，如下我们就来详细看看这些属性
• Animation属性详解（共有属性）

| xml属性 | java方法 | 解释 |
|-----|-----|-----|
| android:detachWallpaper | setDetachWallpaper(boolean) | 是否在壁纸上运行|
| android:duration | setDuration(long) | 动画持续时间，毫秒为单位 |
| android:detachWallpaper | setDetachWallpaper(boolean) | 是否在壁纸上运行|
| android:fillAfter | setFillAfter(boolean) | 控件动画结束时是否保持动画最后的状态|
| android:fillBefore | setFillBefore(boolean) | 控件动画结束时是否还原到开始动画前的状态|
| android:fillEnabled | setFillEnabled(boolean) | 与android:fillBefore效果相同|
| android:interpolator | setInterpolator(Interpolator) | 设定插值器（指定的动画效果，譬如回弹等）|
| android:repeatCount | setRepeatCount(int) | 重复次数 |
| android:repeatMode | setRepeatMode(int) | 重复类型有两个值，reverse表示倒序回放，restart表示从头播放 |
| android:startOffset | setStartOffset(long) | 调用start函数之后等待开始运行的时间，单位为毫秒 |
| android:zAdjustment | setZAdjustment(int) | 表示被设置动画的内容运行时在Z轴上的位置（top/bottom/normal），默认为normal |

上面的这些属性每一种补间动画都具备，下面介绍每一种补间动画特有的属性：

• Alpha属性详解

| xml属性 | java方法 | 解释 |
|------|------|------|
| android:fromAlpha | AlphaAnimation(float fromAlpha, …) | 动画开始的透明度（0.0到1.0，0.0是全透明，1.0是不透明） |
| android:toAlpha | AlphaAnimation(…, float toAlpha) | 动画结束的透明度，同上 |

• Rotate属性详解

| xml属性 | java方法 | 解释 |
|------|------|------|
| android:fromDegrees | RotateAnimation(float fromDegrees, …) | 旋转开始角度，正代表顺时针度数，负代表逆时针度数 |
| android:toDegrees | RotateAnimation(…, float toDegrees, …)| 旋转结束角度，正代表顺时针度数，负代表逆时针度数 |
| android:pivotX | RotateAnimation(…, float pivotX, …) | 缩放起点X坐标（数值、百分数、百分数p，譬如50表示以当前View左上角坐标加50px为初始点、50%表示以当前View的左上角加上当前View宽高的50%做为初始点、50%p表示以当前View的左上角加上父控件宽高的50%做为初始点)|
| android:pivotY | RotateAnimation(…, float pivotY)| 缩放起点Y坐标，同上规律 |

• Scale属性详解

| xml属性 | java方法 | 解释 |
|------|------|------|
| android:fromXScale | ScaleAnimation(float fromX, …) | 初始X轴缩放比例，1.0表示无变化 |
| android:toXScale | ScaleAnimation(…, float toX, …) | 结束X轴缩放比例 |
| android:fromYScale | ScaleAnimation(float fromY, …) | 初始Y轴缩放比例，1.0表示无变化 |
| android:toYScale | ScaleAnimation(…, float toY, …) | 结束Y轴缩放比例 |
| android:pivotX | ScaleAnimation(…, float pivotX, …) | 缩放起点X坐标（数值、百分数、百分数p，譬如50表示以当前View左上角坐标加50px为初始点、50%表示以当前View的左上角加上当前View宽高的50%做为初始点、50%p表示以当前View的左上角加上父控件宽高的50%做为初始点)|
| android:pivotY | ScaleAnimation(…, float pivotY)| 缩放起点Y坐标，同上规律 |

• Translate属性详解

| xml属性 | java方法 | 解释 |
|------|------|------|
| android:fromXDelta | TranslateAnimation(float fromXDelta, …) | 起始点X轴坐标（数值、百分数、百分数p，譬如50表示以当前View左上角坐标加50px为初始点、50%表示以当前View的左上角加上当前View宽高的50%做为初始点、50%p表示以当前View的左上角加上父控件宽高的50%做为初始点|
| android:fromYDelta | TranslateAnimation(…, float fromYDelta, …) | 起始点Y轴从标，同上规律 |
| android:toXDelta | TranslateAnimation(…, float toXDelta, …) | 结束点X轴坐标，同上规律|
| android:toYDelta | TranslateAnimation(…, float toYDelta) | 结束点Y轴坐标，同上规律 |

• AnimationSet详解
AnimationSet继承自Animation，是上面四种的组合容器管理类，他的属性继承自Animation，有两个属性要注意：
	• Android:interpolator代表一个插值器资源，可以引用系统自带插值器资源，也可以用自定义插值器资源，默认值是匀速插值器
	• android:shareInterpolator代表里面的多个动画是否要共享插值器，默认值为true，即共享插值器，如果设置为false，那么的插值器就不再起作用，我们要在每个动画中加入插值器。
特别注意，当我们对set标签使用Animation的属性时会对该标签下的所有子控件都产生影响。

一个简单的XML文件例子：
```java
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@[package:]anim/interpolator_resource"
    android:shareInterpolator=["true" | "false"] >
    <alpha
        android:fromAlpha="float"
        android:toAlpha="float" />
    <scale
        android:fromXScale="float"
        android:toXScale="float"
        android:fromYScale="float"
        android:toYScale="float"
        android:pivotX="float"
        android:pivotY="float" />
    <translate
        android:fromXDelta="float"
        android:toXDelta="float"
        android:fromYDelta="float"
        android:toYDelta="float" />
    <rotate
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotX="float"
        android:pivotY="float" />
    <set>
        ...
    </set>
</set>
```

java代码设置动画：
```java
ImageView spaceshipImage = (ImageView) findViewById(R.id.spaceshipImage);
Animation hyperspaceJumpAnimation = AnimationUtils.loadAnimation(this, R.anim.hyperspace_jump);
spaceshipImage.startAnimation(hyperspaceJumpAnimation);
```
当然也可以直接在java中写视图动画：
```java
AnimationSet set = new AnimationSet(false);
set.addAnimation(scaleAnimation);
set.addAnimation(translateAnimation);
imageView.startAnimation(set);
```

Animation还有如下一些比较实用的方法介绍：

| Animation类的方法 | 解释 |
|------|-------|
| reset() | 重置Animation的初始化 |
| cancel() | 取消Animation动画 |
| start() | 开始Animation动画 |
| setAnimationListener(AnimationListener listener) | 给当前Animation设置动画监听 |
| hasStarted() | 判断当前Animation是否开始 |
| hasEnded() | 判断当前Animation是否结束 |

View中和动画相关的几个常用方法：

| View类的常用动画操作方法 | 解释 |
|------|-------|
|startAnimation(Animation animation)| 对当前View开始设置的Animation动画 |
| clearAnimation() | 取消当View在执行的Animation动画 |


特别特别注意：补间动画执行之后并未改变View的真实布局属性值。

# 视图动画Interpolator插值器
![](http://oeiu2t0ur.bkt.clouddn.com/20150614183554244.png)

解释如下：
![](http://oeiu2t0ur.bkt.clouddn.com/456456454.png)

使用：
```java
<set android:interpolator="@android:anim/accelerate_interpolator">
    ...
</set>
```

## 插值器的自定义
有时候你会发现系统提供的插值器不够用，可能就像View一样需要自定义。所以接下来我们来看看插值器的自定义，关于插值器的自定义分为两种实现方式，xml自定义实现（其实就是对现有的插值器的一些属性修改）或者java代码实现方式。

先看看XML自定义插值器的步骤：
	1. 在res/anim/目录下创建filename.xml文件。
	2. 修改你准备自定义的插值器如下：
XML文件：
```java
<?xml version="1.0" encoding="utf-8"?>
<InterpolatorName xmlns:android="http://schemas.android.com/apk/res/android"
    android:attribute_name="value" />
```
第二步修改的是现有插值器的一些属性，但是有些插值器却不具备修改属性，具体如下：

> 
    <accelerateDecelerateInterpolator>
    	无可自定义的attribute。
    <accelerateInterpolator>
    	android:factor 浮点值，加速速率（默认值为1）。
    <anticipateInterploator>
    	android:tension 浮点值，起始点后拉的张力数（默认值为2）。
    <anticipateOvershootInterpolator>
    	android:tension 浮点值，起始点后拉的张力数（默认值为2）。 
    	android:extraTension 浮点值，拉力的倍数（默认值为1.5）。
    <bounceInterpolator>
    	无可自定义的attribute。
    <cycleInterplolator>
    	android:cycles 整形，循环的个数（默认为1）。
    <decelerateInterpolator>
    	android:factor 浮点值，减速的速率（默认为1）。
    <linearInterpolator>
    	无可自定义的attribute。
    <overshootInterpolator>
    	android:tension 浮点值，超出终点后的张力（默认为2）。

再来看看Java自定义插值器的（Java自定义插值器其实是xml自定义的升级，也就是说如果我们修改xml的属性还不能满足需求，那就可以选择通过Java来实现）方式。

上面所有的`Interpolator`都实现了`Interpolator`接口，而`Interpolator`接口又继承自`TimeInterpolator`，`TimeInterpolator`接口定义了一个`float getInterpolation(float input)`;方法，这个方法是由系统调用的，其中的参数`input`代表动画的时间，在0和1之间，也就是开始和结束之间。

如下就是一个动画始末速率较慢、中间加速`的AccelerateDecelerateInterpolator`插值器：
```java
public class AccelerateDecelerateInterpolator extends BaseInterpolator
    implements NativeInterpolatorFactory {
......
public float getInterpolation(float input) {
    return (float)(Math.cos((input + 1) * Math.PI) / 2.0f) + 0.5f;
}
......
}
```

# 自定义View动画
所有的自定义动画都需要继承`android.view.animation.Animation`抽象类，然后重写`initialize()`和`applyTransformation()`这两个方法。

一般会在`initialize()`方法中对一些变量进行初始化，在`applyTransformation()`方法中通过矩阵修改动画数值，从而控制动画的实现过程，这也是自定义动画的核心。

`applyTransformation(float interpolatedTime, Transformation t)`方法在动画的执行过程中会不断地调用，可以看到接收的两个参数分别是

- `float interpolatedTime` 表示当前动画进行的时间与动画总时间（一般在 setDuration() 方法中设置）的比值，从0逐渐增大到1； 
- `Transformation t`传递当前动画对象，一般可以通过代码`android.graphics.Matrix matrix = t.getMatrix()`获得`Matrix`矩阵对象，再设置`Matrix`对象，一般要用到`interpolatedTime`参数，以此达到控制动画实现的结果。（关于Matrix的使用看另外一篇）

QQ抖一抖动画的例子：

```java
	/**
	 * QQ窗口抖动的效果动画
	 */
public class QQTrembleAnimation extends Animation {

    @Override
    public void initialize(int width, int height, int parentWidth, int parentHeight) {
        super.initialize(width, height, parentWidth, parentHeight);
    }
    @Override
    protected void applyTransformation(float interpolatedTime, Transformation t) {
		super.applyTransformation(interpolatedTime, t);
        t.getMatrix().setTranslate(
                (float) Math.sin(interpolatedTime * 50) * 8,
                (float) Math.sin(interpolatedTime * 50) * 8
        );// 50越大频率越高，8越小振幅越小
    }
}
```
可以看到在上面的代码
```java
t.getMatrix().setTranslate((float) Math.sin(interpolatedTime * 50) * 8, (float) Math.sin(interpolatedTime * 50) * 8)
```
设置了 Matrix 对象的 Translate ，传入的参数是一个正弦函数值，这个值是通过`interpolatedTime`参数计算出来的，这样就实现了动画在x，y轴两个方向上的来回抖动效果。

使用：
```java
// 创建抖一抖动画对象 
QQTrembleAni tremble = new QQTrembleAni(); 
mView.setDuration(800);// 持续时间800ms，持续时间越短频率越高 
mView.setRepeatCount(2);// 重复次数，不包含第一次
mView.startAnimation(tremble);// 启动抖一抖效果 
```
