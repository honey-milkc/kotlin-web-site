---
type: doc
layout: reference_ko
category: "Classes and Objects"
title: "데이터 클래스"
---

# 데이터 클래스

데이터를 보관하기 위한 목적으로 클래스를 자주 만든다.
그런 클래스는 종종 데이터에서 표준 기능이나 유틸리티 함수를 기계적으로 생성한다. 
코틀린에서 이를 _데이터 클래스_라고 부르며 `data`로 표시한다:
 
``` kotlin
data class User(val name: String, val age: Int)
```

컴파일러는 주요 생성자에 선언한 모든 프로퍼티로부터 다음 멤버를 생성한다:
  
  * `equals()`/`hashCode()` 쌍
  * `"User(name=John, age=42)"` 형식의 `toString()`
  * 선언 순서대로 프로퍼티에 대응하는 [`componentN()` 함수](multi-declarations.html) 
  * `copy()` 함수 (아래 참고).

생성한 코드가 일관성있고 의미있는 기능을 제공하기 위해 데이터 클래스는 다음 조건을 충족해야 한다: 

  * 주요 생성자는 최소 한 개의 파라미터가 필요하다.
  * 모든 주요 생성자 파라미터는 `val`이나 `var`여야 한다.
  * 데이터 클래스는 추상, open, sealed, 내부 클래스일 수 없다.
  * (1.1 이전에) 데이터 클래스는 오직 인터페이스만 구현해야 한다.

추가로 멤버 생성은 멤버 상속에 대해 다음 규칙을 따른다:

* 데이터 클래스 몸체에 `equals()`, `hashCode()`, `toString()` 구현이 있거나 또는 
* 상위클래스에 *final*{: .keyword } 구현이 있으면, 이 함수를 생성하지 않고 존재하는 구현을 사용한다.
* 상위타입에 *open*{: .keyword }이고 호환하는 타입을 리턴하는 `componentN()` 함수가 있다면
  데이터 클래스는 오버라이딩하는 대응하는 함수를 생성한다.
  상위타입 함수의 시그너처가 호환되지 않거나 final이어서 오버라이딩할 수 없는 경우 에러를 발생한다. 
* 코틀린 1.2에서 데이터 클래스가 시그너처가 같은 `copy(...)` 함수를 갖는 타입을 상속받는 것을 디프리케이트했고,
  코틀린 1.3에서는 금지할 것이다.
* `componentN()`과 `copy()` 함수를 직접 구현하는 것은 허용하지 않는다.

1.1부터, 데이터 클래스가 다른 클래스를 상속할 수 있다([실드 클래스](sealed-classes.html)에서 예를 볼 수 있다).

JVM에서 생성한 클래스에 파라미터 없는 생성자가 필요하면 모든 프로퍼티에 기본 값을 지정해야 한다([생성자](classes.html#constructors) 참고).

``` kotlin
data class User(val name: String = "", val age: Int = 0)
```

## 복사

객체를 복사할 때 프로퍼티의 _일부_만 변경하고 나머지는 그대로 유지해야 할 때가 있다.
이것이 `copy()` 함수를 생성한 이유이다. 위 `User` 클래스의 경우 구현이 다음과 같다: 
     
``` kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)     
```     

이를 통해 다음과 같이 코드를 작성할 수 있다:

``` kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## 데이터 클래스와 분리 선언

데이터 클래스에 생성되는 _컴포넌트 함수_는 [분리 선언](multi-declarations.html)에 데이터 클래스를 사용할 수 있게 한다.

``` kotlin
val jane = User("Jane", 35) 
val (name, age) = jane
println("$name, $age years of age") // "Jane, 35 years of age" 출력
```

## 표준 데이터 클래스

표준 라이브러리는 `Pair`와 `Triple`을 제공한다. 많은 경우 이름을 가진 데이터 클래스를 사용하는 것이 더 나은 설계 선택이다.
왜냐면 프러피티에 의미있는 이름을 사용해서 코드 가독성을 높여주기 때문이다.
