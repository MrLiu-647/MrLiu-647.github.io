---
title: iOS 为何所有UI控件和代理都用weak而不是strong？
date: 2017-08-20
categories:
- iOS
---

>在ARC中，对象释放的最终根据还是根据引用计数为0时去释放。
而weak与strong的根本区别是在set方法中，
weak的set方法和strong的set方法都是释放旧值保留新值，
但是weak的set方法会对其autorelease，即release一次，
而strong的set方法也是释放旧值保留新值，但是其不会release。
最终效果是strong会+1，weak不会+1.

>**要注意**，用_ 去赋值的时候是不调用set方法的，也就是说无论weak还是strong，只要用_ 赋值都不会对引用计数加1，区别在于self.语法会调用set方法，strong的self.会调用set方法+1.而weak的_ 和self.语法都不会+1.
所以，建议用weak，用weak时无论用_和.语法都不会导致+1。用strong时用self.语法会导致+1.建议用weak。
同时weak在对象回收以后可以将对象指针置成nil。

>在代理中，
以UITableViewController为例，该控制器内部有一个tableView属性，该属性指向一个UITableView对象，UITableView 内又有两个属性：delegate和dataSource，都是assign，即弱指针，以delegate为例，而且一般UITableView的代理就是UITableViewController控制器； 



- 如下图若两条线都是强指针，会引发循环引用的问题，造成内存泄漏
![](http://upload-images.jianshu.io/upload_images/3407530-e70933845931ac28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 所以，一般代理都使用weak，即如下图，一强一弱，不会引发循环引用的问题，当然也不会造成内存泄漏；
![](http://upload-images.jianshu.io/upload_images/3407530-c4911289ec038e9a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![代理对象必须是weak](http://upload-images.jianshu.io/upload_images/3407530-3dcede3678e95eb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
结论：一般，控件的代理都是控制器，而控制器又拥有着该控件，所以，为了不引发循环引用的问题，代理一般都使用weak；
