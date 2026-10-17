---
title: iOS 沙盒机制 & 数据存储
date: 2017-12-18
categories:
- iOS
---

>iOS中，数据存储有四种方式：
plist存储、偏好设置、NSCoding归档、数据库
其中，前三种存储在iPhone的沙盒里，后一种存储在数据库中。

在学习iOS存储方法之前，先了解一下iOS存储机制——**沙盒**
**应用沙盒机制**：每个iOS应用都有自己的应用沙盒（文件系统目录），与其他文件系统隔离。每个应用必须在自己的沙盒里运行，其他应用不能访问该沙盒。
![沙盒目录](http://upload-images.jianshu.io/upload_images/3407530-6f73807f38afba25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 沙盒结构分析：
  - 应用程序包：包含了所有的资源文件和可执行文件。
  - Documents：保存应用运行时成成的**需要持久化的数据**，iTunes同步设备时会**备份**该目录。（例如，游戏应用可将游戏存到保存在该目录。大型数据不能存放在这里，一旦存放，iOS审核不会通过）
  - tmp：保存应用运行时所需的**临时数据**，使用完毕后再将相应的文件从该目录**删除**。应用没有运行时，系统也可能会清除该目录下的文件。iTunes同步设备时**不会备份**该目录。
  - Library/Caches：保存应用运行时生成的**需要持久化的数据**，iTunes同步设备时**不会备份**该目录。一般存储体积大、不需要备份的非重要数据。（一般把缓存下载好的文件放在这里）
  - Library/Preference：保存应用的所有**偏好设置**，iOS的settings（设置）应用会在该目录中查找应用的设置信息。iTunes同步设备会**备份**该目录。
- 打印沙盒路径的方法：
```objc
   NSLog(@"%@",NSHomeDirectory);
```


#### 一、XML属性列表（plist）归档：
数据存储是保存在手机里面的
    plist文件存储一般都是存取字典和数组,直接写成plist文件,把它存到应用沙盒当中.
    只有在ios当中才有plist存储,它是ios特有的存储方式.
```objc
// 存数据
- (IBAction)saveBtnClick:(id)sender {

    //模型：
    NSArray *dataArr = @[@"BQ",@10];
    NSDictionary *dataDict = @{ @"name" : @"BQ",@"age" : @20};

    /*
        参数：
        1.搜索的参数
        2.搜索的范围
        3.是否展开路径(iOS不识别~) iOS应用这个参数 必须是YES
     */
    NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask, YES)[0];
    NSLog(@"%@",path);

    // 拼接一个文件名：自动加一个斜杠，拼接文件专用
    NSString *filePath = [path stringByAppendingPathComponent:@"data.plist"];
    NSString *dictPath = [path stringByAppendingPathComponent:@"dict.plist"];

    // 路径是沙盒的路径
    [dataArr writeToFile:filePath atomically:YES];
    [dataDict writeToFile:dictPath atomically:YES];
}

// 读数据
- (IBAction)readBtnClick:(id)sender {
    NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask, YES)[0];

    // 数组
    NSString *filePath = [path stringByAppendingPathComponent:@"data.plist"];
    NSArray *dataArr = [NSArray arrayWithContentsOfFile:filePath];
    NSLog(@"%@",dataArr);

    // 字典
    NSString *dictPath = [path stringByAppendingPathComponent:@"dict.plist"];
    NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:dictPath];
    NSLog(@"%@",dict);
}
```

#### 二、Preference（偏好设置）：
偏好设置NSUserDefaults：
底层就是封闭了一个字典,利用字典的方式生成plist文件
好处:不需要关心文件名(它会自动生成)快速进行键值对存储.
一般用于存储账号密码等信息。
**注意**：存是用什么key存的, 读的时候就要用什么key值取
存的时候使用的什么类型,取的时候也要用什么类型.
```objc
// 存数据
- (IBAction)save:(id)sender {
    // NSUserDefaults它保存的也是一个plist文件
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    [defaults setObject:@"BQ" forKey:@"name"];
    [defaults setInteger:20 forKey:@"age"];

    // 同步调用，立刻写到文件中,不写这个方法会异步，有延迟
    [defaults synchronize];

    NSLog(@"%@",NSHomeDirectory());
}

// 读数据
- (IBAction)read:(id)sender {
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
    NSString *name = [defaults objectForKey:@"name"];
    NSInteger *age = [defaults integerForKey:@"age"];

    NSLog(@"%@-------%ld",name,age);

}
```
#### 三、NSKeyedArchiver归档（NSCoding）：保存自定义对象
归档一般都是保存自定义对象的时候,使用归档.因为plist文件不能够保存自定义对象.
如果一个字典当中保存有自定义对象,如果把这个字典写入到文件当中,它是不会生成plist文件的.

```objc
// 存数据：
- (IBAction)save:(id)sender {
    Person *per = [[Person alloc] init];
    per.name = @"LSQ";
    per.age = 10;

    Cat *cat = [[Cat alloc] init];
    cat.name = @"wangCai";
    per.cat = cat;

    // 获取沙盒目录
    NSString *tempPath = NSTemporaryDirectory();
    NSString *filePath = [tempPath stringByAppendingPathComponent:@"Person.data"];
    NSLog(@"%@",tempPath);

    // 归档：注意：archiveRootObject会调用encodeWithCoder:方法 得自己写
    [NSKeyedArchiver archiveRootObject:per toFile:filePath];
}

// 读数据：
- (IBAction)read:(id)sender {
    // 获取沙盒目录
    NSString *tempPath = NSTemporaryDirectory();
    NSString *filePath = [tempPath stringByAppendingPathComponent:@"Person.data"];
    // unarchiveObjectWithFile会调用initWithCoder
    Person *per = [NSKeyedUnarchiver unarchiveObjectWithFile:filePath];
    NSLog(@"%@-----%@",per.name,per.cat.name);
}
```
#### 四、SQLite3 & Core Data：
SQL语句+服务器数据库
//...未完待续
