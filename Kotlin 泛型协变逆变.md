
&emsp;&emsp;在 Kotlin 中，泛型可以使用协变（covariant）和逆变（contravariant）来解决一些类型安全问题。

&emsp;&emsp;协变（covariant）指的是对于类型参数 `T`，如果 `A` 是 `B` 的子类型，那么 `List<A>` 就是 `List<B>` 的子类型。例如，对于类型参数 T，`List<T> `是协变的，这意味着如果 S 是 T 的子类型，则 `List<S>` 是 `List<T>` 的子类型。在 Kotlin 中，可以使用 `out` 关键字来标记泛型类型参数 `T`，表示该类型参数 `T` 可以被用在协变的位置，例如：
```
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // 协变，可以把 Source<String> 赋值给 Source<Any>
}
```

&emsp;&emsp;逆变（contravariant）指的是对于类型参数 `T`，如果 `A` 是 `B` 的子类型，那么 `Consumer<B>` 就是 `Consumer<A>` 的子类型。例如，对于类型参数 `T`，`Consumer<T>` 是逆变的，这意味着如果 `S` 是 `T` 的子类型，则 `Consumer<T>` 是 `Consumer<S>` 的子类型。在 Kotlin 中，可以使用 `in `关键字来标记泛型类型参数 `T`，表示该类型参数 `T` 可以被用在逆变的位置，例如：
```
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 是 Double，是 Number 的子类型
    val y: Comparable<Double> = x // 逆变，可以把 Comparable<Number> 赋值给 Comparable<Double>
}
```

&emsp;&emsp;需要注意的是，协变和逆变只能用在接口和类的类型参数上，不能用在函数的参数类型和返回类型上。在函数中，可以使用通配符类型来模拟协变和逆变的效果，例如：
```
fun <T> copy(from: List<out T>, to: MutableList<T>) {
    for (item in from) {
        to.add(item)
    }
}

fun <T> sort(list: MutableList<T>, comparator: Comparator<in T>) {
    // ...
}
```

在上面的代码中，`List<out T>` 表示 List 是协变的，`MutableList<T>` 表示 MutableList 是不变的，`Comparator<in T>` 表示 Comparator 是逆变的。

