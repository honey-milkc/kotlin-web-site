---
type: doc
layout: reference_ko
category: "Other"
title: "타입 별칭 (1.1부터)"
---

## 타입 별칭

타입 별칭은 타입을 위한 다른 이름을 제공한다.
타입 이름이 너무 길면 더 짧은 이름을 추가해서 그 이름을 대신 사용할 수 있다.

이는 긴 지네릭 타입 이름을 짧게 만들때 유용하다.
예를 들어, 콜렉션 타입을 축약할 때 사용한다.

``` kotlin
typealias NodeSet = Set<Network.Node>

typealias FileTable<K> = MutableMap<K, MutableList<File>>
```

함수 타입을 위한 다른 별칭을 제공할 수 있다:

``` kotlin
typealias MyHandler = (Int, String, Any) -> Unit

typealias Predicate<T> = (T) -> Boolean
```

내부와 중첩 클래스를 위한 새 이름을 가질 수 있다:

``` kotlin
class A {
    inner class Inner
}
class B {
    inner class Inner
}

typealias AInner = A.Inner
typealias BInner = B.Inner
```

타입 별칭이 새 타입을 추가하는 것은 아니다.
대응하는 기저 타입은 동일하다.
코드에 `typealias Predicate<T>`를 추가하고 `Predicate<Int>`를 사용하면 
코틀린 컴파일러는 이를 `(Int) -> Boolean`로 확장한다.
그래서 일반 함수 타입이 필요한 곳에 별칭 타입 변수를 전달할 수 있으며 반대도 가능하다:
 
``` kotlin
typealias Predicate<T> = (T) -> Boolean

fun foo(p: Predicate<Int>) = p(42)

fun main(args: Array<String>) {
    val f: (Int) -> Boolean = { it > 0 }
    println(foo(f)) // "true" 출력

    val p: Predicate<Int> = { it > 0 }
    println(listOf(1, -2).filter(p)) // "[1]" 출력
}
```
