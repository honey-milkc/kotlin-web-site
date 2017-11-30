---
type: doc
layout: reference_ko
title: "앤트 사용하기"
description: "이 튜토리얼은 코틀린 코드를 포함한 어플리케이션을 앤트를 사용해서 제작하는 여러 방법을 보여준다"
---

# 앤트 사용하기

## 앤트 태스크 얻기

코틀린은 앤트에서 사용할 세 개 태스크를 제공한다:

* kotlinc: JVM 대상 코틀린 컴파일러
* kotlin2js: 자바스크립트 대상 코틀린 컴파일러
* withKotlin: 표준 *javac* 앤트 태스크를 사용할 때 코틀린 파일을 컴파일하기 위한 태스크

이 태스크는 *kotlin-ant.jar* 라이브러리에 정의되어 있다.
이 파일은 [코틀린 컴파일러]({{site.data.releases.latest.url}})의 *lib* 폴더에 있다.
앤트 1.9+ 버전이 필요하다.

## 코틀린 소스만 있는 경우의 JVM 대상

프로젝트가 코틀린 소스 코드로만 구성되어 있을 경우, 프로젝트를 컴파일하는 가장 쉬운 방법은 *kotlinc* 태스크를 사용하는 것이다:

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlinc src="hello.kt" output="hello.jar"/>
    </target>
</project>
```

${kotlin.lib}는 코틀린 표준 컴파일러 압축을 푼 폴더를 가리킨다.

## 코틀린 소스만 있고 여러 루트를 가진 경우의 JVM 대상

프로젝트에 소스 루트가 여러 개인 경우, 경로를 정의하기 위해 요소로 *src*를 사용한다:

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlinc output="hello.jar">
            <src path="root1"/>
            <src path="root2"/>
        </kotlinc>
    </target>
</project>
```

## 코틀린과 자바 소스를 가진 경우의 JVM 대상

프로젝트가 코틀린과 자바 소스 코드로 구성되어 있으면, *kotlinc*를 사용할 수 있지만
태스크 파라미터 중복을 피하기 위해 *withKotlin* 태스크를 사용할 것을 권한다:

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <delete dir="classes" failonerror="false"/>
        <mkdir dir="classes"/>
        <javac destdir="classes" includeAntRuntime="false" srcdir="src">
            <withKotlin/>
        </javac>
        <jar destfile="hello.jar">
            <fileset dir="classes"/>
        </jar>
    </target>
</project>
```

`<withKotlin>`을 위한 추가 명령행 인자를 지정하려면 중첩한 `<compilerArg>` 파라미터를 사용한다.
사용할 수 있는 전체 인자 목록은 `kotlinc -help`로 확인할 수 있다.
`moduleName` 속성을 사용해서 컴파일할 모듈의 이름을 지정할 수 있다:

``` xml
<withKotlin moduleName="myModule">
    <compilerarg value="-no-stdlib"/>
</withKotlin>
```


## 단일 소스 폴더를 가진 자바 스크립트 대상

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlin2js src="root1" output="out.js"/>
    </target>
</project>
```

## 접두사, 접미사, 소스맵 옵션을 가진 자바스크립트 대상

``` xml
<project name="Ant Task Test" default="build">
    <taskdef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <kotlin2js src="root1" output="out.js" outputPrefix="prefix" outputPostfix="postfix" sourcemap="true"/>
    </target>
</project>
```

## 단일 소스 폴더와 metaInfo 옵션을 가진 자바스크립트 대상

코틀린/자바스크립트 라이브러리로 변환 결과를 배포하고 싶을 때 `metaInfo` 옵션이 유용하다.
`metaInfo`를 true로 설정하면, 컴파일 하는 동안 바이너리 메타데이터를 가진 JS 파일을 추가로 생성한다.
변환 결과와 함께 이 파일을 배포해야 한다:

``` xml
<project name="Ant Task Test" default="build">
    <typedef resource="org/jetbrains/kotlin/ant/antlib.xml" classpath="${kotlin.lib}/kotlin-ant.jar"/>

    <target name="build">
        <!-- out.meta.js will be created, which contains binary metadata -->
        <kotlin2js src="root1" output="out.js" metaInfo="true"/>
    </target>
</project>
```

## 레퍼런스

요소와 속성의 전체 목록은 아래와 같다:

### kotlinc와 kotlin2js를 위한 공통 속성

| 이름 | 설명 | 필수 | 기본 값 |
|------|-------------|----------|---------------|
| `src`  | 컴파일 할 코틀린 소스 파일이나 폴더 | Yes |  |
| `nowarn` | 모든 컴파일 경고를 감춤 | No | false |
| `noStdlib` | 클래스패스에 코틀린 표준 라이브러리를 포함하지 않음 | No | false |
| `failOnError` | 컴파일하는 동안 에러를 발견하면 빌드를 실패함 | No | true |

### kotlinc 속성

| 이름 | 설명 | 필수 | 기본 값 |
|------|-------------|----------|---------------|
| `output`  | 출력 위치 디렉토리 또는 .jar 파일 이름 | Yes |  |
| `classpath`  | 컴파일 클래스패스 | No |  |
| `classpathref`  | 컴파일 클래스패스 레퍼런스 | No |  |
| `includeRuntime`  | `output`이 .jar 파일이면 jar 파일에 코틀린 런타임 라이브러리를 포함 | No | true  |
| `moduleName` | 컴파일 할 모듈의 이름 | No | 프로젝트 또는 (지정한 경우) 대상의 이름 |


### kotlin2js 속성

| 이름 | 설명 | 필수 |
|------|-------------|----------|
| `output`  | 출력 파일 | Yes |
| `libraries`  | 코틀린 라이브러리 경로 | No |
| `outputPrefix`  | 생성할 자바스크립트 파일에 사용할 접두사 | No |
| `outputSuffix` | 생성할 자바스크립트 파일에 사용할 접미사 | No |
| `sourcemap`  | 소스맵 파일을 생성할 위치 | No |
| `metaInfo`  | 바이너리 디스크립터를 가진 메타데이터 파일을 생성할 위치 | No |
| `main`  | 컴파일러가 생성한 코드가 메인 함수를 호출할지 여부 | No |
