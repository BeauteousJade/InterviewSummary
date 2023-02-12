# OkHttp

## 1. 请求
外部先是需要构建一个OkHttpClient对象，这个对象里面包含请求相关的很多配置，比如说，请求超时时间：readTime，connectTime，writeTime，默认都是10s等。请求方式分为两种，分别是同步请求和异步请求。

当我们构造完成请求的Request对象之后，需要通过OkHttpClient的newCall方法，拿到一个Request对象执行对象。newCall方法返回的都是RealCall对象。

同步请求就是调用Call的execute方法。在此方法的内部，是通过getResponseWithInterceptorChain方法进行网络请求的，真实的请求是依赖框架内部的拦截器链完成的。

异步请求就是调用Call的enqueue方法。方法内部有一个readyAsyncCall数组，首先会把Call放到这个数组里面，这个数组就是用来放置等待执行的Call。其次，就是尝试执行这个Call，会根据runningAsyncCalls数组来判断当前正在执行的请求数量，保证不超过阈值(64)，然后就是同一个主机的请求数也不超过阈值(5)。如果这俩条件都满足，就把Call对象从readyAsyncCall队列移到runningAsyncCalls，进而调用线程池执行。线程池的配置是核心线程为0，无上限的最大线程数，线程存活时间是60s。

## 2. 拦截器
默认有5个拦截器，分别是：

|类名|作用|
|---|---|
|RetryAndFollowUpInterceptor|主要负责网络请求的失败重连|
|BridgeInterceptor|主要是将用户的Request转换为能够真正进行网络请求的Request，负责添加一些相应头；其次就是将服务器返回的Response解析成为用户用够真正使用的Response，负责GZip解压之类的|
|CacheInterceptor|主要负责网络请求的Response缓存管理|
|ConnectInterceptor|主要负责打开与服务器的连接，正式进行网络请求|
|CallServerInterceptor|主要负责往网络数据流写入数据，同时接收服务器返回的数据|

加上自定义拦截器，执行顺序是：`自定义普通的拦截器`->`RetryAndFollowUpInterceptor`->`BridgeInterceptor `->`CacheInterceptor ` -> `ConnectInterceptor ` -> `自定义NetWork拦截器`-> `CallServerInterceptor`

> RetryAndFollowUpInterceptor：重试次数默认是20。
> 
> BridgeInterceptor：添加一下请求头，比如说Content-Type、Content-Length等。
> 
> CacheInterceptor：优先从缓存里面中去获取Response，如果缓存中没有，才去网络上获取，并且获取完成，会以Request的Url作为key，使用DiskLruCache将Response缓存起来。注意只有get方法才执行缓存。


## 3. ConnectionPool
负责所有的连接，包括连接的复用，以及无用连接的清理
