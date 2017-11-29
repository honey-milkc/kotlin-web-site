---
type: doc
layout: reference
category: "Introduction"
title: "코틀린/네이티브"
---

# 코틀린/네이티브

[코틀린/네이티브](https://github.com/JetBrains/kotlin-native/)는 VM 없이 실행할 수 있도록 코틀린을 네이티브 바이너리로 컴파일하는 기술이다.
이 기술은 코틀린을 위한 LLVM-기반 백엔드와 코틀린 런타임 라이브러리의 네이티브 구현을 포함한다.
코틀린/네이티브는 주로 가상 머신이 맞지 않거나 가능하지 않은 플랫폼(iOS, 임베디드 대상 등) 또는
또한 개발자가 추가적인 런타임이 필요없는 합리적인 크기의 자체 동작하는 프로그랢을 만들고 싶은 플랫폼에 대해 컴파일이 가능하도록 설계됐다.

코틀린/네이티브는 네이티브 코드와의 완전한 상호운영을 지원한다. 플랫폼 라이브러리를 위해, 대응하는 interop 라이브러리를
기본 제공한다. 다른 라이브러리를 위해 C 헤더 파일로부터
[interop 라이브러리를 생성하는 도구](https://github.com/JetBrains/kotlin-native/blob/master/INTEROP.md)를 제공하며,
이 도구는 모든 C 언어 특징을 완전히 지원한다.
맥OS나 iOS의 경우 Objective/C 코드와의 상호운영 또한 지원한다.

코틀린/네이티브는 현재 개발중이다. 프리뷰 릴리즈를 시험해 볼 수 있다.
[CLion](https://www.jetbrains.com/clion/) 플러그인을 사용해서 IDE에서 코틀린/네이티브를 시도할 수 있다.

### 대상 플랫폼

코틀린/네이티브는 현재 다음 플랫폼을 지원한다:

   * Windows (현재는 x86_64만 지원)
   * Linux (x86_64, arm32, MIPS, MIPS little endian)
   * MacOS (x86_64)
   * iOS (arm64만 지원)
   * Android (arm32와 arm64)
   * WebAssembly (wasm32만 지원)

### 예제 프로젝트

코틀린/네이티브로 뭘 할 수 있는 지 보여주기 위해 몇 개 예제 프로젝트를 만들었다:

 * [코틀린/네이티브 깃헙 리포지토리](https://github.com/JetBrains/kotlin-native/tree/master/samples)에는 여러 예제 프로젝트가 있다.
 * The [KotlinConf Spinner 앱](https://github.com/jetbrains/kotlinconf-spinner)은 완전히 코틀린/네이티브로 작성한 간단한 크로스-플랫폼 모바일 멀티플레이어 게임으로,
   다음 컴포넌트로 구성되어 있다:
     - 데이터 저장소로 SQLite를 사용하고 REST/JSON API를 제공하는 백엔드
     - OpenGL을 사용하고 iOS와 안드로이드를 위한 모바일 클라이언트
     - 게임 점수를 보여주기 위한 WebAssembly 기반 브라우저 프론트엔드
 * [KotlinConf 앱](https://github.com/JetBrains/kotlinconf-app/tree/master/ios)은 UIKit 기반 UI를 사용한 iOS 앱으로 코틀린/네이티브의
   Objective/C interop를 보여준다.

       


