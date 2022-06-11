# 파라미터와 반환 값으로 람다 사용

### 8장 내용
* 함수 타입
* 고차 함수 사용 방법
* 인라인 함수
* 비로컬 return과 레이블
* 무명 함수 

## 시작하며
앞 5장에서는 람다의 개념과 사용 함수에 대해서 설명 했습니다. 하지만 이번 8장에서는 람다를 인자로 받거나 반환하는 함수인 고차 함수를 만드는 방법에 대해 다룰 예정입니다.
고차  함수를 사용하면 **코드의 중복을 없애고 더 나은 추상화**를 구착할 수 있습니다.
또한 람다를 사용함에 따라 발생할 수 있는 **성능상 부가 비용을 없애고 유연하게 흐름을 제어할 수 있는 코틀린 특성인 인라인 함수**가 있습니다. 

## 고차 함수 정의
고차 함수란 **다른 함수를 인자로 받거나 함수를 반환하는 함수**입니다. 코틀린에서는 람다나 함수 참조를 사용해 함수를 값으로 표현할 수 있고 따라서 고차 함수는 람다나 함수 참조를 인자로 넘길 수 있거다 람다난 함수 참조를 반환하는 함수 입니다. 
<br>예를 들면 ```list.filter {x > 0 }``` 이 라이브러리 함수인 filter는 함수를 인자로 받으므로 고차 함수라 할 수 있습니다. 그리고 고차 함수를 좀 더 정의 하기 위해서는 함수 타입에 대해서도 알아야 합니다.<br/>
## 함수 타입이란
람다를 인자로 받는 함수를 정희하려면 람다 인자의 타입을 어떻게 선언할 수 있는지 알아야 합니다. 
간단하게 람다를 로컬 변수에 대하는 경우는 밑에 보이는 예제가 있습니다. 
이 경우는 sum 과 action의 함수 타입임을 추론합니다.
``` 
val sum = { x: Int, y : Int -> x + y }
val action = { println (42) }
```
하지만 여기서 각 변수에 구체적인 타입 선언을 추가할 수 있습니다. 
함수 타입을 정의 하려면 함수 파라미터의 괄호 안에 넣고, 그 뒤에 화살표를 추가한 다음 함수의 반환 타입을 지정하면 됩니다. 
``` 
val sum: (Int, Int) -> Int = { x: Int, y : Int -> x + y }
val action: () -> Unit = { println (42) }
```
이 코드에서 Unit 타입은 의미 있는 값을 반환하지 않는 함수 반환 타입에 쓰는 특별한 타입입니다.<br>그냥 함수를 정의한다면 Unit 반환 타입 지저을 생략해도 되지만, 함수 타입을 선언할 때는 반환 타입을 반드시 명시해야 하므로 Unit을 빼먹으면 안됩니다. 이렇게 변수 타입을 함수 타입으로 지정을 하게 된다면 함수 타입에 있는 파라미터로부터 람다의 파라미터 타입을 유추할 수 있게 됩니다. 따라서 람다 식 안에서 굳이 파라미터 타입을 적을 필요는 없습니다. <br/>
**널 또한 마찬가지로 지정할 수 있습니다**
``` var canReturnNull: (Int, Int) -> Int? = { x, y -> null }```

## 인자로 받은 함수 호출 
고차 함수의 인자로 받는 함수를 호출하는 구문은 일반 함수를 호출하는 구문과 같습니다. 문법은 함수 일므 뒤에 괄호를 붙이고 괄호 안에 원하는 인자를 콤마로 구분하는 것입니다. 
<br>
``` fun String.filter (prerdicate: (Char) -> Boolean): String ```
이 예제를 보면 filter 함수는 술어를 파라미터로 받고 predicate 파라미터는 문자를 파라미터로 받고 불리언 결과 값을 반환하는 형식입니다. 
<br/>
**술어**는 인자로 받은 문자가 filter 함수가 돌려주는 결과 문자열에 남아 있기를 바라면 true를 반환하고 문자열에서 사라지기를 바라면 false를 반환하는 것입니다.

## 디폴트 값을 지정한 함수 타입 파라미터나 널이 될 수 있는 함수 타입 파라미터
파라미터를 함수 타입으로 선언할 때도 디폴트 값을 정할 수 있습니다. 이전까지는 항상 객체를 toString 메소드를 통해 문자열로 바꿨습니다. toString으로 충분한 경우도 많지만 그렇지 않은 상황도 발생합니다. 이러한 상황에서는 원소를 문자열로 바꾸는 방법을 람다로 전달하면 됩니다 하지만 호출할 때마다 매번 람다를 넘기게 만들면 기본 동작으로도 충분한 대부분의 경우 함수 호출을 오히려 불편하게 만든다는 문제가 있습니다. 이러한 방법은 함수 타입의 파라미터에 대한 디폴트 값을 지정하면 이러한 문제를 해결할 수 있습니다. 
<br>
```
fun <T> Collection<T>.joiinToString(
  seperator: String = ",",
  prefix : String = "",
  postfix : String = "",
  transform: <T> -> String = { it.toString() }
) : String {
  val result = StringBUilder (prefix)
  for ((index, element) in this. withIndex()) {
    if (index > 0) result.append(separator)
    result.append(transform(element))
  }
  
  result.appen(postfix)
  return result.toString()
}
val letters = listOf("Alph", "Beta")

결과 
prinln(letters.joinToString())
Alpha, Beta
```

## 함수를 함수에서 반환
함수가 함수를 반환할 필요가 있는 경우보다는 함수가 함수를 인자로 받아야 할 필요가 있는 경우가 훨씬 많습니다. 하지만 함수를 반환하는 함수도 여전히 유용합니다. 프로그램의 상태나 다른 조건에 따라 달라질 수 있는 로직이 있다고 생각해보면 예를 들어 선택한 배송 수단에 따라 배송비를 계산하는 방법이 달라질 수 있습니다. 이럴 때 적절한 로직을 선택해서 함수로 반환하는 함수를 정의해 사용할 수 있습니다. 

```
enum class Delivery { STANDARD, EXPEDITED }
class Order (val itemCount: Int)
fun getShippingCostCalculator ( delivery  : Delivery) : (Order) -> Double {
  if (delivery == Delivery.EXPENITED) {
    return { order -> 6 + 2.1 * order.itemCount }
  }
  return { order -> 1.2*order.intemcount }
}
val calculator = getShippingCostCalculator(Delivery.EXPENITED)

println("$calulator(Order3)원)
>>> 12.3원
```
## 람다를 활용한 중복 제거
함수 타입과 람다 식은 재활용하기 좋은 코드를 만들 때 쓸 수 있는 훌륭한 도구입니다. 람다를 사용할 수 없는 환경에서는 아주 복잡한 구조를 만들어야만 피할 수 있는 코드중복도 람다를 활용하면 간결하고 쉽게 제거할 수 있습니다. enum을 사용을 해 여러 정보를 구할수 있습니다.
``` val averageWindowsDuration = log
.filter { it.os == OS.WINDOWS }
.map(SiteVisit::duration)
.average()

println(averageWindowsDUration)
>>> 23.0
```
### 고차함수를 여기저기 활용하면 전통적인 루프와 조건문을 사용할 때보다 더 느려질수 있습니다 하지만 inline을 통해 람다의 성능 개선이 가능합니다

## 인라인 함수: 람다의 부가 비용 없애기










