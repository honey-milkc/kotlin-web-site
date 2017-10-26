---
type: doc
layout: reference
category: "Syntax"
title: "범위"
---

# 범위

범위(range) 식은 연산자 형식이 `..`인 `rangeTo` 함수로 구성한다.
`..`는 *in*{: .keyword }과 *!in*{: .keyword }을 보완한다.
모든 비교가능한 타입에 대해 범위를 정의할 수 있는데, 정수 기본 타입의 경우 구현을 최적화한다.
다음은 범위의 사용 예이다:

``` kotlin
if (i in 1..10) { // 1 <= i && i <= 10과 동일
    println(i)
}
```

정수 타입 범위(`IntRange`, `LongRange`, `CharRange`)는 추가로 반복할 수 있는 특징이 있다.
컴파일러는 추가 오버헤드없이 정수 타입 범위를 동일한 자바의 인덱스 기반 *for*{: .keyword }-루프로 변환한다:

``` kotlin
for (i in 1..4) print(i) // "1234" 출력

for (i in 4..1) print(i) // 아무것도 출력하지 않음
```

숫자를 역순으로 반복하려면? 간단하다. 표준 라이브리에 있는 `downTo()` 함수를 사용하면 된다:

``` kotlin
for (i in 4 downTo 1) print(i) // "4321" 출력
```

1씩 간격이 아닌 다른 간격으로 숫자를 반복할 수 있나? 물론 가능하다. `step()`를 사용하면 된다:

``` kotlin
for (i in 1..4 step 2) print(i) // "13" 출력

for (i in 4 downTo 1 step 2) print(i) // "42" 출력
```

마지막 요소를 포함하지 않는 범위를 생성하려면 `until` 함수를 사용한다:

``` kotlin
for (i in 1 until 10) { // i in [1, 10), 10은 제외
     println(i)
}
```

## 동작 방식

범위는 라이브러리의 공통 인터페이스인 `ClosedRange<T>`를 구현한다.

`ClosedRange<T>`는 , 비교가능한 타입에 정의한 것으로 수학적 의미로 닫힌 구간을 표시한다.
이는 두 개의 끝점인 `start`와 `endInclusive`을 갖는다. 이 두 점은 범위에 포함된다.
주요 연산자인 `contains`는 *in*{: .keyword }/*!in*{: .keyword } 연산자에서 주로 사용한다.

정수 타입 Progression(`IntProgression`, `LongProgression`, `CharProgression`)은 산술적 증가를 표현한다.
Progression은 `first` 요소, `last`, 0이 아닌 `step`으로 정의한다.
첫 번째 요소는 `first`이고 이어지는 요소는 앞 요소에 `step`을 더한 값이 된다.
`last` 요소는 Progression이 비어서 더 이상 없을 때까지 반복을 통해 도달한다.

Progression은 `Iterable<N>`의 하위 타입으로 `N`은 각각 `Int`, `Long` 또는 `Char`이다.
따라서 Progression을 *for*{: .keyword }나 `map`, `fileter`와 같은 함수에서 사용할 수 있다.
`Progression`에 대한 반복은 자바/자바스크립트의 인덱스 기반 *for*{: .keyword }-루프와 동일하다:

``` java
for (int i = first; i != last; i += step) {
  // ...
}
```

정수 타입에서 `..` 연산자는 `ClosedRange<T>`와 `*Progression`를 구현한 객체를 생성한다.
예를 들어, `IntRange`는 `ClosedRange<Int>`를 구현했고, `IntProgression`를 상속하므로
`IntProgression`에 정의한 모든 오퍼레이션이 `IntRange`에서 사용가능하다..
`downTo()`와 `step()` 함수의 결과는 항상 `*Progression`이다.

Progression의 컴페니언 오브젝트가 제공하는 `fromClosedRange` 함수를 사용해서 Progression을 생성한다:

``` kotlin
IntProgression.fromClosedRange(start, end, step)
```

Progression의 `last` 요소는 양수 `step`에 대해서는 `end` 값보다 크지 않은 최댓값을 계산해서 구하고
음수 `step`에 대해서는 `end`보다 작지 않은 최솟값을 계산해서 구한다.
계산 결과로 얻은 값은 `(last - first) % step == 0`이 된다.


## 유틸리티 함수

### `rangeTo()`

정수 타입의 `rangeTo()` 연산자는 단순히 `*Range` 생서자를 호출한다:

``` kotlin
class Int {
    //...
    operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
    //...
    operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
    //...
}
```

실수 타입(`Double`, `Float`)은 `rangeTo` 연산자가 없고,
대신 범용 `Comparable`를 위해 표준 라이브러리가 제공하는 연산자를 사용한다:

``` kotlin
    public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

이 함수가 리턴하는 범위는 반복에서 사용할 수 없다.

### `downTo()`

모든 정수 타입 쌍에 대해 `downTo()` 확장 함수를 정의했다. 다음의 두 예를 보자:

``` kotlin
fun Long.downTo(other: Int): LongProgression {
    return LongProgression.fromClosedRange(this, other.toLong(), -1L)
}

fun Byte.downTo(other: Int): IntProgression {
    return IntProgression.fromClosedRange(this.toInt(), other, -1)
}
```

### `reversed()`

`reversed()` 확장 함수는 각 `*Progression` 클래스에 존재하며 역순의 Progression을 리턴한다:

``` kotlin
fun IntProgression.reversed(): IntProgression {
    return IntProgression.fromClosedRange(last, first, -step)
}
```

### `step()`

`*Progression` 클래스를 위한 `step()` 확장 함수를 제공한다,
이 함수는 `step` 값(함수 파라미터)을 갖는 Progression을 리턴한다.
증분 값은 항상 양수여야 하므로, 이 함수는 반복의 방향을 바꿀 수 없다:

``` kotlin
fun IntProgression.step(step: Int): IntProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return IntProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
    if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
    return CharProgression.fromClosedRange(first, last, if (this.step > 0) step else -step)
}
```

`(last - first) % step == 0` 규칙을 지키기 때문에
리턴한 Progression의 `last` 값은 원래 Progression의 `last` 값과 다를 수 있다:

``` kotlin
(1..12 step 2).last == 11  // progression with values [1, 3, 5, 7, 9, 11]
(1..12 step 3).last == 10  // progression with values [1, 4, 7, 10]
(1..12 step 4).last == 9   // progression with values [1, 5, 9]
```
