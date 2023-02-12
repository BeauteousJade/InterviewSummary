# Acitivty启动
https://juejin.cn/post/6965194338154315812

## 1. 流程
Activity.startActivity -> Instrumentation -> ActivityTaskManagerService (系统进程)-> ActivityStarter（组装ActivityRecord，以及Activity其他的信息，并且将ActivityRecord添加到栈顶） -> RootWindowContainer.resumeFocusedStacksTopActivities（管理系统的窗口，恢复栈顶的Activity，检查Activity的可见性）-> ActivityStack.resumeTopActivityUncheckedLocked (恢复将要启动的Activity ) -> ActivityTaskSupervisor.startSpecificActivity(真正启动Activity--realStartActivityLocked，且跟App进程通信)


如果在startSpecificActivity阶段还没有启动进程，会调用ActivityTaskManagerService的startProcessAsync启动进程。这个方法里面最终调用到ActivityManagerServer的startProcessLocked方法，这个方法最终会调用到Process的start，然后就是跟Zygote进程通信(socket方法)，进而Zygote进程fork出应用进程。

如果启动了进程了，就会通过如下的方式回到APP进程里面，APP进程此时会SystemServer进程，启动Activity。


## 2. ClientLifecycleManager和ClientTransaction
在ActivityTaskSupervisor.realStartActivityLocked方法里面，会使用ClientLifecycleManager和ClientTransaction跟App进程进行通信。代码如下：

```
final ClientTransaction clientTransaction = ClientTransaction.obtain(proc.getThread(), r.appToken);
clientTransaction.addCallback(LaunchActivityItem.obtain(...));
clientTransaction.setLifecycleStateRequest(ResumeActivityItem.obtain(...));
mService.getLifecycleManager().scheduleTransaction(clientTransaction);
```
其中`mService `就是ActivityTaskManagerService，通过`scheduleTransaction `方法回到App进程。

而在App进程中，是通过ApplicationThread来接受从系统进程来的消息。ApplicationThread收到消息之后，是通过内部名为mH的Handler发送EXECUTE_TRANSACTION的消息


## 3. App进程
在App进程中，ApplicationThread收到系统进程发送过来的ClientTransaction对象之后，会给Handler发送EXECUTE_TRANSACTION消息。而在Handler内部，会把ClientTransaction对象交给TransactionExecutor处理。而在TransactionExecutor内部，就是创建Activity的对象，并且回调其生命周期。


![](https://img-blog.csdnimg.cn/20210523030842665.png)