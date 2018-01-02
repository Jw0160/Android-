# 与Javascript交互

---

[转载](https://www.jianshu.com/p/345f4d8a5cfa "原文链接")
##  交互方式总结

androd 和JS的交互,骑士就是通过webView作为桥梁,相互调用代码

> 对于Android调用JS代码的方法有2种：

- 通过WebView的**loadUrl（）**
- 通过WebView的**evaluateJavascript（String script, ValueCallback<String> resultCallback）**

> 对于JS调用Android代码的方法有3种：

* 通过 WebView的**addJavascriptInterface（）**进行对象映射
* 通过 WebViewClient 的**shouldOverrideUrlLoading ()**方法回调拦截 url
* 通过 WebChromeClient 的**onJsAlert()、onJsConfirm()、onJsPrompt（）**方法回调拦截JS对话框**alert()、confirm()、prompt（）** 消息





##  [WebView 的漏洞](https://www.jianshu.com/p/3a345d27cd42)



### WebView中，主要漏洞有三类：

1. 任意代码执行漏洞
1. 	密码明文存储漏洞
1. 	域控制不严格漏洞

#### 1.任意代码执行漏洞

*	WebView 中 addJavascriptInterface（） 接口
*	WebView 内置导出的 searchBoxJavaBridge_对象
*	WebView 内置导出的 accessibility 和 accessibilityTraversalObject 对象



>  addJavascriptInterface 接口引起远程代码执行漏洞


JS调用Android的其中一个方式是通过addJavascriptInterface接口进行对象映射,当JS拿到Android这个对象后，就可以调用这个Android对象中所有的方法，包括系统类（java.lang.Runtime 类），从而进行任意代码执行。



**B1. Android 4.2版本之后**

Google 在Android 4.2 版本中规定对被调用的函数以 @JavascriptInterface进行注解从而避免漏洞攻击

**B2. Android 4.2版本之前**

在Android 4.2版本之前采用拦截prompt（）进行漏洞修复。





> searchBoxJavaBridge_接口引起远程代码执行漏洞

**A. 漏洞产生原因**

在Android 3.0以下，Android系统会默认通过searchBoxJavaBridge_的Js接口给 WebView 添加一个JS映射对象：searchBoxJavaBridge_对象,该接口可能被利用，实现远程任意代码。

**B. 解决方案**

删除searchBoxJavaBridge_接口
    
    // 通过调用该方法删除接口
    removeJavascriptInterface（）；


### 2 密码明文存储漏洞

WebSettings默认开启密码保存功能 ：

	WebSettings.setSavePassword(true)

开启后，在用户输入密码时，会弹出提示框：询问用户是否保存密码；
如果选择”是”，密码会被明文保到 /data/data/com.package.name/databases/webview.db 中，这样就有被盗取密码的危险

**解决方案**

关闭密码保存提醒

	WebSettings.setSavePassword(false) 


### 3 域控制不严格漏洞

 	// 资源访问

    settings.setAllowContentAccess(true); // 是否可访问Content Provider的资源，默认值 true

    settings.setAllowFileAccess(true);// 是否可访问本地文件，默认值 true

    // 是否允许通过file url加载的Javascript读取本地文件，默认值 false
    settings.setAllowFileAccessFromFileURLs(false);  

    // 是否允许通过file url加载的Javascript读取全部资源(包括文件,http,https)，默认值 false
    settings.setAllowUniversalAccessFromFileURLs(false);


WebView中getSettings类的方法对 WebView 安全性的影响：

1. setAllowFileAccess（）
1. setAllowFileAccessFromFileURLs（）
1. setAllowUniversalAccessFromFileURLs（）


### 1. setAllowFileAccess（） ###
    
    // 设置是否允许 WebView 使用 File 协议
    webView.getSettings().setAllowFileAccess(true); 
    // 默认设置为true，即允许在 File 域下执行任意 JavaScript 代码

如果不允许使用 file 协议，则不会存在上述的威胁；

`webView.getSettings().setAllowFileAccess(true); ` 
   
**但同时也限制了 WebView 的功能，使其不能加载本地的 html 文件**


**解决方案：**

对于不需要使用 file 协议的应用，禁用 file 协议；

    setAllowFileAccess(false); 

对于需要使用 file 协议的应用，禁止 file 协议加载 JavaScript。

    setAllowFileAccess(true); 

// 禁止 file 协议加载 JavaScript

    if (url.startsWith("file://") {
    setJavaScriptEnabled(false);
    } else {
    setJavaScriptEnabled(true);
    }


### 2. setAllowFileAccessFromFileURLs（） ###

    // 设置是否允许通过 file url 加载的 Js代码读取其他的本地文件
    webView.getSettings().setAllowFileAccessFromFileURLs(true);
    // 在Android 4.1前默认允许
    // 在Android 4.1后默认禁止


**解决方案：**
    
    设置setAllowFileAccessFromFileURLs(false);

当设置成为 false 时，上述JS的攻击代码执行会导致错误，**表示浏览器禁止从 file url 中的 javascript 读取其它本地文件**。



### 3. setAllowUniversalAccessFromFileURLs（） ###


    // 设置是否允许通过 file url 加载的 Javascript 可以访问其他的源(包括http、https等源)
    webView.getSettings().setAllowUniversalAccessFromFileURLs(true);
    
    // 在Android 4.1前默认允许（setAllowFileAccessFromFileURLs（）不起作用）
    // 在Android 4.1后默认禁止



**解决方案：**

```
设置setAllowUniversalAccessFromFileURLs(false);
```


## 如何避免WebView内存泄露？ ##

不在xml中定义 Webview ，而是在需要的时候在Activity中创建，并且Context使用 getApplicationgContext()

    LinearLayout.LayoutParams params = new LinearLayout.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT);
    mWebView = new WebView(getApplicationContext());
    mWebView.setLayoutParams(params);
    mLayout.addView(mWebView);

在 Activity 销毁（ WebView ）的时候，先让 WebView 加载null内容，然后移除 WebView，再销毁 WebView，最后置空。
    
    @Override
    protected void onDestroy() {
    if (mWebView != null) {
    mWebView.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
    mWebView.clearHistory();
    
    ((ViewGroup) mWebView.getParent()).removeView(mWebView);
    mWebView.destroy();
    mWebView = null;
    }
    super.onDestroy();
    }
    