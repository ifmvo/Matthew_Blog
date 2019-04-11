---
title: Android Glide v4 笔记
date: 2017-09-28
tag: Android
---

### Glide 关于列表的使用
在 ListView 或 RecyclerView 中加载图片的代码和在单独的 View 中加载完全一样。Glide 已经自动处理了 View 的复用和请求的取消：
```
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    String url = urls.get(position);
    Glide.with(fragment)
        .load(url)
        .into(holder.imageView);
}
```
对 url 进行 null 检验并不是必须的，如果 url 为 null，Glide 会清空 View 的内容，或者显示 placeholder Drawable 或 fallback Drawable 的内容。


> 了解：
Glide 唯一的要求是，对于任何可复用的 View 或 Target ，如果它们在之前的位置上，用 Glide 进行过加载操作，那么在新的位置上要去执行一个新的加载操作，或调用 clear() API 停止 Glide 的工作。
```
@Override
public void onBindViewHolder(ViewHolder holder, int position) {
    if (isImagePosition(position)) {
        String url = urls.get(position);
        Glide.with(fragment)
            .load(url)
            .into(holder.imageView);
    } else {
        Glide.with(fragment).clear(holder.imageView);
        holder.imageView.setImageDrawable(specialDrawable);
    }
}
```
对 View 调用 clear() 或 into(View)，表明在此之前的加载操作会被取消，并且在方法调用完成后，Glide 不会改变 view 的内容。如果你忘记调用 clear()，而又没有开启新的加载操作，那么就会出现这种情况，你已经为一个 view 设置好了一个 Drawable，但该 view 在之前的位置上使用 Glide 进行过加载图片的操作，Glide 加载完毕后可能会将这个 view 改回成原来的内容。
这里的代码以 RecyclerView 的使用为例，但规则同样适用于 ListView。
>

### AppGlideModule 和 GlideExtension 的应用

AppGlideModule 可以进行扩展、适配v3 版本等操作，GlideExtension 可以进行自定义 API 等操作
详情查看下面连链接：

https://muyangmin.github.io/glide-docs-cn/doc/getting-started.html

https://muyangmin.github.io/glide-docs-cn/doc/generatedapi.html

在 lib 库里面进行扩展 应该使用 LibraryGlideModule 而不是 AppGlideModule

### 关于占位符
placeholder(加载中显示)
error(加载错误显示)
fallback(.load(null)即url为空的时候显示)

>占位符是在主线程从Android Resources加载的
>你想加载圆形的图片，并且设置了占位符，这个时候显示的占位符并不是圆形的（官方：可以考虑自定义一个View来剪裁(clip)你的占位符，达到你的transformation的效果。）

### 选项（是tm什么鬼）

1. 请求选项
centerCrop 等效果的正确姿势是这样子的
```
.apply(centerCropTransform())
.apply(circleCropTransform())
.apply(centerInsideTransform())
.apply(placeholderOf())
```
更多请看 RequestOptions 的源码中的 static 方法
>apply() 方法可以被调用多次，因此RequestOption可以被组合使用。如果RequestOptions对象之间存在相互冲突的设置，那么只有最后一个被应用的 RequestOptions会生效。

2. 过渡选项
正确姿势
```
.transition(withCrossFade())
.transition(withCrossFade(int duration))
```
更多请看 DrawableTransitionOptions、BitmapTransitionOptions 等类的源码中的 static 方法

3. RequestBuilder 是个很牛逼的东西
RequestBuilder 是 Glide 中请求的骨架，负责携带请求的 url 和你的设置项来开始一个新的加载过程

4. 组件选项
// TODO

### 变换

1. 多重变换
单次加载中应用多个变换
```
.transform(new MultiTransformation(new FitCenter(), new YourCustomTransformation())
```
>Transformation并在多个加载中使用它，通常是很好的实践

2. ImageView的自动变换
如果scaleType是CENTER_CROP, Glide将会自动应用CenterCrop变换。如果scaleType为FIT_CENTER 或 CENTER_INSIDE，Glide会自动使用 FitCenter变换

>dontTransform() 的方法可以确保不会自动应用任何变换

### 目标（Target）

//TODO

### 过渡

为了提升性能，请在使用Glide向ListView, GridView, 或RecyclerView加载图片时考虑 ***避免使用动画*** ，尤其是大多数情况下，你希望图片被尽快缓存和加载的时候。作为替代方案，请 ***考虑预加载***，这样当用户滑动到具体的item的时候，图片已经在内存中了。

注意：
- 当交叉淡入被禁用时，正在过渡的图片会在原先显示的图像上面淡入。当交叉淡入被启用时，原先显示的图片会从不透明过渡到透明，而正在过渡的图片则会从透明变为不透明.

- 在Glide中，我们默认禁用了交叉淡入，这样通常看起来要好看一些。实际的交叉淡入，如上所述对两个图片同时改变alpha值，通常会在过渡的中间造成一个短暂的白色闪屏，这个时候两个图片都是部分不透明的。

- 不幸的是，虽然禁用交叉淡入通常是一个比较好的默认行为，当待加载的图片包含透明像素时仍然可能造成问题。当占位符比实际加载的图片要大，或者图片部分为透明时，禁用交叉淡入会导致动画完成后占位符在图片后面仍然可见。 如果你在加载透明图片时使用了占位符，你可以启用交叉淡入，具体办法是调整DrawableCrossFadeFactory里的参数并将结果传到transition()中。

### Glide 里的缓存

>默认情况下，Glide会在开始一个新的图片请求之前检查以下多级的缓存：
活动资源 (Active Resources) - 现在是否有另一个View正在展示这张图片？
内存缓存 (Memory cache) - 该图片是否最近被加载过并仍存在于内存中？
资源类型（Resource） - 该图片是否之前曾被解码、转换并写入过磁盘缓存？
数据来源 (Data) - 构建这个图片的资源是否之前曾被写入过文件缓存？

1. 磁盘缓存策略（Disk Cache Strategy）
默认的策略叫做 AUTOMATIC
.diskCacheStrategy(DiskCacheStrategy.AUTOMATIC)
它会尝试对本地和远程图片使用最佳的策略。当你加载远程数据（比如，从URL下载）时，AUTOMATIC策略仅会存储未被你的加载过程修改过(比如，变换，裁剪–译者注)的原始数据，因为下载远程数据相比调整磁盘上已经存在的数据要昂贵得多。对于本地数据，AUTOMATIC策略则会仅存储变换过的缩略图，因为即使你需要再次生成另一个尺寸或类型的图片，取回原始数据也很容易。

2. 仅从缓存加载图片

某些情形下，你可能希望只要图片不在缓存中则加载直接失败（比如省流量模式？–译者注）。如果要完成这个目标，你可以在单个请求的基础上使用onlyRetrieveFromCache方法：

```
GlideApp.with(fragment)
  .load(url)
  .onlyRetrieveFromCache(true)
  .into(imageView);
```


3. 跳过缓存

如果你想确保一个特定的请求跳过磁盘和/或内存缓存（比如，图片验证码 –译者注），Glide也提供了一些替代方案。仅跳过内存缓存，请使用skipMemoryCache():
```
GlideApp.with(fragment)
  .load(url)
  .skipMemoryCache(true)
  .into(view);
```

仅跳过磁盘缓存，请使用 .diskCacheStrategy(DiskCacheStrategy.NONE)

4. 缓存的刷新
大概就是：比如，图片验证码 ，每次验证码图片的URl是相同的，这个时候你就想禁用缓存来实现每次获取最新的图片验证码，但是拉取-解码-转换成一张新图片流畅程度或是体验都是极差的，所以可以通过刷新缓存解决这个问题。

//TODO 通过 Glide 提供的签名(signature)
