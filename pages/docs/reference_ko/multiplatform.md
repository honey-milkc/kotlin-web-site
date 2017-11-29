---
type: doc
layout: reference
category: "Other"
title: "멀티플랫폼 프로젝트 (프리뷰)"
---

# 멀티플랫폼 프로젝트 (프리뷰)

> 멀티플랫폼 프로젝트는 코틀린 1.2에 새로 추가된 실험 특징이다. 이 문서에서 설명하는 모든 언어와 도구 특징은
향후 코틀린 버전에서 바뀔 수 있다.
{:.note}

코틀린 멀티플랫폼 프로젝트는 같은 코드를 여러 대상 플랫폼으로 컴파이할 수 있게 해준다 현재 지원하는 플랫폼은 JVM과 JS이며
앞으로 네이티브를 추가할 예정이다.

## 멀티플랫폼 프로젝트 구조

멀티플랫폼 프로젝트는 세 타입의 모듈로 구성된다:

  * _공통_ 모듈은 특정 플랫폼에 종속되지 않은 코드를 포함하고 또한 플랫폼에 의존적인 API의 구현 없는 선언을 포함한다.
    이 선언은 공통 코드가 플랫폼에 종속된 구현에 의존할 수 있게 한다.
  * _플랫폼_ 모듈은 특정 플랫폼에 대해 공통 모듈에 선언된 플랫폼 의존적인 선언의 구현을 포함한다. 또한 다른 플랫폼 의존적인 코드를 포함한다.
    플랫폼 모듈은 항상 한 개 공통 모듈의 구현체이다.
  * 일반 모듈. 이 모듈은 특정 플랫폼을 대상으로 하며, 플랫폼 모듈의 의존이 되거나 플랫폼 모듈에 의존할 수 있다.

공통 모듈은 오직 다른 공통 모듈과 코틀린 표준 라이브러리(`kotlin-stdlib-common`)의 공통 버전을 포함한 다른 라이브러리에만 의존할 수 있다.
공통 모듈은 오직 코틀린 코드만 포함하며 다른 언어로 만든 코드는 포함하지 않는다.

플랫폼 모듈은 모든 모듈과 주언 플랫폼에서 사용 가능한 라이브러리(코틀린/JVM의 경우 자바 라이브러리를 포함하고 코틀린/JS는 JS 라이브러리를 포함)에 의존할 수 있다.
코틀린/JVM을 대상으로 하는 플랫폼 모듈은 또한 자바와 다른 JVM 언어로 작성한 코드를 포함할 수 있다.

공통 모듈을 컴파일하면 특별한 _메타데이터_ 파일을 생성하는데, 이 파일은 모듈의 모든 선언을 포함한다.
플랫폼 모듈을 컴파일하면 플랫폼 모듈 및 그 모듈이 구현한 공통 모듈에 속한 코드에 대해 대상에 특화된 코드(JVM 바이트코드 또는 JS 소스 코드)를 생성한다. 

따라서 각 멀티플랫폼 라이브러리는 아티팩트 집합으로 배포해야 한다. 이 집합은
공통 코드를 위한 메타데이터를 포함하는 공통 .jar와 각 플랫폼에 맞게 컴파일된 구현 코드를 포함하는 플랫폼에 특화된 .jars를 포함한다. 


## 멀티플랫폼 프로젝트 구성

코틀린 1.2에서 그레이들을 사용해서 멀티플랫폼 프로젝트를 구성해야 한다. 다른 빌드 시스템은 지원하지 않는다.

IDE에서 신규 멀티플랫폼 프로젝트를 만들려면, New Project 대화창에서 "Kotlin" 아래의 "Kotlin (Multiplatform)" 옵션을 선택한다.
이는 공통 모듈과 JVM과 JS를 위한 두 개의 플랫폼 모듈을 생성할 것이다. 모듈을 추가하려면, New Module 대화창에서
"Gradle" 아래의 "Kotlin (Multiplatform)" 옵션 중 하나를 선택한다.

프로젝트를 수동으로 설정해야 한다면 다음 단계를 따른다:

  * buildscript 클래스패스에 코틀린 그레이들 플러그인을 추가한다: `classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"`
  * 공통 모듈에 `kotlin-platform-common` 플러그인을 적용한다
  * 공통 모듈에 `kotlin-stdlib-common` 의존을 추가한다
  * JVM과 JS 플랫폼 모듈에 `kotlin-platform-jvm`과 `kotlin-platform-js` 플러그인을 적용한다
  * 플랫폼 모듈에서 공통 모듈로의 의존을 `implement` 범위로 추가한다

다음 예제는 코틀린 1.2-Beta를 사용한 공통 모듈의 완전한 `build.gradle` 파일을 보여준다:

``` groovy
buildscript {
    ext.kotlin_version = '1.2-Beta'

    repositories {
        maven { url 'http://dl.bintray.com/kotlin/kotlin-eap-1.2' }
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'kotlin-platform-common'

repositories {
    maven { url 'http://dl.bintray.com/kotlin/kotlin-eap-1.2' }
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-common:$kotlin_version"
    testCompile "org.jetbrains.kotlin:kotlin-test-common:$kotlin_version"
}
```

And the example below shows a complete `build.gradle` for a JVM module. Pay special
attention to the `implement` line in the `dependencies` block:

``` groovy
buildscript {
    ext.kotlin_version = '1.2-Beta'

    repositories {
        maven { url 'http://dl.bintray.com/kotlin/kotlin-eap-1.2' }
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'kotlin-platform-jvm'

repositories {
    maven { url 'http://dl.bintray.com/kotlin/kotlin-eap-1.2' }
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    implement project(":")
    testCompile "junit:junit:4.12"
    testCompile "org.jetbrains.kotlin:kotlin-test-junit:$kotlin_version"
    testCompile "org.jetbrains.kotlin:kotlin-test:$kotlin_version"
}
```


## Platform-Specific Declarations

One of the key capabilities of Kotlin's multiplatform code is a way for common code to
depend on platform-specific declarations. In other languages, this can often be accomplished
by building a set of interfaces in the common code and implementing these interfaces in platform-specific
modules. However, this approach is not ideal in cases when you have a library on one of the platforms
that implements the functionality you need, and you'd like to use the API of this library directly
without extra wrappers. Also, it requires common declarations to be expressed as interfaces, which
doesn't cover all possible cases.

As an alternative, Kotlin provides a mechanism of _expected and actual declarations_.
With this mechanism, a common module can define _expected declarations_, and a platform module
can provide _actual declarations_ corresponding to the expected ones. 
To see how this works, let's look at an example first. This code is part of a common module:

``` kotlin
package org.jetbrains.foo

expect class Foo(bar: String) {
    fun frob()
}

fun main(args: Array<String>) {
    Foo("Hello").frob()
}
```

And this is the corresponding JVM module:

``` kotlin
package org.jetbrains.foo

actual class Foo actual constructor(val bar: String) {
    actual fun frob() {
        println("Frobbing the $bar")
    }
}
```

This illustrates several important points:

  * An expected declaration in the common module and its actual counterparts always
    have exactly the same fully qualified name.
  * An expected declaration is marked with the `expect` keyword; the actual declaration
    is marked with the `actual` keyword.
  * All actual declarations that match any part of an expected declaration need to be marked
    as `actual`.
  * Expected declarations never contain any implementation code.

Note that expected declarations are not restricted to interfaces and interface members.
In this example, the expected class has a constructor and can be created directly from common code.
You can apply the `expect` modifier to other declarations as well, including top-level declarations and
annotations:

``` kotlin
// Common
expect fun formatString(source: String, vararg args: Any): String

expect annotation class Test

// JVM
actual fun formatString(source: String, vararg args: Any) =
    String.format(source, args)
    
actual typealias Test = org.junit.Test
```

The compiler ensures that every expected declaration has actual declarations in all platform
modules that implement the corresponding common module, and reports an error if any actual declarations are 
missing. The IDE provides tools that help you create the missing actual declarations.

If you have a platform-specific library that you want to use in common code while providing your own
implementation for another platform, you can provide a typealias to an existing class as the actual
declaration:

``` kotlin
expect class AtomicRef<V>(value: V) {
  fun get(): V
  fun set(value: V)
  fun getAndSet(value: V): V
  fun compareAndSet(expect: V, update: V): Boolean
}

actual typealias AtomicRef<V> = java.util.concurrent.atomic.AtomicReference<V>
```
