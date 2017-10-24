---
type: doc
layout: reference
category: "Syntax"
title: "애노테이션"
---

# 애노테이션

## 애노테이션 선언

애노테이션은 코드에 메타데이터를 붙이는 방법이다. 클래스 앞에 *annotation*{: .keyword } 수식어를 붙여서 애노테이션을 선언한다:

``` kotlin
annotation class Fancy
```

애노테이션 클래스에 메타-애노테이션을 붙여서 애노테이션에 추가 속성을 지정할 수 있다:

  * [`@Target`](/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html)은 
    애노테이션을 어떤 요소에(클래스, 함수, 프로퍼티, 식 등) 적용할 수 있는지 지정한다.
  * [`@Retention`](/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html)은
    애노테이션을 컴파일한 클래스 파일에 보관할지, 런타임에 리플렉션을 통해서 접근할 수 있는 여부를 지정한다.
    (기본값을 둘 다 true이다) 
  * [`@Repeatable`](/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html)은
    한 요소의 같은 애노테이션을 여러 번 적용하는 것을 허용한다.
  * [`@MustBeDocumented`](/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html)은 
    애노테이션이 공개 API에 속하며 생성한 API 문서에서 클래스나 메서드 시그너처에 포함시켜야 함을 지정한다.

``` kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
annotation class Fancy
```

### 사용법

``` kotlin
@Fancy class Foo {
    @Fancy fun baz(@Fancy foo: Int): Int {
        return (@Fancy 1)
    }
}
```

클래스의 주요 생성자에 애노테이션이 필요하면, *constructor*{: .keyword} 키워드를 생성자 선언에 추가하고
그 앞에 애노테이션을 추가해야 한다:


``` kotlin
class Foo @Inject constructor(dependency: MyDependency) {
    // ...
}
```

프로퍼티 접근자에도 애노테이션을 붙일 수 있다:

``` kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### 생성자

애노테이션은 파라미터가 있는 생성자를 가질 수 있다.

``` kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

허용하는 파라미터 타입은 다음과 같다:

 * 자바 기본 타입에 대응하는 타입((Int, Long 등)
 * 문자열
 * 클래스 (`Foo::class`);
 * 열거 타입)
 * 다른 애노테이션
 * 위에서 열거한 타입의 배열

추가 파라미터는 nullable 타입일 수 없는데, 왜냐면 JVM은 애노테이션 속성의 값으로 `null`을 저장하는 것을 지원하지 않기 때문이다.

다른 애노테이션의 파라미터로 애노테이션을 사용할 경우, 해당 애노테이션의 이름에 @ 문자를 접두어로 붙이지 않는다.

``` kotlin
annotation class ReplaceWith(val expression: String)

annotation class Deprecated(
        val message: String,
        val replaceWith: ReplaceWith = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```

애노테이션의 인자로 클래스를 사용하고 싶다면, 코틀린 클래스([KClass](/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html))를 사용한다.
코틀린 컴파일러는 이를 자동으로 자바 클래스로 변환하므로, 자바 코드에서 애노테이션와 인자를 사용할 수 있다.

``` kotlin

import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any?>)

@Ann(String::class, Int::class) class MyClass
```

### 람다

람다에도 애노테이션을 사용할 수 있다. 람다의 몸체에 생성된 `invoke()` 메서드에 적용된다.
이는 병렬 제어를 위한 애노테이션을 사용하는 [Quasar](http://www.paralleluniverse.co/quasar/)와 같은
프레임워크에 유용하다.

``` kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```

## 애노테이션 사용 위치 대상

프로퍼티나 주요 생성자 파라미터에 애노테이션을 적용할 때, 
코틀린 요소에서 여러 자바 요소가 생성될 수 있고 따라서 생성된 자바 바이트코드에는 애노테이션이
여러 위치에 붙을 수 있다.
애노테이션을 정확하게 어떻게 생성할지 지정하려면 다음 구문을 사용한다:

``` kotlin
class Example(@field:Ann val foo,    // 자바 필드에 적용
              @get:Ann val bar,      // 자바 getter에 적용
              @param:Ann val quux)   // 자바 생성자 파라미터에 적용
```

같은 구문을 전체 파일에 애노테이션을 적용할 때에 사용할 수 있다.
이를 위해 파일의 최상단에 패키지 디렉티브 이전에 또는 기본 패키지면 모든 임포트 이전에 
적용 대상으로 `file`을 가진 애노테이션을 붙인다:

``` kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

동일 대상에 여러 애노테이션을 붙일 경우 대상 뒤의 대괄호에 모든 애노테이션을 위치시켜서
대상을 반복하는 것을 피할 수 있다:

``` kotlin
class Example {
     @set:[Inject VisibleForTesting]
     var collaborator: Collaborator
}
```

지원하는 사용 위치 대상은 다음과 같다:

  * `file`
  * `property` (이 대상을 가진 애노테이션은 자바에는 보이지 않는다)
  * `field`;
  * `get` (프로퍼티 getter)
  * `set` (프로퍼티 setter)
  * `receiver` (확장 함수나 프로퍼티의 리시버 파라미터)
  * `param` (생성자 파라미터);
  * `setparam` (프로퍼티 setter 파라미터);
  * `delegate` (위임 프로퍼티의 위임 인스턴스를 보관하는 필드).

확장 함수의 리시버 파라미터에 애노테이션을 붙이려면 다음 구문을 사용한다:

``` kotlin
fun @receiver:Fancy String.myExtension() { }
```

사용위치 대상을 지정하지 않으면, 사용할 애노테이션의 `@Target` 애노테이션에 따라 대상을 선택한다.
적용 가능한 대상이 여러 개면, 다음 목록에서 먼저 적용가능한 대상을 사용한다:

  * `param`
  * `property`
  * `field`


## 자바 애노테이션

자바 애노테이션은 코틀린과 100% 호환된다:

``` kotlin
import org.junit.Test
import org.junit.Assert.*
import org.junit.Rule
import org.junit.rules.*

class Tests {
    // 프로퍼티 getter에 @Rule 애노테이션을 적용
    @get:Rule val tempFolder = TemporaryFolder()

    @Test fun simple() {
        val f = tempFolder.newFile()
        assertEquals(42, getTheAnswer())
    }
}
```

자바에서 작성한 애노테이션는 파라미터의 순서를 정의하지 않기 때문에
인자를 전달하기 위해 일반적인 함수 호출 구문을 사용할 수 있다.
대신 이름을 가진 인자 구문을 사용해야 한다:

``` java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

``` kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

자바와 같이 `value` 파라미터는 특별한 경우이다. `value` 파라미터의 값은 이름 없이 지정할 수 있다:

``` java
// Java
public @interface AnnWithValue {
    String value();
}
```

``` kotlin
// Kotlin
@AnnWithValue("abc") class C
```

자바의 `value` 인자가 배열 타입이면, 코틀린에서 `vararg` 파라미터가 된다:

``` java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

``` kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```

배열 타입을 가진 다른 인자의 경우 `arrayOf`를 직접 사용해야 한다:

``` java
// Java
public @interface AnnWithArrayMethod {
    String[] names();
}
```

``` kotlin
// Kotlin
@AnnWithArrayMethod(names = arrayOf("abc", "foo", "bar")) class C
```

애노테이션 인스턴스의 값은 코틀린 코드에 프로퍼티로 노출된다:

``` java
// Java
public @interface Ann {
    int value();
}
```

``` kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```
