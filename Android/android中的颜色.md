---
title: android中的颜色详解
date: 2016-04-02 12:42:50
categories: Android
tags: android-view
---
# 安卓支持的颜色模式：



- 四种：

	ARGB8888——四通道高精度(32位)  
	ARGB4444——四通道低精度(24位)  
	RGB565——屏幕默认模式(16位)  
	Alpha8——仅有透明通道(8位)  



<!--more-->
- PS：其中字母表示通道类型，数值表示该类型用多少位二进制来描述。如ARGB8888则表示有四个通道(ARGB),每个对应的通道均用8位来描述。



- 注意：我们常用的是ARGB8888和ARGB4444，而在所有的安卓设备屏幕上默认的模式都是RGB565,请留意这一点。




- 其中 A R G B 的取值范围均为0~255(即16进制的0x00~0xff)

	A 从ox00到oxff表示从透明到不透明。
	
	RGB 从0x00到0xff表示颜色从浅到深。
	
	当RGB全取最小值(0或0x000000)时颜色为黑色，全取最大值(255或0xffffff)时颜色为白色

# 创建或使用颜色的方式
### java中定义颜色：
1. 使用Color类的常量  

	```java
		int color = Color.GRAY;     //灰色
	```

2. 通过ARGB构建

	```java
		int color = Color.argb(127, 255, 0, 0);   //半透明红色，其中第一个参数表示透明，0表示完全透明，255(ff)表示完全不透明；后三位分别代表RGB的值了。
	```
3. 直接定义色值

	```java
	int color = 0xaaff0000; //带有透明度的红色
	```
注意：这种方法必须使用0x开头，而不是用我们常用的#。0x代表16进制，值也必须用8位表示 ，不接受6位的颜色表示。分组一下0x|aa|ff0000，0x是代表颜色整数的标记，aa是表示透明度，ff0000表示RGB颜色值。

### 在xml文件中定义颜色：

在/res/values/color.xml 文件中如下定义：

	```java
	<?xml version="1.0" encoding="utf-8"?>
	<resources>
	    <color name="red">#ff0000</color>
	    <color name="green">#00ff00</color>
	</resources>
	```

详解： 在以上xml文件中定义了两个颜色，红色和蓝色，是没有alpha（透明）通道的。

定义颜色以‘#’开头，后面跟十六进制的值，有如下几种定义方式：

	```java
	  #f00            //低精度 - 不带透明通道红色
	  #af00           //低精度 - 带透明通道红色
	
	  #ff0000         //高精度 - 不带透明通道红色
	  #aaff0000       //高精度 - 带透明通道红色
	  #RGB #ARGB #RRGGBB #AARRGGBB这四种都是对的
	```
### java中引用xml中定义的颜色：
	int color = getResources().getColor(R.color.mycolor);

### 在xml文件(layout或style)中引用或者创建颜色

	android:background="@color/red"     //引用在/res/values/color.xml 中定义的颜色
	android:background="@android:color/white"//引用系统的颜色
  	android:background="#ff0000"        //直接使用



# 颜色混合模式(Alpha通道相关)
见[Android图形动画](http://www.cnblogs.com/zhucai/p/android-graphics-animation.html)

![](https://camo.githubusercontent.com/612e4ae739238b864f9bc338a7ef7d7c65a9aa6a/687474703a2f2f7777342e73696e61696d672e636e2f6c617267652f3030355874646932677731663177613066306d7a6a6a333068683066736a74382e6a7067)