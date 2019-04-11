---
title: Android Gradle 加速编译 
date: 2017-08-16
tag: Android
---

> 优化前 rebuild project 2m 10s

### 1. 项目根目录下的 gradle.properties 中添加

```
org.gradle.daemon=true
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
org.gradle.parallel=true
org.gradle.configureondemand=true
```

> 此时 rebuild project 1m 14s

### 2. Android Studio 的配置 offline

- Settings -> gradle -> offline work 勾选
- Settings -> compiler -> Compile independent modules in parallel 勾选
- Settings -> compiler -> Make project automatically 勾选
- Settings -> compiler -> Command-line Options: 中填写 "--offine"

> 此时 rebuild project 1m 4s

网上大多说的就是这两点，当然还有一些高级的

### 3. [Android 模块化编程之引用本地的 AAR](http://stormzhang.com/android/2015/03/01/android-reference-local-aar/)

### 4. [Android 秒级编译 Freeline](http://stormzhang.com/2016/12/02/android-seconds-build-freeline/)

### 5. [加速 Android Studio 的构建速度](https://www.diycode.cc/topics/683)
