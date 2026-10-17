---
title: iOS 纯代码编写“自定义等高tableViewCell”
date: 2017-10-08
categories:
- iOS
---

# 新建一个继承自`UITableViewCell`的子类，比如BQTgCell

```objc
@interface BQTgCell : UITableViewCell
@end
```

# 在BQTgCell.m文件中
- 重写`-initWithStyle:reuseIdentifier:`方法
    - 在这个方法中添加所有的子控件
    - 给子控件做一些初始化设置（设置字体、文字颜色等）

```objc
/**
 *  在这个方法中添加所有的子控件
 */
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
{
    if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier]) {
        // ......
    }
    return self;
}
```

- 重写`-layoutSubviews`方法
    - 一定要调用`[super layoutSubviews]`
    - 在这个方法中计算和设置所有子控件的frame

```objc
/**
 *  在这个方法中计算所有子控件的frame
 */
- (void)layoutSubviews
{
    [super layoutSubviews];

    // ......
}
```

# 在BQTgCell.h文件中提供一个模型属性，比如BQTg模型
```objc
@class BQTg;

@interface BQTgCell : UITableViewCell
/** 团购模型数据 */
@property (nonatomic, strong) BQTg *tg;
@end
```

# 在BQTgCell.m中重写模型属性的set方法
- 在set方法中给子控件设置模型数据

```objc
- (void)setTg:(BQTg *)tg
{
    _tg = tg;

    // .......
}
```

# 在控制器中
- 注册cell的类型

```objc
[self.tableView registerClass:[BQTgCell class] forCellReuseIdentifier:ID];
```

- 给cell传递模型数据

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 访问缓存池
    BQTgCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 设置数据(传递模型数据)
    cell.tg = self.tgs[indexPath.row];

    return cell;
}
```

>效果图：
![效果图](http://upload-images.jianshu.io/upload_images/3407530-0ea718768b794bc6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>实例代码：BQTableViewCell-1
https://github.com/MrLiu-647/BQTableViewCell-1
