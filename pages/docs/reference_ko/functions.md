---
type: doc
layout: reference
category: "Syntax"
title: "함수: infix, vararg, tailrec"
---

# 함수

## 함수 선언

코틀린에서 함수는 *fun*{: .keyword } 키워드를 사용해서 선언한다:

``` kotlin
fun double(x: Int): Int {
    return 2*x
}
```

## 함수 사용

전통적은 방식으로 함수를 호출한다:

``` kotlin
val result = double(2)
```

멤버 함수 호출은 점 부호를 사용한다:

``` kotlin
Sample().foo() // Sample 클래스의 인스턴스를 생성하고 foo를 호출
```

### 파라미터

파스칼 표기법(*name*: *type*)을 사용해서 파라미터를 정의한다.
각 파라미터는 콤마로 구분하며 타입을 가져야 한다:

``` kotlin
fun powerOf(number: Int, exponent: Int) {
...
}
```

### 기본 인자

함수 파라미터는 기본 값을 가질 수 있다. 이 경우 대응하는 인자를 생략할 때 기본 값을 사용한다.
이는 다른 언어와 비교해 오버로드의 개수를 줄여준다.

``` kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) {
...
}
```

타입 뒤에 **=**와 값을 사용해서 기본 값을 정의한다.

오버라이딩 메서드는 항상 베이스 메서드와 같은 기본 파라미터 값을 사용한다.
기본 파라미터 값을 갖는 메서드를 오버라이딩 할때, 시그너처에서 기본 파라미터를 값을 생략해야 한다:

``` kotlin
open class A {
    open fun foo(i: Int = 10) { ... }
}

class B : A() {
    override fun foo(i: Int) { ... }  // 기본 값을 허용하지 않음
}
```

기본 파라미터가 기본 값이 없는 파라미터보다 앞에 위치하면,
[이름 가진 인자](#named-arguments)로 함수를 호출할 때에만 기본 값을 사용한다:

``` kotlin
fun foo(bar: Int = 0, baz: Int) { /* ... */ }

foo(baz = 1) // 기본 값 bar = 0을 사용한다
```

하지만 함수 호출시 괄호 밖으로 마지막 인자를 [람다](lambdas.html#lambda-expression-syntax)로 전달하면,
기본 파라미터에 값을 전달하지 않는 것을 허용한다:

``` kotlin
fun foo(bar: Int = 0, baz: Int = 1, qux: () -> Unit) { /* ... */ }

foo(1) { println("hello") } // 기본 값 baz = 1을 사용 
foo { println("hello") }    // 기본 값 bar = 0와 baz = 1를 사용
```

### 이름 가진 인자(Named Argument)

함수를 호출할 때 함수 파라미터에 이름을 줄 수 있다.
이름을 가진 인자를 사용하면 함수에 파라미터 개수가 많거나 기본 값이 많을 때 편리하다. 


다음과 같은 함수가 있다고 하자:

``` kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
...
}
```

기본 인자를 사용해서 다음과 같이 호출할 수 있다:

``` kotlin
reformat(str)
```

하지만 기본 인자가 없다면 다음과 같이 호출해야 한다:

``` kotlin
reformat(str, true, true, false, '_')
```

이름 가진 인자를 사용하면 다음과 같이 코드의 가독성을 높일 수 있다:

``` kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
)
```

그리고 모든 인자가 필요한 것이 아니면 다음과 같이 작성할 수 있다:

``` kotlin
reformat(str, wordSeparator = '_')
```

위치 기반 인자와 이름 가진 인자를 함께 사용해서 함수를 호출할 때,
모든 위치 기반 인자는 첫 번째 이름 가진 인자보다 앞에 위치해야 한다.
예를 들어, `f(1, y = 2)`는 허용하지만 `f(x = 1, 2)`는 허용하지 않는다.

**spread** 연산자를 사용하면 이름 붙인 형식에
[인자의 변수 이름(*vararg*{: .keyword })](#variable-number-of-arguments-varargs)을 전달할 수 있다: 

``` kotlin
fun foo(vararg strings: String) { /* ... */ }

foo(strings = *arrayOf("a", "b", "c"))
foo(strings = "a") // 단일 값에는 필요 없다
```

자바 함수를 호출할 때에는 이름 가진 인자 구문을 사용할 수 없다. 왜냐면 자바 바이트코드가 함수 파라미터의 이름을
항상 유지하는 것은 아니기 때문이다.

### Unit 리턴 함수

함수가 어떤 유용한 값도 리턴하지 않으면, 리턴 타입으로 `Unit`을 사용한다.
`Unit`은 `Unit`을 유일한 값으로 갖는 타입니다. 이 값을 명시적으로 리턴하면 안 된다:

``` kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` 또는  `return` 생략
}
```

리턴 타입으로 `Unit`을 선언하는 것 또한 생략 가능하다. 위 코드는 다음과 동일하다:

``` kotlin
fun printHello(name: String?) {
    ...
}
```

### 단일 식 함수

함수가 단일 식을 리턴할 때 중괄호를 생략하고 **=** 뒤에 몸체로 단일 식을 지정할 수 있다:

``` kotlin
fun double(x: Int): Int = x * 2
```

컴파일러가 리턴 타입을 유추할 수 있으면 리턴 타입을 명시적으로 선언하는 것을 [생략](#explicit-return-types)할 수 있다.

``` kotlin
fun double(x: Int) = x * 2
```

### 명시적 리턴 타입

블록 몸체를 갖는 함수는 `Unit`을 리턴한다는 것을 의도하지 않는 이상 항상 리턴 타입을 명시적으로 지정해야 한다.
[Unit을 리턴하는 경우 생략 가능하다](#unit-returning-functions).
코틀린은 블록 몸체를 가진 함수의 리턴 타입을 유추할 수 없다.
왜냐면 그런 함수의  몸체에 복잡한 제어 흐름을 가지며 리턴 타입을 독자가 (그리고 때때로 컴파일러 조차도) 알 수 없기 때문이다.


### 가변 인자 (Varargs)

함수 파라미터에 (보통 마지막 파라미터) `vararg` 수식어를 붙일 수 있다:

``` kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

이는 함수에 가변 개수의 인자를 전달할 수 있도록 한다.

``` kotlin
val list = asList(1, 2, 3)
```

`T` 타입의 `vararg` 파라미터는 함수에서 `T` 배열로 접근한다. 예를 들어, 위 코드에서 `ts` 변수는 `Array<out T>` 타입이다.
가변 파라미터 뒤의 파라미터 값을 이름 가진 인자 구문으로 전달할 수 있다.
또는 가변 파라미터 뒤의 파라미터가 함수 타입이면 람다를 괄호 밖으로 전달할 수 있다.

`vararg` 함수를 호출할 때 인자를 `asList(1, 2, 3)`와 같이 한 개 씩 전달할 수 있다.
또는 이미 배열이 있다면 **펼침(spread)** 연산자(배열 앞에 `*` 사용)를 사용해서 배열을 함수의 `vararg` 인자로 전달할 수 있다:

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

### 중위 부호

다음의 경우에 중위 부호를 사용해서 함수를 호출할 수 있다.

* 멤버 함수이거나 [확장 함수](extensions.html)인 경우
* 파라미터가 한 개인 경우
* `infix` 키워드를 붙인 경우

``` kotlin
// Int에 대한 확장 함수 정의
infix fun Int.shl(x: Int): Int {
...
}

// infix 부호를 사용한 확장 함수를 호출

1 shl 2

// 이는 다음과 같음

1.shl(2)
```

## 함수 범위

코틀린은 함수를 파일에서 최상위 수준으로 선언할 수 있다. 이는 자바, C#, 스칼라와 달리 함수를 포함할 클래스를 만들 필요가 없다는 것을 의미한다.
최상위 수준 함수뿐만 아니라 함수를 로컬로, 멤버 함수로, 확장 함수로 선언할 수 있다.

### 로컬 함수

코틀린은 다른 함수 안에 선언한 함수 같은 로컬 함수를 지원한다.

``` kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

로컬 함수는 외부 함수의 로컬 변수에 접근할 수 있으며(클로저),
위 경우에 *visited*를 로컬 변수로 할 수 있다: 

``` kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```

### 멤버 함수

멤버 함수는 클래스나 오브젝트에 선언된 함수이다:

``` kotlin
class Sample() {
    fun foo() { print("Foo") }
}
```

점 부호를 사용해서 멤버 함수를 호출한다:

``` kotlin
Sample().foo() // Sample 클래스의 인스턴스를 생성하고 foo를 호출
```

클래스와 멤버 오버라이딩에 대한 정보는 
[클래스](classes.html)와 [상속](classes.html#inheritance)을 참고한다.

## 지네릭 함수

함수는 지네릭 파라미터를 가질 수 있다. 지네릭 파라미터는 함수 이름 앞에 꺽쇠 괄호를 사용해서 지정한다:

``` kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}
```

지네릭 함수에 대한 정보는 [지네릭](generics.html)을 참고한다.

## 인라인 함수

인라인 함수는 [여기](inline-functions.html)에서 설명한다.

## 확장 함수

확장 함수는 [여기](extensions.html)에서 설명한다.

## 고차 함수와 람다

고차 함수와 람다는 [여기](lambdas.html)에서 설명한다.

## 꼬리 재귀 함수

코틀린은 [꼬리 재귀](https://en.wikipedia.org/wiki/Tail_call)로 알려진
함수형 프로그래밍 방식을 지원한다.
이는 스택 오버플로우의 위험 없이 루프 대신 재귀 함수로 알고리즘을 작성할 수 있도록 한다.
함수가 `tailrec` 수식어가 있고 요구 형식을 충족하면, 컴파일러는 재귀를 최적화해서 빠르고 효율적인 루프 기반 버전으로 바꾼다:

``` kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

이 코드는 코사인의 fixpoint를 계산한다.
이 코드는 Math.cos을 1.0에서 시작해서 더 이상 값이 바뀌지 않을 때까지 반복해서 호출하며
결과로 0.7390851332151607을 생성한다.
결과 코드는 다음의 전통적인 방식과 동일하다:

``` kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```

`tailrec` 수식어가 가능하려면,
함수는 수행하는 마지막 연산으로 자기 자신을 호출해야 한다.
재기 호출 이후에 다른 코드가 있으면 꼬리 재귀를 사용할 수 없다.
그리고 try/catch/finally 블록 안에서 꼬리 재귀를 사용할 수 없다.
현재 꼬리 재귀늰 JVM 백엔드에서만 지원한다.
