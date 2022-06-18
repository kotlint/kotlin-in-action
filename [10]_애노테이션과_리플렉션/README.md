# 애노테이션과 리플렉션

> 자바를 공부했을때, 애노테이션과 리플렉션은 따로 공부하는 것보다 같이 공부하는 것이 좋다고 생각했는데, 개인적으로 이 책에서 애노테이션과 리플렉션을 묶어서 단원으로 낸것이 마음에 든다.

애노테이션은 사실 아무 기능이 없는 Marker 같은 역할을 한다. 하지만, 우리가 프로그래밍을 하면서 다양한 애노테이션들을 사용하고 있으며, 각 어노테이션들은 특정 역할을 담당한다. 
이게 가능한 이유는 리플렉션이라는 기능 때문이다.

## 자바와는 조금 다른 문법 

```kotlin
/**
 * remove 함수 선언이 있다면, remove 를 호출하는 코드에 대해 경고 메시지를 표시해줄 뿐 아니라, 자동으로 그 코드를
 * 새로운 API 버전에 맞는 코드로 바꿔주는 퀵 픽스(quick fix)도 제시해 준다.
 */
@Deprecated("Use removeAt(index) instead.", ReplaceWith("removeAt(index)"))
fun remove(index: Int) {}
```

- __애노테이션 인자를 지정하는 문법은 자바와 약간 다르다__
  - 클래스를 애노테이션 인자로 지정할 때는 @MyAnnotation(MyClass::class) 처럼 ::class 를 클래스 이름 뒤에 넣어야 한다.
  - 다른 애노테이션을 인자로 지정할 때는 인자로 들어가는 애노테이션의 이름 앞에 @를 넣지 않아야 한다. 
    - 위 @Deprecated 안의 ReplaceWith 은 애노테이션이지만 @를 붙이지 않는다.
  - 배열을 인자로 지정하려면 arrayOf 함수를 사용한다.
    - @RequestMapping(path = arrayOf("/foo", "/bar"))
  - 자바에서 선언한 애노테이션을 사용할 때는 value 파라미터가 자동으로 가변 길이 인자로 변환 되기 때문에 arrayOf 를 사용하지 않아도 된다.
    - @JavaAnnotationWithArrayValue("abc", "foo", "bar")

### 애노테이션 인자는 컴파일 시점에 알 수 있어야 한다

- 애노테이션 인자는 컴파일 시점에 알 수 있어야 한다. 따라서, 임의의 프로퍼티를 인자로 지정할 수는 없다.
- 프로퍼티를 애노테이션 인자로 지정하려면 const 변경자를 붙여야 한다.
  - __컴파일러는 const 가 붙은 프로퍼티를 컴파일 시점 상수로 취급한다.__
  - ```kotlin
    const val TEST_TIMEOUT = 100L
    @Test(timeout = TEST_TIMEOUT) fun testMethod() { ... }
    ```
    
> 일반 프로퍼티를 애노테이션 인자로 사용하려 시도하면, "Only const val can be used in constant expressions" 라는 오류가 발생한다.

## 사용자 지점 대상 지정 문법

> @get:Rule: Rule 애노테이션을 Getter 에 적용하라는 의미

```kotlin
class Test {
  @get:Rule
  val folder = TemporaryFolder()
  
  ...
}
```

- __자바에 선언된 애노테이션을 사용해 프로퍼티에 애노테이션을 붙이는 경우 기본적으로 프로퍼티 `필드`에 적용된다.__
- __코틀린으로 애노테이션을 선언하면 경우는 프로퍼티에 직접 적용 할 수 있는 애노테이션을 만들 수 있다.__
  - 사용자 지점 대상을 지정할 때 지원하는 대상 목록
    - property: 프로퍼티 전체. 자바에서 선언된 애노테이션에는 이 사용 지점 대상을 사용할 수 없다.
    - field: 프로퍼티에 의해 생성되는 (뒷받침하는) 필드
    - get: 프로퍼티 게터
    - set: 프로퍼티 세터
    - receiver: 확장 함수나 프로퍼티의 수신 객체 파라미터
    - param: 생성자 파라미터
    - setparam: 세터 파라미터
    - delegate: 위임 프로퍼티의 위임 인스턴스를 담아둔 필드
    - file: 파일 안에 선언된 최상위 함수와 프로퍼티를 담아두는 클래스

## 자바 API 를 애노테이션으로 제어하기

- @JvmName: 코틀린 선언이 만들어내는 자바 필드나 메서드 이름을 변경
- @JvmStatic: 메서드, 객체 선언, 동반 객체에 적용하면 그 요소가 자바 정적 메서드로 노출됨
- @JvmOverloads: 디폴트 파라미터 값이 있는 함수에 대해 컴파일러가 자동으로 오버로딩한 함수를 생성
- @JvmField: 프로퍼티에 사용하면 게터나 세터가 없는 공개된(public) 자바 필드로 프로퍼티를 노출 시킨다.

## 애노테이션을 활용한 JSON 직렬화 제어

- __직렬화(serialization)__
  - 객체를 저장장치에 저장하거나 네트워크를 통해 전송하기 위해 텍스트나 이진 형식으로 변환하는 것
- __역직렬화(deserialization)__
  - 텍스트나 이진 형식으로 저장된 데이터로부터 원래의 객체를 만들어낸다.

직렬화에 자주 쓰이는 형식에는 JSON 이 있다. JSON <-> Object 를 변환할 때 자주 쓰이는 라이브러리로는 Jackson, GSON 이 있다.

## 제이키드 라이브러리 만들기

> https://github.com/yole/jkid

```kotlin
// Person 클래스 직렬화, 역직렬화 하기
data class Person(val name: String, val age: Int)

val person = Person("BAEK", 29)
println(serialize(person))
// {"age": 29, "name": "BAEK"}

val json = {"age": 29, "name": "BAEK"}
println(deserialize<Person>(json))
// Person(name="BAEK", age=29)
```

deserialize 에 객체의 타입을 명시해야하는 이유는, JSON 에는 객체의 타입이 저장되지 않기 때문에 JSON 데이터로부터 인스턴스를 만들려면 타입 인자로 클래스를 명시해야 한다.

제이키드 라이브러리의 두 애노테이션에 대해서 살펴보자.

- __@JsonExclude__
  - 애노테이션을 사용하면 직렬화나 역직렬화 시 그 프로퍼티를 무시할 수 있다.
- __@JsonName__
  - 애노테이션을 사용하면 프로퍼티를 표현하는 키/값 싸으이 키로 프로퍼티 이름 대신 애노테이션이 지정한 이름을 쓰게할 수 있다.

```kotlin
data class Person(
  @JsonName("alias") val firstName: String,
  @JsonExclude val age: Int? = null
)
```

## 애노테이션 선언

- 코틀린

```kotlin
// 파라미터가 있는 애노테이션을 정의하기 위해서는
// val 이 필수로 붙어야 하며, 생성자 파라미터를 선언해야 한다.
// 본문은 작성할 수 없다.
annotation class JsonName(val name: String) 
```

- 자바

```java
public @interface JsonName {
  String value()
}
```

자바의 경우에는 value() 메서드를 사용하는데, value 메서드는 특별하다. @WebServlet("abc") 와 같이 애노테이션에 이름을 사용할 수 있는 이유가, 자바 애노테이션에 value() 메서드가 있기 때문이다.

> [How can I do Java annotation like @name("Luke") with no attribute inside parenthesis?](https://stackoverflow.com/questions/11786354/how-can-i-do-java-annotation-like-nameluke-with-no-attribute-inside-parenth)

## 메타애노테이션

메타애노테이션은 애노테이션을 처리하는 방법을 제어하는 역할을 한다.

메타애노테이션은 애노테이션에 애노테이션을 붙일 수 있다.

> [Annotation](https://github.com/BAEKJungHo/deepdiveinreflection/blob/main/contents/Annotation.md)

## 애노테이션 파라미터 클래스로 사용 

```kotlin
interface Company {
  val name: String
}

data class CompanyImpl(override val name: String): Company

data class Person(
  val name: String,
  @DeserializeInterface(CompanyImpl:class) val company: Company
)
```

제이키드 라이브러리에 있는 @DeserializeInterface 는 인터페이스 타입인 프로퍼티에 대한 역직렬화를 제어할 때 쓰는 애노테이션이다. 인터페이스의 인스턴스를 직접 만들 수는 없다.
따라서, 역직렬화 시 어떤 클래스를 사용해 인터페이스를 구현할지 를 지정할 수 있어야 한다.

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class DeserializeInterface(val targetClass: KClass<out Any>)
```

KClass 는 자바 java.lang.Class 타입과 같은 역할을 하는 코틀린 타입니다. __코틀린 클래스에 대한 참조를 저장할 때 KClass 타입을 사용한다.__

CompanyImpl::class 타입은 `KClass<CompanyImpl>` 이며, `KClass<out Any>` 의 하위 타입이다.

KClass 의 타입 파라미터를 쓸 때 out 변경자 없이 `KClass<Any>` 라고 쓰면 Deserialize Interface 에게 CompanyImpl::class 를 넘길 수 없고 오직 Any::class 만 넘길 수 있다.
반면 out 키워드가 있으면 모든 코틀린 타입 T 에 대해 `KClass<T>` 가 `KClass<out Any>` 의 하위 타입이 된다.(공변성)

## 애노테이션 파라미터로 제네릭 클래스 받기

@CustomSerializer 는 `커스텀 직렬화 클래스`에 대한 참조를 인자로 받는다. 이 직렬화 클래스는 ValueSerializer 인터페이스를 구현해야만 한다.

```kotlin
interface ValueSerializer<T> {
    fun toJsonValue(value: T): Any?
    fun fromJsonValue(jsonValue: Any?): T
}
```

```kotlin
data class Person(
  val name: String,
  @CustomSerializer(DateSerializer::class) val birthdate: Date
)
```

ValueSerializer 클래스는 제네릭 클래스라서 타입 파라미터가 있다. 따라서 ValueSerializer 타입을 참조하려면 항상 타입 인자를 제공해야 한다.
하지만 이 애노테이션이 어떤 타입에 대해 쓰일지 전혀 알 수 없으므로 스타 프로젝션을 사용할 수 있다.

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class CustomSerializer(val serializerClass: KClass<out ValueSerializer<*>>)
```

`out ValueSerializer<*>` ValueSerializer 뿐만 아니라 ValueSerializer 를 구현하는 모든 클래스를 받아 들인다.

# 리플렉션: 실행 시점에 코틀린 객체 내부 관찰

애노테이션에 저장된 데이터에 접근하기 위해서는 리플렉션을 사용해야 한다. 리플렉션은 실행 시점에(동적으로) 객체의 프로퍼티와 메서드에 접근할 수 있게 해주는 방법이다.

> [Reflection](https://github.com/BAEKJungHo/deepdiveinreflection/blob/main/contents/Java%20Reflection.md)

코틀린에서 리플렉션을 사용하려면 두 가지 서로 다른 리플렉션 API 를 다뤄야 한다. __리플렉션을 사용하는 자바 라이브러리와 코틀린 코드는 완전히 호환된다.__

- __java.lang.reflect__
- __kotlin.reflect__
  - 이 API 는 자바에는 없는 프로퍼티나 널이 될 수 있는 타입과 같은 코틀린 고유 개념에 대한 리플렉션을 제공한다.
  - 하지만 현재 코틀린 리플렉션 API 는 자바 리플렉션 API 를 완전히 대체할 수 있는 복잡한 기능을 제공하지는 않는다.
  - 따라서, 자바 리플렉션을 대안으로 사용해야 하는 경우가 생긴다.
  - 코틀린 리플렉션 API 를 사용해도 다른 JVM 언어에서 생성한 바이트코드를 충분히 다룰 수 있다.

## 코틀린 리플렉션 API: KClass, KCallable, KFunction, KProperty

```kotlin
import kotlin.reflect.full.*

val person = Person("Alice", 29)
val kClass = person.javaClass.kotlin // KClass<Person> 의 인스턴스를 반환
println(kClass.simpleName) // Person
```

### KCallable

- KCallable 은 함수와 프로퍼티를 아우르는 공통 상위 인터페이스이다.
- KCallable 의 call 을 사용하면 함수나 프로퍼티의 게터를 호출할 수 있다.

```kotlin
interface KCallable<out R> {
  fun call(vararg args: Any?): R
  ...
}
```

call 을 사용할 떄는 함수 인자를 vararg 리스트로 전달한다.

```kotlin
fun foo(x: Int) = println(x)
>>> val kFunction = ::foo
>>> kFunction.call(42)
```

call 에 넘긴 인자 개수와 함수에 정의된 파라미터 개수가 맞아 떨어져야 한다. 예를 들어, kFunction.call() 로 호출하면 `IllegalArgumentException: Callable expects 1 arguements,
but 0 were provided` Runtime Exception 이 발생한다.

함수를 호출하기 위해 더 구체적인 메서드를 사용할 수도 있다.

`KFunctionN` 을 사용하면 된다.

```kotlin
kFunction1 // 1개의 인자를 받으며, 1개의 인자를 받아서 호출할 수 있는 invoke 메서드도 제공한다.
kFunction2 // 2개의 ...
...
```

kFunction 의 invoke 메서드를 호출할 때는 인자 개수나 타입이 맞아 떨어지지 않으면 컴파일이 안된다.

__KFunctionN 은 KFunction 을 확장하며, N 과 파라미터 개수가 같은 invoke 를 추가로 포함한다.__

예를 들어, KFunction2<P1, P2, R> 에는 operator fun invoke(p1: P2, p2: P2): R 선언이 들어있다.

이런 함수 타입들은 컴파일러가 생성한 합성 타입(synthetic compiler-generated type) 이다. 코틀린에서는 컴파일러가 생성한 합성 타입을 사용하기 때문에 원하는 수 만큼 많은 파라미터를 갖는 함수에
대한 인터페이스를 사용할 수 있다.

### KProperty

```kotlin
val counter = 0
>>> val kProperty = ::counter
>> kProperty.setter.call(21) // reflection 을 통해 세터를 호출하면서 21 인자를 넘긴다.
>> println(kProperty.get()) // 21
```

얘도 마찬가지로 KProperty 를 확장하는 KPropertyN 이 존재한다.

## 코틀린 리플렉션 API 인터페이스 계층 구조

- __KAnnotatedElement__
  - __KClass__
  - __KCallable__
    - KFunction
      - KFunctionN, KProperty.Getter, KProperty.Setter
    - KProperty
      - KMutableProperty, KPropertyN
  - __KParameter__
