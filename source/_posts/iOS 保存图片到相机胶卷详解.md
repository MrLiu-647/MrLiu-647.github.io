---
title: iOS 保存图片到相机胶卷详解
date: 2018-03-09
categories:
- iOS
---

>前言：iOS 保存图片有3种原生API：
使用C函数、AssetsLibrary框架、Photos框架
目前常使用Photos框架，它从 iOS 8 被苹果推出
而AssetsLibrary框架从 iOS 9 开始已被苹果放弃
本文将主要介绍三种方法的使用和注意点

# 方法一：使用C函数 保存图片（简单）
-----
- 方法名：UIImageWriteToSavedPhotosAlbum
只能保存图片到相册，不能自定义照片分组，使用方便，看需求

```objc
// 保存图片
- (IBAction)save:(id)sender {
    UIImageWriteToSavedPhotosAlbum(self.imageView.image, self, @selector(image:didFinishSavingWithError:contextInfo:), nil);
}

- (void)image:(UIImage *)image didFinishSavingWithError:(NSError *)error contextInfo:(void *)contextInfo{
    BQFunc();
    if (error) {
        [SVProgressHUD showErrorWithStatus:@"保存失败！"];
    } else {
        [SVProgressHUD showSuccessWithStatus:@"保存成功！"];
    }
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [SVProgressHUD dismiss];
    });
}
```

# 方法二：AssetsLibrary框架（iOS9开始已过期）
-----
- ALASSetsGroupType: 类型

```objc
ALAssetsGroupLibrary//从iTunes来的相册内容（如本身自带的向日葵照片）。
ALAssetsGroupAlbum//设备自身产生或从iTunes同步来的照片，但是不包括照片流跟分享流中的照片。(例如从各个软件中保存下来的图片)ALAssetsGroupEvent相机接口事件产生的相册
ALAssetsGroupFaces//脸部相册（具体不清楚）
ALAssetsGroupSavedPhotos//相机胶卷照片
ALAssetsGroupPhotoStream//照片流
ALAssetsGroupAll//除了ALAssetsGroupLibrary上面所的内容
```

- AssetsLibrary获取文件夹：

```objc
[self.imagePickerController.assetsLibrary enumerateGroupsWithTypes:[typeunsignedIntegerValue]
                                                       usingBlock:^(ALAssetsGroup*assetsGroup,BOOL*stop) {
                                                       }failureBlock:^(NSError*error) {
                                                           NSLog(@"Error: %@", [errorlocalizedDescription]);
                                                       }];
```

- 获取图片：

```objc
[self.assetsGroup enumerateAssetsUsingBlock:^(ALAsset*result,NSUIntegerindex,BOOL*stop) {
if(result) {
    [assetsaddObject:result];
 }
}];

[self.assetsGroup enumerateAssetsAtIndexes:indexes options:0 usingBlock:^(ALAsset*result,NSUIntegerindex,BOOL*stop) {
}];
```
# 方法三：Photos框架（重点）
-----
## Photos框架须知：
 - 1.PHAsset : 一个PHAsset对象就代表相册中的一张图片或者一个视频
 1> 查 : [PHAsset fetchAssets...]
 2> 增删改 : PHAssetChangeRequest(包括图片\视频相关的所有改动操作)
 -  2.PHAssetCollection : 一个PHAssetCollection对象就代表一个相册
 1> 查 : [PHAssetCollection fetchAssetCollections...]
 2> 增删改 : PHAssetCollectionChangeRequest(包括相册相关的所有改动操作)

- 3.对相片\相册的任何【增删改】操作，都必须放到以下方法的block中执行
 -[PHPhotoLibrary performChanges:completionHandler:]
 -[PHPhotoLibrary performChangesAndWait:error:]

## 保存图片到相机胶卷：
- 先导入框架：

```objc
#import <Photos/Photos.h>
```

- 异步保存方法：

```objc
    // 异步执行修改操作
    [[PHPhotoLibrary sharedPhotoLibrary] performChanges:^{
        [PHAssetChangeRequest creationRequestForAssetFromImage:self.imageView.image];
    } completionHandler:^(BOOL success, NSError * _Nullable error) {
        if (error) {
            [SVProgressHUD showErrorWithStatus:@"保存失败！"];
        } else {
            [SVProgressHUD showSuccessWithStatus:@"保存成功！"];
        }
        dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
            [SVProgressHUD dismiss];
        });
    }];
```

- 同步保存方法：

```objc
    // 同步执行修改操作
    NSError *error = nil;
    [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
        [PHAssetChangeRequest creationRequestForAssetFromImage:self.imageView.image];
    } error:&error];
    if (error) {
        [SVProgressHUD showErrorWithStatus:@"保存失败！"];
    } else {
        [SVProgressHUD showSuccessWithStatus:@"保存成功！"];
    }
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [SVProgressHUD dismiss];
    });
```

## 创建自定义相册：

```objc
#pragma mark - 获得当前App对应的自定义相册/保存的图片

/**
 获取当前App对应的自定义相册
 */
- (PHAssetCollection *)createdCollection {
    NSString *title = [NSBundle mainBundle].infoDictionary[(__bridge NSString *)kCFBundleNameKey];   // 获得app名

    PHFetchResult<PHAssetCollection *> *collections = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeAlbum subtype:PHAssetCollectionSubtypeAlbumRegular options:nil];    //抓取所有的自定义相册

    for (PHAssetCollection *collection in collections) {    // 遍历查找当前app对应的自定义相册
        if ([collection.localizedTitle isEqualToString:title]) {    //找到对应的相册：说明已经创建完毕，直接返回
            return collection;
        }
    }

    PHAssetCollection *createdCollection = nil;
    if (createdCollection == nil){  //自定义相册没被创建过，需要创建自定义相册
        NSError *error = nil;
        __block NSString *createdCollectionID = nil;

        [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
            createdCollectionID = [PHAssetCollectionChangeRequest creationRequestForAssetCollectionWithTitle:title].placeholderForCreatedAssetCollection.localIdentifier;  // 创建一个自定义相册,并拿到相册的唯一标识
        } error:&error];
        if (error) {    //如果有值，则创建相册失败，直接返回
            return nil;
        }

        createdCollection = [PHAssetCollection fetchAssetCollectionsWithLocalIdentifiers:@[createdCollectionID] options:nil].firstObject;   //拿到唯一标识后，拿到唯一相册
    }
    return createdCollection;
}
```

## 获取保存在相机胶卷的这张图片（通过AssetID）

```objc
/**
 获取保存在相机胶卷的这张图片（通过AssetID）
 */
- (PHFetchResult<PHAsset *> *)createdAssets {
    // 同步执行修改操作
    NSError *error = nil;
    // 保存图片到相机胶卷
    __block NSString *assetID = nil;    //相片ID
    [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
        assetID = [PHAssetChangeRequest creationRequestForAssetFromImage:self.imageView.image].placeholderForCreatedAsset.localIdentifier;  // 赋值ID
    } error:&error];

    if (error) {
        return nil;
    }

    // 返回相机胶卷里的该相片
    return [PHAsset fetchAssetsWithLocalIdentifiers:@[assetID] options:nil];
}
```

## 保存图片到自定义相册
```objc
/**
 保存图片到相册
 */
- (void)saveImageIntoAlbum {
    // 1.获取相机胶卷里的该【相片】
    PHFetchResult<PHAsset *> *createdAssets = self.createdAssets;
    if (createdAssets==nil) {
        [SVProgressHUD showErrorWithStatus:@"获取相片失败!"];
        return;
    }

    // 2.获得【自定义相册】
    PHAssetCollection *createdCollection = self.createdCollection;
    if (createdCollection==nil) {
        [SVProgressHUD showErrorWithStatus:@"创建/获取相册失败!"];
        return;
    }


    // 3.保存图片到【自定义相册】
    NSError *error = nil;
    [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
        PHAssetCollectionChangeRequest *request = [PHAssetCollectionChangeRequest changeRequestForAssetCollection:createdCollection];    // 添加刚才保存的图片到【自定义相册】
        [request insertAssets:createdAssets atIndexes:[NSIndexSet indexSetWithIndex:0]];
    } error:&error];
    if (error) {    //最后的判断
        [SVProgressHUD showErrorWithStatus:@"保存失败！"];
    } else {
        [SVProgressHUD showSuccessWithStatus:@"保存成功！"];
    }

    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [SVProgressHUD dismiss];
    });
}
```

## 请求和访问相册权限
```objc
// save点击事件：保存图片至相册胶卷
- (IBAction)save:(id)sender {
    PHAuthorizationStatus oldStatus = [PHPhotoLibrary authorizationStatus];
    // 请求/检查访问权限：
    // 如果用户还没有做出选择，会自动弹框，用户对弹框做出选择后，才会调用block
    // 如果之前已经做过选择，会直接执行block
    [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
        BQLog(@"%zd",status);
        dispatch_async(dispatch_get_main_queue(), ^{
            if (status == PHAuthorizationStatusDenied && oldStatus != PHAuthorizationStatusNotDetermined) { //用户拒绝当前App访问相册
                BQLog(@"提醒用户打开开关");
                [SVProgressHUD showErrorWithStatus:@"无系统权限访问"];
            } else if (status == PHAuthorizationStatusAuthorized){  // 用户允许当前App访问相册
                [self saveImageIntoAlbum];
            } else if (status == PHAuthorizationStatusRestricted){  // 无法访问相册
                [SVProgressHUD showErrorWithStatus:@"因系统原因，无法访问相册"];
            }
        });
    }];
}
```

>感谢观看，欢迎点赞关注互相交流技术问题~
