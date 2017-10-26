---
type: doc
layout: reference
category: Other
title: "콜렉션: List, Set, Map"
---

# 콜렉션: List, Set, Map

많은 언어와 달리 코틀린은 변경 가능한 콜렉션과 불변 콜렉션을 구분한다(리스트, 집합, 맵 등).
콜렉션을 수정할 수 있는지 정확하게 제어하는 것은 버그를 줄이고 좋은 API를 설계할 때 유용하다.

수정 가능한 콜렉션의 읽기 전용 _뷰_와 실제로 불변인 콜렉션의 차이는 미리 이해하는 것이 중요하다.
둘다 생성하기 쉽지만 타입 시스템은 이 차이를 표현하지 않는다.
따라서 (만약 관련이 있다면) 그것을 추적하는 것은 당신의 몫이다. 

코틀린의 `List<out T>` 타입은 `size`, `get`과 같은 읽기 전용 오퍼레이션을 제공하는 인터페이스이다.
자바처럼 `Iterable<T>`를 상속한 `Collection<T>`을 상속한다.
리스트를 수정하는 메서드는 `MutableList<T>`에 있다.
이 패턴은 또한 `Set<out T>/MutableSet<T>`과 `Map<K, out V>/MutableMap<K, V>`에도 적용된다.

아래 코드는 리스트와 집합의 기본 사용법을 보여준다:

``` kotlin
val numbers: MutableList<Int> = mutableListOf(1, 2, 3)
val readOnlyView: List<Int> = numbers
println(numbers)        // "[1, 2, 3]" 출력
numbers.add(4)
println(readOnlyView)   // prints "[1, 2, 3, 4]"
readOnlyView.clear()    // -> 컴파일 안 된다

val strings = hashSetOf("a", "b", "c", "c")
assert(strings.size == 3)
```

코틀린은 리스트나 집합을 생성하기 위한 전용 구문 구성 요소가 없다.
`listOf()`, `mutableListOf()`, `setOf()`, `mutableSetOf()`와 같은 표준 라이브러리에 있는 메서드를 사용해서
생성한다.
 
성능이 중요하지 않은 코드에서는 간단한 
[이디엄](idioms.html#read-only-map)인 `mapOf(a to b, c to d)`을 사용해서 맵을 생성할 수 있다.

`readOnlyView` 변수는 같은 리스트를 가리키고 하부 리스트를 변경할 때 바뀐다는 것에 주목하자.
만약 리스트에 존재하는 유일한 참조가 읽기 전용 타입이면, 콜렉션이 완전히 불변이라고 간주할 수 있다.
그런 콜렉션을 만드는 간단한 방법은 다음과 같다:

``` kotlin
val items = listOf(1, 2, 3)
```

현재 `listOf` 메서드는 배열 리스트를 사용해서 구현하는데,
앞으로는 리스트가 바뀌지 않는다는 사실을 이용해서 메모리에 더 효율적인 완전히 불변인 콜렉션 타입을 리턴하도록 구현할 것이다.

읽기 전용 타입은 [공변](generics.html#variance)이다.
이는 Rectangle이 Shape를 상속한다고 가정할 때
`List<Rectangle>`를 `List<Shape>`에 할당할 수 있다는 것을 의미한다.
변경가능 콜렉션 타입에서는 이를 허용하지 않는다.
왜냐면 런타임에 실패가 발생할 수 있기 때문이다.

때때로 특정 시점의 콜렉션에 대해 변경되지 않는 것을 보장하는 스냅샷을 호출자에게 리턴하고 싶을 때가 있다:

``` kotlin
class Controller {
    private val _items = mutableListOf<String>()
    val items: List<String> get() = _items.toList()
}
```

`toList` 확장 메서드는 단지 리스트 항목을 중복해서 갖는 리스트를 생성하며, 리턴한 리스트가 결코 바뀌지 않는다는 것을 보장한다.

리스트나 맵은 유용하게 쓸 수 있는 다양한 확장 메서드를 제공하는데, 이들 메서드에 익숙해지면 좋다:

``` kotlin
val items = listOf(1, 2, 3, 4)
items.first() == 1
items.last() == 4
items.filter { it % 2 == 0 }   // [2, 4]를 리턴

val rwList = mutableListOf(1, 2, 3)
rwList.requireNoNulls()        // [1, 2, 3]을 리턴
if (rwList.none { it > 6 }) println("No items above 6")  // "No items above 6" 출력
val item = rwList.firstOrNull()
```

... 또한 sort, zip, fold, reduce 등 있었으면 하는 유틸리티도 제공한다.

맵은 다음 패턴을 따른다. 다음과 같이 쉽게 생성하고 접근할 수 있다:

``` kotlin
val readWriteMap = hashMapOf("foo" to 1, "bar" to 2)
println(readWriteMap["foo"])  // "1"을 출력
val snapshot: Map<String, Int> = HashMap(readWriteMap)
```
