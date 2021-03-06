---
type: doc
layout: reference_ko
category: "Introduction"
title: "안드로이드를 위한 코틀린"
---

# 안드로이드를 위한 코틀린

코틀린은 안드로이드 어플리케이션 개발에 안성맞춤이다. 현대 언어의 장점을 새로운 제약없이 안드로이드 플랫폼에 도입할 수 있다.

  * **호환성**: 코틀린은 JDK 6과 완전히 호환되므로, 오래된 버전의 안드로이드 장비에서 문제없이 코틀린 어플리케이션을 실행할 수 있다.
    안드로이드 스튜디오는 코틀린 도구를 완벽히 지원하며 안드로이드 빌드 시스템과 호환된다.
  * **성능**: 코틀린 어플리케이션은 동일한 자바 어플리케이션만큼 빠른데, 이는 매우 유사한 바이트코드 구조 덕분이다.
    코틀린의 인라인 함수 지원으로 종종 람다를 사용한 코드가 자바로 작성한 동일 코드보다 더 빠를 때도 있다.
  * **상호운용성**: 코틀린은 자바와 100% 상호운용할 수 있으며 존재하는 모든 안드로이드 라이브러리를 코틀린 어플리케이션에서 사용할 수 있다.
    여기에는 애노테이션 처리도 포함되므로 Databinding과 Dagger 역시 동작한다.
  * **풋프린트**: 코틀린 라이브러리는 매우 작고, ProGuard를 사용해서 더 줄일 수 있다.
   [실제 어플리케이션](https://blog.gouline.net/kotlin-production-tales-62b56057dc8a)에서 코틀린 런타임은
   수백 개의 메서드만 추가하는데 이는 .apk 파일 크기를 100K 미만으로 증가시킨다.
  * **컴파일 시간**: 코틀린은 효율적인 증분 컴파일을 지원하며, 클린 빌드에 약간의 추가 오버헤드만 발생한다.
    [증분 빌드는 보통 자바만큼 빠르거나 더 빠르다](https://medium.com/keepsafe-engineering/kotlin-vs-java-compilation-speed-e6c174b39b5d).
  * **학습 곡선**: 자바 개발자는 쉽게 코틀린을 시작할 수 있다. 코틀린 플러그인에 포함된 자바-코틀린 자동 변환기로 시작할 때 도움을 받을 수 있다.
   [코틀린 콘](http://kotlinlang.org/docs/tutorials/koans.html)은 대화식으로 구성된 연습을 통해 언어의 핵심 특징을 익힐 수 있는 가이드를 제공한다.

## 안드로이드 케이스 스터디를 위한 코틀린

주요 회사에서 성공적으로 코틀린을 도입했다. 다음은 그 회사 중 일부의 경험을 공유한 것이다:

  * Pinterest는 매달 1억 5천만명 사용자가 사용하는 [어플리케이션에 성공적으로 코틀린을 적용했다](https://www.youtube.com/watch?v=mDpnc45WwlI).
  * Basecamp는 안드로이드 앱을 [100% 코틀린 코드](https://m.signalvnoise.com/how-we-made-basecamp-3s-android-app-100-kotlin-35e4e1c0ef12)로 만들었다.
    프로그래머의 만족도에서 큰 차이를 보였고 결과물의 품질과 속도에서 높은 향상이 있다고 보고했다.
  * Keepsafe의 App Lock 앱 또한 [100% 코틀린으로 전환했으며](https://medium.com/keepsafe-engineering/lessons-from-converting-an-app-to-100-kotlin-68984a05dcb6)
    소스 코드 줄 수를 30% 줄였고 메서드 개수를 10% 줄였다.

## 안드로이드 개발을 위한 도구

코틀린 팀은 표준 언어 특징 이상을 지원하는 안드로이드 개발 도구를 제공한다.

 * [코틀린 안드로이드 확장](http://kotlinlang.org/docs/tutorials/android-plugin.html)은 컴파일러 확장 도구로서
   코드에서 `findViewById()` 호출을 제거하고 이를 컴파일러가 생성한 합성 프로퍼티로 대체할 수 있게 한다.
 * [Anko](http://github.com/kotlin/anko)는 안드로이드 API의 래퍼(wrapper)로 코틀린 친화적인 API를 제공한다.
   또한 레이아웃 .xml 파일을 코틀린 코드로 대체할 수 있는 DSL도 제공한다.  

## 다음 단계

* [안드로이드 스튜디오 3.0](https://developer.android.com/studio/index.html)를 다운받아 설치한다. 이 버전은 코틀린을 기본으로 지원한다.
* [안드로이드와 코틀린 시작하기](http://kotlinlang.org/docs/tutorials/kotlin-android.html) 튜토리얼을 보고 첫 번째 코틀린 애플리케이션을 만들어본다.
* 언어에 대한 더 자세한 소개는 이 사이트의 [레퍼런스 문서](index.html)와 [코틀린 콘](http://kotlinlang.org/docs/tutorials/koans.html)에서 확인할 수 있다.
* [Kotlin for Android Developers](https://leanpub.com/kotlin-for-android-developers)은 또 다른 좋은 자료이다.
 이 책은 코틀린을 이용한 실제 안드로이드 어플리케이션 제작 과정을 단계적으로 안내한다.
 * 구글의 [코틀린으로 작성한 예제 프로젝트](https://developer.android.com/samples/index.html?language=kotlin)를 참고한다.
