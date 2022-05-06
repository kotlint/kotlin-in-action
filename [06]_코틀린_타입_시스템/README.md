# 코틀린 타입 시스템

- __코틀린이 타입 시스템을 통해 코드 가독성을 향상시키는 데 도움이 되는 특성__
  - 널이 될 수 있는 타입
  - 읽기 전용 컬렉션

## nullable

- 코틀린의 널 가능성(nullability)은 NPE 를 피할 수 있게 돕기 위한 코틀린의 타입 시스템의 특성이다.
- 널이 될 수 있는지 여부를 타입 시스템에 추가함으로써 컴파일러가 여러 가지 오류를 컴파일 타임에 감지한다.

## 널이 될 수 있는 타입

널이 될 수 있는 타입이란, 프로퍼티나 변수에 null 을 허용하게 만드는 방법이다. 자바에서는 널이 될 수 있는 타입이라는 것이 존재하지 않아서 매개변수나 변수가 null 인지를 항상 검사해야 했다.

코틀린에서는 `?` 를 타입 이름 뒤에 붙이면 널이 될 수 있는 타입을 나타낸다.

```kotlin
val name: String? = null
```

## 안전한 호출 연산자

`?.` 는 안전한 호출 연산자라고 부른다.

```kotlin
// = if (name != null) name.toUpperCase()
name?.toUpperCase()
```

아래처럼 연쇄적으로 사용할 수 도 있다.

```kotlin
fun Person.countryName(): String {
  val country = this.company?.address?.country
  // 생략
}
```

## 엘비스 연산자

`?:` 를 엘비스 연산자라고 한다. 

엘비스 연산자는 null 대신 사용할 디폴트 값을 지정할때 사용한다. 

```kotlin
// findById 의 결과가 null 이면 예외를 발생 시킨다.
repository.findById(id) ?: throw EntityNotFoundException()
```

## 안전한 캐스트: as?

`as?` 는 값을 대상 타입으로 변환할 수 없으면 null 을 반환한다. 안전한 캐스트를 사용할 때 일반적인 패턴은 캐스트를 수행한 뒤에 엘비스 연산자를 사용하는 것이다.

```kotlin
class Person(val firstName: String, val lastName: String) {
  override fun equals(o: Any?): Boolean {
    val otherPerson = o as? Person ?: return false // 타입이 서로 일치 하지 않으면 false 를 반환한다.
    return otherPerson.firstName == firstName && otherPerson.lastName == lastName
  }
}
```

## 널 아님 단언: !!

널 아님 단언(not-null assertion)은 코틀린에서 널이 될 수 있는 타입의 값을 다룰 때 사용할 수 있는 도구 중에서 가장 단순하다.

```kotlin
fun ignoreNulls(s: String?) {
  val sNotNull: String = s!! // NPE 발생할 수 있음
}
```

> !! 를 남발하게 되면, 예상치 못한 곳에서 NPE 가 발생할 수 있다. 
>
> 코틀린 설계자들은 컴파일러가 검증할 수 없는 단언을 사용하기보다는 더 나은 방법을 찾아보라는 의도를 넌지시 표시하려고 !! 라는 못생긴 기호를 선택하였다.

### 널 아님 단언이 더 나은 해법인 경우

어떤 함수가 값이 널인지 검사한 다음에 다른 함수를 호출한다고 해도 컴파일러는 호출된 함수 안에서 안전하게 그 값을 사용할 수 있음을 인식할 수 없다. 이 경우에는 단언을 사용하는 것이 좋다.

## let

let 은 자신의 수신 객체를 인자로 받아서 사용한다. 안전한 호출 연산자랑 같이 사용하면 널이 될 수 있는 식을 더 쉽게 다룰 수 있다.

```kotlin
// if (email != null) sendEmailTo(email)
email?.let { sendEmailTo(it) }
```

> 만약에 여러 값이 null 인지 검사해야 하는 경우, let 을 중첩해서 처리하는 방식보다, if 문을 사용하는 편이 더 나을 수 있다.

## lateinit

lateinit 은 이름에서 알 수 있듯이 지연 초기화를 위해 사용되는 키워드다. 

그럼 언제 사용할까?

JUnit 에서는 @Before 메서드 안에서 초기화 로직을 수행해야만 한다. 하지만 코틀린에서는 일반적으로 생성자에서 모든 프로퍼티 값을 초기화 한다. 게다가 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 그 프로퍼티를 초기화해야 한다.

```kotlin
class Test {
  private lateinit var service: MyService
  
  @Before
  fun setUp() { 
    service = MyService()
  }

}
```

lateinit property 는 항상 var 이어야 한다. val property 는 항상 final 필드로 컴파일 되며, 생성자 안에서 초기화 해야 한다. 그래서 @Service 어노테이션이 적용된 클래스에서 DI 를 하기 위해서는 아래와 같이 사용한다.

```kotlin
@Service
class OrderService(
    private val orderCommandService: OrderCommandService,
    private val orderQueryService: OrderQueryService,
) {
 ...
}
```

위 코드에 @Autowired 어노테이션이 필요하지 않은 이유는 무엇일까?
 
스프링 의존성 주입의 특징 중 한가지는 다음과 같다.

__어떠한 빈에 생성자가 오직 하나만 있고, 생성자의 파라미터 타입이 빈으로 등록 가능한 존재라면 이 빈은 @Autowired 어노테이션 없이도 의존성 주입이 가능하다.__

그래서 자바에서는 보통 롬복에서 제공하는 @RequriedArgsConstructor 어노테이션과 같이 사용한다.

```java
@RequriedArgsConstructor
@Service
public class OrderService {
  private final OrderCommandService orderCommandService;  
  private final OrderQueryService orderQueryService;
}
```

## 널이 될 수 있는 타입의 확장함수

isEmpty(빈 문자열("") 검사) 나 isBlank(공백 검사) 처럼 널을 검사할 수 있으면 편리할 것이다.

```kotlin
fun String?.isNullOrBlank(): Boolean = this == null || this.isBlank() // 두 번째 this 에는 스마트 캐스트가 적용된다.
```

```kotlin
fun verifyUserInput(input: String?) {
  if (input.isNullOrBlank()) {
    println("Please fill in the required fields")
  }
}
```

### 타입 파라미터의 널 가능성

```kotlin
fun <T> printHashCode(t: T) {
  println(t?.hashCode())
}
```

여기서 T 는 Any? 타입이기 때문에 null 이 될 수 있다. null 이 될 수 없는 타입으로 변경하려면 아래와 같이 하면 된다.

```kotlin
fun <T: Any> printHashCode(t: T) {
  println(t.hashCode())
}
```

## 널 가능성과 자바

코틀린은 자바 상호운용성을 중요시한다. 자바에서는 타입에 대해 널 가능성을 지원하지 않는다. 

자바에서는 어노테이션으로 널 가능성 정보를 표시해준다.

```java
public String print(@Nullable String s)
```

코틀린에서는 `String?` 처럼 표시된다.  

- __(Java) @NotNull + Type = (Kotlin) Type__
- __(Java) @Nullable + Type = (Kotlin) Type?__

이런 널 가능성 어노테이션들이 소스코드에 없는 경우에는 자바의 타입은 코틀린의 플랫폼 타입이 된다.

### 플랫폼 타입

플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입을 말한다. 따라서, 그 타입을 널이 될 수 있는 타입으로 처리해도되고 널이 될 수 없는 타입으로 처리해도 된다.

- __(Java) Type = (Kotlin) Type or Type?__
  - 자바 타입은 코틀린에서 플랫폼 타입으로 표현된다.
  - 플랫폼 타입은 널이 될 수 있는 타입이나 널이 될 수 없는 타입 모두로 사용할 수 있다.
  - 코틀린에서 플랫폼 타입을 선언할 수는 없다. 자바 코드에서 가져온 타입만 플랫폼 타입이 된다.
 
> 코틀린이 왜 플랫폼 타입을 도입했을까?
>
> 모든 자바 타입을 널이 될 수 있는 타입으로 다루면 더 안전하지 않을까? 그래도 되지만, 모든 타입을 널이 될 수 있는 타입으로 다루면 결코 널이 될 수 없는 값에 대해서도 불필요한 널 검사가 들어간다.

### 상속

```java
/** Java */
interface StringProcessor {
  void process(String value);
}
```

코틀린 컴파일러는 다음과 같은 두 구현을 모두 받아들인다.

```kotlin
class StringPrinter: StringProcessor {
  override fun process(value: String) {
    println(value)
  }
}

class StringPrinter: StringProcessor {
  override fun process(value: String?) {
    if (value != null) {
      println(value)
    }
  }
}
```

# 코틀린의 원시 타입

코틀린은 원시 타입과 래퍼 타입을 구분하지 않는다.

> 반면, 자바는 구분한다.

## 원시 타입

자바의 원시 타입에는 값이 들어가지만, 참조 타입(String 등)에는 메모리 상의 객체 위치가 들어간다. 참조 타입을 썼을 때의 장점은 원시 타입 값을 더 효율적으로 저장하고 여기저기 전달할 수 있다. 그리고 원시 타입을 객체로 Wrapping 함으로써, 특정 값이 갖는 역할에 대해서 정의할 수도 있다.

아래는 지하철 요금과 관련된 fare 라는 원시 타입을 객체로 Wrapping 한 것이다. 따라서, fare 가 담당하는 역할을 정의하고 다양한 요금 정책에서 사용할 수 있다.

```java
public class Fare {

    private int fare;
    private static final int STANDARD_FARE = 1_250;

    public static Fare standard() {
        return new Fare(STANDARD_FARE);
    }

    private Fare(int fare) {
        this.fare = fare;
    }

    public void add(int value) {
        fare += value;
    }

    public void change(int value) {
        fare = value;
    }

    public int getFare() {
        return fare;
    }
}
```

> stack, heap 영역에 대해 잘 아는 것이 중요하다.

코틀린은 원시 타입과 래퍼 타입을 구분하지 않고 항상 같은 타입을 사용한다. 또한 코틀린은 숫자 타입 등 원시 타입의 값에 대해 메서드를 호출 할 수 있다.

```kotlin
fun showProcess(progress: Int) {
  val percent = progress.coerceIn(0, 100) // 값을 특정 범위로 제한
  println("We're ${percent}% done!"}
}

>>> showProgress(146)
We're 100% done!
```

코틀린에서 원시 타입과 래퍼 타입을 구분하지 않는다고 해서 항상 객체로 표현되는 것은 아니다. 

실행 시점에 숫자 타입은 가능한 한 가장 효율적인 방식으로 표현된다. 대부분의 경우(변수, 프로퍼티, 파라미터, 반환 타입 등) 코틀린의 Int 타입은 자바 int 타입으로 컴파일된다.
단, 컬렉션과 같은 제네릭 클래스를 사용하는 경우에는 이런 컴파일이 불가능하다. 예를 들어 Int 타입을 컬렉션의 타입 파라미터로 넘기면 자바의 Integer 로 변환된다.

마찬가지로 자바의 원시 타입은 Not null 이기 때문에 코틀린에서 사용할 때도 널이 될 수 없는 타입으로 취급된다.

## References

- https://tourspace.tistory.com/208
