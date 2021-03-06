---
type: doc
layout: reference_ko
category: "Classes and Objects"
title: "가시성 수식어"
---

# 가시성 수식어

클래스, 오브젝트, 인터페이스, 생성자, 함수, 프로퍼티 및 프로퍼티의 setter는 _가시성 수식어_를 가질 수 있다.
(getter는 항상 프로퍼티와 동일한 가시성을 갖는다.)
코틀린에는 `private`, `protected`, `internal`, `public`의 가시성 수식어가 있다.
수식어를 지정하지 않을 경우 기본 가시성은 `public`이다.

타입과 선언 범위에 따른 가시성에 대해서는 뒤에서 설명한다.
  
## 패키지

함수, 프로퍼티와 클래스, 오브젝트와 인터페이스는 "최상위(top-level)"에 선언할 수 있다.
예를 들어 패키지에 직접 선언할 수 있다.
  
``` kotlin
// file name: example.kt
package foo

fun baz() {}
class Bar {}
```

* 가시성 수식어를 명시하지 않으면 기본으로 `public`을 사용한다. 이는 모든 곳에서 접근 가능함을 뜻한다. 
* `private`으로 선언하면, 그 선언을 포함한 파일 안에서만 접근할 수 있다.
* `internal`로 선언하면, 같은 [모듈](#modules)에서 접근 가능하다.
* `protected`는 최상위 선언에 사용할 수 없다.

주의: 다른 패키지에서 최상위 선언에 접근하려면 그것을 [임포트](packages.html#imports)해야 한다.

예제:

``` kotlin
// 파일 이름: example.kt
package foo

private fun foo() {} // example.kt 안에서 접근 가능

public var bar: Int = 5 // 모든 곳에서 접근 가능
    private set         // setter는 example.kt에서만 접근 가능
    
internal val baz = 6    // 같은 모듈에서 접근 가능
```

## 클래스와 인터페이스

클래스에 선언한 멤버에 대해서는 다음과 같다:

* `private`은 오직 클래스 안에서만(그리고 클래스의 모든 멤버에서) 접근 가능함을 의미한다.
* `protected` --- `private` + 하위클래스에서 접근 가능함과 동일하다.
* `internal` --- 선언한 클래스를 볼 수 있는 *모듈 안의* 모든 클라이언트가 `internal` 멤버를 볼 수 있다.
* `public` --- 선언한 클래스를 볼 수 있는 클라이언트가 `public` 멤버를 볼 수 있다.

*주의* 자바와 달리 코틀린에서 외부 클래스는 내부 클래스의 private 멤버를 볼 수 없다.

`protected` 멤버를 오버라이딩할 때 가시성을 명시적으로 지정하지 않으면,
오버라이딩한 멤버 또한 `protected` 가시성을 갖는다.
 
예제:

``` kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // 기본으로 public
    
    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a는 접근 불가
    // b, c, d는 접근 가능
    // Nested와 e는 접근 가능

    override val b = 5   // 'b'는 protected
}

class Unrelated(o: Outer) {
    // o.a, o.b는 접근 불가
    // o.c 와 o.d는 접근 가능(같은 모듈)
    // Outer.Nested는 접근 불가며, Nested::e 역시 접근 불가
}
```

{:#constructors}

### 생성자

``` kotlin
class C private constructor(a: Int) { ... }
```

위 코드의 생성자는 private이다. 기본적으로 모든 생성자는 `public`이며
실질적으로 클래스를 접근할 수 있는 모든 곳에서 생성자에 접근할 수 있다.
(예를 들어 `internal` 클래스의 생성자는 오직 같은 모듈에서만 보인다.)

### 로컬 선언

로컬 변수, 로컬 함수, 로컬 클래스에는 가시성 수식어를 지정할 수 없다.

## 모듈
{:#modules}

`internal` 가시성 수식어는 같은 모듈에서 멤버에 접근할 수 있음을 의미한다.
더 구체적으로 모듈은 함께 컴파일되는 코틀린 파일 집합이다.  

  * IntelliJ IDEA 모듈
  * 메이븐 프로젝트
  * 그레이들 소스 집합
  * kotlinc 앤트 태스크를 한 번 호출할 때 컴파일되는 파일 집합
