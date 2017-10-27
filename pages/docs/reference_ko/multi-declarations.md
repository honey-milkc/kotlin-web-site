---
type: doc
layout: reference_ko
category: "Other"
title: "분리 선언(Destructuring Declarations)"
---

# 분리 선언(Destructuring Declarations)

때때로 다음 예처럼 객체를 여러 변수에 분리해서 할당하는 것이 편리할 때가 있다.

``` kotlin
val (name, age) = person 
```

이런 구문을 _분리 선언(destructuring declaration)_이라고 부른다.
분리 선언은 한 번에 여러 변수를 생성한다.
`name`과 `age`의 두 변수를 선언했고 각자 따로 사용할 수 있다:
 
``` kotlin
println(name)
println(age)
```

분리 선언은 다음과 같은 코드로 컴파일된다:

``` kotlin
val name = person.component1()
val age = person.component2()
```

`component1()`과  `component2()` 함수는 코틀린에서 광범위하게 사용하는 _관례 규칙(principle of conventions)_ 
예다(`+`와 `*`, *for*{: .keyword }-루프 등의 연산자를 보자).
필요한 개수의 component 함수를 호출할 수만 있으면 무엇이든 분리 선언의 오른쪽에 위치할 수 있다. 
물론 `component3()`과 `component4()` 등이 존재할 수 있다.

`componentN()` 함수에 `operator` 키워드를 붙여야 분리 선언에서 그 함수를 사용할 수 있다.

분리 선언은 또한 다음과 같이 *for*{: .keyword } 루프에서도 동작한다:

``` kotlin
for ((a, b) in collection) { ... }
```

변수 `a`와 `b`는 콜렉션 요소의 `component1()`과 `component2()` 함수가 리턴한 값을 구한다.

## 에제: 함수에 두 값 리턴하기

함수에 두 값을 리턴하고 싶다고 하자. 예를 들어 어떤 종류의 결과 객체와 상태를 리턴해야 한다고 가정하자.
코틀린에서 이를 간략하게 처리하는 방법은 [_데이터 클래스_](data-classes.html)를 선언하고
그 클래스의 인스턴스를 리턴하는 것이다: 
 
``` kotlin
data class Result(val result: Int, val status: Status)
fun function(...): Result {
    // computations
    
    return Result(result, status)
}

// 이제 이 함수를 사용한다
val (result, status) = function(...)
```

데이터 클래스는 자동으로 `componentN()` 함수를 선언하므로 여기서 분리 선언이 작동한다.

**주의**: 위 `function()`에서 표준 클래스 `Pair`를 사용한 `Pair<Int, Status>`를 리턴해도 된다. 하지만
데이터에 적당한 이름을 붙이는게 더 나을 때가 있다.  

## 예제: 분린 선언과 맵

맵을 순회하는 가장 나은 방법은 아마도 다음일 것이다:

``` kotlin
for ((key, value) in map) {
   // 키와 값으로 무엇을 함
}
```

이게 작동하려면 다음을 충족해야 한다 

* 맵에서 값의 시퀀스로 제공하는 `iterator()` 함수를 제공한다
* 각 요소를 `component1()`와 `component2()` 함수를 제공하는 Pair로 제공한다 

그리고 사실 표준 라이브러리는 그런 확장을 제공한다:

``` kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```  

따라서 맵을 사용한(또한 데이터 클래스 인스턴스의 콜렉션을 사용한) *for*{: .keyword } 루프에서 분린 선언을 자유롭게 사용할 수 있다.  

## 사용하지 않는 변수를 위한 밑줄 (1.1부터)

분리 선언에서 변수가 필요없으면 이름 대신에 밑줄을 사용할 수 있다:

``` kotlin
val (_, status) = getResult()
```

이 방식으로 생략한 component에 대해 `componentN()` 연산자 함수를 호출하지 않는다.

## 람다에서의 분리 선언 (1.1부터)
{:#destructuring-in-lambdas-since-11}

람다 파라미터에도 분린 선언 구문을 사용할 수 있다.
람다가 `Pair` 타입의 파라미터를 가지면(또는 `Map.Entry`, 또는 적당한 `componentN` 함수를 가진 다른 타입),
괄호 안에 Pair 타입 파라미터를 넣는 대신 분리한 새 파라미터를 사용할 수 있다:

``` kotlin
map.mapValues { entry -> "${entry.value}!" }
map.mapValues { (key, value) -> "$value!" }
```

두 파라미터를 선언하는 것과 한 개 파라미터 대신에 분리한 쌍을 선언하는 것의 차이를 보자.

``` kotlin
{ a -> ... } // 한 개 파라미터
{ a, b -> ... } // 두 개 파라미터
{ (a, b) -> ... } // 분리한 쌍
{ (a, b), c -> ... } // 분리한 쌍과 다른 파라미터
```

분리한 파라미터의 컴포넌트를 사용하지 않으면, 이름을 짓지 않고 밑줄로 대체할 수 있다:

``` kotlin
map.mapValues { (_, value) -> "$value!" }
```

분리한 파라미터 전체 타입이나 일부 컴포넌트의 타입을 개별적으로 지정할 수 있다:

``` kotlin
map.mapValues { (_, value): Map.Entry<Int, String> -> "$value!" }

map.mapValues { (_, value: String) -> "$value!" }
```
