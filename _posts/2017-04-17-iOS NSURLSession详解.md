---
title: iOS NSURLSession详解
date: 2017-04-17
tags:
- iOS
- 网络请求
categories:
- iOS
---

iOS9.0之后，NSURLConnection被苹果官方弃用
而在iOS7.0之后推出的NSURLSession迎来了它独步江湖的时代
NSURLSession为HTTP数据传输提供了一系列接口，
并在异步处理上比NSURLConnection好很多。

下面先说NSURLSession的简单用法：

#####简单GET请求：

```
//NSSession简单演示
- (void)demo1{
    //1. NSURL
    NSURL *url = [NSURL URLWithString:@"http://liubaiqi.cn"];

    //2. NSURLSession -> 苹果提供了一个全局的单例，简化程序开发
    NSURLSession *session = [NSURLSession sharedSession];

    //3. NSURLSessionDataTask -> 在 NSURLSession 开发中，所有任务都是由会话发起的！
    // session 是负责管理所有网络任务的
    NSURLSessionDataTask *task = [session dataTaskWithURL:url completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        //反序列化
        id result = [NSJSONSerialization JSONObjectWithData:data options:0 error:NULL];
        NSLog(@"%@",result);
    }];

    //4. 启动任务
    [task resume];
}

```
注：所有的task都要调用resume方法才会开始进行请求

#####简单POST请求：
POST比GET多一个request

```
-(void)demo2{
NSURL *url = [NSURL URLWithString:@"http://www.daka.com/login"];
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
request.HTTPMethod = @"POST";
request.HTTPBody = [@"username=daka&pwd=123" dataUsingEncoding:NSUTF8StringEncoding];

NSURLSession *session = [NSURLSession sharedSession];
// 由于要先对request先行处理,我们通过request初始化task
NSURLSessionTask *task = [session dataTaskWithRequest:request
                                   completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) { NSLog(@"%@", [NSJSONSerialization JSONObjectWithData:data options:kNilOptions error:nil]); }];
[task resume];
}

```

#####NSURLSession断点续传
下面一个demo通过
NSURLSessionDownloadDelegate代理方法
实现下载功能，并提供断点续传
一共有三个Button，
分别负责 “开始、暂停、继续”

````````
#import "ViewController.h"
#import "Button.h"
@interface ViewController ()<NSURLSessionDownloadDelegate>

/* 全局的网络会话，管理所有的网络任务 */
@property(nonatomic,strong)NSURLSession *session;

@property(weak, nonatomic) IBOutlet Button *progressView;

// 下载任务
@property(nonatomic,strong)NSURLSessionDownloadTask *downloadTask;
//如果使用weak弱引用，不需要清楚 downloadTask，但是开发时不建议使用，自定义控件都应该使用强引用
//@property(nonatomic,weak)NSURLSessionDownloadTask *downloadTask;
// 续传数据
@property(nonatomic,strong)NSData *resumeData;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
}

/*
 取消会话的位置应该在哪？

 1> 下载完成
 demo:
        //完成任务  如果会话已经被设置成完成，就无法再次使用
        //Attempted to create a task in a session that hasbeen invalidated
                    [self.session finishTasksAndInvalidate];
        //清空session
                    self.session = nil;

 2> 视图控制器销毁前
        //取消会话
            [self.session invalidateAndCancel];
            self.session = nil;

 *****两种方式的对比
 第一种方法：可以保证文件“可能”完整被下载完
           这种方法会重复创建和销毁 session，会造成额外的开销
 第二种方法：只是在离开界面前，销毁session，相当开销会小
           缺点：如果一个文件没有下载完成，会直接被取消掉

 真正的解决方法：
 - 在网络访问中，应该将所有的网络访问操作，封装到一个方法中，由一个统一的单例来负责处理所有的网络事件
 - Session对代理（单例）进行强引用，单例本身就是一个静态实例，本身就不需要释放
 - AFN -> 需要建立一个AFN 管理器的单例，统一负责所有的网络事件处理
 */

//懒加载
-(NSURLSession *)session{
    if (_session == nil) {
        //config 提供了一个全局的网络环境配置，包括：身份认证，浏览器类型，cookie，缓存，超时时长...
        //一旦设置可以全局共享，替代 NSURLRequest 中的附加信息
        NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];

        /*
         队列：
         - 下载本身是一步的，是NSURLSession 统一管理调度的
         - 代理工作的队列，指的是，当网络事件需要监听的时候，去执行方法所调度的队列

         参数队列：指定调度代理方法执行的队列，并不会影响到 session 本身的异步执行
         - nil 代理在异步多个线程执行
         - [[NSOperationQueue alloc] init] 和 nil 的执行效果是一样的

         如果希望代理在异步执行，直接使用nil就可以了

         - [NSOperationQueue mainQueue] 主队列
         */

        _session = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:[[NSOperationQueue alloc] init]];
        /*Session 会对代理进行强引用，如果任务执行结束后，不取消 session，会出现内存漏洞*/
    }
    return _session;
}

-(void)viewWillDisappear:(BOOL)animated{
    [super viewWillDisappear:animated];

    //取消会话
    [self.session invalidateAndCancel];
    self.session = nil;
}

-(void)dealloc{
    NSLog(@"我走了");
}


//开始下载
-(IBAction)start{
    //如果有任务下载 清空任务冲下
    if(self.resumeData!=nil){
        NSLog(@"没有暂停任务");
        _resumeData=nil;
    }

    NSURL *url = [NSURL URLWithString:@"http://127.0.0.1/1.mp4"];

    NSLog(@"开始");

    //如果要跟进下载的进度，也需要监听，和 NSURLConnection 一样 都是使用delegate！
    //NSURLSession 如果要跟进进度，不能使用系统提供的全局 session 单例，sharedSession供全局使用，供所有应用使用
    //代理有一个特点：一对一，所以不能使用全局session单例

    //如果要跟进下载进度，不能使用块代码回调的方式
    self.downloadTask = [self.session downloadTaskWithURL:url];
    //启动下载任务
    [self.downloadTask resume];
}

//暂停下载
-(IBAction)pause{
    //如果任务已经被暂停，不应该能够再次被暂停
    //取消下载任务
    //- 所有的任务都是由session发起的，任务一旦发起，session会对任务进行强引用
    //- 一旦任务背取消，session就不再对任务进行强引用，ARC中，如果没有对象对某个对象强引用，会被立即释放
    [self.downloadTask cancelByProducingResumeData:^(NSData * resumeData) {
        //参数：
        //resumeData：续传的数据
        NSLog(@"数据长度%tu",resumeData.length);

        self.resumeData = resumeData;

        //释放了下载任务
        self.downloadTask = nil;
    }];
}

//继续下载
-(IBAction)resume{
    if(self.resumeData==nil){
        NSLog(@"没有暂停任务");
        return;
    }
    //所以下载任务都是由Session发起的
    //使用续传数据启动下载
    //续传数据的作用就是建立新的下载任务，一旦下载任务建立后，续传数据就没用了
    self.downloadTask = [self.session downloadTaskWithResumeData:self.resumeData];

    //清空续传数据
    self.resumeData = nil;

    //所有任务默认都是挂起的
    [self.downloadTask resume];
}


#pragma mark - NSURLSessionDownloadDelegate 代理方法
/**
 在   iOS 7.0中，以下三个方法都是必须的，
 到了  iOS 8.0中，只有 下载完成方法是必须的

 如果需要支持 iOS 7,以下三个方法都要实现
 */
// 1.下载完成方法
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location{

    NSLog(@"完成 %@",location);

}

// 2.下载进度
/**
 1) session
 2) downloadTask                调用代理方法的下载任务
 3) bytesWritten                本次下载的字节数
 4) totalBytesWritten           已经下载的字节数
 5) totalBytesExpectedToWrite   期望下载的字节数->文件总大小
 */
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite{

    float progress = (float) totalBytesWritten/totalBytesExpectedToWrite;
    NSLog(@"%f %@",progress,[NSThread currentThread]);

    //主线程更新UI
    dispatch_async(dispatch_get_main_queue(), ^{
        self.progressView.progress = progress;
    });
}

// 3.下载续传数据
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes{

    //可以什么都不用写
}
@end
``````

NSURLSession对内存异步处理的优化非常好
下载时，内存最高仅为33.7MB

![NSURLSession详解](http://upload-images.jianshu.io/upload_images/3407530-0939dddd27d1720a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
