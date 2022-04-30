# 람다로 프로그래밍

- 람다는 다른 함수에 넘길 수 있는 작은 코드 조각이다.

람다는 공통 코드 구조를 라이브러리 함수로 쉽게 뽑아낼 수 있다.(Kotlin 라이브러리는 람다를 아주 많이 사용한다.)

### 람다 소개 : 코드 블록을 함수 인자로 넘기기

- 무명 내부 쿨래스로 리스너 구현하기

~~~
button.setOnClickListener (new OnClickListener() {
    @Override
    public coid onClick(View view){
        /*클릭시 수행할 동작*/
    }
});
~~~

- 람다로 리스너 구현

~~~
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
fun printMessagesWithPrefix (message : Collection<String>, prefix : String){
    message.forEach{ // 각 원소에 대해 수행할 작업을 람다로 받는다.
        println("$prefix $it") // 람다 안에서 함수의 'prefix' 파라미터를 사용
    }
}
fun main(){
    val error = listOf("403 Forbidden", "404 Not Found")
    printMessagesWithPrefix(error, "Error: ")
}

// 결과
// Error: 403 Forbidden
// Error: 404 Not Found
~~~

### 멤버 참조

### 컬렉션 함수형 API

### 필수적인 함수 : filter 와 map

### all, any, count, find : 컬렉션에 술어 적용

### groupBy : 리스트를 여러 그룹으로 이뤄진 맵으로 변경

### flatMap 과 flatten : 중첨된 컬렉션 안의 원소 처리

### 지연 계산(lazy) 컬렉션 연산

### 시퀀스 연산 실행 : 중간 연산과 최종 연산

### 시퀀스 만들기

### 자바 함수형 인터페이스 활용

### 자바 메서드에 람다를 인자로 전달

### SAM 생성자 : 람다를 함수형 인터페이스로 명시적으로 변경

### 수신 객체 지정 람다: with 와 apply

### apply 함수