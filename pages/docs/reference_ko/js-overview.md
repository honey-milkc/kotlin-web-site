---
type: doc
layout: reference_ko
category: "Introduction"
title: "자바스크립트를 위한 코틀린"
---

# 코틀린 자바스크립트 개요

코틀린은 자바스크립트를 대상(target)으로 할 수 있는 기능을 제공한다. 코틀린을 자바스크립트로 트랜스파일링해서 이를 제공한다.
현재 구현은 ECMAScript 5.1에 맞춰져 있는데 향후 ECMAScript 2015도 지원할 계획이다.

자바스크립트를 대상으로 선택하면 프로젝트 코드와 코틀린 표준 라이브러리의 모든 코드를 자바스크립트로 트랜스파일한다.
하지만 여기에는 사용한 JDK와 JVM 또는 자바 프레임워크나 라이브러리는 포함하지 않는다.
코틀린이 아닌 모든 파일은 컴파일 과정에서 생략한다.

코틀린 컴파일러는 다음 목표를 지키기 위해 노력한다.

* 크기를 최적화한 결과를 제공한다.
* 가독성있는 자바스크립트 결과를 제공한다.
* 기존의 모듈 시스템과의 상호운용성을 제공한다.
* 자바스크립트나 JVM에 상관없이 표준 라이브러리에 (최대한 가능한 수준에서) 동일 기능을 제공한다.

## 어떻게 사용할 수 있나

다음 시나리오에서 코틀린을 자바스크립트로 컴파일할 수 있다:

* 클라이언트 사이드 자바스크립트를 대상으로 코틀린 코드 생성

    * **DOM 요소 다루기**. 코틀린은 DOM을 다룰 수 있는 정적 타입 인터페이스를 제공하며, 이를 이용해 DOM 요소를 생성하고 수정할 수 있다. 

    * **WebGL과 같은 그래픽 다루기**. 코틀린으로 WebGL을 이용한 웹 페이지 그래픽 요소를 생성할 수 있다.  

* 서버 사이드 자바스크립트를 대상으로 코틀린 코드 생성

    * **서버 사이드 기술로 실행하기**. Mode.js와 같은 서버 사이드 자바스크립트를 다루는데 코틀린을 사용할 수 있다. 

jQuery나 React와 같은 기존의 서드파티 라이브러리나 프레임워크와 함께 코틀린을 사용할 수 있다.
강한 타입형 API에서 서드파티 프레임워크에 접근하려면, 
[Definitely Typed](http://definitelytyped.org/) 타입 정의 리포지토리에서 구한 타입스크립트 정의를 
[ts2kt](https://github.com/kotlin/ts2kt) 도구를 사용해서 코틀린으로 전환할 수 있다.
또는 강한 타입없이 [동적 타입](dynamic-type.html)을 사용해서 프레임워크에 접근할 수 있다.

젯브레인즈는 React 커뮤니티를 위한 몇 가지 도구를 개발하고 유지보수하고 있다. [React bindings](https://github.com/JetBrains/kotlin-wrappers)과 [코틀린 앱 만들기](https://github.com/JetBrains/create-react-kotlin-app)를 참고한다. 
후자는 빌드 설정없이 코틀린을 갖고 React 앱 제작을 시작하는데 도움이 된다.

코틀린은 CommonJS, AMD and UMD 모듈 시스템과 호환되며, [다른 모듈 시스템과 상호운용](http://kotlinlang.org/docs/tutorials/javascript/working-with-modules/working-with-modules.html)할 수 있다 


## 자바스크립트에 대해 코틀린으로 시작하기

자바스크립트를 위한 코틀린을 시작하는 방법은 [튜토리얼](http://kotlinlang.org/docs/tutorials/javascript/kotlin-to-javascript/kotlin-to-javascript.html)에서 확인할 수 있다.
