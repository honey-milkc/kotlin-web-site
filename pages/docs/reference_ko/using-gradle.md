---
type: doc
layout: reference_ko
title: "그레이들 사용하기"
---

# 그레이들 사용하기

그레이들에서 코틀린을 빌드하려면 [*kotlin-gradle* 플러그인을 설정](#plugin-and-versions)하고
프로젝트에 [그 플러그인을 적용](#targeting-the-jvm)하고, [*kotlin-stdlib* 의존](#configuring-dependencies)을 추가해야 한다.
인텔리J IDEA의 Tools \| Kotlin \| Configure Kotlin in Project 메뉴를 실행하면 자동으로 이를 처리해준다.

## 플러그인과 버전
{:#plugin-and-versions}

`kotlin-gradle-plugin`은 코틀린 코드와 모듈을 컴파일한다.

사용할 코틀린 버전은 보통 `kotlin_version` 프로퍼티로 정의한다:

``` groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'

    repositories {
        mavenCentral()
    }

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
```

코틀린 그레이들 플러그인 1.1.1과
[그레이들 플러그인 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block)을 사용하면
코틀린 버전을 설정할 필요가 없다.

## JVM 대상
{:#targeting-the-jvm}

JVM을 대상으로 하려면 코틀린 플러그인을 적용해야 한다:

``` groovy
apply plugin: "kotlin"
```

또는 코틀린 1.1.1부터 [그레이들 플러그인 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block)로
플러그인을 적용할 수 있다:

```groovy
plugins {
    id "org.jetbrains.kotlin.jvm" version "{{ site.data.releases.latest.version }}"
}
```
이 블록에서 `version`은 리터럴이어야 하며, 다른 빌드 스크립트로부터 적용할 수 없다.

같은 폴더나 다른 폴더에 코틀린 소스와 자바 소스를 함께 사용할 수 있다.
기본 관례는 다른 폴더를 사용하는 것이다:

``` groovy
project
    - src
        - main (root)
            - kotlin
            - java
```

기본 관례를 사용하지 않으려면 해당 *sourceSets* 프로퍼티를 수정해야 한다:

``` groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
    main.java.srcDirs += 'src/main/myJava'
}
```

## 자바스크립트 대상

자바스크립트를 대상으로 할 때에는 다른 플러그인을 적용해야 한다:

``` groovy
apply plugin: "kotlin2js"
```

이 플러그인은 코틀린 파일에만 작동하므로, (같은 프로젝트에서 자바 파일을 포함한다면) 코틀린과 자바 파일을
구분할 것을 권한다. JVM을 대상으로 하고 기본 관례를 사용하지 않으면,
*sourceSets*을 사용해서 소스 폴더를 지정해야 한다.

``` groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
}
```

플러그인은 자바스크립트 파일을 출력할 뿐만 아니라 바이너리 디스크립터를 가진 JS 파일을 추가로 생성한다.
다른 코틀린 모듈에서 의존할 수 있는 재사용가능한 라이브러리를 빌드하려면 이 파일이 필요하며,
변환 결과에 함께 배포해야 한다.
`kotlinOptions.metaInfo` 옵션으로 파일 생성을 설정한다:

``` groovy
compileKotlin2Js {
	kotlinOptions.metaInfo = true
}
```

## 안드로이드 대상

안드로이드의 그레이들 모델은 일반 그레이들 모델과 약간 다르다. 그래서 안드로이드 코틀린으로 작성한 안드로이드
프로젝트를 빌드하려면, *kotlin* 플러그인 대신 *kotlin-android* 플러그인이 필요하다.   

``` groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'

    ...

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
```

[표준 라이브러리 의존](#configuring-dependencies)도 설정해야 한다.

### 안드로이드 스튜디오

안드로이드 스튜디오를 사용하면 android 설정에 다음을 추가해야 한다:

``` groovy
android {
  ...

  sourceSets {
    main.java.srcDirs += 'src/main/kotlin'
  }
}
```

이 설정은 안드로이드 스튜디오가 코틀린 디렉토리를 소스 루트로 인식하도록 해서
프로젝트 모델을 IDE로 로딩할 때 올바르게 인식하도록 한다.
또는 보통 `src/main/java`에 위치한 자바 소스 디렉토리에 코틀린 클래스를 위치시킬 수도 있다.


## 의존 설정
{:#configuring-dependencies}

위에서 보여준 `kotlin-gradle-plugin` 의존 외에 코틀린 표준 라이브러리를 의존에 추가해야 한다.

``` groovy
repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib"
}
```

자바 스크립트를 대상으로 하면 대신 `compile "org.jetbrains.kotlin:kotlin-stdlib-js"`를 사용한다.

JDK 7이나 JDK 8이 대상이라면 코틀린 표준 라이브러리의 확장 버전을 사용할 수 있다.
확장 버전은 JDK 새 버전에 들어간 API를 위해 추가한 확장 함수를 포함한다.
`kotlin-stdlib` 대신 다음 의존 중 하나를 사용한다:

``` groovy
compile "org.jetbrains.kotlin:kotlin-stdlib-jre7"
compile "org.jetbrains.kotlin:kotlin-stdlib-jre8"
```

[코틀린 리플렉션](http://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect.full/index.html)이나 테스팅 도구를 사용하면
다음 의존을 추가로 설정한다:

``` groovy
compile "org.jetbrains.kotlin:kotlin-reflect"
testCompile "org.jetbrains.kotlin:kotlin-test"
testCompile "org.jetbrains.kotlin:kotlin-test-junit"
```

코틀린 1.1.2부터 `org.jetbrains.kotlin` 그룹에 속한 의존은 적용한 플러그인에서 가져온 버전을 기본으로 사용한다.
`compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"`과 같이 완전한 의존 표기를 사용해서 버전을 수동으로 제공할 수 있다.

## 애노테이션 처리

[코틀린 애노테이션 처리 도구](kapt.html) (`kapt`) 설명을 참고한다.

## 증분 컴파일

코틀린은 그레이들에서 선택적인 증분 컴파일을 지원한다. 증분 컴파일은 빌드 간 소스 파일의 변경을 추적해서
변경에 영향을 받는 파일만 컴파일한다.

코틀린 1.1.1부터 기본으로 증분 컴파일을 활성화한다.

기본 설정을 변경하는 몇 가지 방법이 있다:

  1. `gradle.properties`나 `local.properties` 파일에 `kotlin.incremental=true` 또는 `kotlin.incremental=false` 설정을 추가한다.

  2. 그레이들 명령행 파라미터로 `-Pkotlin.incremental=true` 또는 `-Pkotlin.incremental=false`를 추가한다. 이 경우 각 빌드마다 파라미터를 추가해야 하며
  증분 컴파일을 사용하지 않은 빌드가 존재하면 증분 캐시를 무효로한다.

증분 컴파일을 활성화하면, 빌드 로그에서 다음 경고 문구를 보게 된다:
```
Using kotlin incremental compilation
```

첫 번째 빌드는 증분이 아니다.

## 코루틴 지원

[코루틴](coroutines.html) 지원은 코틀린 1.1에서 실험 특징이므로, 프로젝트에서 코루틴을 사용하면 코틀린 컴파일러는 경고를 발생한다.
경고를 끄려면, `build.gradle`에 다음 블록을 추가한다:

``` groovy
kotlin {
    experimental {
        coroutines 'enable'
    }
}
```

## 모듈 이름

빌드로 생성한 코틀린 모듈은 프로젝트의 `archivesBaseName` 프로퍼티의 이름을 따른다.
프로젝트가 `lib`나 `jvm`과 같이 하위 프로젝트들에 공통인 넓은 의미의 이름을 사용하면,
모듈과 관련된 코틀린 출력 파일은(`*.kotlin_module`) 같은 이름을 갖는 외부 모듈의 출력 파일과 충돌할 수 있다.
이는 프로젝트를 한 개의 아카이브(예, APK)로 패키징할 때 문제를 발생시킨다.

이를 피하려면, 수동으로 유일한 `archivesBaseName`을 설정해야 한다:

``` groovy
archivesBaseName = 'myExampleProject_lib'
```

## 컴파일러 옵션

컴파일러 옵션을 추가로 설정하려면 코틀린 컴파일 태스크의 `kotlinOptions` 프로퍼티를 사용한다.

JVM 대상일 때, 제품을 위한 태스크는 `compileKotlin`이고 테스트 코드를 위한 태스크는 `compileTestKotlin`이다.
커스텀 소스 집합을 위한 태스크는 `compile<Name>Kotlin` 패턴에 따라 호출한다.
 
안드로이드 프로젝트에서 태스크의 이름은 [build variant](https://developer.android.com/studio/build/build-variants.html)를 포함하며,
`compile<BuildVariant>Kotlin` 패턴을 따른다(예, `compileDebugKotlin`, `compileReleaseUnitTestKotlin`).

자바스크립트 대싱일 때, 각각 `compileKotlin2Js`와 `compileTestKotlin2Js` 태스크를 부르고,
커스텀 소스 집합은 `compile<Name>Kotlin2Js` 태스크를 부른다.

단일 태스크를 설정하려면 그 이름을 사용한다. 다음은 예이다:

``` groovy
compileKotlin {
    kotlinOptions.suppressWarnings = true
}

compileKotlin {
    kotlinOptions {
        suppressWarnings = true
    }
}
```

프로젝트의 모든 코틀린 컴파일 태스크를 설정하는 것도 가능하다:

``` groovy
tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).all {
    kotlinOptions {
        // ...
    }
}
```

그레이들 태스크를 위한 전체 목록은 다음과 같다:

### JVM과 JS의 공통 속성

| 이름 | 설명 | 가능한 값 |기본 값|
|------|-------------|-----------------|--------------|
| `apiVersion` | 지정한 버전의 번들 라이브러리에서만 선언 사용을 허용함 | "1.0", "1.1" | "1.1" |
| `languageVersion` | 소스 호환성을 지정한 언어 버전으로 지정함 | "1.0", "1.1" | "1.1" |
| `suppressWarnings` | 경고를 생성하지 않음 |  | false |
| `verbose` | Enable verbose 로깅 출력을 활성화 |  | false |
| `freeCompilerArgs` | 추가 컴파일러 인자 목록 |  | [] |

### JVM 전용 속성

| 이름 | 설명 | 가능한 값 | 기본 값 |
|------|-------------|-----------------|--------------|
| `javaParameters` | 메서드 파라미터에 대해 자바 1.8 리플렉션을 위한 메타데이터 생성 |  | false |
| `jdkHome` | 클래스패스로 포함할 JDK 홈 디렉톨 경로(기본 JAVA_HOME과 다른 경우) |  |  |
| `jvmTarget` | 생성할 JVM 바이트코의 대상 버전(1.6 또는 1.8), 기본은 1.6 | "1.6", "1.8" | "1.6" |
| `noJdk` | 자바 런타임을 클래스패스에 포함시키지 않음 |  | false |
| `noReflect` | 코틀린 리플렉션 구현을 클래스패스에 포함시키지 않음 |  | true |
| `noStdlib` | 코틀린 런타임을 클래스패스에 포함시키지 않음 |  | true |

### JS 전용 속성

| 이름 | 설명 | 가능한 값 | 기본 값 |
|------|-------------|-----------------|--------------|
| `friendModulesDisabled` | internal 선언 추출을 비활성화 |  | false |
| `main` | 메인 함수를 불러야 할지 여부 | "call", "noCall" | "call" |
| `metaInfo` | 메타데이털르 가진 .meta.js와 .kjsm 파일 생성 여부. 라이브러리를 만들 때 사용함 |  | true |
| `moduleKind` | 컴파일러가 생성할 모듈의 종류 | "plain", "amd", "commonjs", "umd" | "plain" |
| `noStdlib` | 번들한 코틀린 표준 라이브러리를 사용하지 않을지 여부 |  | true |
| `outputFile` | 출력 파일 경로 |  |  |
| `sourceMap` | 소스맵 생성 여부 |  | false |
| `sourceMapEmbedSources` | 소스 파일을 소스맵에 삽입할지 여부 | "never", "always", "inlining" | "inlining" |
| `sourceMapPrefix` | 소스맵에 경로에 대한 접두사 |  |  |
| `target` | JS 파일을 생성할 때 사용할 ECMA 버전 지정 | "v5" | "v5" |
| `typedArrays` | 기본 타입 배열을 JS 타입 배열로 변환할지 여부 |  | false |


## 문서 생성

코틀린 프로젝트의 문서를 생성하려면 [Dokka](https://github.com/Kotlin/dokka)를 사용한다.
설정 명령어는 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md#using-the-gradle-plugin) 문서를 참고한다.
Dokka는 언어를 함께 사용하는 프로젝트를 지원하며 표준 JavaDoc을 포함한 다양한 형식으로 출력할 수 있다.

## OSGi

OSGi 지원은 [코틀린 OSGi 페이지](kotlin-osgi.html)를 참고한다.

## 예제

다음 예제는 그레이들 플러그인을 다양한 방법으로 설정하는 예를 보여준다:

* [코틀린](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/hello-world)
* [자바와 코틀린 혼합](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/mixed-java-kotlin-hello-world)
* [안드로이드](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-mixed-java-kotlin-project)
* [자바스크립트](https://github.com/JetBrains/kotlin/tree/master/libraries/tools/kotlin-gradle-plugin-integration-tests/src/test/resources/testProject/kotlin2JsProject)
