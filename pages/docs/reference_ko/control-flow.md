---
type: doc
layout: reference_ko
category: "Syntax"
title: "흐름 제어: if, when, for, while"
---

# 흐름 제어: if, when, for, while

## If 식
{:#if-expression}

코틀린에서 *if*{: .keyword }는 식(expression)으로, 값을 리턴한다.
그래서 삼항 연산자(condition ? then : else)가 없다. 일반 *if*{: .keyword }로 동일하게 할 수 있기 때문이다.

``` kotlin
// 전통적인 용법
var max = a 
if (a < b) max = b

// else 사용 
var max: Int
if (a > b) {
    max = a
} else {
    max = b
}
 
// 식으로 
val max = if (a > b) a else b
```

*if*{: .keyword } 브랜치는 블록일 수 있으며 마지막 식이 블록이 값이 된다.

``` kotlin
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

문장이 아닌 식으로 *if*{: .keyword }를 사용하면(예를 들어 식의 값을 리턴하거나 변수에 값을 할당),
`else` 브랜치를 가져야 한다. 

[*if*{: .keyword } 문법](http://kotlinlang.org/docs/reference/grammar.html#if)을 보자.

## When 식
{:#when-expression}

*when*{: .keyword }은 C와 같은 언어의 switch 연산에 해당한다. 가장 간단한 형식은 다음과 같다:

``` kotlin
when (x) {
    1 -> print("x == 1")
    2 -> print("x == 2")
    else -> { // Note the block
        print("x is neither 1 nor 2")
    }
}
```

*when*{: .keyword }은 특정 브랜치의 조건을 충족할 때까지 순차적으로 모든 브랜치의 인자가 일치하는지 확인한다.
*when*{: .keyword }은 식이나 문장으로 사용할 수 있다. 식으로 사용하면 충족한 브랜치의 값이 전체 식의 값이 된다.
문장으로 사용하면 개별 브랜치의 값은 무시한다. (*if*{: .keyword }처럼 각 브랜치는 블록이 될 수 있으며, 블록의 마지막 식의 값이 브랜치의 값이 된다.)

모든 *when*{: .keyword } 브랜치가 충족하지 않으면 *else*{: .keyword } 브랜치를 평가한다.
*when*{: .keyword }을 식으로 사용할 때, 모든 가능한 경우를 브랜치 조건으로 확인했는지 컴파일러가 알 수 없는 경우
*else*{: .keyword } 브랜치를 필수로 추가해야 한다.

여러 경우를 동일한 방법으로 처리할 경우 브랜치 조건을 콤마로 묶을 수 있다:

``` kotlin
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

브랜치 조건으로 (상수뿐만 아니라) 임의의 식을 사용할 수 있다.

``` kotlin
when (x) {
    parseInt(s) -> print("s encodes x")
    else -> print("s does not encode x")
}
```

*in*{: .keyword }, *!in*{: .keyword }, [범위](ranges.html), 콜렉션을 사용해서 값을 검사할 수 있다:

``` kotlin
when (x) {
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

*is*{: .keyword }나 *!is*{: .keyword }로 특정 타입 여부를 확인할 수 있다.
[스마트 변환](typecasts.html#smart-casts) 덕분에 추가 검사없이 타입의 메서드와 프로퍼티에 접근할 수 있다.

```kotlin
fun hasPrefix(x: Any) = when(x) {
    is String -> x.startsWith("prefix")
    else -> false
}
```

*if*{: .keyword }-*else*{: .keyword } *if*{: .keyword } 체인 대신에 *when*{: .keyword }을 사용할 수도 있다.
인자를 제공하지 않으면 브랜치 조건은 단순한 블리언 식이 되고, 브랜치의 조건이 true일 때 브랜치를 실행한다:

``` kotlin
when {
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

[*when*{: .keyword } 문법](http://kotlinlang.org/docs/reference/grammar.html#when)을 참고하자.


## for 루프
{:#for-loops}

*for*{: .keyword }는 이터레이터를 제공하는 모든 것에 대해 반복을 수행한다. C#과 같은 언어의 `foreach` 루프와 동일하다.
구문은 다음과 같다:

``` kotlin
for (item in collection) print(item)
```

몸체는 블록일 수 있다.

``` kotlin
for (item: Int in ints) {
    // ...
}
```

이전에 언급했듯이, *for*{: .keyword }는 이터레이터를 제공하는 모든 것에 동작한다.

* `iterator()` 멤버 함수나 확장 함수를 갖고 있고, 함수의 리턴 타입이
  * `next()` 멤버 함수나 확장 함수를 갖고,
  * 리턴 타입이 `Boolean`인 `hasNext()` 멤버 함수나 확장 함수를 가짐

이 세 함수는 모두 `operator`로 지정해야 한다.

배열에 대한 `for` 루프는 이터레이터 객체를 생성하지 않는 인덱스 기반 루프로 컴파일된다.

인덱스를 이용해서 배열이나 리스트를 반복하려면 다음과 같이 하면 된다:

``` kotlin
for (i in array.indices) {
    print(array[i])
}
```

"범위에 대한 반복"은 최적화한 구현으로 컴파일해서 객체를 추가로 생성하지 않는다.

indices 대신 `withIndex` 라이브러리 함수를 사용해도 된다:

``` kotlin
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

[*for*{: .keyword } 문법](http://kotlinlang.org/docs/reference/grammar.html#for)을 참고하자.

## While 루프
{:#while-loops}

*while*{: .keyword }과 *do*{: .keyword }..*while*{: .keyword } 예:

``` kotlin
while (x > 0) {
    x--
}

do {
    val y = retrieveData()
} while (y != null) // 여기서 y를 사용할 수 있다!
```

[*while*{: .keyword } 문법](http://kotlinlang.org/docs/reference/grammar.html#while)을 참고하자.

## 루프에서의 break와 continue

코틀린은 루프에서 전통적인 *break*{: .keyword }와 *continue*{: .keyword } 연산자를 지원한다.
[리턴과 점프](returns.html)를 참고한다.

