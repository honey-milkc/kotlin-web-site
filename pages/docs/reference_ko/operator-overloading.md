---
type: doc
layout: reference
title: "연산자 오버로딩"
category: "Syntax"
---

# 연산자 오버로딩

코틀린은 타입에 대해 미리 정의한 연산자의 구현을 제공할 수 있다. 이 연산자는 `+`나 `*`과 같이 부호 표기가 고정되어 있고
정해진 [우선순위](grammar.html#precedence)를 갖는다.
연산자를 구현하려면 정해진 이름을 가진 [멤버 함수](functions.html#member-functions)나 [확장 함수](extensions.html)를
해당 타입에 제공한다. 이 타입은 이항 연산자의 왼쪽 타입이고 단항 연산자의 인자 타입이다.
연산자를 오버로딩한 함수는 `operator` 수식어를 붙여야 한다.

이어서 다른 연산자를 위한 연산자 오버로딩을 규정하는 관례를 살펴보자. 

## 단항 연산자

### 단항 접두 연사자

| 식 | 변환 |
|------------|---------------|
| `+a` | `a.unaryPlus()` |
| `-a` | `a.unaryMinus()` |
| `!a` | `a.not()` |

이 표는 컴파일러가 다음 절차에 따라 처리한다는 것을 보여준다(예로 `+a`를 사용했다):

* `a`의 타입을 결정한다. 타입을 `T`라고 해 보자.
* 리시버 `T`에 대해 파라미터가 없고 `operator`이고 이름이 `unaryPlus()`인 멤버 함수나 확장 함수를 찾는다.  
* 함수가 없거나 모호하면 컴파일 에러가 발생한다.
* 함수가 존재하고 리턴 타입이 `R`이면 `+a`식의 타입은 `R`이다.

이 오퍼레이션뿐만 아니라 다른 오퍼레이션은 [기본 타입](basic-types.html)에 대해 최적화를 하므로
함수 호출에 따른 오버헤드가 발생하지 않는다.

다음 예는 단항 빼기 연산자를 오버라이딩하는 방법을 보여준다:

``` kotlin
data class Point(val x: Int, val y: Int)

operator fun Point.unaryMinus() = Point(-x, -y)

val point = Point(10, 20)
println(-point)  // "(-10, -20)" 출력
```

### 증가와 감소

| 식 | 변환 |
|------------|---------------|
| `a++` | `a.inc()` + 아래 참고 |
| `a--` | `a.dec()` + 아래 참고 |

`inc()`와 `dec()` 함수는 `++`나  `--` 오퍼레이션을 사용하는 변수에 할당할 값을 리턴해야 한다.
두 함수는 `inc`나 `dec`가 불린 객체를 수정하면 안 된다.

컴파일러는 다음 절차에 따라 *접미사* 형식의 연산자를 처리한다(예로 `a++`를 사용했다).

* `a`의 타입을 결정한다. `T`라고 해 보자.
* `T` 타입의 리시버에 적용할 수 있는 파라미터 없고 `operator`가 붙은 `inc()` 함수를 찾는다.
* 함수의 리턴 타입이 `T`의 하위타입인지 검사한다.

계산 식의 효과는 다음과 같다:

* `a`의 초기 값을 임시 저장소 `a0`에 보관한다.
* `a.inc()`의 결과를 `a`에 할당한다.
* `a0`를 식의 결과로 리턴한다.

`a--` 처리 과정도 완전히 같다.

`++a`나 `--a`와 같은 *접두사* 형식도 동일 방식으로 동작하고, 효과는 다음과 같다:

* `a.inc()`를 `a`에 할당한다.
* `a`의 새 값을 식의 결과로 리턴한다.

## 이항 연산자

{:#arithmetic}

### 수치 연산자 

| 식 | 변환 |
| -----------|-------------- |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)`, `a.mod(b)` (deprecated) |
| `a..b ` | `a.rangeTo(b)` |

이 표의 연산자에 대해 컴파일러는 *변환* 칼럼의 식으로 변환만 한다.

코틀린 1.1부터 `rem` 연산자를 지원한다. 코틀린 1.0의 `mod` 연산자는 1.1에서 디프리케이트되었다.


#### 예제

아래 Counter 클래스 예제는 주어진 값으로 시작하고 오버로딩한 `+` 연산자를 사용해서 증가시킬 수 있다.

``` kotlin
data class Counter(val dayIndex: Int) {
    operator fun plus(increment: Int): Counter {
        return Counter(dayIndex + increment)
    }
}
```

{:#in}

### 'In' 연산자


| 식 | 변환 |
| -----------|-------------- |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |

`in`과 `!in`의 절차는 동일하다. 인자의 순서만 반대이다.

{:#indexed}

### 인덱스 기반 접근 연산자

| 식 | 변환 |
| -------|-------------- |
| `a[i]`  | `a.get(i)` |
| `a[i, j]`  | `a.get(i, j)` |
| `a[i_1, ...,  i_n]`  | `a.get(i_1, ...,  i_n)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a[i_1, ...,  i_n] = b` | `a.set(i_1, ..., i_n, b)` |

대괄호는 각각 알맞은 개수의 인자를 사용한 `get`과 `set` 호출로 바뀐다. 

{:#invoke}

### 호출 연산자

| 식 | 변환 |
|--------|---------------|
| `a()`  | `a.invoke()` |
| `a(i)`  | `a.invoke(i)` |
| `a(i, j)`  | `a.invoke(i, j)` |
| `a(i_1, ...,  i_n)`  | `a.invoke(i_1, ...,  i_n)` |

{:#assignments}

### 증강 할당(Augmented assignments)

| 식 | 변환 |
|------------|---------------|
| `a += b` | `a.plusAssign(b)` |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)` |
| `a %= b` | `a.remAssign(b)`, `a.modAssign(b)` (deprecated) |

위 할당 오퍼레이션에 대해, 예를 들어 `a += b`는 다음과 단계를 수행한다:

* 오른쪽 칼럼의 함수가 사용가능하면
  * 대응하는 이항 함수(예, `plusAssign()`의 경우 `plus()` 함수)도 사용가능하면 에러 발생(애매모호함 때문)
  * 리턴 타입이 `Unit`인지 확인하고 그렇지 않으면 에러
  * `a.plusAssign(b)`에 대한 코드 생성
* 사용가능하지 않으면, `a = a + b` 코드 생성을 시도한다(이는 타입 검사를 포함한다. `a + b`의 타입은 `a`의 상위타입이어야 한다)

*주의*: 코틀린에서 할당은 식이 *아니다*.

{:#equals}

### 동등과 비동등 연산자

| 식 | 변환 |
|------------|---------------|
| `a == b` | `a?.equals(b) ?: (b === null)` |
| `a != b` | `!(a?.equals(b) ?: (b === null))` |

*주의*: `===`와 `!==` (동일성 검사)는 오버로딩할 수 없어서 두 연산자를 위한 편의는 제공하지 않는다.

`==` 오퍼레이션은 특수하다. 이 식은 `null`을 선별하기 위해 복잡한 식으로 변환된다.
`null == null`은 항상 true이고 null이 아닌 `x`에 대해 `x == null`은 항상 false이며,`x.equals()`를 호출하지 않는다.

{:#comparison}

### 비교 연산자

| 식 | 변환 |
|--------|---------------|
| `a > b`  | `a.compareTo(b) > 0` |
| `a < b`  | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

모든 비교연산자는 `compareTo` 함수 호출로 바뀌고 `compareTo` 함수는 `Int`를 리턴해야 한다.

### 프로퍼티 위임 연산자
`provideDelegate`, `getValue`, `setValue` 연산자 함수에 대한 내용은 [위임 프로퍼티](delegated-properties.html)에서 설명한다.

## 이름 가진 함수를 위한 중위 호출

[중의 함수 호출](functions.html#infix-notation)을 사용해서 커스텀 중위 연산자를 흉내낼 수 있다. 
