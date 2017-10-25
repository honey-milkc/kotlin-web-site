---
type: doc
layout: reference
category: "Syntax"
title: "인라인 함수와 구체화한(Reified) 타입 파라미터"
---

# 인라인 함수

[고차 함수](lambdas.html)를 사용하면 런타임에 일부 불이익이 발생한다. 각 함수는 객체이고 함수의 몸체에서 접근하는 변수인 클로저를 캡처한다.
메모리 할당(함수 객체와 클래스 둘에 대해)과 가상 호출로 런타임 오버헤드가 증가한다.   

하지만 많은 경우에 람다 식을 인라인해서 이런 종류의 오버헤드를 없앨 수 있다.
아래 함수가 이런 상황의 종은 예다. `lock()` 함수를 쉽게 호출 위치에 인라인할 수 있다.
다음 경우를 보자:

``` kotlin
lock(l) { foo() }
```

파라미터를 위한 함수 객체를 만들고 호출을 생성하는 대신에 컴파일러는 다음 코드를 만든다:

``` kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}
```

이것이 우리가 앞에서 원한 것 아닌가?

컴피얼러가 이렇게 하도록 만들려면, `lock()` 함수에 `inline` 수식어를 붙이면 된다:

``` kotlin
inline fun lock<T>(lock: Lock, body: () -> T): T {
    // ...
}
```

`inline` 수식어는 함수 자체와 함수에 전달되는 람다에 영향을 준다. 모두 호출 위치에 인라인 된다.

인라인은 생성한 코드의 크기를 증가시킨다. 하지만 함리적인 방법으로 처리하면(예, 큰 함수의 인라인을 피함),
성능에서, 특히 루프의 "변형" 호출 위치에서, 큰 성과를 낼 수 있다.

## 인라인 안하기

인라인 함수에 전달되는 람다 중 일부만 인라인되길 원할 경우,
함수 파라미터에 `noinline` 수식어를 붙일 수 있다:

``` kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // ...
}
```

인라인 가능한 람다는 인라인 함수 안에서만 호출할 수 있고 또는 인라인 가능한 인자로만 전달할 수 있지만,
`noinline`은 필드에 저장하거나 다른 곳에 전달하는 등 원하는 방식으로 다룰 수 있다.

만약 인라인 함수가 인라인 가능한 함수 파라미터를 갖고 있지 않고 [reified 타입 파라미터](#reified-type-parameters)를 갖지 않으면,
컴파일러는 경고를 발생한다. 왜냐면 그런 함수를 인라인 하는 것은
이득이 없기 때문이다(인라인이 확실히 필요할 경우 `@Suppress("NOTHING_TO_INLINE")` 애노테애션을 사용해서
경고를 감출 수 있다). 

## 비-로컬 리턴

코틀린에서 이름을 가진 함수나 익명 함수에서 나가려면 일반적인 한정하지 않은 `return`을 사용해야 한다.
이는 람다를 나가려면 [라벨](returns.html#return-at-labels)을 사용해야 한다는 것을 의미한다.
람다에서의 순수 `return`은 금지하는데 람다가 둘러싼 함수를 리턴할 수 없기 때문이다:

``` kotlin
fun foo() {
    ordinaryFunction {
        return // 에러: `foo`를 리턴할 수 없다
    }
}
```

만약 람다를 전달한 함수가 인라인되면, 리턴도 인라인 되기 때문에 다음이 가능하다:

``` kotlin
fun foo() {
    inlineFunction {
        return // OK: 람다도 인라인됨
    }
}
```

이런 리턴(람다에 위치하지만 둘러싼 함수에서 나가는)을 *비-로컬(non-local)* 리턴이라고 부른다.
종종 인라인 함수를 둘러싼 루프에서 이런 종류의 구성을 사용한다:

``` kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // hasZeros에서 리턴한다
    }
    return false
}
```

일부 인라임 함수는 파라미터로 그 함수에 전달한 람다를 
함수 몸체가 아닌 (로컬 객체나 중첩 함수 같은) 다른 실행 컨텍스트에서 호출할 수 있다.
그런 경우 람다에서 비-로컬 흐름 제어를 허용하지 않는다.
이를 명시하려면 람다 파라미터에 `crossinline` 수식어를 붙여야 한다:

``` kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ...
}
```


인라이된 람다에서의 `break`와 `continue`는 아직 사용할 수 없지만, 향후 지원할 계획이다.

## 구체화한(Reified) 타입 파라미터

종종 파라미터로 전달한 타입에 접근해야 할 때가 있다:

``` kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```

이 코드는 트리를 탐색하고 노드가 특정 타입인지 검사하기 위해 리플렉션을 사용한다.
잘 동작하지만, 호출 위치가 그다지 이쁘지 않다:

``` kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```

실제로 원하는 것은 아래 코드처럼 호출해서 함수에 타입을 전달하는 것이다:

``` kotlin
treeNode.findParentOfType<MyTreeNode>()
```

이를 가능하게 하기 위해 인라인 함수는 *구체화한(reified) 타입 파라미터*를 지원하며, 다음과 같은 코드를 작성할 수 있다:

``` kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```

`reified` 수식어로 타입 파라미터를 한정하면, 함수 안에서 마치 일반 클래스처럼 타입 파라미터에 접근할 수 있다.
함수를 인라인하기 때문에 리플렉션이 필요없고  
`!is`나 `as`와 같은 보통의 연산자가 동작한다.
또한 위에서 보여준 `myTree.findParentOfType<MyTreeNodeType>()`처럼 그것을 호출할 수 있다:

많은 경우에 리플렉션이 필요없지만, 구체화한(reified) 타입 파라미터에 대해 리플렉션을 사용할 수 있다:

``` kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```

일반 함수(inline을 붙이지 않은)는 구체화한(reified) 파라미터를 가질 수 없다.
런타임 표현을 갖지 않는 타입(예, 구체화한 타입 파라미터가 아니거나 `Nothing`과 같은 가상 타입)은
구체화한(reified) 타입 파라미터를 위한 인자로 사용할 수 없다.

저수준 설명은 [스펙 문서](https://github.com/JetBrains/kotlin/blob/master/spec-docs/reified-type-parameters.md)에서 볼 수 있다.

{:#inline-properties}

## 인라인 프로퍼티 (1.1부터)

지원 필드가 없는 프로퍼티 접근자에 `inline` 수식어를 사용할 수 있다.
개별 프로퍼티 접근자에 사용한다:

``` kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ...
    inline set(v) { ... }
```

또한 전체 프로퍼티에 붙여서 두 접근자를 모두 인라인으로 설정할 수 있다:

``` kotlin
inline var bar: Bar
    get() = ...
    set(v) { ... }
```

호출 위치에서 인라인 접근자는 일반 인라인 함수처럼 인라인된다.

{:#public-inline-restrictions}

## 공개 API 인라인 함수에 대한 제약사항

`public`이나 `protected`인 인라인 함수가 `private` 또는 `internal` 선언에 포함되어 있지 않으면
[모듈](visibility-modifiers.html#modules)의 공개 API로 간주한다.
다른 모듈에서 호출할 수 있으며 이 때에도 호출 위치에 인라인한다.

이는 인라인 함수를 선언한 모듈을 변경한 뒤에 호출하는 모듈을 재컴파일하지 않아서 발생하는 
바이너리 호환성 깨짐 위험을 유발할 수 있다.

이렇게 모듈의 **비**-공개 API를 변경해서 발생하는 비호환성 위험을 제거하기 위해
공개 API 인라인 함수가 비-공개 API 선언(`private`이나 `internal` 선언과 그 선언의 몸체에 있는 부분)을 사용하는 것을 허용하지 않는다.

`internal` 선언에 `@PublishedApi` 애노테이션을 붙이면 공개 API 인라인 함수에서 그것을 사용할 수 있다.
`internal` 인라인 함수에 `@PublishedApi`를 붙이면, 그것의 몸체 또한 public인 것처럼 검사한다.
 
