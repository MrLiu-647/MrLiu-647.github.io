---
title: iOS SDWebImage的简单使用
date: 2017-04-14
tags: iOS
categories: iOS
---


首先导入 SDWebImage :
```
#import "ViewController.h"
#import "UIImageView+WebCache.h"
#import "UIImage+GIF.h"

@interface ViewController ()
@property (weak, nonatomic) IBOutlet UIImageView *imageview;

@end
```
- 加载图片
```
//加载图片
-(void)demo1{
    NSString *imgArr = @"https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1491303372770&di=47893a1f375d797458173301bb12db0d&imgtype=0&src=http%3A%2F%2Fattach.bbs.miui.com%2Fforum%2F201501%2F29%2F154912worftckqkkv55trf.jpg";
    //参数1：图片的下载地址
    //参数2：占位图片
    //参数3：可选项:包括黑名单 优先级 缓存等等
    //参数4：进度
    //参数5：完成后执行的block(回调)
    [self.imageview sd_setImageWithURL:[NSURL URLWithString:imgArr] placeholderImage:nil options:0 progress:^(NSInteger receivedSize,NSInteger expectedSize){
//        NSLog(@"receivedSize %tu  expectedSize %tu ",receivedSize,expectedSize);
        NSLog(@"%ld",receivedSize *100/expectedSize);
    }completed:^(UIImage *image,NSError *error,SDImageCacheType cacheType,NSURL *imageURL){

    }];
}
```

- 加载gif
```
//加载gif
-(void)demo2{
    UIImage *gif =[UIImage sd_animatedGIFNamed:@"xiaomai"];
    self.imageview.image = gif;
}
```
