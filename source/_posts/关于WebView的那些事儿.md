title: 关于WebView的那些事儿
date: 2016-02-17 09:43:49
tags: WebView
---


### As we know ,Android中的WebView控件令人蛋疼的东西(相比IOS的UIWebView/WKWebView，在多方面都令人感到力不从心。)

<!--more-->

First of all, 献上一篇较为完整的WebView基础使用教程：<br>
<http://tutorials.jenkov.com/android/android-web-apps-using-android-webview.html#loaing-a-url-into-the-webview><br>

### 基础篇
* 1.基本配置WebSettings

```
WebSettings webSettings = mWebView .getSettings();

webview.requestFocusFromTouch();//支持获取手势焦点，输入用户名、密码或其他

setJavaScriptEnabled(true);  //支持js
setPluginsEnabled(true);  //支持插件 

//设置自适应屏幕，两者合用
setUseWideViewPort(true);  //将图片调整到适合webview的大小 
setLoadWithOverviewMode(true); // 缩放至屏幕的大小

setSupportZoom(true);  //支持缩放，默认为true。是下面那个的前提。
setBuiltInZoomControls(true); //设置内置的缩放控件。  若上面是false，则该WebView不可缩放，这个不管设置什么都不能缩放。

setDisplayZoomControls(false); //隐藏原生的缩放控件

setJavaScriptCanOpenWindowsAutomatically(true); //支持通过JS打开新窗口 
setLayoutAlgorithm(LayoutAlgorithm.SINGLE_COLUMN); //支持内容重新布局  
supportMultipleWindows();  //多窗口 
setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);  //关闭webview中缓存 
setAllowFileAccess(true);  //设置可以访问文件 
setNeedInitialFocus(true); //当webview调用requestFocus时为webview设置节点
setLoadsImagesAutomatically(true);  //支持自动加载图片
setDefaultTextEncodingName("utf-8");//设置编码格式

```
* 2.加载URL/HTML代码块

```
//加载Url
webView.loadUrl("http://google.com/"); 

//加载HTML代码块
String data = "<html><body><h1>HTML Loaded Directly</h1></body></html>";
//webView.loadData(data, "text/html; charset=UTF-8", null);
webView.loadData(data, "text/html", "UTF-16");

```
* 3.WebViewClient

设置setWebChromeClient后，页面能够在当前App中打开，而不是调用系统的浏览器。


```
webView.setWebChromeClient(new MyWebChromeClient());

```
* 4.WebViewClient方法简述：

```
private class MyWebViewClient extends WebViewClient {

        @Override//对网页中超链接按钮的响应,当按下某个连接时WebViewClient会调用这个方法，并传递参数：按下的url
        public boolean shouldOverrideUrlLoading(WebView view, String url) 

        @Override//开始载入页面调用，一般用于设置loading 动画
        public void onPageStarted(WebView view, String url, Bitmap favicon) 

        //页面加载在完成,一般用于关闭loading 动画
        public void onPageFinished(WebView view, String url)

        //加载页面资源时调用，每一个资源（比如图片）的加载都会调用一次。
        public void onLoadResource(WebView view, String url)

        //监听所有的Url请求,用于拦截替换
        public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) 

        //报告错误信息 (文件找不到/网络连不上/服务器找不到等问题)
        public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) 

        //应用程序重新请求网页数据
        public void onFormResubmission(WebView view, Message dontResend, Message resend) 

        //更新历史记录
        public void doUpdateVisitedHistory(WebView view, String url, boolean isReload) 

        //组件加载网页发生证书认证错误时调用,重写此方法可以处理https请求。
        public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) 

        //SSL客户端证书请求回调
        public void onReceivedClientCertRequest(WebView view, ClientCertRequest request) 

        //获取返回信息授权请求
        public void onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host, String realm) 

        //重写此方法才能够处理在浏览器中的按键事件
        public boolean shouldOverrideKeyEvent(WebView view, KeyEvent event) 

        //Key事件未被处理的回调
        public void onUnhandledInputEvent(WebView view, InputEvent event) 
        
        public void onScaleChanged(WebView view, float oldScale, float newScale) 

        //通知应用程序有个自动登录的帐号过程
        public void onReceivedLoginRequest(WebView view, String realm, String account, String args) 
    }

```

* 5.共享App的Cookie到WebView中去

```
CookieSyncManager.createInstance(context);
CookieManager cookieManager = CookieManager.getInstance();
cookieManager.setAcceptCookie(true);
cookieManager.setCookie(url, cookies);//cookies是在HttpClient中获得的cookie
CookieSyncManager.getInstance().sync();

```
Cookie头由客户端发送，包含在HTTP请求的头部中。**注意，只有cookie的domain和path与请求的URL匹配才会发送这个cookie。**


### 中级篇
### 1.关于主动获取点击事件
处理WebView的点击响应，最直接方便的方式就是js交互了。*但这里我们聊聊其他的方式* 😂。

##### ①首先是**（单击）**WebView中图片：

```
  webView.setOnTouchListener(new View.OnTouchListener() {
            public boolean onTouch(View v, MotionEvent event) {
                float x = 0, y = 0;
                switch (event.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        x = (int) event.getRawX();
                        y = (int) event.getRawY();
                        break;

                    case MotionEvent.ACTION_UP:
                        if (x - event.getX() < 5 && y - event.getY() < 5) {
                            WebView.HitTestResult hitTestResult = ((WebView) v)
                                    .getHitTestResult();
                            if (null == hitTestResult) {
                                break;
                            }
                            if (hitTestResult.getType() == WebView.HitTestResult.IMAGE_TYPE
                                    || hitTestResult.getType() == WebView.HitTestResult.IMAGE_ANCHOR_TYPE
                                    || hitTestResult.getType() == WebView.HitT
                                    estResult.SRC_IMAGE_ANCHOR_TYPE) {

                                    //TODO 模拟点击图片回调操作
                                    //hitTestResult.getExtra() 为图片的链接 
                            }
                        }
                        break;
                }
                return false;
            }
        });
```

这里通过Touch监听坐标位置和WebView.HitTestResult来共同判断，用户对图片进行了单击。

##### ② **长按** WebView中的图片
根据上面的单击处理方式，相信大家也能高仿出一个long click 事件来，所以我们继续换一种方式 😫（容我再做一个悲伤的表情先）。<br> 
	*registerForContextMenu(mWebView)*的方式:<br>
	
* 1.首先WebView先进行上下文的注册：
	
	```
	registerForContextMenu(webView);
	
	```
* 2.重写 onCreateContextMenu() 方法：
	
	```
	 @Override
    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
        super.onCreateContextMenu(menu, v, menuInfo);
        final WebView.HitTestResult result = ((WebView) v).getHitTestResult();

        MenuItem.OnMenuItemClickListener handler = new MenuItem.OnMenuItemClickListener() {
            public boolean onMenuItemClick(MenuItem item) {
                // todo menu 点击事件
                /**
                switch (item.getItemId()) {
                    case 0:
                        //todo 保存图片
                        break;
                    case 1:
                        //todo 看大图
                        break;
                }
                */
                return true;
            }
        };

        if (result.getType() == WebView.HitTestResult.IMAGE_TYPE ||
                result.getType() == WebView.HitTestResult.SRC_IMAGE_ANCHOR_TYPE) {
            //menu.add(0, 0, Menu.NONE, "保存图片").setOnMenuItemClickListener(handler);
            //menu.add(0, 1, Menu.NONE, "看大图").setOnMenuItemClickListener(handler);
        }
    }
    
	```
	3.反注册：<br>
	
	```
	页面销毁处，解除注册
	unregisterForContextMenu(webView);
	
	```




### 2.关于WebView的视频播放
### 3.关于WebView的缓存
### 4.关于WebView的背景透明
Android4.0环境下WebView背景默认为白色。要设置成透明按如下步骤：<br>
1.在AndroidManifest.xml 文件里application设置

	android:hardwareAccelerated="false"
	
2.loadUrl/loadData后 

	webView.setBackgroundColor(0)
3.确认WebView的父层布局，背景也为透明。
	
### 4.关于WebView的扩展

* Chrome Custom Tabs<br>
<https://developer.chrome.com/multidevice/android/customtabs>


