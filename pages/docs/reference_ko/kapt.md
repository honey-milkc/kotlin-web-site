---
type: doc
layout: reference_ko
title: "kapt 사용하기"
---

# 코틀린을 이용한 애노테이션 처리

코틀린은 *kapt* 컴파일러 플러그인을 통해서 애노테이션 처리기([JSR 269](https://jcp.org/en/jsr/detail?id=269) 참고)를 지원한다.

간단히 설명하면, 코틀린 프로젝트에서 [Dagger](https://google.github.io/dagger/)나 [Data Binding](https://developer.android.com/topic/libraries/data-binding/index.html)와 같은
라이브러리를 사용할 수 있다.

*kapt* 플러그인을 그레이들/메이븐 빌드에 적용하는 방법을 아래를 참고한다.

## 그레이들에서 사용하기
{:#using-in-gradle}

`kotlin-kapt` 그레이들 플러그인을 적용한다:

```groovy
apply plugin: 'kotlin-kapt'
```

또는 플러그인 DSL을 사용해서 적용할 수 있다:

``` groovy
plugins {
    id "org.jetbrains.kotlin.kapt" version "{{ site.data.releases.latest.version }}"
}
```

`dependencies` 블록에 `kapt` 설정을 사용해서 각각의 의존을 추가한다:

``` groovy
dependencies {
    kapt 'groupId:artifactId:version'
}
```

이미 애노테이션 처리기를 위해 [안드로이드 지원](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html#annotationProcessor_config)을 사용했다면,
`annotationProcessor` 설정을 `kapt`로 대체하라.
프로젝트에 포함된 자바 클래스도 `kapt`가 처리해준다.

`androidTest`나 `test` 소스에 대해 애노테이션 처리기를 사용하려면
각 `kapt` 설정에 대해 `kaptAndroidTest`와 `kaptTest` 이름을 사용한다.
`kaptAndroidTest`와 `kaptTest`는 `kapt`를 확장했으므로 `kapt` 의존을 그대로 적용할 수 있고, 
제품 소스와 테스트 소스에 사용할 수 있다.
 
일부 애노테이션 처리기는(`AutoFactory`와 같은) 선언 시그너처의 정확한 타입에 의존한다.
기본적으로 Kapt는 모든 알 수 없는 타입(생성된 클래스의 타입 포함)을 `NonExistentClass`로 대체하는데,
이 기능이 다르게 동작하게 바꿀 수 있다.
`build.gradle` 파일에 다음 플래그를 추가하면 스텁의 에러 타입 추론이 가능해진다:

``` groovy
kapt {
    correctErrorTypes = true
}
```

## 메이븐에서 사용하기

`compile` 전에 kotlin-maven-plugin의 `kapt` goal을 실행하는 설정을 추가한다: 

```xml
<execution>
    <id>kapt</id>
    <goals>
        <goal>kapt</goal>
    </goals>
    <configuration>
        <sourceDirs>
            <sourceDir>src/main/kotlin</sourceDir>
            <sourceDir>src/main/java</sourceDir>
        </sourceDirs>
        <annotationProcessorPaths>
            <!-- Specify your annotation processors here. -->
            <annotationProcessorPath>
                <groupId>com.google.dagger</groupId>
                <artifactId>dagger-compiler</artifactId>
                <version>2.9</version>
            </annotationProcessorPath>
        </annotationProcessorPaths>
    </configuration>
</execution>
```
 
[코틀린 예제 리포지토리](https://github.com/JetBrains/kotlin-examples/tree/master/maven/dagger-maven-example)에서
코틀린, 메이븐, Dagger의 사용 예를 보여주는 완전한 예제 프로젝트를 찾을 수 있다.

인텔리J IDEA 자체의 빌드 시스템은 아직 kapt를 지원하지 않는다.
애노테이션 처리를 다시 실행하고 싶다면 “Maven Projects” 툴바에서 빌드를 실행한다.


## CLI에서 사용하기

Kapt 컴파일러 플러그인은 코틀린 컴파일러 바이너리 배포판에 포함되어 있다.

`Xplugin` kotlinc  옵션을 사용해서 플러그인 JAR 파일에 대한 경로를 제공해서 플러그인을 붙일 수 있다:

```bash
-Xplugin=$KOTLIN_HOME/lib/kotlin-annotation-processing.jar
```

다음은 사용가능한 옵션 목록이다:

* `sources` (*필수*): 생성할 파일을 위한 출력 경로.
* `classes` (*필수*): 생성할 클래스 파일과 자원을 위한 출력 결과.
* `stubs` (*필수*): 스텁 파일을 위한 출력 경로. 즉, 임시 디렉토리.
* `incrementalData`: 바이너리 스텁을 위한 출력 경로.
* `apclasspath` (*반복가능*): 애노테이션 처리기 JAR 파일 경로. 각 JAR 만큼 `apclasspath`를 설정할 수 있다.
* `apoptions`: BASE64로 인코딩한 애노테이션 처리기 옵션 목록. [AP/javac 옵션 인코딩](#apjavac-options-encoding) 참고.
* `javacArguments`: BASE64로 인코딩한 javac에 전달될 옵션 목록. [AP/javac 옵션 인코딩](#apjavac-options-encoding) 참고.
* `processors`: 애노테이션 처리기의 완전한 클래스 이름 목록. 콤마로 구분. 지정할 경우 kapt는 `apclasspath`에서 애노테이션 처리기를 찾는 시도를 하지 않는다.
* `verbose`: 상세 출력 활성화.
* `aptMode` (*필수*)
    * `stubs` – 애노테이션 처리에 필요한 스텁만 생성
    * `apt` – 애노테이션 처리만 실행
    * `stubsAndApt` – 스텁을 생성하고 애노테이션 처리 실행
* `correctErrorTypes`: [앞에](#using-in-gradle) 참고. 기본으로 비활성화

플러그인 옵션 형식은 `-P plugin:<plugin id>:<key>=<value>`이다. 옵션을 반복할 수 있다.

예제:

```bash
-P plugin:org.jetbrains.kotlin.kapt3:sources=build/kapt/sources
-P plugin:org.jetbrains.kotlin.kapt3:classes=build/kapt/classes
-P plugin:org.jetbrains.kotlin.kapt3:stubs=build/kapt/stubs

-P plugin:org.jetbrains.kotlin.kapt3:apclasspath=lib/ap.jar
-P plugin:org.jetbrains.kotlin.kapt3:apclasspath=lib/anotherAp.jar

-P plugin:org.jetbrains.kotlin.kapt3:correctErrorTypes=true
```


## 코틀린 소스 생성하기

Kapt는 코틀린 소스를 생성할 수 있다. `processingEnv.options["kapt.kotlin.generated"]`로 지정한 디렉토리에 생성한 코틀린 소스 파일을 작성하고
메인 소스와 함께 생성한 파일을 컴파일한다.

[kotlin-examples](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/kotlin-code-generation) 깃헙 리포지토리에서 완전한 예제를 찾을 수 있다..

Kapt는 생성한 코틀린 파일에 대한 다중 처리(multiple round)를 지원하지 않는다.


## AP/javac 옵션 인코딩
{:#apjavac-options-encoding}

`apoptions`와 `javacArguments` CLI 옵션은 옵션의 인코딩된 맵을 허용한다.
다음은 직접 옵션을 인코딩하는 방법을 보여준다:

```kotlin
fun encodeList(options: Map<String, String>): String {
    val os = ByteArrayOutputStream()
    val oos = ObjectOutputStream(os)

    oos.writeInt(options.size)
    for ((key, value) in options.entries) {
        oos.writeUTF(key)
        oos.writeUTF(value)
    }

    oos.flush()
    return Base64.getEncoder().encodeToString(os.toByteArray())
}
```
