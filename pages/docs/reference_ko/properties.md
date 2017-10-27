---
type: doc
layout: reference_ko
category: "Syntax"
title: "프로퍼티와 필드: Getters, Setters, const, lateinit"
---

# 프로퍼티와 필드

## 프로퍼티 선언

코틀린에서 클래스는 프로퍼티를 가질 수 있다.
*var*{: .keyword } 키워드로 변경 가능하게 선언하거나 *val*{: .keyword } 키워드로 읽기 전용으로 선언할 수 있다.

``` kotlin
class Address {
    var name: String = ...
    var street: String = ...
    var city: String = ...
    var state: String? = ...
    var zip: String = ...
}
```

자바 필드처럼 이름으로 프로퍼티를 참조할 수 있다:

``` kotlin
fun copyAddress(address: Address): Address {
    val result = Address() // 코틀린에 'new' 키워드 없음
    result.name = address.name // 접근자(accessor)를 실행
    result.street = address.street
    // ...
    return result
}
```

## Getters와 Setters
{:#getters-and-setters}

프로퍼티 선언의 전체 구문은 다음과 같다.

``` kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

initializer, getter, setter는 선택사항이다. 
프로퍼티 타입은 initializer(또는 다음에 보여줄 getter의 리턴 타입에서에서 유추할 수 있으면 생략가능하다.

예제:

``` kotlin
var allByDefault: Int? // 에러: initializer 필요, 기본 getter와 setter 적용
var initialized = 1 // Int 타입을 갖고, 기본 getter와 setter
```

읽기 전용 프로퍼티 선언은 변경가능 프로퍼티 선언과 두 가지가 다르다.
`var` 대신에 `val`로 시작하고 setter를 허용하지 않는다:

``` kotlin
val simple: Int? // Int 타입이고, 기본 getter, 생성자에서 초기화해야 함
val inferredType = 1 // Int 타입이고, 기본 getter
```

일반 함수처럼 프로퍼티 선언에 커스텀 접근자를 작성할 수 있다. 다음은 커스텀 getter의 예이다:

``` kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

커스텀 setter는 다음과 같다:

``` kotlin
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // 문자열을 파싱해서 다른 프로퍼티에 값을 할당한다 
    }
```

setter 파라미터의 이름을 `value`로 하는 것은 관례인데, 선호하는 다른 이름을 사용할 수 있다.

코틀린 1.1부터, getter에서 타입을 추론할 수 있으면 프로퍼티 타입을 생략할 수 있다:

``` kotlin
val isEmpty get() = this.size == 0  // Boolean 타입임
```

프로퍼티의 기본 구현을 바꾸지 않고 애노테이션을 붙이거나 접근자의 가시성을 바꾸고 싶다면
몸체 없는 접근자를 정의하면 된다:

``` kotlin
var setterVisibility: String = "abc"
    private set // setter를 private으로 하고 기본 구현을 가짐

var setterWithAnnotation: Any? = null
    @Inject set // setter에 @Inject 애노테이션 적용
```

{:#backing-fields}

### 지원(Backing) 필드

코틀린 클래스는 필드를 가질 수 없다. 하지만 때때로 커스텀 접근자를 사용할 때 지원(backing) 필드가 필요할 때가 있다.
이런 목적으로 코틀린은 `field` 식별자로 접근할 수 있는 지원 필드를 자동으로 제공한다:

``` kotlin
var counter = 0 // 초기값을 지원 필드에 직접 쓴다
    set(value) {
        if (value >= 0) field = value
    }
```

`field` 식별자는 오직 프로퍼티의 접근자에서만 사용할 수 있다.

접근자의 기본 구현을 적어도 한 개 이상 사용하거나 또는 커스텀 접근자에서 `field` 식별자로 지원 필드에 접근할 경우,
프로퍼티를 위한 지원 필드를 생성한다.

예를 들어 다음 경우 지원 필드가 없다:

``` kotlin
val isEmpty: Boolean
    get() = this.size == 0
```

### 지원(Backing) 프로퍼티

"자동 지원 필드" 방식이 맞지 않는 것을 해야 할 경우,
*지원(backing) 프로퍼티*로 대신할 수 있다:

``` kotlin
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // 타입 파라미터 추론
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```

모든 면에서, 이는 자바와 거의 같다. 왜냐면 기본 getter와 setter를 가진 private 프로퍼티에 접근하면
함수 호출에 따른 오버헤드가 발생하지 않도록 최적화하기 때문이다.


## 컴파일 타임 상수
{:#compile-time-constants}

컴파일 시점에 알아야 할 프로퍼티 값을 `const` 수식어를 이용해서 _컴파일 타임 상수_로 표시할 수 있다. 
그런 프로퍼티는 다음 조건을 충족해야 한다:

  * 최상위 또는 `오브젝트`의 멤버
  * `String`이나 기본 타입 값으로 초기화
  * 커스텀 getter가 아님

이런 프로퍼티는 애노테이션에서 사용할 수 있다:

``` kotlin
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"

@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```


## 초기화 지연(Late-Initialized) 프로퍼티
{:#late-initialized-properties}

보통 null이 아닌 타입으로 선언한 프로퍼티는 생성자에서 초기화해야 한다.
하지만 이게 편리하지 않을 때가 있다.
예를 들어 의존 주입이나 단위 테스트의 셋업 메서드에서 프로퍼티를 초기화한다고 하자.
이 경우 생성자에 null이 아닌 초기값을 제공할 수는 없는데, 
클래스 몸체에서 프로퍼티를 참조할 때 null 검사는 피하고 싶을 것이다.   

이런 경우를 위해 프로퍼티에 `lateinit` 수식어를 붙일 수 있다:

``` kotlin
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // 직접 참조하는 객체에 접근
    }
}
```

`lateinit`는 (주요 생성자가 아닌) 클래스 몸체에 정의된 `var` 프로퍼티가 커스텀 getter나 setter가 없는 경우에만 적용할 수 있다.
프로퍼티 타입은 null 불가여야 하고 기본 타입이면 안 된다.

`lateinit` 프로퍼티를 초기화하기 전에 접근하면 특수한 익셉션을 발생한다.
이 익셉션은 접근한 프로퍼티와 아직 초기화하지 않았다는 것을 명확하게 식별한다.

## 프로퍼티 오버라이딩

[프로퍼티 오버라이딩](classes.html#overriding-properties)을 보자.

## 위임 프로퍼티(Delegated Properties)

보통 대부분 프로퍼티는 단순히 지원 필드에서 읽어온다(어쩌면 쓰기도 한다).
반면에 커스텀 getter와 setter를 가진 프로퍼티는 동작을 구현할 수 있다.
그 중간에 프로퍼티가 동작하는 공통 패턴이 존재한다. 패턴의 예로 지연 값, 주어진 키로 맵에서 읽기, 데이터베이스 접근하기,
접근할 때 리스너에 통지하기 등이 있다.    

[_위임 프로퍼티_](delegated-properties.html)를 사용해서 그런 공통 기능을 구현할 수 있다.
