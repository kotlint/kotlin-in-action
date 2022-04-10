# Decompile

## Kotlin

```kotlin
fun main(args: Array<String>) {
    println("Hello World!")
}
```

위 코드를 Decomplie 하면 어떻게 될까?

```java
@Metadata(
   mv = {1, 5, 1},
   k = 2,
   d1 = {"\u0000\u0014\n\u0000\n\u0002\u0010\u0002\n\u0000\n\u0002\u0010\u0011\n\u0002\u0010\u000e\n\u0002\b\u0002\u001a\u0019\u0010\u0000\u001a\u00020\u00012\f\u0010\u0002\u001a\b\u0012\u0004\u0012\u00020\u00040\u0003¢\u0006\u0002\u0010\u0005¨\u0006\u0006"},
   d2 = {"main", "", "args", "", "", "([Ljava/lang/String;)V", "src.main"}
)
public final class MainKt {
   public static final void main(@NotNull String[] args) {
      // Intrinsics: Kotlin.jvm.internal 에서 제공하는 함수 유틸리티 클래스
      Intrinsics.checkNotNullParameter(args, "args");
      String var1 = "Hello World!";
      boolean var2 = false;
      System.out.println(var1);
   }
}
```

## Java

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

Kotlin 과 똑같이 동작하는 코드를 작성하고 디컴파일을 하면 아래와 같다.

```java
public class Main {
    public Main() {
    }

    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

코틀린에 비해서 훨씬 양이 적은 것을 볼 수있다.

# @Metadata 살펴보기

> https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-metadata/

@Metadata Annotation 을 살펴보면 아래와 같이 Retention 과 Target 이 설정된 것을 볼 수 있다.

```java
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.CLASS)
```

Retention 정책을 RUNTIME 으로 설정한 것으로 보아 Reflection 에서 활용될 수 있다는 것을 짐작할 수 있다.

> Javadoc 을 읽어보자
> 
> This annotation is present on any class file produced by the Kotlin compiler and is read by the compiler and reflection.
Parameters have very short JVM names on purpose: these names appear in all generated class files, and we'd like to reduce their size.

이 주석은 Kotlin 컴파일러에서 생성된 모든 클래스 파일에 있으며 컴파일러와 리플렉션에서 읽는다. 매개변수는 의도적으로 매우 짧은 JVM 이름을 갖는다. 이러한 이름은 생성된 모든 클래스 파일에 나타나며, 크기를 줄이고 싶어한다.


@Metadata 에서 선언된 여러 변수 중, 디컴파일된 코드에서 표시된 `mv, k, d1, d2` 만 살펴보자.

- __mv__
  - ```java
    /**
     * The version of the metadata provided in the arguments of this annotation.
     */
    @get:JvmName("mv")
    val metadataVersion: IntArray = [],
    ```
- __k__
  - ```java
    /**
     * A kind of the metadata this annotation encodes. Kotlin compiler recognizes the following kinds (see KotlinClassHeader.Kind):
     *
     * 1 Class
     * 2 File
     * 3 Synthetic class
     * 4 Multi-file class facade
     * 5 Multi-file class part
     *
     * The class file with a kind not listed here is treated as a non-Kotlin file.
     */
    @get:JvmName("k")
    val kind: Int = 1,
    ```
- __d1__
  - 사용자 지정 형식의 메타데이터, 형식은 종류에 따라 다를 수 있다.(또는 아예 없을 수도 있다.)
  - ```
    /** Metadata in a custom format. The format may be different (or even absent) for different kinds. */
    @get:JvmName("d1")
    val data1: Array<String> = [],
    ```
- __d2__
  -  메타데이터에서 발생하는 문자열 배열로, 상수 풀에 이미 있는 문자열을 재사용할 수 있도록 일반 텍스트로 작성된다. 그런 다음 이 문자열은 이 배열의 정수 인덱스에 의해 메타데이터에서 인덱싱될 수 있다.
  - ```java
     // An addition to [data1]: array of strings which occur in metadata, written in plain text so that strings already present
     // in the constant pool are reused. These strings may be then indexed in the metadata by an integer index in this array.
    @get:JvmName("d2")
    val data2: Array<String> = [],
    ```
  
# Statement vs Expression
  
자바에서는 모든 제어 구조가 문(statement)인 반면, 코틀린에서는 루프를 제외한 대부분의 제어 구조가 식(expression)이다.
  
```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

정적 타입 지정언어임에도 불구하고 함수의 반환타입을 생략할 수 있는 이유는 `타입 추론(Type inference)` 때문이다.

# 변경 가능한 변수, 변경 불가능한 변수

- __val__
    - immutable variable
    - Java 의 final 변수에 해당
- __var__
    - mutable variable
- __전략__
    - 기본으로 val 을 사용하고 변경이 필요한 경우에만 var 을 사용한다.

# 문자열 템플릿

- `$변수명`
    - ```kotlin
      // 한글을 문자열 템플릿에서 사용할 때 { } 를 꼭 붙이자. 한글이 아니더라도 중괄호를 쓰는 습관을 들이자.
      println("${name}님 반가워요")
      ```
    - ```kotlin
      // 중괄호로 둘러싼 식 안에서 큰 따옴표 사용
      println("Hello, ${if (args.size > 0) args[0] else "someone"}!")
      ```

# 클래스

- __클래스 구조__
    - 이름
    - 헤더: 기본 생성자, 매개변수 등
    - 본문: 중괄호로 묶인 부분
    - ```kotlin
      class User(val name: String) { }
      ```
- __생성자__
    - 기본 생성자(Primary Constructor): 클래스 헤더의 일부이며, 클래스 이름 뒤에 선언한다.
        - ```kotlin
          // constructor 키워드 생략 가능
          class User constructor(val name: String) { }
          ```
        - 기본 생성자에 어노테이션이나, 접근 제한자(가시성 수정자)가 없으면 생략 가능하다.
    - 보조 생성자(Secondary Constructor)

아래 코드의 동작 순서를 예측해 보자.
          
```java
class InitOrderDemo(name: String) {
    // 속성 Initializer
    val firstProperty = "First property: $name".also(::println)

    // Initializer Block
    init {
        println("First initializer block that prints $name")
    }

    val secondProperty = "Second property: ${name.length}".also(::println)

    init {
        println("Second initializer block that prints ${name.length}")
    }
}

fun main(args: Array<String>) {
    InitOrderDemo("JungHo")
}
```

위 코드를 디컴파일해서 간략하게 보면 다음과 같다.

```java
public final class InitOrderDemo {
   @NotNull
   private final String firstProperty;
   @NotNull
   private final String secondProperty;

   @NotNull
   public final String getFirstProperty() {
      return this.firstProperty;
   }

   @NotNull
   public final String getSecondProperty() {
      return this.secondProperty;
   }

   public InitOrderDemo(@NotNull String name) {
      Intrinsics.checkNotNullParameter(name, "name");
      super();
      String var2 = "First property: " + name;
      System.out.println(var2);
      this.firstProperty = var2;
      var2 = "First initializer block that prints " + name;
      System.out.println(var2);
      var2 = "Second property: " + name.length();
      System.out.println(var2);
      this.secondProperty = var2;
      var2 = "Second initializer block that prints " + name.length();
      System.out.println(var2);
   }
}
```

즉, `클래스 본문 = 생성자 본문` 이라고 생각하면 된다.

## 보조 생성자

말 그대로 주 생성자 외에, 다른 생성자를 만드는 경우를 의미한다. 즉, `오버로딩한 생성자`라고 보면된다.

아래 코드가 어떻게 디컴파일될 지 예측해보자.

```kotlin
class InitOrderDemo(name: String) {
    constructor(key: Long): this(
        name = key.toString()
    )
}
```

디컴파일 결과는 다음과 같다.

```java
public final class InitOrderDemo {
   public InitOrderDemo(@NotNull String name) {
      Intrinsics.checkNotNullParameter(name, "name");
      super();
   }

   public InitOrderDemo(long key) {
      this(String.valueOf(key));
   }
}
```

> 디컴파일 결과를 유심하게 본 사람은 눈치 챘을 것이다. 코틀린의 기본 접근 제한자는 public 이며 생략 가능하다. 또한, 기본이 final 로 되어있다.

# 프로퍼티

> 클래스의 목적은 데이터의 캡슐화(encapsulation)이다.

자바에서는 필드와 접근자를 묶어 `프로퍼티(property)`라고 부른다.

```kotlin
class Person(
    // 읽기 전용 프로퍼티로, 코틀린은 비공개 필드와 공개 Getter 를 제공한다.
    val name: String,
    // 쓸 수 있는 프로퍼티로, 코틀린은 비공개 필드, 공개 Getter, Setter 를 제공한다.
    var isMarried: Boolean
)
```

# 커스텀 접근자

```kotlin
class InitOrderDemo(name: String) {
    // 자체 값을 저장하는 필드가 필요 없다. 게터만 존재한다.
    val firstProperty: String
        get() {
            return "FIRST_PROPERTY"
        }
}

// 호출
val orderDemo = InitOrderDemo("JungHo")
// getFirstProperty 메서드를 호출한 것이라고 생각하면 된다.
orderDemo.firstProperty
```

디컴파일 결과를 예측해보자.

```java
public final class InitOrderDemo {
   @NotNull
   public final String getFirstProperty() {
      return "FIRST_PROPERTY";
   }

   public InitOrderDemo(@NotNull String name) {
      Intrinsics.checkNotNullParameter(name, "name");
      super();
   }
}
```

# enum

- enum 은 soft keyword 라고 부른다.
    - class 의 앞에 선언 될 때만 특별한 의미를 지니고, 그 외에는 enum 을 변수명에 사용할 수 있다.

## values(), valueOf()

EnumClass 에서 자체적으로 상수를 가져오기 위한 values() 와 valueof() 를 제공한다.

```kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

만약에 valueOf 의 함수의 파라미터에 넘긴 value 값에 일치하는 상수가 없다면 IllegalArgumentException 을 발생시킨다.

> inline 함수를 만들어서 아래와 같이 응용해서 사용할 수 있다.

```kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

fun test() {
    printAllValues<RGB>() // RED, GREEN, BLUE
}
```

## Anonoymous classes

enum 상수들은 자체적으로 익명 클래스를 가질 수 있다.

```kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```

# when

when 은 자바의 switch 같은 역할을 한다.

```kotlin
fun mix(c1: Color, c2: Color) = when (setOf(c1, c2)) {
    setOf(Color.RED, Color.YELLOW) -> "ORANGE"
    else -> throw Exception("Dirty Color")
}
```

코틀린 1.3 부터는 when 의 대상을 변수에 대입할 수 있다.

```kotlin
fun Request.getBody() = 
    when (val response = executeRequest()) {
        is Success -> response.body
        is HttpError -> throw HttpException(response.status)
    }
```

이러한 방식의 장점은 when 문 안에서만 사용하는 변수를 명확하게 표시할 수 있다. (기존에는 when 밖에 선언되어 있는 변수에 식의 결과 값을 대입하고 when 을 사용해야 했다.)

when 은 인자가 존재하지 않아도 된다. 대신 각 분기의 조건이 Boolean 결과를 반환해야 한다.

```kotlin
fun mixOptimized(c1: Color, c2: Color) = 
    when {
        c1 == RED && c2 == YELLOW -> ORANGE
        else -> throw Exception("Dirty Color")
    }
```

# 스마트 캐스트

자바에서는 `instanceof` 키워드로 타입을 검사했다. 그리고 타입이 올바르면, `캐스팅`을 진행해야 했다.

```java
if (a instanceof Dog){  
   Dog dog = (Dog) a;
   dog.name;
   ...  
}  
```

코틀린에서는 `is` 라는 키워드로 변수 타입을 검사한다. 코틀린은 is 로 타입을 검사하고 나면 개발자가 굳이 변수를 원하는 타입으로 캐스팅하지 않아도 된다. 왜냐하면 컴파일러가
대신 캐스팅을 수행해주기 때문이다. 이를 `스마트 캐스트(smart cast)`라고 한다.

```kotlin
if (a instanceof Dog){  
   a.name
   ...
}  
```

스마트 캐스트가 동작하기 위해서는 is 로 변수에 든 값의 타입을 검사한 다음에 그 값이 `바뀔 수 없는 경우`에만 동작한다. 따라서, 스마트 캐스트를 사용한다면, 그 프로퍼티는 `val` 이며 커스텀 접근자를 사용하면 안된다.

# 명시적으로 캐스팅하기

코틀린에서는 `as` 라는 키워드를 사용하여 원하는 타입으로 변경할 수 있다.

```kotlin
// In Java - int n = (Integer) a
val n = a as Num
```

# while 과 for 루프

> 코틀린의 while 문은 자바와 동일하기 때문에 생략하겠다.

코틀린에서는 자바의 for 루프에 해당하는 요소가 없다. 대신 `range` 라는 것이 존재한다.

```kotlin
val oneToTen = 1..10
```
```kotlin
// 100 downTo 1 > 100부터 1까지
// step 2 > 2 단계씩 진행
for (i in 100 downTo 1 step 2) {
    ...
}

// 1부터 100까지
for (i in 1..100) {
    ...
}
```

`..` 은 항상 우항을 포함한다. 즉, 1..100 은 1부터 100까지를 의미한다. 만약에, 1부터 99까지(끝 값을 포함하지 않는 반만 닫힌 범위, half-closed range, 반폐구간 또는 반개구간)에
대해 이터레이션하면 편할 때가 있는데 `until` 이라는 키워드를 사용하면 된다.

```kotlin
// 0 ~ size-1
for (x in 0 until size)
```

## 맵에 대한 이터레이션

```kotlin
// 맵의 키와 값을 두 변수에 대입한다.
for ((letter, binary) in binaryReps) {
    ...
}
```

인덱스와 함께 컬렉션을 이터레이션할 수 있다.

```kotlin
for ((index, element) in list.withIndex()) {
   ... 
}
```

## in 으로 컬렉션이나 범위의 원소 검사

```kotlin
fun isDigit(c: Char) = c in '0'..'9'
fun isNotDigit(c: Char) = c !in '0'..'9'
```

특정 문자열이 집합에 속하는지 검사할 수 있다.

```kotlin
println("Kotlin" in setOf("Java", "Scala"))
```

# 예외 처리

코틀린의 예외 처리는 자바와 비슷하다. `throw` 를 사용하여 예외를 던질 수 있다.

```kotiln
val response = when (request.type) {
    PENDING -> doSomething1()
    DENY -> doSomething2()
    else -> throw NotFoundTypeException()
}
```

## try, catch, finally

코틀린과 자바의 가장 큰 차이는 throws 절이 코드에 없다는 점이다.

```java
// In Java
public void create() throws IOException {
    // do something
}
```

```kotlin
fun create() {
    // do something
}
```

자바에서 IOException 은 checked exception 이기 때문에 명시적으로 처리해야 한다. 반면, 코틀린은 checked exception 과 unchecked exception 을 구별하지 않는다.

## 코틀린이 예외를 구별하지 않는 이유

> https://kotlinlang.org/docs/exceptions.html#the-nothing-typeS
  
StringBuilder 를 사용하는 코드를 작성한다고 가정하자.

```java
Appendable append(CharSequence csq) throws IOException;
```

append 시 IO 작업을 수행할 수도 있기 때문에 메서드 시그니처에 IOException 이 붙은 것을 볼 수 있다. 따라서, 아래와 같이 try-catch 를 작성해야 한다.

아래와 같은 코드는 좋은 코드일까?

```java
try {
    log.append(message) 
} catch (IOException e) {
    // Must be safe
}
```

Effective Java 3. Item 77 을 보면 예외 처리란, 예외상황에 적절한 처리를 하기 위해 존재한다라고 설명되어있다. 위와 같은 코드를 `예외 블랙홀` 이라고 한다. 

> 예외 블랙홀은 아무 예외도 처리하지 않거나. e.printStackTrace 로 로그만 출력하는 경우를 의미한다.

Bruce Eckel 이라는 사람은 Checked Exception 에 대해서 아래와 같이 말한다.

> 소규모 프로그램을 조사하면 예외 사양을 요구하는 것이 개발자 생산성을 향상시키고 코드 품질을 향상시킬 수 있다는 결론으로 이어집니다. 그러나 대규모 소프트웨어 프로젝트의 경험은 다른 결과를 나타냅니다. 생산성은 감소하고 코드 품질은 거의 또는 전혀 증가하지 않습니다.

- [The Trouble with Checked Exceptions (Anders Hejlsberg)](https://www.artima.com/articles/the-trouble-with-checked-exceptions)
- [Java's checked exceptions were a mistake (Rod Waldhoff)](https://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html)
- [Checked Exceptions Are Of Dubious Value](http://wiki.c2.com/?CheckedExceptionsAreOfDubiousValue)
- [Exception Tunneling](http://wiki.c2.com/?ExceptionTunneling)

__즉, 자바에서 Checked Exception 이 발생하는 코드를 작성하다보면 예외 블랙홀이 생길 수도 있고, 대규모 소프트웨어 프로젝트에서 생산성과 코드의 품질이 감소하기 때문에 코틀린에서는 Exception 을 구분하지 않는다.__

코틀린의 try-catch 구문은 Expression 이기 때문에 아래와 같이 사용할 수 있다.

```kotlin
// catch 에서 값 반환하기
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        null
    }
}
```

## Result

try - catch - finally 구문은 Java 스럽다. 뭔가 좀 더 `코틀린스럽게 Exception Handling` 을 하고 싶다.

코틀린 1.3 에서는 Result 클래스와 runCatching 이라는 것을 제공해 준다. (kotlin 패키지안에 존재한다.)

> runCatching 

```kotlin
/**
 * Calls the specified function [block] and returns its encapsulated result if invocation was successful,
 * catching any [Throwable] exception that was thrown from the [block] function execution and encapsulating it as a failure.
 */
@InlineOnly
@SinceKotlin("1.3")
public inline fun <R> runCatching(block: () -> R): Result<R> {
    return try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
}
```

아래와 같은 예제 코드를 만들었다.

```kotlin
// NumberFormatException 발생 가능성이 있는 함수
fun read(reader: BufferedReader): Int {
    return reader.readLine().toInt()
}
```

```kotlin
fun exceptionHandling() {
    val reader = BufferedReader(InputStreamReader(System.`in`))
    runCatching {
        read(reader)
    }.onSuccess {
        println("do Something")
    }.onFailure {
        e -> e.printStackTrace()
    }.also {
        println("do finally here")
    }
}
```

exceptionHandling() 을 실행하고 runCatching 구문에서 예외가 발생한다면 onFailure 에서 잡힐 것이고, 끝나면 finally 구문에 해당하는 also 가 실행될 것이다. 예외가 발생하지 않는다면 onSuccess 안의 로직들을 실행할 것이다.

## References

- https://developer88.tistory.com/257
