---
type: doc
layout: reference
category: FAQ
title: "자바와 비교"
---

# 자바 프로그래밍 언어와 비교

## 코틀린에서 해결한 몇 가지 자바 이슈

코틀린은 자바에서 겪는 몇 가지 이슈를 고쳤다:

* 널 레퍼런스는 [타입 시스템으로 제어](null-safety.html)함
* [raw 타입 없음](java-interop.html)
* 코틀린 배열은 [공변](basic-types.html#arrays)임
* 자바의 SAM 변환과 달리 코틀린에는 제대로 된 [함수 타입](lambdas.html#function-types)이 있음
* 와일드카드 없는 [사용 위치 변성](generics.html#use-site-variance-type-projections)
* 코틀린에는 checked [익셉션](exceptions.html)이 없음

## 자바에는 있는데 코틀린에는 없는 것

* [Checked 익셉션](exceptions.html)
* 클래스가 아닌 [기본 타입](basic-types.html)
* [정적 멤버](classes.html)
* [private 아닌 필드](properties.html)
* [와일드카드 타입](generics.html)

## 자바에는 없는데 코틀린에 있는 것

* [람다 식](lambdas.html) + [인라인 함수](inline-functions.html) = 더 효율적인 커스텀 제어 구조
* [확장 함수](extensions.html)
* [Null-안정성](null-safety.html)
* [스마트 변환](typecasts.html)
* [문자열 템플릿](basic-types.html#strings)
* [프로퍼티](properties.html)
* [주요 생성자](classes.html)
* [1급 위임](delegation.html)
* [변수와 프로퍼티 타입에 대한 타입 추론](basic-types.html)
* [싱글톤](object-declarations.html)
* [선언 위치 가변성과 타입 프로젝션](generics.html)
* [범위 식](ranges.html)
* [연산자 오버로딩](operator-overloading.html)
* [컴페니언 오브젝트](classes.html#companion-objects)
* [데이터 클래스](data-classes.html)
* [읽기 전용과 변경 가능 콜렉션의 인터페이스 분리](collections.html)
* [코루](coroutines.html)
