---
title: SDWebImage常用方法
date: 2014-12-12 17:35:16
categories: 第三方库
tags: [ThirdLibrary] 
description: SDWebImage几乎每个项目都会用到这个强大的第三方网络加载库,在这里记载一下他的常用方法,以后会陆续补充。
---

#SDWebImage中那些好用的方法
[SDWebImage](https://github.com/rs/SDWebImage)主要作用:
> 一个异步下载图片并且支持缓存的UIImageView分类

[SDWebImage](https://github.com/rs/SDWebImage)是iOS开发者几乎都会用到的一个第三方库，Star数量以上万，使用其提供的方法,一句话就能让`UIImageView`自动去加载并显示网络图片,特别实在`UITableViewCell
`中有需要显示来自网络的图片,进行异步加载[SDWebImage](https://github.com/rs/SDWebImage)会自动去管理这些图片, 包括缓存到内存和缓存到磁盘等等。包括gif图片的显示也是轻松完成。本文主要分享除了基本方法以外的一些其他给力方法。

## 图片下载
图片下载有两种方式。

```
// 方法1:
[[SDWebImageDownloader sharedDownloader] downloadImageWithURL:url options:0 progress:^(NSInteger receivedSize, NSInteger expectedSize) {
// 下载进度block
} completed:^(UIImage *image, NSData *data, NSError *error, BOOL finished) {
// 下载完成block
}];

// 方法2:
[[SDWebImageManager sharedManager] downloadImageWithURL:url options:0 progress:^(NSInteger receivedSize, NSInteger expectedSize) {
// 下载进度block
} completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
// 下载完成block
}];
```

以上两个方法可以下载图片，但是肯定是有差别的，都提供下载进度回调和下载完成回调，不同点:

* 方法1和方法2是两个不同的类，方法1(`SDWebImageDownloader`)只下载图片，不管理图片，不会缓存图片到内存和本地。方法2(`SDWebImageManager`) 既下载图片，又管理图片。其实方法2内部也是调用的方法1。只不过方法2会缓存图片，并且管理下载队列。
* 下载完成之后的回调传递的参数不一样，方法1会将图片的NSData数据和传过来，而方法2只传递了image对象。
* 方法1的`completed`回调不是在主线程，如果需要在主线程做事情，得手动回到主线程。方法2的`completed`回调是主线程，在做进一步处理时就不用再手动回到主线程了。

## 图片存储(SDImageCache类)
可以将手动将图片通过SDWebImage存储，由SDWebImage统一管理。

```
/**
 * 方法1
 * @param key    图片的url路径
 * @param toDisk 是否要缓存到磁盘
 */
- (void)storeImage:(UIImage *)image forKey:(NSString *)key toDisk:(BOOL)toDisk;

/**
 * 方法2
 * @param recalculate 是否需要将image转换为data 
 * @param imageData   图片的NSData对象
 */
- (void)storeImage:(UIImage *)image recalculateFromImage:(BOOL)recalculate imageData:(NSData *)imageData forKey:(NSString *)key toDisk:(BOOL)toDisk;
```
* 方法1只能存储png或者jpg图片，不能存储gif图片，方法内部会对image对象进行转换，然后写到本地。
* 方法2就相对给力了，因为提供了`imageData`参数，可以将图片的`NSData`对象传过去，什么类型的图片都可以存储。这里需要注意的是，如果提供了`imageData`, 那么`recalculate`传递`NO`就可以了, 方法内部就不会再拿`UIImage`对象去转换成`NSData`对象了了。

### 获取缓存图片的路径
有时候可能需要拿到缓存图片的路径做做一些事情，SDWebImage也提供了相应的方法。

```
url是图片的网络路径
NSString *imageCachePath = [[SDImageCache sharedImageCache] defaultCachePathForKey:url];
```

## 获取缓存图片
```
// url是图片的网络路径
UIImage *image = [[SDImageCache sharedImageCache] imageFromDiskCacheForKey:src];
```
调用这个方法时，`SDImageCache`会先到内存中查找图片，如果没找到，再到磁盘上找。

## 获取缓存占用空间
```
[[SDImageCache sharedImageCache] getSize];
```
如有问题,请联系GiantForJade@163.com。

