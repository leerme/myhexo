---
title: iOS React/RCTBundleURLProvider.h' file not found
date: 2017-06-27 13:52:15
---
	学习React Native使用react-native init 项目名称之后运行项目，编译就报React/RCTBundleURLProvider.h' file not found怎么办。

### 下面先看一下实现效果.
<div align=center>
![屏幕截图](http://ow7iaz7ej.bkt.clouddn.com/2reactnative.png)
</div>

### 解决方法

原来是新版本有问题
新建项目指定版本：
用--version参数创建指定版本的项目。例如react-native init ceshi --version 0.44.3注意版本号必须精确到两个小数点。
项目创建好之后：执行：react-native run-ios