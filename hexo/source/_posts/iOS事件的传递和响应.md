---
title: iOS 事件的传递和响应
date: 2017-08-24 10:41:45
---
	谈一谈iOS事件的产生和传递
# 1.事件的产生
* 发生触摸事件后，系统会将该事件加入到一个由UIApplication管理的事件队列中.
* UIApplication会从事件队列中取出最前面的事件，并将事件分发下去以便处理，通常，先发送事件给应用程序的主窗口（keyWindow）。
* 主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件，这也是整个事件处理过程的第一步

# 2.事件的传递
 1. 首先判断主窗口（keyWindow）自己是否能接受触摸事件
 2. 判断触摸点是否在自己身上
 3. 子控件数组中从后往前遍历子控件，重复前面的两个步骤（所谓从后往前遍历子控件，就是首先查找子控件数组中最后一个元素，然后执行1、2步骤
 4. view，比如叫做subView，那么会把这个事件交给这个subView，再遍历这个subView的子控件，直至没有更合适的view为止
 5. 如果没有符合条件的子控件，那么就认为自己最合适处理这个事件，也就是自己是最合适的view
 
### UIView不能接收触摸事件的三种情况：
* **不允许交互**：userInteractionEnabled = NO
* **隐藏**：如果把父控件隐藏，那么子控件也会隐藏，隐藏的控件不能接受事件
* **透明度**：如果设置一个控件的透明度<0.01，会直接影响子控件的透明度。alpha：0.0~0.01为透明。


**注 意**:默认UIImageView不能接受触摸事件，因为不允许交互，即userInteractionEnabled = NO，所以如果希望UIImageView可以交互，需要userInteractionEnabled = YES。发生在事件传递的时候。

# 总结
1. 点击一个UIView或产生一个触摸事件A，这个触摸事件A会被添加到由UIApplication管理的事件队列中（即，首先接收到事件的是UIApplication）。
2. UIApplication会从事件对列中取出最前面的事件（此处假设为触摸事件A），把事件A传递给应用程序的主窗口（keyWindow）。
3. 窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件。（至此，第一步已完成）

![iOS事件示例](http://ow7iaz7ej.bkt.clouddn.com/3iosevent.png)

如果想让某个view不能接收事件（或者说，事件传递到某个view那里就断了），那么可以通过刚才提到的三种方式。比如，设置其userInteractionEnabled = NO;那么传递下来的事件就会由该view的父控件处理。
例如，不想让蓝色的view接收事件，那么可以设置蓝色的view的userInteractionEnabled = NO;那么点击黄色的view或者蓝色的view所产生的事件，橙色的view就会成为最合适的view。事件都会由橙色的veiw处理。
所以，不管视图能不能处理事件，只要点击了视图就都会产生事件，关键看该事件是由谁来处理！也就是说，如果蓝色视图不能处理事件，点击蓝色视图产生的触摸事件不会由被点击的视图（蓝色视图）处理！

**注意**：如果设置父控件的透明度或者hidden，会直接影响到子控件的透明度和hidden。如果父控件的透明度为0或者hidden = YES，那么子控件也是不可见的！

# 3.如何寻找最合适的view

	应用如何找到最合适的控件来处理事件？

1. 首先判断主窗口（keyWindow）自己是否能接受触摸事件
2. 触摸点是否在自己身上
3. 从后往前遍历子控件，重复前面的两个步骤（首先查找数组中最后一个元素）
4. 如果没有符合条件的子控件，那么就认为自己最合适处理

**详述：**
1、主窗口接收到应用程序传递过来的事件后，首先判断自己能否接手触摸事件。如果能，那么在判断触摸点在不在窗口自己身上
2、如果触摸点也在窗口身上，那么窗口会从后往前遍历自己的子控件（遍历自己的子控件只是为了寻找出来最合适的view）
3、遍历到每一个子控件后，又会重复上面的两个步骤（传递事件给子控件，1.判断子控件能否接受事件，2.点在不在子控件上）
4、如此循环遍历子控件，直到找到最合适的view，如果没有更合适的子控件，那么自己就成为最合适的view。
找到最合适的view后，就会调用该view的touches方法处理具体的事件。所以，只有找到最合适的view，把事件传递给最合适的view后，才会调用touches方法进行接下来的事件处理。找不到最合适的view，就不会调用touches方法进行事件处理。
注意：之所以会采取从后往前遍历子控件的方式寻找最合适的view只是为了做一些循环优化。因为相比较之下，后添加的view在上面，降低循环次数。

## 3.1.寻找最合适的view底层剖析
两个重要的方法：
hitTest:withEvent:方法
pointInside方法
### 3.1.1.hitTest：withEvent：方法

#### 什么时候调用？
	只要事件一传递给一个控件,这个控件就会调用他自己的hitTest：withEvent：方法
#### 作用
	寻找并返回最合适的view(能够响应事件的那个最合适的view)
**注 意：**不管这个控件能不能处理事件，也不管触摸点在不在这个控件上，事件都会先传递给这个控件，随后再调用hitTest:withEvent:方法

#### 拦截事件的处理
* 正因为hitTest：withEvent：方法可以返回最合适的view，所以可以通过重写hitTest：withEvent：方法，返回指定的view作为最合适的view。
* 不管点击哪里，最合适的view都是hitTest：withEvent：方法中返回的那个view。
* 通过重写hitTest：withEvent：，就可以拦截事件的传递过程，想让谁处理事件谁就处理事件。
事件传递给谁，就会调用谁的hitTest:withEvent:方法。

**注 意：如果hitTest:withEvent:方法中返回nil，那么调用该方法的控件本身和其子控件都不是最合适的view，也就是在自己身上没有找到更合适的view。那么最合适的view就是该控件的父控件。
所以事件的传递顺序是这样的：
　　产生触摸事件->UIApplication事件队列->[UIWindow hitTest:withEvent:]->返回更合适的view->[子控件 hitTest:withEvent:]->返回最合适的view**
　　
事件传递给窗口或控件的后，就调用hitTest:withEvent:方法寻找更合适的view。所以是，先传递事件，再根据事件在自己身上找更合适的view。
不管子控件是不是最合适的view，系统默认都要先把事件传递给子控件，经过子控件调用自己的hitTest:withEvent:方法验证后才知道有没有更合适的view。即便父控件是最合适的view了，子控件的hitTest:withEvent:方法还是会调用，不然怎么知道有没有更合适的！即，如果确定最终父控件是最合适的view，那么该父控件的子控件的hitTest:withEvent:方法也是会被调用的。
技巧：想让谁成为最合适的view就重写谁自己的父控件的hitTest:withEvent:方法返回指定的子控件，或者重写自己的hitTest:withEvent:方法 return self。但是，建议在父控件的hitTest:withEvent:中返回子控件作为最合适的view！

原因在于在自己的hitTest:withEvent:方法中返回自己有时候会出现问题。因为会存在这么一种情况：当遍历子控件时，如果触摸点不在子控件A自己身上而是在子控件B身上，还要要求返回子控件A作为最合适的view，采用返回自己的方法可能会导致还没有来得及遍历A自己，就有可能已经遍历了点真正所在的view，也就是B。这就导致了返回的不是自己而是触摸点真正所在的view。所以还是建议在父控件的hitTest:withEvent:中返回子控件作为最合适的view！
例如：whiteView有redView和greenView两个子控件。redView先添加，greenView后添加。如果要求无论点击那里都要让redView作为最合适的view（把事件交给redView来处理）那么只能在whiteView的hitTest:withEvent:方法中return self.subViews[0];这种情况下在redView的hitTest:withEvent:方法中return self;是不好使的！
```
// 这里redView是whiteView的第0个子控件
#import "redView.h"

@implementation redView
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{ 
    return self;
}
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{ 
    NSLog(@"red-touch");
}@end
// 或者
#import "whiteView.h"

@implementation whiteView
- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{ 
    return self.subviews[0];
}
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{ 
    NSLog(@"white-touch");
}
@end
```

hit:withEvent:方法底层会调用pointInside:withEvent:方法判断点在不在方法调用者的坐标系上。

### 3.1.2.pointInside:withEvent:方法

pointInside:withEvent:方法判断点在不在当前view上（方法调用者的坐标系上）如果返回YES，代表点在方法调用者的坐标系上;返回NO代表点不在方法调用者的坐标系上，那么方法调用者也就不能处理事件。

# 4事件的响应
## 4.1.触摸事件处理的整体过程
1>用户点击屏幕后产生的一个触摸事件，经过一系列的传递过程后，会找到最合适的视图控件来处理这个事件2>找到最合适的视图控件后，就会调用控件的touches方法来作具体的事件处理touchesBegan…touchesMoved…touchedEnded…3>这些touches方法的默认做法是将事件顺着响应者链条向上传递（也就是touch方法默认不处理事件，只传递事件），将事件交给上一个响应者进行处理

## 4.2.响应者链条示意图
**响应者链条**：在iOS程序中无论是最后面的UIWindow还是最前面的某个按钮，它们的摆放是有前后关系的，一个控件可以放到另一个控件上面或下面，那么用户点击某个控件时是触发上面的控件还是下面的控件呢，这种先后关系构成一个链条就叫“响应者链”。也可以说，响应者链是由多个响应者对象连接起来的链条。

**响应者对象**：能处理事件的对象，也就是继承自UIResponder的对象
**作用**：能很清楚的看见每个响应者之间的联系，并且可以让一个事件多个对象处理。

#### 如何判断上一个响应者

1. 如果当前这个view是控制器的view,那么控制器就是上一个响应者
2. 如果当前这个view不是控制器的view,那么父控件就是上一个响应者
响应者链的事件传递过程:

#### 事件处理的整个流程总结：
1. 触摸屏幕产生触摸事件后，触摸事件会被添加到由UIApplication管理的事件队列中（即，首先接收到事件的是UIApplication）。
2. UIApplication会从事件队列中取出最前面的事件，把事件传递给应用程序的主窗口（keyWindow）。
3. 主窗口会在视图层次结构中找到一个最合适的视图来处理触摸事件。（至此，第一步已完成)
4. 最合适的view会调用自己的touches方法处理事件
5. touches默认做法是把事件顺着响应者链条向上抛。

#### 事件的传递与响应：
1. 当一个事件发生后，事件会从父控件传给子控件，也就是说由UIApplication -> UIWindow -> UIView -> initial view,以上就是事件的传递，也就是寻找最合适的view的过程。

2. 接下来是事件的响应。首先看initial view能否处理这个事件，如果不能则会将事件传递给其上级视图（inital view的superView）；如果上级视图仍然无法处理则会继续往上传递；一直传递到视图控制器view controller，首先判断视图控制器的根视图view是否能处理此事件；如果不能则接着判断该视图控制器能否处理此事件，如果还是不能则继续向上传 递；（对于第二个图视图控制器本身还在另一个视图控制器中，则继续交给父视图控制器的根视图，如果根视图不能处理则交给父视图控制器处理）；一直到 window，如果window还是不能处理此事件则继续交给application处理，如果最后application还是不能处理此事件则将其丢弃

3. 在事件的响应中，如果某个控件实现了touches...方法，则这个事件将由该控件来接受，如果调用了[supertouches….];就会将事件顺着响应者链条往上传递，传递给上一个响应者；接着就会调用上一个响应者的touches….方法

#### 如何做到一个事件多个对象处理：
因为系统默认做法是把事件上抛给父控件，所以可以通过重写自己的touches方法和父控件的touches方法来达到一个事件多个对象处理的目的。

```- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{ 
// 1.自己先处理事件...
NSLog(@"do somthing...");
// 2.再调用系统的默认做法，再把事件交给上一个响应者处理
[super touchesBegan:touches withEvent:event]; 
}```
#### 事件的传递和响应的区别：

事件的传递是从上到下（父控件到子控件），事件的响应是从下到上（顺着响应者链条向上传递：子控件到父控件。

参考链接：http://www.jianshu.com/p/2e074db792ba

