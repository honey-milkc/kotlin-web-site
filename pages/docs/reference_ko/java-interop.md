---
type: doc
layout: reference_ko
category: "Java Interop"
title: "코틀린에서 자바 호출하기"
---

# 코틀린에서 자바 호출하기

코틀린은 자바와의 상호운용성을 염두에 두고 설게했다. 기존 자바 코드를 코틀린에서 자연스럽게 호출할 수 있고,
자바 코드에서도 비교적 부드럽게 코틀린 코드를 사용할 수 있다.
이 절에서는 코틀린에서 자바 코드를 호출하는 것에 대해 자세히 살펴본다.

아무 문제 없이 거의 모든 자바 코드를 사용할 수 있다:

``` kotlin
import java.util.*

fun demo(source: List<Int>) {
    val list = ArrayList<Int>()
    // 자바 콜렉션에 대해 'for'-루프 동작
    for (item in source) {
        list.add(item)
    }
    // 연산자 규칙 또한 동작:
    for (i in 0..source.size - 1) {
        list[i] = source[i] // get과 set 호출됨
    }
}
```

## Getters와 Setters

getter와 setter를 위한 자바 규칙(이름이 `get`으로 시작하는 인자 없는 메서드와 이름이 `set`으로 시작하고 한 개 인자를 갖는 메서드)을 따르는 메서드는 코틀린에서 프로퍼티로 표현된다.
`Boolean` 접근 메서드(getter 이름이 `is`로 시작하고 setter 이름은 `set`으로 시작)는
getter 메서드와 같은 이름을 갖는 프로퍼티로 표현한다.

예제:

``` kotlin
import java.util.Calendar

fun calendarDemo() {
    val calendar = Calendar.getInstance()
    if (calendar.firstDayOfWeek == Calendar.SUNDAY) {  // getFirstDayOfWeek() 호출
        calendar.firstDayOfWeek = Calendar.MONDAY      // setFirstDayOfWeek() 호출
    }
    if (!calendar.isLenient) {                         // isLenient() 호출
        calendar.isLenient = true                      // setLenient() 호출
    }
}
```

자바 클래스에 setter만 존재하면 코틀린에서 프로퍼티로 보이지 않는다.
왜냐면 현재 코틀린은 쓰기 전용 프로퍼티를 지원하지 않기 때문이다.

## void를 리턴하는 메서드

void를 리턴하는 자바 메서드를 코틀린에서 호출하면 `Unit`을 리턴한다.
만약 리턴 값을 사용하면, 값 자체를 미리 알 수 있기 때문에(`Unit`) 코틀린 컴파일러는 호출 위치에 값을 할당한다.  

## 코틀린에서 키워드인 자바 식별자 처리

*in*{: .keyword }, *object*{: .keyword }, *is*{: .keyword } 등 일부 코틀린 키워드는 자바에서 유효한 식별자이다.
자바 라이브러리가 코틀린 키워드를 메서드 이름으로 사용할 경우, 역따옴표를 탈출 문자로 사용해서 메서드를 호출할 수 있다:

``` kotlin
foo.`is`(bar)
```

## null 안정성과 플랫폼 타입
{:#null-safety-and-platform-types}

자바에서 모든 레퍼런스는 *null*{: .keyword }일 수 있는데, 이는 자바에서 온 객체에 대해
코틀린의 엄격한 null 안정성 요구를 비실용적으로 만든다.
코틀린에서는 *플랫폼 타입*이라고 불리는 자바 선언 타입으로 별도 처리한다.
그 타입에 대한 null 검사를 완화해서 안정성 보장을 자바와 동일하게 한다([아래](#mapped-types) 참고) 

다음 예를 보자:

``` kotlin
val list = ArrayList<String>() // non-null (생성자 결과)
list.add("Item")
val size = list.size // non-null (기본 int)
val item = list[0] // 플랫폼 타입 유추 (일반 자바 객체)
```

플랫폼 타입 변수의 메서드를 호출할 때, 코틀린은 컴파일 시점에 null 가능성 에러를 발생하지 않는다.
대신 코틀린이 null이 전파되는 것을 막기 위해 생성한 단언이나 널포인터 익셉션 때문에 런타임에 호출이 실패할 수 있다. 

``` kotlin
item.substring(1) // 허용, 단 item이 null이면 익셉션이 발생
```

플랫폼 타입은 *non-denotable*인데, 이는 언어에서 그 타입을 바로 사용할 수 없다는 것을 의미한다.
플랫폼 값을 코틀린 변수에 할당하면, 타입 추론에 의존하거나(위 예에서 `item`은 추론한 플랫폼 타입을 가짐)
또는 기대하는 타입을 선택할 수 있다(null 가능과 non-null 타입 둘 다 허용).

``` kotlin
val nullable: String? = item // 허용, 항상 동작
val notNull: String = item // 허용, 런타임에 실패 가능
```

non-null 타입을 선택하면 컴파일러는 할당 위치에 단언을 생성(emit)한다. 이는 코틀린의 non-null 변수가 null 값을 갖는 것을 막아준다.
단언은 또한 non-null 값을 기대하는 코틀린 함수에 플랫폼 값을 전달할 때에도 생성된다.
전반적으로 컴파일러는 null이 프로그램에 퍼지는 것을 막기 위해 최선을 다한다(지네릭 때문에 완전히 없애는 것은 불가능하다). 

### 플랫폼 타입을 위한 표기

위에서 언급한 것처럼, 플랫폼 타입을 프로그램에서 직접 언급할 수 없기 때문에 언어에 플랫폼 타입을 위한 구문이 없다.
그럼에도 불구하고 컴파일러와 IDE는 때때로 플랫폼 타입을 표시해야 할 때가 있으며(에러 메시지에서나 파라미터 정보 등),
따라서 이를 위한 기억용 표기를 제공한다:

* `T!`은 "`T` 또는 `T?`"를 의미한다.
* `(Mutable)Collection<T>!`은 "`T`의 자바 콜렉션이 수정가능하거나 불변일 수 있고, null 가능이거나 non-null일 수 있다"는 것을 의미한다.
* `Array<(out) T>!`은 "`T`의(또는 `T`의 하위타입의) 자바 배열이 null 가능이거나 아닐 수 있다"를 의미한다.

### 널가능성 애노테이션

널가능성(nullability) 애노테이션을 가진 자바 타입은 플랫폼 타입이 아닌 실제 null 가능 또는 not-null 코틀린 타입으로 표현한다.
컴파일러는 다음을 포함한 여러 널가능성 애노테이션을 지원한다: 

  * [JetBrains](https://www.jetbrains.com/idea/help/nullable-and-notnull-annotations.html)
(`org.jetbrains.annotations` 패키지의 `@Nullable`과 `@NotNull`)
  * Android (`com.android.annotations` and `android.support.annotations`)
  * JSR-305 (`javax.annotation`, 아래에서 자세히 설명)
  * FindBugs (`edu.umd.cs.findbugs.annotations`)
  * Eclipse (`org.eclipse.jdt.annotation`)
  * Lombok (`lombok.NonNull`).

전체 목록은 [코틀린 컴파일러 소스 코드](https://github.com/JetBrains/kotlin/blob/master/core/descriptors.jvm/src/org/jetbrains/kotlin/load/java/JvmAnnotationNames.kt)에서
찾을 수 있다.

### JSR-305 지원

[JSR-305](https://jcp.org/en/jsr/detail?id=305)에 정의된 [`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html) 애노테이션은
자바 타입의 널가능성을 표시하기 위해 지원한다.

`@Nonnull(when = ...)` 값이 `When.ALWAYS`이면, 애노테이션이 붙은 타입을 non-null로 다룬다. `When.MAYBE`와 `When.NEVER`는
null 가능 타입을 표시한다. `When.UNKNOWN`은 타입을 [플랫폼 타입](#null-safety-and-platform-types)으로 강제한다.

라이브러리를 JSR-305 애노테이션에 기대어 컴파일할 수 있지만, 라이브러리리 사용자를 위한 컴파일 의존에 애노테이션 아티팩트(예, `jsr305.jar`)를
포함시킬 필요는 없다. 코틀린 컴파일러는 클래스패스에 존재하는 JSR-305 애노테이션을 읽을 수 있다.  

코틀린 1.1.50 부터, 
[커스텀 널가능성 한정자(KEEP-79)](https://github.com/Kotlin/KEEP/blob/41091f1cc7045142181d8c89645059f4a15cc91a/proposals/jsr-305-custom-nullability-qualifiers.md)를
지원한다(아래 참고).

#### 타입 한정자 별명 (1.1.50부터)

애노테이션 타입에 [`@TypeQualifierNickname`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierNickname.html)과
JSR-305 `@Nonnull`(또는 이것의 다른 이름인 `@CheckForNull`와 같은 것)을 함께 붙이면,
정확한 널가능성을 검색하기 위해 그 애노테이션 타입 자체를 사용할 수 있고, 적용한 널가능성 애노테이션과 동일한 의미를 갖는다:

``` java
@TypeQualifierNickname
@Nonnull(when = When.ALWAYS)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyNonnull {
}

@TypeQualifierNickname
@CheckForNull // 다른 타입 한정 별칭에 대한 별칭
@Retention(RetentionPolicy.RUNTIME)
public @interface MyNullable {
}

interface A {
    @MyNullable String foo(@MyNonnull String x); 
    // 코틀린에서 (strict 모드): `fun foo(x: String): String?`
    
    String bar(List<@MyNonnull String> x);       
    // 코틀린에서 (strict 모드): `fun bar(x: List<String>!): String!`
}
```

#### 타입 한정자 디폴트 (1.1.50부터)

[`@TypeQualifierDefault`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/meta/TypeQualifierDefault.html)는
새로운 애노테이션 도입을 허용하는데, 그 애노테이션을 적용할 때 애노테이션을 붙인 요소의 범위 안에서 기본 널가능성을 정의한다.  

그런 애노테이션 타입은 `@Nonnull`(또는 닉네임)과 한 개 이상의 `ElementType` 값을 갖는 `@TypeQualifierDefault(...)` 애노테이션을
붙여야 한다:
* `ElementType.METHOD` : 메서드 리턴 타입 대상
* `ElementType.PARAMETER` : 밸류 파라미터 대상
* `ElementType.FIELD` : 필드 대상
* `ElementType.TYPE_USE` (1.1.60 부터) : 타입 인자를 포함한 모든 타입, 타입 파라미터의 상위 한계, 와일드카드 타입 대상 

타입 자체에 널가능성 애노테이션이 없을 때 기본 널가능성을 사용하며,
타입 용도와 일치하는 `ElementType`을 가진 @TypeQualifierDefault을 붙인 가장 안쪽을 둘러싼 요소가 기본 널가능성을 결정한다.

```java
@Nonnull
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER})
public @interface NonNullApi {
}

@Nonnull(when = When.MAYBE)
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE_USE})
public @interface NullableApi {
}

@NullableApi
interface A {
    String foo(String x); // fun foo(x: String?): String?
 
    @NotNullApi // 인터페이스로부터 기본을 오버라이딩
    String bar(String x, @Nullable String y); // fun bar(x: String, y: String?): String 

    // `@NullableApi`가 `TYPE_USE` 요소 타입을 갖기 때문에
    // The List<String> 타입 인자는 nullable로 보인다: 
    String baz(List<String> x); // fun baz(List<String?>?): String?

    // The type of `x` 파라미터 타입은 플랫폼으로 남는다.
    // 이유는 널가능성 애노테이션 값을 명시적으로 UNKNOWN으로 표시했기 때문이다.
    String qux(@Nonnull(when = When.UNKNOWN) String x); // fun baz(x: String!): String?
}
```

> 주의: 이 예제의 타입은 strict 모드를 활성화할 때만 적용되며, 그렇지 않을 경우 플랫폼 타입으로 남는다. [`@UnderMigration` 애노테이션](#undermigration-annotation-since-1160)과 [컴파일러 설정](#compiler-configuration) 절을 참고한다.

패키지 수준의 기본 널가능성 또한 지원한다:

```java
// FILE: test/package-info.java
@NonNullApi // 'test' 패키지에 있는 모든 타입의 기본 널가능성을 non-nullable로 선언한다
package test;
```

#### `@UnderMigration` 애노테이션 (1.1.60 부터 지원)

`@UnderMigration` 애노테이션은 (별도 아티팩트인 `kotlin-annotations-jvm`에서 제공) 
라이브러리 유지보수 담당자가 널가능성 타입 한정자를 마이그레이션 상태로 정의하고 싶을 때 사용할 수 있다. 

`@UnderMigration(status = ...)`의 status 값은 애노테이션이 붙은 타입을 코틀린에서 알맞지 않게 사용한 경우
(예, `@MyNullable`이 붙은 타입 값을 non-null에 사용) 
컴파일러가 이를 어떻게 처리할지 지정한다.   

* `MigrationStatus.STRICT` : 일반 널가능성 애노테이션처럼 애노테이션이 동작하게 하고(예를 들어, 잘못 사용하면 에러를 발생한다)
애노테이션을 적용한 선언의 타입을 코틀린에서 볼 수 있기 때문에 해당 타입에 작용한다.

* `MigrationStatus.WARN` : 잘못 사용하면 에러 대신 컴파일 경고를 발생하지만, 애노테이션이 붙은 타입은 플랫폼 타입으로 남는다.

* `MigrationStatus.IGNORE` : 널가능성 애노테이션을 완전히 무시한다.

라이브러리 유지보수 담당자는 `@UnderMigration`의 status를 타입 한정자 별명과 타입 한정자 디폴트에 추가할 수 있다. 

```java
@Nonnull(when = When.ALWAYS)
@TypeQualifierDefault({ElementType.METHOD, ElementType.PARAMETER})
@UnderMigration(status = MigrationStatus.WARN)
public @interface NonNullApi {
}

// 클래스에서 타입은 non-null이지만, `@NonNullApi`에 `@UnderMigration(status = MigrationStatus.WARN)`을
// 붙였으므로 경고만 발생한다.
@NonNullApi 
public class Test {}
```

주의: 널가능성 애노테이션의 마이그레이션 상태는 타입 한정자 별칭에 의해 상속되지 않지만,
디폴트 타입 한정자에서 그것을 적용한다. 

만약 디폴트 타입 한정자가 타입 한정자 별명을 사용하고 둘 다 `@UnderMigration`이라면,
디폴트 타입 한정자의 상태를 사용한다.

#### 컴파일러 설정

다음 옵션을 갖는 `-Xjsr305` 컴파일러 플래그를 추가(및 조합)해서 JSR-305 검사를 설정할 수 있다:

* `-Xjsr305={strict|warn|ignore}`은 `@UnderMigration`이 아닌 애노테이션의 동작을 설정한다.  
커스텀 널가능성 한정자(특히 `@TypeQualifierDefault`)는 이미 많은 알려진 라이브러리에 퍼져 있으며,
사용자는 JSR-305 지원을 포함하는 코틀린 버전으로 업데이트 할 때 매끄럽게 마이그레이션해야 할 수도 있다.
코틀린 1.1.60부터 이 플래그는 오직 `@UnderMigration`이 아닌 애노테이션에만 영향을 준다.

* `-Xjsr305=under-migration:{strict|warn|ignore}` (1.1.60 부터)은 `@UnderMigration` 애노테이션의 동작을 정의한다.
사용자는 라이브러리의 마이그레이션 상태에 다른 뷰를 가질 수 있다.
공식적인 마이그레이션 상태가 'WARN'인 동안 에러를 발생하길 원할 수도 있고,
마이그레이션이 끝날 때가지 에러 발생을 미루고 싶을 수도 있다.

* `-Xjsr305=@<fq.name>:{strict|warn|ignore}` (1.1.60 부터) 은 애노테이션의 완전한 클래스 이름이
`<fq.name>`인 애노테이션의 동작을 정의한다. 각 애노테이션별로 설정할 수 있다.
이는 특정 라이브러리를 위한 마이그레이션 상태를 관리할 때 유용하다.

`strict`, `warn`, `ignore` 값의 의미는 `MigrationStatus`의 값과 같으며,
오직 `strict` 모드의 경우, 애노테이션을 붙인 선언의 타입을 코틀린에서 볼 수 있기 때문에 해당 타입에 영향을 준다.

> 주의: 내장된 JSR-305 애노테이션인 
[`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html), 
[`@Nullable`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nullable.html),
[`@CheckForNull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/CheckForNull.html)는
항상 활성화되며, 컴파일러 설정에 `-Xjsr305` 플래그를 추가했는지 여부에 상관없이
애노테이션을 붙인 선언의 타입을 코틀린에서 볼 수 있기 때문에 해당 타입에 영향을 준다.

예를 들어, 컴파일러 인자에 `-Xjsr305=ignore -Xjsr305=under-migration:ignore -Xjsr305=@org.library.MyNullable:warn`를 추가하면
컴파일러는 `@org.library.MyNullable`을 붙인 타입을 잘못 사용한 것에 대해 경고를 발생하지만
다른 JSR-305 애노테이션은 무시한다.

코틀린 1.1.50+/1.2 버전의 기본 동작은 [`@Nonnull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nonnull.html), [`@Nullable`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/Nullable.html), [`@CheckForNull`](https://aalmiray.github.io/jsr-305/apidocs/javax/annotation/CheckForNull.html)를
제외하면 `-Xjsr305=warn`과 같다. `strict` 값은 실험적으로만 고려해야 한다(향후에 더 다양한 검사를 추가할 것이다).

## 매핑된 타입
{:#mapped-types}

코틀린은 일부 자바 타입을 특별하게 다룬다. 그런 타입은 자바 타입을 "그대로"로 로딩하지 않고 대응하는 코틀린 타입으로 _매핑_한다.
매핑은 컴파일 시점에만 중요하며 런타임 표현은 바뀌지 않고 유지한다.
자바 기본 타입은 대응하는 코틀린 타입으로 매핑된다([플랫폼 타입](#null-safety-and-platform-types)을 염두한다).

| **자바 타입** | **코틀린 타입**  |
|---------------|------------------|
| `byte`        | `kotlin.Byte`    |
| `short`       | `kotlin.Short`   |
| `int`         | `kotlin.Int`     |
| `long`        | `kotlin.Long`    |
| `char`        | `kotlin.Char`    |
| `float`       | `kotlin.Float`   |
| `double`      | `kotlin.Double`  |
| `boolean`     | `kotlin.Boolean` |
{:.zebra}

일부 기본 타입이 아닌 내장 클래스도 매핑한다:

| **자바 타입** | **코틀린 타입**  |
|---------------|------------------|
| `java.lang.Object`       | `kotlin.Any!`    |
| `java.lang.Cloneable`    | `kotlin.Cloneable!`    |
| `java.lang.Comparable`   | `kotlin.Comparable!`    |
| `java.lang.Enum`         | `kotlin.Enum!`    |
| `java.lang.Annotation`   | `kotlin.Annotation!`    |
| `java.lang.Deprecated`   | `kotlin.Deprecated!`    |
| `java.lang.CharSequence` | `kotlin.CharSequence!`   |
| `java.lang.String`       | `kotlin.String!`   |
| `java.lang.Number`       | `kotlin.Number!`     |
| `java.lang.Throwable`    | `kotlin.Throwable!`    |
{:.zebra}

자바의 박싱된 기본 타입은 null 가능 코틀린 타입으로 매핑한다:

| **자바 타입**           | **코틀린 타입**  |
|-------------------------|------------------|
| `java.lang.Byte`        | `kotlin.Byte?`   |
| `java.lang.Short`       | `kotlin.Short?`  |
| `java.lang.Integer`     | `kotlin.Int?`    |
| `java.lang.Long`        | `kotlin.Long?`   |
| `java.lang.Character`   | `kotlin.Char?`   |
| `java.lang.Float`       | `kotlin.Float?`  |
| `java.lang.Double`      | `kotlin.Double?`  |
| `java.lang.Boolean`     | `kotlin.Boolean?` |
{:.zebra}

타입 파라미터로 박싱된 기본 타입을 사용하면 플랫폼 타입으로 매핑된다. 예를 들어 `List<java.lang.Integer>`는 코틀린에서 `List<Int!>`가 된다.

콭렉션 타입은 코틀린에서 읽기 전용 타입과 변경가능 타입이 존재하므로 자바 콜렉션을 다음과 같이 변경한다(이 표에서 모든 코틀린 타입은 
`kotlin.collections` 패키지에 위치한다):

| **자바 타입** | **코틀린 읽기 전용 타입**  | **코틀린 수정가능 타입** | **로딩된 플랫폼 타입** |
|---------------|------------------|----|----|
| `Iterator<T>`        | `Iterator<T>`        | `MutableIterator<T>`            | `(Mutable)Iterator<T>!`            |
| `Iterable<T>`        | `Iterable<T>`        | `MutableIterable<T>`            | `(Mutable)Iterable<T>!`            |
| `Collection<T>`      | `Collection<T>`      | `MutableCollection<T>`          | `(Mutable)Collection<T>!`          |
| `Set<T>`             | `Set<T>`             | `MutableSet<T>`                 | `(Mutable)Set<T>!`                 |
| `List<T>`            | `List<T>`            | `MutableList<T>`                | `(Mutable)List<T>!`                |
| `ListIterator<T>`    | `ListIterator<T>`    | `MutableListIterator<T>`        | `(Mutable)ListIterator<T>!`        |
| `Map<K, V>`          | `Map<K, V>`          | `MutableMap<K, V>`              | `(Mutable)Map<K, V>!`              |
| `Map.Entry<K, V>`    | `Map.Entry<K, V>`    | `MutableMap.MutableEntry<K,V>` | `(Mutable)Map.(Mutable)Entry<K, V>!` |
{:.zebra}

자바 배열은 [아래](java-interop.html#java-arrays)에서 언급한 것처럼 매핑된다:

| **Java type** | **Kotlin type**  |
|---------------|------------------|
| `int[]`       | `kotlin.IntArray!` |
| `String[]`    | `kotlin.Array<(out) String>!` |
{:.zebra}

## 코틀린에서의 자바 지네릭

코틀린 지네릭은 자바 지네릭과 조금 다르다([지네릭](generics.html)을 보자). 자바 타입을 코틀린에 임포트할 때 일부 변환을 수행한다:

* 자바의 와일드카드는 타입 프로젝션으로 변환
  * `Foo<? extends Bar>`는 `Foo<out Bar!>!`가 됨
  * `Foo<? super Bar>`는 `Foo<in Bar!>!`가 됨

* 자바 원시 타입은 스타 프로젝션으로 변환
  * `List`는 `List<*>!`가 된다 (예를 들어, `List<out Any?>!`)

자바와 마찬가지로 코틀린 지네릭도 런타임에 유지되지 않는다. 예를 들어, 객체는 생성자에 전달된 실제 타입 인자에 대한 정보를 담지 않으므로
`ArrayList<Integer>()`와 `ArrayList<Character>()`를 구분할 수 없다.
이는 지네릭을 고려한 *is*{: .keyword }를 수행하기 어렵게 만든다.
코틀린은 오직 스타 프로젝션인 지네릭 타입에 대한 *is*{: .keyword }만 허용한다:

``` kotlin
if (a is List<Int>) // 에러: 실제로 Int 리스트인지 확인할 수 없다
// 하지만
if (a is List<*>) // OK: List 내용에 대한 보장을 하지 않음
```

## 자바 배열
{:#java-arrays}

코틀린 배열은 자바와 달리 무공변이어서 발생 가능한 런타임 실패를 방지한다. 상위타입 배열 파라미터를 사용하는 코틀린 메서드에서
하위타입 배열을 전달하는 것을 금지한다. 자바 메서드의 경우 이를 허용한다(`Array<(out) String>!` 형식의 [플랫폼 타입). 

박싱/언박싱 오퍼레이션 비용을 없애기 위해 자바 플랫폼의 기본 데이터 타입을 가진 배열을 사용한다.
코틀린은 이런 구현 상세를 감추기 때문에 자바 코드를 사용하려면 다른 방법이 필요하다.
이런 경우를 다루기 위해 모든 기본 타입 배열을 위한 특수 클래스(`IntArray`, `DoubleArray`, `CharArray` 등)를 제공하고 있다.
특수 클래스는 `Array` 클래스와 상관없으며 최대 성능을 위해 자바의 기본타입 배열로 컴파일된다.

다음과 같이 int 타입 배열인 indices를 받는 자바 메서드가 있다고 하자:

``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // 여기 코드 ...
    }
}
```

다음과 같이 코틀린에서 기본타입 값을 갖는 배열을 전달할 수 있다:

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndices(array)  // 메서드에 int[] 전달
```

JVM 바이트 코드로 컴파일할 때, 컴파일러는 배열에 대한 접근을 최적화해서 오버헤드가 발생하지 않는다:

``` kotlin
val array = arrayOf(1, 2, 3, 4)
array[x] = array[x] * 2 // get()과 set()에 대한 실제 호출이 없음
for (x in array) { // 이터레이터 만들지 않음
    print(x)
}
```

인덱스를 갖고 조작할 때 조차도 오버헤드가 발생하지 않는다:

``` kotlin
for (i in array.indices) { // 이터레이터 만들지 않음
    array[i] += 2
}
```

마지막으로 *in*{: .keyword } 검사도 오베헤드가 없다::

``` kotlin
if (i in array.indices) { // (i >= 0 && i < array.size)와 동일
    print(array[i])
}
```

## 자바 가변인자

자바 클래스는 종종 가변 인자를 가진 메서드 선언을 사용한다(varars):

``` java
public class JavaArrayExample {

    public void removeIndicesVarArg(int... indices) {
        // 코드 사용
    }
}
```

이 경우 `IntArray`에 `*` 연산자(펼침 연산자)를 사용해서 배열을 전달할 수 있다:

``` kotlin
val javaObj = JavaArrayExample()
val array = intArrayOf(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

현재 varargs로 선언한 메서드에 *null*{: .keyword }을 전달할 수는 없다.

## 연산자

자바는 연산자 구문을 사용하기에 알맞은 메서드 표시 방법이 없이 때문에,
코틀린은 연산자 오버로딩과 다른 규칙(`invoke()` 등)에 맞는 이름과 시그너처를 가진 자바 메서드를 허용한다.
중위 호출 구문을 사용해서 자바 메서드를 호출하는 것은 허용하지 않는다.


## Checked 익셉션

코틀린에서 모든 익셉션은 unchecked다. 이는 컴파일러가 익셉션 처리(catch)를 강제하지 않음을 의미한다.
따라서 checked 익셉션을 선언한 자바 메서드를 호출할 때 코틀린은 어떤 것도 강제하지 않는다.

``` kotlin
fun render(list: List<*>, to: Appendable) {
    for (item in list) {
        to.append(item.toString()) // 자바는 IOException을 catch할 것을 요구한다
    }
}
```

## Object 메서드

자바 타입을 코틀린으로 임포트할 때, `java.lang.Object` 타입의 모든 레퍼런스는 `Any`로 바뀐다.
`Any`는 플랫폼에 종속되지 않기 때문에, 멤버로 `toString()`, `hashCode()`, `equals()`만 선언하고 있다.
`java.lang.Object`의 다른 멤버를 사용하려면 [확장 함수](extensions.html)를 사용한다.

### wait()/notify()

[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) Item 69는
`wait()`와 `notify()` 보다 병행 유틸리티를 쓰라고 제안한다.
그래서 `Any` 타입 레퍼런스에는 이 두 메서드를 사용할 수 없다.
실제로 이 두 메서드를 호출하고 싶다면 `java.lang.Object`로 변환해서 호출할 수는 있다:

```kotlin
(foo as java.lang.Object).wait()
```

### getClass()

객체의 자바 클래스를 구하려면, [클래스 레퍼런스](reflection.html#class-references)의 `java` 확장 프로퍼티를 사용한다:

``` kotlin
val fooClass = foo::class.java
```

위 코드는 코틀린 1.1부터 지원하는 [객체에 묶인 클래스 레퍼런스](reflection.html#bound-class-references-since-11)로,
대신 `javaClass` 확장 프로퍼티를 사용할 수도 있다:

``` kotlin
val fooClass = foo.javaClass
```

### clone()

`clone()`을 오버라이딩하려면, `kotlin.Cloneable`를 확장해야 한다:

```kotlin

class Example : Cloneable {
    override fun clone(): Any { ... }
}
```

[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 11: *Override clone judiciously*를 잊지 말자.

### finalize()

`finalize()`를 오버라이딩 하려면, *override*{:.keyword} 키워드 없이 메서드를 선언하면 된다:

```kotlin
class C {
    protected fun finalize() {
        // finalization logic
    }
}
```

자바 규칙에 따라, `finalize()`는 *private*{: .keyword }이면 안 된다.

## 자바 클래스를 상속

코틀린에서 최대 한 개의 자바 클래스를 상속할 수 있다(자바 인터페이스는 필요한 만큼 많이 상속 가능).

## 정적 멤버에 접근

자바 클래스의 정적 멤버는 그 클래스를 위한 "컴페니언 오브젝트"로 구성한다. 그 "컴페니언 오브젝트"를 값으로 전달할 수 없지만,
멤버에 접근할 수는 있다:

``` kotlin
if (Character.isLetter(a)) {
    // ...
}
```

## 자바 리플렉션

자바 리플렉션은 코틀린 클래스에 동작하며, 반대로도 동작한다. 위에서 언급한 것처럼
`instance::class.java`, `ClassName::class.java`, `instance.javaClass`를 사용해서 자바 리플렉션에 필요한
`java.lang.Class`를 구할 수 있다.

자바 getter/setter 메서드 또는 코틀린 프로퍼티를 위한 지원 필드 구하기, 자바 필드를 위한 `KProperty` 구하기,
`KFunction`을 위한 자바 메서드나 생성자 구하기 또는 그 반대 구하기 등을 지원한다.

## SAM 변환

자바 8과 마찬가지로, 코틀린은 SAM 변환을 지원한다. 이는 코틀린 함수 리터럴을 한 개의 추상 메서드를 가진 자바 인터페이스 구현으로
자동으로 변환한다는 것을 뜻한다. 코틀린 함수의 파라미터 타입이 인터페이스 메서드의 파라미터 타입과 일치하면 된다.

SAM 인터페이스의 인스턴스를 생성할 때 이를 사용할 수 있다:

``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

...그리고 메서드 호출에서 사용할 수 있다:

``` kotlin
val executor = ThreadPoolExecutor()
// Java signature: void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```

자바 클래스가 함수형 인터페이스를 인자로 받는 메서드를 여러 개 가지고 있다면, 어댑터 함수를 사용해서 호출할 메서드를 선택할 수 있다.
어댑터 함수는 람다를 특정한 SAM 타입으로 변환한다. 컴파일러는 또한 필요할 때 그런 어댑터 함수를 생성한다:

``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```

SAM 변환은 인터페이스에만 동작한다. 추상 클래스가 단지 1개의 추상 메서드만 갖고 있다 해서 추상 클래스에는 동작하지 않는다.

이런 특징은 자바 상호운용에서만 동작한다. 코틀린은 적당한 함수 타입이 있기 때문에,
함수를 코틀린 인터페이스 구현으로 자동으로 변환해주는 기능이 불필요하며 그래서 지원하지 않는다.

## 코틀린에서 JNI 사용하기
{:#using-jni-with-kotlin}

네이티브(C나 C++) 코드로 구현한 함수를 선언하려면 `external` 수식어를 지정해야 한다:

``` kotlin
external fun foo(x: Int): Double
```

프로시저의 나머지는 자바와 똑같게 동작한다.
