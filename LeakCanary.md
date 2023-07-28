# LeakCanary

## 1.基本原理
弱引用对象会绑定一个队列，当弱引用对象内部存储的对象被垃圾回收器给回收的时候，会自动把其对象的弱引用放到这个队列里面。

而LeakCanary的原理就是基于此，在指定的时刻，在这个队列里面查找对应的弱引用实例，如果有，那么就表示当前实例没有被泄露；反之则泄露了。


## 2. MainProcessAppWatcherInstaller
在老版本的LeakCanary中，需要手动Application的onCreate调用install方法。而在新版本中，通过给AndroidManifest注册一个ContentProvider，然后在onCreate方法自动install。这个ContentProvider就是MainProcessAppWatcherInstaller。

install过程中加载4各种类型的监听器，分别是ActivityWatcher、FragmentAndViewModelWatcher、RootViewWatcher以及ServiceWatcher。

## 3. ActivityWatcher
用来监听Activity是否泄露。主要原理是通过给Application注册一个CallBack，用以监听每个Activity的生命周期。而当每个Activity destroy的时候，会开始触发内存泄露检测。

## 4. 泄露检测
泄露检测是在ObjectWatcher里面进行的，这个类里面有两个属性，分别是：
>1. watchedObjects:用来存储需要检测泄露的对象，key-value的形式存储。当每次开始检测之前，都会更新这个map，一是移除确定不泄露的实例，二是将待检测的实例放进去。
>2. queue：弱引用对象绑定的队列。如果一个对象会成功回收，那么对应的弱引用会放到这个队列。从而这个队列来判断是否泄露。
>

expectWeaklyReachable方法检测的入口，方法里面会创建对象对应的弱引用，并且绑定队列。

然后就是触发检测。检测最终会到HeapDumpTrigger的scheduleRetainedObjectCheck方法，这会把检测任务post 一下，保证垃圾回收能够足够时间执行。

最后，就是开始检测，检测之前首先会手动触发一下GC。然后获取当前没有被回收对象的数量，如果数量没超过阈值(默认是5)，任务直接结束；其次，就是看上一次检测时间跟本次的检测时间的间隔，没超过60s，也直接结束任务。最后就是dump内存，然后分析引用链，有泄漏的地方，对应抛出提示。

## 5. 内存分析
LeakCanary内部使用Shark来分析内存泄漏的引用链，主要分为三步：
>1. 分析 hprof 文件，获取镜像所有的 instance 实例
>2. 遍历所有的实例，判断这个实例与各个 Detectors 是否有存在泄漏，如果有，则记录 objectId 到集合
>3. 根据 objectId 集合获取各个泄漏实例引用链，分析出 gcRoot，并遍历 gcRoot 下的引用路径

这个地方重点在于如何找到泄漏的 objectId，因为找到 objectId，即可找到泄漏引用链。在分析 hprof 的时候我们可以拿到 dump 时的内存实例，那么，我们可以根据这个实例来判断是否泄漏，例如：
> Activity : 判断实例是否是 android.app.Activity 的子类，并且 mFinished 或 mDestroyed 是否为 true (Activity 关闭时该值会为 true)，因为 Activity 不泄露的话肯定是会被释放，所以，不可能存在于 dump 的实例中，有就是发生了泄漏

由于 dump hprof 会暂停所有 java 线程问题，致使 LeakCanary 只能应用于线下检测。但 Koom 和 Liko 另辟蹊径，采用 linux 的 copy-on-write 机制，从当前的主线程 fork 出一个子进程，然后在子进程进行 dump 分析，对于用户所在的进程不会有任何感知。 ​

这个地方会有个坑，就是在 fork 子进程的时候 dump hprof。由于 dump 前会先 suspend 所有的 java 线程，等所有线程都挂起来了，才会进行真正的 dump。由于 copy-on-write 机制，子进程也会将父进程中的 threadList 也拷贝过来，但由于 threadList 中的 java 线程活动在父进程，子进程是无法挂起父进程中的线程的，然后就会一直处于等待中。 ​

为了解决这个问题，Koom 和 Liko 采用欺骗的方式，在 fork 子进程之前，先将父进程中的 threadList 全部设置为 suspend 状态，然后 fork 子进程，子进程在 dump 的时候发现 threadList 都为挂起状态了，就立马开始 dump hprof，然后父进程在 fork 操作之后，立马 resume 恢复回 threadList 的状态 ​


## 6. 新应用的实现方案
[冷知识 —— 如何实现 LeakCanary 桌面多出一个“新应用”的效果](https://juejin.cn/post/6844904100031643661)

`<activity-alias>`，顾名思义，Activity 的别名，是在应用清单文件 AndroidManifest.xml 中申明，`在 <application>` 标签之下。

语法大致如下：
```
<activity-alias android:enabled=["true" | "false"]
                android:exported=["true" | "false"]
                android:icon="drawable resource"
                android:label="string resource"
                android:name="string"
                android:permission="string"
                android:targetActivity="string" >
                . . .
</activity-alias>
```
`<activity-alias>` 会包含一个目标 Activity，具有自己的一组 Intent 过滤器，Intent 过滤器可以指定 android.intent.action.MAIN 和 android.intent.category.LAUNCHER 标志，这样就会以一个新的图标入口出现在手机的桌面启动器当中。

示例如下：
```
<application
    . . .>
    
    . . .
    
    <activity
        android:name=".SecondActivity"/>
    
    <activity-alias
        android:name="com.jason.demo.dynamicshortcut.ShortcutLauncherActivity"
        android:enabled="true"
        android:icon="@mipmap/ic_launcher_alias"
        android:label="ActivityAlias"
        android:targetActivity=".SecondActivity">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity-alias>
    
</application>
```