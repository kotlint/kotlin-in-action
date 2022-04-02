# 코틀린이란 무엇이며, 왜 필요한가?
 
- _코틀린의 철학_
  - __코틀린은 간결하고 실용적이며, 자바 코드와의 `상호 운용성(Interoperability)` 을 중시한다.__
- 코틀린은 자바보다 `생산성`이 더 높다.
- 코틀린의 성능은 자바와 같은 수준이다.
  - 책에서는 이렇게 나와있다. Kotlin Compiler 개발자가 그렇다고 하면 뭐, 거의 맞는 말이지만...
  - Kotlin Compiler 에 의해 생성되는 코드를 보면 꽤 많은 양의 코드가 작성된다.
  - ms 단위 차이의 성능은 존재한다.
  - 즉, Java 보다 느린건 사실

> 상호 운용성(Interoperability): Kotlin 에서 Java 코드 호출, Java 에서 Kotlin 코드 호출

## Kotlin 에서 Java 코드 호출

- Kotlin 은 Java 를 고려하여 설계되었기 때문에 비교적 자연스럽게 사용 가능

## Java 에서 Kotlin 코드 호출

- 기존 Java 는 Kotlin 을 고려하여 설계된 것이 아니기 때문에, 항상 자연스럽지는 않음
- Java 쪽에서 Kotlin 클래스가 `어떻게 보이는 지`가 중요
- 특히 Reflection 을 활용하는 경우
 - Ex. JPA, Proxy ...

# Type

> 코틀린, 자바, 타입스크립트 등은 자바스크립트와 달리 다양한 타입을 지원한다. 타입을 통해 얻는 이점이 무엇일까?

코틀린의 경우에 `kotlinc` 에 들어있는 `TypeChecker`를 통해 Runtime 이 아닌 컴파일 타임에 타입을 체크해준다. 따라서 RuntimeException 으로부터 안전하다.

> 코틀린의 Null safety 한 특징도 RuntimeException 을 방지한다.

__즉, Kotlin 은 `안전성`을 중시하는 언어이고, 런타임 오류를 방지하도록 설계되었다.__

# 대상 플랫폼

서버, 안드로이드 등 자바가 실행되는 모든 곳

# 정적 타입 지정 언어(statically typed)

- 정적 타입 지정 언어란 모든 프로그램의 구성 요소의 타입을 `Compile time` 에 알 수 있고, 프로그램 안에서 객체의 필드나 메서드를 사용할 때마다 컴파일러가 타입을 검증해준다는 뜻이다.
  - 동적 타입 지정 언어는 타입과 관계없이 모든 값을 변수에 넣을 수 있고, 메서드나 필드 접근에 대한 검증이 `Run time` 에 알 수 있다.
- 코틀린은 `타입 추론(Type Inference)` 을 통해서 변수의 타입을 결정한다.

# 코틀린의 장점

## Null safety

> 코틀린이 갖는 가장 큰 장점중 하나

Kotlin's type system is aimed at __eliminating the danger of null references__, also known as __The Billion Dollar Mistake__

> The Billion Dollar Mistake
> 
> Speaking at a software conference in 2009, Tony Hoare apologized for inventing the null reference

Elvis 연산자와 !! 연산자 맛보기

- __Elvis__
  - ```kotlin
    // b is not null ? b.length : -1
    val l = b?.length ?: -1
    ```
- __!!__
  - ```kotlin
    // b is not null ? b.length : throw NPE
    val l = b!!.length
    ```
    
> 프로그램이 NullPointerException 으로 인해 중단되거나 충돌할 때 Java 개발자에게는 항상 고통 스러운 일이다. 이제 Kotlin을 사용하여 런타임 프로그램 충돌에 작별을 고할 수 있다.

## 간결한 코드

- Java

```java
final List<String> greetings = people.stream()
  .map(it -> "Hello " + it.getName() + "!")
  .collect(Collectors.toList());
```

- Kotlin

```kotlin
val greetings = people.map { "Hello ${it.name}!" }
```

## 코틀린은 함수형 프로그래밍을 풍부하게 지원한다.

- 함수형 프로그래밍을 강제하진 않지만, 함수형 프로그래밍을 할 수 있도록 풍부하게 지원한다.
- __함수형 프로그래밍의 특징__
  - `일급 시민(First class) 함수`
    - 함수를 일반 값처럼 다룰 수 있다. 변수에 저장할 수 있고, 다른 함수의 인자로 전달할 수 있으며, 함수에서 새로운 함수를 만들어서 반환할 수 있다.
  - `불변성(immutablility)`
    - 일단 만들어지면 내부 상태가 변하지 않는다.
  - `부수 효과(side effect) 없음`
    - 함수형 프로그래밍에서는 입력이 같으면 항상 같은 출력을 내놓고, 객체의 상태를 변경하지 않으며, 함수 외부나 다른 바깥 환경과 상호작용하지 않는 `순수 함수(pure function)`를 사용한다.
- __함수형 프로그래밍으로 인한 장점__
  - 간결성, 가독성
  - 함수를 값 처럼 활용하기 때문에 강력한 `추상화(abstraction)` 제공
  - 멀티 스레드에서 안전하다.
  - 테스트 코드 작성이 쉽다.

> 객체지향 프로그래밍은 움직이는 부분을 캡슐화하여 코드 이해를 돕고, 함수형 프로그래밍은 움직이는 부분을 최소화하여 코드 이해를 돕는다. - 마이클 페더스 

# 컴파일

![kotlin-runtime-diagram](https://user-images.githubusercontent.com/47518272/159108434-73c18498-c55c-43d3-a568-96a02fd063d8.jpg)

코틀린 컴파일러로 컴파일한 코드는 코틀린 런타임 라이브러리에 의존한다.

- _Kotlin Runtime Library_
  - 코틀린 자체 표준 라이브러리 클래스 + 자바 API 의 기능을 확장한 내용

코틀린은 Maven, Gradle, Ant 등의 빌드 시스템과 호환된다.

## References

- https://kotlinlang.org/docs/null-safety.html
