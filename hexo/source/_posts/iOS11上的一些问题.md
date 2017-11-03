---
title: 项目在iOS11上遇到的小问题
date: 2017-10-20 15:42:37
---
iOS11正式版出了这么久了，在忙完新版本开发，写下在iOS11上的一些小问题。
### 一、App图标不显示
**现象**：升级到iOS11系统下自己的项目桌面app图标不见了
![图标不见](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE3.jpg)
出现这种情况我还以为自己手动删除了项目 **Images.xcassets**中的**AppIcon**导致没有图标。查看项目和发现这些AppIcon还在，突然发现在Xcode 9中AppIcon有了改变。
![AppIcon](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE2.png)
发现Xcode 8及之前的版本是可以直接在iTunes Connect上添加App icon。而Xcode 9则是把App icon放置在项目的asset catalog。所以需要把空缺的图标补上，其中一张是1024pt 1x的尺寸。如果没有正确添加iOS App图标，上传到App Store后可能会受到拒绝邮件。
心想既然图标变了那么**LaunchImage**呢，果不其然LaunchImage也有小变化为了适配苹果即将售出的iPHone X，这里我们需要在这里添加一张新的尺寸图1125px*2436px。
![LaunchImage](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE1.png)
那么究竟是什么导致，在iOS 11上面图标消失的呢。查证后发现。

**原因**： 使用了CocoaPods的Xcode工程,在iOS11版的手机上AppIcon不显示,原因是CocoaPods的资源编译脚本在iOS11下出了点问题。

**解决办法**：
1.在Podfile添加脚本修改：
1). 在Podfile 添加如下代码.

``` 
post_install do |installer|
    copy_pods_resources_path = "Pods/Target Support Files/Pods-[工程名]/Pods-[工程名]-resources.sh"
    string_to_replace = '--compile "${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}"'
    assets_compile_with_app_icon_arguments = '--compile "${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}" --app-icon "${ASSETCATALOG_COMPILER_APPICON_NAME}" --output-partial-info-plist "${BUILD_DIR}/assetcatalog_generated_info.plist"'
    text = File.read(copy_pods_resources_path)
    new_contents = text.gsub(string_to_replace, assets_compile_with_app_icon_arguments)
    File.open(copy_pods_resources_path, "w") {|file| file.puts new_contents }
end ```

需要注意的是,将[工程名] 换成自己工程的名称

2).然后运行

```
$pod install
```
2.手动修改
打开工程目录下:[工程名]/Pods/Target Support Files/Pods/Pods-resources.sh这个文件,替换最后一段代码:

修改前:

 ```
 printf "%s\0" "${XCASSET_FILES[@]}" | xargs -0 xcrun actool --output-format human-readable-text --notices --warnings --platform "${PLATFORM_NAME}" --minimum-deployment-target "${!DEPLOYMENT_TARGET_SETTING_NAME}" ${TARGET_DEVICE_ARGS} --compress-pngs --compile "${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}"
 fi
 ```

修改后:

```
printf "%s\0" "${XCASSET_FILES[@]}" | xargs -0 xcrun actool --output-format human-readable-text --notices --warnings --platform "${PLATFORM_NAME}" --minimum-deployment-target "${!DEPLOYMENT_TARGET_SETTING_NAME}" ${TARGET_DEVICE_ARGS} --compress-pngs --compile "${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}" --app-icon "${ASSETCATALOG_COMPILER_APPICON_NAME}" --output-partial-info-plist "${BUILD_DIR}/assetcatalog_generated_info.plist"
fi
```
然后重新运行工程即可,配置完成后如果启动后发现还没有图标，现在系统低于iOS 11下的手机运行一次，再在iOS 11上启动就回发现有了。

参考：https://github.com/CocoaPods/CocoaPods/issues/7003

### 二、用到相机功能时闪退
**现象**：在用户在手机系统iOS 11里面使用App进行身份证、银行卡ORC识别的时候打开相机发生Crash现象；在iOS 11以下正常。
**原因**：iOS11下，苹果对相册的权限key做了调整
详见：{% link CocoaKeys https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW17 [external] [CocoaKeys] %}
**解决办法**：在Info.plist里添加以下权限
![添加权限](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE4.png)

### 三、H5和Native交互的时候控制更严格
**现象**：在iOS 11上点击UIWebView加载的H5页面，来与原生App进行交互出现Crash，报错webTread。

**原因**：iOS11下，UIWebView与原生交互的时候出现了线程安全问题；控制更加严格。

**解决办法**：把调用原生方法的代码放到主线程中运行。
![web和Native交互](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE5.png)

### 四、在Xcode 9中的无线调试
Xcode 9 里面把很多简单的快捷键给改复杂了，一些插件不支持了。最有利于开发者的地方就是Xcode 9中的无线调试了。
苹果诟病最多的产品：数据线
心疼的抱住我自己，穿着缝缝补补的衣服，用着自己拼拼接接的数据线
![我的数据线](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE7.jpeg)

在Xcode 9中没有这样的烦恼了。

升级到Xcode9.0之后，可以通过Wifi连接iOS或tvOS设备进行无线调试。

要求: Xcode 9.0 以上版本、macOS 10.12.4以上版本、iOS 11.0以上版本, tvOS 11.0以上版本

操作步骤：

打开菜单 Window > Devices and Simulators, 然后在打开的菜单中选择 Devices选项.

通过数据线将您的设备，比如iPhone，连接至Mac电脑.
在如下图选择连接的设备，然后在右侧勾选[通过网络连接]复选框.
![](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE8.png)
Xcode 会和你的设备进行配对. 一旦Xcode和设备配对成功，设备名称的右侧会显示一个网络图标
![](http://ow7iaz7ej.bkt.clouddn.com/6%E5%9B%BE9.png)
最后将设备的数据线从Mac电脑上取出，就可以通过Wifi无线调试了！
