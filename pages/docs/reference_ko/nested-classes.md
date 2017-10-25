---
type: doc
layout: reference
category: "Syntax"
title: "중첩 클래스와 내부 클래스"
---

# 중첩 클래스와 내부 클래스

다른 클래스에 클래스를 중첩할 수 있다:

``` kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

## 내부 클래스

*inner*{: .keyword }로 지정한 클래스는 외부 클래스의 멤버에 접근할 수 있다.
내부 클래스는 외부 클래스의 객체에 대한 레퍼런스를 갖는다:

``` kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

내부 클래스에서 *this*{: .keyword }에 대한 모호함에 대한 것은
[한정된 *this*{: .keyword } 식](this-expressions.html)을 참고한다.

## 익명 내부 클래스

[object 식](object-declarations.html#object-expressions)을 사용해서
익명 내부 클래스를 생성할 수 있다:
                                                      
``` kotlin
window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }
                                                                                                            
    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

객체가 함수형 자바 인터페이스의 인스턴스라면(예를 들어 한 개의 추상 메서드를 가진 자바 인터페이스),
인터페이스 타입을 접두어로 갖는 람다 식을 사용해서 익명 내부 객체를 생성할 수 있다: 

``` kotlin
val listener = ActionListener { println("clicked") }
```
