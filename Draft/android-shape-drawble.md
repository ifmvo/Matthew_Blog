---
title: Andorid Shape 这一篇就够了
date: 2017-09-20
tag: Android
---

```
<?xml version="1.0" encoding="utf-8"?>
<!--对应的实体类：GradientDrawable-->

<!--line oval rectangle ring 这四种shape 的类型-->
<!--
    ring 有5个特殊的属性
    1.innerRadius 圆的内半径和 innerRadiusRatio 同时存在的时候 以innerRadius为准
    2.thickness 圆环的厚度，即外半径和内半径的差，和thicknessRatio 同时存在的时候，以thickness为准
    3.innerRadiusRatio 内半径占整个宽度的比例 默认是9 例：内半径 = 宽度 / n
    4.thicknessRatio 厚度占整个宽度的比例
    5.useLevel 一般是false，true 的话 就可以当做LevelListDrawable 来使用

-->
<!-- line 和 ring 必须要通过 stroke 标签来制定线的宽度和颜色，否则无法达到预期的效果-->
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line">

    <corners
        android:bottomLeftRadius="9dp"
        android:bottomRightRadius="9dp"
        android:topLeftRadius="9dp"
        android:topRightRadius="9dp"
        android:radius="9dp"/><!-- 优先级最低会被其他的四个属性覆盖 -->


    <!--
        与solid 标签是排斥的，solid 表示的是纯色的填充，而gradient 表示的是渐变的效果
        angle 指的是渐变的角度，默认是0 ， 值必须是45 的倍数，0表示从左到右，90表示从下到上
        centerX 渐变的中心点的横坐标
        centerY 渐变的中心点的纵坐标
        startColor 渐变的起始色
        centerColor 渐变的中间色
        endColor 渐变的结束色
        gradientRadius 渐变的半径 当type = radial时才有效
        type = linear（线性） / radial（光圈） / sweep（雷达一样） 渐变类型
     -->
    <gradient
        android:centerX="9dp"
        android:centerColor=""
        android:startColor=""
        android:endColor=""
        android:gradientRadius=""
        android:type="sweep"
        android:angle="9dp" />

    <!--与solid 标签是排斥的，solid 表示的是纯色的填充，而gradient 表示的是渐变的效果-->
    <solid android:color=""/>

    <!-- line 和 ring 必须要通过 stroke 标签来制定线的宽度和颜色，否则无法达到预期的效果-->
    <!-- 
        描边
        width 描边的宽度边缘线的宽度
        dashGap 组成虚线的线段之间的间隔，间隔越大，虚线看起来空隙就越大
        dashWidth 组成虚线的线段的宽度
        color 描边的颜色
        注：dashGap 和 dashWidth 任何一个为 0 就不是虚线的效果了
     -->
    <stroke
        android:width="50dp"
        android:dashGap="60dp"
        android:dashWidth="70dp"
        android:color="#5c1e1e"/>

    <!--表示的不是shape 的空白，而是包含它的view 的空白-->
    <padding 
        android:bottom="1dp"
        android:right="1dp"
        android:top="1dp"
        android:left="1dp"/>
    
    <!--Drawable 宽高-->
    <size 
        android:height="1dp"
        android:width="1dp"/>
    
</shape>
```