---
type: doc
layout: reference_ko
category: "Tools"
title: "키워드와 연산자"
---

# 키워드와 연산자

## Hard 키워드

다음 토큰은 항상 키워드로 해석하며 식별자로 사용할 수 없다:

 * `as` 
      - [타입 변환](typecasts.html#unsafe-cast-operator)에 사용
      - [임포트 별칭](packages.html#imports) 지정
 * `as?`: [안전한 타입 변환](typecasts.html#safe-nullable-cast-operator)에 사용  
 * `break`: [루프 실행을 종료](returns.html)
 * `class`: [클래스](classes.html) 선언
 * `continue`: [가장 가까운 루프의 다음 단계를 진행](returns.html) 
 * `do`: [do/while 루프](control-flow.html#while-loops) 시작 (사후조건을 가진 루프)
 * `else`: [if 식](control-flow.html#if-expression)에서 조건이 false일 때 실행하는 브랜치 정의
 * `false`: [불리언 타입](basic-types.html#booleans)의 `false` 값
 * `for`: [for 루프](control-flow.html#for-loops)를 시작
 * `fun`: [함수](functions.html) 선언 
 * `if`: [if 식](control-flow.html#if-expression) 시작
 * `in`
     - [for 루프](control-flow.html#for-loops)에서 반복할 객체를 지정
     - 콜렉션이나 ['contains' 메서드를 정의한](operator-overloading.html#in) 다른 엔티티,
        [범위](ranges.html)에 값이 속하는지 검사하기 위한 중위 연산자로 사용 
     - [when 식](control-flow.html#when-expression)에서 같은 목적으로 사용
     - [반공변](generics.html#declaration-site-variance)으로 타입 파라미터 지정
 * `!in`
     - 콜렉션이나 ['contains' 메서드를 정의한](operator-overloading.html#in) 다른 엔티티,
        [범위](ranges.html)에 값이 속하지 않는지 검사하기 위한 중위 연산자로 사용 
     - [when 식](control-flow.html#when-expression)에서 같은 목적으로 사용
 * `interface`: [인터페이스](interfaces.html)
 * `is` 
     - [값이 특정 타입인지](typecasts.html#is-and-is-operators) 검사
     - [when 식](control-flow.html#when-expression)에서 같은 목적으로 사용
 * `!is`
     - [값이 특정 타입이 아닌지](typecasts.html#is-and-is-operators) 검사
     - [when 식](control-flow.html#when-expression)에서 같은 목적으로 사용
 * `null`: 어떤 객체도 가리키지 않는 객체 레퍼런스를 표현하는 상수
 * `object`: [클래스와 그것으 인스턴스를 동시에](object-declarations.html) 선언
 * `package`: [현재 파일을 위한 패키지](packages.html)를 지정
 * `return` [가장 가깝게 둘러싼 함수나 임의 함수에서 리턴](returns.html)  
 * `super` 
     - [메서드나 프로퍼티의 상위클래스 구현을 참조](classes.html#calling-the-superclass-implementation)
     - [보조 생성자에서 상위클래스의 생성자를 호출](classes.html#inheritance)
 * `this` 
     - [현재 리시버](this-expressions.html)를 참조
     - [보조 생성자에서 같은 클래스의 다른 생성자를 호출](classes.html#constructors)
 * `throw`: [익셉션을 발생](exceptions.html)
 * `true`: [불리언 타입](basic-types.html#booleans)의 'true' 값을 지정
 * `try`: [익셉션 처리 블록을 시작](exceptions.html)
 * `typealias`: [타입 별칭](type-aliases.html)을 선언
 * `val`: 읽기 전용 [프로퍼티](properties.html)나 [로컬 변수](basic-syntax.html#defining-variables)를 선언
 * `var` 수정 가능 [프로퍼티](properties.html)나 [로컬 변수](basic-syntax.html#defining-variables)를 선언
 * `when`: [when 식](control-flow.html#when-expression) 시작(주어진 브랜치 중 하나 실행)
 * `while`: [while 루프](control-flow.html#while-loops) (전위조건을 가진 루프)

## Soft 키워드

다음 토큰은 적용 가능한 문맥에서는 키워드로 쓰이고, 다른 문맥에서는 식별자로 사용할 수 있다:

 * `by`
     - [인터페이스 구현을 다른 객체에 위임](delegation.html)
     - [프로퍼티를 위한 접근자 구현을 다른 객체에 위임](delegated-properties.html)
 * `catch`: [특정 익셉션 타입을 처리하는](exceptions.html) 블록 시작
 * `constructor`: [주요 또는 보조 생성자](classes.html#constructors) 선언
 * `delegate`: [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용 
 * `dynamic`: 코틀린/JS 코드에서 [동적 타입](dynamic-type.html)을 참조
 * `field`: [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `file`: [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `finally`: [try 블록이 끝날 때 항상 실행하는](exceptions.html) 블록 시작
 * `get`
     - [프로퍼티의 getter](properties.html#getters-and-setters) 선언
     - [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `import`: [현재 파일에 다른 패키지의 선언을 임포트](packages.html)
 * `init`: [초기화 블록](classes.html#constructors) 시작
 * `param`: [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `property`: [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `receiver`: [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `set`
     - [프로퍼티 setter](properties.html#getters-and-setters) 선언
     - [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `setparam`: [애노테이션 사용 위치 대상](annotations.html#annotation-use-site-targets)으로 사용
 * `where`: [지네릭 타입 파라미터에 대한 제약](generics.html#upper-bounds) 지정
 
## 수식어 키워드

다음 토큰은 선언의 수식어 목록에서 키워드로 사용하며, 다른 문맥에서는 식별자로 사용할 수 있다:

 * `actual`: [멀티 플랫폼 프로젝트](multiplatform.html)에서 플랫폼에 특정한 구현을 표시 
 * `abstract`: 클래스나 멤버를 [추상](classes.html#abstract-classes)으로 표시
 * `annotation`: [애노테이션 클래스](annotations.html) 선언
 * `companion`: [컴페니언 오브젝트](object-declarations.html#companion-objects) 선언
 * `const`: [컴파일 타입 상수](properties.html#compile-time-constants)로 프로퍼티 표시
 * `crossinline`: [인라인 함수에 전달한 람다에서 비-로컬 리턴](inline-functions.html#non-local-returns) 금지 
 * `data`: 컴파일러가 [클래스를 위한 canonical 멤버를 생성하도록](data-classes.html) 지시
 * `enum`: [열거형](enum-classes.html) 선언
 * `expect`: 플랫폼 모듈에서 구현할 거로 기대하는 [플랫폼에 특정한](multiplatform.html) 선언을 표시
 * `external`: 선언을 코틀린이 아닌 구현으로 표시([JNI](java-interop.html#using-jni-with-kotlin)로 접근 가능하거나 [자바스크립트](js-interop.html#external-modifier)에 구현한 것) 
 * `final`: [멤버 오버라이딩](classes.html#overriding-methods) 금지
 * `infix`: [중위 표기법](functions.html#infix-notation) 함수 호출 허용
 * `inline`: [호출 위치에서 전달한 함수와 람다를 인라인하도록](inline-functions.html) 컴파일러에게 말함
 * `inner`: [중첩 클래스](nested-classes.html)에서 외부 클래스의 인스턴스 참조를 허용
 * `internal`: 선언을 [현재 모듈에 보이도록](visibility-modifiers.html) 표시
 * `lateinit`: [생성자 밖에서 non-null 프로퍼티를](properties.html#late-initialized-properties-and-variables) 초기화하는 것을 허용
 * `noinline`: [인라인 함수에 전달한 람다를 인라인하지](inline-functions.html#noinline) 않음
 * `open`: [클래스 상속 또는 멤버 오버라이딩](classes.html#inheritance)을 허용
 * `operator`: 함수를 [연사자 오버로딩이나 컨벤션 구현으로](operator-overloading.html) 표시
 * `out`: 타입 파라미터를 [공변](generics.html#declaration-site-variance)으로 표시
 * `override`: [상위클래스 멤버의 오버라이드](classes.html#overriding-methods)로 멤버를 표시
 * `private`: 선언을 [현재 클래스나 파일에 보이도록](visibility-modifiers.html) 표시 
 * `protected`: 선언을 [현재 클래스와 그 클래스의 하위클래스에 보이도록](visibility-modifiers.html) 표시
 * `public`: 선언을 [모든 곳에 보이도록](visibility-modifiers.html) 표시
 * `reified`: 인라인 함수의 타입 파라미터를 [런타임에서 접근가능하게](inline-functions.html#reified-type-parameters) 표시
 * `sealed`: [sealed 클래스](sealed-classes.html) 선언 (제한된 하위클래스를 갖는 클래스)
 * `suspend`: 함수나 람다를 서스펜딩으로 표시([코루틴](coroutines.html)으로 사용가능)
 * `tailrec`: [꼬리 재귀](functions.html#tail-recursive-functions) 함수로 표시 (컴파일러가 재귀를 반복으로 바꾸도록 허용)
 * `vararg`: allows [파라미터를 가변 인자로 전달할 수 있게](functions.html#variable-number-of-arguments-varargs) 허용

## 특수 식별자

컴파일러는 특정한 문맥에서 다음 식별자를 정의하며, 다른 문백에서는 일반 식별자로 사용할 수 있다:

 * `field`: 프로퍼티 접근자에서 [프로퍼티의 지원 필드](properties.html#backing-fields)를 참조하기 위해 사용 
 * `it`: 람다 안에서 [명시적 선언 없이 파라미터에 접근하기 위해](lambdas.html#it-implicit-name-of-a-single-parameter) 사용
 
 
## 연산자와 특수 문자

코틀린은 다음 연산자와 특수 문자를 지원한다:

 * `+`, `-`, `*`, `/`, `%` - 수치 연산자
     - `*`: [가변 인자에 배열을 전달](functions.html#variable-number-of-arguments-varargs)할 때에도 사용
 * `=`
     - 할당 연산자
     - [파라미터의 기본 값](functions.html#default-arguments)을 지정할 때 사용 
 * `+=`, `-=`, `*=`, `/=`, `%=` - [augmented 할당 연산자](operator-overloading.html#assignments)
 * `++`, `--` - [증가 감소 연산자](operator-overloading.html#increments-and-decrements)
 * `&&`, `||`, `!` - 논리 'and', 'or', 'not' 연산자 (비트 연산자의 경우, 대응하는 [중의 연산자](basic-types.html#operations)를 사용)
 * `==`, `!=` - [동등비교 연산자](operator-overloading.html#equals) (기본 타입이 아닌 경우 `equals()` 호출로 변환) 
 * `===`, `!==` - [참조 동일비교 연산자](equality.html#referential-equality)
 * `<`, `>`, `<=`, `>=` - [비교 연산자](/docs/referencev/operator-overloading.html#comparison) (기본 타입이 아닌 경우 `compareTo()` 호출로 변환)
 * `[`, `]` - [인덱스기반 접근 연산자](operator-overloading.html#indexed) (`get`과 `set` 호출로 변환)
 * `!!` : [식이 null이 아님을 단언한다](null-safety.html#the--operator)
 * `?.` : [안전한 호출](null-safety.html#safe-calls) 수행 (리시버가 null이 아니면 메서드를 호출하거나 프로퍼티에 접근)
 * `?:` : 왼쪽 값이 null이면 오른쪽 값을 취한다([엘비스 연산자](null-safety.html#elvis-operator))
 * `::` : [멤버 레퍼런스](reflection.html#function-references)나 [클래스 레퍼런스](reflection.html#class-references) 생성
 * `..` : [범위](ranges.html) 생성 
 * `:` : 선언에서 타입과 이름을 구분
 * `?` : 타입을 [null 가능](null-safety.html#nullable-types-and-non-null-types)으로 표시 
 * `->`
     - [람다 식](lambdas.html#lambda-expression-syntax)의 파라미터와 몸체를 분리
     - [함수 타입](lambdas.html#function-types)에서 파라미터와 리턴 타입 선언을 분리
     - [when 식](control-flow.html#when-expression) 브랜치에서 조건과 몸체를 분리
 * `@`
    - [애노테이션](annotations.html#usage) 시작
    - [루프 라벨](returns.html#break-and-continue-labels) 시작 또는 참조 
    - [람다 라벨](returns.html#return-at-labels) 시작 또는 참조
    - [외부 범위에서 'this' 식](this-expressions.html#qualified)을 참조
    - [외부 상위클래스](classes.html#calling-the-superclass-implementation)를 참조
 * `;` : 같은 줄의 여러 문장을 구분
 * `$` : [문자열 템플릿](basic-types.html#string-templates)에서 변수나 식을 참조    
 * `_`
     - [람다 식](lambdas.html#underscore-for-unused-variables-since-11)에서 사용하지 않는 파라미터 대체
     - [분리 선언](/reference_ko/multi-declarations.html#underscore-for-unused-variables-since-11)에서 사용하지 않는 파라미터 대체
     