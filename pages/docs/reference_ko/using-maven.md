---
type: doc
layout: reference
title: "메이븐 사용하기"
description: "이 튜토리얼은 코틀린 코드를 포함한 어플리케이션을 메이븐을 사용해서 제작하는 여러 방법을 보여준다"
---

# 메이븐 사용하기

## 플러그인과 버전

*kotlin-maven-plugin*은 코틀린 소스와 모듈을 컴파일한다. 현재는 메이븐 V3만 지원한다.

*kotlin.version* 프로퍼티로 사용할 코틀린 버전을 정의한다:

``` xml
<properties>
    <kotlin.version>{{ site.data.releases.latest.version }}</kotlin.version>
</properties>
```

## 의존

코틀린은 어플리케이션이 사용할 수 있는 확장 표준 라이브러리를 제공한다. 다음 의존을 pom 파일에 설정한다:

``` xml
<dependencies>
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-stdlib</artifactId>
        <version>${kotlin.version}</version>
    </dependency>
</dependencies>
```


JDK 7이나 JDK 8이 대상이라면 코틀린 표준 라이브러리의 확장 버전을 사용할 수 있다.
확장 버전은 JDK 새 버전에 들어간 API를 위해 추가한 확장 함수를 포함한다.
`kotlin-stdlib` 대신 사용할 JDK 버전에 따라 다음 의존 중 하나를 사용한다:
JDK 7이나 JDK 8을 `kotlin-stdlib-jre7` 또는 `kotlin-stdlib-jre8`을 사용한다. 


[코틀린 리플렉션](/api/latest/jvm/stdlib/kotlin.reflect.full/index.html)이나 테스팅 도구를 사용할 경우
리플렉션 라이브러리를 위한 아티팩트 ID는 `kotlin-reflect`이고 테스트 라이브러리를 위한 아티팩트 ID는 
`kotlin-test`와 `kotlin-test-junit`이다.

## 코틀리 소스코드만 컴파일하기

소스 코드를 컴파일하려면 <build> 태그에 소스 디렉토리를 지정한다:

``` xml
<build>
    <sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
    <testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
</build>
```

소스를 컴파일할 때 코틀린 메이븐 플러그인을 참조해야 한다:

``` xml
<build>
    <plugins>
        <plugin>
            <artifactId>kotlin-maven-plugin</artifactId>
            <groupId>org.jetbrains.kotlin</groupId>
            <version>${kotlin.version}</version>

            <executions>
                <execution>
                    <id>compile</id>
                    <goals> <goal>compile</goal> </goals>
                </execution>

                <execution>
                    <id>test-compile</id>
                    <goals> <goal>test-compile</goal> </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 코틀린과 자바 소스 컴파일

자바와 코틀린을 함께 사용하는 코드를 컴파일하려면 자바 컴파일러보다 코틀린 컴파일러는 먼저 호출해야 한다.
메이븐 관점에서는 다음과 같이 pom.xml 파일에서 maven-compiler-plugin 위에 코틀린 플러그인을 위치시켜서
maven-compiler-plugin보다 kotlin-maven-plugin을 먼저 실행해야 함을 의미한다:

``` xml
<build>
    <plugins>
        <plugin>
            <artifactId>kotlin-maven-plugin</artifactId>
            <groupId>org.jetbrains.kotlin</groupId>
            <version>${kotlin.version}</version>
            <executions>
                <execution>
                    <id>compile</id>
                    <goals> <goal>compile</goal> </goals>
                    <configuration>
                        <sourceDirs>
                            <sourceDir>${project.basedir}/src/main/kotlin</sourceDir>
                            <sourceDir>${project.basedir}/src/main/java</sourceDir>
                        </sourceDirs>
                    </configuration>
                </execution>
                <execution>
                    <id>test-compile</id>
                    <goals> <goal>test-compile</goal> </goals>
                    <configuration>
                        <sourceDirs>
                            <sourceDir>${project.basedir}/src/test/kotlin</sourceDir>
                            <sourceDir>${project.basedir}/src/test/java</sourceDir>
                        </sourceDirs>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.5.1</version>
            <executions>
                <!-- Replacing default-compile as it is treated specially by maven -->
                <execution>
                    <id>default-compile</id>
                    <phase>none</phase>
                </execution>
                <!-- Replacing default-testCompile as it is treated specially by maven -->
                <execution>
                    <id>default-testCompile</id>
                    <phase>none</phase>
                </execution>
                <execution>
                    <id>java-compile</id>
                    <phase>compile</phase>
                    <goals> <goal>compile</goal> </goals>
                </execution>
                <execution>
                    <id>java-test-compile</id>
                    <phase>test-compile</phase>
                    <goals> <goal>testCompile</goal> </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 증분 컴파일

빌드를 더 빠르게 하기 위해, 메이븐을 위한 증분 컴파일을 활성화할 수 있다(코틀린 1.1.2부터 지원).
증분 컴파일을 하려면 `kotlin.compiler.incremental` 프로퍼티를 정의한다:

``` xml
<properties>
    <kotlin.compiler.incremental>true</kotlin.compiler.incremental>
</properties>
```

대신, `-Dkotlin.compiler.incremental=true` 옵션을 사용해서 빌드를 실행해도 된다.

## 애노테이션 처리

[코틀린 애노테이션 처리 도구](kapt.html) (`kapt`)의 설명을 참고한다.

## Jar 파일

모듈의 코드만 포함하는 작은 Jar 파일을 만들려면 메이븐 pom.xml 파일의 `build->plugins`에 다음 설정을 포함한다.
`main.class` 프로퍼티는 코틀린이나 자바 메인 클래스를 가리킨다.

``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>2.6</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>${main.class}</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

## 독립(Self-contained) Jar 파일

모듈의 코드와 의존을 포함한 독립 Jar 파일을 만드려면 메이븐 pom.xml 파일의 `build->plugins`에 다음 설정을 포함한다.
`main.class` 프로퍼티는 코틀린이나 자바 메인 클래스를 가리킨다.

``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <version>2.6</version>
    <executions>
        <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals> <goal>single</goal> </goals>
            <configuration>
                <archive>
                    <manifest>
                        <mainClass>${main.class}</mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </execution>
    </executions>
</plugin>
```

이 독립 jar 파일을 JRE에 바로 전달하면 어플리케이션을 실행할 수 있다:

``` bash
java -jar target/mymodule-0.0.1-SNAPSHOT-jar-with-dependencies.jar
```

## 자바스크립트 대상

자바스크립트 코드를 컴파일하려면 `compile` 실행의 goal로 `js`와 `test-js`를 사용해야 한다:

``` xml
<plugin>
    <groupId>org.jetbrains.kotlin</groupId>
    <artifactId>kotlin-maven-plugin</artifactId>
    <version>${kotlin.version}</version>
    <executions>
        <execution>
            <id>compile</id>
            <phase>compile</phase>
            <goals>
                <goal>js</goal>
            </goals>
        </execution>
        <execution>
            <id>test-compile</id>
            <phase>test-compile</phase>
            <goals>
                <goal>test-js</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

또한 표준 라이브러리 의존을 변경해야 한다:

``` xml
<groupId>org.jetbrains.kotlin</groupId>
<artifactId>kotlin-stdlib-js</artifactId>
<version>${kotlin.version}</version>
```

단위 테스팅을 지원하려면 `kotlin-test-js`를 의존으로 추가한다.

더 많은 정보는 
[Getting Started with Kotlin and JavaScript with Maven](/docs/tutorials/javascript/getting-started-maven/getting-started-with-maven.html)
튜토리얼을 참고한다.

## 컴파일러 옵션 지정

컴파일러를 위한 추가 옵션을 메이븐 플러그인 노드의 `<configuration>` 요소에 태그로 지정할 수 있다:

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>
    <executions>...</executions>
    <configuration>
        <nowarn>true</nowarn>  <!-- 경고 비활성화 -->
    </configuration>
</plugin>
```

프로퍼티를 통해 다양한 옵션을 설정할 수 있다:

``` xml
<project ...>
    <properties>
        <kotlin.compiler.languageVersion>1.0</kotlin.compiler.languageVersion>
    </properties>
</project>
```

다음 속성을 지원한다:

### Attributes common for JVM and JS

| 이름 | 프로퍼티 이름 | 설명 | 가능한 값 |기본 값 |
|------|---------------|-------------|-----------------|--------------|
| nowarn | | 경고를 생성하지 않음 | true, false | false |
| languageVersion | kotlin.compiler.languageVersion | 소스 호환성을 지정한 언어 버전으로 지정함 | "1.0", "1.1" | "1.1"
| apiVersion | kotlin.compiler.apiVersion | 지정한 버전의 번들 라이브러리에서만 선언 사용을 허용함 | "1.0", "1.1" | "1.1"
| sourceDirs | | 컴파일할 소스 파일을 포함한 폴더목록 | | 프로젝트 소스 루트
| compilerPlugins | | 활성화함 [컴파일러 플러그인](compiler-plugins.html)  | | []
| pluginOptions | | 컴파일러 플러그인을 위한 옵션 | | []
| args | | 추가 컴파일러 인자 | | []


### JVM 전용 속성

| 이름 | 프로퍼티 이름 | 설명 | 가능한 값 |기본 값 |
|------|---------------|-------------|-----------------|--------------|
| jvmTarget | kotlin.compiler.jvmTarget | 생성할 JVM 바이트코드의 대상 버전 | "1.6", "1.8" | "1.6" |
| jdkHome | kotlin.compiler.jdkHome |  	클래스패스로 포함할 JDK 홈 디렉톨 경로(기본 JAVA_HOME과 다른 경우) | | |

### JS 전용 속성

| 이름 | 프로퍼티 이름 | 설명 | 가능한 값 |기본 값 |
|------|---------------|-------------|-----------------|--------------|
| outputFile | | 출력 파일 경로 | | |
| metaInfo |  | 메타데이털르 가진 .meta.js와 .kjsm 파일 생성 여부. 라이브러리를 만들 때 사용함 | true, false | true
| sourceMap | | 소스맵 생성 여부 | true, false | false
| sourceMapEmbedSources | | 소스 파일을 소스맵에 삽입할지 여부 | "never", "always", "inlining" | "inlining" |
| sourceMapPrefix | | 소스맵에 경로에 대한 접두사 |  |  |
| moduleKind | | 컴파일러가 생성할 모듈의 종류 | "plain", "amd", "commonjs", "umd" | "plain"

## 문서 생성

표준 JavaDoc 생성 플러그인(`maven-javadoc-plugin`)은 코틀린 코드를 지원하지 않는다.
코틀린 프로젝트를 위한 문서를 생성하려면 [Dokka](https://github.com/Kotlin/dokka)를 사용한다.
설정 명령어는 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md#using-the-gradle-plugin) 문서를 참고한다.
Dokka는 언어를 함께 사용하는 프로젝트를 지원하며 표준 JavaDoc을 포함한 다양한 형식으로 출력할 수 있다.

## OSGi

OSGi 지원은 [코틀린 OSGi 페이지](kotlin-osgi.html)를 참고한다.

## 예제

메이븐 예제 프로젝트는 [깃헙 리포지토리에서 바로 다운로드](https://github.com/JetBrains/kotlin-examples/archive/master/maven.zip)받을 수 있다.
