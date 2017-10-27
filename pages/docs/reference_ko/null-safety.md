---
type: doc
layout: reference_ko
category: "Syntax"
title: "Null 안전성"
---

# Null 안정성

## Null 가능 타입과 Not-null 타입
{:#nullable-types-and-non-null-types}

코틀린 타입 시스템은 [10억 달러 실수](http://en.wikipedia.org/wiki/Tony_Hoare#Apologies_and_retractions)라고 알려진,
코드에서 null을 참조하는 위험을 없애는데 도움을 준다.

자바를 포함한 많은 프로그래밍 언어에서 공통으로 발견되는 실수 중 하나는 null 레퍼런스의 멤버를 참조해서
null 참조 익셉션이 발생하는 것이다. 자바에서는 `NullPointerException`, 줄여서 NPE이 이에 해당한다.

코틀린 타입 시스템은 코드에서 `NullPointerException`을 제거하는데 도움을 준다.
NPE는 다음 상황에서만 가능하다:

* `throw NullPointerException()`를 명시적으로 실행
* 뒤에서 설명한 `!!` 연산자 사용
* NPE를 발생하는 외부의 자바 코드 사용
* 초기화와 관련해서 데이터의 일관성이 깨졌을 때(생성자 어딘가에서 초기화되지 않은 *this*를 사용할 수 있음)

코틀린 타입 시스템은 *null*{: .keyword }을 가질 수 있는 레퍼런스(nullable 레퍼런스)와 가질 수 없는 레퍼런스(non-null 레퍼런스)를 구분한다.
예를 들어, `String` 타입 변수는 *null*{: .keyword }을 가질 수 없다:

``` kotlin
var a: String = "abc"
a = null // 컴파일 에러
```

null을 가질 수 있으려면 `String?`와 같이 변수를 null 가능 String으로 선언해야 한다:

``` kotlin
var b: String? = "abc"
b = null // ok
```

이제 `a`의 프로퍼티에 접근하거나 메서드를 호출할 때 NPE가 발생하지 않고 안전하게 사용할 수 있다는 것을 보장한다:

``` kotlin
val l = a.length
```

하지만, `b`의 동일한 프로퍼티에 접근시도를 하면 안전하지 않으므로 컴파일러 에러가 발생한다:

``` kotlin
val l = b.length // 에러: 변수 'b'는 null일 수 있다
```

하지만, 여전히 그 프로퍼티에 접근해야 한다면, 이를 위한 몇 가지 방법이 있다.

## 조건으로 *null*{: .keyword } 검사하기

첫 번째로 `b`가 *null*{: .keyword }인지 명시적으로 검사하고 두 옵션을 따로 처리할 수 있다:

``` kotlin
val l = if (b != null) b.length else -1
```

컴파일러는 검사 결과 정보를 추적해서 *if*{: .keyword } 안에서 `length`를 호출할 수 있도록 한다.
더 복잡한 조건도 지원한다:

``` kotlin
if (b != null && b.length > 0) {
    print("String of length ${b.length}")
} else {
    print("Empty string")
}
```

이 코드는 `b`가 불변인 경우에만 작동한다(예를 들어, null 검사와 사용 사이에 수정할 수 없는 로컬 변수나 
지원 필드가 있고 오버라이딩할 수 없는 *val*{: .keyword } 멤버인 경우).
왜냐면, 그렇지 않을 경우 검사 이후에 `b`를 *null*{: .keyword }로 바꿀 수 있기 때문이다.

## 안전한 호출

두 번째는 안전 호출 연산자인 `?.`를 사용하는 것이다:

``` kotlin
b?.length
```

이 코드는 `b`가 null이 아니면 `b.length`를 리턴하고 그렇지 않으면 *null*{: .keyword }을 리턴한다.
이 식의 타입은 `Int?`이다.

안전 호출은 체인에서 유용하다. Employee 타입의 Bob이 Department에 속할(또는 속하지 않을) 수 있고,
department가 또 다른 Employee를 department의 head로 가질 수 있다고 할 때,
존재할 수도 있는 Bod의 department의 head를 구하는 코드를 다음과 같이 작성할 수 있다:

``` kotlin
bob?.department?.head?.name
```

이 체인은 프로퍼티 중 하나라도 null이면 *null*{: .keyword }을 리턴한다.

non-null 값에만 특정 오퍼레이션을 수행하고 싶다면 안전 호출 연산자를 [`let`](/api/latest/jvm/stdlib/kotlin/let.html)과 
함게 사용하면 된다:

``` kotlin
val listWithNulls: List<String?> = listOf("A", null)
for (item in listWithNulls) {
     item?.let { println(it) } // A를 출력하고 null을 무시
}
```

## Elvis 연산자
{:#elvis-operator}

nullable 레퍼런스인 `r`에 대해, "`r`이 null이 아니면 그것을 사용하고 그렇지 않으면 다른 not-null 값인 `x`를 사용한다"는
코드를 다음과 같이 작성할 수 있다:

``` kotlin
val l: Int = if (b != null) b.length else -1
```

완전한 *if*{: .keyword }-식 외에, 엘비스 연산자인 `?:`를 사용해서 다음과 같이 표현할 수도 있다:

``` kotlin
val l = b?.length ?: -1
```

`?:`의 왼쪽 식이 null이 아니면, 엘비스 연산자는 그것을 리턴하고, 그렇지 않으면 오른쪽 식을 리턴한다.
오른쪽 식은 왼쪽이 null일 경우에만 평가한다.

코틀린에서 *throw*{: .keyword }와 *return*{: .keyword }은 식이므로,
엘비스 연산자의 오른쪽에 둘을 사용할 수 있다. 이는 예를 들어 함수 인자를 검사할 때 매우 유용하다: 

``` kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ...
}
```

## `!!` 연산자
{:#the--operator}

`!!` 연산자는 NPE 선호자를 위한 세 번째 옵션이다. `b!!`라고 작성하면 `b`가 null이 아니면 b를 리턴하고(예를 들어, 
`String`), null이니면 NPE를 발생한다:

``` kotlin
val l = b!!.length
```

따라서 NPE를 원한다면 그것을 사용할 수 있다.
하지만, NPE는 명시적으로 사용해야 하고 뜻밖인 곳에서 나타나지 않도록 한다.

## 안전한 타입 변환

일반 타입 변환은 객체가 지정한 타입이 아니면 `ClassCastException`을 발생한다.
다른 옵션은 타입 변화에 실패하면 *null*{: .keyword }을 리턴하는 안전한 타입 변환을 사용하는 것이다:

``` kotlin
val aInt: Int? = a as? Int
```

## null가능 타입의 콜렉션

null 가능 타입의 요소를 갖는 콜렉션에서 null이 아닌 요소를 걸러내고 싶다면, `filterNotNull`를 사용한다:

``` kotlin
val nullableList: List<Int?> = listOf(1, 2, null, 4)
val intList: List<Int> = nullableList.filterNotNull()
```
