# HashMap
https://tech.meituan.com/2016/06/24/java-hashmap.html

## 1. Hash冲突
解决hash冲突有两种方式，分别是链地址法和开发地址法。HashMap使用链地址法解决hash冲突，默认是链表，当链表长度大于8，会将链表转换为红黑树。

## 2. 长度规则
HashMap默认长度是16，且长度必须是2 的n 次方。设计的原因是原因有两个：
>1. 取模方便， & length - 1即可。
>2. 扩容方便。扩容之后，元素要么在原始位置不懂，要不在 oldIndex + oldLength 的位置。
>

## 3. Hash算法
hash算法：

```
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
根据hashCode，计算index:

```
(n - 1) & hash
```

## 线程不安全
由于HashMap同一个位置上是一个链表，当多线程put的时候，链表容易出现环，从而导致在get的时候，形成死循环。

