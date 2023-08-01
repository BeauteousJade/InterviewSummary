# 1. 加载
>1. System.load(): 用于加载指定的本地代码库文件（通常是以 ".so" 为后缀的共享库文件）。需要指定本地代码库的绝对路径，如 System.load("/data/app/libnative.so")。
>2. System.loadlibary：用于加载预先安装在系统库路径中的本地代码库，只需要指定库的名称，不需要指定路径。比如说，APK内部打包的so包

# 2. 注册
>1. 静态注册：即在C++代码里面直接申明对应的方法，格式是Java_包名_类名_方法名。
>2. 动态注册：即在C++代码里面需要重写JNI_ONLOAD方法，然后调用`RegisterNatives`方法注册对应的方法。