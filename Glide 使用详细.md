# Glide 使用详细

标签（空格分隔）： 文档整理 图片加载

---
[TOC]
## 简介

Glide是一个Android图片加载开源库  https://github.com/bumptech/glide

主要作用:加载图片

## 功能特点

![功能特点](https://upload-images.jianshu.io/upload_images/944365-eed879898f9228e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700 )

## 优点

* 多样化媒体加载
Glide 不仅是一个图片缓存，它支持 Gif、WebP、缩略图。甚至是 Video

* 生命周期集成
通过设置绑定生命周期，我们可以更加高效的使用Glide提供的方式进行绑定，这样可以更好的让加载图片的请求的生命周期动态管理起来

* 高效的缓存策略
A. 支持Memory和Disk图片缓存
B. Picasso 只会缓存原始尺寸的图片，而 Glide 缓存的是多种规格，也就意味着 Glide 会根据你 ImageView 的大小来缓存相应大小的图片尺寸
> 比如你 ImageView 大小是200\*200，原图是 400\*400 ，而使用 Glide 就会缓存 200\*200 规格的图，而 Picasso 只会缓存 400\*400 规格的。这个改进就会导致 Glide 比 Picasso 加载的速度要快，毕竟少了每次裁剪重新渲染的过程，非常灵活 & 加载速度快

C. 内存开销小
默认的 Bitmap 格式是 RGB\_565 格式，而 Picasso 默认的是 ARGB_8888 格式，这个内存开销要小一半。
>Android关于图片内存计算，共有四种，分别是：

> * ALPHA_8：每个像素占用1byte内存
> * ARGB_4444:每个像素占用2byte内存
> * ARGB_8888:每个像素占用4byte内存（默认，色彩最细腻=显示质量最高=占用的内存也最大）
> * RGB_565:每个像素占用2byte内存（8bit = 1byte）
举例说明：一个32位的PNG=ARGB_8888=1204x1024,那么占用空间是:1024x1024x(32/8) = 4,194,304kb=4M左右
在解析图片的时候，为了避免oom和节省内存，最好使用ARGB_4444模式（节省一半的内存空间）

## API
### 基本
* 图片的异步加载（基础功能）
```java
  Glide.with(context).load(Url).into(targetImageView);
```
* 设置加载中以及加载失败图片
``` java
placeholder(R.mipmap.ic_launcher)
error(R.mipmap.ic_launcher)
```
* 设置加载尺寸
``` java
Glide.with(this).load(imageUrl)
    .override(800, 800)
    .into(imageView);
```
* 设置动态转换
```
Glide.with(this).load(imageUrl)
    .centerCrop()
.into(imageView);
```
* 设置下载优先级
```
Glide.with(this).load(imageUrl)
    .priority(Priority.NORMAL)
.into(imageView);
```
* 设置加载动画
``` java
Glide.with(this).load(imageUrl).animate(R.anim.item_alpha_in).into(imageView);
//api也提供了几个常用的动画：比如crossFade()
```
* 设置要加载的内容
项目中有很多需要先下载图片然后再做一些合成的功能，比如项目中出现的图文混排，该如何实现目标下
```java
Glide.with(this).load(imageUrl).centerCrop().into(new SimpleTarget<GlideDrawable>() {
            @Override
            public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
                imageView.setImageDrawable(resource);
            }
        });
```
### 多样式的媒体加载
```java
Glide
     .with(context)
     .load(imageUrl)；
     .thumbnail(0.1f)；//设置缩略图支持：先加载缩略图 然后在加载全图
                        //传了一个 0.1f 作为参数，Glide 将会显示原始图像的10%的大小。
                       //如果原始图像有 1000x1000 像素，那么缩略图将会有 100x100 像素。
     .asBitmap()//显示gif静态图片 
     .asGif();//显示gif动态图片
     .into(imageView)；

```
### 关于缓存

*  设置磁盘缓存策略
```java
Glide.with(this).load(imageUrl)
     .diskCacheStrategy(DiskCacheStrategy.ALL)
     .into(imageView);

// 缓存参数说明
// DiskCacheStrategy.NONE：不缓存任何图片，即禁用磁盘缓存
// DiskCacheStrategy.ALL ：缓存原始图片 & 转换后的图片（默认）
// DiskCacheStrategy.SOURCE：只缓存原始图片（原来的全分辨率的图像，即不缓存转换后的图片）
// DiskCacheStrategy.RESULT：只缓存转换后的图片（即最终的图像：降低分辨率后 / 或者转换后 ，不缓存原始图片

```
* 设置跳过内存缓存
``` 
Glide
  .with(this)
  .load(imageUrl)
  .skipMemoryCache(true)
  .into(imageView);
  //设置跳过内存缓存
  //这意味着 Glide 将不会把这张图片放到内存缓存中去
  //这里需要明白的是，这只是会影响内存缓存！Glide 将会仍然利用磁盘缓存来避免重复的网络请求。
```
* 清理缓存
```
Glide.get(this).clearDiskCache();//清理磁盘缓存 需要在子线程中执行 
Glide.get(this).clearMemory();//清理内存缓存 可以在UI主线程中进行
```





