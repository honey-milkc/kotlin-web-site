---
type: doc
layout: reference
category: "Syntax"
title: "타입에 안전한 그루비 방식 빌더"
---

# 타입에 안전한 빌더

*그루비* 커뮤니티에서 [빌더](http://www.groovy-lang.org/dsls.html#_nodebuilder) 개념은 유명하다.
빌더는 반정도 선언적인 방식으로 데이터를 정의할 수 있도록 한다.
[XML 생성](http://www.groovy-lang.org/processing-xml.html#_creating_xml),
[UI 컴포넌트 배치](http://www.groovy-lang.org/swing.html),
[3D 장면 설명](http://www.artima.com/weblogs/viewpost.jsp?thread=296081) 등이 빌더의 좋은 예이다. 

코틀린은 많은 사용처를 위해 *타입에 안전한* 빌더를 허용한다. 심지어 이 빌더는 그루비 자체에서 만든
동적 타입 구현보다 더 매력적이다.

코틀린은 동적 타입 빌더를 제공해서 타입에 안전한 빌더로 하지 못하는 경우도 지원한다.

## 타입에 안전한 빌더 예제

다음 코드를 보자:

``` kotlin
import com.example.html.* // 아래 선언을 보자

fun result(args: Array<String>) =
    html {
        head {
            title {+"XML encoding with Kotlin"}
        }
        body {
            h1 {+"XML encoding with Kotlin"}
            p  {+"this format can be used as an alternative markup to XML"}

            // 속성과 텍스트 내용을 가진 요소
            a(href = "http://kotlinlang.org") {+"Kotlin"}

            // 혼합 내용
            p {
                +"This is some"
                b {+"mixed"}
                +"text. For more see the"
                a(href = "http://kotlinlang.org") {+"Kotlin"}
                +"project"
            }
            p {+"some text"}

            // 생성한 내용
            p {
                for (arg in args)
                    +arg
            }
        }
    }
```

이 코드는 완전히 올바른 코틀린 코드이다.
이 코드를 [여기](http://try.kotlinlang.org/#/Examples/Longer examples/HTML Builder/HTML Builder.kt)에서
실행해 볼 수 있다(브라우저에서 수정하고 실행해보자).

## 동작 방식

코틀린의 타입에 안전한 빌더를 구현하는 메커니즘을 살펴보자.
먼저 생성하고 싶은 모델을 정의해야 한다. 이 예에서는 HTML 태그를 위한 모델이 필요하다.
약간의 클래스로 쉽게 모델을 정의할 수 있다.
예를 들어, `<html>` 태그를 위한 클래스는 `HTML`이다. 이는 `<head>`와 `<body>`와 같은 자식을 정의한다.
([아래](#full-definition-of-the-comexamplehtml-package) 클래스 선언을 참고한다.)

이제 코드에서 다음과 같이 호출할 수 있는 이유를 살펴보자:

``` kotlin
html {
 // ...
}
```

`html`은 실제로는 인자로 [람다 식](lambdas.html)을 받는 함수 호출이다.
임 함수는 다음과 같이 정의되어 있다:

``` kotlin
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```

이 함수는 `init`이란 이름의 파라미터를 가지며, 이 파라미터 자체는 함수이다.
이 함수의 타입은 `HTML.() -> Unit`인데, 이는 _리시버를 가진 함수 타입_이다.
이는 함수에 `HTML`(_리시버_) 타입의 인스턴스를 전달해야 하고 함수 안에서 그 인스턴스의 멤버를 호출할 수 있다는 것을 의미한다.
*this*{: .keyword } 키워드로 리시버에 접근할 수 있다:

``` kotlin
html {
    this.head { /* ... */ }
    this.body { /* ... */ }
}
```

(`head`와 `body`는 `HTML`의 멤버 함수이다)

*this*{: .keyword }를 생략할 수 있고, 이미 빌더처럼 보이는 것을 얻었다:

``` kotlin
html {
    head { /* ... */ }
    body { /* ... */ }
}
```

그러면 이것은 무엇을 호출할까? 위에서 정의한 `html` 함수의 몸체를 보자.
함수는 `HTML` 인스턴스를 생성하고 인자로 전달받은 함수를 호출해서 그 인스턴스를 초기화하고(이 예에서는
`HTML` 인스턴스의 `head`와 `body`를 호출한다), 인스턴스를 리턴한다. 이것이 정확하게 빌더가 해야 하는 일이다.

`HTML`의 `head`와 `body` 함수도 `html`과 유사하게 정의한다.
유일한 차이점은 두 함수는 둘러싼 `HTML` 인스턴스의 `children` 콜렉션에 생성한 인스턴스를 추가한다는 것이다:

``` kotlin
fun head(init: Head.() -> Unit) : Head {
    val head = Head()
    head.init()
    children.add(head)
    return head
}

fun body(init: Body.() -> Unit) : Body {
    val body = Body()
    body.init()
    children.add(body)
    return body
}
```

실제로 이 두 함수는 같은 것을 하므로, 지네릭 버전인 `initTag`를 만들 수 있다:

``` kotlin
protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
    tag.init()
    children.add(tag)
    return tag
}
```

이제 두 함수가 매우 단순해진다:

``` kotlin
fun head(init: Head.() -> Unit) = initTag(Head(), init)

fun body(init: Body.() -> Unit) = initTag(Body(), init)
```

그리고, `<head>`와 `<body>` 태그를 생성하기 위해 두 함수를 사용할 수 있다.

여기서 논의할 만한 다른 한 가지는 어떻게 몸체에 텍스트를 넣느냐는 것이다. 예제 코드에서는 다음과 같이 했다:

``` kotlin
html {
    head {
        title {+"XML encoding with Kotlin"}
    }
    // ...
}
```

기본적으로 태그 몸체에 단지 문자열을 넣었다. 하지만, 문자열에 앞에 `+`가 붙어 있는데,
이는 접두사 `unaryPlus()` 오퍼레이션을 호출하는 함수 호출이다.
이 오퍼레이션은 실제로 `TagWithText` 추상 클래스(`Title`의 부모)의 멤버인 `unaryPlus()` 확장 함수로 정의되어 있다. 

``` kotlin
fun String.unaryPlus() {
    children.add(TextElement(this))
}
```

그래서 접두사 `+`는 문자열을 `TextElement` 인스턴스로 감싸고, `children` 콜렉션에 추가하며,
따라서 태그 트리의 알맞은 부분이 된다.

지금까지 보여준 코드는 빌더 예제의 상단에서 임포트한 `com.example.html` 패키지에 정의했다.
마지막 절에서 이 패키지 전체를 볼 수 있다.

## 범위 제어: @DslMarker (1.1부터)

DSL을 사용할 때, 그 문백에서 호출할 수 있는 함수가 너무 많아질 수 있는 문제가 발생할 수 있다.
람다 안에서 모든 사용가능한 암묵적인 리서버의 메서드를 호출할 수 있고,
그 결과로 다른 `head` 안에 `head`를 넣는 것고 같이 일관성이 깨지는 결과를 얻을 수 있다:

``` kotlin
html {
    head {
        head {} // 금지해야 한다
    }
    // ...
}
```

이 예에서는 가장 가까운 암묵적인 리시버인 `this@head`의 멤버만 가능해야 한다.
`head()`는 외부 리서버인 `this@html`의 멤버이므로, `head()`를 호출하는 것은 막아야 한다.

이 문제를 해결하기 위해, 코틀린 1.1은 리시버 범위를 제어하기 위한 특별한 메커니즘을 추가했다.
컴파일러가 범위 제어를 시작하도록 하려면, DLS에서 사용할 모든 리시버 타입에 동일한 마커 애노테이션을 붙여야 한다.
예를 들어, HTML 빌더의 경우 `@HTMLTagMarker` 애노테이션을 선언한다:
 
``` kotlin
@DslMarker
annotation class HtmlTagMarker
```

`@DslMarker` 애노테이션이 붙은 애노테이션 클래스를 DLS 마커라고 부른다.

DLS에서 모든 태그 클래스는 같은 상위 클래스인 `Tag`를 상속한다.
그 상위클래스에만 `@HtmlTagMarker`를 붙여도 충분하며,
그 뒤로 코틀린 컴파일러는 모든 상속한 클래스를 `@HtmlTagMarker`로 다룬다:

``` kotlin
@HtmlTagMarker
abstract class Tag(val name: String) { ... }
```

`HTML`나 `Head` 클래스에 `@HtmlTagMarker`를 붙이면 안 되는데, 그 이유는 상위클래스에 이미 붙였기 때문이다:

```
class HTML() : Tag("html") { ... }
class Head() : Tag("head") { ... }
```

이 애노테이션을 추가하면, 코틀린 컴파일러는 암묵적 리시버가 동일 DLS의 어느 부분인지 알 수 있게 되고,
가장 가까운 리시버의 멤버만 호출할 수 있게 만들 수 있다:: 

``` kotlin
html {
    head {
        head { } // 에러: 외부 리시버의 멤버
    }
    // ...
}
```

외부 리시버의 멤버를 호출하는 것은 여전히 가능하지만, 이를 하려면 명시적으로 리시버를 지정해야 한다:

``` kotlin
html {
    head {
        this@html.head { } // 가능
    }
    // ...
}
```

## `com.example.html` 패키지 전체 설명

이 절은 `com.example.html` 패키지를 어떻게 정의했는지 보여준다(위 예제에서 사용한 요소만 설명).
이 예는 HTML 트리를 생성한다. 이 예제는 [식 함수](extensions.html)와 
[리시버를 가진 람다](lambdas.html#function-literals-with-receiver)를 많이 사용한다.

`@DslMarker` 애노테이션은 코틀린 1.1부터 가능하다.

<a name='declarations'></a>

``` kotlin
package com.example.html

interface Element {
    fun render(builder: StringBuilder, indent: String)
}

class TextElement(val text: String) : Element {
    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent$text\n")
    }
}

@DslMarker
annotation class HtmlTagMarker

@HtmlTagMarker
abstract class Tag(val name: String) : Element {
    val children = arrayListOf<Element>()
    val attributes = hashMapOf<String, String>()

    protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }

    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent<$name${renderAttributes()}>\n")
        for (c in children) {
            c.render(builder, indent + "  ")
        }
        builder.append("$indent</$name>\n")
    }

    private fun renderAttributes(): String {
        val builder = StringBuilder()
        for ((attr, value) in attributes) {
            builder.append(" $attr=\"$value\"")
        }
        return builder.toString()
    }

    override fun toString(): String {
        val builder = StringBuilder()
        render(builder, "")
        return builder.toString()
    }
}

abstract class TagWithText(name: String) : Tag(name) {
    operator fun String.unaryPlus() {
        children.add(TextElement(this))
    }
}

class HTML : TagWithText("html") {
    fun head(init: Head.() -> Unit) = initTag(Head(), init)

    fun body(init: Body.() -> Unit) = initTag(Body(), init)
}

class Head : TagWithText("head") {
    fun title(init: Title.() -> Unit) = initTag(Title(), init)
}

class Title : TagWithText("title")

abstract class BodyTag(name: String) : TagWithText(name) {
    fun b(init: B.() -> Unit) = initTag(B(), init)
    fun p(init: P.() -> Unit) = initTag(P(), init)
    fun h1(init: H1.() -> Unit) = initTag(H1(), init)
    fun a(href: String, init: A.() -> Unit) {
        val a = initTag(A(), init)
        a.href = href
    }
}

class Body : BodyTag("body")
class B : BodyTag("b")
class P : BodyTag("p")
class H1 : BodyTag("h1")

class A : BodyTag("a") {
    var href: String
        get() = attributes["href"]!!
        set(value) {
            attributes["href"] = value
        }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```
