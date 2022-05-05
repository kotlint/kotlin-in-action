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
