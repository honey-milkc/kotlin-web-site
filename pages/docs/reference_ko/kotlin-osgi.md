---
type: doc
layout: reference
category: "Tools"
title: "코틀린과 OSGi"
---

# 코틀린과 OSGi

코틀린 OSGi 지원을 활성화하려면 일반 코틀린 라이브러리 대신 `kotlin-osgi-bundle`를 포함해야 한다.
`kotlin-osgi-bundle`이 `kotlin-runtime`, `kotlin-stdlib`, `kotlin-reflect`를 포함하므로
이 세 의존을 제거할 것을 권한다.
또한, 외부 코틀린 라이브러리를 포함할 경우 주의해야 한다.
대부분 일반적인 코틀린 의존은 OSGi를 지원하지 않으므로
그것을 사용하지 말고 프로젝트에서 제거해야 한다.

## 메이븐

메이븐 프로젝트에 코틀린 OSGi 번들을 포함하려면 다음 설정을 사용한다:

```xml
   <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-osgi-bundle</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
    </dependencies>
```

외부 라이브러리에서 표준 라이브러리를 제외하려면 다음 설정을 사용한다("*" 표시 제외는 메이븐 3에서만 동작한다):

```xml
        <dependency>
            <groupId>some.group.id</groupId>
            <artifactId>some.library</artifactId>
            <version>some.library.version</version>

            <exclusions>
                <exclusion>
                    <groupId>org.jetbrains.kotlin</groupId>
                    <artifactId>*</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

## 그레이들

다음 설정으로 그레이들 프로젝트에 `kotlin-osgi-bundle`를 추가한다:

```groovy
compile "org.jetbrains.kotlin:kotlin-osgi-bundle:$kotlinVersion"
```

의존 전이로 가져오는 기본 코틀린 라이브러리를 제외하려면 다음 접근을 사용한다:

```groovy
dependencies {
 compile (
   [group: 'some.group.id', name: 'some.library', version: 'someversion'],
   .....) {
  exclude group: 'org.jetbrains.kotlin'
}
```

## FAQ

#### 왜 필요한 메니페스트 옵션을 모든 코틀린 라이브러리에 추가하지 않았나

비록 OSGi 지원을 제공하는 가장 선호하는 방법임에도 불구하고 
["패키지 분리" 이슈](http://wiki.osgi.org/wiki/Split_Packages) 때문에 지금은 할 수 없다.
이 문제는 쉽게 없앨 수 없고 그런 큰 변화는 현재 계획에 없다.
`Require-Bundle` 특징이 있지만 최선의 옵션이 아니며 사용을 권하지 않는다
그래서 OSGi를 위한 아티팩트를 구분하기로 결정했다.
