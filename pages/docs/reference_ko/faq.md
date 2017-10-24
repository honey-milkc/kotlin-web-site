---
type: doc
layout: reference
category: FAQ
title: FAQ
---

# FAQ

### 코틀린은 무엇인가?

코틀린은 OSS 정적 타입 프로그래밍 언어로서, JVM, 안드로이드 자바스크립트와 네이티비를 대상으로 한다.
코틀린은 [젯브레인Brains](http://www.jetbrains.com)이 개발한다.
2010년에 프로젝트를 시작했고 초기부터 오픈 소스였다.
첫 번째 공식 1.0 버전은 2016년 2월에 나왔다. 

### 현재 코틀린 버전은?

현재 릴리즈한 버전은 {{ data.releases.latest.version }}로 {{ data.releases.latest.date }}에 공개했다.

### 코틀린은 무료인가?

맞다. 코틀린은 무료이며, 무료였고, 앞으로도 무료일 것이다. 아파치 2.0 라이선스이며 [깃헙에서](https://github.com/jetbrains/kotlin) 소스 코드를 구할 수 있다.

### 코틀린은 객체 지향 언어인가? 함수형 언어인가?

코틀린은 객체 지향과 함수형의 구성 요소를 모두 갖고 있다. OO와 FP 방식 또는 두 방식을 섞어서 사용할 수 있다.
고차 함수, 함수 타입과 람다와 같은 특징을 1급으로 지원하므로 함수형 프로그래밍을 하거나 알아보고 싶다면
코틀린이 훌륭한 선택이다.

### 코틀린이 자바 프로그래밍 언어에 비해 주는 장점은 무엇인가?

코틀린은 더 간결하다. 대략적인 측정 결과 코드 줄 수에서 거의 40%를 줄여준다. 또한 타입에 더 안전하다. 예를 들어, non-nullable 타입은 어플리케이션에서
NPE를 줄여준다 스마트 변환, 고차 함수, 확장 함수, 리시버를 가진 람다를 포함한 다른 특징은
표현력 좋은 코드를 작성하는 능력을 제공하며 또한 DSL 작성을 촉진한다.
 
### 코틀린은 자바 프로그래밍 언어와 호환되나?

그렇다. 코틀린은 자바 프로그래밍 언어와 100% 호환되며, 기존 코드베이스를 코틀린에 알맞게 사용할 수 있도록 하는 것에 중점을 뒀다.
쉽게 자바에서 코틀린 코드를 호출할 수 있고 코틀린에서 자바 코드를 호출할 수 있다.
이는 매우 쉽고 저위험으로 코를린을 적용할 수 있게 해준다. 또한 자바에서 코틀린으로 자동으로 변환해주는 도구를 IDE에 탑재해서 기존 코드의 마이그레이션을 단순하게 할 수 있다.

### 코틀린을 어디에 쓸 수 있나?

서버사이드, 클라이언트 사이드 웹과 안드로이드 등 모든 종류의 개발에서 코틀린을 사용할 수 있고,
임베디드 시스템, 맥OS, iOS와 같은 다른 플랫폼을 지원하기 위한
코틀린/네이티브도 현재 작업중에 있다. 어디에 사용할 수 있는 몇 가지만 말해보면,
사람들은 모바일과 서버 사이드 어플리케이션, 자바 스크립트와 JavaFX를 위한 클라이언트 사이드, 데이터 과학에 코틀린을 사용하고 있다.

### 안드로이드 개발에 코틀린을 사용할 수 있나?

그렇다. 안드로이드는 1급 언어로 코틀린을 지원한다. 이미 Basecamp, Pinterest 등 수 백여개의 안드로이드 어플리케이션이 코틀린을 사용하고 있다.
[안드로이드 개발 관련 자료](/docs/reference/android-overview.html)에서 더 많은 정보를 확인할 수 있다.

### 서버 사이드 개발에 코틀린을 사용할 수 있다?

그렇다. 코틀린은 JVM과 100% 호환되며, 스프링 부트, vert.x 또는 JSF와 같은 기존 프레임워크를 사용할 수 있다.
추가로 [Ktor](http://github.com/kotlin/ktor)와 같은 코틀린으로 작성한 전용 프레임워크도 있다.
[서버 사이드 개발 관련 자료](/docs/reference/server-overview.html)에서 더 많은 정보를 확인할 수 있다.

### 웹 개발에 코틀린을 사용할 수 있나?

그렇다. 백엔드 웹에서 사용할 수 있을 뿐만 아니라, 클라이언트 사이드 웹에 코틀린/JS를 사용할 수 있다. 
많이 쓰이는 자바스크립트 라이브러리에 대한 정적 타입을 얻기 위해 코틀린은 [DefinitelyTyped](http://definitelytyped.org)의 정의를 사용할 수 있고,
AMD나 CommonJS 같은 기존의 모듈 시스템과 호환된다.
[클라이언트 사이드 개발 관련 자료](/docs/reference/js-overview.html)에서 더 많은 정보를 확인할 수 있다.

### 데스크톱 개발에 코틀린을 사용할 수 있나?

그렇다. JavaFx, 스윙과 같은 자바 UI 프레임워크를 사용할 수 있다.
추가로 [TornadoFX](https://github.com/edvin/tornadofx)와 같은 코틀린 전용 프레임워크가 존재한다. 

### 네이티브 개발에 코틀린을 사용할 수 있나?

현재 코틀린/네이티브를 [작업중](https://blog.jetbrains.com/kotlin/tag/native/)이다. 코틀린/네이티브는 코틀린을 JVM 없이 실행할 수 있는 네이티브 코드로 컴파일한다.
테크놀러지 프리뷰 릴리즈가 있는데 아직 제품으로 준비된 상태는 아니며, 1.0 버전에서는 모든 플랫폼을 지원할 예정은 아니다.
[코틀린/네이브티 소개 블로그 포스트](https://blog.jetbrains.com/kotlin/2017/04/kotlinnative-tech-preview-kotlin-without-a-vm/)에서
더 많은 정보를 확인할 수 있다.

### 코틀린을 지원하는 IDE는?

[인텔리J IDEA](/docs/tutorials/getting-started.html), [안드로이드 스튜디오](/docs/tutorials/kotlin-android.html),
[이클립스](/docs/tutorials/getting-started-eclipse.html), [넷빈즈](http://plugins.netbeans.org/plugin/68590/kotlin)를 포함한 주요 자바 IDE에서 코틀린을 지원한다.
또한 [명령행 컴파일러](/docs/tutorials/command-line.html)를 사용해서 컴파일하고 어플리케이션을 실행할 수 있다.
  
### 코틀린을 지원하는 빌드 도구는?

JVM 영역에서는 [그레이들](/docs/reference/using-gradle.html), [메이븐](/docs/reference/using-maven.html), 
[앤트](/docs/reference/using-ant.html), [코발트](http://beust.com/kobalt/home/index.html) 등 주요 빌드 도구가 지원한다.
클라이언트 사이드 자바스크립트 대상으로 사용가능한 빌드 도구도 있다. 

### 코틀린은 무엇으로 컴파일되나?

JVM을 대상으로 할 때, 코틀린은 자바와 호환되는 바이트코드를 생성한다. 자바스크립를 대상으로 할 때, 코틀린을 ES5.1로 트랜스파일하며 AMD와 CommonJS를 포함한 모들 시스템에
호환되는 코드를 생성한다. 네이티브를 대상으로 할 때, 코틀린은 (LLVM을 통해) 플랫폼 전용 코드를 생성할 것이다.

### 코틀린은 자바 6만 대상으로 할 수 있나?

아니다. 코틀린은 자바 6이나 자바 8에 호환되는 바이트코드 생성을 선택할 수 있다. 더 높은 버전의 플랫폼에 대해 더 최적화된 바이트 코드를 생성한다.

### 코틀린은 어렵나?

코틀린은 자바, C#, 자바스크립트, 스칼라, 그루비와 같은 기존 언어에서 영향을 받았다. 코틀린을 배우기 쉽게 하려고 노력했으며,
코틀린을 시작해서 몇 일이면 쉽게 코틀린을 읽고 작성할 수 있을 것이다.
코틀린의 특징을 배우고 몇 가지 고급 특징을 배우는데는 좀 더 걸리겠지만, 전반적으로 코틀린은 복잡한 언어가 아니다.
 
### 어떤 회사에서 코틀린을 사용하나?
 
코틀린을 회사는 너무 많아서 나열할 수 없다. 블로그나 깃헙 리포지토리, 토크로 코틀린 사용을 공개적으로 선언한 회사에는
[Square](https://medium.com/square-corner-blog/square-open-source-loves-kotlin-c57c21710a17),
[Pinterest](https://www.youtube.com/watch?v=mDpnc45WwlI),
[Basecamp](https://m.signalvnoise.com/how-we-made-basecamp-3s-android-app-100-kotlin-35e4e1c0ef12)가 있다.
 
### 누가 코틀린을 개발하나?

코틀린은 주로 젯브레인의 엔지니어 팀이 개발한다(현재 40명 이상 참여). 언어 설계를 이끄는 사람은 [Andrey Breslav](https://twitter.com/abreslav)이다.
핵심 팀 외에 깃헙에 100명 이상의 외부 공헌자가 존재한다. 

### 코틀린에 대해서 어디서 더 배울 수 있나?

시작하기 가장 좋은 출발점은 [코틀린 웹사이트](https://kotlinlang.org)이다. 거기서 컴파일러를 다운로드 받을 수 있고,
[온라인으로 시도](https://try.kotlinlang.org)할 수 있고, 또한 [레퍼런스 문서](/docs/reference/index.html)와
[튜토리얼](/docs/tutorials/index.html), 자료를 구할 수 있다.

### 코틀린에 대한 책이 있나?

코틀린 팀 멤버인 Dmitry Jemerov and Svetlana Isakova가 쓴 [Kotlin in Action](https://www.manning.com/books/kotlin-in-action),
안드로이드 개발자를 대상으로 한 [Kotlin for Android Developers](https://leanpub.com/kotlin-for-android-developers) 등
이미 코틀린에 대한 [책이 여러 개](/docs/books.html) 있다.  

### 코틀린을 배울 수 있는 온라인 강좌가 존재하나?

Kevin Jones의 [Pluralsight Kotlin Course](https://www.pluralsight.com/courses/kotlin-getting-started),
Hadi Hariri의 [O’Reilly Course](http://shop.oreilly.com/product/0636920052982.do),
Peter Sommerhoff의 [Udemy Kotlin Course](http://petersommerhoff.com/dev/kotlin/kotlin-beginner-tutorial/) 등
몇 개의 코틀린 강좌가 존재한다. 
또한 유튜브와 비메오에서 많은 [Kotlin talks](http://kotlinlang.org/community/talks.html)를 볼 수 있다. 

### 코틀린 커뮤니티가 있나?

있다. 코틀린은 매우 활발한 커뮤니티를 갖고 있다. [코틀린 포럼](http://discuss.kotlinlang.org), [스택오버플로우](http://stackoverflow.com/questions/tagged/kotlin),
에서 코틀린 개발자를 만날 수 있고, [코틀린 슬랙](http://slack.kotlinlang.org)(2017년 5월 기준 거의 7000명의 멤)에서 더 활발하게 활동하고 있다.

### 코틀린 이벤트가 있나?

있다. 코틀린에만 초점을 둔 많은 사용자 그룹과 미트업이 있다. [웹 사이트의 목록](/community/user-groups.html)에서 찾을 수 있다.
또한 전세계적으로 커뮤니티가 주도하는 [코틀린 나이트](/community/kotlin-nights.html) 이벤트가 열린다.

### 코틀린 컨퍼런스가 있나?

있다. 첫 번째 공식 [KotlinConf](https://kotlinconf.com)이 2017년 11월 2-3일에 샌프란시스코에서 열린다.
또한 전세계적으로 다른 컨퍼런스에서 코틀린을 다루고 있는 중이다.
[웹 사이트에서 에정된 발표](/community/talks.html?time=upcoming) 목록을 확인할 수 있다.

### 소셜 미디어에 코틀린이 있나?

있다. 가장 활발히 활동하는 소셜미디어는 [트위터 계정](https://twitter.com/kotlin)이다.
[구글+ 그룹](https://plus.google.com/communities/104597899765146112928)도 있다. 

### 다른 온라인 코틀린 자료는?

커뮤니티가 멤버가 만든 [코틀린 다이제스트](https://kotlin.link),
[뉴스레터](http://www.kotlinweekly.net), [팟캐스트](https://talkingkotlin.com) 등의 사이트를
[온라인 자료](https://kotlinlang.org/community/)에서 확인할 수 있다.

### HD 코틀린 로고는 어디서 구할 수 있나?

[여기](https://resources.jetbrains.com/storage/products/kotlin/docs/kotlin_logos.zip)에서 로고를 다운로드할 수 있다.
아카이브에 포함된 `guidelines.pdf`의 간단한 규칙을 따라주길 바란다.
