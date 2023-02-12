# 1. 特点
SparseArray内部是一个Key为int类型的map数组。

# 2. 实现
>1. 内部有两个数组，一个数组用来存储Key，一个用来存储Value。
>2. put 的时候，会通过二分搜索找到Key所对应的index，然后再两个数组对应index位置上存储对应的数据。扩容的时候是 * 2。
>3. remove 的时候并不是真正的remove，只是将对应位置上标记为DELETE状态。在put的时候，发现该位置是该标记，那么就直接覆盖。