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
코틀린에서 람다를 함수 인자로 넘기는 구문이 if 나 for와 일반 문장과 비슷합니다. 하지만 코드의 성능은 훨씬 느리게 작동할 수 있습니다. 람다가 변수를 포획하면 람다가 생성되는 시점마다 새로운 무명 클래스 객체가 생기기 때문에 람다를 사용하는 구현은 똑같은 작업을 수행하는 일반 함수를 사용한 구현보다 덜 효율적 입니다.
그래서 반복되는 코드를 별도의 라이브러리 함수로 뺴내되 컴파일러가 자바의 일반 명령문만큼의 효율적인 코드를 생성할 수 있습니다. inline 변경자를 붙이면 그 함수를 호출하는 모든 문장을 함수 본문에 해당하는 바이트코드로 바꿔치기 해주빈다 이 과정은 부가 비용을 없앨수 있습니다.
## 인라이닝이 작동하는 방식
어떤 함수를 inline으로 선언하면그 함수의 본문이 인라인이 됩니다. 
<br>synchronized를 inline 으로 선언했으므로 synchronized를 호출하는 코들는 모두 자바의 synchronized문과 같아지게 됩니다. <br/>
```
inline fun <T> synchronized (lock : Lock, action : () -> T) : T {
  lock.lock()
  try {
    return action()
  }
  finally {
    lock.unlock()
  }
}
val l = Lock)
synchronized(l) {}
```
## 인라인 함수의 한계
인라이닝을 하는 방식으로 인해 람다를 사용하는 모든 함수를 인라이닝할 수는 없습니다. 함수가 인라이닝될 때 그 함수에 인자로 전달되 람다 식의 본문은 결과 코드에 직접 들어갈 수 있습니다 하지만 이렇게 람다가 본문에 직접 펼쳐지기 떄문에 함수가 파라미터로 전달받은 람다를 본문에 사용하는 방식이 한정될 수밖에 없습니다. 이미 받은 라맏를 다른 변수에 저장한다면 람다를 표현하는 객체가 어딘가는 존재해야 하기 때문에 람다를 인라이닝할 수 없습니다. **하지만 본문에서 람다 식을 바로 호출하거나 람다 식을 인자로 전달받아 바로 호출하는 경우에는 그 람다를 인라이닝할 수 있습니다.
 
## 컬렌션 연산 인라이닝
코틀린 표준 라이브러리 컬렉션 함수는 대부분 람다를 인자로 받습니다. 여기서 드는 생각은 표준 라이브러리 함수를 사용하지 않고 직접 이런 연산을 구현한다면 더 효율적이지 않을까 입니다

**람다를 사용해 컬렉션 거르기**
```
data class Persone(val name: String, val age: Int)
val people = listOf(Person("Alice", 29), Person("Bob", 31))

prinln(people.filter {it.age < 30 })
>>> Person(name : "Alice",age : 29)
```
**람다 X 컬렉션을 직접 거르기**
```
val result = mutableListOf<Person>()
for (person in people) {
  if (person.age < 30) result.add(person)
}
prinln(result)
>>> Person(name : "Alice",age : 29)
```
filter는 인라인 함수입니다 그래서 앞 예제에서 filter를 써서 생긴 바이트코드와 뒤 예제에서 생긴 바이트코드는 거의 같습니다. 

## 함수의 인라인으로 선언해야 하는 경우
inline 키워드를 배우고 나면 더 빠르게 만들기 위해 inline을 많이 사용할것이지만 무분별하게 사용을 하는것은 좋은 방법이 아닙니다. inline은 람다를 인자로 받는 함수에만 성능이 좋아질 가능성이 높습니다.

## 자원 관리를 위해 인라인된 람다 사용
람다로 중복을 없앨 수 있는 일반적인 패턴 중 한가지는 어떤 작업을 하기 전에 자원을 획득하고 작업을 마친 후 자원을 해제하는 자원 관리입니다. 여기서 자원은 파일, 락데이터베이스 트랜잭션 등 여러 다른 대상을 가리킬 수 있습니다. 자원 관리 패턴을 만들때 보통 try/finally 문을 사용합니다. 이전에는 try/finally 문의 로직을 함수로 캡슐화 하고 자우너을 사용하는 코드를 람다식으로 그 함수에 전달하는 예제를 본적이 있습니다. 그 예제에는 자바의 synchronized문과 똑같이 구문을 제공하는 synchronized함수가 있었습니다 하지만 코틀린 라이브러리에는 좀 더 코틀린 다운 api를 통해 같은 기능을 제공하는 wihLock이라는 함수가 있습니다 withLock은 Lock 인터페이스의 확장 함수입니다
<br>**withLock 사용법**<br/>
```
val l: Lock = ...
l.withLock {
  //락에 의해 보호되는 자원
}
```
## 고차 함수 안에서 흐름 제어
루프와 같은 명령형 코드를 람다로 바꾸기 시작했다면 return 문제에 부딪힐 겁니다. 루프의 중간에 있는 return문의 의미는 이해하기 쉽지만 그 루프를 filter와 같이 라맏를 호출하는 함수로 바꾸고 인자로 전달하는 람다 안에서 return을 사용할 수 있습니다.

### 람다 안의 return
일반 루트 안에서 return 사용
```
data class Preson(val name: String, val age: Int) 
val people = listOf(Person("Alice",29), Person("Bob", 31)
fun lookForAlice(people: List<Person>) {
  for (person in people) {
    if (person.name == "Alice") {
      println("Found!")
      return
    }
  }
  println("Alice is not found ")
}
lookForAlice(people)
>>> Found!
```
람다 안에서 return을 사용하면 람다로부터만 반환되는 게 아니라 그 람다를 호출하는 함수가 실행을 끝내고 반환됩니다. 

### 람다로부터 반환 : 레이블을 사용한 return
람다 식에서도 로컬 return을 사용할 수 있습니다. 람다 안에서 로컬 return은 for루프의 break와 비슷한 역할을 합니다. 로컬 return은 람다의 실행을 끝내고 람다를 호출했던 코드의 실행을 계속 이어갑니다. 로컬 return과 넌로컬 return을 구분하기 우해 레이블을 사용해야 합니다. return으로 실행을 끝내고 싶은 람다 식 앞에 레이블을 붙이고, return 키워드 뒤에 그 레이블을 추가하면 됩니다.
<br>람다 식에 레이블을 붙이려면 레이블 이름 뒤에 @ 문자를 추가한 것을 람다를 여는 { 앞에 넣으면 됩니다. 람다에 레이블을 붙여서 사용하는 대신 람다를 인자로 받는 인라인 함수의 이름을 return 뒤에 레이블로 사용해도 됩니다<br/>

### 무명 함수 : 기본적으로 로컬 return
무명 함수는 코드 블록을 함수에 넘길 때 사용할 수 있는 다른 방법입니다.
```
fun lookForAlice(people: List<Person>) {
  people.forEach(fun (person) {
    if (person.name == "Alice") return
    prinln("${person.name} is not Alice")
  }
}
```
무명 함수는 일반 함수와 비슷해 보이지만 차이는 이름이나 파라미터 타입을 생략할 수 있다는 점 뿐입니다.
