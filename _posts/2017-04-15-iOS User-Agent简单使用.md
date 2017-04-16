---
title: iOS Uer-Agent简单使用
date: 2017-04-15
tags:
- iOS
- Uer-Agent
categories:
- iOS
---

#### - User-Agent：中文名为[用户代理](http://baike.baidu.com/item/%E7%94%A8%E6%88%B7%E4%BB%A3%E7%90%86)，简称 UA
User-Agent在http协议里，头信息中有一个 User-Agent

>作用：告诉服务器，用户客户端是什么 浏览器 / 操作系统等 的信息。在某些特殊的情况下，服务器根据 浏览器 / 操作系统 的不同类型，输出不同的内容。

>user-agent 是非常不可靠的，原因就是它是客户端自己决定并发送给服务器

在iOS中如何设置User-Agent？
```
//访问优酷视频
-(void)demo{
    //访问网络

    //1.创建网络请求
    //从外界告诉服务器，我使用iPhone手机请求的数据，如果需要自己设置网络请求，使用可变的网络请求
    NSURL *url = [NSURL URLWithString:@"http://v.youku.com/v_show/id_XMjcwMjQxMzYwNA==.html?spm=a2hww.20023042.m_223465.5~5~5~5~5~1~3!5~A&from=y1.3-idx-beta-1519-23042.223465.1-3"];

    //可变的网络请求
    NSMutableURLRequest *request =[NSMutableURLRequest requestWithURL:url];

    //告诉服务器，客户端的软件环境
    //    [request setValue:@"iPhone" forHTTPHeaderField:@"User-Agent"];//会出来iPhone 简单界面
    //2.发送网络请求

    [request setValue:@"iPhone AppleWebKit" forHTTPHeaderField:@"User-Agent"];
    //User-Agent:通过改变这个量，可以得到自己想要的一些页面
    //现在，一些大公司的页面都做成 响应式（自动适配PC端 和 移动端）的

    UIWebView *web = [[UIWebView alloc]initWithFrame:self.view.frame];

    [web loadRequest:request];

    [self.view addSubview:web];
}
```
以上就会用户代理为iPhone的AppleWebKit框架
所以会根据 iphone的AppleWebKit 输出对应的界面。
