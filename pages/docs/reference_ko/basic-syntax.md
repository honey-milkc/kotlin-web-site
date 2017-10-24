---
type: doc
layout: reference
category: "Basics"
title: "기초 구문"
---

# 기초 구문

## 패키지 정의

패키지 정의는 소스 파일의 처음에 위치해야 한다:

``` kotlin
package my.demo

import java.util.*

// ...
```

디렉토리와 패키지가 일치할 필요는 없으며, 파일 시스템 어디든 소스 파일을 위치시킬 수 있다.

[패키지](packages.html)를 보자.

## 함수 정의

두 개의 `Int` 파라미터와 리턴 타입이 `Int`인 함수:

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun sum(a: Int, b: Int): Int {
    return a + b
}
//sampleEnd

fun main(args: Array<String>) {
    print("sum of 3 and 5 is ")
    println(sum(3, 5))
}
```
</div>

식 몸체(expression body)를 사용하고 리턴 타입 추론:

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun sum(a: Int, b: Int) = a + b
//sampleEnd

fun main(args: Array<String>) {
    println("sum of 19 and 23 is ${sum(19, 23)}")
}
```
</div>

의미없는 값을 리턴하는 함수:

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main(args: Array<String>) {
    printSum(-1, 8)
}
```
</div>

`Unit` 리턴 타입은 생략 가능:

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
//sampleEnd

fun main(args: Array<String>) {
    printSum(-1, 8)
}
```
</div>

[함수](functions.html)를 보자.

## 로컬 변수 정의

한번 할당(읽기 전용) 로컬 변수:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val a: Int = 1  // 특시 할당
    val b = 2   // `Int` 타입으로 추론
    val c: Int  // 초기화를 하지 않으면 타입 필요
    c = 3       // 할당 연기
//sampleEnd
    println("a = $a, b = $b, c = $c")
}
```
</div>

변경가능 변수:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    var x = 5 // `Int` 타입 추론
    x += 1
//sampleEnd
    println("x = $x")
}
```
</div>

또한 [프로퍼티오 필드](properties.html)를 참고하자.


## 주석

자바나 자바스크립트 마찬가지로 코틀린은 주석 블록과 줄 주석을 지원한다,

``` kotlin
// 이는 줄 주석

/* 이는 여러 줄에 걸친
   주석 블록이다. */
```

자바와 달리 코틀린은 블록 주석을 중첩할 수 있다.

문서화 주석 구문은 [코틀린 코드 문서화](kotlin-doc.html) 문서를 참고한다.

## 문자열 템플릿 사용

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    var a = 1
    // 템플릿에서 단순 이름 사용:
    val s1 = "a is $a" 
    
    a = 2
    // 템플릿에서 임의의 식 사용:
    val s2 = "${s1.replace("is", "was")}, but now is $a"
//sampleEnd
    println(s2)
}
```
</div>

[문자열 템플릿](basic-types.html#string-templates)을 참고한다.

## 조건 식 사용


<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
//sampleEnd

fun main(args: Array<String>) {
    println("max of 0 and 42 is ${maxOf(0, 42)}")
}
```
</div>


*if*{: .keyword }를 식으로 사용하기:

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun maxOf(a: Int, b: Int) = if (a > b) a else b
//sampleEnd

fun main(args: Array<String>) {
    println("max of 0 and 42 is ${maxOf(0, 42)}")
}
```
</div>

[*if*{: .keyword }-식](control-flow.html#if-expression)을 참고한다.

## nullable 값 사용과 *null*{: .keyword } 검사

*null*{: .keyword } 값이 가능할 때 반드시 레퍼런스를 명시적으로 nullable로 표시해야 한다.

아래 코드가 `str`이 정수를 갖지 않으면 *null*{: .keyword }을 리턴한다고 할 때: 

``` kotlin
fun parseInt(str: String): Int? {
    // ...
}
```

nullable 값을 리턴하는 함수를 다음과 같이 사용한다:


<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

//sampleStart
fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)

    // `x * y` 코드는 에러를 발생하는데, 이유는 null을 가질 수 있기 때문이다.
    if (x != null && y != null) {
        // null 검사 이후에 x와 y를 자동으로 non-nullable로 변환
        println(x * y)
    }
    else {
        println("either '$arg1' or '$arg2' is not a number")
    }    
}
//sampleEnd


fun main(args: Array<String>) {
    printProduct("6", "7")
    printProduct("a", "7")
    printProduct("a", "b")
}
```
</div>

또는


<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun parseInt(str: String): Int? {
    return str.toIntOrNull()
}

fun printProduct(arg1: String, arg2: String) {
    val x = parseInt(arg1)
    val y = parseInt(arg2)
    
//sampleStart
    // ...
    if (x == null) {
        println("Wrong number format in arg1: '$arg1'")
        return
    }
    if (y == null) {
        println("Wrong number format in arg2: '$arg2'")
        return
    }

    // null 검사 이후에 x와 y를 자동으로 non-nullable로 변환
    println(x * y)
//sampleEnd
}

fun main(args: Array<String>) {
    printProduct("6", "7")
    printProduct("a", "7")
    printProduct("99", "b")
}
```
</div>

[Null-안전성](null-safety.html)을 참고한다.

## 타입 검사와 자동 변환 사용

*is*{: .keyword } 연산자는 식이 타입 인스턴스인지 검사한다.
불변 로컬 변수나 프로퍼티를 특정 타입에 대해 검사할 경우 명시적으로 타입 변환할 필요가 없다:


<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // 이 블록에서는 `obj`를 자동으로 `String`으로 변환
        return obj.length
    }

    // 타입 검사 블록 밖에서 `obj`는 여전히 `Any` 타입
    return null
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")
    printLength(1000)
    printLength(listOf(Any()))
}
```
</div>

또는

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj`를 자동으로 `String`으로 변환
    return obj.length
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, not a string"} ")
    }
    printLength("Incomprehensibilities")
    printLength(1000)
    printLength(listOf(Any()))
}
```
</div>

또는 심지어 다음도 가능

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun getStringLength(obj: Any): Int? {
    // 우측의 `&&`에서 `obj`를 자동으로 `String`으로 변환
    if (obj is String && obj.length > 0) {
        return obj.length
    }

    return null
}
//sampleEnd


fun main(args: Array<String>) {
    fun printLength(obj: Any) {
        println("'$obj' string length is ${getStringLength(obj) ?: "... err, is empty or not a string at all"} ")
    }
    printLength("Incomprehensibilities")
    printLength("")
    printLength(1000)
}
```
</div>

[클래스](classes.html)와 [타입 변환](typecasts.html)을 참고한다.

## `for` 루프 사용

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwi")
    for (item in items) {
        println(item)
    }
//sampleEnd
}
```
</div>

or

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwi")
    for (index in items.indices) {
        println("item at $index is ${items[index]}")
    }
//sampleEnd
}
```
</div>


[for 루프](control-flow.html#for-loops)를 참고한다.

## `while` 루프 ㅅ용

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val items = listOf("apple", "banana", "kiwi")
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index++
    }
//sampleEnd
}
```
</div>


[while 루프](control-flow.html#while-loops)를 참고한다.

## `when` 식 사용

<div class="sample" markdown="1">

``` kotlin
//sampleStart
fun describe(obj: Any): String =
    when (obj) {
        1          -> "One"
        "Hello"    -> "Greeting"
        is Long    -> "Long"
        !is String -> "Not a string"
        else       -> "Unknown"
    }
//sampleEnd

fun main(args: Array<String>) {
    println(describe(1))
    println(describe("Hello"))
    println(describe(1000L))
    println(describe(2))
    println(describe("other"))
}
```
</div>


[when 식](control-flow.html#when-expression)을 참고한다.

## 범위 사용

*in*{: .keyword } 연산자를 사용해서 숫자가 범위에 들어오는지 검사한다:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val x = 10
    val y = 9
    if (x in 1..y+1) {
        println("fits in range")
    }
//sampleEnd
}
```
</div>


숫자가 범위를 벗어나는지 검사한다:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val list = listOf("a", "b", "c")
    
    if (-1 !in 0..list.lastIndex) {
        println("-1 is out of range")
    }
    if (list.size !in list.indices) {
        println("list size is out of valid list indices range too")
    }
//sampleEnd
}
```
</div>


범위를 반복:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    for (x in 1..5) {
        print(x)
    }
//sampleEnd
}
```
</div>

또는 단순 범위 이상:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    for (x in 1..10 step 2) {
        print(x)
    }
    for (x in 9 downTo 0 step 3) {
        print(x)
    }
//sampleEnd
}
```
</div>

[범위](ranges.html)를 참고한다.

## 콜렉션 사용

콜렉션에 대한 반복:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
    val items = listOf("apple", "banana", "kiwi")
//sampleStart
    for (item in items) {
        println(item)
    }
//sampleEnd
}
```
</div>


*in*{: .keyword } 연산자로 콜렉션이 객체를 포함하는지 검사:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
    val items = setOf("apple", "banana", "kiwi")
//sampleStart
    when {
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
//sampleEnd
}
```
</div>


콜렉션을 걸러내고 변환하기 위해 람다 식 사용:


<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
    val fruits = listOf("banana", "avocado", "apple", "kiwi")
//sampleStart
    fruits
        .filter { it.startsWith("a") }
        .sortedBy { it }
        .map { it.toUpperCase() }
        .forEach { println(it) }
//sampleEnd
}
```
</div>

[고차 함수와 람다](lambdas.html)를 참고한다.

## 기본 클래스와 인스턴스 만들기:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val rectangle = Rectangle(5.0, 2.0) //no 'new' keyword required
    val triangle = Triangle(3.0, 4.0, 5.0)
//sampleEnd
    println("Area of rectangle is ${rectangle.calculateArea()}, its perimeter is ${rectangle.perimeter}")
    println("Area of triangle is ${triangle.calculateArea()}, its perimeter is ${triangle.perimeter}")
}

abstract class Shape(val sides: List<Double>) {
    val perimeter: Double get() = sides.sum()
    abstract fun calculateArea(): Double
}

interface RectangleProperties {
    val isSquare: Boolean
}

class Rectangle(
    var height: Double,
    var length: Double
) : Shape(listOf(height, length, height, length)), RectangleProperties {
    override val isSquare: Boolean get() = length == height
    override fun calculateArea(): Double = height * length
}

class Triangle(
    var sideA: Double,
    var sideB: Double,
    var sideC: Double
) : Shape(listOf(sideA, sideB, sideC)) {
    override fun calculateArea(): Double {
        val s = perimeter / 2
        return Math.sqrt(s * (s - sideA) * (s - sideB) * (s - sideC))
    }
}
```
</div>

[클래스](classes.html)와 [오브젝트와 인스턴스](object-declarations.html)를 참고한다.
