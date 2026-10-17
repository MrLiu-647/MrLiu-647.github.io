---
title: iOS 内存泄漏检查及原因分析
date: 2018-03-20
categories:
- iOS
---
>首先，补充两个基本概念的解释：
>- 内存溢出 （out of memory）：
是指程序在申请内存时，没有足够的内存空间供其使用，出现out of memory。通俗理解就是内存不够，通常在运行大型软件或游戏时，软件或游戏所需要的内存远远超出了你主机内安装的内存所承受大小，就叫内存溢出。
>- 内存泄露（ memory leak）：
是指程序在申请内存后，无法释放已申请的内存空间，一次内存泄露危害可以忽略，但内存泄露堆积后果很严重，无论多少内存,迟早会被占光。

# 一、排查方法
我们知道，iOS开发中对内存管理的要求非常严格，一旦存在内存泄漏，后果是非常严重的，会导致程序非常容易崩溃。尽管目前iOS开发基本上都是采用的ARC方式进行内存管理，但是一不小心就会存在内存泄漏的问题。

>首先，我们需要定位内存泄漏的问题，目前比较常用的内存泄漏的排查方法有两种，都在xcode中可以直接使用：静态分析方法（Analyze）和动态分析方法（Instrument的leak）。

## 1.1 静态内存泄漏分析方法

通过xcode打开项目，然后点击product-->Analyze，如下图左侧的图所示，这样就开始对项目进行静态内存泄漏分析，分析结果如下图右侧的图所示。根据分析结果进行休整之后在进行分析就好了。![静态内存泄漏分析](http://upload-images.jianshu.io/upload_images/3407530-1dcd720091ca9632..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![静态内存泄漏分析](http://upload-images.jianshu.io/upload_images/3407530-7f3440ae34ab5abc..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>静态分析方法能发现大部分的问题，但是只能是静态分析结果，有一些并不准确，还有一些动态分配内存的情形并没有进行分析。所以仅仅使用静态内存泄漏分析得到的结果并不是非常可靠，如果需要，我们需要将对项目进行更为完善的内存泄漏分析和排查。那就需要用到我们下面要介绍的动态内存泄漏分析方法Instruments中的Leaks方法进行排查。

## 1.2 动态内存泄漏分析方法
分析内存泄露不能把所有的内存泄露查出来，有的内存泄露是在运行时，用户操作时才产生的。那就需要用到Instruments了。具体操作是通过xcode打开项目，然后点击product-->profile，如下图所示。

![动态内存泄漏分析](http://upload-images.jianshu.io/upload_images/3407530-2393941b877be661..jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![动态内存泄漏分析](http://upload-images.jianshu.io/upload_images/3407530-c3c3792214e580ee..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
按上面操作，build成功后跳出Instruments工具，如上图右侧图所示。选择Leaks选项，点击右下角的【choose】按钮，这时候项目程序也在模拟器或手机上运行起来了，在手机或模拟器上对程序进行操作，工具显示效果如下：

![leaks](http://upload-images.jianshu.io/upload_images/3407530-ab8cfd4bdeae196b..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击左上角的红色圆点，这时项目开始启动了，由于leaks是动态监测，所以手动进行一系列操作，可检查项目中是否存在内存泄漏问题。如图所示，橙色矩形框中所示绿色为正常，如果出现如右侧红色矩形框中显示红色，则表示出现内存泄漏。

![leaks](http://upload-images.jianshu.io/upload_images/3407530-2a59e90cb1d772cd..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

选中Leaks Checks,在Details所在栏中选择CallTree,并且在右下角勾选Invert Call Tree 和Hide System Libraries，会发现显示若干行代码，双击即可跳转到出现内存泄漏的地方，修改即可。
![leaks](http://upload-images.jianshu.io/upload_images/3407530-1a7c5b1cabc8ab6b..png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 二、内存泄漏的原因分析
在目前主要以ARC进行内存管理的开发模式，导致内存泄漏的根本原因是代码中**存在循环引用**，从而导致一些内存无法释放，这就会导致dealloc()方法无法被调用。主要原因大概有一下几种类型。

## 2.1 ViewController中存在NSTimer
如果你的ViewController中有NSTimer，那么你就要注意了，因为当你调用
```objc
[NSTimer scheduledTimerWithTimeInterval:1.0
                                 target:self
                               selector:@selector(updateTime:)
                               userInfo:nil
                                repeats:YES];
```
>这时->  【target: self】，增加了ViewController的return count，
如果不将这个【timer invalidate】，就别想调用dealloc。

## 2.2 ViewController中的代理delegate
>一个比较隐秘的因素，你去找找与这个类有关的代理，有没有强引用属性？如果你这个VC需要外部传某个Delegate进来，来通过Delegate+protocol的方式传参数给其他对象，那么这个delegate一定不要强引用，尽量assign或者weak，否则你的VC会持续持有这个delegate，直到它自身被释放。

## 2.3 ViewController中Block
>这个可能就是经常容易犯的一个问题了，Block体内使用实例变量也会造成循环引用，使得拥有这个实例的对象不能释放。因为该block本来就是当前viewcontroller的一部分，现在盖子部门又强引用self，导致循环引用无法释放。 例如你这个类叫OneViewController,有个属性是NSString *name; 如果你在block体中使用了self.name，或者_name，那样子的话这个类就没法释放。 要解决这个问题其实很简单，就是在block之前申明当前的self引用为弱引用即可。
```objc
//MRC下代码如下
__block Viewcontroller *weakSelf = self;
//ARC下代码如下
__weak Viewcontroller *weakSelf = self;
```

## 2.4 ViewController的子视图对self的持有
>这个问题也是我的项目中内存泄漏的问题所在。我们有时候需要在子视图或者某个cell中点击跳转等操作，需要在子视图或cell中持有当前的ViewController对象，这样跳转之后的back键才能直接返回该页面，同时也不销毁当前ViewController。此时，你就要注意在子视图或者cell中对当前页面的持有对象不能是强引用，尽量assign或者weak，否则会造成循环引用，内存无法释放。

注：[引用博客](https://www.cnblogs.com/mukekeheart/p/8144742.html)
