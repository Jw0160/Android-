# WebViewClient
---
  [参考](https://www.jianshu.com/p/3c94ae673e2a "详细应用")

[本文转载，原文地址](http://reezy.me/p/20170515/android-webview/)

	// 拦截页面加载，返回true表示宿主app拦截并处理了该url，否则返回false由当前WebView处理
    // 此方法在API24被废弃，不处理POST请求
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
    return false;
    }

    // 拦截页面加载，返回true表示宿主app拦截并处理了该url，否则返回false由当前WebView处理
    // 此方法添加于API24，不处理POST请求，可拦截处理子frame的非http请求
    @TargetApi(Build.VERSION_CODES.N)
    public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
    return shouldOverrideUrlLoading(view, request.getUrl().toString());
    }

    // 此方法废弃于API21，调用于非UI线程
    // 拦截资源请求并返回响应数据，返回null时WebView将继续加载资源
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
    return null;
    }

    // 此方法添加于API21，调用于非UI线程
    // 拦截资源请求并返回数据，返回null时WebView将继续加载资源
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
    return shouldInterceptRequest(view, request.getUrl().toString());
    }

    // 页面(url)开始加载
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
    }

    // 页面(url)完成加载
    public void onPageFinished(WebView view, String url) {
    }

    // 将要加载资源(url)
    public void onLoadResource(WebView view, String url) {
    }

    // 这个回调添加于API23，仅用于主框架的导航
    // 通知应用导航到之前页面时，其遗留的WebView内容将不再被绘制。
    // 这个回调可以用来决定哪些WebView可见内容能被安全地回收，以确保不显示陈旧的内容
    // 它最早被调用，以此保证WebView.onDraw不会绘制任何之前页面的内容，随后绘制背景色或需要加载的新内容。
    // 当HTTP响应body已经开始加载并体现在DOM上将在随后的绘制中可见时，这个方法会被调用。
    // 这个回调发生在文档加载的早期，因此它的资源(css,和图像)可能不可用。
    // 如果需要更细粒度的视图更新，查看 postVisualStateCallback(long, WebView.VisualStateCallback).
    // 请注意这上边的所有条件也支持 postVisualStateCallback(long ,WebView.VisualStateCallback)
    public void onPageCommitVisible(WebView view, String url) {
    }

    // 此方法废弃于API23
    // 主框架加载资源时出错
    public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
    }

    // 此方法添加于API23
    // 加载资源时出错，通常意味着连接不到服务器
    // 由于所有资源加载错误都会调用此方法，所以此方法应尽量逻辑简单
	//作用：加载页面的服务器出现错误时（如404）调用。	
    @TargetApi(Build.VERSION_CODES.M)
    public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
    if (request.isForMainFrame()) {
    onReceivedError(view, error.getErrorCode(), error.getDescription().toString(), request.getUrl().toString());
    }
    }

    // 此方法添加于API23
    // 在加载资源(iframe,image,js,css,ajax...)时收到了 HTTP 错误(状态码>=400)
    public void onReceivedHttpError(WebView view, WebResourceRequest request, WebResourceResponse errorResponse) {
    }

    // 是否重新提交表单，默认不重发
    public void onFormResubmission(WebView view, Message dontResend, Message resend) {
    dontResend.sendToTarget();
    }

    // 通知应用可以将当前的url存储在数据库中，意味着当前的访问url已经生效并被记录在内核当中。
    // 此方法在网页加载过程中只会被调用一次，网页前进后退并不会回调这个函数。
    public void doUpdateVisitedHistory(WebView view, String url, boolean isReload) {
    }

    // 加载资源时发生了一个SSL错误，应用必需响应(继续请求或取消请求)
    // 处理决策可能被缓存用于后续的请求，默认行为是取消请求
	//webView默认是不处理https请求的，页面显示空白，需要进行如下设置：
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
      handler.proceed();    //表示等待证书响应
        // handler.cancel();      //表示挂起连接，为默认方式
        // handler.handleMessage(null);    //可做其他处理
    }

    // 此方法添加于API21，在UI线程被调用
    // 处理SSL客户端证书请求，必要的话可显示一个UI来提供KEY。
    // 有三种响应方式：proceed()/cancel()/ignore()，默认行为是取消请求
    // 如果调用proceed()或cancel()，Webview 将在内存中保存响应结果且对相同的"host:port"不会再次调用 onReceivedClientCertRequest
    // 多数情况下，可通过KeyChain.choosePrivateKeyAlias启动一个Activity供用户选择合适的私钥
    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    public void onReceivedClientCertRequest(WebView view, ClientCertRequest request) {
    request.cancel();
    }

    // 处理HTTP认证请求，默认行为是取消请求
    public void onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host, String realm) {
    handler.cancel();
    }

    // 通知应用有个已授权账号自动登陆了
    public void onReceivedLoginRequest(WebView view, String realm, String account, String args) {
    }

    // 给应用一个机会处理按键事件
    // 如果返回true，WebView不处理该事件，否则WebView会一直处理，默认返回false
    public boolean shouldOverrideKeyEvent(WebView view, KeyEvent event) {
    return false;
    }

    // 处理未被WebView消费的按键事件
    // WebView总是消费按键事件，除非是系统按键或shouldOverrideKeyEvent返回true
    // 此方法在按键事件分派时被异步调用
    public void onUnhandledKeyEvent(WebView view, KeyEvent event) {
    super.onUnhandledKeyEvent(view, event);
    }

    // 通知应用页面缩放系数变化
    public void onScaleChanged(WebView view, float oldScale, float newScale) {
    }



