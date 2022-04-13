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

### receiver

> 수신 객체라는 말은 생소할 것이다. 자바언어를 사용하면서 개발을 할 때에도 들어보지 못했다. 하지만 어려운 개념은 아니다.

```
someObject.someFunction()
```

someObject 가 바로 `receiver` 다.

## References

- https://stackoverflow.com/questions/45875491/what-is-a-receiver-in-kotlin
