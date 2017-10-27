---
type: doc
layout: reference
category: "Syntax"
title: "리플렉션"
---

# 리플렉션

리플렉션은 란타임에 프로그램의 구조를 볼 수 있게 해주는 언어와 라이브러리 특징이다.
코틀린에서 함수와 프로퍼티는 1급이며, 
이 둘의 내부를 알아내는 것은(예, 런타임에 이름, 프로퍼티 타입이나 함수를 알아내는 것)
함수형 또는 리액티브 방식을 간단하게 사용하는 것과 밀접하게 관련있다.

> 자바에서 리플렌션 특징을 사용하는데 필요한 런타임 컴포넌트는 별도 JAR 파일(`kotlin-reflect.jar`)로 제공한다.
  이를 통해 리플렉션 특징을 사용하지 않는 어플리케이션이 필요로 하는 런타임 라이브러리의 크기를 줄였다.
  리플렉션을 사용한다면 그 jar 파일을 프로젝트의 클래스패스에 추가해야 한다.
{:.note}

## 클래스 레퍼런스

가장 기본적인 리플렉션 특징은 코틀린 클래스에 대한 런타임 레퍼런스를 구하는 것이다. 정적으로 알려진 코틀린 클래스에 대한
레퍼런스를 얻으려면 _class 리터럴_ 구문을 사용하면 된다:

``` kotlin
val c = MyClass::class
```

레퍼런스의 값은 [KClass](/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html) 타입이다.

코틀린 클래스 레퍼런스는 자바 클래스 레퍼런스와 같지 않다. 자바 클래스 레퍼런스를 구하려면
`KClass` 인스턴스의 `.java` 프로퍼티를 사용한다.

## 바운드(Bound) 클래스 레퍼런스 (1.1부터)

리시버로 객체를 사용하면 `::class` 구문을 통해 특정 객체의 클래스에 대한 레퍼런스를 구할 수 있다:

``` kotlin
val widget: Widget = ...
assert(widget is GoodWidget) { "Bad widget: ${widget::class.qualifiedName}" }
```

예를 들어, 위 코드는 리시버 식의 타입(`Widget`)에 상관없이 `GoodWidget` 또는 `BadWidget` 인스턴스의 클래스에 대한
레퍼런스를 구한다.  

## 함수 레퍼런스

다음과 같이 이름 가진 함수를 선언했다고 하자:

``` kotlin
fun isOdd(x: Int) = x % 2 != 0
```

`isOdd(5)`와 같이 쉽게 함수를 바로 호출할 수 있지만, 또한 함수를 값으로 다른 함수에 전달할 수 있다.
이를 위해  `::` 연산자를 사용한다:

``` kotlin
val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // [1, 3] 출력
```

여기서 `::isOdd`는 함수 타입 `(Int) -> Boolean`의 값이다.

`::`는 문백을 통해 원하는 타입을 알 수 있을 때, 오버로딩한 함후세어도 사용할 수 있다. 다음 예를 보자:

``` kotlin
fun isOdd(x: Int) = x % 2 != 0
fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

val numbers = listOf(1, 2, 3)
println(numbers.filter(::isOdd)) // isOdd(x: Int)를 참조
```

또는 변수에 명시적으로 타입을 지정해서 메서드 레퍼런스를 저장할 때 필요한 문맥을 제공할 수 있다:

``` kotlin
val predicate: (String) -> Boolean = ::isOdd   // isOdd(x: String)를 참고
```

클래스 멤버나 확장 함수를 사용해야 한다면 한정자를 붙여야 한다.
예를 들어, `String::toCharArray`는 `String`: `String.() -> CharArray` 타입을 위한 확장 함수를 제공한다.

### 예: 함수 조합

다음 함수를 보자:

``` kotlin
fun <A, B, C> compose(f: (B) -> C, g: (A) -> B): (A) -> C {
    return { x -> f(g(x)) }
}
```

이 함수는 파라미터로 전달한 두 함수를 조합한다(`compose(f, g) = f(g(*))`).
이제 이 함수에 호출가능한 레퍼런스를 적용할 수 있다:


``` kotlin
fun length(s: String) = s.length

val oddLength = compose(::isOdd, ::length)
val strings = listOf("a", "ab", "abc")

println(strings.filter(oddLength)) // "[a, abc]" 출력
```

## 프로퍼티 참조

코틀린에서 1급 객체로서 프로퍼티에 접근할 때에도 `::` 연산자를 사용한다:

``` kotlin
var x = 1

fun main(args: Array<String>) {
    println(::x.get()) // "1" 출력
    ::x.set(2)
    println(x)         // "2" 출력
}
```

`::x` 식은 `KProperty<Int>` 타입의 프로퍼티 객체로 평가하는데 이는 `get()`을 사용해서 프로퍼티의 값을 읽거나
`name` 프로퍼티를 사용해서 프로퍼티 이름을 가져올 수 있도록 해 준다.
더 자세한 정보는 [`KProperty` 클래스 문서](/api/latest/jvm/stdlib/kotlin.reflect/-k-property/index.html)를 참고한다.

수정 가능한 프로퍼티, 예를 들어 `var y = `에 대해 `::y`는 
`set()` 함수를 가진 [`KMutableProperty<Int>`](/api/latest/jvm/stdlib/kotlin.reflect/-k-mutable-property/index.html) 타입의 값을 리턴한다.

파라미터가 없는 함수가 필요한 곳에 프로퍼티 레퍼런스를 사용할 수 있다:
 
``` kotlin
val strs = listOf("a", "bc", "def")
println(strs.map(String::length)) // [1, 2, 3] 출력
```

클래스 멤버인 프로퍼티에 접근할 때에는 클래스로 한정한다:

``` kotlin
class A(val p: Int)

fun main(args: Array<String>) {
    val prop = A::p
    println(prop.get(A(1))) // "1" 출력
}
```

확장 프로퍼티의 경우:


``` kotlin
val String.lastChar: Char
    get() = this[length - 1]

fun main(args: Array<String>) {
    println(String::lastChar.get("abc")) // "c" 출력
}
```

### 자바 리플렉션과 상호운영성

자바 플랫폼에서, 표준 라이브러리는 자바 리플렉션 객체와의 매핑을 제공하는 리플렉션 클래스 확장을 포함한다(`kotlin.reflect.jvm` 패키지 참고).
예를 들어, Backing 필드를 찾거나 코틀린 프로퍼티에 대한 Getter로 동작하는 자바 메서드를 찾으려면,
다음과 같은 코드를 사용할 수 있다:


``` kotlin
import kotlin.reflect.jvm.*
 
class A(val p: Int)
 
fun main(args: Array<String>) {
    println(A::p.javaGetter) // "public final int A.getP()" 출력
    println(A::p.javaField)  // "private final int A.p" 출력
}
```

자바 클래스에 대응한 코틀린 클래스를 얻으려면, `.kotlin` 확장 프로퍼티를 사용한다:

``` kotlin
fun getKClass(o: Any): KClass<Any> = o.javaClass.kotlin
```

## 생성자 레퍼런스

메서드나 프로퍼티처럼 생성자도 참조할 수 있다. 생서자와 같은 파라미터를 갖고 해당 타입의 객체를 리턴하는
함수 타입 객체가 필요한 곳에 생성자 레퍼런스를 사용할 수 있다.
`::` 연산자에 클래스 이름을 붙여서 생성자 레퍼런스를 구한다.
다음 함수를 보자. 이 함수는 파라미터가 없고 리턴 타입이 `Foo`인 함수 파라미터를 갖고 있다:

``` kotlin
class Foo

fun function(factory: () -> Foo) {
    val x: Foo = factory()
}
```

Foo 클래스의 인자없는 생성자인 `::Foo`를 사용하면 다음과 같이 간단하게 생성자를 호출할 수 있다:

``` kotlin
function(::Foo)
```

## 바운드(Bound) 함수 레퍼런스와 바운드 프로퍼티 레퍼런스 (1.1부터)

특정 객체의 인스턴스 메서드를 참조할 수 있다:

``` kotlin 
val numberRegex = "\\d+".toRegex()
println(numberRegex.matches("29")) // "true" 출력
 
val isNumber = numberRegex::matches
println(isNumber("29")) // "true" 출력
```

`matches` 메서드를 직접 호출하는 대신, 그 메서드에 대한 레퍼런스를 저장하고 있다.
이 레퍼런스는 그 리서버에 묶인다.
위 예제처럼 직접 레퍼런스를 호출할 수 있고 또는 해당 함수 타입이 필요한 곳에 사용할 수 있다:

``` kotlin
val strings = listOf("abc", "124", "a70")
println(strings.filter(numberRegex::matches)) // "[124]" 출력
```

객체에 묶인 타입과 그에 대응하는 묶이지 않은 레퍼런스를 비교해보자.
묶은 호출가능한 레퍼런스는 리시버를 갖는다. 그래서 리시버 타입 파라미터가 필요없다:

``` kotlin
val isNumber: (CharSequence) -> Boolean = numberRegex::matches

val matches: (Regex, CharSequence) -> Boolean = Regex::matches
```

프로퍼티 레퍼런스도 묶을 수 있다:

``` kotlin
val prop = "abc"::length
println(prop.get())   // prints "3"
```
