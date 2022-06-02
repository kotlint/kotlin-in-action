# 제네릭스

---

### 9장에서 배우는 내용

- 제네릭 함수와 클래스를 정의하는 방법
- 타입 소거와 실체화한 타입 파라미터
- 선언 지점과 사용 지점 변성

---

### 9.1 제네릭 타입 파라미터

제네릭스를 사용하면 받는 타입을 정의할 수 있음.

제네릭 타입의 인스턴스를 만드려면 타입 파라미터를 구체적인 `타입 인자`로 치환해야함.

```kotlin
List<String>
```

타입을 인스턴스화 하는 방법

```kotlin
Map<K, V>를 Map<String, Person>처럼 구체적인 타입을 타입 인자로 넘기면 됨
```

코틀린 컴파일러는 타입인자도 추론 가능하다

자바와 달리 코틀린은 제네릭 타입의 **타입 인자를 프로그래머가 명시하거나 컴파일러가 추론할 수 있어야한다**. 이유는 자바는 맨처음에 제네릭 지원이 없었고 자바 1.5에서 뒤늦게 생겨 이전버전과 호환성을 유지하기 위해 타입 인자가 없는 제네릭 타입(`raw 타입`)을 허용한다. 그러나 코틀린은 처음부터 제네릭을 도입했기때문에 `raw타입`을 지원하지 않고 제네릭 타입의 타입인자를 항상 정의해야 한다.

```kotlin
int list = new List<>() //자바는 가능하지만 코틀린은 안됨
var list = List<String>() //항상 타입 인자를 정의해야함
```

---

### 9.1.1 제네릭 함수와 프로퍼티

제네릭 함수를 호출할 때는 반드시 구체적 타입으로 타입 인자를 넘겨야 한다.

```kotlin
fun <T> List<T>.slice(indices: IntRange): List<T>
    (1)     (2)                               (3)
(1) : 타입 파라미터 선언
(2), (3) : 타입 파라미터가 수신 객체와 반환 타입에 쓰임
```

이런 함수를 구체적인 리스트에 대해 호출할 때 타입 인자를 명시적으로 지정할 수 있다.

하지만 실제로는 대부분 컴파일러가 타입 인자를 추론할 수 있으므로 그럴필요가 없다.

```kotlin
val letters = ('a'..'z').toList()
println(letters.slice<Char>(0..2))
>>>[a,b,c]

println(letters.slice(0..2))
>>>[a,b,c]
```

- 제네릭 고차 함수 호출하기

```kotlin
val authors = listOf("Dmitry", "Svetlana")
val readers = mutableListOf<String>()

fun <T> List<T>.filter(predicate: (T) -> Boolean): List<T> -> filter함수 정의
>>>readers.filter{ it !in authors)

it의 타입은 T라는 제네릭 타입
컴파일러는 filter가 List<T>타입의 리스트에 의해서 호출될 수 있다는 사실과 filter 객체의
readers의 타입이 List<String>이라는 사실을 알고 T가 String이라는 사실을 추론한다.
```

- 제네릭 확장 프로퍼티 선언

```kotlin
val <T> List<T>.penultimate: T (1)
	get() = this[size - 2]
println(listOf(1, 2, 3, 4).penultimate) (2)
>>> 3

(1) : 모든 리스트 타입에 대해 이 제네릭 확장 프로퍼티를 사용할 수 있다.
(2) : 이 호출에서 타입 파라미터 T는 Int로 추론됨
```

**확장 프로퍼티만 제네릭하게 만들 수 있다**.

- 일반 프로퍼티는 타입 파라미터를 가질 수 없다. 클래스 프로퍼티에 여러 타입의 값을 저장할 수는 없으므로 제네릭한 일반 프로퍼티는 말이 되지 않는다. 일반 프로퍼티를 제네릭하게 정의하면 컴파일러가 다음과 같은 오류를 표시한다.

```kotlin
val <T> x: T = TODO()
>>> ERROR: type parameter of a property must be used in its receiver type
```

---

### 9.1.2 제네릭 클래스 선언

코틀린에서 타입 파라미터를 넣은 `<>`를 클래스 이름 뒤에 붙히면 클래스를 제네릭하게 만들 수 있다. 타입 파라미터를 이름 뒤에 붙히고 나면 클래스 본문 안에서 타입 파라미터를 다른 일반 타입처럼 사용할 수 있다.

```kotlin
interface List<T> { (1)
	operator fun get(index: Int): T (2)
}
(1) : List 인터페이스에 T라는 타입 파라미터를 정의한다
(2) : 인터페이스 안에서 T를 일반 타입처럼 사용할 수 있다
```

제네릭 클래스를 확장하는 클래스를 정의하려면 기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야 한다. 이때 구체적인 타입을 넘길 수도 있고 타입 파라미터로 받은 타입을 넘길 수도 있다.

```kotlin
class StringList: List<String>{ (1)
	override fun get(index: Int): String = ... 
}

class ArrayList<T>: List<T>{ (2)
	override fun get(index: Int): T = ...
}
(1) : 이 클래스는 구체적인 타입 인자로 String을 지정해 List를 구현한다.
(2) : ArrayList의 제네릭 타입 파라미터 T를 List의 타입 인자로 넘긴다.
```

- `StringList`클래스는 String타입의 원소만을 포함한다. 따라서 String을 기반 타입의 타입 인자로 지정한다. 하위 클래스에서 상위 클래스에 정의된 함수를 오버라이드하거나 사용하려면 타입 인자 `T`를 구체적 타입 String으로 치환해야 한다. 따라서 `StringList`에서는 `fun get(int): T`가 아니라 `fun get(Int): String`이라고 해야함
- ArrayList클래스는 자신만의 타입 `T`를 정의하면서 그 `T`를 기반 클래스의 타입 인자로 사용한다. 여기서 ArrayList<T>와 List<T>의 `T`는 같지 않다. ArrayList<T>의 `T`는 앞에서 본 List<T>와 전혀 다른 타입 파라미터며, 실제로는 `T`가 아니라 다른 이름을 사용해도 의미에는 아무 차이가 없다.

```kotlin
class ArrayList<K>: List<K>{
    override fun get(index: Int): K = ...
}
```

클래스가 자기 자신을 타입 인자로 참조할 수도 있다. Comparable 인터페이스를 구현하는 클래스가 이런 패턴의 예다. 비교 가능한 모든 값은 자신을 같은 타입의 다른 값과 비교하는 방법을 제공해야만 한다.

```kotlin
interface Comparable<T>{
	fun compareTo(other: T): Int
}

class String: Comparable<String>{
	override fun compareTo(other: String): Int = ...
}
```

---

### 9.1.3 타입 파라미터 제약

클래스나 함수에 사용할 수 있는 타입 인자를 제한하는 기능

어떤 타입을 제네릭 타입의 타입 파라미터에 대한 `상한`으로 지정하면 그 제네릭 타입을 인스턴스화할때 사용하는 타입 인자는 반드시 그 상한타입이거나 그 상한 타입의 하위 타입이어야 한다.

제약을 가하려면 타입 파라미터 이름 뒤에 콜론(:)을 표시하고 그 뒤에 상한 타입을 적으면 된다.

```kotlin
fun <T : Number> List<T>.sum(): T
	  (1)    (2)
(1) : 타입 파라미터
(2) : 상한
```

- 타입 파라미터 `T`에 대한 상한을 정하고 나면 `T`타입의 값을 그 상한 타입의 값으로 취급할 수 있다. 예를 들면 상한 타입에 정의된 메서드를 `T`타입 값에 대해 호출할 수 있다.

```kotlin
fun <T : Number> oneHalf(value: T): Double{ (1)
	return value.toDouble() / 2.0 (2)

println(oneHalf(3))
>>> 1.5

(1) : Number를 타입 파라미터 상한으로 정한다
(2) : Number 클래스에 정의된 메서드(toDouble())를 호출한다
```

- 두 파라미터 사이에서 큰 값을 찾는 제네릭 함수

```kotlin
fun <T : Comparable<T> max(first: T, second: T): T {
	return if(first > second) first else second
}

println(max("kotlin", "java"))
>>> kotlin
```

- 아주 드물지만 타입 파라미터에 대해 둘 이상의 제약을 가해야 하는 경우도 있다. 이럴때는 약간 다른 구문을 사용한다.

```kotlin
fun <T> ensureTrailingPeriod(seq: T) where T : CharSequence, T : Appendable { (1)
	if(!seq.endWith('.')){ (2)
		seq.append('.') (3)
	}
}

val helloworld = StringBuilder("Hello World")
ensureTrailingPeriod(helloworld)
println(helloworld)
>>> Hello World.

(1) : 타입 파라미터 제약 목록
(2) : CharSequence 인터페이스의 확장 함수를 호출한다.
(3) : Appendable 인터페이스의 메서드를 호출한다
```

---

### 9.1.4 타입 파라미터를 널이 될 수 없는 타입으로 한정

제네릭 클래스나 함수를 정의하고 그 타입을 인스턴스화 할때는 널이 될 수 있는 타입을 포함하는 어떤 타입으로 타입 인자를 지정해도 타입 파라미터를 치환할 수 있다. 아무런 상한을 정하지 않은 타입 파라미터는 결과적으로 `Any?`를 상한으로 정한 파라미터와 같다.

```kotlin
class Processor<T>{
	fun process(value: T){
		value?.hashCode() -> value는 null이 될 수 있으므로 안전한 호출을 사용해야 함
	}
}
```

- 항상 널이 될 수 **없는** 타입만 타입 인자로 받게하는 방법

```kotlin
class Processor<T: Any>{ (1)
	fun process(value: T){
		value.hashCode() (2)
	}
}
(1) : null이 될 수 없는 타입 상한을 지정
(2) : T 타입의 value는 null이 될 수 없다
```

---

### 9.2 실행 시 제네릭스 동작 : 소거된 타입 파라미터와 실체화된 타입 파라미터

JVM에서의 제네릭스는 보통 타입 소거를 사용해 구현된다. 이는 실행 시점에 제네릭 클래스의 인스턴스에 타입 인자 정보가 들어있지 않다는 뜻이다.

코틀린에서는 타입 소거를 하지 않기 위해 함수를 inline으로 선언함으로써 타입 소거를 우회하는 것을 실체화 라고한다.

---

### 9.2.1 실행 시점의 제네릭 : 타입 검사와 캐스트

코틀린 제네릭 타입 인자 정보는 실행시점에 지워진다. 이는 제네릭 클래스 인스턴스가 그 인스턴스를 생성할 때 쓰인 타입 인자에 대한 정보를 유지하지 않는다는 뜻이다.

```kotlin
val list1: List<String> = listOf("a","b")
val list2: List<Int> = listOf(1,2,3)

실행 시점에 list1이나 list2가 문자열이나 정수형 리스트로 선언됬다는 사실을 알 수 없다.
각 객체는 list일 뿐이다.

우리가 list1에는 문자열만, list2에는 정수형만 들어있다고 가정할 수 있는 이유는 컴파일러가 
타입 인자를 알고 올바른 타입의 값만 각 리스트에 넣도록 보장하기 때문이다.
```

타입 소거로 인한 한계점 : 실행 시점에 타입 인자를 검사할 수 없다.

```kotlin
if(value is List<String>){...}
>>> ERROR: Cannot check for instance of erased type
```

코틀린에서는 타입 인자를 명시하지 않고 제네릭 타입을 사용할 수 없다. 이때 어떤 값이 맵이 아니라 리스트라는 사실을 어떻게 확인할 수 있는 방법은 스타 프로젝션을 사용하면 된다.

```kotlin
if(value is List<*>){...}

타입 파라미터가 2개 이상이라면 모든 파라미터에 * 을 포합해야 한다.
```

is 처럼 as나 as? 캐스팅에도 제네릭 타입을 사용할 수 있다. 

**하지만 실행 시점에는 제네릭 타입의 타입 인자를 알 수 없으므로 캐스팅은 항상 성공한다**.

```kotlin
fun printSum(c: Collection<*>){
	val intList = c as? List<Int> (1)
		?: throw IllegalArgumentException("List is expected")
	println(intList.sum())
}
println(listOf(1,2,3)) (2)
>>> 6

(1) : Unchecked cast: List<*> to List<Int> 경고 발생, 타입인자를 알 수 없기 때문
(2) : 경고는 발생하지만 예상대로 작동함

*만약 정수 리스트가 아닌 정수 집합을 넣는 경우 에는 IllegalArgumentException이 발생한다.
*잘못된 타입의 원소가 들어있는 리스트를 전달한다면 실행 시점에 ClassCastException이 발생한다.

코틀린 컴파일러는 컴파일 시점에 타입 정보가 주어진 경우에는 is 검사를 허용한다.

fun printSum(c: Collection<Int>){
	if(c is List<Int>){ -> 올바른 검사
		println(c.sum())
	}
}
```

---

### 9.2.2 실체화한 타입 파라미터를 사용한 함수 선언

코틀린의 제네릭 타입의 타입 인자 정보는 실행시점에 지워진다. 따라서 제네릭 함수가 호출되도 그 함수의 본문에서는 호출 시 쓰인 타입 인자를 알 수 없다.

```kotlin
fun <T> isA(value: Any) = value is T

println(isA<String>("abc"))
>>> ERROR: Cannot check for instance of erased type: T
```

이러한 제약을 피하기위해 인라인 함수를 사용한다. 

```kotlin
inline fun <reified T> isA(value: Any) = value is T

println(isA<String>("abc"))
>>> true

* reified 
런타임에서 제네릭스 타입정보를 알기위해 사용함

만약 inline 함수에서 reified가 없다면 명시적으로 파라미터를 전달해야함
```

실체화한 타입 파라미터를 사용하는 예제 중 가장 간단한 filterIsInstance의 사용법을 알아보자 

filterIsInstance는 인자로 받은 클래스의 원소 중에서 타입 인자로 지정한 클래스의 인스턴스만 모아서 만든 리스트를 반환하는 함수이다.

```kotlin
val items = listOf("one", 2, "three")
println(items.filterIsInstance<String>())

>>> [one, three]

filterIsInstance의 타입인자로 String을 지정함으로써 반환되는 리스트는 
List<String> 이라는 것을 알 수 있다. 여기서는 타입 인자를 실행 시점에 알 수 있고
filterIsInstance는 그 타입 인자를 이용해 리스트의 원소 중에 타입 인자와 타입이 일치하는
원소만을 추려낼 수 있다.

코틀린 표준 라이브러리에 있는 filterIsInstance 선언
inline fun <reified T> Iterable<*>.filterIsInstance(): List<T>{ (1)
	val destination = mutableListOf<T>()
	for(element in this){
		if(element is T){ (2)
			detination.add(element)
		}
	}
	return destination 
}

(1) : "reified" 키워드는 이 타입 파라미터가 실행 시점에 지워지지 않음을 표시한다
(2) : 각 원소가 타입 인자로 지정한 클래스의 인스턴스인지 검사할 수 있다

* 인라인 함수에서만 실체화한 타입 인자를 쓸 수 있는 이유 :
컴파일러가 인라인 함수의 본문을 구현한 바이트코드를 그 함수가 호출되는 모든 지점에 삽입한다
컴파일러는 실체화한 타입 인자를 사용해 인라인 함수를 호출하는 각 부분의 정확한 타입인자를
알 수 있다. 따라서 컴파일러는 타입 인자로 쓰인 구체적인 클래스를 참조하는 바이트코드를
생성해 삽입할 수 있다.
```

---

### 9.2.3 실체화한 타입 파라미터로 클래스 참조를 대신

java.lang.Class 타입 인자를 파라미터로 받는 경우 실체화한 타입 파라미터를 자주 사용한다.

java.lang.Class를 사용하는 API의 예로는 JDK의 ServiceLoader가 있다. ServiceLoader는 어떤 추상 클래스나 인터페이스를 표현하는 java.lang.Class를 받아서 그 클래스나 인스턴스를 구현한 인스턴스를 반환한다.

```kotlin
val serviceImpl = ServiceLoader.load(Service::class.java)

::class.java 구문은 코틀린 클래스에 대응하는 java.lang.Class 참조를 얻는 방법을 보여준다.
Service::class.java라는 코드는 Service.class라는 자바 코드와 완전히 같다.

이 예제를 구체화하면
val serviceImpl = loadService<Services>()

이제는 읽어 들일 서비스 클래스를 loadService 함수의 타입 인자로 지정한다.
loadService 함수를 어떻게 정의할 수 있을까

inline fun <reified T> loadService(){
	return ServiceLoader.load(T::class.java)
}

비슷한 예로 안드로이드에서 activity를 이동할때 실체화한 타입 파라미터를 사용할 수 있다.

inline fun <reified T : Activity> Context.startActivity(){
	val intent = Intent(this, T::class.java)
	startActivity(intent)
}

startActivity<DetailActivity>()
```

---

### 9.2.4 실체화한 타입 파라미터의 제약

실체화한 타입 파라미터를 사용 가능한 경우

- 타입 검사와 캐스팅 `is` `as` `!is` `!as`
- 코틀린 리플렉션 API `::class`
- 코틀린 타입에 대응하는 java.lang.Class를 얻기 `::class.java`
- 다른 함수를 호출할 때 타입 인자로 사용

사용불가능한 경우

- 타입 파라미터 클래스의 인스턴스 생성
- 타입 파라미터 클래스의 동반 객체 메서드 호출
- 실체화한 타입 파라미터를 요구하는 함수를 호출하면서 실체화하지 않은 타입 파라미터로 받은 타입을 타입인자로 넘기기
- 클래스, 프로퍼티, 인라인 함수가 아닌 함수의 타입 파라미터를 reified로 지정하기

---

### 9.3 변성 : 제네릭과 하위 타입

변성 - `List<String>`와 `List<Any>`와 같이 기저 타입이 같고 타입 인자가 다른 여러 타입이 서로 어떤 관계가 있는지 설명하는 개념

제네릭 클래스나 함수를 직접 정의하려면 변성을 꼭 이해해야 하며 변성을 잘 활용하면 사용에 불편하지 않으면서 타입 안전성을 보장하는 API를 만들 수 있다.

---

### 9.3.1 변성이 있는 이유 : 인자를 함수에 넘기기

`List<Any>` 타입의 파라미터를 받는 함수에 `List<String>` 인자를 넘기면 안전할까? → 안전하다 

이유는 `String`클래스는 `Any`를 확장하므로 `Any`타입 값을 파라미터로 받는 함수에 `String` 값을 넘겨도 안전하기 때문이다.

그렇다면 `Any`와 `String`이 `List` 인터페이스의 타입 인자로 들어가는 경우에도 안전할까? → 아니다

```kotlin
fun addAnswer(list: MutableList<Any>){
	list.add(42)
}

이 함수에 문자열 리스트를 넘겨보자
val strings = mutableListOf("abc", "bac")
addAnswer(strings)
println(strings.maxBy{ it.length })

>>> ClassCastException: Integer cannot be cast to String

아예 컴파일 되지 않음
```

결론 : 어떤 함수가 리스트의 원소를 추가하거나 변경한다면 타입 불일치가 생길 수 있어서 `List<Any>` 대신 `List<String>`을 넘길 수 없다. 그러나 **원소 추가나 변경이 없는 경우**에는 `List<String>`을 `List<Any>` 대신 넘겨도 안전하다. 따라서 코틀린에서는 리스트의 변경 가능성에 따라 적절한 인터페이스를 선택하면 안전하지 못한 함수 호출을 막을 수 있다.

---

### 9.3.2 클래스, 타입, 하위 타입

타입

- 제네릭 클래스가 아닌 클래스 에서는 클래스 이름을 바로 타입으로 사용가능한데 이때 널이 될 수 있는 타입도 사용가능함 `var x : String` , `var x : String?`
- 제네릭 클래스의 경우에는 타입 파라미터를 구체적인 타입 인자로 바꿔줘야함 `List<Int>`, `List<String>`… 각각의 제네릭 클래스에는 무수히 많은 타입을 만들어 낼 수 있다.

하위 타입 : 어떤 타입 A의 값이 필요한 모든 장소에 어떤 타입 B값을 넣어도 문제가 없다면 

B는 A의 하위 타입이다.

 

```kotlin
예를 들어 Int는 Number의 하위 타입이지만 String의 하위 타입은 아니다.
```

상위 타입은 하위 타입의 반대이다.

그렇다면 어떤 타입이 하위 타입인지 왜 중요할까? 이유는 컴파일러가 변수 대입이나, 함수 인자 전달 시 하위 타입 검사를 매번 수행하기 때문이다.

```kotlin
fun test(i: Int){
	val n: Number = i (1)
	
	fun f(s: String){...}
	f(i) (2)
}

(1) : Int가 Number의 하위 타입이어서 컴파일 된다.
(2) : Int가 String의 하위 타입이 아니라서 컴파일 되지 않는다.
```

간단한 경우 하위 타입은 하위 클래스와 근본적으로 같다. 예를 들면 Int 클래스는 Number 클래스의 하위 클래스 이므로 Int 는 Number의 하위 타입이다. String은 CharSequence의 하위 타입인 것처럼 어떤 인터페이스를 구현하는 클래스의 타입은 그 인터페이스 타입의 하위 타입이다.

제네릭 타입에 대해 이야기할 때 특히 하위 클래스와 하위 타입의 차이가 중요해진다.

List<String> 타입의 값을 List<Any>를 파라미터로 받는 함수에 전달해도 괜찮은가 `=` 

List<String>은 List<Any>의 하위 타입인가

제네릭 타입을 인스턴스화할 때 타입 인자로 서로 다른 타입이 들어가면 인스턴스타입 사이에 하위 타입 관계가 성립하지 않으면 그 제네릭 타입을 `무공변` 이라고 한다. 

예를 들어 MutableList A와 B가 서로 다르기만 하면 A는 B의 하위 타입이 아니다. 

A가 B의 하위 타입이면 List<A>는 List<B>의 하위 타입이다. → `공변적`

---

### 9.3.3 공변성 : 하위 타입 관계를 유지

Producer<T>를 예로 공변성 클래스를 설명하면, A가 B의 하위 타입일때 Producer<A>가

Producer<B>의 하위타입이면 Producer는 `공변적`이다. 이를 하위 타입 관계가 유지된다고 말한다.

예를 들어 Cat이 Animal의 하위 타입이기 때문에 Producer<Cat>은 Producer<Animal>의 하위 타입이라고 할 수 있다.

코틀린에서 제네릭 클래스가 타입 파라미터에 대해 `공변적`임을 표시하려면 타입 파라미터 앞에 `out` 을 붙혀야 한다.

```kotlin
interface Producer<out T>{ -> 클래스 T에 대해 공변적이라고 선언
	fun produce(): T
}
```

클래스의 타입 파라미터를 공변적으로 만들면 함수 정의에 사용한 파라미터 타입과 타입 인자의 타입이 정확히 일치하지 않더라도 그 클래스의 인스턴스를 함수 인자나 반환 값으로 사용할 수 있다.

```kotlin
open class Animal{
	fun feed(){...}
}

class Herd<T: Animal>{ -> 무공변성으로 지정
	val size: Int get() = ...
	operator fun get(i: Int): T {...}
}

fun feedAll(animals: Herd<Animal>){
	for(i in 0 until animals.size){
		animals[i].feed()
	}
}

Herd클래스의 타입 파라미터는 그 동물이 어떤 동물무리인지 알려준다.

사용자 코드가 고양이 무리를 만들어서 관리한다고 하자

class Cat: Animal(){
	fun cleanLitter() {...}
}

fun takeCareOfCats(cats: Herd<Cat>){
	for(i in 0 until cats.size){
		cats[i].cleanLitter()
		//feedAll(cats) -> inferred type is Herd(Cat), but Herd(Animal) was expected 오류 발생
	}
}
```

오류가 나는 이유는 Herd 클래스의 T 타입 파라미터에 아무 변성도 지정하지 않았기 때문에 고양이는 동물 무리의 하위 클래스가 아니라서 오류가 나는 것이다.

위 오류를 해결하기 위해서는 class Herd<out T: Animal>로 Herd 클래스를 공변적으로 지정하면 된다.

모든 클래스를 공변적으로 만들 수는 없다. 타입 파라미터를 공변적으로 지정하면 클래스 내부에서 그 파라미터를 사용하는 방법을 제한한다. 타입 안전성을 보장하기 위해 공변적 파라미터는 항상

out 위치에 있어야한다. 이는 클래스가 T 타입의 값을 생성할 수는 있지만 T 타입의 값을 사용할 수는 없다는 뜻이다.

클래스 멤버를 선언할 때 타입 파라미터를 사용할 수 있는 지점은 모두 `in` 과 `out` 위치로 나뉜다. 

T 라는 타입 파라미터를 선언하고 T 를 사용하는 함수가 멤버로 있는 클래스를 생각하면 

T 가 함수의 `반환 타입`에 쓰인다면 T는 `out` 위치에 있다 → T 타입의 값을 `생성` 한다. 

T 가 함수의 `파라미터 타입`에 쓰인다면 T는 `in` 위치에 있다 → T 타입의 값을 `사용`한다.

```kotlin
interface TransFormer<T>{
	fun transform(t: T): T
								 (1)  (2)
(1) : in 위치
(2) : out 위치
```

클래스 타입 파라미터 T 앞에 `out` 키워드를 붙히면 클래스 안에서 T를 사용하는 메서드가 `out` 위치에서만 T를 사용가능하게 하고 `in` 위치에서는 T를 사용하지 못하도록 하는데 이는 T로 인해 생기는 타입 관계의 타입 안전성을 보장하기 때문이다.

Herd클래스를 살펴보면 T를 공변적으로 사용하기 때문에 `out` 위치에만 T를 사용하는것을 볼 수 있다.

따라서 Cat이 Animal의 하위 타입이기 때문에 Herd<Animal>의 get을 호출하는 모든 코드는 get이 Cat을 반환해도 문제없이 작동한다.

T에 붙은 `out` 키워드의 의미

- 공변성 : 하위 타입 관계가 유지됨
- 사용 제한 : T를 out위치(반환) 에서만 사용할 수 있다.

List<T> 인터페이스를 보면 T 타입의 원소를 반환하는 get 메서드는 있지만 T 타입의 값을 추가하거나 리스트의 기존값을 변경하는 메서드는 없다. 그러므로 List는 T에 대해 공변적이다.

```kotlin
interface List<out T>: Collection<T>{
	operator fun get(index: Int): T -> T가 반환 위치에 있으므로 공변적임
}
```

타입 파라미터를 함수의 파라미터 타입이나 반환 타입에만 쓸 수 있는것은 아니다. 타입 파라미터를 다른 타입의 타입 인자로 사용할 수도 있다.

```kotlin
interface List<out T>: Collection<T>{
	fun subList(fromIndex: Int, toIndex: Int): List<T> -> 반환 위치에 있으므로 공변적임
}
```
