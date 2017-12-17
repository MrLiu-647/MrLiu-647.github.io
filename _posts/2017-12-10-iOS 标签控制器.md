---
title: iOS UITabBarController详解
date: 2017-12-5
categories:
- iOS
---

>UITabBarController跟UINavigationController类似，
UITabBarController也可以轻松地管理多个控制器。

# UITabBarController 标签控制器：

## View的结构：
UITabBarController它是一个控制器，所以它也有自己的View.
它view内部的结构为一个专门存放子控制器View的View，和一个UITabBar条。
**注意**：UITabBar的高度为 **49**.

## 基本使用：
1.初始化UITabBarController
2.设置UIWindow的rootViewController为UITabBarController
3.根据具体情况，通过addChildViewController方法添加对应个数的子控制器

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // 1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

    self.window.backgroundColor = [UIColor blueColor];

    // 2.设置窗口的根控制器：默认显示的是第一个子控制器的View
    UITabBarController *tabVC = [[UITabBarController alloc] init];

    // 创建第一个控制器
    UIViewController *vc1 = [[UIViewController alloc] init];
    vc1.view.backgroundColor = [UIColor redColor];
    vc1.title = @"首页";
    UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:vc1];

    nav.tabBarItem.title = @"消息";
    nav.tabBarItem.badgeValue = @"10";
    nav.tabBarItem.image = [UIImage imageNamed:@"tab_recent_nor"];

    [tabVC addChildViewController:nav]; // 添加子控制器

    // 创建第二个控制器
    UIViewController *vc2 = [[UIViewController alloc] init];
    vc2.view.backgroundColor = [UIColor blueColor];
    vc2.tabBarItem.title = @"联系人";
    vc2.tabBarItem.image = [UIImage imageNamed:@"tab_buddy_nor"];
    [tabVC addChildViewController:vc2];

    // 创建第三个控制器
    UIViewController *vc3 = [[UIViewController alloc] init];
    vc3.view.backgroundColor = [UIColor whiteColor];
    vc3.tabBarItem.title = @"动态";
    vc3.tabBarItem.image = [UIImage imageNamed:@"tab_qworld_nor"];
    [tabVC addChildViewController:vc3];

    // 初始化时，选择的子控制器
    tabVC.selectedIndex = 1;

    self.window.rootViewController = tabVC;
    // 3.显示窗口
    [self.window makeKeyAndVisible];

    return YES;
}
```

- UITabBarController添加控制器的方式有2种:

```objc
// 添加单个子控制器
- (void)addChildViewController:(UIViewController *)childController;
// 设置子控制器数组
@property(nonatomic,copy) NSArray *viewControllers;
```


## 实现原理：
- 步骤：
1 创建窗口
2 创建TabController对象，设置窗口的根控制器，
TabBarController被窗口根控制器强引用，所以不会被销毁。
3 添加TabBarController子控制器
添加的所有子控制器都保存在TabBarController的childViewControllers数组当中
所以添加的子控制器也不会被销毁
4 显示窗口（会把TabBarController的View添加到窗口上面）
- 注意点：
TabBarController默认做法是把它的第一个子控制器的View添加到TabBarController存放子控制器的View当中.（也可以设置第几个，改个属性就可以了）
如果UITabBarController有N个子控制器，那么UITabBar内部就会有N个按钮
点击每一个按钮，它会先把当前控制器的View从TabBarController存放子控件View的View当中移除（只是移除view，子控制器还在数组当中，没有移除）
再把当前选中按钮对应的子控制器的View添加到TabBarController存放子控件的View当中
