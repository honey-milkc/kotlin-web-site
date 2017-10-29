---
type: doc
layout: reference_ko
category: "Syntax"
title: "패키지와 임포트"
---

# 패키지

소스 파일은 패키지 선언으로 시작한다:

``` kotlin
package foo.bar

fun baz() {}

class Goo {}

// ...
```

소스 파일의 모든 내용(클래스와 함수 등)은 선언된 패키지에 포함된다.
위 예의 경우 `baz()`의 전체 이름은 `foo.bar.baz`이고 `Goo`의 전체 이름은 `foo.bar.Goo`이다.

패키지를 지정하지 않으면, 파일의 내용은 이름이 없는 "디폴트" 패키지에 속한다.

## 디폴트 임포트

모든 코틀린 파일은 다음 패키지를 기본적으로 임포트한다:

- [kotlin.*](/api/latest/jvm/stdlib/kotlin/index.html)
- [kotlin.annotation.*](/api/latest/jvm/stdlib/kotlin.annotation/index.html)
- [kotlin.collections.*](/api/latest/jvm/stdlib/kotlin.collections/index.html)
- [kotlin.comparisons.*](/api/latest/jvm/stdlib/kotlin.comparisons/index.html)  (1.1부터)
- [kotlin.io.*](/api/latest/jvm/stdlib/kotlin.io/index.html)
- [kotlin.ranges.*](/api/latest/jvm/stdlib/kotlin.ranges/index.html)
- [kotlin.sequences.*](/api/latest/jvm/stdlib/kotlin.sequences/index.html)
- [kotlin.text.*](/api/latest/jvm/stdlib/kotlin.text/index.html)

대상 플랫폼에 따라 추가 패키지를 임포트한다:

- JVM:
  - java.lang.*
  - [kotlin.jvm.*](/api/latest/jvm/stdlib/kotlin.jvm/index.html)

- JS:    
  - [kotlin.js.*](/api/latest/jvm/stdlib/kotlin.js/index.html)

{:#imports}

## 임포트

디폴트 임포트에 추가로 각 파일은 자신만의 import 디렉티브를 포함할 수 있다.
import 문법은 [문법](http://kotlinlang.org/docs/reference/grammar.html#import)에서 설명한다.

한 개 이름을 임포트하는 예:

``` kotlin
import foo.Bar // 완전한 이름이 아닌 Bar로 접근 가능
```

범위(패키지, 클래스, 오브젝트 등)의 모든 내용에 접근 가능:

``` kotlin
import foo.* // 'foo'의 모든 것에 접근 가능
```

이름 충돌이 발생하면, *as*{: .keyword } 키워드로 충돌하는 대상에 다른 이름을 부여해서 모호함을 없앨 수 있다:

``` kotlin
import foo.Bar // Bar로 접근 가능
import bar.Bar as bBar // bBar로 'bar.Bar'에 접근 가능
```

`import` 키워드는 클래스뿐만 아니라 다음 선언을 임포트할 수 있다:

  * 최상위 함수와 프로퍼티;
  * [오브젝트 선언](object-declarations.html#object-declarations)에 있는 함수와 프로퍼티;
  * [enum 상수](enum-classes.html).

자바와 달리 코틀린은 별도의 ["import static"](https://docs.oracle.com/javase/8/docs/technotes/guides/language/static-import.html) 구문이 없다.
`import` 키워드로 모든 선언을 임포트할 수 있다.

## 최상위 선언의 가시성

최상위 선언을 *private*{: .keyword }로 선언한 경우, 선언된 파일에 대해 private 이다([가시성 수식어](visibility-modifiers.html)를 보자).
