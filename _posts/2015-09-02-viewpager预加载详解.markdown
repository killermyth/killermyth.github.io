---
layout: post
title: viewpager
---

<div class="message">
viewpager是android常用控件之一，一般在tab页带滑动切换的场景下都会用到。viewpager有一个非常重要的特性，那就是预加载，预加载会为页面带来性能上得提升，但同时也会引入一些问题，在作者踩过这个坑后总结出了这篇文章。
</div>

### 1. 预加载的机制
viewpager为了提高各页面滑动切换时的流畅性，会对页面进行预加载。预加载就是第一页加载完成后，会默认将第二页也渲染出来，虽然第二页此时是不可见的。预加载数量的默认值为1，也就是默认会将当前页的下一页渲染出来，这样在滑动到下一页时由于页面已经渲染完成，就会变得很顺畅。
那我们如何调整加载数量呢？可以通过
{% highlight java %}
setOffscreenPageLimit(num);
{% endhighlight %}
注意此处num只能为>=1的值，为什么呢？来看源码
{% highlight java %}
public void setOffscreenPageLimit(int limit) {
        if (limit < DEFAULT_OFFSCREEN_PAGES) {
            Log.w(TAG, "Requested offscreen page limit " + limit + " too small; defaulting to " +
                    DEFAULT_OFFSCREEN_PAGES);
            limit = DEFAULT_OFFSCREEN_PAGES;
        }
        if (limit != mOffscreenPageLimit) {
            mOffscreenPageLimit = limit;
            populate();
        }
    }
{% endhighlight %}
> **注意：**在该方法的实现中，对入参进行了判断，如果小于DEFAULT_OFFSCREEN_PAGES就不会操作成功，而且DEFAULT_OFFSCREEN_PAGES=1

### 2. 预加载带来的问题
> * 带来的性能问题

预加载了下一个页面，在一般情况下都是不会出什么问题的。但是如果我们为了滑动的流畅性把这个值设置的非常大，比如一共5个页面将预加载数量设置为4，这时候在一些低配机型上就会出现一些意想不到的情况，比如oom。页面布局越复杂页面包含图片越多越容易出现。所以这个值要适当的做控制值不能太大，够用就好。

> * 带来的其它问题

在做产品的时候，为了方便运营同学统计，我们一般都会在用户发起某项操作时记录用户的行为。由于预加载的存在，我们还没有跳转到该页面时该页面的统计信息就发送出去了，这会大大的降低统计信息的准确性。

### 3. 如何优雅的解决问题
> 以下解决方案适用于viewpager中各页面是fragment填充

在fragment中存在一个方法（setUserVisibleHint）可以判断当前fragment是否对用户可见，我们只在可见情况下才加载数据。还有一点需要特别注意，在fragment第一次加载时，通过log可以看到fragment的生命周期如下
setUserVisibleHint
onCreateView
onResume
我们可以看到setUserVisibleHint是在onCreateView之前，这样就导致了如果我们在setUserVisibleHint中去操作ui,如果这时view还没有被inflate，这样就会出现空指针问题。下面是综合以上问题的懒加载fragment的模板
{% highlight java %}
public class Fragmenta extends Fragment {
    private boolean isPrepared;//view初始化是否完成
    private boolean isVisible;//是否对用户可见
    private TextView a;
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,Bundle savedInstanceState) {
        return inflater.inflate(R.layout.mylayout, container, false);
    }
    
    @Override
    public void onViewCreated(View view, Bundle savedInstanceState){
        super.onViewCreated(view, savedInstanceState);
        a = view.findViewById(R.id.tv);
        isPrepared = true;
        lazyLoadData();
    }
    
    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (getUserVisibleHint()) {
            isVisible = true;
            lazyLoadData();
        }
    }
    
    private void lazyLoadData() {
        if (isPrepared && isVisible) {
            //加载数据
            loadData(mydata);
        }
    }
}
{% endhighlight %}
这样实现其实最精妙的部分就是将view的加载与数据的加载分开来了，这样view的加载还是使用了预加载机制，这样滑动还是非常顺畅，同时数据的加载变成了懒加载，只有在对用户可见的情况下才去加载数据。
> 举一反三，其实所有不想被预加载的操作都可以放到loadData中，这样问题就解决了。
