---
layout: post
title: Facebook Fresco默认配置汇总
---


<div class="message">
为了方便开发，结合源码，本文对Fresco的默认配置进行一次梳理。
</div>

### 1. 默认内存缓存控制类DefaultBitmapMemoryCacheParamsSupplier

{% highlight java %}
   * @param maxCacheSize 缓存区域大小（bytes）见如下算法
   if (maxMemory < 32 * ByteConstants.MB) {
      return 4 * ByteConstants.MB;
    } else if (maxMemory < 64 * ByteConstants.MB) {
      return 6 * ByteConstants.MB;
    } else {
      // We don't want to use more ashmem on Gingerbread for now, since it doesn't respond well to
      // native memory pressure (doesn't throw exceptions, crashes app, crashes phone)
      if (Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
        return 8 * ByteConstants.MB;
      } else {
        return maxMemory / 4;
      }
    }
   * @param maxCacheEntries 缓存区最大数量  256个
   * @param maxEvictionQueueSize 最大待清除队列大小 0x7FFFFFFF（bytes）
   * @param maxEvictionQueueEntries 最大待清除队列数量 0x7FFFFFFF
   * @param maxCacheEntrySize 单缓存条目大小 0x7FFFFFFF（bytes）
{% endhighlight %}
我们从上面可以看出，默认的缓存策略在后三个参数基本没有做限制，在缓存大小控制中考虑的还是比较多的

### 2. 默认图片质量
{% highlight java %}
   ARGB_8888
{% endhighlight %}
1个像素占4字节，480x800的图片为1500KB

### 3. 默认图片质量
{% highlight java %}
   ARGB_8888
{% endhighlight %}
1个像素占4字节，480x800的图片为1500KB
