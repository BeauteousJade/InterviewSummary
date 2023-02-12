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
