# 文章连接
[深入理解Android类加载机制](https://juejin.cn/post/7143900207380430855)

# Class loader
>1. Java class load有：`Bootstrap ClassLoader`、`Extensions Classloader`和`Application Classloader`
>2. Android class load 有：`BootClassloader`、`PathClassloader`和DexClassLoader`

# 关系和作用
&emsp;&emsp;关系图如下：
![](./image/Android%20classLoader%20%E5%85%B3%E7%B3%BB%E5%9B%BE.webp)
>1. `BaseDexClassLoader`：实现应用层类文件的加载，真正的加载逻辑委托给PathList来完成。
>2. `PathClassLoader`：继承自BaseDexClassLoader，加载系统类和应用程序的类，通常用来加载已安装的apk的dex文件，实际上外部存储的dex文件也能加载。
>3. `DexClassLoader`：继承自BaseDexClassLoader，可以加载dex文件以及包含dex的压缩文件（apk，dex，jar，zip），不管加载哪种文件，最终都要加载dex文件。Android8.0之后和PathClassloader无异。
>4. `BootClassLoader`：Android系统启动时会使用BootClassLoader来预加载常用类，它继承自ClassLoader，是顶层的父加载器parent。

&emsp;&emsp;加载流程图：
![](./image/Android%20classloader%20%E5%8A%A0%E8%BD%BD%E6%B5%81%E7%A8%8B%E5%9B%BE.webp)