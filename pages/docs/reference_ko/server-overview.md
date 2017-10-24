---
type: doc
layout: reference
category: "Introduction"
title: "서버 사이드 코틀린"
---

# 서버 개발에서 코틀린 사용하기

코틀린은 간결하고 표현력있는 코드를 작성할 수 있게 해주면서 기존의 자바 기반 기술 스택과 완벽한 호환성을 유지하며
학습이 어렵지 않기에 서버 어플리케이션 개발에 매우 적합하다:

 * **표현력**: 코틀린은 [type-safe builders](/docs/reference/type-safe-builders.html)나 [delegated properties](/docs/reference/delegated-properties.html)와 같은
   혁신적인 언어 특징을 제공하며, 이를 통해 강력하하고 사용하기 쉬운 추상화를 만들 수 있다.  
 * **확장성**: 코틀린이 지원하는 [coroutines](/docs/reference/coroutines.html)은 적정 수준의 하드웨어 수준에서
   대량의 클라이언트를 처리할 수 있는 확장성있는 서버 어플리케이션을 구축할 수 있게 해준다. 
 * **상호운용성**: 코틀린은 모든 자바 기반 프레임워크가 완전히 호환되므로 익숙한 기술 스택을 유지하면서 최신 언어의 장점을 누릴 수 있다.
 * **마이그레이션**: 코틀린은 코드베이스를 자바에서 코틀린으로의 단계적이고 점진적인 이전을 지원한다. 자바로 작성된 시스템을 유지하면서
   신규 코드를 코틀린으로 작성할 수 있다.
 * **도구**: IDE에 지원이 대체로 훌륭할뿐만 아니라, Intellij IDEA Ultimate 플러그인으로 프레임워크에 특화된 도구(예, 스프링을 위한 도구)를 제공한다. 
 * **학습 곡선**: 자바 개발자는 쉽게 코틀린을 시작할 수 있다. 코틀린 플러그인에 포함된 자바-코틀린 자동 변환기가 시작을 돕는다.
   [Kotlin Koans](/docs/tutorials/koans.html)은 대화식으로 구성된 연습을 통해 언어의 핵심 특징을 익힐 수 있는 가이드를 제공한다.
 
## 코틀린을 지원하는 서버 개발 프레임워크

 * [Spring](https://spring.io)은 5.0 버전부터 [더 간결한 APIs](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0)를
   제공하기 위해 코틀린 언어 특징을 사용한다. [온라인 프로젝트 생성기](https://start.spring.io/#!language=kotlin)를 사용하면 신규 코틀린 프로젝트를 빠르게 생성할 수 있다. 

 * [Vert.x](http://vertx.io)는 JVM에서 동작하는 리액티브 웹 어플리케이션을 만들기 위한 프레임워크로 
   [완전한 문서화](http://vertx.io/docs/vertx-core/kotlin/)를 포함한 [크틀린 전용](https://github.com/vert-x3/vertx-lang-kotlin) 모듈을 제공한다.
 
 * [Ktor](https://github.com/kotlin/ktor)은 JetBrains가 만든 코틀린-네이티브 웹 프레임워크이다. 이 프레임워크는
    높은 확장성과 사용하기 쉽고 관용적인 API를 제공하기 위해 코루틴을 사용한다.

 * [kotlinx.html](https://github.com/kotlin/kotlinx.html)는 웹 어플리케이션에서 HTML을 만들 때 사용할 수 있는 DSL이다
   JSP나 FreeMarker와 같은 전통적인 템플릿 시스템에 대한 대안으로 사용한다.

 * 직접적인 JDBC 사용, JPA, 자바 드라이버를 통한 NoSQL 데이터베이스 사용 등 영속성 기술로 사용할 수 있다
   JPA의 경우 [kotlin-jpa 컴파일러 플러그인](/docs/reference/compiler-plugins.html#kotlin-jpa-compiler-plugin)을 사용하면
   코틀린으로 컴파일한 클래스를 프레임워크의 요구에 맞출 수 있다. 

## 코틀린 서버 사이드 어플리케이션 배포

아마존 웹 서비스 구글 클라우언트 플랫폼 등 자바 웹 어플리케이션을 지원하는 모든 호스트에 코틀린 어플리케이션을 배포할 수 있다

[Heroku](https://www.heroku.com)에 코틀린 애플리케이션을 배포하려면
[공식 Heroku 튜토리얼](https://devcenter.heroku.com/articles/getting-started-with-kotlin)을 따라하면 된다.

AWS Labs는 [AWS Lambda](https://aws.amazon.com/lambda/) 함수를 작성할 때 코틀린을 사용하는   
[예제 프로젝트](https://github.com/awslabs/serverless-photo-recognition)를 제공한다.

## 서버 사이드에서 코틀린 사용자

[Corda](https://www.corda.net/2017/01/10/kotlin/)는 주요 은행이 지원하는 오픈소스 분산 원장(ledger) 플랫픔으로 전체를 코틀린으로 만들었다. 

[JetBrains Account](https://account.jetbrains.com/)는 JetBrains에서 전체 라이스선 판매와 검증을 담당하는 시스템으로
100% 코틀린으로 작성됐고 2015년부터 큰 문제 없이 운영에서 돌고 있다


## 다음 단계

* [HTTP 서블릿으로 웹 어플리케이션 만들기](/docs/tutorials/httpservlets.html)와
  [스프링 부트로 RESTful 웹 서비스 만들기](/docs/tutorials/spring-boot-restful.html) 튜토리얼을 보면 코틀린으로 
  매우 작은 웹 어플리케이션을 만드는 방법을 볼 수 있다  

* 언어에 대한 더 깊은 소개는 이 사이트의 [레퍼런스 문서](/docs/reference/index.html)와 [Kotlin Koans](/docs/tutorials/koans.html)에서 확인할 수 있다.
