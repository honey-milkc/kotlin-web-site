---
type: doc
layout: reference_ko
category: "Syntax"
title: "고차 함수와 람다"
---

# 고차 함수와 람다

## 고차 함수

고차 함수는 함수를 파라미터로 받거나 함수를 리턴하는 함수이다.
고차 함수의 좋은 예로 `lock()`이 있다. 이 함수는 Lock 객체와 함수를 받아서,
Lock을 얻고, 그 함수를 실행하고, Lock을 해제한다:

``` kotlin
fun <T> lock(lock: Lock, body: () -> T): T {
    lock.lock()
    try {
        return body()
    }
    finally {
        lock.unlock()
    }
}
```

위 코드를 보자. `body`는 [함수 타입](#function-types)인 `() -> T`이다.
이 함수 타입은 파라미터가 없고 `T` 타입을 리턴하는 함수이다.
`lock`으로 보호하는 동안 *try*{: .keyword } 블록에서 이 함수를 호출하고 그 결과를 `lock()` 함수의 결과로 리턴한다.

`lock()`을 호출할 때 인자로 다른 함수를 전달할 수 있다([함수 레퍼런스](reflection.html#function-references) 참고):

``` kotlin
fun toBeSynchronized() = sharedResource.operation()

val result = lock(lock, ::toBeSynchronized)
```

보통 더 간편한 방법은 [람다 식](#lambda-expressions-and-anonymous-functions)을 전달하는 것이다:

``` kotlin
val result = lock(lock, { sharedResource.operation() })
```
람다 식에 대한 자세한 내용은 [아래에서 설명한다](#lambda-expressions-and-anonymous-functions).
하지만 이 절을 계속하기 위해 간단하게 람다 식의 개요를 정리했다.

* 람다 식은 항상 증괄호로 둘러 싼다.
* `->` 앞에 파라미터를 선언한다. (파라미터 타입은 생략할 수 있다.)
* `->` 뒤에 몸체가 온다(몸체가 존재할 때).

코틀린에서 함수의 마지막 파라미터가 함수면, 괄호 밖에서 람다 식을 인자로 전달할 수 있다.

``` kotlin
lock (lock) {
    sharedResource.operation()
}
```

고차 함수의 다른 예는 `map()`이다:

``` kotlin
fun <T, R> List<T>.map(transform: (T) -> R): List<R> {
    val result = arrayListOf<R>()
    for (item in this)
        result.add(transform(item))
    return result
}
```

이 함수는 다음과 같이 부를 수 있다:

``` kotlin
val doubled = ints.map { value -> value * 2 }
```

함수를 호출할 때 인자가 람다뿐이면 괄호를 완전히 생략할 수 있다.

### `it`: 단일 파라미터의 암묵적 이름
{:#it-implicit-name-of-a-single-parameter}

다른 유용한 한 가지 규칙은 함수 리터럴의 파라미터가 한 개면, (`->`를 포함한) 파라미터 선언을 생략할 수 있고,
파라미터 이름이 `it`이 된다는 것이다:

``` kotlin
ints.map { it * 2 }
```

이 규칙은 [LINQ-방식](http://msdn.microsoft.com/en-us/library/bb308959.aspx) 코드를 작성할 수 있게 한다:

``` kotlin
strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```

### 사용하지 않는 변수의 밑줄 표기 (1.1부터)
{:#underscore-for-unused-variables-since-11}

람다의 파라미터를 사용하지 않으면 이름 대신에 밑줄을 사용할 수 있다:

``` kotlin
map.forEach { _, value -> println("$value!") }
```

### 람다에서의 분리 선언 (1.1부터)

람다에서의 분리 선언은 [분리 선언](multi-declarations.html#destructuring-in-lambdas-since-11)에서 설명한다. 

## 인라인 함수

때때로 고차 함수의 성능을 향상시키기 위해 [인라인 함수](inline-functions.html)를 사용하는 것이 이익이 된다.

## 람다 식과 익명 함수
{:#lambda-expressions-and-anonymous-functions}

람다 식 또는 익명 함수는 함수 선언 없이 바로 식으로 전달한 함수인 "함수 리터럴"이다. 다음 예를 보자:

``` kotlin
max(strings, { a, b -> a.length < b.length })
```

`max` 함수는 고차 함수로서 두 번째 인자로 함수 값을 취한다.
두 번째 인자는 자체가 함수인 식으로 함수 리터럴이다. 다음 함수와 동등하다:

``` kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### 함수 타입
{:#function-types}

함수가 다른 함수를 파라미터로 받을 때, 파라미터를 위한 함수 타입을 지정해야 한다.
예를 들어, 앞서 언급한 `max` 함수는 다음과 같이 정의했다:

``` kotlin
fun <T> max(collection: Collection<T>, less: (T, T) -> Boolean): T? {
    var max: T? = null
    for (it in collection)
        if (max == null || less(max, it))
            max = it
    return max
}
```

`less` 파라미터는 `(T, T) -> Boolean` 타입인데, 이 함수는 `T` 타입의 파라미터가 두 개이고 `Boolean`을 리턴한다.
`less`는 첫 번째가 두 번째보다 작으면 true를 리턴한다.

위 코드의 4번째 줄에서 `less`를 함수로 사용한다. `T` 타입의 두 인자를 전달해서 `less`를 호출한다.

함수 타입을 위와 같이 작성하거나 또는 각 파라미터의 의미를 문서화하고 싶다면 파라미터에 이름을 붙일 수 있다.

``` kotlin
val compare: (x: T, y: T) -> Int = ...
```

함수 타입을 null 가능 변수로 선언하려면 전체 함수 타입을 괄호로 감싸고 그 뒤에 물음표룔 붙인다:

``` kotlin
var sum: ((Int, Int) -> Int)? = null
```


### 람다 식 구문
{:#lambda-expression-syntax}

람다 식(즉 함수 타입의 리터럴)의 완전한 구문 형식은 다음과 같다:

``` kotlin
val sum = { x: Int, y: Int -> x + y }
```

람다 식은 항상 중괄호로 감싼다. 완전한 구문 형식에서 파라미터 선언은 중괄호 안에 위치하고 선택적으로 타입을 표시한다.
몸체는 `->` 부호 뒤에 온다. 람다의 추정한 리턴 타입이 `Unit`이 아니면 람다 몸체의 마지막(또는 단일) 식을 리턴 값으로 처리한다.

모든 선택 사항을 생략하면 람다 식은 다음과 같이 보인다:

``` kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

람다 식이 파라미터가 한 개인 경우는 흔하다.
코틀린이 시그너처 자체를 알아낼 수 있다면, 한 개인 파라미터를 선언하지 않는 것이 가능하며
`it`이라는 이름으로 파라미터를 암묵적으로 선언한다: 

``` kotlin
ints.filter { it > 0 } // 이 리터럴은 '(it: Int) -> Boolean' 타입이다.
```

[한정한 리턴](returns.html#return-at-labels) 구문을 사용해서 람다에서 값을 명시적으로 리턴할 수 있다.
그렇지 않으면 마지막 식의 값을 암묵적으로 리턴한다.
따라서 다음의 두 코드는 동일하다:

``` kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```

함수가 다른 함수를 마지막 파라미터로 받으면 괄호로 둘러싼 인자 목록 밖에 람다 식 인자를 전달할 수 있음에 주목하자. 
[callSuffix](http://kotlinlang.org/docs/reference/grammar.html#callSuffix)에 대한 문법을 참고한다.

### 익명 함수
{:#anonymous-functions}

위에서 보여준 람다 식 구문에서 빼 먹은 한 가지는 함수의 리턴 타입을 지정할 수 있다는 점이다.
많은 경우, 리턴 타입을 자동으로 유추할 수 있기 때문에 이것이 불필요하다.
하지만, 명확하게 리턴 타입을 지정하고 싶다면 다른 구문인 _익명 함수_를 사용할 수 있다.

``` kotlin
fun(x: Int, y: Int): Int = x + y
```

익명 함수는 일반 함수 선언과 비슷해 보인다. 차이점은 이름을 생략했다는 점이다.
몸체는 (위에서 보이는) 식이거나 블록일 수 있다:

``` kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

파라미터와 리턴 타입은 일반 함수와 같은 방법으로 지정한다.
문맥에서 파라미터 타입을 유추할 수 있다면 생략할 수 있다는 점은 다르다.

``` kotlin
ints.filter(fun(item) = item > 0)
```

익명 함수에 대한 리턴 타입 추론은 보통 함수와 동일하게 동작한다.
몸체가 식인 익명 함수에 대한 리턴 타입은 자동으로 유추한다.
블록 몸체를 가진 익명 함수의 경우 명시적으로 리턴 타입을 지정해야 한다(아니면 `Unit`으로 가정).

익명 함수 파라미터는 항상 괄호 안에 전달해야 한다.
괄호 밖에 함수 전달을 허용하는 간단 구문은 람다 식에 대해서만 동작한다.

람다 식과 익명 함수 간의 한 가지 다른 차이점은 [비로컬 리턴](inline-functions.html#non-local-returns)의 동작 방식이다.
라벨이 없는 *return*{: .keyword } 문장은 항상 *fun*{: .keyword } 키워드로 선언한 함수에서 리턴한다.
이는 람다 식의 *return*{: .keyword }은 둘러싼 함수를 리턴하는 반면
익명 함수의 *return*{: .keyword }은 익명 함수 자체로부터 리턴한다는 것을 의미한다.

### 클로저

람다 식 또는 익명 함수(그리고 [로컬 함수](functions.html#local-functions)와 [오브젝트 식](object-declarations.html#object-expressions))은
그것의 _클로저_에, 즉 외부 범위에 선언된 변수에 접근할 수 있다.
자바와 달리 클로저에 캡처한 변수를 수정할 수 있다:

``` kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```


### 리시버를 가진 함수 리터럴
{:#function-literals-with-receiver}

코틀린은 _리시버 객체_를 가진 함수 리터럴을 호출하는 기능을 제공한다.
함수 리터럴 몸체 안에서 별도 한정자 없이 리시버 객체의 메서드를 호출할 수 있다.
이는 함수 몸체 안에서 리서버 객체의 멤버에 접근할 수 있는 확장 함수와 유사하다.
이의 중요한 예제 중 하나는 [타입에 안전한 그루비 방식 빌더](type-safe-builders.html)이다.

이런 함수 리터럴의 타입은 리시버를 가진 함수 타입이다:

``` kotlin
sum : Int.(other: Int) -> Int
```

마치 리시버 객체에 메서드가 존재하는 것처럼 함수 리터럴을 호출할 수 있다:

``` kotlin
1.sum(2)
```

익명 함수 구문은 함수 리터럴의 리시버 타입을 직접 지정할 수 있게 한다.
이는 리시버를 사용한 함수 타입의 변수를 선언하고 나중에 사용해야 할 때 유용하다.

``` kotlin
val sum = fun Int.(other: Int): Int = this + other
```

리시버 타입을 갖는 함수의 비-리터럴 값을 할당하거나 
추가로 리시버 타입의 *첫 번째* 파라미터를 가진 걸로 기대하는 일반 함수의 인자로 전달할 수 있다.
예를 들어, `String.(Int) -> Boolean`과 `(String, Int) -> Boolean`은 호환된다:

``` kotlin
val represents: String.(Int) -> Boolean = { other -> toIntOrNull() == other }
println("123".represents(123)) // true

fun testOperation(op: (String, Int) -> Boolean, a: String, b: Int, c: Boolean) =
    assert(op(a, b) == c)
    
testOperation(represents, "100", 100, true) // OK
```

문맥에서 리시버 타입을 추론할 수 있는 경우 람다 식을 리시버를 가진 함수 리터럴로 사용할 수 있다.

``` kotlin
class HTML {
    fun body() { ... }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // 리시버 객체 생성
    html.init()        // 리시버 객체를 람다에 전달
    return html
}


html {       // 리시버로 시작하는 람다
    body()   // 리시버 객체에 대한 메서드 호출
}
```


