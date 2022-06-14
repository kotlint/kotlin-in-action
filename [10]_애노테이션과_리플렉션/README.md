# 애노테이션과 리플렉션

> 자바를 공부했을때, 애노테이션과 리플렉션은 따로 공부하는 것보다 같이 공부하는 것이 좋다고 생각했는데, 개인적으로 이 책에서 애노테이션과 리플렉션을 묶어서 단원으로 낸것이 마음에 든다.
>
> - [Reflection](https://github.com/BAEKJungHo/deepdiveinreflection/blob/main/contents/Java%20Reflection.md)
> - [Annotation](https://github.com/BAEKJungHo/deepdiveinreflection/blob/main/contents/Annotation.md)

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

