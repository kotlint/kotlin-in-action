# 함수 정의와 호출

## 정적 유틸리티 클래스 없애기

자바에서는 유틸리티 클래스를 만들 때 아래와 같이 사용했을 것이다.

```java
public final class CollectionUtils {
  // Implementation
}
```

코틀린에서는 따로 컬렉션 기능을 제공하지 않는다. 그 이유는 이유는 `자바와의 상호 작용성` 때문이다. 
코틀린에서 자바 함수를 호출할 때 자바와 코틀린 컬렉션을 서로 변환할 필요가 없다.

> 여기서도 코틀린 철학을 엿볼 수 있다.

코틀린에서는 유틸리티용 클래스를 만들 필요가 없다. 어떠한 유틸리티들의 모음인지 `파일명`으로 나타내고, 최상위 프로퍼티와 함수를 사용하면 된다.

```kotlin
// CollectionUtils.kt

@file:JvmName("StringFunctions") // 클래스 이름을 지정
package utils

/** 최상위 함수 */
/** 자바에서는 파일명.메서드명으로 호출 Ex. CollectionUtils.joinToString */

/**
 * @JvmOverloads -> 파라미터 개수가 1개인 것 부터 4개인 것 까지 오버로딩한 함수가 만들어진다.
 */
@JvmOverloads
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}

/**
 * 최상위 프로퍼티
 *
 */
var operationCount = 0
```

## 확장 함수

> 자바로 작성된 코드와 코틀린 코드를 자연스럽게 통합하는 것은 코틀린의 핵심 목표 중 하나이다.

확장 함수(extension function)는 기존 자바 API 를 재작성하지 않고도 코틀린이 제공하는 편리한 기능을 사용하게 해준다.

```kotlin
fun String.lastChar(): Char = this.get(this.length - 1)
```

자바에서 확장 함수를 호출하는 방법은 아래와 같다.

```java
// KotlinFileName.ExtensionFunction
char c = StringUtilsKt.lastChar("Java")
```

### receiver

> 수신 객체라는 말은 생소할 것이다. 자바언어를 사용하면서 개발을 할 때에도 들어보지 못했다. 하지만 어려운 개념은 아니다.

```
someObject.someFunction()
```

someObject 가 바로 `receiver` 다.

### 임포트

확장 함수를 사용하기 위해서는 똑같이 Import 를 해야 한다.

- import 패키지명.확장 함수명
- import 패키지명.와일드카드

## 확장 함수로 유틸리티 함수 정의

> 확장 함수는 클래스 밖에 선언되기 때문에 오버라이드를 할 수 없다.

```kotlin
@JvmOverloads
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    // this 는 T 타입의 수신객체
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
```

## 가변인자

매개변수의 개수가 달라질 가능성이 있으면서, 매개변수의 개수와 상관없이 내부 로직을 동일하게 가져가야하는 경우에는 `vararg` 키워드를 사용해서 함수를 만들 수 있다.

```kotlin
/**
 * Returns a new read-only list of given elements.  The returned list is serializable (JVM).
 * @sample samples.collections.Collections.Lists.readOnlyList
 */
public fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()
```

사용은 아래와 같다.

```kotlin
var list = listOf(1, 2, 3)
```

## 스프레드 연산자

`*` 키워드를 spread 연산자라고 한다.

```kotlin
fun printList(args: Array<String>) {
    val list = listOf("args : ", *args)
    println(list)
}
```

## 중위 호출(infix call)

중위 호출은 receiver 와 유일한 메서드 인자 사이에 메서드 이름을 넣는다. 이때 메서드 이름과 인자 사이에는 공백이 들어가야 한다.

```kotlin
// 일반적인 방식
1.to("one")
// 중위 호출 방식
1 to "one"
```

- __중위 호출을 사용할 수 있는 경우__
  - 인자가 하나뿐인 메서드
  - 인자가 하나뿐인 확장 함수
- __함수를 중위 호출을 사용할 수 있도록 허용하기 위한 방법__
  - `infix` 변경자를 함수 앞에 추가
  - ```kotlin
    // Pair 를 반환하는 to 함수
    infix fun Any.to(other: Any) = Pair(this, other)
    ```
    
## 구조 분해 선언

구조 분해 선언(destructuring declaration)은 자바스크립트에서 자주 사용되는 기능이다.

```kotlin
val (number, name) = 1 to "one"
```

```kotlin
fun test() {
    val collection = listOf("A", "B", "C")
    for ((index, element) in collection.withIndex()) {
        println("$index: $element")
    }
}
```

위 코드를 디컴파일해보자.

```kotlin
@Metadata(
   mv = {1, 5, 1},
   k = 2,
   d1 = {"\u0000\b\n\u0000\n\u0002\u0010\u0002\n\u0000\u001a\u0006\u0010\u0000\u001a\u00020\u0001¨\u0006\u0002"},
   d2 = {"test", "", "src.main"}
)
public final class DestructuringKt {
   public static final void test() {
      List collection = CollectionsKt.listOf(new String[]{"A", "B", "C"});
      int index = 0;

      for(Iterator var3 = ((Iterable)collection).iterator(); var3.hasNext(); ++index) {
         String element = (String)var3.next();
         String var4 = index + ": " + element;
         System.out.println(var4);
      }
   }
}
```

### 객체 구조 분해 선언

```kotlin
val (name, age) = person
```

위 코드를 디컴파일하면 아래와 같은 모습이다.

```kotlin
val name = person.component1()
val age = person.component2()
```

> The component1() and component2() functions are another example of the principle of conventions widely used in Kotlin (see operators like + and *, for-loops as an example). The componentN() functions need to be marked with the operator keyword to allow using them in a destructuring declaration.

## 확장 함수와 로컬 함수

```kotlin
class User(val id: Int, val name: String, val address: String) {
    fun User.validateBeforeSave() { // 확장 함수
        fun validate(value: String, fieldName: String) { // 로컬 함수
            if (value.isEmpty()) {
                throw IllegalArgumentException()
            }
        }
        validate(name, "Name")
        validate(address, "Address")
    }
    
    fun saveUser(user: User) {
        user.validateBeforeSave()
    }
}
```

위 코드에서 validateBefore 라는 확장 함수를 만들고 그 안에, 저장하기 전에 필요한 검증 로직을 해야하는 로컬 함수들을 여러개 만든다고 해보자.

> 개인적으로는 과연 Best Practice 일까 의문이 든다.

응집력이 있는 것은 좋지만, 함수의 깊이가 너무 깊어질 가능성이 있고 가독성 측면에서 좋지 않을 수 있다.

> 소트웍스 앤솔러지 객체지향 생활 체조에서는 `한 메서드에 오직 한 단계의 들여쓰기만 한다` 라는 규칙을 소개하고 있다. 

확장 함수를 만들고 그 안에 로컬 함수를 만들어서 응집력을 높이는 것보다는, 다른 방법을 사용해서 응집력을 높이고 가독성도 높이는게 좋지 않을까 생각한다.

### Refactoring

> 위 코드를 내가 생각한 Best Practice 방식으로 리팩토링하면 아래와 같다.

```kotlin
class User(val id: Int, private val name: String, private val address: String) {
    fun save(user: User) {
        validateBeforeSave(user)
        // do save
    }

    private fun validateBeforeSave(user: User) {
        checkNotNullParameter(user.name, "Name")
        checkNotNullParameter(user.address, "Address")
        // do something else
    }
}

// 다른 클래스에서도 공통으로 쓰일만한 함수는 유틸리티성으로 관리하는게 좋다
fun checkNotNullParameter(value: String, fieldName: String) {
    if (value.isEmpty()) throw IllegalArgumentException()
}
```

## References

- https://stackoverflow.com/questions/45875491/what-is-a-receiver-in-kotlin
