# Retrofit

## 1.基本概念
Retrofit内部分为4层，执行过程是：定义网络请求接口、解析接口生成合法的Request、由OkHttpCall来执行Request，然后得到Response、解析Response传到前端。

实现上，由Retrofit动态代理，将每个请求接口生成一个ServiceMethod对象，然后通过调用ServiceMethod的invoke方法拿到返回类型，由返回类型自己触发网络的请求。其中RxJava的返回类型是CallEnqueueObservable和CallExecuteObservable，这两个Observable在subscribeActual里面会使用OkHttpCall来进行网络请求。

## 2. ServiceMethod
ServiceMethod是由请求接口的方法生成的，它根据方法自行定义的格式生成，比如说方法返回类型、方法注解、方法类型等，内部内部有一个RequestFactory来存储这个请求相关的信息，最后会通过create方法来创建一个Request。

## 3. CallAdapter--网络执行适配器
在初始化ServiceMethod的过程中，还有一件事就是根据定义的方法，从注册的CallAdapterFactory去找到合法的CallAdapter。这个CallAdaper就是用来返回接口方法的返回类型对象的。RxJava对应的是RxJava2CallAdapterFactory。


## 4. Converter--数据转换器
当外部拿到网络执行适配器时进行网络请求的时候，拿到的Response是是有一个OkHttp封装好的，不利于直接使用。此时OKHttpCall中，会在拿到Response之后，使用Converter的convert方法进行转换。

Converter也是在ServiceMethod初始化过程中，根据返回类型来创建的。

