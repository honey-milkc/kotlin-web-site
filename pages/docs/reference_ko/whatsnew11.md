---
type: doc
layout: reference
title: "코틀린 1.1의 새로운 점"
---

# 코틀린 1.1의 새로운 점

## 내용

* [코루틴](#coroutines-experimental)
* [다른 언어 특징](#other-language-features)
* [표준 라이브러리](#standard-library)
* [JVM 백엔드](#jvm-backend)
* [자바스크립트 백엔드](#javascript-backend)

## 자바스크립트

코틀린 1.1부터 자바스크립트 대상(target)은 더 이상 실험단계가 아니다. 모든 언어 특징을 지원하고 프론트엔드 개발 환경과 통합을 위한
많은 새 도구를 제공한다. 자세한 변경 목록은 [아래](#javascript-backend)에서 확인할 수 있다.

## 코루틴 (실험)

코틀린 1.1의 핵심 특징은 *코루틴*이다. 코루틴은 `async`/`await`, `yield` 내지 이와 유사한 프로그램 패턴을 지원한다
코틀린 설계의 핵심 특징은 코루틴 실행 구현이 언어가 아닌 라이브러리의 일부라는 점이다.
이는 코루틴이 특정 프로그래밍 패러다임이나 병행 라이브러리에 묶이지 않도록 한다.

코루틴은 효과적인 경량 쓰레드로 나중에 중지나 재시작할 수 있다.
[*서스펜딩 함수*](coroutines.html#suspending-functions)를 통해 코루틴을 지원한다.
이 함수를 호출하면 코루틴을 잠재적으로 지연할 수 있고, 익명 지연 함수(예, 지연 람다)를 사용하면 새로운 코루틴을 시작할 수 있다.

외부 라이브러리인 [kotlinx.coroutines](https://github.com/kotlin/kotlinx.coroutines)로 구현한 `async`/`await`를 보자: 

``` kotlin
// 백그라운드 쓰레드 풀에서 코드 실행
fun asyncOverlay() = async(CommonPool) {
    // 두 개의 비동기 오퍼레이션 시작
    val original = asyncLoadImage("original")
    val overlay = asyncLoadImage("overlay")
    // 그리고 두 결과에 Overlay를 적용
    applyOverlay(original.await(), overlay.await())
}

// UI 컨텍스트에서 새로운 코루틴을 시작
launch(UI) {
    // 비동기 Overlay가 끝날때까지 대기
    val image = asyncOverlay().await()
    // 그리고 UI에 표시
    showImage(image)
}
```

여기서 `async { ... }`는 코루틴을 시작하고, `await()`를 사용하면 대기하는 오퍼레이션을 실행하는 동안 코루틴 실행을 지연하고,  
(아마도 다른 쓰레드에서) 대기중인 오퍼레이션이 끝날 때 실행을 재개한다. 

표준 라이브러리는 `yield`와 `yieldAll` 함수를 사용해서 *시퀀스 지연 생성*을 지원하기 위해 코루틴을 사용한다. 
그 시퀀스에서, 시퀀스 요소를 리턴하는 코드 블럭은 각 요소를 읽어온 이후에 멈추고
다음 요소를 요청할 때 재개한다. 다음은 예이다: 


<div class="sample" markdown="1" data-min-compiler-version="1.1"> 

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
  val seq = buildSequence {
      for (i in 1..5) {
          // i의 제곱을 생성
          yield(i * i)
      }
      // 범위를 생성
      yieldAll(26..28)
  }
  
  // 시퀀스를 출력
  println(seq.toList())
//sampleEnd
}
```

</div>

위 코드를 실행하고 결과를 보자. 마음껏 수정해서 실행해보자!

[코루틴 문서](/docs/reference_ko/coroutines.html)와 [튜토리얼](/docs/tutorials/coroutines-basic-jvm.html)에서
더 많은 정보를 참고할 수 있다.

코루틴은 현재 **실험적인 특징*으로, 코틀린 팀은 최종 1.1 버전 이후에 코루틴의 하위호환성을 지원하지 않을 수도 있다.


## 다른 언어 특징

### 타입 별칭

타입 별칭을 사용하면 기존 타입을 다른 이름으로 정의할 수 있다.
이는 함수 타입이나 콜렉션과 같은 지네릭 타입에 매우 유용하다. 다음은 예이다.

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
typealias OscarWinners = Map<String, String>

fun countLaLaLand(oscarWinners: OscarWinners) =
        oscarWinners.count { it.value.contains("La La Land") }

// 타입 이름(초기와 타입 별칭)을 상호사용할 수 있다:
fun checkLaLaLandIsTheBestMovie(oscarWinners: Map<String, String>) =
        oscarWinners["Best picture"] == "La La Land"
//sampleEnd

fun oscarWinners(): OscarWinners {
    return mapOf(
            "Best song" to "City of Stars (La La Land)",
            "Best actress" to "Emma Stone (La La Land)",
            "Best picture" to "Moonlight" /* ... */)
}

fun main(args: Array<String>) {
    val oscarWinners = oscarWinners()

    val laLaLandAwards = countLaLaLand(oscarWinners)
    println("LaLaLandAwards = $laLaLandAwards (in our small example), but actually it's 6.")

    val laLaLandIsTheBestMovie = checkLaLaLandIsTheBestMovie(oscarWinners)
    println("LaLaLandIsTheBestMovie = $laLaLandIsTheBestMovie")
}
```
</div>

다 자세한 내용은 [문서](type-aliases.html)와 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/type-aliases.md)을 참고한다.


### 객체에 묶인 callable 참조

이제 `::` 연산자를 이용해서 특정 객체 인스턴스의 메서드나 프로퍼티를 가리키는 [멤버 참조](reflection.html#function-references)를 구할 수 있다.
이전 버전에서는 람다에서만 표현할 수 있었다.
다음은 예이다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
val numberRegex = "\\d+".toRegex()
val numbers = listOf("abc", "123", "456").filter(numberRegex::matches)
//sampleEnd

fun main(args: Array<String>) {
    println("Result is $numbers")
}
```
</div>


다 자세한 내용은 [문서](reflection.html#bound-function-and-property-references-since-11)와 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/bound-callable-references.md)을 참고한다.


### 실드(sealed)와 데이터 클래스

코틀린 1.1은 1.0 버전에 있는 실드(sealed)와 데이터(data) 클래스의 일부 제약을 없앴다.
이제 최상위 실드 클래스의 하위클래스를 실드 클래스의 중첩 클래스뿐만 아니라 같은 파일에 정의할 수 있다.
데이터 클래스가 다른 클래스를 상속할 수 있다.
이를 사용하면 식(expression) 클래스의 계층을 멋지고 깔끔하게 정의할 수 있다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
sealed class Expr

data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()

fun eval(expr: Expr): Double = when (expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
}
val e = eval(Sum(Const(1.0), Const(2.0)))
//sampleEnd

fun main(args: Array<String>) {
    println("e is $e") // 3.0
}
```
</div>

자세한 내용은 [문서](sealed-classes.html),
[실드 클래스](https://github.com/Kotlin/KEEP/blob/master/proposals/sealed-class-inheritance.md) KEEP 또는
[데이터 클래스](https://github.com/Kotlin/KEEP/blob/master/proposals/data-class-inheritance.md) KEEP을 참고한다.


### 람다에서의 분리 선언

이제 람다에 인자를 전달할 때 [분리 선언](multi-declarations.html) 구문을 사용할 수 있다. 다음은 예이다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val map = mapOf(1 to "one", 2 to "two")
    // 이전
    println(map.mapValues { entry ->
        val (key, value) = entry
        "$key -> $value!"
    })
    // 이제
    println(map.mapValues { (key, value) -> "$key -> $value!" })
//sampleEnd    
}
```
</div>

더 자세한 내용은 [문서](multi-declarations.html#destructuring-in-lambdas-since-11)와 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/destructuring-in-parameters.md)을 참고한다.


### 사용하지 않는 파라미터를 위한 밑줄

여러 파라미터를 갖는 람다에 대해, 사용하지 않는 파라미터 자리에 이름 대신에 `_` 글자를 사용할 수 있다:   

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
    val map = mapOf(1 to "one", 2 to "two")

//sampleStart
    map.forEach { _, value -> println("$value!") }
//sampleEnd    
}
```
</div>

이는 또한 [분리 선언](multi-declarations.html)에서도 동작한다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
data class Result(val value: Any, val status: String)

fun getResult() = Result(42, "ok").also { println("getResult() returns $it") }

fun main(args: Array<String>) {
//sampleStart
    val (_, status) = getResult()
//sampleEnd
    println("status is '$status'")
}
```
</div>

더 자세한 정보는 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/underscore-for-unused-parameters.md)을 참고한다.


### 숫자 리터럴에서의 밑줄

자바 7처럼 코틀린도 숫자를 그룹으로 나누기 위해 숫자 리터럴에서 밑줄을 사용할 수 있다.

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
val oneMillion = 1_000_000
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
//sampleEnd

fun main(args: Array<String>) {
    println(oneMillion)
    println(hexBytes.toString(16))
    println(bytes.toString(2))
}
```
</div>

자세한 내용은 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/underscores-in-numeric-literals.md)을 참고한다.


### 프로퍼티를 위한 짧은 구문

프로퍼티의 getter를 식 몸체로 정의한 경우, 프로퍼티 타입을 생략할 수 있다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
data class Person(val name: String, val age: Int) {
    val isAdult get() = age >= 20 // 프로퍼티 타입을 'Boolean'으로 추론
}
//sampleEnd

fun main(args: Array<String>) {
    val akari = Person("Akari", 26)
    println("$akari.isAdult = ${akari.isAdult}")
}
```
</div>

### 인라인 프로퍼티 접근자

프러퍼티가 backing 필드를 갖지 않으면 프로퍼티 접근자에 `inline` 수식어를 붙일 수 있다.
이 접근자는 [인라인 함수](inline-functions.html)와 동일한 방법으로 컴파일된다.

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
public val <T> List<T>.lastIndex: Int
    inline get() = this.size - 1
//sampleEnd

fun main(args: Array<String>) {
    val list = listOf('a', 'b')
    // getter를 인라인함
    println("Last index of $list is ${list.lastIndex}")
}
```
</div>

또한 전체 프로퍼티에 `inline`을 붙일 수 있으며, 이 경우 두 접근자에 수식어를 적용한다.

자세한 내용은 [문서](inline-functions.html#inline-properties)와 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/inline-properties.md)을 참고한다.


### 로컬 위임 프로퍼티

이제 로컬 변수에 [위임 프로퍼티](delegated-properties.html) 구문을 사용할 수 있다.
한 가지 가능한 사용은 지연 평가되는 로컬 변수를 정의하는 것이다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
import java.util.Random

fun needAnswer() = Random().nextBoolean()

fun main(args: Array<String>) {
//sampleStart
    val answer by lazy {
        println("Calculating the answer...")
        42
    }
    if (needAnswer()) {                     // 임의 값을 리턴
        println("The answer is $answer.")   // 이 시점에 answer를 계산
    }
    else {
        println("Sometimes no answer is the answer...")
    }
//sampleEnd
}
```
</div>

자세한 내용은 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/local-delegated-properties.md)을 참고한다.


### 위임 프로퍼티 바인딩 인터셉션

[위임 프로퍼티](delegated-properties.html)에 대해 `provideDelegate` 연산자를 사용해서 프로퍼티 바인딩에 대한 위임을 가로챌 수 있다. 
예를 들어 바인딩 전에 프로퍼티 이름을 검사하고 싶다면 다음과 같이 작성할 수 있다:

``` kotlin
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(thisRef: MyUI, prop: KProperty<*>): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, prop.name)
        ... // 프로퍼티 생성
    }

    private fun checkProperty(thisRef: MyUI, name: String) { ... }
}

fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { ... }

class MyUI {
    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```

`MyUI` 인스턴스를 생성하는 동안 각 프로퍼티에 대해 `provideDelegate` 메서드를 호출하고,
바로 필요한 검증을 수행할 수 있다.

자세한 내용은 [문서](delegated-properties.html#providing-a-delegate-since-11)를 참고한다.


### 지네릭 enum 값 접근

이제 지네릭 방식으로 enum 클래스의 값을 차례로 열거할 수 있다.

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}
//sampleEnd

fun main(args: Array<String>) {
    printAllValues<RGB>() // RED, GREEN, BLUE 출력
}
```
</div>

### DSL에서 implicit 리시버를 위한 범위 제어

[`@DslMarker`](/api/latest/jvm/stdlib/kotlin/-dsl-marker/index.html) 애노테이션은 DSL 컨텍스트의 바깥 범위에서 리시버 사용을 제한한다
기준이 되는 [HTML 빌더 예제](type-safe-builders.html)를 보자:

``` kotlin
table {
    tr {
        td { +"Text" }
    }
}
```

코틀린 1.0에서 `td`에 전달한 람다에 있는 코드는 `table`에 전달된 것, `tr`에 전달된 것, `td`에 전달된 것
이렇게 세 개의 implicit 리시버에 접근한다. 이는 컨텍스트에서 의미없는 메서드를 호출할 수 있게 허용한다.
예를 들어 `td` 안에서 `tr`을 호출해서 `<td>`에 `<tr>` 태그를 넣는 것이 가능하다.

코틀린 1.1에서는 `td`의 implicit 리서버에 정의된 메서드는 오직 `td`에 전달된 람다에서만 접근할 수 있게 제한할 수 있다.
`@DslMarker` 메타 애노테이션을 지정한 애노테이션을 정의하고 이를 태그 클래스의 베이스 클래스에 적용하면 된다.

더 많은 정보는 [문서](type-safe-builders.html#scope-control-dslmarker-since-11)와 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/scope-control-for-implicit-receivers.md)을 참고한다.


### `rem` 연산자

`mod` 연산자를 디프리케이트했고 대신 `rem`을 사용한다. 이렇게 바꾼 동기는 [이슈](https://youtrack.jetbrains.com/issue/KT-14650)에서 확인할 수 있다.

## 표준 라이브러리

### String에서 숫자로의 변환

잘못된 숫자에 대해 익셉션 발생없이 문자열을 숫자로 변환할 수 있는 확장을 String에 추가했다:
`String.toIntOrNull(): Int?`, `String.toDoubleOrNull(): Double?` 등.

```
val port = System.getenv("PORT")?.toIntOrNull() ?: 80
```

또한 `Int.toString()`, `String.toInt()`, `String.toIntOrNull()`과 같은 정수 변환 함수도
`radix` 파라미터를 가진 오버로딩한 함수를 갖는다. 이 함수는 변환의 진수(2에서 36)를 지정할 수 있다.

### onEach()

`onEach`는 콜렉션과 시퀀스의 확장 함수로 작지만 유용하다., 
이 함수는 오퍼레이션 체인에서 콜렉션/시퀀스의 각 요소에 대해 어떤 액션-아마도 부수효과를 가진-을 수행할 수 있게 한다. 
Iterable에 대해 이 확장함수는 `forEach`처럼 동작하지만, 이어지는 Iterable 인스턴스를 리턴한다.
시퀀스의 경우 이 확장함수는 래핑한 시퀀스를 리턴하며, 요소를 반복할 때 주어진 액션을 지연시켜 적용한다.

``` kotlin
inputDir.walk()
        .filter { it.isFile && it.name.endsWith(".txt") }
        .onEach { println("Moving $it to $outputDir") }
        .forEach { moveFile(it, File(outputDir, it.toRelativeString(inputDir))) }
```

### also(), takeIf() 그리고 takeUnless()

모든 리시버에 적용할 수 있는 범용 목적의 확장 함수 세 개가 있다.

`also`는 `apply`와 유사하다. 리서버를 받고, 리시버에 어떤 행위를 하고, 리시버를 리턴한다.
차이는 `apply` 안의 블록에서는 리시버를 `this`로 접근 가능한데 반해 `also`의 블록에서는 `it`으로 접근 가능하다는 것이다(원한다면 다른 이름을 줄 수도 있다).
이는 바깥 범위의 `this`를 감추고 싶지 않을 때 쓸모 있다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
class Block {
    lateinit var content: String
}

//sampleStart
fun Block.copy() = Block().also {
    it.content = this.content
}
//sampleEnd

// 대신 'apply' 사용
fun Block.copy1() = Block().apply {
    this.content = this@copy1.content
}

fun main(args: Array<String>) {
    val block = Block().apply { content = "content" }
    val copy = block.copy()
    println("Testing the content was copied:")
    println(block.content == copy.content)
}
```
</div>

`takeIf`는 단일 값에 대해서 `filter`와 유사하다. 리시버가 조건을 충족하는지 검사하고 충족하면 리시버를 리턴하고 그렇지 않으면 `null`을 리턴한다
엘비스-연산자, 이른 리턴과 조합하면 다음과 같은 구성이 가능하다:

``` kotlin
val outDirFile = File(outputDir.path).takeIf { it.exists() } ?: return false
// 존재하는 outDirFile로 뭔가 하기
```

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
    val input = "Kotlin"
    val keyword = "in"

//sampleStart
    val index = input.indexOf(keyword).takeIf { it >= 0 } ?: error("keyword not found")
    // 입력 문자열에서 키워드를 발견하면 해당 키워드의 인덱스를 갖고 무언가 함
//sampleEnd
    
    println("'$keyword' was found in '$input'")
    println(input)
    println(" ".repeat(index) + "^")
}
```
</div>

`takeUnless`는 `takeIf`와 동일하다. 단 조건의 반대를 취한다. 조건을 충족하지 않을 때 리시버를 리턴하고 그렇지 않으면 `null`을 리턴한다. 위 예를 `takeUnless`로 재작성하면 다음과 같다:

``` kotlin
val index = input.indexOf(keyword).takeUnless { it < 0 } ?: error("keyword not found")
```

또한 람다 대신에 callable 레퍼런스를 함께 사용하면 편리하다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
private fun testTakeUnless(string: String) {
//sampleStart
    val result = string.takeUnless(String::isEmpty)
//sampleEnd

    println("string = \"$string\"; result = \"$result\"")
}

fun main(args: Array<String>) {
    testTakeUnless("")
    testTakeUnless("abc")
}
```
</div>

### groupingBy()

키를 이용해서 콜렉션을 그룹으로 나누고 동시에 그룹을 접을(fold) 때 이 API를 사용할 수 있다.
예를 들어, 다음과 같이 이 API를 이용해서 각 글자로 시작하는 단어의 수를 셀 수 있다:     

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
    val words = "one two three four five six seven eight nine ten".split(' ')
//sampleStart
    val frequencies = words.groupingBy { it.first() }.eachCount()
//sampleEnd
    println("Counting first letters: $frequencies.")

    // 'groupBy'와 'mapValues'를 사용하는 방법은 중간 맵을 만든다.
    // 반면에 'groupingBy'는 처리 중에 개수를 센다.
    val groupBy = words.groupBy { it.first() }.mapValues { (_, list) -> list.size }
    println("Comparing the result with using 'groupBy': ${groupBy == frequencies}.")
}
```
</div>

### Map.toMap()과 Map.toMutableMap()

이 함수를 이용하면 맵을 쉽게 복사할 수 있다:

``` kotlin
class ImmutablePropertyBag(map: Map<String, Any>) {
    private val mapCopy = map.toMap()
}
```

### Map.minus(key)

`plus` 연산자는 읽기 전용 맵에 키값 쌍을 추가해서 새로운 맵을 생성하는 방법을 제공하는데, 반대 기능은 간단한 방법이 없었다.
맵에서 키를 삭제하려면 `Map.filter()`나 `Map.filterKeys()`와 같이 직접적이지 않은 방법을 사용해야 했다.
이제는 `minus` 연산자를 사용하면 된다. 한 개 키 삭제, 키 콜렉션 삭제, 키의 시퀀스로 삭제, 키 배열로 삭제하는 4개의 연산자를 사용할 수 있다. 

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val map = mapOf("key" to 42)
    val emptyMap = map - "key"
//sampleEnd
    
    println("map: $map")
    println("emptyMap: $emptyMap")
}
```
</div>

### minOf() and maxOf()

이 함수는 두 개 이상의 값(기본 숫자 타입이나 `Comparable` 객체)에서 최소와 최대 값을 찾는다.
`Comparable`이 아닌 객체를 비교할 수 있도록 `Comparator` 인스턴스를 받는 함수도 추가로 제공한다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val list1 = listOf("a", "b")
    val list2 = listOf("x", "y", "z")
    val minSize = minOf(list1.size, list2.size)
    val longestList = maxOf(list1, list2, compareBy { it.size })
//sampleEnd
    
    println("minSize = $minSize")
    println("longestList = $longestList")
}
```
</div>

### 배열과 같은 리스트 생성 함수

`Array` 생성자와 유사하게, `List`와 `MutableList` 인스턴스를 생성하고 람다를 통해 각 요소를 초기화하는 함수를 제공한다:

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val squares = List(10) { index -> index * index }
    val mutable = MutableList(10) { 0 }
//sampleEnd

    println("squares: $squares")
    println("mutable: $mutable")
}
```
</div>

### Map.getValue()

`Map`에 대한 이 확장은 주어진 키가 존재하면 해당하는 값을 리턴하고 키가 없으면 익셉션을 발생한다.
`withDefault`를 사용해서 맵을 생성한 경우, 이 함수는 익셉션을 발생하는 대신에 기본 값을 리턴한다.

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {

//sampleStart    
    val map = mapOf("key" to 42)
    // 널일수 없는 Int 값 42를 리턴
    val value: Int = map.getValue("key")

    val mapWithDefault = map.withDefault { k -> k.length }
    // returns 4
    val value2 = mapWithDefault.getValue("key2")

    // map.getValue("anotherKey") // <- NoSuchElementException을 발생
//sampleEnd
    
    println("value is $value")
    println("value2 is $value2")
}
```
</div>

### 추상 콜렉션

코틀린 콜렉션 클래스를 구현할 때 추상 클래스를 기반 클래스로 사용할 수 있다.
읽기 전용 콜렉션을 구현할 때에는 `AbstractCollection`, `AbstractList`, `AbstractSet`, `AbstractMap`을 사용하며,
수정 가능 콜렉션은 `AbstractMutableCollection`, `AbstractMutableList`, `AbstractMutableSet`, `AbstractMutableMap`을 사용한다.
JVM에서 이들 수정 가능한 추상 콜렉션은 JDK의 추상 콜렉션의 대부분 기능을 상속한다.

### 배열 조작 함수

이제 표준 라이브러리로 배열의 요소간 연산을 위한 함수를 제공한다. 이 연산에는 비교(`contentEquals`와 `contentDeepEquals`),
해시 코드 계산(`contentHashCode`와 `contentDeepHashCode`), 문자열로 변환(`contentToString`와 `contentDeepToString`)이 있다.
JVM(`java.util.Arrays`에 대응하는 함수에 대한 별칭으로 동작)과 JS(코틀린 표준 라이브러리가 구현을 제공)에서 모두 지원한다.

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val array = arrayOf("a", "b", "c")
    println(array.toString())  // JVM implementation: type-and-hash gibberish
    println(array.contentToString())  // nicely formatted as list
//sampleEnd
}
```
</div>

## JVM 백엔드

### 자바 8 바이트코드 지원

이제 자바 8 바이트코드 생성 옵션(`-jvm-target 1.8` 명령행 옵션이나 대응하는 앤트/메이븐/그레이들 옵션)을 제공한다.
현재는 이 옵션이 바이트코드의 세만틱을 변경하지 않지만(특히 인터페이스의 디폴트 메서드와 람다를 코틀린 1.0처럼 생성한다),
향후에 이를 더 사용할 계획이다.


### 자바 8 표준 라이브러리 지원

이제 자바 7과 8에 추가된 새로운 JDK API를 지원하는 표준 라이브러리 버전을 따로 제공한다. 새 API에 접근하려면 표준 `kotlin-stdlib` 대신에
`kotlin-stdlib-jre7`와 `kotlin-stdlib-jre8` 메이븐 아티팩트를 사용하면 된다.
이 아티팩트는 `kotlin-stdlib`를 일부 확장한 것으로 의존성 전이로 `kotlin-stdlib`를 포함한다. 


### 바이트코드의 파라미터 이름

이제 바이트코드의 파라미터 이름 저장을 지원한다. `-java-parameters` 명령행 옵션을 사용해서 활성화할 수 있다.


### 상수 인라인

컴파일러는 이제 `const val` 프로퍼티의 값을 상수 사용 위치에 인라인한다.


### 수정가능 클로저 변수

람다에서 수정가능 클로저 변수를 캡처하기 위해 사용한 박스 클래스는 더 이상 volatile 필드를 갖지 않는다.
이 변화는 성능을 향상시키지만, 매우 드물게 새로운 경쟁 조건(race condition)을 유발할 수 있다.
경쟁 조건에 영향을 받는다면 변수에 접근할 때 자신만의 동기화를 제공해야 한다.


### javax.script 지원

코틀린은 이제 [javax.script API](https://docs.oracle.com/javase/8/docs/api/javax/script/package-summary.html)(JSR-223)와의 통합을 지원한다.
이 API를 사용하면 런타임에 코드를 평가할 수 있다:

``` kotlin
val engine = ScriptEngineManager().getEngineByExtension("kts")!!
engine.eval("val x = 3")
println(engine.eval("x + 2"))  // 5를 출력
```

이 API를 사용하는 더 큰 예제 프로젝트는 [여기](https://github.com/JetBrains/kotlin/tree/master/libraries/examples/kotlin-jsr223-local-example)를 참고한다.


### kotlin.reflect.full

[자바 9 지원을 준비하기](https://blog.jetbrains.com/kotlin/2017/01/kotlin-1-1-whats-coming-in-the-standard-library/) 위해
`kotlin-reflect.jar` 라이브러리에 있는 확장 함수와 프로퍼티를 `kotlin.reflect.full`로 옮겼다.
이전 패키지(`kotlin.reflect`)의 이름을 디프리케이트했고 코틀린 1.2에서는 삭제할 것이다.
(`KClass`와 같은) 핵심 리플렉션 인터페이스는 `kotlin-reflect`가 아닌 코틀린 표준 라이브러리에 속하므로 이동에 영향을 받지 않는다.


## 자바스크립트 백엔드

### 표준 라이브러리 통합

자바스크립트로 컴파일한 코드에서 코틀린 표준 라이브러리의 더 많은 부분을 사용할 수 있다.
콜렉션(`ArrayList`, `HashMap` 등), 익셉션(`IllegalArgumentException`), 몇 가지 다른 것(`StringBuilder`, `Comparator`) 등의 핵심 클래스가
이제 `kotlin` 패키지 아래에 정의되어 있다.
JVM에서 이들 이름은 대응하는 JDK 클래스에 대한 타입 별칭이며 JS의 경우 코틀린 표준 라이브러리에 클래스를 구현했다.

### 더 나아진 코드 생성

자바스크립트 백엔드는 이제 더욱 정적인 검사 가능한 코드를 생성하며, minifier나 최적화 linter 등의 JS 코드 처리 도구에 친화적이다.

### `external` 수식어

타입에 안전한 방법으로 자바스크립트로 정의한 클래스를 코틀린에서 접근하고 싶다면,
`external` 수식어를 사용해서 코틀린 선언을 작성할 수 있다. (코틀린 1.0에서는 `@native` 애노테이션을 사용했다.)
JVM 대상과 달리 JS 대상은 클래스와 프로퍼티에 `external` 수식어를 사용하는 것을 허용한다.
예를 들어 다음은 DOM `Node` 클래스에 선언한 예를 보여준다:

``` kotlin
external class Node {
    val firstChild: Node

    fun appendChild(child: Node): Node

    fun removeChild(child: Node): Node

    // etc
}
```

### import 처리 개선

자바스크립트 모듈에 정확하게 임포트될 수 있도록 선언을 제공할 수 있다.
external 선언에 `@JsModule("<module-name>")` 애노테이션을 추가하면, 컴파일 과정에서 모듈 시스템(CommonJS나 AMD)에 알맞게 임포트할 수 있다.
예를 들어 CommonJS를 사용하면 `require(...)` 함수로 선언을 임포트한다.
추가로 모듈이나 글로벌 자바스크립트 객체로 선언을 임포트하고 싶다면 `@JsNonModule` 애노테이션을 사용한다.

다음은 JQuery를 코틀린 모듈로 임포트한 예를 보여준다:

``` kotlin
external interface JQuery {
    fun toggle(duration: Int = definedExternally): JQuery
    fun click(handler: (Event) -> Unit): JQuery
}

@JsModule("jquery")
@JsNonModule
@JsName("$")
external fun jquery(selector: String): JQuery
```

이 예에서 JQuery를 이름이 `jquery`인 모듈로 임포트한다. 코틀린 컴파일러가 사용하기로 설정한 모듈 시스템에 따라
대안으로 $-객체를 사용해서 접근할 수 있다

어플리케이션에서 선언을 다음과 같이 사용할 수 있다:  

``` kotlin
fun main(args: Array<String>) {
    jquery(".toggle-button").click {
        jquery(".toggle-panel").toggle(300)
    }
}
```
