---
type: doc
layout: reference_ko
category: "Syntax"
title: "위임"
---

# 위임

## 클래스 위임


[위임 패턴](https://en.wikipedia.org/wiki/Delegation_pattern)은 구현 상속보다 좋은 대안임이 증명됐다.
코틀린은 중복 코드없는 위임 패턴을 지원한다.
아래 코드에서 `Derived` 클래스는 `Base` 인터페이스를 상속할 수 있으며,
모든 public 메서드를 지정한 객체로 위임한다:

``` kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).print() // 10 출력
}
```

`Derived`의 상위타입 목록에 있는 *by*{: .keyword } 절은
`Derived`의 객체 내부에 `b`를 저장한다는 것을 의미한다.
컴파일러는 `Base`의 모든 메서드를 `b`로 전달하도록 `Derived`를 생성한다.

override는 그대로 동작한다. `override`가 존재하면 컴파일러는 위임 객체의 메서드 대신 `override` 구현을 사용한다.
`Derived`에 `override fun print() { print("abc") }`를 추가하면
프로그램은 "10" 대신 "abc"를 출력한다.
