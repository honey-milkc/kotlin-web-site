---
type: doc
layout: reference_ko
category: "Syntax"
title: "확장(Extensions)"
---

# 확장(Extensions)

C#이나 Gosu처럼, 코틀린은 클래스 상속이나 데코레이터 같은 설계 패턴없이
클래스에 새로운 기능을 확장할 수 있는 기능을 제공한다.
_확장_이라고 부르는 특별한 선언을 통해 기능을 확장한다.
코틀린은 _확장 함수_와 _확장 프로퍼티_를 지원한다.

## 확장 함수

확장 함수를 선언하려면 _리시버(receiver) 타입_의 이름을 접두어로 가져야 한다.
리시버 타입의 이름은 확장할 타입의 이름이다.
다음 코드는 `MutableList<Int>`에 `swap` 함수를 추가한다:

``` kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this'는 List에 대응한다.
    this[index1] = this[index2]
    this[index2] = tmp
}
```

확장 함수에서 *this*{: .keyword } 키워드는 리시버 객체에 대응한다(리시버 객체는 
함수 이름에서 점 앞에 위치한 타입이다):

``` kotlin
val l = mutableListOf(1, 2, 3)
l.swap(0, 2) // 'swap()'에서 'this'는 'l' 값을 갖는다
```

이 함수는 `MutableList<T>`에도 적용되므로 지네릭으로 만들 수 있다:

``` kotlin
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // 'this'는 List에 대응한다
    this[index1] = this[index2]
    this[index2] = tmp
}
```

리시버 타입 식에서 사용할 수 지네릭 타입 파라미터를 함수 이름 앞에 선언했다. 
[지네릭 함수](generics.html)를 참고하자.

## **정적인** 확장 결정

확장이 실제로 확장할 클래스를 변경하지 않는다.
확장을 정의하는 것은 클래스에 새 멤버를 추가하기보다는, 그 타입의 변수에 점 부호로 호출할 수 있는 새 함수를 만드는 것이다.

확장 함수는 **정적으로** 전달된다는 점을 강조하고 싶다. 예를 들어, 리시버 타입에 따라 동적으로(virtual) 확장 함수를 결정하지 않는다.
이는 함수 호출 식의 타입에 따라 호출할 확장 함수를 결정한다는 것을 뜻한다.
런타임에 식을 평가한 결과 타입으로 결정하지 않는다. 다음 예를 보자:  

``` kotlin
open class C

class D: C()

fun C.foo() = "c"

fun D.foo() = "d"

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())
```

이 예는 "c"를 출력한다. printFoo() 함수의 c 파라미터 타입이 `C` 클래스이므로 `C` 타입에 대한 확장 함수를 호출하기 때문이다.

클래스가 해당 클래스를 리시버 타입으로 갖는 확장 함수와 동일한 멤버 함수를 가진 경우, **항상 멤버 함수가 이긴다**. 다음 예를 보자: 

``` kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }
```

`C`의 모든 `c`에 대해 `c.foo()` 호출은 "extension"이 아닌 "member"를 출력한다.

하지만 멤버 함수와 이름이 같지만 다른 시그너처를 갖도록 오버로딩한 확장 함수는 완전히 괜찮다:

``` kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo(i: Int) { println("extension") }
```

`C().foo(1)`는 "extension"을 출력한다.


## null 가능 리서버

확장이 null 가능 리시버 타입을 가질 수 있도록 정의할 수 있다.
이 확장은 객체 변수가 null인 경우에도 호출할 수 있고, 몸체 안에서 `this == null`로 이를 검사할 수 있다.
이는 코틀린에서 null 검사 없이 toString()을 호출할 수 있도록 한다.
확장 함수 안에서 이 검사를 한다.

``` kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // null 검사 후에 'this'는 자동으로 non-null 타입으로 변환된다. 따라서 아래 toString()을 모든 클래스의 
    // 멤버 함수로 처리한다.
    return toString()
}
```

## 확장 프로퍼티

함수와 유사하게, 코틀린은 확장 프로퍼티를 지원한다:

``` kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

확장은 실제로 클래스에 멤버를 추가하지 않으므로,
[지원 필드](properties.html#backing-fields)를 가진 확장 프로퍼티를 위한 효율적인 방법이 없다.
이것이 **확장 프로퍼티에 대한 초기화**를 허용하지 않는 이유이다.
이 기능은 명시적으로 getter/setter를 제공해서 정의할 수 있다.

예제:

``` kotlin
val Foo.bar = 1 // 에러: 확장 프로퍼티에 대한 초기화는 허용하지 않는다.
```


## 컴페니언 오브젝트 확장

클래스에 [컴페니언 오브젝트](object-declarations.html#companion-objects)가 있으면,
컴페니언 오브젝트를 위한 확장 함수와 프로퍼티를 정의할 수 있다:

``` kotlin
class MyClass {
    companion object { }  // "Companion"으로 접근
}

fun MyClass.Companion.foo() {
    // ...
}
```

컴페니언 오브젝트의 일반 멤버처럼 클래스 이름만 한정자로 사용해서 호출할 수 있다:

``` kotlin
MyClass.foo()
```


## 확장의 범위

대부분 패키지와 같은 최상위 수준에 확장을 정의한다.
 
``` kotlin
package foo.bar
 
fun Baz.goo() { ... } 
``` 

확장 함수를 선언한 패키지 밖에서 확장을 사용하려면 사용 위치에서 확장 함수를 임포트해야 한다:

``` kotlin
package com.example.usage

import foo.bar.goo // "goo"의 모든 확장을 임포트
                   // 또는
import foo.bar.*   // "foo.bar"로부터 모든 것을 임포트

fun usage(baz: Baz) {
    baz.goo()
}

```

더 많은 정보는 [임포트](packages.html#imports)를 참고한다.

## 멤버로 확장 선언하기

클래스 안에 다른 클래스를 위한 확장을 선언할 수 있다.
그런 확장안에서는 한정자 없이 접근할 수 있는 _암묵적인(implicit ) 리시버_ 객체 멤버가 존재한다.
확장 함수를 선언하고 있는 클래스의 인스턴스를 _디스패치 리시버(전달 리시버)_라고 부르며,
확장 메서드의 리시버 타입 인스턴스를 _확장 리시버_라고 부른다. 

``` kotlin
class D {
    fun bar() { ... }
}

class C {
    fun baz() { ... }

    fun D.foo() {
        bar()   // D.bar 호출
        baz()   // C.baz 호출
    }

    fun caller(d: D) {
        d.foo()   // 확장 함수를 호출
    }
}
```

디스패치 리시버와 확장 리시버의 멤버 간에 이름 충돌이 있는 경우 확장 리시버가 우선한다.
디스패치 리시버의 멤버를 참조하려면 [한정한 `this` 구문](this-expressions.html#qualified)을 사용하면 된다.

``` kotlin
class C {
    fun D.foo() {
        toString()         // D.toString() 호출
        this@C.toString()  // C.toString() 호출
    }
```

멤버로 선언한 확장을 `open`으로 선언할 수 있고, 이를 하위 클래스에서 오버라이딩할 수 있다.
이는 디스패치 리시버 타입에 따라 확장 함수를 선택함을 의미한다.
하지만 확장 리시버 타입에 대해서는 정적이다.

``` kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()   // 확장 함수를 호출
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

C().caller(D())   // "D.foo in C" 출력
C1().caller(D())  // "D.foo in C1" 출력 - 디스패치 리시버를 동적으로 선택
C().caller(D1())  // "D.foo in C" 출력 - 확장 리시버를 정적으로 선택
```

 
## 동기

자바는 `FileUtils`, `StringUtils` 등 "\*Utils"라는 이름을 가진 클래스를 사용한다. 유명한 `java.util.Collections`도 같은 부류다.
이런 Utils 클래스에서 마음에 안 드는 점은 코드가 다음과 같은 모양을 갖는다는 것이다:

``` java
// 자바
Collections.swap(list, Collections.binarySearch(list, Collections.max(otherList)), Collections.max(list));
```

클래스 이름이 항상 나온다. 정적 임포트를 사용하면 다음과 같다:

``` java
// 자바
swap(list, binarySearch(list, max(otherList)), max(list));
```

조금 나아졌지만 IDE의 강력한 코드 완성 기능을 거의 사용할 수 없다.
다음과 같이 코딩할 수 있다면 훨씬 더 좋을 것이다:

``` java
// Java
list.swap(list.binarySearch(otherList.max()), list.max());
```

하지만 `List` 클래스에 모든 가능한 메서드를 구현할 수는 없다. 확장이 돕는 것이 바로 이 지점이다.
