---
type: doc
layout: reference_ko
category: "Syntax"
title: "익셉션: try, catch, finally, throw, Nothing"
---

# 익셉션

## 익셉션 클래스

코틀린의 모든 익셉션 클래스는 `Throwable` 클래스의 자식 클래스이다.
모든 익셉션은 메시지, 스택 트레이스, 선택적인 원인을 포함한다.

익셉션 객체를 던지려면, *throw*{: .keyword }-식을 사용한다:

``` kotlin
throw MyException("Hi There!")
```

익셉션을 잡으려면 *try*{: .keyword }-식을 사용한다:

``` kotlin
try {
    // some code
}
catch (e: SomeException) {
    // handler
}
finally {
    // optional finally block
}
```

*catch*{: .keyword } 블록은 없거나 여러 개 올 수 있다. *finally*{: .keyword } 블록은 생략할 수 있다.
하지만, 최소 한 개의 *catch*{: .keyword } 블록이나 *finally*{: .keyword } 블록은 존재해야 한다:

### try는 식이다

*try*{: .keyword }는 식이어서 값을 리턴할 수 있다:

``` kotlin
val a: Int? = try { parseInt(input) } catch (e: NumberFormatException) { null }
```

*try*{: .keyword }-식이 리턴한 값은 *try*{: .keyword } 블록의 마지막 식이거나
*catch*{: .keyword } 블록(또는 블록들의)의 마지막 식이다. 
*finally*{: .keyword } 블록의 내용은 식의 결과에 영향을 주지 않는다.

## Checked 익셉션

Kotlin에는 Checked 익셉션이 없다. 여기에는 여러 이유가 있는데, 간단한 예로 살펴보자.
Kotlin does not have checked exceptions. There are many reasons for this, but we will provide a simple example.

다음은 JDK의 `StringBuilder` 클래스가 구현한 인터페이스 예이다.

``` java
Appendable append(CharSequence csq) throws IOException;
```

이 시그너처는 무엇을 말할까? 이것은 어딘가에(`StringBuilder`, 어떤 종류의 로그, 콘솔 등) 문자열을 추가할 때마다
`IOExceptions`을 잡아야 한다고 말한다. 왜? 그것이 IO 연산을 할 수도 있기 때문이다(`Writer`도 `Appendable`를 구현하고 있다)...
이는 모든 곳에 다음과 같은 코드를 만든다:

``` kotlin
try {
    log.append(message)
}
catch (IOException e) {
    // 안전해야 함
}
```

이는 좋은 게 아니다. 이에 대한 내용은 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 65: *Don't ignore exceptions*를 참고하자.

Bruce Eckel은 [Does Java need Checked Exceptions?](http://www.mindview.net/Etc/Discussions/CheckedExceptions)에서 다음과 같이 말했다:

> 작은 프로그램에서의 실험 결과 익셉션 규약을 강제해하는 것이 개발 생산성과 코드 품질을 향상시킨다는 결론을 얻었다. 하지만, 대형 소프트웨어 프로젝트에서 경험한 결과는 다른 결론을 말한다. 생산성이 떨어지고 코드 품질에서 거의 향상이 이루어지지 않았다.

다음은 이와 관련된 다른 논의이다:

* [Java's checked exceptions were a mistake](http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html) (Rod Waldhoff)
* [The Trouble with Checked Exceptions](http://www.artima.com/intv/handcuffs.html) (Anders Hejlsberg)

## Nothing 타입
{:#the-nothing-type}

코틀린에서 `throw`는 식이므로, 식을 사용할 수 있다. 예를 들어, 엘비스 식에서 사용할 수 있다:

``` kotlin
val s = person.name ?: throw IllegalArgumentException("Name required")
```

`throw` 식의 타입은 특수 타입인 `Nothing`이다.
이 타입은 값을 갖지 않으면 도달할 수 없는 코드를 표시하는 용도로 사용한다.
코드에서 리턴하지 않는 함수를 표시할 때 `Nothing`을 사용할 수 있다:

``` kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

이 함수를 호출하면, 컴파일러는 호출 이후 실행이 계속되지 않는 것을 알 것이다:

``` kotlin
val s = person.name ?: fail("Name required")
println(s)     // 이 시점에 's'가 초기화된 것을 안다
```

Nothing 타입을 만나는 또 다른 경우는 타입 추론이다. Nothing의 nullable 가능 타입인 `Nothing?`은
정확하게 `null` 한 가지 값만 가진다. `null`을 사용해서 추론할 타입의 값을 초기화하면
더 구체적인 타입을 결정할 수 있는 다른 정보가 없기 때문에 컴파일러는 `Nothing?` 타입으로 유추한다: 


``` kotlin
val x = null           // 'x'는 `Nothing?` 타입을 가짐
val l = listOf(null)   // 'l'은 `List<Nothing?> 타입을 가짐
```

## 자바 상호운용성

익센션의 자바 상호운용성에 대한 내용은 [자바 상호운용 섹션](java-interop.html)에서 확인할 수 있다.
