---
type: doc
layout: reference
category: "Classes and Objects"
title: "실드 클래스"
---

# 실드 클래스

값이 제한된 타입 집합 중 하나를 가질 수 있지만 다른 타입은 가질 수 없을 때,
클래스 게층의 제한을 표현하기 위해 실드(Sealed) 클래스를 사용한다.
어떤 의미에서 실드 클래스는 enum 클래스를 확장한 것이다
enum 타입도 값 집합이 제한되어 있지만, 각 enum 상수는 단지 한 개 인스턴스로 존재하는 반면에
실드 클래스의 하위 클래스는 상태를 가질 수 있는 여러 인스턴스가 존재할 수 있다.

실드 클래스를 선언하려면 클래스 이름 앞에 `sealed` 수식어를 붙인다. 실드 클래스는 하위 클래스를 가질 수 있지만
모든 하위클래스는 반드시 실드 클래스와 같은 파일에 선언해야 한다.
(코틀린 1.1 이전에, 이 규칙은 더 엄격했었다. 실드 클래스 선언 내부에 클래스를 중첩해야 했다.)

``` kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

(위 예는 코틀린 1.1에 추가된 기능을 사용한다. 데이터 클래스는 실드 클래스를 포함한 클래스를 확장할 수 있다.)

실드 클래스 자체는 [추상](classes.html#abstract-classes)이므로, 바로 인스턴스화할 수 없으며
*abstract*{: .keyword } 멤버를 가질 수 있다.

실드 클래스는 *private*{: .keyword }이 아닌 생성자를 허용하지 않는다(실드 클래스의 생성자는 기본적으로 *private*{: .keyword }이다).

실드 클래스의 하위 클래스를 확장하는 클래스는 (간접 상속) 어디든 위치할 수 있다. 반드시 같은 파일일 필요는 없다.

실드 클래스를 사용해서 얻게 되는 핵심 이점은 [`when` 식](control-flow.html#when-expression)과 함께 사용할 때 드러난다.
when 식이 모든 경우를 다루는 것을 확신할 수 있다면 `else` 절을 문장에 추가할 필요가 없다.
하지만 이는 `when`을 문장이 아닌 식으로 사용하는 경우에만 동작한다.

``` kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // 모든 경우를 다루므로 `else` 절이 필요 없다
}
```
