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

코틀린은 어플리케이션 사용할 수 있는 확장 표준 라이브러리를 제공한다. 다음 의존을 pom 파일에 설정한다:

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
 방법으로 maven-compiler-plugin보다 kotlin-maven-plugin을 먼저 실행해야 함을 의미한다:

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

## Jar file

To create a small Jar file containing just the code from your module, include the following under `build->plugins` in your Maven pom.xml file,
where `main.class` is defined as a property and points to the main Kotlin or Java class:

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

## Self-contained Jar file

To create a self-contained Jar file containing the code from your module along with dependencies, include the following under `build->plugins` in your Maven pom.xml file,
where `main.class` is defined as a property and points to the main Kotlin or Java class:

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

This self-contained jar file can be passed directly to a JRE to run your application:

``` bash
java -jar target/mymodule-0.0.1-SNAPSHOT-jar-with-dependencies.jar
```

## Targeting JavaScript

In order to compile JavaScript code, you need to use the `js` and `test-js` goals for the `compile` execution:

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

You also need to change the standard library dependency:

``` xml
<groupId>org.jetbrains.kotlin</groupId>
<artifactId>kotlin-stdlib-js</artifactId>
<version>${kotlin.version}</version>
```

For unit testing support, you also need to add a dependency on the `kotlin-test-js` artifact.

See the [Getting Started with Kotlin and JavaScript with Maven](/docs/tutorials/javascript/getting-started-maven/getting-started-with-maven.html)
tutorial for more information.

## Specifying compiler options

Additional options for the compiler can be specified as tags under the `<configuration>` element of the
Maven plugin node:

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>
    <executions>...</executions>
    <configuration>
        <nowarn>true</nowarn>  <!-- Disable warnings -->
    </configuration>
</plugin>
```

Many of the options can also be configured through properties:

``` xml
<project ...>
    <properties>
        <kotlin.compiler.languageVersion>1.0</kotlin.compiler.languageVersion>
    </properties>
</project>
```

The following attributes are supported:

### Attributes common for JVM and JS

| Name | Property name | Description | Possible values |Default value |
|------|---------------|-------------|-----------------|--------------|
| nowarn | | Generate no warnings | true, false | false |
| languageVersion | kotlin.compiler.languageVersion | Provide source compatibility with specified language version | "1.0", "1.1" | "1.1"
| apiVersion | kotlin.compiler.apiVersion | Allow to use declarations only from the specified version of bundled libraries | "1.0", "1.1" | "1.1"
| sourceDirs | | The directories containing the source files to compile | | The project source roots
| compilerPlugins | | Enabled [compiler plugins](compiler-plugins.html)  | | []
| pluginOptions | | Options for compiler plugins  | | []
| args | | Additional compiler arguments | | []


### Attributes specific for JVM

| Name | Property name | Description | Possible values |Default value |
|------|---------------|-------------|-----------------|--------------|
| jvmTarget | kotlin.compiler.jvmTarget | Target version of the generated JVM bytecode | "1.6", "1.8" | "1.6" |
| jdkHome | kotlin.compiler.jdkHome |  	Path to JDK home directory to include into classpath, if differs from default JAVA_HOME | | |

### Attributes specific for JS

| Name | Property name | Description | Possible values |Default value |
|------|---------------|-------------|-----------------|--------------|
| outputFile | | Output file path | | |
| metaInfo |  | Generate .meta.js and .kjsm files with metadata. Use to create a library | true, false | true
| sourceMap | | Generate source map | true, false | false
| sourceMapEmbedSources | | Embed source files into source map | "never", "always", "inlining" | "inlining" |
| sourceMapPrefix | | Prefix for paths in a source map |  |  |
| moduleKind | | Kind of a module generated by compiler | "plain", "amd", "commonjs", "umd" | "plain"

## Generating documentation

The standard JavaDoc generation plugin (`maven-javadoc-plugin`) does not support Kotlin code.
To generate documentation for Kotlin projects, use [Dokka](https://github.com/Kotlin/dokka);
please refer to the [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md#using-the-maven-plugin)
for configuration instructions. Dokka supports mixed-language projects and can generate output in multiple
formats, including standard JavaDoc.

## OSGi

For OSGi support see the [Kotlin OSGi page](kotlin-osgi.html).

## Examples

An example Maven project can be [downloaded directly from the GitHub repository](https://github.com/JetBrains/kotlin-examples/archive/master/maven.zip)
