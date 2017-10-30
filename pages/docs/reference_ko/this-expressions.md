---
type: doc
layout: reference_ko
category: "Syntax"
title: "this 식"
---

# this 식

현재 _리시버_를 지정하려면 *this*{: .keyword } 식을 사용한다.

* [클래스](classes.html#inheritance)의 멤버에서 *this*{: .keyword }는 그 클래스의 현재 객체를 참조한다.
* [확장 함수](extensions.html)나 [리시버를 가진 함수 리터럴](lambdas.html#function-literals-with-receiver)에서
  *this*{: .keyword }는 점의 왼쪽 편에 전달한 _리시버_ 파라미터를 지정한다.

*this*{: .keyword }에 한정자가 없으면 _가장 안쪽을 둘러썬 범위_를 참조한다.
다른 범위의 *this*{: .keyword }를 참조하려면 _라벨 한정자_를 사용한다.

## 한정된 *this*{: .keyword }
{:#qualified}

외부 범위([클래스](classes.html)나 [확장 함수](extensions.html), 또는 
라벨이 있는 [리시버를 가진 함수 리터럴](lambdas.html#function-literals-with-receiver))에서 *this*{: .keyword }에 접근하려면
`this@label` 형식을 사용한다. 여기서 `@label`은 *this*{: .keyword }가 속한 범위를 의미하는 [라벨](returns.html)이다:

``` kotlin
class A { // 암묵적으로 @A 라벨 사용
    inner class B { // 암묵적으로 @B 라벨 사용
        fun Int.foo() { // 암묵적으로 @foo 라벨 사용
            val a = this@A // A의 this
            val b = this@B // B의 this

            val c = this // foo()의 리시버인 Int
            val c1 = this@foo // foo()'의 리시버인 Int

            val funLit = lambda@ fun String.() {
                val d = this // funLit의 리시버
            }


            val funLit2 = { s: String ->
                // 둘러싼 람다식에 리시버가 없으므로 
                // foo()의 리시버
                val d1 = this
            }
        }
    }
}
```
