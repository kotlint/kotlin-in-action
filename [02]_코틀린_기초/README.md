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
