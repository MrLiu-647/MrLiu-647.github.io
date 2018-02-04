---
title: iOS 跳转网页的四种方法
date: 2018-02-04
categories:
- iOS
---

>跳转界面 push 展示网页

# Safari ：
openURL：自带很多功能 （进度条，刷新，前进，倒退..）就是打开了一个浏览器，跳出自己的应用
# UIWebView：
没有功能，在当前应用中打开网页，自己去实现某些功能，但不能实现进度条功能（有些软件做了假进度条，故意卡到70%不动，加载完成前秒到100%）
# SFSafariViewController：
iOS9+ 专门用来展示网页 需求：既想要在当前应用展示网页，又想要safari功能
- 需要导入#import <SafariServices/SafariServices.h>框架

```objc
#pragma mark - UICollectionViewDelegate
- (void)collectionView:(UICollectionView *)collectionView didSelectItemAtIndexPath:(NSIndexPath *)indexPath{    // 小盒子的点击事件
    BQLog(@"%@",indexPath);
    // 跳转界面 push 展示网页
    /*
        1.Safari openURL：自带很多功能 （进度条，刷新，前进，倒退..）就是打开了一个浏览器，跳出自己的应用
        2.UIWebView：没有功能，在当前应用中打开网页，自己去实现某些功能，但不能实现进度条功能
        3.SFSafariViewController：iOS9+ 专门用来展示网页 需求：既想要在当前应用展示网页，又想要safari功能
            需要导入#import <SafariServices/SafariServices.h>框架
        4.WKWebView：iOS8+ （UIWebView升级版本）添加功能：1）监听进度条 2）缓存
     */
    BQSquareItem *item = self.squareItems[indexPath.row];

    if (![item.url containsString:@"http"]) {
        return;
    }

    SFSafariViewController *safariVc = [[SFSafariViewController alloc] initWithURL:[NSURL URLWithString:item.url]];
//    safariVc.delegate = self;
//    self.navigationController.navigationBarHidden = YES;
//    [self.navigationController pushViewController:safariVc animated:YES];
    [self presentViewController:safariVc animated:YES completion:nil]; // 推荐使用modal自动处理 而不是push
}
```

# WKWebView：
iOS8+ （UIWebView升级版本）添加功能：1）监听进度条 2）缓存
- 需要手动导入WebKit框架 编译器默认不会导入

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    // 添加WebView
    WKWebView *webView = [[WKWebView alloc] init];
    _webView = webView;
    [self.contentView addSubview:webView];

    // 加载网页
    NSURLRequest *request = [NSURLRequest requestWithURL:self.url];
    [webView loadRequest:request];

    // KVO监听属性改变
    /*
     KVO使用:
        addObserver：观察者
        forKeyPath：观察webview哪个属性
        options：NSKeyValueObservingOptionNew观察新值改变
     注意点：对象销毁前 一定要记得移除 -dealloc
     */
    [webView addObserver:self forKeyPath:@"canGoBack" options:NSKeyValueObservingOptionNew context:nil];
    [webView addObserver:self forKeyPath:@"canGoForward" options:NSKeyValueObservingOptionNew context:nil];
    [webView addObserver:self forKeyPath:@"title" options:NSKeyValueObservingOptionNew context:nil];

    // 进度条
    [webView addObserver:self forKeyPath:@"estimatedProgress" options:NSKeyValueObservingOptionNew context:nil];
}

- (void)viewDidLayoutSubviews{
    [super viewDidLayoutSubviews];

    _webView.frame = self.contentView.bounds;
}

#pragma mark - KVO
// 只要观察者有新值改变就会调用
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    self.backItem.enabled = self.webView.canGoBack;
    self.forwardItem.enabled = self.webView.canGoForward;
    self.title = self.webView.title;
    self.progressView.progress = self.webView.estimatedProgress;
    self.progressView.hidden = self.webView.estimatedProgress>=1;
}

- (void)dealloc {
    [self.webView removeObserver:self forKeyPath:@"canGoBack"];
    [self.webView removeObserver:self forKeyPath:@"canGoForward"];
    [self.webView removeObserver:self forKeyPath:@"title"];
    [self.webView removeObserver:self forKeyPath:@"estimatedProgress"];
}

#pragma mark - 按钮的点击事件
- (IBAction)goBack:(id)sender { // 回退
    [self.webView goBack];
}

- (IBAction)goForward:(id)sender {  // 前进
    [self.webView goForward];
}

- (IBAction)reload:(id)sender { //刷新
    [self.webView reload];
}

```
