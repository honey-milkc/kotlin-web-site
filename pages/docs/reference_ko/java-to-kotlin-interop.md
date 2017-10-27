---
type: doc
layout: reference
category: "Java Interop"
title: "자바에서 코틀린 호출하기"
---

# 자바에서 코틀린 호출하기

자바에서 쉽게 코틀린 코드를 호출할 수 있다.

## 프로퍼티

코틀린 프로퍼티는 다음 자바 요소로 컴파일된다:

 * `get` 접두사가 앞에 붙은 계산된 이름을 가진 getter 메서드
 * `set` 접두사가 앞에 붙은 계산된 이름을 가진 setter 메서드 (`var` 프로퍼티에만 적용)
 * 프로퍼티 이름과 같은 이름을 가진 private 필드 (지원 필드를 가진 프로퍼티에만 적용)

예를 들어, `var firstName: String` 코드는 다음의 자바 선언으로 컴파일된다:

``` java
private String firstName;

public String getFirstName() {
    return firstName;
}

public void setFirstName(String firstName) {
    this.firstName = firstName;
}
```

프로퍼티 이름이 `is`로 시작하면, 다른 이름 매핑 규칙을 사용한다. getter 이름으로 프로퍼티 이름을 그대로 사용하고,
setter 이름은 `is` 대신에 `set`을 사용한다.
예를 들어, `isOpen` 프로퍼티의 경우 getter는 `isOpen`이고 setter는 `setOpen()`이 된다.
이 규칙은 `Boolean` 뿐만 아니라 모든 타입의 프로퍼티에 적용된다.

## 패키지 수준 함수

`example.kt` 파일의 `org.foo.bar` 안에 선언한 모든 함수(확장 함수 포함)와 프로퍼티는
`org.foo.bar.ExampleKt`라 불리는 자바 클래스의 정적 메서드로 컴파일된다.

``` kotlin
// example.kt
package demo

class Foo

fun bar() {
}

```

``` java
// 자바
new demo.Foo();
demo.ExampleKt.bar();
```

`@JvmName` 애노테이션을 사용해서 생성할 자바 클래스 이름을 바꿀 수 있다:

``` kotlin
@file:JvmName("DemoUtils")

package demo

class Foo

fun bar() {
}

```

``` java
// 자바
new demo.Foo();
demo.DemoUtils.bar();
```

같은 자바 클래스 이름을 생성하는 파일이 여러 개 존재하면(같은 패키지와 같은 이름 또는 같은 @JvmName 애노테이션) 
보통 에러이다.
하지만, 컴파일러는 이를 위한 파사드 클래스를 만드는 기능을 제공한다.
이 파사드 클래스는 같은 이름을 가진 모든 파일의 모든 선언을 포함하며, 지정한 이름을 갖는다.
이 파사드 클래스를 생성하려면 모든 파일에 @JvmMultifileClass 애노테이션을 사용하면 된다.

``` kotlin
// oldutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun foo() {
}
```

``` kotlin
// newutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun bar() {
}
```

``` java
// 자바
demo.Utils.foo();
demo.Utils.bar();
```

## 인스턴스 필드

코틀린 프로퍼티를 자바 필드로 노출하고 싶다면, 프로퍼티에 `@JvmField` 애노테이션을 붙이면 된다.
이 필드는 해당 프로퍼티와 동일한 가시성을 갖는다. 프로퍼티가 지원 필드를 갖고, private이 아니고,
`open`이나 `override`나 `const` 수식어를 갖지 않고, 위임 프로퍼티가 아니면,
`@JvmField` 애노테이션을 붙일 수 없다.

``` kotlin
class C(id: String) {
    @JvmField val ID = id
}
```

``` java
// 자바
class JavaClient {
    public String getID(C c) {
        return c.ID;
    }
}
```

[초기화 지연](properties.html#late-initialized-properties) 프로퍼티 또한 필드로 노출할 수 있다.
필드는 `lateinit` 프로퍼티의 setter와 동일한 가시성을 갖는다.

## 정적 필드

이름 가진 오브젝트나 컴페니언 오브젝트에 선언한 코틀린 프로퍼티는 이름 가진 오브젝트나 컴페니언 오브젝트를 포함하는 클래스에
정적 지원 필드를 갖는다.

보통 이 필드는 private이지만 다음 중 하나로 노출할 수 있다:

 - `@JvmField` 애노테이션
 - `lateinit` 수식어
 - `const` 수식어

이 프로퍼티에 `@JvmField`를 붙이면, 프로퍼티와 같은 가시성을 가진 정적 필드로 만든다.

``` kotlin
class Key(val value: Int) {
    companion object {
        @JvmField
        val COMPARATOR: Comparator<Key> = compareBy<Key> { it.value }
    }
}
```

``` java
// 자바
Key.COMPARATOR.compare(key1, key2);
// Key 클래스에 public static final 필드
```

객체나 컴페니언 오브젝트의 [초기화 지연](properties.html#late-initialized-properties) 프로퍼티는
프로퍼티 setter와 같은 가시성을 갖는 정적 지원 필드를 갖는다.

``` kotlin
object Singleton {
    lateinit var provider: Provider
}
```

``` java
// 자바
Singleton.provider = new Provider();
// Singleton 클래스에 public static이고 final이 아닌 필드
```

(클래스나 최상위 수준에 선언한) `const` 프로퍼티는 자바의 정적 필드로 바뀐다:

``` kotlin
// file example.kt

object Obj {
    const val CONST = 1
}

class C {
    companion object {
        const val VERSION = 9
    }
}

const val MAX = 239
```

자바:

``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
int v = C.VERSION;
```

## 정적 메서드

위에서 말한 것처럼, 코틀린은 패키지 수준 함수를 정적 메서드로 표현한다.
코틀린은 또한 이름가진 객체나 컴페니언 오브젝트에 정의한 함수에 `@JvmStatic`을 붙이면 
정적 메서드를 생성한다.
이 애노테이션을 사용하면 컴파일러는 객체를 둘러싼 클래스에 정적 메서드를 생성하고 객체 자체에 인스턴스 메서드를 생성한다.
다음은 예이다:

``` kotlin
class C {
    companion object {
        @JvmStatic fun foo() {}
        fun bar() {}
    }
}
```

이제 `foo()`는 자바에서 정적 메서드이고, `bar()`는 아니다:

``` java
C.foo(); // 동작
C.bar(); // 에러: 정적 메서드 아님
C.Companion.foo(); // 인스턴스 메서드 유지
C.Companion.bar(); // 오직 이렇게만 동작
```

이름 가진 오브젝트도 동일하다:

``` kotlin
object Obj {
    @JvmStatic fun foo() {}
    fun bar() {}
}
```

자바:

``` java
Obj.foo(); //동작
Obj.bar(); // 에러
Obj.INSTANCE.bar(); // 동작, 싱글톤 인스턴스를 통해 호출
Obj.INSTANCE.foo(); // 역시 동작
```

`@JvmStatic` 애노테이션을 오브젝트나 컴페니언 오브젝트의 프로퍼티에도 적용할 수 있다.
이는 getter와 setter 메서드를 오브젝트나 컴페니언 오브젝트를 감싼 클래스의 정적 멤버로 만든다.

## 가시성

코틀린 가시성은 다음 규칙에 따라 자바로 매핑된다:

* `private` 멤버는 `private` 멤버로 컴파일
* `private` 최상위 선언은 패키지-로컬 선언으로 컴파일
* `protected`는 `protected`로 유지 (자바는 같은 패키지의 다른 클래스에서 protected 멤버에 접근하는 것을 허용하나 코틀린은 그렇지 않다. 따라서, 자바 클래스가 코드에 대한 더 넓은 접근을 갖는다.) 
* `internal` 선언은 자바에서 `public`이 됨. `internal` 클래스의 이름을 변형해서 자바에서 우연히 사용하는 것을 어렵게 하고
 코틀린 규칙에 따라 서로 볼 수 없는 동일 시그너처를 가진 멤버를 오버로딩할 수 있게 한다.
* `public`은 `public`으로 남음

## KClass

`KClass` 타입의 파라미터를 갖는 코틀린 메서드를 호출해야 할 때가 있다.
`Class`에서 `KClass`로의 자동 변환은 없기에 `Class<T>.kotlin` 확장 프로퍼티와 동일한 메서드를 호출해서 
수동으로 구해야 한다.  

```kotlin
kotlin.jvm.JvmClassMappingKt.getKotlinClass(MainView.class)
```

## @JvmName으로 시그너처 깨짐 처리

코틀린에서 이름 가진 함수가 바이트코드에서 다른 JVM 이름을 갖도록 해야 할 때가 있다.
가장 두드러진 예는 *타입 제거*때문에 발생한다:

``` kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```

두 함수를 함께 정의할 수 없는데, 그 이유는 JVM 시그너처가 `filterValid(Ljava/util/List;)Ljava/util/List;`로 같기 때문이다.
정말로 코틀린에서 같은 이름을 가진 함수가 필요하다면 한 함수(또는 두 함수)에 
다른 이름을 인자로 갖는 `@JvmName`을 붙이면 된다:

``` kotlin
fun List<String>.filterValid(): List<String>

@JvmName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```

위 예에서 코틀린은 `filterValid` 이름으로 두 함수를 접근할 수 있지만, 자바에서는
`filterValid`와 `filterValidInt` 이름을 사용한다.

같은 트릭을 `getX()` 함수와 `x` 프로퍼티를 함께 사용해야 할 때 적용할 수 있다:

``` kotlin
val x: Int
    @JvmName("getX_prop")
    get() = 15

fun getX() = 10
```


## 오버로드 생성

보통 디폴트 파라미터 값을 갖는 코틀린 함수를 작성하면, 자바에서는 모든 파라미터가 필요한 전체 시그너처로 보인다.
자바에 여러 오버로드 메서드를 노출하고 싶다면 `@JvmOverloads` 애노테이션을 사용하면 된다.

애노테이션은 생성자와 정적 메서드 등에도 동작한다. 인터페이스에 정의한 메서드를 포함한 추상 메서드에는 사용할 수 없다.

``` kotlin
class Foo @JvmOverloads constructor(x: Int, y: Double = 0.0) {
    @JvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
        ...
    }
}
```

기본 값을 가진 모든 파라미터에 대해, 추가로 한 개의 오버로드 메서드를 생성한다. 추가 메서드는 파라미터 목록에서 기본 값을 가진 파라미터와 
그 파라미터 목록의 오른쪽에 있는 모든 파라미터를 제거한 나머지 파라미터를 갖는다.
위 예의 경우 다음을 생성한다:

``` java
// 생성자:
Foo(int x, double y)
Foo(int x)

// 메서드
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```

[보조 생성자](classes.html#secondary-constructors)에서 설명한 것처럼, 클래스가 모든 생서자 파라미터에 대해
기본 값을 가지면, 인자 없는 public 생성자를 생성한다. 이는 `@JvmOverloads`을 지정하지 않아도 동작한다.

## Checked 익셉션

앞서 언급한 것처럼, 코틀린에는 checked 익셉션이 없다.
그래서 보통 코틀린 함수의 자바 시그너처는 발생할 익셉션을 선언하지 않는다.
아래와 같은 코틀린 함수가 있다고 하자.

``` kotlin
// example.kt
package demo

fun foo() {
    throw IOException()
}
```

다음과 같이 자바 코드에서 이 함수를 호출하고 익셉션을 catch 하면,

``` java
// Java
try {
  demo.Example.foo();
}
catch (IOException e) { // 에러: foo()는 throws 목록에 IOException을 선언하지 않았음
  // ...
}
```

자바 컴파일러가 에러를 발생한다. 왜냐면 `foo()`가 `IOException`을 선언하지 않았기 때문이다.
이 문제를 해결하려면 코틀린에 `@Throws` 애노테이션을 사용한다:

``` kotlin
@Throws(IOException::class)
fun foo() {
    throw IOException()
}
```

## Null-안정성

자바에서 코틀린 함수를 호출할 때 non-null 파라미터에 *null*{: .keyword }을 전달하는 것을 막을 수 없다.
이것이 코틀린이 non-null을 기대하는 모든 public 함수에 대해 런타임 검사를 생성하는 이유이다.
이 방법은 자바 코드에서 즉시 `NullPointerException`을 얻는다.

## 가변 지네릭

코틀린 클래스에서 [선언 위치 가변성](generics.html#declaration-site-variance)을 사용할 때,
자바 코드에 어떻게 보여줄지에 대한 두 가지 선택이 있다. 선언 위치 변성을 사용하는 클래스와 두 함수 예를 보자:

``` kotlin
class Box<out T>(val value: T)

interface Base
class Derived : Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```

이 함수를 자바 코드로 해석하는 순진한 방법은 다음과 같이 하는 것이다:
 
``` java
Box<Derived> boxDerived(Derived value) { ... }
Base unboxBase(Box<Base> box) { ... }
``` 

문제는 코틀린은 `unboxBase(boxDerived("s"))`가 가능하지만, 자바는 불가능하다는 것이다. 왜냐면, 자바에서
`Box` 클래스는 `T` 파라미터에 대해 *무공변*이며, 그래서 `Box<Derived>`는 `Box<Base>`의 하위타입이 아니기 때문이다.
자바에서 이것을 사용하려면 `unboxBase`를 다음과 같이 정의해야 한다:
  
``` java
Base unboxBase(Box<? extends Base> box) { ... }  
```  

여기서 사용 위치 변성으로 선언 위치 변성을 흉내내기 위해 *와일드카드 타입*(`? extends Base`)을 사용했다. 자바는 사용 위치 변성만 있기 때문이다.

코틀린 API가 자바에서 동작할 수 있도록 타입 파라미터가 *파라미터로 쓰일* 때 
공변으로 정의한 `Box`에 대한 `Box<Super>`와 `Box<? extends Super>`를 생성한다(또는 반공변으로 정의한 `Foo`에 대한 `Foo<? super Bar>`를 생성).
파라미터가 리턴 값일 때, 와일카드를 생성하지 않는다. 왜냐면, 자바 클라이언트가 이를 처리할 수 있기 때문이다(그리고, 리턴 타입에서 와일드카드는
보통의 자바 코딩 스타일에 반한다). 따라서 예제의 함수를 실제로 다음과 같이 해석한다:
  
``` java
// 리턴 타입 - 와일드카드 없음
Box<Derived> boxDerived(Derived value) { ... }
 
// 파라미터 - 와일드카드 
Base unboxBase(Box<? extends Base> box) { ... }
```

주의: 인자 타입이 final이면 와일드카드를 생성할 위치가 없고, 따라서 `Box<String>`은 그것이 취하는 위치가 무엇이냐에 상관없이
항상 `Box<String>`이다. 

기본 생성되지 않는 타입에 대해 와일드카드가 필요하면 `@JvmWildcard` 주석을 사용한다:

``` kotlin
fun boxDerived(value: Derived): Box<@JvmWildcard Derived> = Box(value)
// 다음으로 해석 
// Box<? extends Derived> boxDerived(Derived value) { ... }
```

반면에 와일드카드를 생성하지 않기를 원하면 `@JvmSuppressWildcards`를 사용한다:

``` kotlin
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
// 다음으로 해석 
// Base unboxBase(Box<Base> box) { ... }
```

주의: `@JvmSuppressWildcards`는 개별 타입 인자에 적용할 수 있을 뿐만 아니라 함수나 클래스에 적용할 수 있다.
클래스나 함수에 적용하면 그 안에 있는 모든 와일드카드를 생성하지 않는다.

### Nothing 타입의 해석
 
[`Nothing`](exceptions.html#the-nothing-type) 타입은 자바에 알맞은 비슷한 것이 없다. 사실 `java.lang.Void`를 포함한
모든 자바 레퍼런스 타입은 값으로 `null`을 가질 수 있는데 `Nothing`은 그것 조차 안 된다.
따라서, 이 타입을 자바에서 정확하게 표현할 수 없다.
이것이 코틀린이 `Nothing` 타입의 인자를 사용할 때 raw 타입을 생성하는 이유이다.

``` kotlin
fun emptyList(): List<Nothing> = listOf()
// 다음으로 해석
// List emptyList() { ... }
```
