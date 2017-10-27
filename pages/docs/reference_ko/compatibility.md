---
type: doc
layout: reference
category: "Compatibility"
title: "호환성"
---

# 호환성

이 문서는 코틀린의 하위시스템과 다른 버전 간에 보장하는 호환성을 설명한다. 

## 호환성 용어

주어진 두 코틀린 버전(예, 1.2와 1.1.5)에 대해 한 버전으로 작성한 코드를 다른 버전에서 사용할 수 있나? 호환성은 이 질문에 답하는 것을 의미한다. 
아래 목록은 다른 버전 간의 호환성 모드를 설명한다. 버전 숫자가 작으면 (비록 큰 버전 숫자보다 작은 버전이 나중에 나왔어도) 오래된 버전이다.
"오래된 버전(Older Version)"에 OV를 사용하고 "새로운 버전(Newer Version)"에 대해 NV를 사용한다.

- **C** - 완전환 호환성(**C**ompatibility) 
  - 언어
    - 구문 변화 없음 (*모듈 버그*\*)
    - 신규 경고/힌트를 추가하거나 제거할 수 있음
  - API (`kotlin-stdlib-*`, `kotlin-reflect-*`)
    - API 변경 없음
    - `WARNING` 수준의 디프리케이션을 추가하거나 제거할 수 있음
  - 바이너리 (ABI)
    - 런타임: 바이너리를 교환해서 사용할 수 있음
    - 컴파일: 바이너리를 교환해서 사용할 수 있음
- **BCLA** - 언어 API에 대한 하위 호환(**B**ackward **C**ompatibility for the **L**anguage and **A**PI)
  - 언어
    - OV에서 디프리케이션한 구문을 NV에서 제거할 수 있음
    - 이 외에 OV에 호환되는 모든 코드는 NV에 호환(modulo bugs*)
    - NV에 새 구문을 추가할 수 있음
    - OV의 제약이 NV에서 완화될 수 있음 
    - 신규 경고/힌트를 추가하거나 제거할 수 있음
  - API (`kotlin-stdlib-*`, `kotlin-reflect-*`)
    - 새 API를 추가할 수 있음
    - `WARNING` 수준의 디프리케이션을 추가하거나 제거할 수 있음
    - `WARNING` 수준의 디프리케이션이 NV에서 `ERROR`나 `HIDDEN`으로 등급이 올라갈 수 있음 
- **BCB** - 바이너리에 대한 하위 호환(**B**ackward **C**ompatibility for **B**inaries)
  - 바이너리 (ABI)
    - 런타임: OV 바이너리가 동작하는 모든 곳에 NV-바이너리를 사용할 수 있음
    - NV 컴파일러: OV 바이너리에 대한 컴파일 가능한 코드는 NV 바이너리에 대해 호환
    - OV 컴파일러는 NV 바이너리를 수용하지 않을 수 있음 (예, 새로운 언어 특징이 API가 존재하는 경우)
- **BC** - 완전한 하위 호환(**B**ackward **C**ompatibility)
  - BC = BCLA & BCB
- **EXP** - 실험 특징
  - [아래](#experimental-features) 참고
- **NO** - 호환성 보장하지 않음
  - 마이그레이션이 잘 되도록 최선을 다하지만, 어떤 보장도 하지 않음
  - 모든 비호환 하위시스템에 대해 개별적으로 마이그레이션 계획 

---

\* 변화 없음 *modulo bugs*는 중요한 버그가 발견되면(예, 컴파일러 진단이나 다른 곳에서)
버그를 고쳤을 때 호환성을 깨는 변경이 발생할 수도 있으므로 그런 변경에 매우 주의한다는 것을 의미한다.   

## 코틀린 릴리즈 호환성

**JVM을 위한 코틀린**:
  - 패치 버전 업데이트 (예 1.1.X)는 완전히 호환
  - 마이너 버전 업데이트 (예 1.X)는 하위 호환

| Kotlin    | 1.0 | 1.0.X | 1.1 | 1.1.X | ... | 2.0 |
|----------:|:---:|:-----:|:---:|:-----:|:---:|:---:|
| **1.0**   | -   | C     | BC  | BC    | ... | ?   |
| **1.0.X** | C   | -     | BC  | BC    | ... | ?
| **1.1**   | BC  | BC    | -   | C     | ... | ?
| **1.1.X** | BC  | BC    | C   | -     | ... | ?
| **...**   | ... | ...   | ... | ...   | ... | ... |
| **2.0**   | ?   | ?     | ?   | ?     | ... | - 

**JS를 위한 코틀린**: 코틀린 1.1부터, 패치와 마이너 버전 업데이트에 대해 언어와 API(BCLA)에 대한 하위 호환성을 제공하지만,
BCB는 제공하지 않는다.

| Kotlin    | 1.0.X | 1.1  | 1.1.X | ... | 2.0 |
|----------:|:-----:|:----:|:-----:|:---:|:---:|
| **1.0.X** | -     |  EXP | EXP   | ... | EXP |
| **1.1**   | EXP   |  -   | BCLA  | ... | ?
| **1.1.X** | EXP   | BCLA | -     | ... | ?
| **...**   | ...   | ...  | ...   | ... | ... |
| **2.0**   | EXP   | ?    | ?     | ... | - 

**코틀린 스크립트**: 패치와 마이너 버전 업데이트에 대해 언어와 API(BCLA)에 대한 하위 호환성을 제공하지만,
BCB는 제공하지 않는다. 

## 플랫폼 간 호환성
 
여러 플랫폼(JVM/안드로이드, 자바스크립트와 앞으로 지원할 네이티브 플랫폼)에서 코틀린을 사용할 수 있다. 모든 플랫폼은
자신만의 특성을 갖고 있어서(예, 자바스크립트는 정수가 없다), 언어에 알맞게 맞춰야 한다.
우리의 목적은 많은 희생없이 합리적인 코드 이식성을 제공하는 것이다.

모든 플랫폼은 특정한 언어 확장(JVM의 플랫폼 타입과 자바스크립트의 동적 타입)이나 
제약(예, JVM에서의 일부 오버로딩 관련 제약)을 갖지만, 핵심 언어는 동일하게 유지한다.

표준 라이브러리는 모든 플랫폼에서 가능한 핵심 API를 제공하며, 이 API가 모든 플랫폼에서 동일하게 동작하도록 만들기 위해 노력했다.
이와 함께, 표준 라이브러리는 플랫폼에 특화된 확장(예, JVM을 위한 `java.io`와 자바스크립트를 위한 `js()`)과 
추가로 동일하게 호출할 수 있지만 다르게 동작하는 일부 API(예, JVM과 자바스키릅트의 정규 표현식)를 제공한다.

## 실험 특징

코틀린 1.1의 코루틴과 같은 실험 특징은 아래 표시한 호환성 모드에서 제외된다.
이런 특징을 컴파일러 경고없이 사용하려면 사전 동의(opt-in)를 요구한다.
실험 특징은 패치 버전 업데이트에 대해서는 하위 호환성을 유지하지만,
마이너 버전 업데이트에서는 어떤 호환성도 보장하지 않는다(가능하면 마이그레이션 도구를 제공할 것이다);    

| Kotlin    | 1.1 | 1.1.X | 1.2 | 1.2.X | 
|----------:|:---:|:-----:|:---:|:-----:|
| **1.1**   | -   | BC    | NO  | NO  
| **1.1.X** | BC  | -     | NO  | NO
| **1.2**   | NO  | NO    | -   | BC 
| **1.2.X** | NO  | NO    | BC  | - 

## EAP 빌드

EAP(Early Access Preview)를 특수 채널로 제공하는데 이 채널을 통해 커뮤니티의 얼리 어댑터가 시도해보고 피드백을 줄 수 있다. 
EAP 빌드는 (각 릴리즈마다 알맞게 호환성을 유지하게 위해 노력은 하겠지만) 어떤 호환성 보장도 하지 않는다. 
EAP 빌드에 대한 품질 기대 수준은 정식 릴리즈보다 매우 낮다. 베타 빌드도 이 부류에 속한다.

**중요 사항**: 컴파일러의 릴리즈 빌드는 1.x의 EAP 빌드(예, 1.1.0-eap-X)로 컴파일한 모든 바이너리를 거부한다.
안정된 버전을 릴리즈한 이후에 이전 버전으로 컴파일한 어떤 코드도 유지하길 원치 않기 때문이다. 
패치 버전의 EAP(예, 1.1.3-eap-X)는 이와 상관없으며 이 EAP는 안정한 ABI를 사용해서 빌드를 생성할 수 있다.

## 호환성 모드

큰 팀이 새 버전으로 마이그레이션을 할 때, 일부 개발자가 이미 업데이트했는데 다른 개발자는 하지 않아서,
어떤 시점에 "깨진 상태(inconsistent state)"가 될 수 있다.
다른 개발자가 컴파일할 수 없는 코드를 작성하고 커밋하는 것을 방지하기 위해,
다음 명령행 스위치를 제공한다(IDE와 [그레이들](https://kotlinlang.org/docs/reference/using-gradle.html#compiler-options)/[메이븐](https://kotlinlang.org/docs/reference/using-maven.html#specifying-compiler-options)에서도 사용 가능하다).

- `-language-version X.Y` - 코틀린 언어 X.Y 버전에 대한 위한 호환성 모드. 이후에 나온 모든 언어 특징에 대해 에러를 발생한다.
- `-api-version X.Y` - 코틀린 API X.Y 버전에 대한 호환성 모드. 코틀린 표준 라이브러리에서 신규 API를 사용하는 모든 코드에 대해(컴파일러가 생성한 코드 포함) 에러를 발생한다.

## 바이너리 호환성 경고

NV 코틀린 컴파일러를 사용하는데 클래스패스에 OV 표준 라이브러리나 OV 리플레션 라이브러리가 존재하면,
프로젝트를 잘못 설정했다는 신호일 수 있다.
컴파일이나 런타임에 원치 않는 문제가 발생하는 것을 막으려면,
의존을 NV로 업데이트하거나 API 버전/언어 버전 인자를 명시적으로 지정할 것을 권한다.
그렇지 않을 경우 컴파일러는 뭔가 잘못될 수 있다는 것을 탐지하고 경고를 발생한다.

예를 들어, OV = 1.0 이고 NV = 1.1 이면, 다음 경고 중 하나를 볼 수 있다:

```
Runtime JAR files in the classpath have the version 1.0, which is older than the API version 1.1. 
Consider using the runtime of version 1.1, or pass '-api-version 1.0' explicitly to restrict the 
available APIs to the runtime of version 1.0.
```

이는 1.0 버전의 표준 또는 리플렉션 라이브러리로 코틀린 1.1 컴파일러를 사용한다는 것을 의미한다. 이 경고는 몇 가지 방법으로 처리할 수 있다:   
* 1.1 표준 라이브러리의 API나 그 API에 의존한 언어 특징을 사용하길 원한다면, 의존 버전을 1.1로 올려야 한다.
* 1.0 표준 라이브러리와 호환되도록 코드를 유지하고 싶다면, `-api-version 1.0`를 전달한다.
* 코틀린은 1.1로 업그레이드는 하지만 새 언어 특징은 사용할 수 없다면(예, 일부 팀원이 아직 업그레이드 전인 경우),
  모든 API와 언어 특징을 1.0으로 제한하기 위해 `-language-version 1.0`를 전달할 수 있다. 

```
Runtime JAR files in the classpath should have the same version. These files were found in the classpath:
    kotlin-reflect.jar (version 1.0)
    kotlin-stdlib.jar (version 1.1)
Consider providing an explicit dependency on kotlin-reflect 1.1 to prevent strange errors
Some runtime JAR files in the classpath have an incompatible version. Consider removing them from the classpath
```

이 메시지는 다른 버전의 라이브러리에 대한 의존이 존재함을 의미한다. 예를 들어, 1.1 표준 라이브러리와 1.0 리플렉션 라이브러리에 의존이 존재하는 경우이다.
런타임에 발생하는 미묘한 에러를 막으려면, 모든 코틀린 라이브러리가 같은 버전을 사용할 것을 권한다.
이 예의 경우 1.1 리플렉션 라이브러리를 의존에 명시적으로 추가할 것을 고려한다.

```
Some JAR files in the classpath have the Kotlin Runtime library bundled into them. 
This may cause difficult to debug problems if there's a different version of the Kotlin Runtime library in the classpath. 
Consider removing these libraries from the classpath
```

이 메시지는 클래스패스에 코틀린 표준 라이브러리에 의존하지 않는 라이브러리가 그레이들/메이븐 의존으로 존재하는데,
그 라이브러리가 표준 라이브러리를 포함한다는 것을(예, _번들_한 경우) 의미한다.  
표준 빌드 도구는 이런 라이브러리를 코틀린 표준 라이브러리의 인스턴스로 고려하지 않고, 
따라서 의존 버전 분석 메커니즘에 적용을 받지 않으며, 
결과적으로 클래스패스에 동일 라이브러리의 여러 버전을 가질 수 있다. 그래서 이런 라이브러리는 문제를 발생할 수 있다.
라이브러리 제작자에 연락해서 그레이들/메이븐 의존을 대신 사용해달라고 제안하는 것을 고려한다.
