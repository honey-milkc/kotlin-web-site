---
type: doc
layout: reference_ko
title: "컴파일러 플러그인"
---

# 컴파일러 플러그인

## all-open 컴파일러 플러그인

코틀린은 기본적으로 클래스와 멤버가 `final`이다. 이는 `open`인 클래스를 요구하는 스프링 AOP와 같은 프레임워크나 라이브러리를 사용할 때 불편하다.
`all-open` 컴파일러 플러그인은 그런 프레임워크의 요구에 코틀린을 맞추고
`open` 키워드를 직접 사용하지 않아도 클래스에 특정 애노테이션을 붙이고 멤버를 open으로 만든다.    
예를 들어 스프링을 사용할 때 모든 클래스를 open으로 만들 필요가 없다.
단지 클래스에 `@Configuration`나 `@Service`와 같은 특정 애노테이션만 붙이면 된다.
`all-open` 플러그인은 이 애노테이션을 지정하는 것을 허용한다.

그레이들, 메이븐, IDE를 위한 all-open 플러그인을 제공한다.
스프링의 경우 `kotlin-spring` 컴파일러 플러그인을 사용한다([아래 참고](compiler-plugins.html#kotlin-spring-compiler-plugin)).

### all-open 플러그인을 사용하는 방법

`build.gradle`에 플러그인을 추가한다: 

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version"
    }
}

apply plugin: "kotlin-allopen"
```

또는 그레이들 플러그인 DSL을 사용한다면 `plugins` 블록에 all-open을 추가한다:

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.allopen" version "{{ site.data.releases.latest.version }}"
}
```

그리고 클래스를 open으로 만들 애노테이션을 지정한다:

```groovy
allOpen {
    annotation("com.my.Annotation")
}
```

클래스(또는 그 상위타입)에 `com.my.Annotation`이 붙이면, 클래스 그 자체와 클래스의 모든 멤버가 open이 된다.

또한 메타-애노테이션에도 동작한다:

``` kotlin
@com.my.Annotation
annotation class MyFrameworkAnnotation

@MyFrameworkAnnotation
class MyClass // 모드 open이 된다
```

`MyFrameworkAnnotation`에 `com.my.Annotation`을 붙였으므로
이 애노테이션 또한 클래스를 open으로 만든다. 

다음은 메이븐에서 all-open을 사용하는 방법이다:

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <configuration>
        <compilerPlugins>
            <!-- 또는 스프링 지원은 "spring" -->
            <plugin>all-open</plugin>
        </compilerPlugins>

        <pluginOptions>
            <!-- 각 줄에 애노테이션이 위치 -->
            <option>all-open:annotation=com.my.Annotation</option>
            <option>all-open:annotation=com.their.AnotherAnnotation</option>
        </pluginOptions>
    </configuration>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-allopen</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
    </dependencies>
</plugin>
```

{:#kotlin-spring-compiler-plugin}

### kotlin-spring 컴파일러 플러그인 Kotlin-spring compiler plugin

스프링 애노테이션을 직접 지정할 필요는 없다.
`kotlin-spring` 플러그인을 사용하면 자동으로 스프링의 요구에 맞게 all-open 플러그인을 설정한다:

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version"
    }
}

apply plugin: "kotlin-spring"
```

또는 그레이들 플러그인 DSL을 사용한다:

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.spring" version "{{ site.data.releases.latest.version }}"
}
```

메이븐 예제는 위와 유사하다.

플러그인은 다음 애노테이션을 지정한다. 
[`@Component`](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html), 
[`@Async`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/Async.html), 
[`@Transactional`](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html), 
[`@Cacheable`](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html). 
메타 애노테이션 지원 덕에 `@Configuration`, `@Controller`, `@RestController`, `@Service`, `@Repository`를 
붙인 클래스는 자동으로 open이 된다. 왜냐면 이들 애노테이션이 `@Component`를 가진 메타 애노테이션이기 때문이다.

물론, 한 프로젝트에서 `kotlin-allopen`과 `kotlin-spring`를 사용할 수 있다.
[start.spring.io](http://start.spring.io/#!language=kotlin)를 사용하면
`kotlin-spring` 플러그인이 기본으로 활성화된다.


### no-arg 컴파일러 플러그인

no-arg 플러그인은 지정한 애노테이션을 가진 클래스에 인자 없는 생성자를 추가로 생성한다.
생성한 생성자는 조작했기(synthetic) 때문에 자바나 코틀린에서 직접 호출할 수 없지만
리플레션으로 호출할 수는 있다.
코틀린이나 자바 관점에서 `data` 클래스는 인자 없는 생성자가 없지만 
JPA에서 `data` 클래스의 인스턴스를 생성할 수 있다([아래의](compiler-plugins.html#kotlin-jpa-compiler-plugin)
`kotlin-jpa` 설명을 보자).
 
### no-arg 플러그인을 사용하는 방법

사용법은 all-open과 매우 유사하다.
플러그인을 추가하고 인자 없는 생성자를 생성하고 싶은 애노테이션 목록을 지정한다.

다음은 그레이들에서의 no-arg 사용법이다. 

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-noarg:$kotlin_version"
    }
}

apply plugin: "kotlin-noarg"
```

또는 그레이들 플러그인 DSL을 사용한다:

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.noarg" version "{{ site.data.releases.latest.version }}"
}
```

인자 없는 생성자를 생성할 애노테이션 타입을 지정한다:

```groovy
noArg {
    annotation("com.my.Annotation")
}
```

플러그인이 생성하는 인자 없는 생성자가 초기화 로직을 수행하도록 하려면 `invokeInitializers` 옵션을 활성화한다.
코틀린 1.1.3-2부터 앞으로 해결할
[`KT-18667`](https://youtrack.jetbrains.com/issue/KT-18667)와 
[`KT-18668`](https://youtrack.jetbrains.com/issue/KT-18668) 때문에
기본으로 비활성화되었다:

```groovy
noArg {
    invokeInitializers = true
}
```

다음은 메이븐에서의 no-arg 사용법이다:

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <configuration>
        <compilerPlugins>
            <!-- Or "jpa" for JPA support -->
            <plugin>no-arg</plugin>
        </compilerPlugins>

        <pluginOptions>
            <option>no-arg:annotation=com.my.Annotation</option>
            <!-- Call instance initializers in the synthetic constructor -->
            <!-- <option>no-arg:invokeInitializers=true</option> -->
        </pluginOptions>
    </configuration>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-noarg</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
    </dependencies>
</plugin>
```

{:#kotlin-jpa-compiler-plugin}

### kotlin-jpa 컴파일러 플러그인

이 플러그인은 
[`@Entity`](http://docs.oracle.com/javaee/7/api/javax/persistence/Entity.html)와 
[`@Embeddable`](http://docs.oracle.com/javaee/7/api/javax/persistence/Embeddable.html)을
인자없는 생성자를 생성해야 할 마커로 사용한다.
그레이들에서 플러그인을 추가하는 방법은 다음과 같다: 

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-noarg:$kotlin_version"
    }
}

apply plugin: "kotlin-jpa"
```

또는 그레이들 플러그인 DSL을 사용해도 된다:

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.jpa" version "{{ site.data.releases.latest.version }}"
}
```

메이븐 예제는 위 설정과 유사하다.
