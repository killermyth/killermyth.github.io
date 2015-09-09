---
layout: post
title: Android布局优化工具Hierarchy Viewer
---


<div class="message">
在android开发中我们会经常遇到布局渲染慢的问题（本文中得‘慢’单针对布局不合理造成的渲染缓慢，不涉及由于UI线程中有不合理操作造成的卡顿），出现这样的问题我们一般首先需要去定位问题出现在哪里，本文主要介绍定位这样的问题时我们会用到的工具Hierarchy Viewer。
</div>


### 1. 如何使用Hierarchy Viewer
工具位于 **android-sdk/tools/**路径下，直接双击hierarchyviewer打开即可。打开后会看到从设备到各个view的一个树形图，选中其中一个需要被分析的view点击Load View Hierarchy按钮就会进入我们分析时会用到的主界面。在打开工具之前需要确保被分析的view在当前设备中是可见的。

> **注意**：此处有可能出现界面中只有设备没有具体view的情况，首先要确保被分析的view在当前设备中是可见的，如果是可见的还无法在工具中找到，那就有可能是由android安全控制的原因造成，官方原话为“To preserve security, Hierarchy Viewer can only connect to devices running a developer version of the Android system.”大意为因为安全原因，Hierarchy Viewer只能连接运行android开发版本的设备。具体如何解决见[ViewServer](https://github.com/romainguy/ViewServer)

主界面主要由**Tree View**、**Tree Overview**、**Properties View**、**Layout View**四部分组成，在左侧的Tree View中我们可以看到由整个view形成的一颗树，通过这颗树我们可以很容易的得到整个布局的层深。从tree view中我们可以看到一些特殊的地方比如
我们的整个布局是从一个PhoneWindow@DecorView的根view开始的，根view后面又是一个FrameLayout，再下一层才是我们在布局文件中添加的viewgroup，具体为什么就不再展开了。
点击这颗树中得某个元素在点击上方的**Profile Node**按钮会显示出一些更详细的信息如下：
{% highlight java %}
15 views
Measure:1ms
Layout:1ms
Draw:1ms
{% endhighlight %}
从中我们可以清楚的看到每个元素渲染所需要的具体时间，通过分析，我们就可以定位到哪些元素比较慢从而进行优化。

> 为了保持界面显示流畅，android需要把界面保持在60fps，android每16ms对界面进行一次重新绘制，如果16ms能够完成整个界面的绘制，那系统就会一直保持这么流畅的显示，也不会丢帧，如果16ms没有完成界面绘制那么绘制就会在32ms时再进行，这样所谓的卡顿就出现了。

### 2. 实际遇到过的布局不合理导致的问题
> * 层次过深造成的java.lang.StackOverflowError

曾在一个项目中遇到这样的问题，在布局层次比较深的情况下（>10）部分低配机型在渲染时会报StackOverflowError，之后优化布局深度（<10）后问题被解决。
谷歌官方说明默认最大布局深度为10，所以我们在布局时尽量要控制布局深度。


> 关于布局优化其实应该是形成一个比较好的编码习惯，尽量在前期去规避问题的发生，而不要等问题发生再去解决，这样还是太被动。
