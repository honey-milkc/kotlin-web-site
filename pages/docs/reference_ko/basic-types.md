---
type: doc
layout: reference_ko
category: "Syntax"
title: "기본 타입: 숫자, 문자열, 배열"
---

# 기본 타입

코틀린에서 모든 것은 객체로서 변수에 대한 멤버 함수나 프로퍼티를 호출할 수 있다.
어떤 타입은 특별한 내부 표현을 갖지만-예를 들어 숫자, 문자, 불리언 같은 타입은 런타임에 기본 값으로 표현된다- 사용자에게는 일반 클래스처럼 보인다.
이 절에서는 코틀린의 기본 타입인 숫자, 문자, 불리언, 배열, 문자열에 대해 설명한다.

## 숫자

코틀린은 자바와 유사한 방법으로 숫자를 다루는데 완전히 같지는 않다. 예를 들어 숫자에 대해 넓은 타입으로의 자동 변환이 없고,
어떤 경우에는 리터럴도 약간 다르다.

코틀린이 제공하는 숫자 내장 타입은 다음과 같다(이는 자바와 유사하다):

| 타입	 | 비트 크기  |
|--------|----------|
| Double | 64       |
| Float	 | 32       |
| Long	 | 64       |
| Int	 | 32       |
| Short	 | 16       |
| Byte	 | 8        |

코틀린에서 문자는 숫자가 아니다.

### 리터럴 상수

정수 값을 위한 리터럴 상수 종류는 다음과 같다:

* 십진수: `123`
  * Long은 대문자 `L`로 표시: `123L`
* 16진수: `0x0F`
* 2진수: `0b00001011`

주의: 8진수 리터럴은 지원하지 않음

또한 부동소수점을 위한 표기법을 지원한다:
 
* 기본은 Double: `123.5`, `123.5e10`
* Float은 `f`나 `F`로 표시: `123.5f`
 
### 숫자 리터럴에 밑줄(1.1부터)
 
숫자 상수의 가독성을 높이기 위해 밑줄을 사용할 수 있다:

``` kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 표현

자바 플랫폼에서는 JVM 기본 타입으로 숫자를 저장한다.
만약 null 가능 숫자 레퍼런스(예 `Int?`)가 필요하거나 지네릭이 관여하면 박싱(boxing) 타입으로 저장한다. 

숫자를 박싱하면 동일성(Identity)을 유지하지 않는다:

``` kotlin
val a: Int = 10000
print(a === a) // 'true' 출력
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // !!!'false' 출력!!!
```

하지만 값의 동등함은 유지한다:

``` kotlin
val a: Int = 10000
print(a == a) // 'true' 출력
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // 'true' 출력
```

### 명시적 변환

표현이 다르므로 작은 타입이 큰 타입의 하위 타입은 아니다.
하위 타입이 된다면 다음과 같은 문제가 발생한다:

``` kotlin
// 가상의 코드로, 실제로는 컴파일되지 않음:
val a: Int? = 1 // 박싱된 Int (java.lang.Integer)
val b: Long? = a // 자동 변환은 박싱된 Long을 생성 (java.lang.Long)
print(a == b) // 놀랍게도 이는 "false"를 리턴한다! 왜냐면 Long의 equals()은 비교 대상도 Long인지 검사하기 때문이다
```

동일성뿐만 아니라 동등함조차 모든 곳에서 나도 모르게 사라지게 된다.

이런 이유로 작은 타입을 큰 타입으로 자동으로 변환하지 않는다.
이는 명시적 변환없이 `Byte` 타입 값을 `Int` 변수에 할당할 수 없음을 뜻한다.

``` kotlin
val b: Byte = 1 // OK, 리터럴은 정적으로 검사함
val i: Int = b // ERROR
```

명시적으로 숫자를 넓히는 변환을 할 수 있다:

``` kotlin
val i: Int = b.toInt() // OK: 명시적으로 넓힘
```

모든 숫자 타입은 다음 변환을 지원한다:

* `toByte(): Byte`
* `toShort(): Short`
* `toInt(): Int`
* `toLong(): Long`
* `toFloat(): Float`
* `toDouble(): Double`
* `toChar(): Char`

자동 변환의 부재를 거의 느낄 수 없는데 이유는 타입을 컨텍스트에서 추론하고 수치 연산을 변환에 알맞게 오버로딩했기 때문이다. 다음은 예이다.

``` kotlin
val l = 1L + 3 // Long + Int => Long
```

### 연산
{:#operations}

코틀린은 숫자에 대한 표준 수치 연산을 지원한다. 이들 연산은 알맞은 클래스의 멤버로 선언되어 있다.
(하지만 컴파일러는 메서드 호출을 대응하는 명령어로 최적화한다.)
[연산자 오버로딩](operator-overloading.html)을 참고하자.

비트 연산자를 위한 특스 문자는 없고 중의 형식으로 호출할 수 있는 함수를 제공한다. 다음은 예이다:

``` kotlin
val x = (1 shl 2) and 0x000FF000
```

다음은 전체 비트 연산자 목록이다(`Int`와 `Long`에서만 사용 가능):

* `shl(bits)` – 부호있는 왼쪽 시프트 (자바의  `<<`)
* `shr(bits)` – 부호있는 오른쪽 시프트 (자바의 `>>`)
* `ushr(bits)` – 부호없는 오른쪽 시프트 (자바의 `>>>`)
* `and(bits)` – 비트 AND
* `or(bits)` – 비트 OR
* `xor(bits)` – 비트 XOR
* `inv()` – 비트 역

{:#floating-point-numbers-comparison}

### 부동소수 비교

이 절에서는 실수에 대한 연산을 설명한다:

* 동등함 비교: `a == b` 와 `a != b`
* 비교 연산자: `a < b`, `a > b`, `a <= b`, `a >= b`
* 범위 생성과 검사: `a..b`, `x in a..b`, `x !in a..b`

피연산자 `a`와 `b`가 정적으로 `Float`나 `Double`이거나 또는 이에 해당하는 null 가능 타입일 때(타입을 선언하거나 추정하거나 또는 [스마트 변환](typecasts.html#smart-casts)의 결과),
숫자나 범위에 대한 연산은 부동소수 연산에 대한 IEEE 754 표준을 따른다.

하지만 범용적인 용례를 지원하고 완전한 순서를 제공하기 위해,
피연산자가 정적으로 부동소수점 타입이 아니면(예 `Any`, `Comparable<...>`, 타입 파라미터)
연산은 `Float`과 `Double`을 위한 `equals`와 `compareTo` 구현을 사용한다. 이는 표준과 비교해 다음이 다르다.

* `NaN`은 자신과 동일하다고 간주한다
* `NaN`은 `POSITIVE_INFINITY`를 포함한 모든 다른 요소보다 크다고 간주한다
* `-0.0`는 `0.0`보다 작다고 간주한다

{:#characters}

## 문자

`Char` 타입으로 문자를 표현한다. 이 타입을 바로 숫자로 다룰 수 없다.

``` kotlin
fun check(c: Char) {
    if (c == 1) { // ERROR: 비호환 타입
        // ...
    }
}
```

문자 리터럴은 작은 따옴표 안에 표시한다: `'1'`.
특수 문자를 역슬래시로 표시한다.
다음 특수문자를 지원한다: `\t`, `\b`, `\n`, `\r`, `\'`, `\"`, `\\`, `\$`.
임의의 문자를 인코딩하려면 유니코드 표기법을 사용한다: `'\uFF00'`.

문자를 `Int` 숫자로 명시적으로 변환할 수 있다:

``` kotlin
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // 숫자로 명시적 변환
}
```

숫자와 마찬가지로, 문자도 null 가능 레퍼런스가 필요하면 박싱된다. 박싱 연산을 하면 동일성은 유지되지 않는다.

## 불리언
{:#booleans}

`Boolean` 타입은 불리언을 표현하고 두 값이 존재한다: *true*{: .keyword }와 *false*{: .keyword }.

null 가능 레퍼런스가 필요하면 불리언도 박싱된다.

불리언에 대한 내장 연산에는 다음이 있다.

* `||` – 지연 논리합
* `&&` – 지연 논리곱
* `!` - 역

{:#arrays}

## 배열

코틀리은 `Array` 클래스로 배열을 표현하며, 이 클래스는 `get`과 `set` 함수(연산자 오버로딩 관례에 따라 `[]`로 바뀜),
`size` 프로퍼티와 그외 유용한 멤버 함수를 제공한다:

``` kotlin
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ...
}
```

라이브러리 함수 `arrayOf()`를 사용해서 배열을 생성할 수 있다. 이 함수에는 항목 값을 전달하는데 `arrayOf(1, 2, 3)`은 [1, 2, 3] 배열을 생성한다.
`arrayOfNulls()` 라이브러리 함수를 사용하면 주어진 크기의 null로 채운 배열을 생성할 수 있다.

배열을 생성하는 다른 방법은 팩토리 함수를 사용하는 것이다. 팩토리 함수는 배열 크기와 해당 인덱스에 위치한 요소의 초기값을 리턴하는 함수를 인자로 받는다:

``` kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

앞서 말했듯이, `[]` 연산자는 `get()`과 `set()` 멤버 함수 호출을 의미한다.

주의: 자바와 달리 코틀린 배열은 무공변(invariant)이다. 이는 코틀린은 `Array<String>`을 `Array<Any>`에 할당할 수 없음을 의미하며, 런타임 실패를 방지한다.
(하지만 `Array<out Any>`을 사용할 수 있다. [타입 프로젝션](generics.html#type-projections)을 참고하자.)
 
코틀린은 또한 `ByteArray`, `ShortArray`, `IntArray` 등 기본 타입의 배열을 표현하기 위한 특수 클래스를 제공한다. 이들 클래스는 박싱 오버헤드가 없다.
이 클래스는 Array 클래스와 상속 관계에 있지 않지만 같은 메서드와 프로퍼티를 제공한다. 각 배열 타입을 위한 팩토리 함수를 제공한다:


``` kotlin
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

{:#strings}

## 문자열

문자열은 `String` 타입으로 표현한다. 문자열은 불변이다.
문자열의 요소는 문자로서 `s[i]`와 같은 인덱스 연산으로 접근할 수 있다.
*for*{: .keyword }-루프로 문자열을 순회할 수 있다:

``` kotlin
for (c in str) {
    println(c)
}
```

### 문자열 리터럴

코틀린은 두 가지 타입의 문자열 리터럴을 지원한다. 하나는 escaped 문자열로 탈출 문자를 가질 수 있다.
다른 하나는 raw 문자열로 뉴라인과 임의 텍스트를 가질 수 있다.
escaped 문자열은 자바 문자열과 거의 같다:

``` kotlin
val s = "Hello, world!\n"
```

특수 문자는 전형적인 방식인 역슬래시를 사용한다.
지원하는 특수 문자 목록은 앞서 [문자](#characters)를 참고한다.

raw 문자열은 세 개의 따옴표로 구분하며(`"""`),
특수 문자를 포함하지 않고 뉴라인과 모든 다른 문자를 포함할 수 있다:

``` kotlin
val text = """
    for (c in "foo")
        print(c)
"""
```

[`trimMargin()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/trim-margin.html) 함수를 사용해서
앞쪽 공백을 제거할 수 있다:

``` kotlin
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

기본으로 `\`를 경계 접두문자로 사용하지만 `trimMargin(">")`과 같이 파라미터를 이용해서 다른 문자를 경계 문자로 사용할 수 있다.

{:#string-templates}

### 문자열 템플릿

문자열은 템플릿 식을 포함할 수 있다. 예를 들어, 코드 조각을 계산하고 그 결과를 문자열에 넣을 수 있다.
템플릿 식은 달러 부호($)로 시작하고 간단한 이름으로 구성된다:

``` kotlin
val i = 10
val s = "i = $i" // "i = 10"로 평가
```

또는 중괄호 안에 임의의 식을 넣을 수 있다:

``` kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // "abc.length is 3"로 평가
```

raw 문자열과 escaped 문자열 안에 템플릿을 넣을 수 있다.
역슬래시 특수 문자를 지원하지 않는 raw 문자열에서 `$` 문자를 표현하고 싶다면
다음 구문을 사용한다:

``` kotlin
val price = """
${'$'}9.99
"""
```
