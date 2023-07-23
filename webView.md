# 1. 文章
[WebView内存泄漏](https://blog.csdn.net/qq_27248989/article/details/129528176)

# 2. 内存泄漏的原因
webView引起的内存泄漏主要是因为`org.chromium.android_webview.AwContents`类中注册了component callbacks，但是未正常反注册而导致的。 
`org.chromium.android_webview.AwContents`类中有这两个方法 `onAttachedToWindow` 和`onDetachedFromWindow`；系统会在attach和detach处进行注册和反注册component callback；
```
if (isDestroyed()) {
    return;
}
```
如果 isDestroyed() 返回 true 的话，那么后续的逻辑就不能正常走到，所以就不会执行unregister的操作；我们的activity退出的时候，都会主动调用 WebView.destroy() 方法，这会导致 isDestroyed() 返回 true；destroy()的执行时间又在onDetachedFromWindow之前，所以就会导致不能正常进行unregister()。

# 3. 解决办法
### (1). 在Activity里面动态创建
因为在xml里面定义webView,webView 的context可能是Activity.所以,可以换种思路,就是在Activity中动态创建,然后Context传递为ApplicationContext.

### (2). 正确销毁
让onDetachedFromWindow先走，在主动调用destroy()之前，把webview从它的parent上面移除掉。
```
ViewParent parent= mWebView.getParent();
if (parent!=null) {
    ((ViewGroup)parent).removeView(mWebView);
}
mWebView.destroy();
mWebView.stopLoading();// 退出时调用此方法，移除绑定的服务，否则某些特定系统会报错
mWebView.getSettings().setJavaScriptEnabled(false);
mWebView.clearHistory();
mWebView.clearView();
mWebView.removeAllViews();
mWebView.destroy();
super.onDestroy();
```
