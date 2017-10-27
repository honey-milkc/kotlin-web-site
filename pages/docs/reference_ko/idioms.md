---
type: doc
layout: reference_ko
category: "Basics"
title: "이디엄"
---

# 이디엄

코틀린에서 종종 사용되는 이디엄을 정리했다.
선호하는 이디엄이 있다면 풀리퀘스트를 날려 기여해보자.

### DTO 생성 (POJO/POCO)

``` kotlin
data class Customer(val name: String, val email: String)
```

다음 기능을 가진 `Customer` 클래스를 제공한다:

* 모든 프로퍼티에 대한 getter (그리고 *var*{: .keyword }의 경우 setter)
* `equals()`
* `hashCode()`
* `toString()`
* `copy()`
* 모든 프로퍼티에 대한 `component1()`, `component2()`, ..., ([data 클래스](data-classes.html) 참고)


### 함수 파라미터의 기본 값

``` kotlin
fun foo(a: Int = 0, b: String = "") { ... }
```

### 리스트 필터링

``` kotlin
val positives = list.filter { x -> x > 0 }
```

더 짧게 표현:

``` kotlin
val positives = list.filter { it > 0 }
```

### 스트링 삽입

``` kotlin
println("Name $name")
```

### 인스턴스 검사

``` kotlin
when (x) {
    is Foo -> ...
    is Bar -> ...
    else   -> ...
}
```

### 쌍으로 맵이나 목록 탐색

``` kotlin
for ((k, v) in map) {
    println("$k -> $v")
}
```

`k`, `v` 대신 임의 이름을 사용할 수 있다.

### 범위 사용

``` kotlin
for (i in 1..100) { ... }  // 닫힌 범위: 100 포함
for (i in 1 until 100) { ... } // 반만 열린 범위: 100 미포함
for (x in 2..10 step 2) { ... }
for (x in 10 downTo 1) { ... }
if (x in 1..10) { ... }
```

### 읽기 전용 리스트

``` kotlin
val list = listOf("a", "b", "c")
```

{:#read-only-map}

### 읽기 전용 맵

``` kotlin
val map = mapOf("a" to 1, "b" to 2, "c" to 3)
```

### 맵 접근

``` kotlin
println(map["key"])
map["key"] = value
```

### 지연(lazy) 프로퍼티

``` kotlin
val p: String by lazy {
    // 문자열 계산
}
```

### 확장 함수

``` kotlin
fun String.spaceToCamelCase() { ... }

"Convert this to camelcase".spaceToCamelCase()
```

### 싱글톤 생성

``` kotlin
object Resource {
    val name = "Name"
}
```

### If not null 축약

``` kotlin
val files = File("Test").listFiles()

println(files?.size)
```

### If not null and else 축약

``` kotlin
val files = File("Test").listFiles()

println(files?.size ?: "empty")
```

### null이면 문장 실행

``` kotlin
val values = ...
val email = values["email"] ?: throw IllegalStateException("Email is missing!")
```

### null이 아니면 실행

``` kotlin
val value = ...

value?.let {
    ... // null이 아닌 블록 실행
}
```

### null이 아니면 null 가능 값 매핑

``` kotlin
val value = ...

val mapped = value?.let { transformValue(it) } ?: defaultValueIfValueIsNull
```

### when 문장에서 리턴

``` kotlin
fun transform(color: String): Int {
    return when (color) {
        "Red" -> 0
        "Green" -> 1
        "Blue" -> 2
        else -> throw IllegalArgumentException("Invalid color param value")
    }
}
```

### 'try/catch' 식

``` kotlin
fun test() {
    val result = try {
        count()
    } catch (e: ArithmeticException) {
        throw IllegalStateException(e)
    }

    // 결과로 작업
}
```

### 'if' 식

``` kotlin
fun foo(param: Int) {
    val result = if (param == 1) {
        "one"
    } else if (param == 2) {
        "two"
    } else {
        "three"
    }
}
```

### `Unit`을 리턴하는 메서드의 빌더-방식 용법

``` kotlin
fun arrayOfMinusOnes(size: Int): IntArray {
    return IntArray(size).apply { fill(-1) }
}
```


### 단일 식 함수

``` kotlin
fun theAnswer() = 42
```

이는 다음과 동일하다.

``` kotlin
fun theAnswer(): Int {
    return 42
}
```

단일 식 함수는 효과적으로 다른 이디엄과 쓸 수 있으며 코드를 더 짧게 만들어준다.
예를 들어, 다음은 *when*{: .keyword } 식과 함께 사용하는 코드이다:

``` kotlin
fun transform(color: String): Int = when (color) {
    "Red" -> 0
    "Green" -> 1
    "Blue" -> 2
    else -> throw IllegalArgumentException("Invalid color param value")
}
```

### 객체 인스턴스의 메서드를 여러 번 호출('with')

``` kotlin
class Turtle {
    fun penDown()
    fun penUp()
    fun turn(degrees: Double)
    fun forward(pixels: Double)
}

val myTurtle = Turtle()
with(myTurtle) { //100 pix 사각형 그리기
    penDown()
    for(i in 1..4) {
        forward(100.0)
        turn(90.0)
    }
    penUp()
}
```

### 자바 7의 자원을 사용한 try

``` kotlin
val stream = Files.newInputStream(Paths.get("/some/file.txt"))
stream.buffered().reader().use { reader ->
    println(reader.readText())
}
```

### 지네릭 타입 정보를 요구하는 지네릭 함수를 위한 편리한 양식

``` kotlin
//  public final class Gson {
//     ...
//     public <T> T fromJson(JsonElement json, Class<T> classOfT) throws JsonSyntaxException {
//     ...

inline fun <reified T: Any> Gson.fromJson(json: JsonElement): T = this.fromJson(json, T::class.java)
```

### null 가능 Boolean 사용

``` kotlin
val b: Boolean? = ...
if (b == true) {
    ...
} else {
    // `b`는 false나 null
}
```
