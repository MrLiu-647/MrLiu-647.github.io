---
title: Objective-C基础加强整理
date: 2017-06-22
categories:
- Objective-C
---
>前言：本篇作为自己对Objective-C的基础加强篇总结，
整理各种OC知识点，会持续更新。
复习一下，也为以后做准备。
第一次整理Objective-C的内容，如有错误和欠缺，
希望大神们提出，我修改。

## 内存管理：MRC
------------
### 管理内存的分配和释放
>分配：alloc、new、copy 分配内存

>释放：release

### 什么时候下会释放内存？
>当计数器值为0时会自动释放

### 怎么证明对象即将被释放？
>重写dealloc方法
要求：
必须要调用父类的dealloc方法

### 内存管理的原则：
>谁创建，谁释放。
谁retain，谁release

### set的方法的内存管理
```set
- (void)setCat:(Cat *)cat
{
   if(_cat != cat)
   {
       [_cat release];
       _cat = [cat retain];
   }
}
```

### @property的标准格式和参数
>- 格式：
@property(参数1,参数2)成员变量类型 成员变量名称（去_） ;
- 参数：
1) nonatomic:非原子性,线程不加锁,性能高,安全性低
atomic:原子性,线程加锁,性能低,安全性高.
>2) retain:用于对象类型,生成set方法的内存管理代码.
assign:用于基本数据类型,不生成内存管理代码,直接进行赋值操作.
>3) readwrite:可读可写.
readonly:只读
>4) setter:更改set方法的名字,注意:setter:
getter:更改get方法的名字.

### @class
>简单申明一个类
跟#import的区别：
import 引入文件里面所有的内容，包括成员变量和方法。
@class 仅告诉编译器，有这个类
建议：在.h文件里使用@class

### 循环retain：
>A包含B，B包含A
对象都无法释放。
解决方法：一边用assign，一边用retain

### NSString类的内存管理
>NSString是系统单独做处理的，不能对其进行内存管理操作

```Demo
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {

    NSMutableArray *ary = [[NSMutableArray array] retain];

    NSString *str = [NSString stringWithFormat:@"test"];
    [str retain];

    [ary addObject:str];
    NSLog(@"%@ %d",str,[str retainCount]);

    [str retain];
    [str release];
    [str release];
    NSLog(@"%@ %d",str,[str retainCount]);

    [ary removeAllObjects];
    NSLog(@"%@ %d",str,[str retainCount]);

    return 0;
}
//注：NSString 不能手动管理内存 而是由系统分配内存
//运行结果：全为-1
//2017-06-13 09:35:18.183653+0800 OC[602:14290] test -1
//2017-06-13 09:35:18.183846+0800 OC[602:14290] test -1
//2017-06-13 09:35:18.183868+0800 OC[602:14290] test -1
//Program ended with exit code: 0
```


## 内存管理：ARC
-------------
### 自动释放池
>以前在创建1个对象时,需要在合适的位置调用release.
自动释放池的作用:
对池子里面的所有对象,在池子被释放时,统一做一次release操作.前提:调用autorelease,当池子释放时,才会对对象做release操作.

### ARC的工作原理：
>ARC不是垃圾回收，而是在编译阶段，在代码合适的地方插入retain、release、autorelease方法

### strong和weak指针
>在ARC中引入strong和weak指针，
`如果对象有强指针引用则保留对象的存在，否则立即回收。`
```Strong
__strong Person *p = [[Person alloc] init];
```
```Weak
__weak Person *p = [[Person alloc] init];
```
注意：
1）有强指针引用，则对象存在。
2）无强指针，即无引用或者弱指针引用，都会立即回收对象，指针赋值为nil.
![strong & weak](http://upload-images.jianshu.io/upload_images/3407530-2a0e9502ca95b75e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ARC中循环引用问题：
>解决方案：一边用strong，一边用weak
如果两边同时strong会发生内存泄漏。

### ARC下 @property参数：
>strong用于OC对象，相当于MRC中的retain
weak用于OC对象，相当于MRC中的assign
assign用于基本类型，与MRC中assign一样
copy一般用于NSString，与MRC中copy一样
![ARC property](http://upload-images.jianshu.io/upload_images/3407530-24b0016eb5877bb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### ARC使用注意事项：
>1.不允许用release,retain,retainCount
2.允许重写dealloc方法，但是不允许调用[super dealloc]
3.ARC中只要弱指针指向的对象不存在了，该指针会自动清空，赋值为nil
4.下面代码不能写，不合理。 一创建就回收。
```weak
__weak Person *p =[[Person alloc] init];
```

## Category 范畴/分类
-------------
### 范畴作用：
>1.对类的结构进行分类管理，便于团队开发以及维护
2.对类的方法进行扩展，提供新增方法
3.模拟现私有方法

```Category
//NSString+numeric类.h
#import <Foundation/Foundation.h>

@interface NSString(numeric)

-(BOOL)isNum;

@end


//NSString+numeric类.m
#import "NSString+numeric.h"
@implementation NSString(numeric)
-(BOOL)isNum{
    for (int i=0; i<self.length; i++) {
        unsigned short ch = [self characterAtIndex:i];
        if (ch<'0'||ch>'9') {
            return false;
        }
    }
    return true;
}
@end


//main函数
#import <Foundation/Foundation.h>
#import "NSString+numeric.h"

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSString *str = @"123";
        BOOL isTrue = [str isNum];
        NSLog(@"%d",isTrue);

        NSString *str2 = @"abc";
        BOOL isTrue2 = [str2 isNum];
        NSLog(@"%d",isTrue2);
    }
    return 0;
}
```
### Category使用的注意事项：
>1.分类中只能添加方法，不能添加成员变量
2.分类中不允许属性语法
3.出现重名方法，由编译顺序决定方法内容

## 非正式协议：informal protocol 继承
-------------
>给父类创建的分类，由子类实现方法。

```protocol
//Person.h
#import <Foundation/Foundation.h>
@class Car;
@interface Person : NSObject
@property(nonatomic,weak)Car *car;

-(void)sayHello;

@end

//Student.h

#import "Person.h"

@interface Student : Person

@end

@implementation Student

-(void)sayHello{
    NSLog(@"Hello,大家好！");
}

@end
```
>很简单不多说。

## 类拓展 class extension
--------------------
>所谓类扩展，是一个特殊的分类。只是普通分类圆括号中有分类的名字。但是类扩展没有。一般在.m文件中也会有。
>也很简单，不多说。
```Extension
@interface person()
.....
@end
```

## Block 代码块/闭包
-------
![Block](http://upload-images.jianshu.io/upload_images/3407530-b0416383ed0b5cda.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>block允许访问外部变量，但是不能改变，而是拷贝一个同样的变量，调用完block会回收，而外部变量不会发生改变。

## protocol 协议
--------
```protocol
@protocol 协议名称<NSObject>  //默认遵守NSObject 协议
//方法申明
@end
```

```interface
@interface 类名：父类名 <协议1，协议2，...>
@required（默认，必须实现）
//成员方法...
@optional（可以不实现）
//成员方法...
@end
```
>只要采纳了协议，相当于已经声明了协议中的方法，只需要在implementation中实现该方法。
OC不支持多继承，但支持多协议。

### protocol使用注意：
>1.protocol不能声明成员变量
2.某个类遵守了协议，就拥有了协议中的所有方法申明
3.父类遵循某个协议，子类也遵守
4.OC不能多继承，但能多协议
5.协议可以遵循协议

## 代理设计模式：
--------
![iOS代理设计模式](http://upload-images.jianshu.io/upload_images/3407530-533f829aacad6c5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>1.有一个协议，关联属性和代理对象
2.一个目标对象，可以有多个代理对象
3.在目标对象中，有id<协议>类型的属性，作为代理属性，可以赋值代理对象
4.给目标对象发送消息，执行代理对象中的方法
5.代理对象也实现协议中方法，是真正执行代码的对象，目标对象只是发出一个通知。

### 代理模式常见类型：
>1.监听模型：利用代理对象监听目标的行为，一代目标对象某个特征改变，触发代理对象的行为（UIScrollView模型）
2.通知模型：目标对象在某个条件下，通知代理对象做某些事情（开枪，听音乐等）
3.附加行为模型：需要为对象附加一些方法，但是不希望利用继承完成，所以附加代理对象完成。

## NSString
----------
>1.stringWithFormat将格式化的字符串返回
```stringWithFormat
  NSString *s1 = [NSString stringWithFormat:@"一个字符串"];
  NSString *s2 = [NSString stringWithFormat:@"一个字符串|%@",s1];
  NSLog(@"%@",s1);
  NSLog(@"%@",s2);
```
2.stringWithUTF8String OC中没有字符串输入函数，只有用C语言中的输入函数，再将C字符串转换成OC字符串。
````stringWithUTF8String
char arr[256];
scanf("%s",arr);
NSString *s = [NSString stringWithUTF8String:arr];
````
3.从文件或URL获得字符串
```String
- initWithContentsOfURL
- intitWithContentsOfFile
+ stringWithContentsOfURL
+ stringWithContentsOfFile
```
4.比较大小：
```compare
- (NSComparisonResult)compare:(NSString *)string;
```
5.前后缀检查：
```fix
- (BOOL)hasPrefix:(NSString *)aString;
- (BOOL)hasSuffix:(NSString *)aString;
```
6.查找字符串：返回首次出现位置
```findString
- (NSRange)rangeOfString:(NSString *)aString;
```
NSRange：Foundation中频繁使用的结构体
![NSRange结构体](http://upload-images.jianshu.io/upload_images/3407530-da7aa5b521b0fdb7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
7.字符串的截取：
```subString
- (NSString *)substringFromIndex:(NSUInteger)from; //从开头到结尾
- (NSString *)substringToIndex:(NSUInteger)to; //开头到该位置
- (NSString *)substringWithRange:(NSRange)range; //截取NSRange
```
8.字符串的替换
```Replace
- (NSString *)stringByReplacingOccurrenceOfString:(NSString *)target;
```

## NSMutableString
--------
>1.追加：
```append
- (void)appendString:(NSString *)aString;
- (void)appendFormat:(NSString *)format,...;
```
2.插入：
```insert
- (void)insertString:(NSString *)aString atIndex:(NSUInteger)loc;
```
3.删除：
```delete
- (void)deleteCharactersInRange:(NSRange)range;
```
4.替换
```replaceC
- (void)replaceCharactersInRange:(NSRange)range withString:(NSString)...
```

## NSArray
--------
>1.写入文件：
```writeToFile
- (BOOL)writeToFile:(NSString *)path atomically:(BOOL)useAuxiliaryFile...
```
2.读取数据：
```readFile
+ (NSArray *)arrayWithContentsOfFile:(NSString *)path;
- (NSArray *)initWithContentsOfFile:(NSString *)path;
```
3.数组连接成字符串
```joinByString
- (NSString *)componentsJoinedByString:(NSString *)separator;
```
4.字符串拆成数组
```SeparatedByString
- (NSArray *)componentsSeparatedByString:(NSString *)separator;
```

## NSMutableArray
-----------
>1.新增数据：
```addObject
- (void)addObject:(id)anObject;
- (void)insertObject:(id)anObject atIndex:(NSUInteger)index;
```
2.删除数据：
```remove
- (void)removeLastObject;
- (void)removeObjectAtIndex:(NSUInteger)index;
- (void)removeObject:(id)anObject;
```
3.修改数据：
```replaceMutable
- (void)replaceObjectAtIndex:(NSUInteger)index withObject:(id)anObject...
```
4.交换元素：
```exchange
- (void)exchangeObjectAtIndex:(NSUInteger)idx1 withObjectAtIndex:...
```

## NSDictionary
------------
1.字典的数据存储，键值必须一一对应，同时键不允许重复
2.键值对是散列数据，是没有顺序的
3.键值对集合利用count返回长度，表示有几组键值对
4.键值对集合使用下标访问对应键的值，早期使用objectForKey方法

>1.创建字典
```createDictionary
+ dictionaryWithObject:(id)object forKey:(id <NSCopying>....)
- initWithObjects:(const id [])objects forKeys:(const id ....)
+ dictionaryWithObjectsAndKeys:(id)firstObject,....
- initWithObjectsAndKeys:(id)firstObject,...
```
2.遍历字典
```ergodicity
//for-in循环遍历
for(NSString *key in dic){
      NSLog(@"%@ = %@",key, dic[key]);
}
//使用block遍历
- (void)enumerateKeysAndObjectsUsingBlock:(void(^)(id key, id obj,...));
```
3.文件操作：
```FileOperation
+ (NSDictionary *)dictionaryWithContentsOfFile:(NSString *)path;
+ (NSDictionary *)dictionaryWithContentsOfURL:(NSURL *)url;
- (BOOL)writeToFile:(NSString *)path atomically:(BOOL)useAuxiliaryFile...
- (BOOL)writeToURL:(NSURL *)url atomically:(BOOL)atomically;....
```

## NSMutableDictionary
------------
>常用方法：
```CommonlyUsedMethod
- (void)removeObjectForKey:(id)aKey;
- (void)setObject:(id)anObject forKey:(id <NSCopying>)aKey;
- (void)removeAllObjects;
//插入可以使用
dic[key]= value;
```

## NSFileManager
-------------
>NSFileManager对文件进行管理，包括创建，复制，移动，删除...
它是一个单例的存在，整个程序只允许有一个该对象。
使用defaultManager方法可以获得该对象
>1.常用方法：
```commonlyUsedMethods
- (BOOL)fileExistsAtPath:(NSString *)path;
- (BOOL)fileExistsAtPath:(NSString *)path isDirectory:(BOOL *)isDirectory...
- (BOOL)isReadableFileAtPath:(NSString *)path;
- (BOOL)isWriteableFileAtPath:(NSString *)path;
- (BOOL)isExecutableFileAtPath:(NSString *)path;
- (BOOL)isDeletableFileAtPath:(NSString *)path;
```
2.获取文件：
```attribute
- (NSDictionary *)attributesOfItemAtPath:(NSString *)path error:(NSError *)....
```
3.获取目录下内容
```subpath
- (NSArray *)subpathsAtPath:(NSString *)path;
- (NSArray *)subpathsOfDirectoryAtPath:(NSString *)path error:....
```
4.获得当前目录下内容
```contentsDirectory
- (NSArray *)contentsOfDirectoryAtPath:(NSString *)path error:(NSError *)....
```
5.对文件进行增删改查
```createFile
//创建文件
- (BOOL)createFileAtPath:(NSString *)path contents:(NSData *)data ....
//创建文件夹
- (BOOL)createDirectoryAtPath:(NSString *)path attributes:(NSDictionary *)....
- (BOOL)createDirectoryAtPath:(NSString *)path withIntermediateDirection....
//复制文件
- (BOOL)copyItemAtPath:(NSString *)srcPath toPath:(NSString *)dstPath...
//剪切文件
- (BOOL)removeItemAtPath:(NSString *)path error:(NSError **)error....
```

## SandBox 沙盒
-------------
>沙盒是文件系统目录，苹果没有开放安全、隐私等方面的内容。
所有的app都是在自己的沙盒中运行，不能实现跨沙盒运行。

>这种架构主要出于两方面考虑：
1.完美的用户体验，需要对跨应用程序进行整合统一
2.封闭跨应用可以保证系统的安全性

>iOS8后开放访问（extension）: 一种对几个固定系统区域的扩展机制，在一定程度上弥补iOS的沙盒机制对应用间通信的限制。它为用户提供了在其他应用提供的服务的便捷方式。

获取沙盒路径：
```homeDirectory
//Home目录
NSString *homeDirectory = NSHomeDirectory();
```
获取绝对路径的方法：
```SearchPath
NSSearchPathForDirectoriesInDomains;
```
第一个参数：要获取的沙盒文件夹名称
第二个参数：
```Mask
NSUserDomainMask = 1,    //用户主目录中
NSLocalDomainMask = 2,  //当前机器中
NSNetworkDomainMask = 4,  //网络中可见的主机
NSSystemDomainMask = 8,    //系统目录，不可修改（/System）
NSAllDomainsMask = 0x0fff,   //全部
```
第三个参数：YES/NO 是否获取全路径

## 常见结构体
-------
>页面布局：
NSPoint 与 CGPoint
NSSize 与 CGSize
NSRect 与 CGRect

>CoreGraphics
Make函数
官方建议使用CG。

## NSNumber
------------------
>在OC中，数组和字典不允许存储基础数据类型（int，float等）
要想存储基本数据类型，就包装厂NSNumber对象类型，再存入字典数组
![NSNumber方法](http://upload-images.jianshu.io/upload_images/3407530-188c098d8cb58a87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 十九、NSValue
-----------------
>存储C或OC数据，是一个数据的容器，相当于对NSNumber的扩展
```value
+ (NSValue *)valueWithPoint:(NSPoint)point;
+ (NSValue *)valueWithSize:(NSSize)size;
+ (NSValue *)valueWithRect:(NSRect)rect;
```

## 二十、NSDate
------------------
>1.获取当前时间：
```date
[NSDate date];
```
2.格式化日期：
使用NSDateFormatter的dateFormat属性设置时间格式字符串
```fun
- (NSString *)stringFromDate:(NSDate *)date;
- (NSDate *)dateFromString:(NSString *)string;
```
3.计算：
OC中使用NSTimeInterval表示时间间隔
在当前时间上添加秒数：
```addTimeInterval
- (id)addTimeInterval:(NSTimeInterval)seconds NS_DEPRECATED...
+ (instancetype)dateWithTimeIntervalSinceNow:(NSTimeInterval)secs;
```

## copy
---------------------
>copy就是创建一个副本与源没有直接关系，修改其中一个，对另一个没有影响

>1. OC中的copy就是对象的复制：
一个对象可以调用一个copy方法或mutableCopy方法进行拷贝
copy创建的是不可变副本（NSString、NSArray、NSDictionary）
mutableCopy创建可变副本

>2. 使用copy与mutableCopy：
copy：遵守NSCopying协议，实现copyWithZone方法
mutableCopy：遵守NSMutableCopying协议，实现mutableCopyWithZone方法

>3. 深拷贝与浅拷贝：
深拷贝：产生了新的对象，是两个对象，创建了一块新的内存地址
浅拷贝：没有产生了新的对象，是一个对象，使用源内存地址

>4.@property中的copy
MRC中：
1）copy:用于NSString/Block
2）retain：除NSString/Block外的OC对象
3）assign：基本数据类型、枚举、结构体（非OC对象），当两个对象互相引用，一端用retain，一端用assign.
ARC中：
1）copy：用于NSString/Block
2）strong：除NSString/Block外的OC对象
3）weak：当两个对象互相引用时，一端用strong，一端用weak
4）assign：基本数据类型、枚举、结构体（非OC对象）
```copy
//本质：
- (void)setName:(NSString *)name{
    if (_name == name) return;
    [_name release];
    _name = [name copy];
}
```

## 单例设计模式 Singleton
--------------------------
>1. 定义：类的对象成为系统中唯一的实例，提供 "唯一" 一个访问点，供客户类共享资源。

>2. 什么时候使用？
1）类只能有一个实例，并且必须从一个为人熟知的访问点对其进行访问，比如工厂方法。
2）这个唯一的实例只能通过子类化进行扩展，而且扩展的对象不会破坏客户端代码。

>3. 设计要点：
1）某个类只能有一个实例
2）他必须自行创建这个对象
3）必须自行向整个系统提供这个实例
4）为了保证实例的唯一性，需要注意用好：
```SingleMethod
-(id)copyWithZone:(NSZone *)zone
+(id)allocWithZone:(NSZone *)zone
//以下仅MRC中：
-(id)retain
-(NSUInteger)retainCount
-(oneway void)release
-(id)autorelease
```

>4. 实例代码：
```SingleTools
//SingleTools.h
@interface SingleTools : NSObject
@property(nonatomic,assign)int num;
@property(nonatomic,copy)NSString *test;
+(instancetype)shareInstances;
@end
//SingleTools.m
#import "SingleTools.h"
static SingleTools *instance = nil;
@implementation SingleTools
+(instancetype)shareInstances{
    if (instance == nil) {
        instance = [[self allocWithZone:NULL] init ];
    }
    return instance;
}
-(id)copyWithZone:(NSZone *)zone{
    return self;
}
+(id)allocWithZone:(struct _NSZone *)zone{
    @synchronized (self) {
        if (instance == nil) {
            instance = [super allocWithZone:zone];
            return instance;
        }
    }
    return instance;
}
/* MRC中：
-(id)retain{
    return self;
}
-(NSUInteger)retainCount{
    return NSUIntegerMax;
}
-(oneway void)release{
}
-(id)autorelease{
    return self;
}
*/
@end
```
