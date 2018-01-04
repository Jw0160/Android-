# Android 6.0 动态权限

标签（空格分隔）： 动态权限 文档整理

---

## 申请步骤

* 将targetSdkVersion设置为23，注意，如果你将targetSdkVersion设置为>=23，则必须按照Android谷歌的要求，动态的申请权限，如果你暂时不打算支持动态权限申请，则targetSdkVersion最大只能设置为22.

* 在AndroidManifest.xml中申请你需要的权限，包括普通权限和需要申请的特殊权限。

* 开始申请权限，此处分为3部。
    * （1）检查是否由此权限checkSelfPermission()，如果已经开启，则直接做你想做的。

    * （2）如果未开启，则判断是否需要向用户解释为何申请权限shouldShowRequestPermissionRationale。
    * （3）如果需要（即返回true），则可以弹出对话框提示用户申请权限原因，用户确认后申请权限requestPermissions()，如果不需要（即返回false），则直接申请权限requestPermissions()。

### 备注！！！
（1）checkSelfPermission：检查是否拥有这个权限
（2）requestPermissions：请求权限，一般会弹出一个系统对话框，询问用户是否开启这个权限。
（3）shouldShowRequestPermissionRationale：Android原生系统中，如果第二次弹出权限申请的对话框，会出现“以后不再弹出”的提示框，如果用户勾选了，你再申请权限，则shouldShowRequestPermissionRationale返回true，意思是说要给用户一个 解释，告诉用户为什么要这个权限。然而，在实际开发中，需要注意的是，很多手机对原生系统做了修改，比如小米，小米4的6.0的shouldShowRequestPermissionRationale 就一直返回false，而且在申请权限时，如果用户选择了拒绝，则不会再弹出对话框了。。。。 所以说这个地方有坑，我的解决方法是，在回调里面处理，如果用户拒绝了这个权限，则打开本应用信息界面，由用户自己手动开启这个权限。
（4）每个应用都有自己的权限管理界面，里面有本应用申请的权限以及各种状态，即使用户已经同意了你申请的权限，他也随时可以关闭

## 第三方框架 [AndPermissions](https://github.com/yanzhenjie/AndPermission/blob/master/README-CN.md)---[yanzhenjie](http://www.yanzhenjie.com)


### 特性

* 支持申请权限组，兼容Android8.0，最大程度上兼容国产机。
* 链式调用，一句话申请权限，不需要判断版本和是否拥有某权限。
* 支持注解回调结果、支持Listener回调结果。
* 对于某个权限拒绝过一次后，下次申请可以使用RationaleDailog提示用户权限的重要性，面得被用户勾选不再提示从而再也申请不了权限（只能在系统Setting中授权）。
* 就算用户拒绝权限并勾选不再提示，可使用SettingDialog提示用户去设置中授权。
* RationaleDialog和SettingDialog允许开发者自定义。
* AndPermission自带默认对话框除可自定义外，也支持国际化。
* 支持在任何地方申请权限，不仅限于Activity和Fragment等。

### 引用方法

``` groovy
compile 'com.yanzhenjie:permission:1.1.2'
```

### 流程

 `注入申请权限数组` --> `注册监听(PermissionListener,RationaleListener)` -->`checkSelfPermission` --> `跳转申请权限界面(PermissionActivity)` -->`shouldShowRequestPermissionRationale` --> `requestPermissions` --> `得到回调onRequestPermissionsResult` --> `判断是否成功`







