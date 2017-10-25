---
type: doc
layout: reference
category: "Syntax"
title: "위임 프로퍼티"
---

# 위임 프로퍼티

필요할 때마다 수동으로 구현할 수 있지만 한번만 구현하고 라이브러리에 추가하면 좋을만한
공통 프로퍼티가 존재한다. 다음은 예이다.

* lazy 프로퍼티: 처음 접근할 때 값을 계산한다.
* observable 프로퍼티: 프로퍼티가 바뀔 때 리스너에 통지한다.
* 맵에 저장할 프로퍼티: 각 프로퍼티를 별도 필드에 저장하는 대신 맵에 저장

이런 (그리고 다른) 경우를 다를 수 있도록 코틀린은 _위임 프로퍼티_를 지원한다:

``` kotlin
class Example {
    var p: String by Delegate()
}
```

구문은 `val/var <property name>: <Type> by <expression>`이다. 
*by*{:.keyword} 뒤의 식이 _대리객체(delegate)_로 프로퍼티에 대응하는 `get()`(그리고 `set()`)은
대리객체의 `getValue()`와 `setValue()` 메서드에 위임한다.

프로퍼티 대리객체는 인터페이스를 구현하면 안 되며,
`getValue()` 함수를 제공해야 한다(그리고 *var*{:.keyword} 프로퍼티의 경우 `setValue()`도 제공).
다음은 예이다:

``` kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name} in $thisRef.'")
    }
}
```

위 예제의 `p` 프로퍼티를 읽을 때, 위임한 `Delegate` 인스턴스의 `getValue()` 함수를 호출한다.    
`getValue()`의 첫 번째 파라미터는 `p`를 포함한 객체이고,
두 번째 파라미터는 `p` 자체에 대한 설명을 포함한다(예를 들어, 이름을 포함한다).
다음 예를 보자.

``` kotlin
val e = Example()
println(e.p)
```

이 코드는 다음을 출력한다:

``` kotlin
Example@33a17727, thank you for delegating ‘p’ to me!
```

비슷하게 `p`에 할당할 때 `setValue()` 함수를 호출한다.
첫 번째와 두 번째 파라미터는 동일하고, 세 번째 파라미터는 할당할 값이다:

``` kotlin
e.p = "NEW"
```

이 코드는 다음을 출력한다.
 
``` kotlin
NEW has been assigned to ‘p’ in Example@33a17727.
```

대리 객체에 대한 요구사항 명세는 
[아래](delegated-properties.html#property-delegate-requirements)에서 볼 수 있다.

코틀린 1.1부터 함수나 코드 블록에 위임 프로퍼티를 선언할 수 있으므로,
반드시 클래스의 멤버로 선언할 필요는 없다.
아래에서 [예제](delegated-properties.html#local-delegated-properties-since-11)를 볼 수 있다.

## 표준 위임

코틀린 표준 라이브러리는 여러 유용한 위임을 위한 팩토리 메서드를 제공한다.

### Lazy

[`lazy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html)는
람다를 파라미터로 받고 `Lazy<T>` 인스턴스를 리턴하는 함수이다.
이는 lazy 프로퍼티를 구현하기 위한 대리객체로 동작한다.
이 객체는 `get()`을 처음 호출할 때 `lazy()`에 전달한 람다를 실행하고 그 결과를 기억한다.
이어진 `get()` 호출은 단순히 기억한 결과를 리턴한다.

``` kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue)
    println(lazyValue)
}
```

이 예는 다음을 출력한다:

```
computed!
Hello
Hello
```

기본적으로 lazy 프로퍼티의 평가를 **동기화**한다.
한 쓰레드에서만 값을 계산하고, 모든 쓰레드는 같은 값을 사용한다.
위임 초기화에 동기화가 필요없으면  
`lazy()` 함수에 파라미터로 `LazyThreadSafetyMode.PUBLICATION`를 전달해서
동시에 여러 쓰레드가 초기화를 실행할 수 있게 허용할 수 있다.
단일 쓰레드가 초기화를 할 거라 확신할 수 없다면 `LazyThreadSafetyMode.NONE` 모드를 사용한다.
이는 쓰레드 안정성을 보장하지 않으며 관련 부하를 발생하지 않는다.


### Observable

[`Delegates.observable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html)은 두 개의 인자를 갖는다. 
첫 번째는 초기 값이고 두 번째는 수정에 대한 핸들러이다.
프로퍼티에 값을 할당할 때마다 (할당이 끝난 _이후에_) 핸들러를 호출한다.
핸들러는 할당된 프로퍼티, 이전 값, 새 값의 세 파라미터를 갖는다:

``` kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main(args: Array<String>) {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

이 예는 다음을 출력한다:

```
<no name> -> first
first -> second
```

만약 할당을 가로채서 그것을 "거부"하고 싶다면, `observable()` 대신
[`vetoable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html)을 사용한다.
프로퍼티에 새 값을 할당하기 _전에_ `vetoable`에 전달한 핸들러를 호출한다.

## 맵에 프로퍼티 저장하기

한 가지 공통된 용도는 맵에 프로퍼티 값을 저장하는 것이다.
JSON을 파싱하거나 다른 "동적인" 것을 하는 어플리케이션에 주로 사용한다.
이 경우 위임 프로퍼티를 위한 대리 객체로 맵 인스턴스 자체를 사용할 수 있다.

``` kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

이 예제에서 생성자는 맵을 받는다:

``` kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

위임 프로퍼티는 이 맵에서 값을 구한다(프로퍼티의 이름을 키로 사용):


``` kotlin
println(user.name) // "John Doe"를 출력
println(user.age)  // 25를 출력
```

읽기 전용 `Map` 대신에 `MutableMap`을 사용하면 *var*{:.keyword} 프로퍼티에 동작한다:

``` kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```

## 로컬 위임 프로퍼티 (1.1부터 지원)

로컬 변수를 위임 프로퍼티로 선언할 수 있다. 예를 들어, 로컬 변수를 lazy로 만들 수 있다:

``` kotlin
fun example(computeFoo: () -> Foo) {
    val memoizedFoo by lazy(computeFoo)

    if (someCondition && memoizedFoo.isValid()) {
        memoizedFoo.doSomething()
    }
}
```

`memoizedFoo` 변수에 처음 접근할 때만 계산을 한다. 만약 `someCondition`이 false면 `memoizedFoo` 변수를 계산하지 않는다.

## 프로퍼티 위임 요구사항

아래 대리객체에 대한 요구사항을 요약했다.

**읽기 전용** 프로퍼티의 경우(예, *val*{:.keyword}), 위임 객체는 이름이 `getValue`이고 다음 파라미터를 갖는 함수를 제공해야 한다:

* `thisRef` --- _프로퍼티 소유자_와 같거나 또는 상위 타입이어야 한다(확장 프로퍼티의 경우 --- 확장한 타입).
* `property` --- `KProperty<*>` 타입 또는 그 상위타입이어야 한다.
 
 이 함수는 프로퍼티와 같은 타입(또는 하위 타입)을 리턴해야 한다.
 
**변경 가능** 프로퍼티의 경우 (a *var*{:.keyword}), 위임 객체는 _추가로_ 이름이 `setValue`이고 다음 파라미터를 갖는 함수를 제공해야 한다:
 
* `thisRef` --- `getValue()`와 동일
* `property` --- `getValue()`와 동일
* 새 값 --- 프로퍼티와 같은 타입 또는 상위타입이어야 한다.
 
`getValue()`와 `setValue()` 함수는 위임 클래스의 멤버 함수나 확장 함수로도 제공할 수 있다.
원래 이 함수를 제공하지 않는 객체에 위임 프로퍼티가 필요한 경우 확장 함수가 편하다. 
두 함수 모두 `operator` 키워드를 붙여야 한다.

위임 클래스는 요구한 `operator` 메서드를 포함하고 있는 `ReadOnlyProperty`와 `ReadWriteProperty` 인터페이스 중 하나를 구현할 수 있다.
이 두 인터페이스는 코틀린 표준 라이브러리에 선언되어 있다:

``` kotlin
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}
```

### 변환 규칙

모든 위임 프로퍼티에 대해, 코틀린 컴파일러는 보조 프로퍼티를 생성하고 그 프로퍼티에 위임한다.
예를 들어, `prop` 프로퍼티에 대해 `prop$delegate`이라는 숨겨진 프로퍼티를 생성하고, 접근자 코드는 단순히 이 프로젝트에 위임한다:

``` kotlin
class C {
    var prop: Type by MyDelegate()
}

// 이 코드는 컴파일러가 대신 생성한다
class C {
    private val prop$delegate = MyDelegate()
    var prop: Type
        get() = prop$delegate.getValue(this, this::prop)
        set(value: Type) = prop$delegate.setValue(this, this::prop, value)
}
```

코틀린 컴파일러가 `prop`에 대한 모든 필요한 정보를 인자로 제공한다.
첫 번째 인자 `this`는 외부 클래스인 `C`의 인스턴스를 참조하고,
`this::prop`는 `prop` 자체를 설명하는 `KProperty` 타입의 리플레션 객체이다.

코드에서 직접 [바운드 callable 레퍼런스](reflection.html#bound-function-and-property-references-since-11)를
참조하는 `this::prop` 구문은 코틀린 1.1부터 사용 가능하다.  

### 위임객체 제공 (1.1부터)

`provideDelegate` 연산자를 정의하면, 위임 프로퍼티가 위임할 객체를 생성하는 로직을 확장할 수 있다.
`by`의 오른쪽에서 사용할 객체가 `provideDelegate`를 멤버 함수나 확장 함수로 정의하면,
프로퍼티의 위임 대상 인스턴스를 생성할 때 그 함수를 호출한다.

`provideDelegate`의 가능한 한 가지 용도는 프로퍼티의 getter나 setter뿐만 아니라 생성할 때 프로퍼티의 일관성을 검사하는 것이다.

예를 들어, 값을 연결하기 전에 프로퍼티 이름을 검사하고 싶다면, 다음 코드로 이를 처리할 수 있다:

``` kotlin
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(
            thisRef: MyUI,
            prop: KProperty<*>
    ): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, prop.name)
        // 대리 객체 생성
    }

    private fun checkProperty(thisRef: MyUI, name: String) { ... }
}

fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { ... }

class MyUI {
    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```

`provideDelegate` 파라미터는 `getValue`와 동일하다:

* `thisRef` --- _프로퍼티 소유자_와 같거나 또는 상위 타입이어야 한다(확장 프로퍼티의 경우 --- 확장한 타입).
* `property` --- `KProperty<*>` 타입 또는 그 상위타입이어야 한다.

`MyUI` 인스턴스를 생성하는 동안 각 프로퍼티에 대해 `provideDelegate` 메서드를 호출하고
즉시 필요한 검증을 수행한다.

프로퍼티와 대리객체 사이의 연결을 가로채는 능력이 없는데, 같은 기능을 구현하려면
편한 방법은 아니지만 명시적으로 프로퍼티 이름을 전달해야 한다.

``` kotlin
// "provideDelegate" 기능 없이 프로퍼티 이름을 검사
class MyUI {
    val image by bindResource(ResourceID.image_id, "image")
    val text by bindResource(ResourceID.text_id, "text")
}

fun <T> MyUI.bindResource(
        id: ResourceID<T>,
        propertyName: String
): ReadOnlyProperty<MyUI, T> {
   checkProperty(this, propertyName)
   // 대리객체를 생성한다
}
```

생성한 코드는 보조 프로퍼티인 `prop$delegate`를 초기화하기 위해 `provideDelegate`를 호출한다.
`val prop: Type by MyDelegate()` 프로퍼티 선언을 위해 생성한 코드와 
[위에서](delegated-properties.html#translation-rules) 생성한 코드(`provideDelegate` 메서드가 없는 경우)를 비교하자.

``` kotlin
class C {
    var prop: Type by MyDelegate()
}

// 'provideDelegate'가 사용가능할 때 
// 컴파일러가 이 코드를 생성한다:
class C {
    // 추가 "delegate" 프로퍼티를 생성하기 위해 "provideDelegate"를 호출
    private val prop$delegate = MyDelegate().provideDelegate(this, this::prop)
    val prop: Type
        get() = prop$delegate.getValue(this, this::prop)
}
```

`provideDelegate` 메서드는 보조 프로퍼티의 생성에만 영향을 주고, getter나 setter를 위한 코드 생성에는 영향을 주지 않는다. 
