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

어떤 함수가 값이 널인지 검사한 다음에 다른 함수를 호출한닫고 해도 컴파일러는 호출된 함수 안에서 안전하게 그 값을 사용할 수 있음을 인식할 수 없다.

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

## References

- https://tourspace.tistory.com/208
