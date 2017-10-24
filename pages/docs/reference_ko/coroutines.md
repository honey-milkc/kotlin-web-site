---
type: doc
layout: reference
category: "Syntax"
title: "코루틴"
---

# 코루틴

> 코틀린 1.1에서 코루틴은 *실험* 버전이다. 자세한 내용은 [아래](#experimental-status-of-coroutines)를 참고한다. 
{:.note}

일부 API는 길게 실행되는 오퍼레이션을(네트워크 IO, 파일 IO, CPU나 GPU에 집중된 작업 등) 시작하고,
끝날때까지 호출자가 멈출 것을 요구한다.
코루틴은 쓰레드 차단을 피하고 더 싸고 더 제어가능한 오퍼레이션으로 대체하는 방법을 제공한다.
그 오퍼레이션은 코루틴의 *suspension*이다.

코루틴은 복잡함을 라이브러리에 넣어서 비동기 프로그래밍을 단순하게 한다.
코루틴을 사용하면 프로그램 로직을 *순차적으로* 표현할 수 있는데 하위 라이브러리는 이를 비동기로 해석한다.

라이브러리가 사용자 코드의 관련 부분을 콜백으로 감싸고, 관련 이벤트를 구독하고, 다른 쓰레드로 실행 스케줄을 잡지만(심지어 다른 머신에서),
코드는 마치 순차적으로 실행하는 것처럼 단순한 형태를 유지한다.

코틀린의 코루타을 이용해서 다른 언어에서 사용할 수 있는 다양한 비동기 기법을 라이브러리로 구현할 수 있다.
이런 라이브러리에는 C#과 ECMAScript에서 온
[`async`/`await`](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#composing-suspending-functions), 
Go에서 온 [채널](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#channels)과 [`select`](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#select-expression),
C#과 파이썬에서 온 [제너레이터/`yield`](#generators-api-in-kotlincoroutines)가 있다.
그런 구성 요소를 제공하는 라이브러리에 대한 설명은 [아래](#standard-apis)를 참고한다.

## 블로킹 vs 연기

기본적으로 코루틴은 *쓰레드 블로킹*없이 *연기*할 수 있는 계산이다.
쓰레드 블로킹은 비싸다. 특히 부하가 높은 상황에서 상대적으로 작은 개수의 쓰레드만 실용적으로 사용되기 때문에
그 쓰레드 중 하나를 블록킹하면 일부 중요한 작업을 지연시키게 된다. 

반면에 코루틴 서스펜션은 거의 비용이 없다. 컨텍스트 스위치나 OS의 관련된 다른 것이 필요없다.
그 기반하에 사용자 라이브러리는 높은 정도로 서스펜션을 제어할 수 있다.
라이브러리 작성자로서 우리는 우리 요구에 따라 최적화/로그/가로채기와 서스펜션이 발생하도록 결정했다.(?)

또 다른 차이점은 코루틴은 임의 명령어를 연기할 수 없다는 것이다.
코루틴은 *서스펜션 포인트*라고 불리는 곳만 연기할 수 있다. 서스펜션 포인트는 특별히 표시한 함수를 호출하는 지점이다.

## 서스펜딩 함수(suspending function)

서스펜션은 `suspend`라는 특별한 수식어를 붙인 함수를 호출할 때 발생한다:

``` kotlin 
suspend fun doSomething(foo: Foo): Bar {
    ...
}
```

이런 함수를 *서스펜딩 함수(suspending function)*라고 부르는데, 이 함수를 호출하면 코루틴을 연기하기 때문이다(라이브러리는
함수 호출에 대한 결과가 이미 사용가능하면, 서스펜션없이 진행할지 여부를 결정할 수 있다.) 
서스펜딩 함수는 일반 함수와 같은 방법으로 파라미터와 리턴 값을 가질 수 있다.
하지만 서스펜딩 함수느느 코루틴이나 다른 서스펜딩 함수에서만 호출할 수 있다.
사실 코루틴을 시작하려면, 적어도 한 개의 서스펜딩 함수가 있어야 하며
보통 임의 함수를 사용한다(즉, 서스펜딩 람다).
예제를 살펴보자. 다음은 단순화한 `async()` 함수이다([`kotlinx.coroutines`](#generators-api-in-kotlincoroutines) 라이브러리).
    
``` kotlin
fun <T> async(block: suspend () -> T)
``` 

여기서 `async()`는 일반 함수인데(서스펜딩 함수가 아님), `block` 파라미터는 `suspenc` 수식어를 가진 `suspend () -> T` 함수 타압이다.
람다를 `async()`에 전달하면, 그것은 *서스펜딩 람다*가 되고, 그 람다에서 서스펜딩 함수를 호출할 수 있다:
   
``` kotlin
async {
    doSomething(foo)
    ...
}
```

> **주의:** 현재, 서스펜딩 함수 타입은 상위 타입으로 사용할 수 없고, 임의 서스펜딩 함수는 지원하지 않는다.

공통점을 계속해보면, `await()`는 서스펜딩 함수일 수 있다(그래서 `aync {}` 블록 안에서 호출가능한),
이는 특정 계산이 끝나고 그 결고를 리턴할 때까지 코루틴을 연기한다:

``` kotlin
async {
    ...
    val result = computation.await()
    ...
}
```

`async/await` 함수가 실제로 어떻게 동작하는지에 대한 내용은 [여기](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md#composing-suspending-functions)를 참고한다.

서스펜딩 함수인 `await()`와 `doSomething()`는 `main()`과 같은 일반 함수에서 호출할 수 없다:

``` kotlin
fun main(args: Array<String>) {
    doSomething() // 에러: 코루틴 컨텍스트가 아닌 곳에서 서스펜딩 함수를 호출 
}
```

서스펜딩 함수는 가상일 수 있고, 그 함수를 오버라이딩할 때는 `suspend` 수식어를 명시해야 한다:
 
``` kotlin
interface Base {
    suspend fun foo()
}

class Derived: Base {
    override suspend fun foo() { ... }
}
``` 

### `@RestrictsSuspension` 애노테이션
 
확장 함수(그리고 람다)도 일반 함수처럼 `suspec`를 붙일 수 있다.
이는 사용자가 확장할 수 있는 다른 API와 [DSL](type-safe-builders.html)의 생성을 가능하게 한다.
어떤 경우에 라이브러리 제작자는 사용자가 서스펜딩 코루틴의 *새로운 방법*을 추가하는 금지해야 할 때가 있다.

이럴 때 [`@RestrictsSuspension`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-restricts-suspension/index.html) 애노테이션을 사용한다. 
리시버 클래스나 인터페이스 `R`에 이 애노테이션을 붙이면,
모든 서스펜딩 확장은 `R`의 멤버나 `R`에 대한 다른 확장으로 위임해야 한다.
확장을 무한히 다른 확장에 위임할 수 없기 때문에(프로그램이 끝나지 않는다),
이는 모든 서스펜션이 `R` 멤버의 호출을 통해서 일어나는 것을 보장하며, 이를 통해 라이브러리 제작자는 완전히 제어할 수 있다.

이는 모든 서스펜션을 라이브러리에 특별한 방식으로 처리하는 상대적으로 _드문_ 경우이다.
예를 들어 [아래]#generators-api-in-kotlincoroutines)에서 설명하는 
[`buildSequence()`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/build-sequence.html) 함수를 통해서
제너레이터를 구현할 때,
코루틴의 서스펜딩 호출이 다른 함수가 아닌 반드시 `yield()`나 `yieldAll()`로 끝나야 할 필요가 있다.
왜냐면 [`SequenceBuilder`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/-sequence-builder/index.html)는
`@RestrictsSuspension`을 달았기 때문이다.

``` kotlin
@RestrictsSuspension
public abstract class SequenceBuilder<in T> {
    ...
}
```
 
[Github에서](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/src/kotlin/coroutines/experimental/SequenceBuilder.kt)
소스를 참고하자.   

## 코루틴의 내부 작동

여기서 코루틴이 내부적으로 어떻게 동작하는지 완벽하게 설명하진 않겠지만, 어떤 식으로 진행되는지 대략 감을 잡는 것은 매우 중요하다.
코루틴은 완전히 컴파일 기법으로 구현하며(VM이나 OS의 지원이 필요없다), 서스펜션은 코드 변환을 통해 작동한다.
기본적으로 모든 서스펜딩 함수는(최적화를 적용하지만 여기서는 이에 대해선 언급하지 않는다) 서스펜딩 호출에 대한 상태를 갖는 상태 머신으로 변환된다.
서스펜션 직전에 컴파일러가 생성한 클래스의 필드에 다음 상태를 관련된 로컬 변수와 함께 저장한다.
코루틴을 재개할 때, 로컬 변수를 복원하고 서스펜션 이후의 상태에서 상태 머신을 진행한다.

연기된 코루틴은 연기한 상태와 로컬을 유지하는 객체에 전달해서 저장할 수 있다.
그런 객체의 타입이 `Continuation`이며 여기서 설명한 전반적인 코드 변환은 고전적인 [Continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style)에 해당한다.
결과적으로 서스펜딩 함수는 실제 구현에서 `Continuation`의 추가 파라미터를 갖는다.

코루틴이 자세한 동작 방식은 [설계 문서](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)를 참고한다.

(C#이나 ECMAScript 2016와 같은) 다른 언어의 async/await의 설명은 여기 설명과 관련이 있지만,
다른 언거가 구현한 언어 특징은 코틀린 코루틴만큼 일반적이지 않다. 

## 코루틴의 실험 상태

코루틴 설계는 [실험 단계](compatibility.html#experimental-features)로 앞으로 바뀔 수 있다.
코틀린 1.1에서 코루틴을 컴파일할 때 기본적으로 *The feature "coroutines" is experimental*라는 경고를 출력한다.
이 경고를 없애고 싶으면 [opt-in 플래그](/docs/diagnostics/experimental-coroutines.html)를 지정하면 된다.
최종적으로 설계가 끝나고 실험 상태에서 승격되면 마지막 API를 `kotlin.coroutines`고
하위호환성을 위해 실험용 패키지는 (아마도 별도 아티팩트로 분리해서) 유지할 것이다.

**중요 사항**: 라이브러리 제작자가 같은 규칙을 따를 것을 권한다. 코루틴 기반 API를 노출하는 패키지에는
"experimental"(예 `com.example.experimental`)를 추가해서 라이브러리의 바이너리 호환성을 유지하자.
최종 API를 릴리즈할 때 다음 과정을 따른다:
 * 모든 API를 `com.example`에 복사한다(experimental 접미사없이),
 * 하위 호환성을 위해 experimental 패키지를 유지한다. 
 
이는 라이브러리 사용자의 마이그레이션 이슈를 최소화할 것이다. 

## 표준 API
 
코루틴에는 세 가지 주요 요소가 있다:
Coroutines come in three main ingredients: 
 - 언어 지원(예, 앞에서 설명한 서스펜딩 함수)
 - 코틀린 표준 라이브러리의 저수준 핵심 API
 - 사용자 코드에서 직접 사용할 상위수준 API
 
### 저수준 API: `kotlin.coroutines` 

저수준 API는 상대적으로 작고 고수준 라이브러리를 작성하는 것 외에 다른 용도로 사용하면 안 된다.
저수준 API는 두 개의 주요 패키지로 구성된다:
- [`kotlin.coroutines.experimental`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/index.html)는 다음과 같은 주요 타입과 기본 요소를 가짐 
  - [`createCoroutine()`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/create-coroutine.html),
  - [`startCoroutine()`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/start-coroutine.html),
  - [`suspendCoroutine()`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/suspend-coroutine.html);
- [`kotlin.coroutines.experimental.intrinsics`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental.intrinsics/index.html) 
  [`suspendCoroutineOrReturn`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental.intrinsics/suspend-coroutine-or-return.html)과 같은
  저수준 intrinsic(본질?)을 가짐
 
이들 API에 대한 자세한 사용법은 [여기](https://github.com/Kotlin/kotlin-coroutines/blob/master/kotlin-coroutines-informal.md)를 참고한다.

### `kotlin.coroutines`의 제너레이터 API
  
  `kotlin.coroutines.experimental`의 유일한 "어플리케이션 수준" 함수는 다음과 같다:
- [`buildSequence()`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/build-sequence.html)
- [`buildIterator()`](/api/latest/jvm/stdlib/kotlin.coroutines.experimental/build-iterator.html)

이들 함수는 시퀀스와 관련이 있어서 `kotlin-stdlib`에 들어가 있다. 사실 이들 함수는(여기서는 `buildSequence()`로 제한할 수 있다) _제너레이터_를 구현해서
lazy 시퀀스를 싼 비용으로 구축할 수 있는 방법을 제공한다.
 
<div class="sample" markdown="1" data-min-compiler-version="1.1"> 

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
    val fibonacciSeq = buildSequence {
        var a = 0
        var b = 1
        
        yield(1)
        
        while (true) {
            yield(a + b)
            
            val tmp = a + b
            a = b
            b = tmp
        }
    }
//sampleEnd

    // 처음 8개의 피보나치 숫자를 출력
    println(fibonacciSeq.take(8).toList())
}
```

</div>


이는 지연된, 잠재적으로 무한 피보나치 시퀀스를 만든다. 이 시퀀스는
`yield()` 함수를 호출해서 연속된 피보나치 숫자를 생성하는 코루틴을 생성해서 만든다.

그 시퀀스를 반복할 때 이터레이터의 각 단계마다 다음 숫자를 생성하는 코루틴의 다른 부분을 실행한다.
그래서 이 시퀀스로부터 유한한 숫자 목록을 취할 수 있다.
예를 들어 `fibonacciSeq.take(8).toList()`의 결과는 `[1, 1, 2, 3, 5, 8, 13, 21]`이 된다.
그리고 코루틴은 충분히 싸서 실제로 쓸모 있게 만든다.

이런 시퀀스가 실제로 지연되는지 보기 위해, `buildSequence()` 호출 안에 디버그 문구를 출력해보자:
  
<div class="sample" markdown="1" data-min-compiler-version="1.1"> 

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
    val lazySeq = buildSequence {
        print("START ")
        for (i in 1..5) {
            yield(i)
            print("STEP ")
        }
        print("END")
    }

    // 시퀀스의 처음 세 개 요소를 출력
    lazySeq.take(3).forEach { print("$it ") }
//sampleEnd
}
```

</div>  
   
위 코드를 실행하면 처음 세 개의 요소를 출력한다. 생성 루프에서 `STEP`과 숫자가 교차로 출력된다.
이는 계산이 실제로 지연됨을 의미한다.
`1`을 출력하기 위해 `START` 출려과 함께 첫 번째 `yield(i)`까지만 실행한다.
그리고 `2`를 출력하기 위해 다음 `yield(i)`까지 진행하고 `STEP`을 출력한다. `3`도 같으며
다음 `STEP`을(또한 `END`를) 출력하지 않는다. 왜냐면 시퀀스의 다음 요소를 요청하지 않았기 때문이다.

한 번에 값의 콜렉션(또는 시퀀스)를 생성하려면, `yieldAll()` 함수를 사용한다:

<div class="sample" markdown="1" data-min-compiler-version="1.1"> 

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
    val lazySeq = buildSequence {
        yield(0)
        yieldAll(1..10) 
    }

    lazySeq.forEach { print("$it ") }
//sampleEnd
}
```

</div>  

`buildIterator()`는 `buildSequence()`와 비슷하게 동작하는데, lazy 이터레이터를 리턴한다.

`SequenceBuilder` 클래스(이는 [위에서](#restrictssuspension-annotation) 설명한 `@RestrictsSuspension` 애노테이션을 가능하게 한다)에 
대한 서스펜딩 확장을 작성하면 `buildSequence()`에 커스텀 생성 로직을 추가할 수 있다.

<div class="sample" markdown="1" data-min-compiler-version="1.1"> 

``` kotlin
import kotlin.coroutines.experimental.*

//sampleStart
suspend fun SequenceBuilder<Int>.yieldIfOdd(x: Int) {
    if (x % 2 != 0) yield(x)
}

val lazySeq = buildSequence {
    for (i in 1..10) yieldIfOdd(i)
}
//sampleEnd

fun main(args: Array<String>) {
    lazySeq.forEach { print("$it ") }
}
```

</div>  
  
### 다른 고수준 API: `kotlinx.coroutines`

오직 코루틴과 관련된 핵심 API는 코루틴 표준 라이브러리에서만 가능하다.
이는 거의 모든 코루틴 기반 라이브러리가 사용하는 핵심 프리미티브와 인터페이스로 구성된다.

코루틴에 기반한 대부분의 어플리케이션 수준 API는 별도 라이브러리인 [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines)로 릴리즈된다.
이 라이브러리는 다음을 다룬다:
 * `kotlinx-coroutines-core`를 이용한 플랫폼에 무관한 비동기 프로그래밍
   * 이 모듈은 Go와 같은 채널을 포함하며 `select`와 다른 편리한 프리미티브를 지원한다.
   * 이 라이브러리에 대한 종합적인 안내는 [여기](https://github.com/Kotlin/kotlinx.coroutines/blob/master/coroutines-guide.md)에서 확인할 수 있다.
 * JDK 8의 `CompletableFuture`에 기반한 API: `kotlinx-coroutines-jdk8`
 * 자바 7 또는 상위 버전의 API에 기반한 논블로킹 IO (NIO): `kotlinx-coroutines-nio`
 * Swing 지원(`kotlinx-coroutines-swing`)과 JavaFx 지원(`kotlinx-coroutines-javafx`)
 * RxJava 지원: `kotlinx-coroutines-rx`.

이 라이브러리는 공용 작업을 쉽게 만들어주는 편리한 API와 코루틴 기반 라이브러리를 작성하는 방법을 보여주는 완전한 예제를 제공한다.
