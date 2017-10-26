---
type: doc
layout: reference
category: "Other"
title: "동등성(Equality)"
---

# 동등성(Equality)

코틀린에는 두 가지 종류의 동등성이 있다:

* 참조 동등성 (두 참조가 동일 객체를 가리킴)
* 구조 동등성 (`equals()`로 검사).

## 참조 동등

참조 동등은 `===` 오퍼레이션으로 검사한다(반대는 `!==`로 검사).
`a === b`는 `a`와 `b`가 동일 객체를 가리키는 경우에만 true이다.

## 구조 동등

구조 동등은 `==` 오퍼레이션으로 검사한다(반대는 `!=`로 검사).
`a == b`와 같은 식을 다음과 같이 바꾼다.

``` kotlin
a?.equals(b) ?: (b === null)
```

`a`가 `null`이 아니면 `equals(Any?)` 함수를 호출하고, `a`가 `null`이면 `b`가 `null`인지 검사한다.
`null`을 직접 비교할 때에는 코드를 최적화할 필요가 없다.
`a == null`를 자동으로 `a === null`로 바꾼다.

## 부동 소수점 숫자의 동등

동등 비교의 피연산자가 정적으로 (nullable이거나 아닌) `Float`나 `Double`임을 알 수 있으면,
부동 소수점 계산을 위한 IEEE 754 표준에 따라 검사한다.

이 외의 경우 구조 동등을 사용한다.
이는 표준을 따르지 않으므로 `NaN`는 그 자신과 같고, `-0.0`은 `0.0`과 같지 않다.

[부동 소수점 비교](basic-types.html#floating-point-numbers-comparison)를 참고한다.
