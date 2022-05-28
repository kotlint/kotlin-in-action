# 람다로 프로그래밍

- 람다는 다른 함수에 넘길 수 있는 작은 코드 조각이다.

람다는 공통 코드 구조를 라이브러리 함수로 쉽게 뽑아낼 수 있다.(Kotlin 라이브러리는 람다를 아주 많이 사용한다.)

### 람다 소개 : 코드 블록을 함수 인자로 넘기기

- 익명 내부 클래스로 리스너 구현하기

~~~java
button.setOnClickListener (new OnClickListener() {
    @Override
    public coid onClick(View view){
        /*클릭시 수행할 동작*/
    }
});
~~~

- 람다로 리스너 구현

~~~kotlin
button.setOnClickListener {/*클릭시 수행할 동작*/}
~~~

### 람다와 컬렉션

- 컬렉션 직접 검색하기

~~~kotlin
data class Person(val name: String, val age: Int)

fun findTheOldest(people: List<Person>) {

    var maxAge = 0 // 가장 많은 나이를 저장
    var theOldest: Person? = null // 가장 연장자인 사람을 저장
    for (person in people) {
        if (person.age > maxAge) { // 현재까지 발견한 최 연장자보다 더 나이가 많은 사람을 찾으면 최대값을 바꾼다.
            maxAge = person.age
            theOldest = person
        }
    }
    println(theOldest)
}

fun main() {
    val person = listOf(Person("A", 29), Person("B", 31))
    findTheOldest(person)
}
~~~

- 람다를 사용해 컬렉션 검색하기
    - 위 코드처럼 findTheOldest() 메소드를 직접 구현하지 않아도, kotlin 에서는 라이브러리 함수를 사용하면 된다.

~~~kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val person = listOf(Person("A", 29), Person("B", 31))
    println(person.maxByOrNull { it.age }) // 나이 프로퍼티를 비교해서 값이 가장 큰 원소 찾기
}
~~~

### 번외, kotlin 에서의 배열 array, List(listOf), ArrayList(mutableListOf)

- Array

> Array 의 경우 배열의 크기가 정해져 있다는 특징이 있다.
> 기본적인 선언
>
> var ArrayName : Array<T> = Array<T>(배열의 크기){초기값}
>
> 기본적으로 아래와 같이 사용

~~~kotlin
var array = arrayOf(100, 200, 300)
~~~

- List

> listOf : 읽기 전용 List 변경이 불가하기 때문에 val 로 선언하는것이 좋겠다.

~~~kotlin
var list = listOf(100, 200, 300)
// list.add. x
// list.set x
var result = list.get(0)
//100
~~~

- ArrayList(mutableListOf)

> mutableListOf : 변경이 가능한 List (java 에서 ArrayList 와 동일 기능을 한다.)

~~~kotlin
var arrayList = arrayListOf<Int>()
arrayList.add(100)
arrayList.add(200)
println(arrayList)
//{100, 200}
~~~

### 람다식 문법

- 람다 식 문법

~~~kotlin

{ x: Int, y: Int -> x + y }

// x: Int, y: Int 파라미터 부분, x + y 본문
// 람다식은 항상 중 괄호 한에 위치한다.
~~~

- 변수에 지정된 람다 호출

~~~kotlin
val sum = { x: Int, y: Int -> x + y }

println(sym(1, 2))
// 결과 3
~~~

위에서 했던 예제 코드 줄이기.

~~~kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val person = listOf(Person("A", 29), Person("B", 31))
    println(person.maxByOrNull { it.age }) // 나이 프로퍼티를 비교해서 값이 가장 큰 원소 찾기
}
~~~

위의 코드에서 {it.age} 부분을 정식으로 람다를 작성하면 다음과 같다.

~~~kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val person = listOf(Person("A", 29), Person("B", 31))
    person.maxByOrNull { p: Person -> p.age } // 파라미터 타입을 명시
    person.maxByOrNull { p -> p.age } // 컴파일 타입 추론
    // 람다의 파라미터가 하나뿐이고 그 타입을 컴파일러가 추론할 수 있는 경우 it을 바로 쓸 수 있다.
    person.maxByOrNull { it.age } // it은 자동으로 생성된 파라미터 이름이다.
}
~~~

**!!** 위와같이 it 을 사용하는것은 좋은 습관이다(Why? 코드를 아주 간단하게 만들어 주기 떄문이다.), 하지마느 남용하면 안된다. **특히** 람다안에 중첩되는 경우 각 람다의 파라미터를 명시하는 편이
좋다(Why? 가독성 때문).

- 람다를 변수에 저장할 떄는 파라미터의 타입을 추론할 문맥이 존재하지 않는다. 따라서 파라미터 **타입을 명시** 해야한다.

~~~kotlin
data class Person(val name: String, val age: Int)

fun main() {
    val getAge = { p: Person -> p.age } // 타입을 명시해야한다.
    person.maxByOrNull { getAge }
}
~~~

- 본문이 여러줄인 람다 식. ( 마지막에 있는 식이 람다의 결과 값)

~~~kotlin
fun main() {
    val sum = { x: Int, y: Int ->
        println("$x + $y")
        x + y
    }
    println(sum(1, 2))
}

// 3
~~~

### 현재 영역에 있는 변수에 접근

- 람다를 함수 안에서 정의하면, 함수의 파라미터 뿐 아니라 람다의 정의의 앞에 선언된 로컬 변수까지 람다에서 모두 사용할 수 있다.
- forEach 는 컬렉션의 모든 원소에 대해 람다를 호출해 준다.

~~~kotlin
// 함수 파라미터를 람다 안에서 사용하기.
fun printMessagesWithPrefix(message: Collection<String>, prefix: String) {
    message.forEach { // 각 원소에 대해 수행할 작업을 람다로 받는다.
        println("$prefix $it") // 람다 안에서 함수의 'prefix' 파라미터를 사용
    }
}
fun main() {
    val error = listOf("403 Forbidden", "404 Not Found")
    printMessagesWithPrefix(error, "Error: ")
}

// 결과
// Error: 403 Forbidden
// Error: 404 Not Found
~~~

- 람다 안에서 바깥함수 로켤변수 포획

~~~kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientError = 0;
    var serverError = 0;
    responses.forEach {
        if (it.startsWith("4")) {
            clientError++
        } else if (it.startsWith("5")) {
            serverError++
        }
    }
}
~~~

### 멤버 참조

- :: 를 사용하는 식을 멤버 참조라고 부른다.
    - 멤버 참조는 프로퍼티나 메서드를 단 하나만 호출하는 함수 값을 만들어 준다.

~~~kotlin
val getAge = Person::age

// 기존 형태
val getAge = { person: Person -> person.age }
~~~

- Class 이름 생략하고 :: 참조 바로시작

~~~kotlin
// 기존
val action = { person: Person, message: String ->
    sendEmail(person, message)
} // sendEmail() 에 파라미터를 람다식으로 넘김

// 람대 대신 멤버 참조
val nextAction = ::sendEnaul
val mail = nextAction(Person, "message")
println(mail)

fun salute() = println("salute!");

fun main() {
    // 라이브러리 함수  
    run(::salute)

    // 출력 : "salute!
}
~~~

### 컬렉션 함수형 API

- #### 필수적인 함수 : filter 와 map
    - filter 와 map 은 컬렉션을 활용하때 기반이 되는 함수이다.

- filter() 함수는 컬렉션을 이터레이션하면서 주어진 람다에 각 원소를 넘겨서 함다가 true 를 반환하는 원소만 모은다.

~~~kotlin
val list = listOf(1, 2, 3, 4)
println(list.filter { it % 2 == 0 })

// 출력 " 2, 4
~~~  

- filter() 함수는 원치 않는 원소를 제거한다. 원도도 변환할 수 없다. 변환하려면 map() 함수를 써야한다.

~~~kotlin
val list = listOf(1, 2, 3, 4)
println(list.map { it * it })
// 결과 1, 4, 9, 16 원소를 변환했쥬? 
~~~

- 사람의 리스트가아니라 사람의 이름의 리스트를 출력하고싶다면? map!!

~~~kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("봉만식", 27), Person("국뽕만", 31))
println(people.map { it.name })
// 출력 봉만식, 국뽕만

// 참조 응용 people.map { Person::name }
println(people.filter { it.age > 30 }.map { Person::name })
// 국뽕만
~~~

### all, any, count, find : 컬렉션에 술어 적용

~~~kotlin
data class Person(val name: String, val age: Int)

val canBeInClub27 = { p: Person -> p.age <= 27 }

val people = listOf(Person("봉만식", 27), Person("국뽕만", 31))
// 모든 원소가 조건에 맞는기 검사
println(people.all(canBeInClub27))// 결과 false
// 만족하는 원소가 하나라도 있나?
println(people.any(canBeInClub27)) // true
// 만족하는 원소의 개수를 구하려면
println(people.count(canBeInClub27)) // 1
// 원소 하나만 찾고싶은 경우
println(people.find(canBeInClub27)) // Person(name=봉만식, age=27)
~~~

### groupBy : 리스트를 여러 그룹으로 이뤄진 맵으로 변경

- groupBy Type 는 특성을 파라미터로 전달하면 컬렉션이 자동으로 구분해준다.

~~~kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("봉만식", 27), Person("국뽕만", 31), Person("한석뽕", 31))

println(people.groupBy { it.age })

// { // 결과 (key 로 그룹 지워줌)
// 27=[Person(name=봉만식, age=27)], 
// 31=[Person(name=국뽕만, age=31), Person(name=한석뽕, age=31)]
// }
~~~

### flatMap 과 flatten : 중첨된 컬렉션 안의 원소 처리

~~~kotlin
class Book(val tile: String, val authors: List<String>)

val books = listOf(
    Book("숲속의 곰탱이", listOf("개미같은 니들")),
    Book("바다의 지느러미", listOf("하늘과 바다", "해저속 니네집")),
    Book("하늘의 기레기", listOf("하늘과 바다", "비행기타고 우리집"))
)
println(books.flatMap { it.authors }.toSet())
// [개미같은 니들, 하늘과 바다, 해저속 니네집, 비행기타고 우리집]
~~~

- toSet()은 flatMap 의 결과 리스트에서 중복을 없애고 집합으로 만든다.

- 리스트의 리스트가 있는데 모든 중첩된 리스트의 원소를 한 리스트로 모아야한다면 flatMap() 을 떠올리면 된다.
- 특별히 변환해야 할 내용이 없다면 리스트의 리스트를 평평하게 펼치기만 하면 된다. 그런 경우 flatten() 을 사용하면 된다.

~~~kotlin
val numbers = listOf(listOf(1, 2, 3), listOf(5, 6, 7), listOf(8, 9, 0))
val result = numbers.flatten()
println(result)
~~~

### 지연 계산(lazy) 컬렉션 연산

- 즉시 계산
    - 앞서 살펴본 map 이나 filter 같은 컬렉션은 결과 컬렉션을 즉시 생성한다.

~~~kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("봉만식", 27), Person("국뽕만", 31), Person("한석뽕", 31))

// 즉시계산
println(people.map(Person::name).filter { it.startsWith("한") })

// 결과 
// [한석뽕]
~~~

- filter 와 map 은 리스트를 **반환한다.** 위 프린터문에서 연쇄 호출이 리스트를 2개 만든다는 뜻이다.
    - 한 리스트는 filter 의 결과를 담고 다른 하나는 map 의 결과를 담는다. 원본리스트에 수백만개의 Data 가 들어있다면 효울이 많이 떨어진다.


- 지연계산
    - 시퀀스를 통해 중간 임시 컬렉션없이 컬렉션 연산을 수행한다.

~~~kotlin
data class Person(val name: String, val age: Int)

val people = listOf(Person("봉만식", 27), Person("국뽕만", 31), Person("한석뽕", 31))

// 지연계산
println(
    people.asSequence() // 원본 컬렉션을 시퀀스로 변환
    .map(Person::name)               // 시퀀스도 컬렉션과 똑같은 
    .filter { it.startsWith("한") }   // API를 제공
    .toList() // 결과 시퀀스를 가시 리스트로 변환
)
~~~

- asSequence() 는 컬렉션함수를 시퀀스로 변환해주는 함수이다.

> - 처리할 Data 가 많을경우 시퀀스를 이용해라.
>- 시퀀스로 처리하면 filter 와 map 처럼 연쇄 호출이 일어나지 않는다.

- 시퀀스는 중간 처리결과를 저장하지 않고도 연산을 열쇄적으로 적용해서 효율적으로 계산을 수행한다.

### 시퀀스 연산 실행 : 중간 연산과 최종 연산

- 시퀀스에 대한 연산은 중간연산과 최종연상으로 나뉜다.
    - 중간연산
        - 또 다른 시퀀스를 반환한다.
        - 항상 지연 계산이 수행된다.(최종연산이 수행되면)
    - 최종연산
        - 중산연산을 실제로 적용시켜 결과를 반환한다.

~~~kotlin
// 최종 연산이 없어서 아무런 동작도 수행하지 않는다
println(
listOf(1, 2, 3, 4).asSequence()
    .map { it * it }
    .filter { it % 2 == 0 }
)

println(
// .toList()라는 최종 연산이 있어서 비로소 이 때 map, filter동작도 수행
listOf(1, 2, 3, 4).asSequence()
    .map { it * it }
    .filter { it % 2 == 0 }
    .toList() // 최종 연산
)
// 결과
// [4, 16]
~~~

### 시퀀스 만들기
- Collection.asSequence() 함수
    - Collection => Sequence 로 변환
    

- generateSequence() 함수
    - 이 전의 원소를 인자로 받아서 다음 원소를 계산하는 시퀀스를 생성
    
~~~kotlin
val naturalNumbers = generateSequence(0) { it + 1 }
val numbersTo100 = naturalNumbers.takeWhile { it <= 100 }
// sum() 이라는 최종 연산을 통해 모든 원소를 더한결과를 반환
println(numbersTo100.sum())

// 결과
// 5050
~~~

### 자바 함수형 인터페이스 활용
- Java에서 특정 로직을 수행하기 위해서는 익명 클래스를 이용했다
  
  => Kotlin에서는 람다를 통해서 이를 대신할 수 있다. (이런 인터페이스를 함수형 인터페이스 or SAM 인터페이스)

- SAM 인터페이스
    - Single Abstract Method 의 약자
    - 추상 메소드가 단 하나만 있는 인터페이스
    
### 수신 객체 지정 람다: with 와 apply
- with 함수 : 어떤 객체의 이름을 반복허지 않고도 그 객체에 대해 다양한 연산을 수행할 수 있는 라이브러리

~~~kotlin
fun alphabet() : String{
  val result = StringBuilder()
  for(letter in 'A'..'Z'){
    result.append(letter) // result 중복
  }
  result.append("append!") // result 중복 
  return result.toString() // result 중복
}

// with() 사용 result 중복 제거
fun alphabet() : String{
  val stringBuilder = StringBuilder()
  return with(stringBuilder){ // 메서드를 호출하려는 수신객체 지정
    for(letter in 'A'..'Z'){
      this.append(letter) // this 를 명시해서 앞에서 지정한 수신객체의 메서드를 호출
    }
    append("append!") // this 없이도 함수에 바로 접근 가능
    this.toString() // return 값
  }
}
~~~

- apply 함수 : with 와 유사하지만, 항상 자신에게 전달된 객체를 반환한다는 차이점이 존재한다.
  - with() 는 람다의 결과를 반환 / apply() 는 객체 자체를 반환
  
~~~kotlin
/* apply 사용 O */
fun alphabet() = StringBuilder().apply {
  for(letter in 'A'..'Z'){
    append(letter) // 함수에 바로 접근해서 사용
  }
  append("append!") // 함수에 바로 접근해서 사용
}.toString()
~~~