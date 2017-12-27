# Android性能优化 

[引用](https://mp.weixin.qq.com/s/YPwApikK57BMu-FbnxbSHg)

[链接](http://www.jianshu.com/p/9755da0f4e8f)


## 概述

 Android系统的卡顿是所有Android用户都有过的糟糕体验,即使硬件设备看上去比iPhone高的不知道哪里去了,却还是有卡顿,闪退.

究其原因是Android的碎片化问题,Android源码的开源使得市场上充斥的大量ROM ,搭配性能参差不齐的硬件

性能优化包括一下四种:

* 布局(卡顿)优化
* 避免内存泄漏
* 缩小APK体积
* 减少冗余代码


## 布局优化

布局分析工具:[HierarchyView](https://www.2cto.com/kf/201404/296960.html)

![卡顿原因](http://mmbiz.qpic.cn/mmbiz_png/MOu2ZNAwZwNJpSw1ChDcicGbluRJnfyib8eUC5icibdOy26bVASZSJxr3HoyK9UwgS3GZ0x4Qib4vnF5MRexlQ9QamQ/?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

Android中官方提到的的布局优化有三种

- <include\\>
- <merge \\>
- <ViewStub \\>

> 1.重用布局 <include\>

1. <include \\>标签可以使用单独的layout属性，这个也是必须使用的。
2. 若<include \\>标签指定了ID属性，而你的layout也定义了ID，则你的layout的ID会被覆盖
3. 在include标签中所有的android:layout\_\* 都是有效的，前提是必须要写layout_width和layout_height两个属性。
4. 布局中可以包含两个相同的include标签，引用时可以使用如下方法解决

> 2.减少视图层级<merge \\>


作用是为了防止在引用布局文件时产生多余的布局嵌套。

例如你的主布局文件是垂直布局，引入了一个垂直布局的include，这是如果include布局使用的LinearLayout就没意义了，使用的话反而减慢你的UI表现。这时可以使用<merge/>标签优化。

- <merge\\>多用于替换FrameLayout或者当一个布局包含另一个时，
- <merge\\>标签消除视图层次结构中多余的视图组。


> 3.需要时使用<ViewStub \\>

- <ViewStub \\>是一个不可见的，大小为0的View。
- <ViewStub />标签最大的优点是当你需要时才会加载，使用他并不会影响UI初始化时的性能。
- 各种不常用的布局想进度条、显示错误消息等可以使用<ViewStub />标签，以减少内存使用量，加快渲染速度。

    <ViewStub  
    android:id="@+id/stub_import"  
    android:inflatedId="@+id/panel_import"  
    android:layout="@layout/progress_overlay"  
/> 


 当你想加载布局时，可以使用下面其中一种方法：

 `((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);` 

 or  
`View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();`
  
注：ViewStub目前有个缺陷就是还不支持 <merge \\> 标签。

总结如下：

- 减少层级。合理使用 RelativeLayout 和 LinerLayout，合理使用Merge。

- 提高显示速度。使用 ViewStub，它是一个看不见的、不占布局位置、占用资源非常小的视图对象。

- 布局复用。可以通过<include\\> 标签来提高复用。

- 尽可能少用wrap_content。wrap_content 会增加布局 measure 时计算成本，在已知宽高为固定值时，不用wrap_content 。

- 删除控件中无用的属性
    

----

## 内存优化


在 Android系统中有个垃圾内存回收机制，在虚拟机层自动分配和释放内存，因此不需要在代码中分配和释放某一块内存，从应用层面上不容易出现内存泄漏和内存溢出等问题，但是需要内存管理。

Android 系统在内存管理上有一个 Generational Heap Memory 模型，当内存达到一个阈值时，系统会根据不同的规则自动释放系统认为可以释放的内存

部分 Android 应用开发人员在开发过程中并没有特别关注内存的合理使用，也没有在内存方面做太多的优化，当应用程序同时运行越来越多的任务，加上越来越复杂的业务需求时，完全依赖 Android 的内存管理机制就会导致一系列性能问题逐渐呈现，对应用的稳定性和性能带来不可忽视的影响，因此，解决内存问题和合理优化内存是非常有必要的。

![Java对象生命周期](http://mmbiz.qpic.cn/mmbiz_jpg/MOu2ZNAwZwNJpSw1ChDcicGbluRJnfyib8niaBWB4frMGRrRArkXZF9cRQP40T8hQtM18z4a95ur1icqichXVKswMbA/?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1)

一般Java对象在虚拟机上有7个运行阶段：



* 创建阶段->应用阶段->不可见阶段->不可达阶段->收集阶段->终结阶段->对象空间重新分配阶段

>常见内存泄漏场景

如果在内存泄漏发生后再去找原因并修复会增加开发的成本，最好在编写代码时就能够很好地考虑内存问题，写出更高质量的代码

这里列出一些常见的内存泄漏场景，在以后的开发过程中需要避免这类问题:


- 资源性对象未关闭。比如Cursor、File文件等，往往都用了一些缓冲，在不使用时，应该及时关闭它们。
- 注册对象未注销。比如事件注册后未注销，会导致观察者列表中维持着对象的引用。
- 类的静态变量持有大数据对象。
- 非静态内部类的静态实例。
	*  1. 首先，非静态内部类默认会持有外部类的引用。
	*  2. 然后又使用了该非静态内部类创建了一个静态的实例。
	*  3. 该静态实例的生命周期和应用的一样长，这就导致了该静态实例一直会持有该Activity的引用，导致Activity的内存资源不能正常回收。
- Handler临时性内存泄漏。如果Handler是非静态的，容易导致 Activity 或 Service 不会被回收。
- 容器中的对象没清理造成的内存泄漏。
- WebView。WebView 存在着内存泄漏的问题，在应用中只要使用一次 WebView，内存就不会被释放掉。


>优化内存空间

在移动设备上，由于物理设备的存储空间有限，Android 系统对每个应用进程也都分配了有限的堆内存，因此使用最小内存对象或者资源可以减小内存开销，同时让GC 能更高效地回收不再需要使用的对象，让应用堆内存保持充足的可用内存，使应用更稳定高效地运行。常见做法如下：

- 对象引用。强引用、软引用、弱引用、虚引用四种引用类型，根据业务需求合理使用不同，选择不同的引用类型。

- 减少不必要的内存开销。注意自动装箱，增加内存复用，比如有效利用系统自带的资源、视图复用、对象池、Bitmap对象的复用。

- 使用最优的数据类型。比如针对数据类容器结构，可以使用ArrayMap数据结构，避免使用枚举类型，使用缓存Lrucache等等。

- 图片内存优化。可以设置位图规格，根据采样因子做压缩，用一些图片缓存方式对图片进行管理等等。

---

## 安装包大小优化

应用安装包大小对应用使用没有影响，但应用的安装包越大，用户下载的门槛越高，特别是在移动网络情况下，用户在下载应用时，对安装包大小的要求更高，因此，减小安装包大小可以让更多用户愿意下载和体验产品。

![安装包的构成](http://mmbiz.qpic.cn/mmbiz_png/MOu2ZNAwZwNJpSw1ChDcicGbluRJnfyib8NIbZnJTibuklaO762J457IdZNRhDMJaPSNia5NYZSLKMk90pZTYE1gwQ/?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1)

从图中我们可以看到：

- assets文件夹。存放一些配置文件、资源文件，assets不会自动生成对应的 ID，而是通过 AssetManager 类的接口获取。

- res。res 是 resource 的缩写，这个目录存放资源文件，会自动生成对应的 ID 并映射到 .R 文件中，访问直接使用资源 ID。

- META-INF。保存应用的签名信息，签名信息可以验证 APK 文件的完整性。

- AndroidManifest.xml。这个文件用来描述 Android 应用的配置信息，一些组件的注册信息、可使用权限等。

- classes.dex。Dalvik 字节码程序，让 Dalvik 虚拟机可执行，一般情况下，Android 应用在打包时通过 Android SDK 中的 dx 工具将 Java 字节码转换为 Dalvik 字节码。

- resources.arsc。记录着资源文件和资源 ID 之间的映射关系，用来根据资源 ID 寻找资源。


**减少安装包大小的常用方案**

- 代码混淆。使用proGuard 代码混淆器工具，它包括压缩、优化、混淆等功能。

- 资源优化。比如使用 Android Lint 删除冗余资源，资源文件最少化等。

- 图片优化。比如利用 AAPT 工具对 PNG 格式的图片做压缩处理，降低图片色彩位数等。

- 避免重复功能的库，使用 WebP图片格式等。

- 插件化。比如功能模块放在服务器上，按需下载，可以减少安装包大小。

---





















