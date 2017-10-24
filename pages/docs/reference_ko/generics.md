---
type: doc
layout: reference
category: "Syntax"
title: "지네릭: in, out, where"
---

# 지네릭

자바와 마찬가지로, 코틀린 클래스는 타입 파라미터를 가질 수 있다:

``` kotlin
class Box<T>(t: T) {
    var value = t
}
```

일반적으로 지네릭 클래스의 인스턴스를 생성할 때 타입 인자를 전달해야 한다:

``` kotlin
val box: Box<Int> = Box<Int>(1)
```

하지만 파라미터를 추론할 수 있다면, 예를 들어 생성자 인자나 다른 방법으로, 타입 인자를 생략할 수 있다.

``` kotlin
val box = Box(1) // 1은 Int 타입이므로 컴파일러는 Box<Int>를 말한다는 것을 알아낼 수 있다
```

## 가변(Variance)

자바 타입 시스템에서 가장 다루기 힘든 것 중 하나가 와일드카드 타입이다([자바 지네릭 FAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html) 참고),
코틀린에는 와일드카드 타입이 없다. 대신 선언 위치 가변(declaration-site variance)과 타입 프로젝션(type projections)을 제공한다.

첫째로 자바에 왜 그런 미스테리한 와일드카드가 필요한자 생각해보자.
[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 28: *Use bounded wildcards to increase API flexibility*에서
문제를 설명하고 있다.
먼저 자바의 지네릭 타입은 **무공변**이다. 이는 `List<String>`이 `List<Object>`의 하위타입이 **아님을** 의미한다.
왜 그럴까? 만약 리스트가 **무공변**이면 자바 배열보다 나을 게 없다. 왜냐면 다음 코드가 컴파일되고 런타임에 익셉션이 발생하기 때문이다:

``` java
// 자바
List<String> strs = new ArrayList<String>();
List<Object> objs = strs; // !!! 뒤이어 오는 문제 원인이 여기에 있다. 자바는 이를 허용하지 않는다!
objs.add(1); // 여기서 Integer를 String 배열에 넣는다
String s = strs.get(0); // !!! ClassCastException: Integer를 String으로 변환할 수 없다
```
따라서 자바는 런타임 안정성을 보장하기 위해 이런 것을 막는다. 하지만 여기에는 어떤 영향(??? implication)이 있다.
`Collection` 인터페이스의 `addAll()` 메서드를 보자. 이 메서드의 시그너처는 어떨까? 직관적으로 다음과 같이 생각할 수 있다:

``` java
// 자바
interface Collection<E> ... {
  void addAll(Collection<E> items);
}
```

하지만 이는 다음과 같은 (완전히 안전한) 간단한 것도 할 수 없다:

``` java
// 자바
void copyAll(Collection<Object> to, Collection<String> from) {
  to.addAll(from); // !!! addAll의 실제 선언으로는 컴파일할 수 없다:
                   //       Collection<String>은 Collection<Object>의 하위 타입이 아니다
}
```

(자바에서 이를 힘겹게 배웠다. [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 25: *Prefer lists to arrays*를 보자.)


이것이 `addlAll()`의 실제 시그너처가 다음과 같은 이유이다:

``` java
// 자바
interface Collection<E> ... {
  void addAll(Collection<? extends E> items);
}
```

**와일드카드 타입 인자** `? extends E`는 이 메서드가 `E` 객체의 콜렉션이나 또는 `E` 자체만이 아닌 `E`의 *하위 타입* 콜렉션을 허용한다는 것을 나타낸다.
이는 items에서 안전하게 `E`를 **읽을* 수 있지만(이 콜렉션의 요소는 E의 하위 타입 인스턴스이다),
콜렉션에 **쓸 수는 없다**는 것을 의미한다. 왜냐면 items가 `E`의 어떤 하위 타입인지 알 수 없기 때문이다.
이런 제약의 대가로 `Collection<String>`가 `Collection<? extends Object>`의 하위 타입이 *되는* 것을 얻는다.
 
전문 용어로 **extends**\-bound(**upper** bound)를 가진 와일드카드는 타입을 **공변(covariant)**으로 만든다.

왜 이 트릭이 어떻게 동작하는지 이해하기 위한 핵심은 다소 간단하다. 콜렉션에서 항목을 **가져**올 수만 있다면 `String` 콜렉션을 사용하고 
`Object`로 읽어도 괜찬하다. 역으로 콜렉션에 항목을 **넣을* 수만 있다면 `Object` 콜렉션에 `String`을 넣어도 괜찮다.
자바에서 `List<Object>`의 **상위타입*이 `List<? super String>`이다. 

후자를 **반공변(contravariance)**이라고 부른다. `List<? super String>` 타입 인자에 대해 String을 취하는 메서드를 
호출할 수 있다(예, `add(String)`이나 `set(int, String)`을 호출할 수 있다). 반면에 List<T>에서 `T`를 리턴하는 무언가를 호출하면
`String`이 아닌 `Object`를 얻는다.

Joshua Bloch는 **읽기만** 할 수 있는 오브젝트를 **Producer**라 부르고 **쓰기만* 할 수 있는 오브젝트를 **Consumer**라고 부른다.
그는 "최대의 유연함을 위해, Producer나 Consumer를 표현하는 입력 파라미터에 와일드카드를 사용하라"고 권하고 있으며 쉽게 기억할 수 있는 약자를 제시했다.

*PECS는 Producer-Extends, Consumer-Super를 의미한다.*

*주의*: `List<? extends Foo>` 타입의 Producer 객체를 사용하면 `add()`나 `set()`을 호출할 수 없는데, 그렇다고 이 객체가 **불변*인 것은 아니다.
예를 들어, 리스트의 모든 항목을 삭제하기 위해 `clear()`를 호출하는 것을 막을 수 없다. 왜냐면 `clear()`는 어떤 파라미터가 받지 않기 때문이다.
와일드카드가(또는 다른 타입의 공변성) 보장하는 것은 **타입 안정성**뿐이다. 불변성은 완전히 다른 얘기다.

### 선언 위치 가변(Declaration-site variance)

지네릭 인터페이스 `Source<T>`가 파라미터로 `T`를 갖는 메서드가 없고 오직 `T`를 리턴하는 메서드만 있다고 하자:

``` java
// Java
interface Source<T> {
  T nextT();
}
```

`Source<String>` 인스턴스에 대한 레퍼런스를 `Source<Object>` 타입 변수에 저장하는 것은 완전히 안전하다 --
Consumer-메서드 호출이 없기 때문이다. 하지만 자바는 이를 알지 못하기에 여전히 이를 금지한다: 

``` java
// 자바
void demo(Source<String> strs) {
  Source<Object> objects = strs; // !!! 자바는 허용하지 않음
  // ...
}
```

이를 고치려면 `Source<? extends Object>` 타입의 객체를 선언해야 하는데 이는 의미가 없다. 
왜냐면 전처럼 변수에 대해 동일 메서드를 호출할 수 있고 이득없이 타입만 더 복잡해지기 때문이다.
하지만 컴파일러는 이를 모른다.

코틀린에서는 이런 종류를 컴파일러에 알려주는 방법을 제공한다. 이를 **선언 위치 가변(declaration-site variance)** 이라고 부른다.
Source의 **타입 파라미터** T에 대해 `Source<T>`의 멤버에서 T를 **리턴**(제공)만 하고 소비하지 않는다는 것을 명시할 수 있다.
이를 위해 **out** 수식어를 제공한다:

``` kotlin
abstract class Source<out T> {
    abstract fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs // 괜찮다. T가 out 파라미터기 때문이다.
    // ...
}
```

일반적인 규칙은 다음과 같다. 클래스 `C`의 타입 파라미터 `T`를 **out**으로 선언하면, `C`의 멤버에서 **out**-위치에만 T가 위치할 수 있다.
하지만 리턴에서 `C<Base>`가 안전하게 `C<Derivied>`의 상위타입이 될 수 있다.

전문 용어로 클래스 `C`는 파라미터 `T`에 **공변**한다 또는 `T`가 **공변** 타입 파라미터라고 말한다.
`C`를 `T`의 **소비자**가 아닌 `T`의 **생산자**로 볼 수 있다.

**out** 수식어를 **가변 애노테이션**이라고 부른다. 타입 파라미터 선언 위치에 이를 제공하므로 **선언 위치 가변**에 대해 말한다.
이는 타입 사용의 와일드카드를 통해 타입을 공변으로 만드는 자바의 **사용 위치 가변**과 대조된다.

**out** 외에 코틀린은 보완하는 가변 애노테이션인 **in**을 제공한다. 이는 타입 파라미터를 **반공변(contravariant)**으로 만든다.
반공변 파라미터는 오직 소비만 할 수 있고 생산할 수는 없다. 반공변의 좋은 예가 `Comparable`이다:

``` kotlin
abstract class Comparable<in T> {
    abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0은 Double 타입인데 이는 Number의 하위타입이다
    // 따라서 Comparable<Double> 타입의 변수에 x를 할당할 수 있다
    val y: Comparable<Double> = x // OK!
}
```

**in**과 **out** 단어가 자명하다고 믿기에(이미 C#에서 꽤나 오래 성공적으로 용어를 사용하고 있다),
앞서 언급한 약어는 더 이상 필요 없으며, 더 상위 목적으로 약어를 바꿔볼 수 있다:

**[실존주의](http://en.wikipedia.org/wiki/Existentialism) 변환: Consumer in, Producer out\!** :-)

## 타입 프로젝트

### 사용 위치 가변(Use-site variance): 타입 프로젝션

타입 파라미터 T를 *out*으로 선언하면 매우 편리하며, 사용 위치에서 하위타입에 대한 문제를 피할 수 있다.
하지만 어떤 클래스는 실제로 오직 `T`만 리턴하도록 제약할 수 없다.
배열이 좋은 예이다: 

``` kotlin
class Array<T>(val size: Int) {
    fun get(index: Int): T { /* ... */ }
    fun set(index: Int, value: T) { /* ... */ }
}
```

이 클래스는 `T`에 대해 공변이나 반공변일 수 없다. 이는 유연하지 못하게 만든다. 다음 함수를 보자:

``` kotlin
fun copy(from: Array<Any>, to: Array<Any>) {
    assert(from.size == to.size)
    for (i in from.indices)
        to[i] = from[i]
}
```

이 함수는 한 배열에서 다른 배열로 항목을 복사한다. 실제로 적용해보자:

``` kotlin
val ints: Array<Int> = arrayOf(1, 2, 3)
val any = Array<Any>(3) { "" } 
copy(ints, any) // Error: (Array<Any>, Array<Any>)를 기대함
```

여기서 유사한 문제가 발생한다. `Array<T>`는 `T`에 대해 **무공변(invariant)**이므로
`Array<Int>`와 `Array<Any>`는 각각 상대의 하위타입이 아니다.
왜? copy가 잘못된 것을 할 **수도** 있기 때문이다.
예를 들어, `from`이 Int 배열인데 `from`에 String을 **쓰는** 시도를 할 수 있으며   
이는 뒤에 `ClassCastException`을 발생할 수 있다.

우리가 원하는 것은 `copy()`가 잘못된 것을 하지 않도록 확신할 수 있는 것이다.
다음과 같이 `from`에 **쓰는** 것을 막을 수 있다:

``` kotlin
fun copy(from: Array<out Any>, to: Array<Any>) {
 // ...
}
```

여기서 일어난 일을 **타입 프로젝트(type projection)**이라고 부른다.
이제 `from`은 단순히 배열이 아니고 제한된(**projected**) 배열이다.
이제 타입 파라미터 `T`를 리턴하는 메서드만 호출할 수 있는데, 이 경우에는 `get()`만 호출할 수 있음을 의미한다.
이것이 코틀린의 **사용 위치 가변성**에 대한 접근법이며, 이는 자바의 `Array<? extends Object>`에 대응하지만
더 간단한 방법이다.

타입 프로젝트에 **in**을 사용할 수도 있다:

``` kotlin
fun fill(dest: Array<in String>, value: String) {
    // ...
}
```

`Array<in String>`는 자바의 `Array<? super String>`에 대응한다. 예를 들어, `fill()` 함수에
`Object` 배열이나 `CharSequence` 배열을 전달할 수 있다.

### 스타 프로젝션(Star-projections)

때때로 타입 인자에 대해 알지 못하지만 안전한 방법으로 타입 인자를 사용하고 싶을 때가 있다.
여기서 안전한 방법은 지네릭 타입의 프로젝션을 정의하는 것이다.
그 지네릭 타입의 모든 컨크리트 인스턴스는 그 프로젝션의 하위타입이 된다.

코틀린은 이를 위해 **스타 프로젝션**이라 불리는 구문을 제공한다:

 - `Foo<out T>`에서 `T`는 상위 한계로 `TUpper`를 가진 공변 타입 파라미터면, `Foo<*>`는 `Foo<out TUpper>`과 동일하다.
   이는 `T`를 알 수 없을 때 `Foo<*>`에서 안전하게 `TUpper`의 값을 *읽을* 수 있다는 것을 의미한다.
 - `Foo<in T>`에서 `T`가 반공변 타입 파라미터면, `Foo<*>`는 `Foo<in Nothing>`와 동일하다.
   이는 `T`를 알 수 없을 때 안전하게 `Foo<*>`에 *쓸* 수 없다는 것을 의미한다.
 - `Foo<T>`에서 `T`가 상위 한계로 `TUpper`를 가진 무공변 타입 파라미터면,
   `Foo<*>`는 값을 읽는 것은 `Foo<out TUpper>`와 동일하고
   값을 쓰는 것은 `Foo<in Nothing>`와 동일하다.

지네릭 타입이 타입 파라미터가 여러 개이면, 각 타입 파라미터를 따로 프로젝트셩할 수 있다.
예를 들어, `interface Function<in T, out U>` 타입이 있을 때 다음의 스타-프로젝션을 만들 수 있다:

 - `Function<*, String>`은 `Function<in Nothing, String>`을 의미한다.
 - `Function<Int, *>`은 `Function<Int, out Any?>`를 의미한다.
 - `Function<*, *>`은 `Function<in Nothing, out Any?>`를 의미한다.

*노트*: 스타-프로젝션은 자바의 war 타입과 거의 같지만 안전하다.

## 지네릭 함수

클래스뿐만 아니라 함수도 타입 파라미터를 가질 수 있다.
함수 이름 앞에 타입 파라미터를 위치시킨다:

``` kotlin
fun <T> singletonList(item: T): List<T> {
    // ...
}

fun <T> T.basicToString() : String {  // 확장 함수
    // ...
}
```

지네릭 함수를 호출하려면 호출 위치에서 함수 이름 **뒤에** 타입 인자를 지정한다:

``` kotlin
val l = singletonList<Int>(1)
```

## 지네릭 제한

주어진 타입 파라미터에 올 수 있는 모든 가능한 타입 집합은 **지네릭 제한**에 제약을 받는다.

### 상위 한계

대부분 공통 타입의 제한은 자바의 *extends* 키워드에 해당하는 **상위 한계**이다:

``` kotlin
fun <T : Comparable<T>> sort(list: List<T>) {
    // ...
}
```

콜론 뒤에 지정한 타입이 **상위 한계**이다. 오직 `Comparable<T>`의 하위타입만 `T`에 사용할 수 있다.
다음 예를 보자:

``` kotlin
sort(listOf(1, 2, 3)) // OK. Int는 Comparable<Int>의 하위타입이다.
sort(listOf(HashMap<Int, String>())) // 에러: HashMap<Int, String>는 Comparable<HashMap<Int, String>>의 하위타입이 아니다.  
is not a subtype of Comparable<HashMap<Int, String>>
```

기본 상위 한계는 (지정하지 않은 경우) `Any?`이다.
꺽쇠 괄호 안에는 한 개의 상위 한계만 지정할 수 있다.
한 개의 타입 파라미터에 한 개 이상의 상위 한계를 지정해야 하면 별도의 **where**\-절을 사용해야 한다:

``` kotlin
fun <T> cloneWhenGreater(list: List<T>, threshold: T): List<T>
    where T : Comparable,
          T : Cloneable {
  return list.filter { it > threshold }.map { it.clone() }
}
```
