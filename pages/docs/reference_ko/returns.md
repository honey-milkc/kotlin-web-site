---
type: doc
layout: reference
category: "Syntax"
title: "리턴과 점프: break와 continue"
---

# 리턴과 점프

코틀린에는 세 가지 구조의 점프 식이 있다.

* *return*{: .keyword }: 기본으로 가장 가깝게 둘러싼 함수나 [익명 함수](lambdas.html#anonymous-functions)에서 리턴한다.
* *break*{: .keyword }: 가장 가깝게 둘러싼 루프를 끝낸다.
* *continue*{: .keyword }: 가장 가깝게 둘러싼 루프의 다음 단계를 진행한다.

세 식 모두 더 큰 식의 일부로 사용할 수 있다:

``` kotlin
val s = person.name ?: return
```

이 세 식의 타입은 [Nothing](exceptions.html#the-nothing-type)이다.

## break와 continue 라벨

코틀린의 모든 식에 *label*{: .keyword }을 붙일 수 있다.
라벨은 `@` 부호 뒤에 식별자가 붙는 형식으로, 예를 들어 `abc@`, `fooBar@`는 유효한 라벨이다([문법](grammar.html#labelReference) 참고).
식 앞에 라벨을 위치시켜 라벨을 붙인다.

``` kotlin
loop@ for (i in 1..100) {
    // ...
}
```

이제 라벨을 사용해서 *break*{: .keyword }나 a *continue*{: .keyword }를 한정할 수 있다:

``` kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (...) break@loop
    }
}
```

라벨로 한정한 *break*{: .keyword }는 해당 라벨이 붙은 루프 이후로 실행 지점을 점프한다.
*continue*{: .keyword }는 루프의 다음 반복을 진행한다.


## 라벨에 리턴하기

코틀린은 함수 리터럴, 로컬 함수, 객체 식에서 함수를 중첩할 수 있는데,
한정한 *return*{: .keyword }을 사용하면 바깥 함수로부터 리턴할 수 있다.
가장 중요한 용도는 람다 식에서 리턴하는 것이다. 아래 코드를 보자:

``` kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return  // 내부 람다에서 foo()의 콜러로 바로 리턴하는 비로컬 리턴 
        print(it)
    }
}
```

*return*{: .keyword }-식은 가장 가깝게 둘러싼 함수(예, `foo`)에서 리턴한다.
(이런 비로컬 리턴은 [인라인 함수](inline-functions.html))로 전달한 람다 식에서만 지원한다.)
람다 식에서 리턴하고 싶다면 담다 식에 라벨을 붙여 *return*{: .keyword }을 한정해야 한다:

``` kotlin
fun foo() {
    ints.forEach lit@ {
        if (it == 0) return@lit
        print(it)
    }
}
```

이제 위 코드는 람다 식에서 리턴한다. 종종 암묵적으로 제공하는 라벨을 사용하는 게 편리할 때가 있다.
이런 라벨은 람다를 전달한 함수와 같은 이름을 갖는다. 

``` kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

람다 식 대신 [익명 함수](lambdas.html#anonymous-functions)를 사용해도 된다.
익명 함수에서 *return*{: .keyword } 문장은 익명 함수 자체에서 리턴한다.

``` kotlin
fun foo() {
    ints.forEach(fun(value: Int) {
        if (value == 0) return  // 익명 함수 호출에 대한 로컬 리턴. 예, forEach 루프로 리턴
        print(value)
    })
}
```

값을 리턴할 때 파서는 한정한 리턴에 우선순위를 준다.

``` kotlin
return@a 1
```

이 코드는 "라벨 `@a`에 `1`을 리턴"하는 것을 의미한다. "라벨을 붙인 식 `(@a 1)`을 리턴"하는 것이 아니다.
