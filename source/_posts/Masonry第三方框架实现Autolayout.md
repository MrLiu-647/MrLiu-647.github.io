---
title: iOS Masonry第三方框架实现Autolayout
date: 2017-09-16
categories:
- iOS
---

# Masonry：
------------
## 简介：
- 目前最流行的Autolayout第三方框架
- 用优雅的代码方式编写Autolayout
- 省去了苹果官方复杂的Autolayout代码
- 很大程度提高了开发效率

## 框架地址：
https://github.com/snapKit/Masonry

## 示例：
```Demo1
//边距约束：
-(void)test1{
    // 1.红色的View
    UIView *redView= [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];

    // 2.1添加约束：//默认mutipliedBy = 1.0 可以不写
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.top.equalTo(self.view.mas_top).multipliedBy(1.0).offset(20);
    //        make.bottom.equalTo(self.view.mas_bottom).multipliedBy(1.0).offset(-20);
    //        make.left.equalTo(self.view.mas_left).multipliedBy(1.0).offset(20);
    //        make.right.equalTo(self.view.mas_right).multipliedBy(1.0).offset(-20);
    //    }];

    // 2.2可以简化：
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.top.equalTo(self.view.mas_top).offset(20);
    //        make.bottom.equalTo(self.view.mas_bottom).offset(-20);
    //        make.left.equalTo(self.view.mas_left).offset(20);
    //        make.right.equalTo(self.view.mas_right).offset(-20);
    //    }];

    // 2.3可以再简化：
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.top.offset(20);
    //        make.bottom.offset(-20);
    //        make.left.offset(20);
    //        make.right.offset(-20);
    //    }];

    // 2.4可以再再简化：
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.top.and.left.offset(20);
    //        make.bottom.and.bottom.offset(-20);
    //    }];

    // 2.5可以再再再简化：
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.top.left.offset(20);
    //        make.bottom.bottom.offset(-20);
    //    }];

    // 2.6可以再再再再简化：
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.edges.equalTo(self.view).insets(UIEdgeInsetsMake(20, 20, 20, 20));
    //    }];

    // 2.7可以再再再再再简化：
    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.edges.insets(UIEdgeInsetsMake(20, 20, 20, 20));
    }];
}
```

```Demo2
//中心约束：
-(void)test2{
    // 1.红色的View
    UIView *redView= [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];

    // 2.1添加约束
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.width.equalTo(@100);
    //        make.height.equalTo(@100);
    //        make.centerX.equalTo(self.view.mas_centerX);
    //        make.centerY.equalTo(self.view.mas_centerY);
    //    }];

    // 2.2添加约束 : 自动转换成相应的类型  一个很庞大的枚举
//    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
//        make.width.mas_equalTo(100);
//        make.height.mas_equalTo(100);
//        make.centerX.mas_equalTo(self.view.mas_centerX);
//        make.centerY.mas_equalTo(self.view.mas_centerY);
//    }];

    // 2.3 添加约束
    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.size.mas_equalTo(CGSizeMake(100, 100));
        make.center.mas_equalTo(self.view);
    }];
}
```

```Demo3
//综合示例:
-(void)test3{
    // 1.红色的View
    UIView *redView= [[UIView alloc] init];
    redView.backgroundColor = [UIColor redColor];
    [self.view addSubview:redView];

    // 2.蓝色的View
    UIView *blueView = [[UIView alloc]init];
    blueView.backgroundColor = [UIColor blueColor];
    [self.view addSubview:blueView];

    //    // 3.蓝色View的约束
    //    [blueView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.left.mas_equalTo(self.view.mas_left).offset(30);
    //        make.bottom.mas_equalTo(self.view.mas_bottom).offset(-30);
    //        make.right.mas_equalTo(redView.mas_left).offset(-30);
    //        make.width.mas_equalTo(redView.mas_width);
    //        make.height.mas_equalTo(50);
    //    }];
    //
    //    // 4.红色View的约束
    //    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
    //        make.right.mas_equalTo(self.view.mas_right).offset(-30);
    //        make.top.mas_equalTo(blueView.mas_top);
    //        make.bottom.mas_equalTo(blueView.mas_bottom);
    //    }];


    // 3.1蓝色View的约束: 添加宏 可以去掉mas
    [blueView makeConstraints:^(MASConstraintMaker *make) {
        make.left.equalTo(self.view.left).offset(30);
        make.bottom.equalTo(self.view.bottom).offset(-30);
        make.right.equalTo(redView.left).offset(-30);
        make.width.equalTo(redView.width);
        make.height.equalTo(50);
    }];

    // 4.1红色View的约束: 添加宏 可以去掉mas
    [redView makeConstraints:^(MASConstraintMaker *make) {
        make.right.equalTo(self.view.right).offset(-30);
        make.top.equalTo(blueView.top);
        make.bottom.equalTo(blueView.bottom);
    }];

    // 更新约束:
    [blueView mas_makeConstraints:^(MASConstraintMaker *make) {
        make.height.equalTo(80);
    }];

    //删除之前添加的所有约束，重新添加下面的约束
    //    [blueView remakeConstraints:^(MASConstraintMaker *make) {
    //
    //    }];

    //with  and 可以任意写 不影响使用
}
```
