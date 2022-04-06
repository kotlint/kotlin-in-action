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
    val isMarried: Boolean
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
