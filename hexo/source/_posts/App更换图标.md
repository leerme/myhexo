---
title: iOS App更换图标
date: 2017-10-27 13:52:27
---

前段时间查看Apple官方文档，发现了一个新功能就是更换App图标，发现这个功能后，一顿暗喜，装逼的时候又到了! 可以给用户更好的体验、

例如可以在不同的节日更换不同的图标啦
![屏幕截图](http://ow7iaz7ej.bkt.clouddn.com/7%E8%8A%82%E6%97%A5%E5%9B%BE%E6%A0%87.jpg)

美中不足的是该功能（API）当前只支持**iOS10.3**以上的系统，所以只能当做一项附加功能来进行使用。下面将详细讲解下如何使用代码来实现此功能。

该功能应用的场景
1、白天/夜间模式切换，在切换App主色调同时切换App图标。
2、各类皮肤主题（淘宝就可换肤），附带App图标一块更换。
3、利用App图标表达某种特定功能，如节日，天气。


{% link API与文档 https://developer.apple.com/documentation/uikit/uiapplication/2806818-setalternateiconname?preferredLanguage=occ %}
### API方法 ###
![](http://ow7iaz7ej.bkt.clouddn.com/7%E6%96%87%E6%A1%A31.png)
### 文档 ###
1、supportsAlternateIcons
![](http://ow7iaz7ej.bkt.clouddn.com/7%E6%96%87%E6%A1%A32.png)
只有系统允许改变你的app图标时该值才为YES。你需要在Info.plist文件中的CFBundleIcons这个键内声明可更换的app图标。

2、alternateIconName
![](http://ow7iaz7ej.bkt.clouddn.com/7%E6%96%87%E6%A1%A33.png)
当系统展示的是你更换后的app图标时，该值即为图标名字（Info.plist中定义的图标名字）。如果展示的是主图标时，这个值为nil。

3、setAlternateIconName:completionHandler:
![](http://ow7iaz7ej.bkt.clouddn.com/7%E6%96%87%E6%A1%A34.png)
alertnateIconName参数：该参数为需要更换的app图标名字，是在你的Info.plist中的CFBundleAlertnateIcons键里定义的。如果你想显示的是用CFBundlePrimaryIcon键所定义的主图标的话，就传入nil。CFBundleAlertnateIcons与CFBundlePrimaryIcon键都是在CFBundleIcons里面定义的。

completionHandler参数：该参数用来处理（更换）结果。当系统尝试更改app的图标后，会将结果数据通过该参数传入并执行（该执行过程是在UIKit所提供的队列执行，并非主队列）。该执行过程会携带一个参数：error。如果更换app图标成功，那么这个参数就是nil。如果更换过程中发生了错误，那么该对象会指明错误信息，并且app的图标保持不变。

![](http://ow7iaz7ej.bkt.clouddn.com/7%E6%96%87%E6%A1%A35.png)
使用该方法改变app图标为主图标或者可更换的图标。只有在supportsAlternateIcons的返回值为YES时才能更换

你必须在Info.plist文件的CFBundleIcons键里面声明可以更换的app图标（主图标和可更换图标）。如果需要获取关于可更换图标的配置信息，请查阅 {% link Information Property List Key https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009247 %} 里面有关CFBundleIcons的描述。

### 可变更App图标的配置方法 ###
![](http://ow7iaz7ej.bkt.clouddn.com/7%E6%96%87%E6%A1%A36.png)
Info.plist是个字典，假设为NSDictionary *infoPlist。

1.CFBundleIcons是Info.plist字典里的一个键@"CFBundleIcons"。
2.CFBundleIcons对应的value是个字典。
3.CFBundleIcons里面能够包含的键有：CFBundlePrimaryIcon、CFBundleAlternateIcons、UINewsstandIcon。

代码展示下这个绕口的结构：
```
NSDictionary *infoPlist;
infoPlist = @{
               @"CFBundleIcons" : @{
                                     @"CFBundlePrimaryIcon" : xxx,
                                     @"CFBundleAlternateIcons" : xxx,
                                     @"UINewsstandIcon" : xxx
                                   }
             };
             ```
             
             
1.CFBundleAlternateIcons所对应的value是个字典（iOS中），假设为NSDictionary * alertnateIconsDic。
2.alertnateIconsDic的键，都是备用图标的名字，假设为@"newAppIcon"和@"newAppIcon2"。
3.@"newAppIcon"的value是个包含CFBundleIconFiles和UIPrerenderedIcon这两个键的字典。
4.CFBundleIconFiles的value是字符串或者数组（数组内容也为字符串）。字符串的内容为各尺寸备用图标的名字。
5.UIPrerenderedIcon的value是BOOL值。这个键值所代表的作用在iOS7之后（含iOS7）已失效，在iOS6中可渲染app图标为带高亮效果。所以这个值目前我们可以不用关心。
代码展示下这个绕口的结构：
```
@"CFBundleAlternateIcons" : @{
                               @"newAppIcon" : @{
                                                 @"CFBundleIconFiles" : @[
                                                      					  @"newAppIcon"
                                                    					 ],
                                                 @"UIPrerenderedIcon" : NO
                                				},
                               @"newAppIcon2" : @{
                                                 @"CFBundleIconFiles" : @[
                                                      					  @"newAppIcon2"
                                                                         ],
                                                 @"UIPrerenderedIcon" : NO
                                				 }
							 };
             ```
             
### 实际配置文件(Info.plist) ###
对照着上述的配置文档，我们实际配置完的Info.plist是这样子的：           
![](http://ow7iaz7ej.bkt.clouddn.com/7%E9%A1%B9%E7%9B%AE1.png)
拖入对应的App图标：
![](http://ow7iaz7ej.bkt.clouddn.com/7%E9%A1%B9%E7%9B%AE2.png)
在Build Phases -> Copy Bundle Resources加入对应的图片资源           
![](http://ow7iaz7ej.bkt.clouddn.com/7%E9%A1%B9%E7%9B%AE3.png)

然后调用此代码，传入对应的图标名称就可以看到App图标发生了改变啦
```
- (void)setAppIconWithName:(NSString *)iconName {
    if (![[UIApplication sharedApplication] supportsAlternateIcons]) {
        return;
    }
    if ([iconName isEqualToString:@""]) {
        iconName = nil;
    }
    [[UIApplication sharedApplication] setAlternateIconName:iconName completionHandler:^(NSError * _Nullable error) {
        if (error) {
            NSLog(@"更换app图标发生错误了 ： %@",error);
        }
    }];
}
```

难受 苹果限制真多，上传提交审核的时候出现了。
![](http://ow7iaz7ej.bkt.clouddn.com/7%E9%A1%B9%E7%9B%AE4.png)
老老实实在info.plist里面加上这个key