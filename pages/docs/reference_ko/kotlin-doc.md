---
type: doc
layout: reference_ko
category: "Tools"
title: "코틀린 코드 문서화"
---

# 코틀린 코드 문서화

코틀린 코드를 문서화하는데 사용하는 언어를(자바의 JavaDoc과 같은 언어) **KDoc**이라 부른다.
KDoc은 블록 태그를 위한 JavaDoc의 구문(코틀린에 특정한 구성 요소를 지원하기 위해 확장 추가)과 
인라인 마크업을 위한 마크다운을 합쳤다.

## 문서 생성

코틀린 문서 생성 도구는 [Dokka](https://github.com/Kotlin/dokka)이다. 명령어 사용법은
[Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md)를 참고한다.

Dokka는 그레이들, 메이븐, 앤트 플러그인을 지원하므로 빌드 과정에 문서 생성을 통합할 수 있다.

## KDoc 구문

JavaDoc처럼 KDoc 주석은 `/**`로 시작해서 `*/`로 끝난다. 주석의 각 줄은 별표로 시작할 수 있으며 별표는 주석 내용에 포함되지 않는다.

관례상, 문서 텍스트의 첫 번째 단락은 요소의 요약 내용이 오고 다음 문장부터 상세 설명이 온다.

모든 블록 태그는 새 줄에 작성하고 `@` 글자로 시작한다.

다음은 KDoc을 이용해서 클래스를 문서화한 예이다:

``` kotlin
/**
 * *members*의 그룹
 *
 * 이 클래스는 유용한 로직이 없다. 단지 문서화 예제일 뿐이다.
 *
 * @param T 이 그룹의 멤버 타입
 * @property name 이 그룹의 이름
 * @constructor 빈 그룹을 생성한다.
 */
class Group<T>(val name: String) {
    /**
     * 이 그룹에 [member]를 추가한다.
     * @return 그룹의 새 크기
     */
    fun add(member: T): Int { ... }
}
```

## 블록 태그

현재 KDoc은 다음 블록 태그를 지원한다:

#### `@param <name>`

클래스, 프로퍼티, 함수의 타입 파라미터나 함수의 값 파라미터를 문서화한다.
설명에서 파라미터 이름을 더 잘 구분하는 것을 선호하면, 대괄호로 파라미터 이름을 감싸면 된다.
다음 두 구문은 동일하다:

```
@param name description.
@param[name] description.
```

#### `@return`

함수의 리턴 값을 문서화한다.

#### `@constructor`

클래스의 주요 생성자를 문서화한다.

#### `@receiver`

확장 함수의 리시버를 문서화한다.

#### `@property <name>`

특정 이름을 가진 클래스의 프로퍼티를 문서화한다. 주요 생성자에 선언한 프로퍼티를 문서화할 때 이 태그를 사용할 수 있다.
프로퍼티 선언 앞에 바로 문서 주석을 넣는 것이 어색할 수 있다.

#### `@throws <class>`, `@exception <class>`

메서드가 발생할 수 있는 익셉션을 문서화한다. 코틀린에는 checked 익셉션이 없으므로 모든 발생 가능한 익셉션을 문서화한다고 
기대하지 않는다. 하지만, 클래스 사용자에게 유용한 정보를 제공하고 싶을 때 이 태그를 사용할 수 있다.

#### `@sample <identifier>`

지정한 이름의 한정자를 가진 함수의 몸체를 현재 요소를 위한 문서화에 삽입한다. 요소의 사용 방법 예를 보여주기 위한 용도로
사용할 수 있다.

#### `@see <identifier>`

문서의 **See Also** 블록에 지정한 클래스나 메서드에 대한 링크를 추가한다.

#### `@author`

문서화할 요소의 작성자를 지정한다.

#### `@since`

문서화한 요소를 추가한 소프트웨어의 버전을 지정한다.

#### `@suppress`

생성할 문서에서 요소를 제외한다. 모듈의 공식 API에 속하지 않지만 외부에서 여전히 접근할 수 있는 요소에 사용할 수 있다.

> KDoc은 `@deprecated` 태그를 지원하지 않으며, 대신 `@Deprecated` 애노테이션을 사용한다.
{:.note}


## 인라인 마크업

KDoc은 인라인 마크업을 위해 일반 [마크다운](http://daringfireball.net/projects/markdown/syntax) 구문을 사용한다.
코드에서 다른 요로의 링크를 위한 간결한 구문을 추가로 확장했다.

### 요소에 대한 링크

다른 요소(클래스, 메서드, 파라미터의 프로퍼티)를 링크하려면, 단순히 대괄호에 이름을 넣으면 된다: 

```
Use the method [foo] for this purpose.
```

링크에 커스텀 라벨을 지정하고 싶으면 마크다운의 참조-방식 구문을 사용한다:

```
Use [this method][foo] for this purpose.
```

링크에 이름을 한정해서 사용할 수 있다. JavaDoc과 달리 한정한 이름은 메서드 이름 시작 전에 컴포넌트를 구분하기 위해  
항상 점(.) 글자를 사용한다. 

```
Use [kotlin.reflect.KClass.properties] to enumerate the properties of the class.
```

문서화할 요소 안에서 사용할 이름과 같은 규칙을 사용해서 링크의 이름을 찾아낸다.
특히, 이는 현재 파일에 임포트한 이름을 가진 경우 KDoc 주석에서 그 이름을 사용할 때 완전한 이름을 사용할 필요가 없다는 것을 의미한다.

KDoc에는 오버로딩한 이름을 알아내기 위한 구문이 없다. 코틀린 문서 생성 도구는 같은 페이지의 모든 오버로딩된 함수를 위한 문서화를 추가하므로,
링크에 특정한 오버로딩된 함수를 지정하지 않아도 링크가 동작한다.


## 모듈과 패키지 문서화

모듈 전체와 그 모듈의 모든 패키지에 대한 문서화는 별도 마크다운 파일로 제공한다.
그 파일 경로는 `-include` 명령행 파라미터를 사용해서 Dokka에 전달하고, 앤트, 메이븐, 그레이들 플러그인에서 해당하는 파라미터로 전달한다.

그 파일 안에는 전체 모듈과 개별 패키지에 대한 문서화를 대응하는 첫 번째 레벨 헤딩으로 추가한다.
모듈을 위한 헤딩 텍스트는 "Module `<모듈명>`" 형식이어야 하고, 패키지의 경우는 "Package `<완전한 패키지 이름>`"여야 한다.

다음은 예제이다.

```
# Module kotlin-demo

The module shows the Dokka syntax usage.

# Package org.jetbrains.kotlin.demo

Contains assorted useful stuff.

## Level 2 heading

Text after this heading is also part of documentation for `org.jetbrains.kotlin.demo`

# Package org.jetbrains.kotlin.demo2

Useful stuff in another package.
```

