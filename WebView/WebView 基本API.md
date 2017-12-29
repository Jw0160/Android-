# WebView 基本API

---

[本文转载，原文地址](http://reezy.me/p/20170515/android-webview/)
## WebView [Api](https://developer.android.com/reference/android/webkit/WebView.html)

> 基本


    // 获取当前页面的URL
    public String getUrl();
    // 获取当前页面的原始URL(重定向后可能当前url不同)
    // 就是http headers的Referer参数，loadUrl时为null
    public String getOriginalUrl();
    // 获取当前页面的标题
    public String getTitle();
    // 获取当前页面的favicon
    public Bitmap getFavicon();
    // 获取当前页面的加载进度
    public int getProgress();
    // 通知WebView内核网络状态
    // 用于设置JS属性`window.navigator.isOnline`和产生HTML5事件`online/offline`
    public void setNetworkAvailable(boolean networkUp)
    // 设置初始缩放比例
    public void setInitialScale(int scaleInPercent)；


> 加载网页

    // 加载URL指定的网页
    public void loadUrl(String url);
    // 携带http headers加载URL指定的网页
    public void loadUrl(String url, Map<String, String> additionalHttpHeaders);
    // 使用POST请求加载指定的网页
    public void postUrl(String url, byte[] postData);
    // 重新加载当前网页
    public void reload();
    // 加载内容
    public void loadData(String data, String mimeType, String encoding);
    // 使用baseUrl加载内容
    public void loadDataWithBaseURL(String baseUrl, String data, String mimeType, String encoding, String historyUrl);


> Javascript


    // 注入Javascript对象
    public void addJavascriptInterface(Object object, String name);
    // 移除已注入的Javascript对象，下次加载或刷新页面时生效
    public void removeJavascriptInterface(String name);
    // 对传入的JS表达式求值，通过resultCallback返回结果
    // 此函数添加于API19，必须在UI线程中调用，回调也将在UI线程
    public void evaluateJavascript(String script, ValueCallback<String> resultCallback)


> 导航(前进后退)
    
    // 复制一份BackForwardList
    public WebBackForwardList copyBackForwardList();
    // 是否可后退
    public boolean canGoBack();
    // 是否可前进
    public boolean canGoForward();
    // 是否可前进/后退steps页，大于0表示前进小于0表示后退
    public boolean canGoBackOrForward(int steps);
    // 后退一页
    public void goBack();
    // 前进一页
    public void goForward();
    // 前进/后退steps页，大于0表示前进小于0表示后退
    public void goBackOrForward(int steps);
    // 清除当前webview访问的历史记录
    public void clearHistory();

> 网页查找功能

    // 设置网页查找结果回调
    public void setFindListener(FindListener listener);
    // 异步执行查找网页内包含的字符串并设置高亮，查找结果会回调.
    public void findAllAsync (String find);
    // 查找下一个匹配的字符串
    public void findNext (boolean forward);
    // 清除网页查找的高亮匹配字符串
    public void clearMatches();


> 截屏/翻页/缩放
    
    // 保存网页(.html)到指定文件
    public void saveWebArchive(String filename);
    // 保存网页(.html)到文件
    public void saveWebArchive(String basename, boolean autoname, ValueCallback<String> callback)；
    // 上翻一页，即向上滚动WebView高度的一半
    public void pageUp(boolean top);
    // 下翻一页，即向下滚动WebView高度的一半
    public void pageDown(boolean bottom);
    // 缩放
    public void zoomBy(float factor);
    // 放大
    public boolean zoomIn();
    // 缩放
    public boolean zoomOut();


> 其它

    // 清除网页缓存，由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序
    public void clearCache(boolean includeDiskFiles);
    // 清除自动完成填充的表单数据
    public void clearFormData();
    // 清除SSL偏好
    public void clearSslPreferences();
    // 查询文档中是否有图片，查询结果将被发送到msg.getTarget()
    // 如果包含图片，msg.arg1 为1，否则为0
    public void documentHasImages(Message msg);
    // 请求最近轻叩(tapped)的 锚点/图像 元素的URL，查询结果将被发送到msg.getTarget()
    // msg.getData()中的url是锚点的href属性，title是锚点的文本，src是图像的src
    public void requestFocusNodeHref(Message msg);
    // 请求最近触摸(touched)的 图像元素的URL，查询结果将被发送到msg.getTarget()
    // msg.getData()中的url是图像链接
    public void requestImageRef(Message msg) 
    // 清除证书请求偏好，添加于API21
    // 在WebView收到`android.security.STORAGE_CHANGED` Intent时会自动清除
    public static void clearClientCertPreferences(Runnable onCleared)
    // 开启网页内容(js,css,html...)调试模式，添加于API19
    public static void setWebContentsDebuggingEnabled(boolean enabled)










