---
title: nAdroid布局属性
date: 2015-12-07 16:06:57
categories: Android
tags:Android

---

常用的布局属性

<!--more-->

# LinearLayout
```java
android:orientation="vertical" 
android:orientation="horizontal"
android:layout_width="match_parent"  
android:layout_height="match_parent"  
android:layout_weight 
```


# RelativeLayout
```java

android:layout_toLeftOf         	//该组件位于引用组件的左方  
android:layout_toRightOf        	//该组件位于引用组件的右方 
 
android:layout_above            	//该组件位于引用组件的上方  
android:layout_below                //该组件位于引用组件的下方 

android:layout_alignTop 将该控件的顶部边缘与给定ID的顶部边缘对齐;  
android:layout_alignBottom 将该控件的底部边缘与给定ID的底部边缘对齐;  
android:layout_alignLeft 将该控件的左边缘与给定ID的左边缘对齐;  
android:layout_alignRight 将该控件的右边缘与给定ID的右边缘对齐;  
 
android:layout_alignParentLeft      //该组件是否对齐父组件的左端  
android:layout_alignParentRight     //该组件是否齐其父组件的右端  
android:layout_alignParentTop       //该组件是否对齐父组件的顶部  
android:layout_alignParentBottom    //该组件是否对齐父组件的底部  

android:layout_centerInParent       //该组件是否相对于父组件居中  
android:layout_centerHorizontal     //该组件是否横向居中  
android:layout_centerVertical       //该组件是否垂直居中  

```

# 通用

(1)layout_margin 
用于设置控件边缘相对于父控件的边距

- android:layout_marginLeft   
- android:layout_marginRight
- android:layout_marginTop
- android:layout_marginBottom


(2) layout_padding 
用于设置控件内容相对于控件边缘的边距

- android:layout_paddingLeft
- android:layout_paddingRight
- android:layout_paddingTop
- android:layout_paddingBottom

(3) layout_width/height

用于设置控件的高度和宽度
wrap_content 内容包裹，表示这个控件的里面文字大小填充
fill_parent 跟随父窗口
match_parent


(4) gravity 
android:gravity="bottom"
用于设置View组件里面内容的对齐方式
top bottom  left   right   center等

(5)layout_gravity
android:layout_gravity="center"
本组件在父布局中的位置。只能在LinearLayout中使用


(5) 
用于设置Container组件的对齐方式
android:layout_alignTop 本元素的上边缘和某元素的的上边缘对齐
android:layout_alignLeft 本元素的左边缘和某元素的的左边缘对齐
android:layout_alignBottom 本元素的下边缘和某元素的的下边缘对齐
android:layout_alignRight 本元素的右边缘和某元素的的右边缘对齐

# 分类
第一类:属性值为true或false
android:layout_centerHrizontal 水平居中  
android:layout_centerVertical 垂直居中  
android:layout_centerInparent 相对于父元素完全居中
android:layout_alignParentBottom 贴紧父元素的下边缘
android:layout_alignParentLeft 贴紧父元素的左边缘
android:layout_alignParentRight 贴紧父元素的右边缘
android:layout_alignParentTop 贴紧父元素的上边缘
android:layout_alignWithParentIfMissing 如果对应的兄弟元素找不到的话就以父元素做参照物

第二类：属性值必须为id的引用名"@id/id-name"
android:layout_below 在某元素的下方
android:layout_above 在某元素的的上方
android:layout_toLeftOf 在某元素的左边
android:layout_toRightOf 在某元素的右边
android:layout_alignTop 本元素的上边缘和某元素的的上边缘对齐
android:layout_alignLeft 本元素的左边缘和某元素的的左边缘对齐
android:layout_alignBottom 本元素的下边缘和某元素的的下边缘对齐
android:layout_alignRight 本元素的右边缘和某元素的的右边缘对齐

第三类：属性值为具体的像素值，如30dip，40px
android:layout_marginBottom 离某元素底边缘的距离
android:layout_marginLeft 离某元素左边缘的距离
android:layout_marginRight 离某元素右边缘的距离
android:layout_marginTop 离某元素上边缘的距离