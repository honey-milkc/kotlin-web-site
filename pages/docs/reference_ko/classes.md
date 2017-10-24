---
type: doc
layout: reference
category: "Syntax"
title: "클래스와 상속"
related:
    - functions.md
    - nested-classes.md
    - interfaces.md
---

# 클래스와 상속

## 클래스

코틀린에서 클래스는 *class*{: .keyword } 키워드를 사용해서 선언한다:

``` kotlin
class Invoice {
}
```

클래스 선언은 클래스 이름과 클래스 헤더(타입 파라미터를 지정, 주요 생성자 등), 중괄호로 둘러 싼 클래스 몸체로 구성된다.
헤더와 몸체는 선택적이다. 클래스에 몸체가 없으면 중괄호를 생략할 수 있다.

``` kotlin
class Empty
```


### 생성자

코틀린 클래스는 **주요 생성자**와 한 개 이상의 **보조 생성자*를 가질 수 있다.
주요 생성자는 클래스 헤더의 한 부분으로 클래스 이름(그리고 필요에 따라 타입 파라미터) 뒤에 위치한다.

``` kotlin
class Person constructor(firstName: String) {
}
```

주요 생성자에 애노테이션이나 가시성 수식어가 없으면, *constructor*{: .keyword } 키워드를 생략할 수 있다:

``` kotlin
class Person(firstName: String) {
}
```

주요 생성자는 어떤 코드도 포함할 수 없다. **초기화 블록** 블록에 초기화 코드가 위치할 수 있다.
초기화 블록에는 *init*{: .keyword } 키워드를 접두사로 붙인다:    

``` kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```

초기화 블록에서 주요 생성자의 파라미터를 사용할 수 있다.
주요 생성자 파라미터는 클래스 몸체에 선언한 프로퍼티 초기화에서도 사용할 수 있다:

``` kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

사실 코틀린은 주요 생성자에서 프로퍼티를 선언하고 초기화할 수 있는 간결한 구문을 제공한다:
In fact, for declaring properties and initializing them from the primary constructor, 
Kotlin has a concise syntax:


``` kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ...
}
```

일반 프로퍼티와 마찬가지로 주요 생성자에 선언한 프로퍼티는 수정가능(*var*{: .keyword })이거나 읽기 전용(*val*{: .keyword })일 수 있다.

생성자가 애노테이션이나 가시성 수식어를 가지면 *constructor*{: .keyword } 키워드가 필요하며,
키워드 앞에 수식어가 온다:

``` kotlin
class Customer public @Inject constructor(name: String) { ... }
```

자세한 내용은 [가시성 수식어](visibility-modifiers.html#constructors)를 참고한다.


#### 보조 생성자

클래스를 **secondary constructors** 생성자를 선언할 수 있다. 보조 생성자는 *constructor*{: .keyword }를 접두사로 붙인다:

``` kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

클래스에 주요 생성자가 있다면, 각 보조 생성자는 직접적으로 또는 다른 보조 생성자를 통해 간접적으로 주요 생성자를 호출해야 한다.
같은 클래스의 다른 생성자를 호출할 때에는 *this*{: .keyword } 키워드를 사용한다:

``` kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

비추상 클래스에 어떤 생성자(주요 또는 보조)도 없으면, 인자가 없는 주요 생성자를 생성한다.
이 생성자의 가시성은 public이다. public 생성자를 원하지 않으면 기본 가시성이 아닌 빈 주요 생성자를 선언해야 한다. 

``` kotlin
class DontCreateMe private constructor () {
}
```

> **주의**: JVM에서 주요 생성자의 모든 파라미터가 기본 값을 가지면, 컴파일러는 추가로 파라미터가 없는 생성자를 생성한다. 이 생성자는 기본 값을 사용하는 주요 생성자를 사용한다.
> 이는 Jackson이나 JPA처럼 파라미터없는 생성자를 사용해서 인스턴스를 생성하는 라이브러리에서 코틀린을 쉽게 사용할 수 있게 해준다.
>
> ``` kotlin
> class Customer(val customerName: String = "")
> ```
{:.info}

### 클래스의 인스턴스 생성

클래스의 인스턴스를 생성하려면 일반 함수와 같이 생성자를 호출한다:

``` kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

코틀린에는 *new*{: .keyword } 키워드가 없음에 유의하자.

중첩된, 내부 또는 임의 내부 클래스의 생성은 [중첩](nested-classes.html)를 참고한다.

### 클래스 멤버

클래스는 다음을 가질 수 있다:

* [생성자와 초기화 블록](classes.html#constructors)
* [함수](functions.html)
* [프로퍼티](properties.html)
* [중첩 클래스와 내부 클래스](nested-classes.html)
* [오브젝트 선언](object-declarations.html)


## 상속

코틀린의 모든 클래스는 공통의 최상위 클래스 `Any`를 상속한다. 상위 타입을 선언하지 않으면 `Any`가 기본 상위 타입이다:

``` kotlin
class Example // 기본으로 Any를 상속한다
```

`Any`는 `java.lang.Object`가 아니다.
특히 `Any`는 `equals()`, `hashCode()`, `toString()` 외에 다른 멤버를 갖지 않는다.
더 자세한 내용은 [자바 상호운용성](java-interop.html#object-methods)을 참고한다.

상위타입을 직접 선언하러면 클래스 헤더에 콜론 뒤에 타입을 위치시킨다:

``` kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

클래스에 주요 생성자가 있으면, 주요 생성자의 파라미터를 사용해서 기반 타입을 그 자리에서 초기화할 수 있다. 

클래스에 주요 생성자가 없으면, 각 보조 생성자에서 *super*{: .keyword } 키워드로 기반 타입을 초기화하거나
그걸 하는 다른 생성자를 호출해야 한다.

``` kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx)

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs)
}
```

*open*{: .keyword } 애노테이션은 자바의 *final*{: .keyword }과 반대이다. 
*open*{: .keyword }은 다른 클래스가 이 클래스를 상속할 수 있게 허용한다.
기본으로 코틀린의 모든 클래스는 final인데 이는 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html)의
Item 17: *Design and document for inheritance or else prohibit it*를 따른 것이다.

### 메서드 오버라이딩

코틀린에서 메서드 오버라이딩을 명시적으로 해야 한다. 자바와 달리 코틀린에서는 오버라이딩 가능한 멤버에 
(*open*) 애노테이션을 명시적으로 설정해야 한다:


``` kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}
class Derived() : Base() {
    override fun v() {}
}
```

`Derived.v()`에는 *override*{: .keyword } 애노테이션이 필요하다. 이를 빼먹으면 컴파일러에 실패한다.
`Base.nv()`처럼 *open*{: .keyword } 애노테이션이 없는 경우,
*override*{: .keyword }를 사용하든 안 하든 하위클래스에서 동일 시그너처를 갖는 메서드를 선언할 수 없다.
final 클래스(예, *open*{: .keyword } 애노테이션이 없는 클래스)는 open 멤버를 가질 수 없다.

*override*{: .keyword }가 붙은 멤버는 그 자체가 open 이며,
하위 클래스에서 다시 오버라이딩하는 것을 막고 싶다면 *final*{: .keyword }을 사용해야 한다:

``` kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

### 프로퍼티 오버라이딩 

프로퍼티 오버라이딩은 메서드 오버라이딩과 유사하게 동작한다.
상위클래스에 선언된 프로퍼티를 하위 클래스에 재선언하려면
*override*{: .keyword }를 사용해야 하며 호환되는 타입을 사용해야 한다.
선언한 프로퍼티는 initializer를 가진 프로퍼티나 getter 메서드를 가진 프로퍼티로 오버라이딩할 수 있다.


``` kotlin
open class Foo {
    open val x: Int get() { ... }
}

class Bar1 : Foo() {
    override val x: Int = ...
}
```

또한 `val` 프로퍼티를 `var` 프로퍼티로 재정의할 수 있지만 반대는 안 된다.
이것이 허용되는 이유는 `var` 프로퍼티는 근본적으로 getter 메서드를 선언하는데,
그것을 `var`로 오버라이딩하면 추가로 하위 클래스에서 setter 메서드를 선언하기 때문이다.

주요 생성자에 선언한 프로퍼티에도 *override*{: .keyword } 키워드를 사용할 수 있다.

``` kotlin 
interface Foo {
    val count: Int
}

class Bar1(override val count: Int) : Foo

class Bar2 : Foo {
    override var count: Int = 0
}
```

### 상위클래스 구현 호출

하위클래스는 *super*{: .keyword } 키워드를 사용해서 상위클래스의 함수와 프로퍼티 접근 구현를 호출할 수 있다:

```kotlin
open class Foo {
    open fun f() { println("Foo.f()") }
    open val x: Int get() = 1
}

class Bar : Foo() {
    override fun f() { 
        super.f()
        println("Bar.f()") 
    }
    
    override val x: Int get() = super.x + 1
}
```

내부 클래스는 `super@Outer`와 같이 외부 클래스의 이름을 사용해서 외부 클래스의 상위 클래스에 접근할 수 있다:

```kotlin
class Bar : Foo() {
    override fun f() { /* ... */ }
    override val x: Int get() = 0
    
    inner class Baz {
        fun g() {
            super@Bar.f() // Foo의 f()의 구현
            println(super@Bar.x) // Foo의 x의 getter 사용
        }
    }
}
```

### 규책 오버라이딩

코틀린은 구현 상속은 다음 규칙에 따라 제한된다:
클래스가 바로 위의 상위 클래스에서 같은 멤버의 구현을 여러개 상속받으면,
반드시 멤버를 오버라이딩하고 자신의 구현을 제공해야 한다(아마도, 상속한 것 중 하나를 사용하는 코드).
어떤 상위타입의 구현을 사용할지 선택하려면 꺽쇠 괄호 안에 상위 타입의 이름을 붙인 *super*{: .keyword }를 사용한다.  

``` kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // 인터페이스 멤버는 기본이 'open'이다
    fun b() { print("b") }
}

class C() : A(), B {
    // 컴파일러는 f()를 오버라이딩할 것을 요구한다:
    override fun f() {
        super<A>.f() // call to A.f()
        super<B>.f() // call to B.f()
    }
}
```

`A`와 `B`를 둘 다 상속해도 `a()`와 `b()`는 문제가 되지 않는다. `C`가 이 함수의 구현을 한 개만 상속하기 때문이다.
하지만 `f()`의 경우 `C`는 두 개의 구현을 상속받으므로 `C`에 `f()`를 오버라이딩하고 자신의 구현을 제공해서 애매함을 없애야 한다.

## 추상 클래스

클래스나 클래스의 멤버를 *abstract*{: .keyword }로 선언할 수 있다.
추상 멤버는 그 클래스에 구현을 갖지 않는다.
추상 클래스나 함수에 open을 붙일 필요는 없다.

추상이 아닌 open 멤버를 추상 멤버로 오버라이딩할 수 있다.

``` kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

## 컴페니언 오브젝트

코틀린은 자바나 C#과 달리 클래스에 정적 메서드가 없다. 많은 경우 간단하게 패키지 수준의 함수를 대신 사용할 것을 권한다.

클래스 인스턴스없이 클래스의 내부에 접근해야 하는 함수를 작성해야 한다면(예를 들어, 팩토리 메서드),
그 클래스에 [오브젝트 선언](object-declarations.html)의 멤버로 함수를 작성할 수 있다.

더 구체적으로 [컴페니언 오브젝트](object-declarations.html#companion-objects)를 클래스 안에 선언하면,
한정자로 클래스 이름을 사용해서 자바나 C#의 정적 메서드를 호출하는 것과 동일 구문으로 컴페니언 오브젝트의 멤버를 호출할 수 있다.
