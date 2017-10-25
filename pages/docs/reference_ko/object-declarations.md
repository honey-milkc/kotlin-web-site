---
type: doc
layout: reference
category: "Syntax"
title: "오브젝트 식, 오브젝트 선언, 컴페니언 오브젝트"
---

# 오브젝트 식과 선언

때때로 하위클래스를 새로 만들지 않고 특정 클래스를 약간 수정한 객체를 만들고 싶을 때가 있다.
자바에서는 *익명 내부 클래스*를 사용해서 이런 경우를 처리한다.
코틀린은 *오브젝트 식*과 *오브젝트 선언*으로 이 개념을 일부 일반화했다.

## 오브젝트 식

특정 타입을 상속한 익명 클래스의 객체를 생성하려면 다음과 같이 작성한다: 

``` kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

상위타입이 생성자를 가지면, 알맞은 생성자 파라미터를 전달해야 한다.
상위타입이 여러 개면, 콜론 뒤에 콤마로 구분해서 지정한다:


``` kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B {...}

val ab: A = object : A(1), B {
    override val y = 15
}
```

만약 별도 상위타입 없이 "단지 객체"가 필요한거라면, 다음과 같이 단순히 생성할 수 있다:

``` kotlin
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}
```

익명 객체는 로컬과 private 선언에서만 타입으로 사용할 수 있다.
public 함수의 리턴 타입이나 public 프로퍼티의 타입으로 익명 객체를 사용하면,
그 함수나 프로퍼티의 실제 타입은 익명 객체에 선언한 상위 타입이 된다.
상위 타입을 선언하지 않았다면 `Any`가 된다.
익명 객체에 추가한 멤버는 접근하지 못한다.

``` kotlin
class C {
    // private 함수이므로 리턴 타입은 익명 객체 타입이다  
    private fun foo() = object {
        val x: String = "x"
    }

    // public 함수이므로 리턴 타입은 Any이다 
    fun publicFoo() = object {
        val x: String = "x"
    }

    fun bar() {
        val x1 = foo().x        // 동작
        val x2 = publicFoo().x  // 에러: 레퍼런스'x'를 찾을 수 없음
    }
}
```

자바의 익명 내부 클래스와 마찬가지로, 오브젝트 식의 코드는 둘러싼 범위의 변수에 접근할 수 있다.
(자바와 달리, final 변수로 제한되지 않는다.)

``` kotlin
fun countClicks(window: JComponent) {
    var clickCount = 0
    var enterCount = 0

    window.addMouseListener(object : MouseAdapter() {
        override fun mouseClicked(e: MouseEvent) {
            clickCount++
        }

        override fun mouseEntered(e: MouseEvent) {
            enterCount++
        }
    })
    // ...
}
```

## 오브젝트 선언

[싱글톤](http://en.wikipedia.org/wiki/Singleton_pattern)은 매우 유용한 패턴이며,
코틀린은 싱글톤을 쉽게 선언할 수 있다(스칼라를 따라했다):

``` kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ...
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ...
}
```

이를 *오브젝트 선언*이라고 부른다. 오브젝트 선언에는 *object*{: .keyword } 키워드 뒤에 이름이 위치한다.
변수 선언처럼, 오브젝트 선언은 식이 아니며 할당 문장의 오른쪽에 사용할 수 없다.

오브젝트를 참조하려면 이름을 직접 사용한다:

``` kotlin
DataProviderManager.registerDataProvider(...)
```

오브젝트는 상위타입을 가질 수 있다:

``` kotlin
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }

    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
}
```

**주의**: 오브젝트 선언은 로컬일 수 없다(예, 함수 안에 바로 중첩할 수 없다),
하지만, 다른 오브젝트 선언이나 내부가 아닌 클래스에는 중첩할 수 있다.


### 컴페니언 오브젝트

클래스 내부의 오브젝트 선언은 *companion*{: .keyword } 키워드를 붙일 수 있다:

``` kotlin
class MyClass {
    companion object Factory {
        fun create(): MyClass = MyClass()
    }
}
```

컴페니언 오브젝트의 멤버는 클래스 이름을 한정자로 사용해서 호출할 수 있다:

``` kotlin
val instance = MyClass.create()
```

컴페니언 오브젝트의 이름은 생략이 가능하며, 이 경우 이름으로 `Companion`을 사용한다:

``` kotlin
class MyClass {
    companion object {
    }
}

val x = MyClass.Companion
```

컴페니언 오브젝트의 멤버가 다른 언어의 정적 멤버처럼 보이겠지만,
런타임에 컴페니언 오브젝트는 실제 객체의 인스턴스 멤버이으몰, 따라서 인터페이스를 구현할 수 있다:

``` kotlin
interface Factory<T> {
    fun create(): T
}


class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
```

하지만 JVM에서 `@JvmStatic` 애노테이션을 사용하면 컴페니언 오브젝트의 멤버를 실제 정적 메서드와 필드로 생성한다.
더 자세한 내용은 [자바 상호운용성](java-to-kotlin-interop.html#static-fields)을 참고한다.


### 오브젝트 식과 선언의 의미 차이

오브젝트 식과 오브젝트 선언 사이에는 중요한 의미 차이가 있다:

* 오브젝트 식은 사용한 위치에서 **즉시** 초기화되고 실행된다.
* 오브젝트 선언은 처음 접근할 때까지 초기화를 **지연**한다. 
* 컴페니언 오브젝트는 대응하는 클래스를 로딩할 때 초기화된다. 이는 자바의 정적 초기화와 동일하다.
