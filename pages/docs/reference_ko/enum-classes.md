---
type: doc
layout: reference
category: "Syntax"
title: "Enum 클래스"
---

# Enum 클래스

enum 클래스의 가장 기본적인 용도는 타입에 안전한 열거값을 구현하는 것이다:

``` kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

각 enum 상수는 객체이며, 콤마로 구분한다.

## 초기화

각 enum 값은 enum 클래스의 인스턴스로 초기화할 수 있다:

``` kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

## 익명 클래스

enum 상수는 자신만의 익명 클래스를 선언할 수 있다:

``` kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```

익명 클래스는 자신의 메서드를 가질 수 있고 또한 기반 메서드를 오버라이딩할 수 있다.
enum 클래스가 멤버를 정의할 경우, 자바와 마찬가지로 enum 상수 정의와 멤버를 세미콜론으로 구분해야 한다.

## enum 상수 사용

자바와 동일하게 코틀린의 enum 클래스는 정의한 enum 상수 목록을 구하거나 이름으로 enum 상수를 구하는
메서드를 제공한다. 이 메서드의 시그너처는 다음과 같다(enum 클래스의 이름이 `EnumClass`라고 가정):

``` kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

`valueOf()` 메서드는 지정한 이름에 해당하는 enum 상수가 존재하지 않으면 `IllegalArgumentException`을 발생한다.

코틀린 1.1부터 `enumValues<T>()`와 `enumValueOf<T>()` 함수를 사용해서 지네릭 방식으로 enum 클래스에 있는 상수에 접근할 수 있다:

``` kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // RED, GREEN, BLUE 출력
```

모든 enum 상수에는 enum 클래스에 선언한 상수의 이름과 위치를 제공하는 프로퍼티가 존재한다:

``` kotlin
val name: String
val ordinal: Int
```

enum 상수는 또한 [Comparable](/api/latest/jvm/stdlib/kotlin/-comparable/index.html) 인터페이스를 구현하고 있어서
enum 클래스에 정의된 순서를 기준으로 비교를 할 수 있다.
