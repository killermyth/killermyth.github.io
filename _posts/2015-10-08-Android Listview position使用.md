---
layout: post
title: Android Listview postion相关总结
---


<div class="message">
长假结束，回来第一件事情就是把这一个版本中开发遇到的一些问题总结一下，本文要说的问题是listview在存在header和footer的情况下，postion会变化的问题。
</div>

### 1. onItemClick中使用position
在存在headerview或者footerview的情况下，我们会发现onItemClick方法传入的position有下面的规律，如果只有headerview，点击position实际为0的item，传入的position为1；当headerview及footerview都有的情况，点击position实际为0的item，传入的position为2，到底是什么原因导致的，我们看下listview中得setAdapter方法
{% highlight java %}
@Override
public void setAdapter(ListAdapter adapter) {
    if (null != mAdapter) {
        mAdapter.unregisterDataSetObserver(mDataSetObserver);
    }

    resetList();
    mRecycler.clear();

    if (mHeaderViewInfos.size() > 0|| mFooterViewInfos.size() > 0) {
        mAdapter = new HeaderViewListAdapter(mHeaderViewInfos, mFooterViewInfos, adapter);
    } else {
        mAdapter = adapter;
    }

    //其它的一些代码这里省略之...
}
{% endhighlight %}
我们从上面可以看到，当存在headerview或者footerview的情况下，我们传入的adapter就会被重新包装，就这样，我们原先的position发生了变化

### 2. 如何正确的使用position
> * 从adapter中获取item

在onItemClick方法中使用传入的position时，我们通过parent.getAdapter()获取包装后的adapter，这样getitem就没有问题了
{% highlight java %}
@Override
public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
    FavorCommentMsgBean commentBean = (FavorCommentMsgBean) parent.getAdapter().getItem(position);
    ...
}
{% endhighlight %}

> * setselection使用

在使用listview.setselection时，如果该listview含有header时，我们需要在setselection的参数中加入header的高度

{% highlight java %}

mListView.setSelection(position + mListView.getHeaderViewsCount());

{% endhighlight %}
