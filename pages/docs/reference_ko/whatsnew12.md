---
type: doc
layout: reference
title: "코틀린 1.2의 새로운 점"
---

# 코틀린 1.2의 새로운 점

## 내용

* [멀티플랫폼 프로젝트](#multiplatform-projects-experimental)
* [다른 언어 특징](#other-language-features)
* [표준 라이브러리](#standard-library)
* [JVM 백엔드](#jvm-backend)
* [자바스크립트 백엔드](#javascript-backend)

## 멀티플랫폼 프로젝트 (실험)

멀티플랫폼 프로젝트는 코틀린 1.2에 새로 추가된 **실험** 특징으로, 코틀린이 지원하는 대상 플랫폼-JVM, 자바스크립트, (앞으로) 네이티브-간에
코트를 재사용할 수 있게 한다. 멀티 플랫폼 프로젝트는 세 종류의 모듈을 갖는다:

* *공통(common)* 모듈은 특정 플랫폼에 특화되지 않은 코드와 플랫폼에 의존적인 API 구현이 없는 선언을 포함한다.  
* *플랫폼* 모듈은 공통 모듈에서 특정 플랫폼을 위한 플랫폼 의존적인 선언의 구현 및 다른 플랫폼 의존적인 코드를 포함한다. 
* 일반 모듈은 특정 플랫폼을 대상으로 하고, 플랫폼 모듈의 의존이 되거나 플랫폼 모듈에 의존할 수 있다.

특정 플랫폼에 맞게 멀티플랫폼 프로젝트를 컴파일하면, 공통과 플랫폼에 특화된 부분을 위한 코드가 만들어진다.

멀티플랫폼 프로젝트의 핵심 특징은 **기대(expected)와 실제 선언**을 통해 플랫폼에 특화된 부분에서 공통 코드의 의존을 표현이 가능하다는 것이다.
기대 선언은 API(클래스, 인터페이스, 애노테이션, 최상위 선언 등)를 지정한다.
실제 선언은 API의 플랫폼 의존 구현 또는 외부 라이브러리에 존재하는 API 구현에 대한 타입별칭이다.
여기 예제가 있다:

공통 코드에서:

```kotlin
// 기대하는 플랫폼 특화 API:
expect fun hello(world: String): String

fun greet() {
    // 기대 API 사용:
    val greeting = hello("multi-platform world")
    println(greeting)
}

expect class URL(spec: String) {
    open fun getHost(): String
    open fun getPath(): String
}
```

JVM 플랫폼 코드에서:

```kotlin
actual fun hello(world: String): String =
    "Hello, $world, on the JVM platform!"

// 존재하는 플랫폼에 맞는 구현 사용:
actual typealias URL = java.net.URL
```

멀티플랫폼 프로젝트를 만드는 절차 및 상세 내용은 [문서](multiplatform.html)를 참고한다.

## 다른 언어 특징

### 애노테이션에서 배열 리터럴

코틀린 1.2부터 애노테이션의 배열 인자에 `arrayOf` 함수 대신에 새로운 배열 리터럴 구문을 전달할 수 있다:

```kotlin
@CacheConfig(cacheNames = ["books", "default"])
public class BookRepositoryImpl {
    // ...
}
```

배열 리터럴 구문은 애노테이션 인자에서만 가능하다.

### Lateinit 최상위 프로퍼티와 로컬 변수

이제 `lateinit` 수식어를 최상위 프로젝트와 로컬 변수에 사용할 수 있다. 예를 들어, 한 객체의 생성자 인자에 전달할 람다가
나중에 정의할 다른 객체를 참조할 때 `lateinit` 로컬 변수를 사용할 수 있다:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
class Node<T>(val value: T, val next: () -> Node<T>)

fun main(args: Array<String>) {
    //sampleStart
    // A cycle of three nodes:
    lateinit var third: Node<Int>
    
    val second = Node(2, next = { third })
    val first = Node(1, next = { second })
    
    third = Node(3, next = { first })
    //sampleEnd
    
    val nodes = generateSequence(first) { it.next() }
    println("Values in the cycle: ${nodes.take(7).joinToString { it.value.toString() }}, ...")
}
```
</div>

### lateinit var가 초기화되었는지 검사

이제 프로퍼티 참조에 대해 `isInitialized`를 사용해서 lateinit 변수가 초가화되었는지 검사할 수 있다:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
class Foo {
    lateinit var lateinitVar: String
    
    fun initializationLogic() {
        //sampleStart
        println("isInitialized before assignment: " + this::lateinitVar.isInitialized)
        lateinitVar = "value"
        println("isInitialized after assignment: " + this::lateinitVar.isInitialized)    
        //sampleEnd
    }
}

fun main(args: Array<String>) {
	Foo().initializationLogic()
}
```
</div>

### 기본 함수타입 파라미터를 가진 인라인 함수

이제 인라인 함수의 인라인된 함수타입 파라미터를 위한 기본 값을 허용한다:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
//sampleStart
inline fun <E> Iterable<E>.strings(transform: (E) -> String = { it.toString() }) = 
    map { transform(it) }

val defaultStrings = listOf(1, 2, 3).strings()
val customStrings = listOf(1, 2, 3).strings { "($it)" } 
//sampleEnd

fun main(args: Array<String>) {
    println("defaultStrings = $defaultStrings")
    println("customStrings = $customStrings")
}
```
</div>

### 타입 추론을 위해 명시적인 변환 정보를 사용

코틀린 컴파일러는 이제 타입 변환 정보를 타입 추론에 사용할 수 있다. 만약 타입 파라미터 `T`를 리턴하는 지네릭 메서드를 호출하고
리턴 값을 `Foo` 타입으로 변환한다면, 컴파일러가 이 호출에 대해 `T`가 `Foo` 타입이 된다는 것을 이해할 수 있다.

이는 특히 안드로이드 개발자에게 중요한다. 왜냐면, 컴파일러가 이제 Android API level 26의  
지네릭 `findViewById` 호출을 올바르게 분석할 수 있기 때문이다:

```kotlin
val button = findViewById(R.id.button) as Button
```

### 스마트 변환 향상

When a variable is assigned from a safe call expression and checked for null, the smart cast is now applied to 
the safe call receiver as well:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun countFirst(s: Any): Int {
    //sampleStart
    val firstChar = (s as? CharSequence)?.firstOrNull()
    if (firstChar != null)
       return s.count { it == firstChar } // s: Any is smart cast to CharSequence
    
    val firstItem = (s as? Iterable<*>)?.firstOrNull()
    if (firstItem != null)
       return s.count { it == firstItem } // s: Any is smart cast to Iterable<*>
    //sampleEnd
    
    return -1
}

fun main(args: Array<String>) {
    val string = "abacaba"
    val countInString = countFirst(string)
    println("called on \"$string\": $countInString")
    
    val list = listOf(1, 2, 3, 1, 2)
    val countInList = countFirst(list)
    println("called on $list: $countInList")
}
```
</div>

Also, smart casts in a lambda are now allowed for local variables that are only modified before the lambda:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun main(args: Array<String>) {
    val flag = args.size == 0
    
    //sampleStart
    var x: String? = null
    if (flag) x = "Yahoo!"

    run {
        if (x != null) {
            println(x.length) // x is smart cast to String
        }
    }
    //sampleEnd
}
```
</div>

### this::foo를 ::foo로 약식표기하는 것 지원

`this`의 멤버에 묶인 호출가능 레퍼런스를 이제 명시적인 리시버 없이 작성할 수 있다(예 `this::foo` 대신 `::foo`).
이는 또한 외부 리시버의 멤버를 참조하는 람다에서 호출가능 레퍼런스를 보다 사용하기 편라하게 만들어준다. 

### 기존을 깨는 변경: try 블록 이후 충분히 스마트한 변환

앞서 코틀린은 `try` 블록 이후에 스마트 변환을 위해 `try` 블록 안에서 할당을 사용했다.
이는 타입이나 null 안정성을 깰 수 있고, 런타임에 실패를 유바할 수 있다. 
1.2는 이 이슈를 고쳐서, 스마트 변환을 보다 엄격하게 만들었지만, 그러한 스마트 변환에 기댄 일부 코드가 깨진다.

이전 스마트 변환 동작을 사용하려면 컴파일러 인자로 `-Xlegacy-smart-cast-after-try` 대체 플래그를 전달하면 된다.
이 플래그는 코틀린 1.3에서 디프리케이드될 것이다.

### 디프리케이션: copy를 오버라이딩하는 데이터 클래스

데이터 클래스가 이미 `copy` 함수를 가진 타입을 상속할 경우,
데이터 클래스를 위해 생성한 `copy` 구현은 상위 타입의 기본 값을 사용했다.
이는 직관에 반하는 결과를 만들거나 상위타입에 기본 파라미터가 없으면 런타임에 실패했다.

`copy` 충돌을 야기하는 상속은 코틀린 1.2에서 디프리케이트되어 경고를 발생하며,
코틀린 1.3에서는 에러를 발생할 것이다.

### 디프리케이션: enum 엔트리의 중첩 타입

enum 엔트리에서 `inner class`가 아닌 중첩 타입을 정의하는 것은 초기화 로직에서의 이슈로 디프리케이트되었다.
코틀린 1.2에서 이는 경고를 발생하며, 코틀린 1.3에서는 에러를 발생할 것이다.

### 디프리케이션: vararg를 위한 한 개의 이름 가진 인자

애노테이션의 배열 리터럴과의 일관성을 위해, 이름 가진 형식으로 vararg 파라미터에 한 개 항목을 전달하는 것을(`foo(items = i)`)
디프리케이트했다. 대응하는 배열 생성 함수와 펼침 연산자를 사용해라:

```kotlin
foo(items = *intArrayOf(1))
```

이 경우 성능 저하를 막기 위해 중복된 배열 생성을 제거하는 최적화를 한다.
한 개 인자는 코틀린 1.2에서 경고를 발생하고, 코틀린 1.3에서는 없어질 것이다.

### 디프리케이션: Throwable을 확장한 지네릭 클래스의 내부 클래스

`Throwable`을 상속한 지네릭 타입의 내부 클래스는 throw-catch 시나리오에서 타입 안정성을 위반하며 그래서 디프리케이트했다.
코틀린 1.2에서는 경고이며 코틀린 1.3에서는 에러이다.

### 디프리케이션: 읽기 전용 프로퍼티의 지원 필드 수정

커스텀 게터에서 `field = ...`로 읽기 전용 프로퍼티의 지원 필드를 수정하는 것을 디프리케이트했다.
코틀린 1.2에서는 경고이며 코틀린 1.3에서는 에러이다.

## 표준 라이브러리

### 코틀린 표준 라이브러리 아티팩트와 패키지 분리

이제 코틀린 표준 라이브러리는 패키지 분리(여러 jar 파일이 같은 패키지의 클래스를 선언)를 금지하는 자바 9 모듈 시스템과 완전히 호환된다.
이를 지원하기 위해 `kotlin-stdlib-jdk7`와 `kotlin-stdlib-jdk8`의 새 아티팩트를 추가했고,
이 아티팩트가 기존의 `kotlin-stdlib-jre7`와 `kotlin-stdlib-jre8`을 대체한다.

코틀린 입장에서 새 아티팩트의 선언은 같은 패키지 이름으로 보이지만, 자바 입장에서는 다른 패키지 이름을 갖는다.
따라서, 새 아티팩트로 바꿔도 소스 코드를 바꿀 필요가 없다.

새 모들 시스템과 호환성을 보장하기 위한 또 다른 변경은 `kotlin-reflect` 라이브러리에서
`kotlin.reflect` 패키지의 디프리케이트한 선언을 제거한 것이다. 만약 그 패키지를 사용하고 있다면,
코틀린 1.1부터 지원하는 `kotlin.reflect.full` 패키지에 있는 선언을 사용하게 바꿔야 한다.

### windowed, chunked, zipWithNext

`Iterable<T>`, `Sequence<T>`, `CharSequence`의 새 확장은 버퍼링이나 배치 처리(`chunked`), 슬라이딩 윈도우와 슬라이딩 평균 계산(`windowed`),
다음 항목과의 쌍(pair) 처리(`zipWithNext`)와 같은 경우를 다룬다:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun main(args: Array<String>) {
    //sampleStart
    val items = (1..9).map { it * it }

    val chunkedIntoLists = items.chunked(4)
    val points3d = items.chunked(3) { (x, y, z) -> Triple(x, y, z) }
    val windowed = items.windowed(4)
    val slidingAverage = items.windowed(4) { it.average() }
    val pairwiseDifferences = items.zipWithNext { a, b -> b - a }
    //sampleEnd
    
    println("items: $items\n")
    
    println("chunked into lists: $chunkedIntoLists")
    println("3D points: $points3d")
    println("windowed by 4: $windowed")
    println("sliding average by 4: $slidingAverage")
    println("pairwise differences: $pairwiseDifferences")
}
```
</div>

### fill, replaceAll, shuffle/shuffled

리스트 조작을 위해 `MutableList`을 위한 `fill`, `replaceAll`, `shuffle`와 읽기 전용 `List`를 위한 `shuffled` 확장 함수를 추가했다:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun main(args: Array<String>) {
    //sampleStart
    val items = (1..5).toMutableList()
    
    items.shuffle()
    println("Shuffled items: $items")
    
    items.replaceAll { it * 2 }
    println("Items doubled: $items")
    
    items.fill(5)
    println("Items filled with 5: $items")
    //sampleEnd
}
```
</div>

### kotlin-stdlib의 수치 연산

예전부터 들어온 요청을 충족하기 위해 코틀린 1.2에 `kotlin.math` API를 추가했다. 이 API는 JVM과 JS에 공통된 수치 연산을 지원하며
다음을 포함한다:

* 상수: `PI`와 `E`;
* 삼각함수: `cos`, `sin`, `tan`와 역함수: `acos`, `asin`, `atan`, `atan2`;
* 쌍곡선함수(cosh): `cosh`, `sinh`, `tanh`와 역함수: `acosh`, `asinh`, `atanh`
* 지수함수: `pow` (확장 함수), `sqrt`, `hypot`, `exp`, `expm1`;
* 로그함수: `log`, `log2`, `log10`, `ln`, `ln1p`;
* 올림:
    * `ceil`, `floor`, `truncate`, `round` (짝수로 올림) 함수;
    * `roundToInt`, `roundToLong` (정수로 반올림) 확장 함수;
* 부호와 절댓값:
    * `abs`와 `sign` 함수
    * `absoluteValue`와 `sign` 확장 프로퍼티
    * `withSign` 확장 함수
* 두 값의 `max`와 `min`;
* 이진 표현:
    * `ulp` 확장 함수
    * `nextUp`, `nextDown`, `nextTowards` 확장 함수
    * `toBits`, `toRawBits`, `Double.fromBits` (이는 `kotlin` 패키지에 포함)

`Float` 인자에도 (상수를 제외한) 동일한 함수 집합이 가능하다.

### BigInteger와 BigDecimal을 위한 연산자와 변환

코틀린 1.2는 `BigInteger`와 `BigDecimal`를 사용한 연산과 다른 숫자 타입에서 이 두 타입을 생성하기 위한 함수를 추가했다:

* `Int`와 `Long`을 위한 `toBigInteger`
* `Int`, `Long`, `Float`, `Double`, `BigInteger`를 위한 `toBigDecimal` 
* 수치와 비트 연산자 함수:
    * 이항 연산자 `+`, `-`, `*`, `/`, `%`와 중위 함수 `and`, `or`, `xor`, `shl`, `shr`
    * 단항 연산자 `-`, `++`, `--`와 함수 `inv`.

### 실수 타입과 비트 변환

`Double` 및 `Float`과 비트 표현 간의 변환을 위한 새 함수를 추가했다:

* `toBits`와 `toRawBits`는 `Double`에 대해 `Long`을 리턴하고 `Float`에 대해 `Int`를 리턴
* 비트 표현에서 실수를 생성하는 `Double.fromBits`와 `Float.fromBits`

### Regex는 이제 Serializable

`kotlin.text.Regex` 클래스가 `Serializable`이 되어, 이제 직렬화 계층에서 사용할 수 있다.

### Closeable.use는 가능한 경우 Throwable.addSuppressed 를 호출함

다른 익셉션 이후에 자원을 닫는 과정에서 익셉션이 발생한 경우, `Closeable.use` 함수는 `Throwable.addSuppressed`를 호출한다.

이 동작을 사용하려면 의존에 `kotlin-stdlib-jdk7`를 추가해야 한다.

## JVM 백엔드

### 생성자 호출 정규화(normalization)

1.0 버전부터 코틀린은 try-catch 식이나 인라인 함수 호출과 같은 복잡합 제어 흐름을 갖는 식을 지원했다.
그런 코드는 자바 가상 머신 스펙에 따르면 유효하다. 안타깝게도, 일부 바이트코드 처리 도구는 
생성자 호출 인자에 그런 식이 위치할 때, 그런 코드를 잘 처리하지 못한다.

그런 바이트코드 처리 도구 사용자가 겪는 이런 문제를 완화하기 위해, 명령행 옵션(`-Xnormalize-constructor-calls=MODE`)을 추가했다.
이 옵션을 사용하면 컴파일러는 생성자에 대해 자바와 비슷한 바이트코드를 생성한다. `MODE`는 다음 중 하나이다:

* `disable` (기본) – 코틀린 1.0 및 1.1과 같은 방법으로 바이트코드를 생성한다.
* `enable` – 생성자 호출에 대해 자바와 같은 바이트코드를 생성한다. 이는 클래스를 로딩하고 초기화하는 순서를 바꿀 수 있다.
* `preserve-class-initialization` – 생성자 호출에 대해 자바와 같은 바이트코드를 생성한다. 클래스 초기화 순서가 유지됨을 보장한다.
이는 어플리케이션의 전체 성능에 영향을 줄 수 있다. 여러 클래스가 복잡한 상태를 공유하고 클래스 초기화 시점에 수정하는 경우에 한해서만 사용해라.

“수동” 차선책은 호출 인자에서 직접 하위 식을 평가하는 대신, 변수에 흐름 제어를 갖는 하위 식 값을 저장하는 것이다.
이는 `-Xnormalize-constructor-calls=enable`와 유사하다.

### 자바 디폴트 메서드 호출 

코틀린 1.2 이전에, JVM 1.6을 대상으로 할 때 인터페이스가 자바의 디폴트 메서드를 오버라이딩할 경우 상위 호출에 대해 
`Super calls to Java default methods are deprecated in JVM target 1.6. Recompile with '-jvm-target 1.8'` 경고를 발생했다.
코틀린 1.2부터는 경고 대신 **에러**가 발생하며, JVM 1.8 대상으로 코드를 컴파일해야 한다.

### 기존을 깨는 변경: 플랫폼 타입을 위해 x.equals(null)의 일관된 동작

자바 기본 타입(`Int!`, `Boolean!`, `Short`!, `Long!`, `Float!`, `Double!`, `Char!`)에 매핑되는 플랫폼 타입에 대해 `x.equals(null)`를 호출하면
`x`가 null일 때 잘못된 값을 리턴한다. 코틀린 1.2부터 플랫폼 타입의 null 값에 대해 `x.equals(...)`를 호출하면
**NPE를 발생한다**(하지만 `x == ...`는 아니다).

1.2 이전과 동일하게 동작하게 하려면, 컴파일러에 `-Xno-exception-on-explicit-equals-for-boxed-null` 플래그를 전달한다.

### 기존을 깨는 변경: fix for platform null escaping through an inlined extension receiver

플랫폼 타입인 null 값에 대해 호출되는 인라인 확장 함수는 리시버가 null인지 검사하지 않았고,
그래서 null이 다른 코드로 퍼지는 것을 허용했다.
코틀린 1.2는 호출 위치에서 이런 검사를 강제하며, 리시버가 null이면 익셉션을 발생한다.

이전 동작으로 바꾸려면, 컴파일러에 대체 플래그인 `-Xno-receiver-assertions`를 전달한다.

## 자바스크립트 백엔드

### TypedArrays 지원이 기본으로 가능

JS 타입 가진 배열은 `IntArray`, `DoubleArray`와 같은 코틀린 원시 타입 배열을
[자바스크립트 타입 가진 배열](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)로
변환하는 JS 타입드 배열 지원을 기본으로 사용한다. 이는 앞서 선택(opt-in) 기능이었다. 

## 도구

### 에러로서의 경고

컴파일러는 이제 모든 경고를 에러로 다루는 옵션을 제공한다. 명령행에서 `-Werror`를 사용하거나 그레이들에서 다음 코드를 사용하면 된다:

```groovy
compileKotlin {
    kotlinOptions.allWarningsAsErrors = true
}
```
