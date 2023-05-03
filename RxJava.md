# RxJava

## 1. Observable
在create 操作符创建Observable对象的时候，会传入一个ObservableOnSubscribe对象，该对象用户发射元素的功能。当我们使用subscribe操作符注册一个观察者的时候，会触发调用Observable的subscribeActual方法，从而让上游发送事件。--冷流

## 2. Flowable
调用原理基本跟Observable是一致的，内部有5中被压处理策略：
>1. BufferAsyncEmitter:默认策略，有一个128大小的缓冲池，用来存储数据。
>2. MissingEmitter：丢失策略，默认是发送最新的数据，不处理被压问题。
>3. ErrorAsyncEmitter：报错策略，缓存的数量超过了指定数量，就报错。
>4. DropAsyncEmitter：抛弃策略，当超出指定数量，就不管了。
>5. LatestAsyncEmitter：消费最新数据的策略。内部有一个reference，用来记录当前最新数据，消费者消费的也是这个reference里面记录的数据。
>

## 3. 线程调度
> subscribeOn是上游的执行线程，observeOn是下游的执行线程。onSubscribe处于subscribeOn之前的线程。subscribeOn是第一次生效，因为subscribeOn是不断的包裹，所以最里面的那一层是生效的。observeOn每次调用都生效，调用之后，后续的监听都在指定的线程执行。
> 
> subscribeOn内部会把发送事件动作封装一个Runnable，然后调用Scheduler的scheduleDirect方法，去执行这个任务。在Scheduler内部会创建一个Worker对象，由这个Worker来执行这个问题。
> 
> observeOn内部会获取Scheduler的Worker的对象，然后使用这个Worker对象分发事件的任务。



 