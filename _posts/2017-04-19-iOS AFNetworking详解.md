---
title: iOS AFNetworking详解
date: 2017-04-19
tags:
- 网络请求
categories:
- iOS
---

AFNetworking开源库封装了原生的方法，
iOS9.0之后，
由于NSURLConnection的弃用，
AFNetworking的使用也有一些改变。

####GET请求：
-----------

```
//GET请求
-(void)demo1{
    NSString *urlString = @"http://www.liubaiqi.cn";

    AFHTTPSessionManager *manger =[AFHTTPSessionManager manager];

    [manger GET:urlString parameters:nil progress:^(NSProgress * _Nonnull downloadProgress) {

    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"成功");
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"%@",error);
    }];
}
```

####POST请求：
-----------

```
//POST请求
-(void)demo2{
    NSString *urlString = @"http://192.168.1.101:8080";

    AFHTTPSessionManager *manger =[AFHTTPSessionManager manager];

    NSMutableDictionary *parameter= @{@"":@"",@"":@""};

    [manger POST:urlString parameters:parameter progress:^(NSProgress * _NonnulluploadProgress){

    }success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        NSLog(@"成功");
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        NSLog(@"%@",error);
    }];
}
```

####Download请求：
-----------

```
//DownLoad请求
-(void)demo3{
    //1. 创建NSURLSessionConfiguration
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];

    //2. 创建管理者对象
    AFURLSessionManager *manager = [[AFURLSessionManager alloc]initWithSessionConfiguration:configuration];

    //3. 设置url
    NSURL *url = [NSURL URLWithString:@"http://127.0.0.1/1.mp4"];

    //4. 创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //5. 下载任务
    NSURLSessionDownloadTask *downloadTask = [manager downloadTaskWithRequest:request progress:^(NSProgress * _Nonnull downloadProgress) {
        //打印下载进度
        NSLog(@"%lf",1.0*downloadProgress.completedUnitCount/downloadProgress.totalUnitCount);

    } destination:^NSURL * _Nonnull(NSURL * _Nonnull targetPath, NSURLResponse * _Nonnull response) {
        //设置下载路径
        NSURL *documentsDirectoryURL = [[NSFileManager defaultManager]URLForDirectory:NSDocumentDirectory inDomain:NSUserDomainMask appropriateForURL:nil create:NO error:nil];

        return [documentsDirectoryURL URLByAppendingPathComponent:[response suggestedFilename]];

    } completionHandler:^(NSURLResponse * _Nonnull response, NSURL * _Nullable filePath, NSError * _Nullable error) {
        //下载完成
        NSLog(@"File downloaded to : %@",filePath);
    }];
    //启动任务
    [downloadTask resume];
}
```

####Upload请求：
-----------

```
//UpLoad请求
-(void)demo4{
    //创建NSURLSessionConfiguration
    NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];

    //创建管理者对象
    AFURLSessionManager *manager = [[AFURLSessionManager alloc]initWithSessionConfiguration:configuration];

    //设置url
    NSURL *url = [NSURL URLWithString:@"http://127.0.0.1"];
    //创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    //文件路径
    NSURL *filePath = [NSURL fileURLWithPath:@"file://Users/Liu/Desktop/Note"];

    //上传任务
    NSURLSessionUploadTask *uploadTask = [manager uploadTaskWithRequest:request fromFile:filePath progress:nil completionHandler:^(NSURLResponse * _Nonnull response, id  _Nullable responseObject, NSError * _Nullable error) {
        if(error){
            //错误
            NSLog(@"Error:%@",error);
        }else{
            //成功
            NSLog(@"Success %@ %@",response,responseObject);
        }
    }];
    //启动任务
    [uploadTask resume];
}
```

####网络状态：
-----------

```
//网络状态
-(void)demo5{
    //1. 创建网络监测者
    AFNetworkReachabilityManager *manager = [AFNetworkReachabilityManager sharedManager];

    [manager setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        //监测网络改变
        switch (status) {
            case AFNetworkReachabilityStatusUnknown:
                NSLog(@"未知网络状态");
                break;
            case AFNetworkReachabilityStatusNotReachable:
                NSLog(@"无网络");
                break;
            case AFNetworkReachabilityStatusReachableViaWWAN:
                NSLog(@"蜂窝数据网络");
                break;
            case AFNetworkReachabilityStatusReachableViaWiFi:
                NSLog(@"WiFi网络");
                break;

            default:
                break;
        }
    }];
}
```

ps:本文参考组长的博客：[MrFung's Blog](http://mrfung.cn/ios/2017/03/07/school#more)
