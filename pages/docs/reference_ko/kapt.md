---
type: doc
layout: reference_ko
title: "kapt 사용하기"
---

# 코틀린 애노테이션 처리 도구 사용하기

코틀린 플러그인은 _Dagger_나 _DBFlow_와 같은 애노테이션 처리기를 지원한다.
애노테이션 처리기를 코틀린 클래스에 사용하려면 `kotlin-kapt` 플러그인을 적용한다.

## 그레이들 설정

``` groovy
apply plugin: 'kotlin-kapt'
```

또는, 코틀린 1.1.1부터 DSL 플러그인을 사용해서 애노테이션 처리기를 적용할 수 있다:

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

이미 [android-apt](https://bitbucket.org/hvisser/android-apt) 플러그인을 사용한다면 `build.gradle` 파일에서 그 설정을 제거하고
 `apt` 설정을 `kapt`로 대체한다.
프로젝트가 자바 클래스를 포함한다면, `kapt`가 자바 클래스도 처리한다.

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

이 옵션은 실험 단계이므로 기본으로 비활성화되어 있다.


## 메이븐 설정 (코틀린 1.1.2부터)

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
애노테이션 처리를 다시 실행하고 싶다면“Maven Projects”툴바에서 빌드를 실행한다.
