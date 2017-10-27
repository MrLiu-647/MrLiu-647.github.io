---
title: iOS 纯代码编写“自定义不等高tableViewCell”
date: 2017-10-15
categories:
- iOS
---

# 给模型增加frame数据
- 所有子控件的frame
- cell的高度(cellHeight)

```objc
@interface BQStatus : NSObject
/**** 文字\图片数据 ****/
// .....

/**** frame数据 ****/
/** 头像的frame */
@property (nonatomic, assign) CGRect iconFrame;
// .....
/** cell的高度 */
@property (nonatomic, assign) CGFloat cellHeight;
@end
```

- 重写模型cellHeight属性的get方法

```objc
- (CGFloat)cellHeight
{
    if (_cellHeight == 0) {
        // ... 计算所有子控件的frame、cell的高度
    }
    return _cellHeight;
}
```

# 在控制器中
- 实现一个返回cell高度的代理方法
    - 在这个方法中返回indexPath位置对应cell的高度

```objc1
/**
 *  返回每一行cell的具体高度
 */
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath
{
    BQStatus *status = self.statuses[indexPath.row];
    return status.cellHeight;
}
```

- 给cell传递模型数据

```objc2
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    // 访问缓存池
    BQStatusCell *cell = [tableView dequeueReusableCellWithIdentifier:ID];

    // 设置数据(传递模型数据)
    cell.status = self.statuses[indexPath.row];

    return cell;
}
```

# 新建一个继承自`UITableViewCell`的子类，比如BQStatusCell

```objc3
@interface BQStatusCell : UITableViewCell
@end
```

# 在BQStatusCell.m文件中
- 重写`-initWithStyle:reuseIdentifier:`方法
    - 在这个方法中添加所有可能需要显示的子控件
    - 给子控件做一些初始化设置（设置字体、文字颜色等）

```objc4
/**
 *  在这个方法中添加所有可能需要显示的子控件
 */
- (instancetype)initWithStyle:(UITableViewCellStyle)style reuseIdentifier:(NSString *)reuseIdentifier
{
    if (self = [super initWithStyle:style reuseIdentifier:reuseIdentifier]) {
        // ......
    }
    return self;
}
```

# 在BQStatusCell.h文件中提供一个模型属性，比如BQStatus模型

```objc5
@class BQStatus;

@interface BQStatusCell : UITableViewCell
/** 微博模型数据 */
@property (nonatomic, strong) BQStatus *status;
@end
```

# 在BQStatusCell.m中重写模型属性的set方法
- 在set方法中给子控件设置模型数据

```objc6
- (void)setStatus:(BQStatus *)status
{
    _status = status;

    // .......
}
```

# 重写`-layoutSubviews`方法
- 一定要调用`[super layoutSubviews]`
- 在这个方法中设置所有子控件的frame

```objc7
/**
 *  在这个方法中设置所有子控件的frame
 */
- (void)layoutSubviews
{
    [super layoutSubviews];

    // ......
}
```

>效果图：![BQTableView-2](http://upload-images.jianshu.io/upload_images/3407530-c17733d0b70ab55c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>实例代码：BQTableViewCell-2
https://github.com/MrLiu-647/BQTableViewCell-2
