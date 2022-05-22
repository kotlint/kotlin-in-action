# 연산자 오버로딩과 기타 관례

## 7장에서 다루는 내용

### 연산자 오버로딩

### 관례 : 여러 연산을 지원하기 위해 특별한 이름이 붙은 메서드

### 위임 프로퍼티

## 연산자 오버로딩

Overloading : 파라미터의 타입, 갯수와 상관 없이 어떤 파라미터를 받아도 유사한 기능을 수행하도록 함수를 구현한 것.

Convention : 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는(또다른 문법적 약속) 기법

## 7.1 산술 연산자 오버로딩

코틀린에서 관례를 사용하는 가장 단순한 예는 산술 연산자임.
자바에서는 원시 타입에 대해서만 산술 연산자를 사용할 수 있고, 추가로 String에 대해 + 연산자를 사용할 수 있음.
ex). "a" + "b" = "ab"

### 7.1.1 이항 산술 연산 오버로딩

더하기(+) 오버로딩 구현하기

```kotlin
// 7.1 plus 연산자 구현하기
data class Point(val x : Int, val y : Int){
    operator fun plus(other : Point) : Point {
        return Point( x + other.x, y + other.y)
    }
}

fun main(){
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)

    val p3 = p1 + p2 // a + b --> a.plus(b)
    println(p3)
}

```

관례 + 를 사용하려면 plus 앞에 연산자 오버로딩을 한다는 의미인 operator가 붙어야 함

[오버로딩 가능한 이항 산술 연산자]

a \* b ---> a.times(b)
a / b ---> a.div(b)
a % b ---> a.mod(b)
a + b ---> a.plus(b)
a - b ---> a.minus(b)

```kotlin
operator fun Point.minus(other : Point) : Point{
    return Point(x - other.x, y - other.y)
}

fun main(){
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)

    val p3 = p1 - p2

    println(p3)
}
```

```kotlin
data class Point(val x : Int, val y : Int){
    operator fun plus(other: Point) : Point{
        return Point(x + other.x, y + other.y)

    }
}
```

연산자를 오버로딩 하는 함수 앞에는 반드시 operator가 있어야 함
코틀린에서는 프로그래머가 직접 연산자를 만들어 사용할 수 없고 언어에서 미리 정해둔 연산자만 오버로딩 할 수 있다

직접 오버라이딩을 해도 우선순위는 표준 숫자 타입과 같음

연산자를 정의 할 때 두 피연산자가 같은 타입일 필요는 없음.

```kotlin
// 7.3 두 피연산자의 타입이 다른 연산자 정의하기

operator fun Point.times(scale : Double) : Point{
    return Point((x * scale).toInt(), (y * scale).toInt())
}

fun main(){
    val p1 = Point(10, 20)
   	p1 * 3.0

    println(p1)
}

```

코틀린 연산자가 자동으로 교환법칙을 지원하지는 않음 (즉 오버라이딩을 해야 함)

```kotlin
operator fun Int.times(p : Point) : Point{
    return Point((this * p.x).toInt(), (this * p.y).toInt())
}

```

```kotlin
//7.4 결과 타입이 피연산자 타입과 다른 연산자 정의하기
operator fun Char.times(count : Int) : String{
    return toString().repeat(count)
}

fun main() {
	println('a'*5) // input : Char , output : String
}

```

일반 함수와 마찬가지로 operator 함수도 오버로딩이 가능함 ex) plus(a, b), plus(a, b, c)
따라서 이름은 같지만 파라미터 타입이 서로 다른 연산자 함수를 만들 수 있음

### 7.1.2 복합 대입 연산자 오버로딩

plus와 같은 연산자를 오버로딩하면 코틀린은 + 연산자뿐 아니라 그와 관련 있는 연산자인 복합대입연산자(+=, ==)도 함께 지원함.
(변수가 변경 가능한 경우에만 복합 대입 연산자를 사용 할 수 있음)

```kotlin
data class Point(val x : Int, val y : Int){
    operator fun plus(other : Point) : Point {
        return Point( x + other.x, y + other.y)
    }
}

fun main(){
    var p1 = Point(10, 20)
    p1 += Point(30, 40) // a + b --> a.plus(b)
    println(p1)
}
```

반환 타입이 Unit인 plusAssign 함수를 정의하면 코틀린은 += 연산자에 그 함수를 사용.
plusAssign : +=
minusAssign : -=
timesAssign : \*=

### 7.1.2 단항 연산자 오버로딩

```kotlin
data class Point(val x : Int, val y : Int){
operator fun unaryMinus() : Point {
        return Point(-x, -y)
    }

operator fun not() : Point {
    return Point(0, 0)
}

}

// operator fun Point.unaryMinus() : Point {
//         return Point(-x, -y)
// }


fun main(){
    val p = Point(10, 20)
    println(-p)
    println(!p)
}
```

## 7.2 비교 연산자 오버로딩

코틀린에서는 모든 객체에 대해 비교 연산자를 수행 할 수 있음

equals나 compareTo를 호출해야하는 자바와 달리 코틀린에서는 == 비교 연산자를 직접 사용할 수 있음

관례(convention)를 살펴보자

### 7.2.1 동등성 연산자: equals

a == b --> a?.equalse(b) ?: (b==null)

### 7.3.1 인덱스로 원소에 접근 : get과 set

```kotlin
mutableMap[key] = newValue
```

인덱스 연산자를 사용해 원소를 읽는 연산은 get 연산자 메서드로 변환됨
x[a] ---> x.get(a)

인덱스 연산자를 사용해 원소를 쓰는 연산은 set 연산자 메서드로 변환됨
x[b] = c ---> x.set(b, c)

```kotlin
//7.9 get 관례 구현하기

fun main() {
    val point = Point(1, 2)
    println(point[0])
    println(point[1])
    println(point[2])
}

class Point(val x : Int, val y : Int)

operator fun Point.get(index: Int) : Int{
    return when(index){
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException("Invalid coordinate $index")
    }

}
```

인덱스에 해당하는 컬렉션 원소를 쓰고 싶을 때는 set이라는 이름의 함수를 정의하면 됨.

```kotlin

//7.10 관례를 따르는 set 구현하기

fun main() {
	val point = MutablePoint(1, 2)
    point[0] = 0 // point.set(0, 0)
    point[1] = 100 // point.set(0, 100)
    println("${point[0]} , ${point[1]}")
}

data class MutablePoint(var x : Int, var y : Int)

operator fun MutablePoint.set(index : Int, value : Int){
    when(index){
        0 -> x = value
        1 -> y = value
        else ->
        	throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

operator fun MutablePoint.get(index : Int) : Int{
    when(index){
        0 -> return x
        1 -> return y
        else ->
        	throw IndexOutOfBoundsException("Invalid coordinate $index")
    }
}

```

위 코드에서 get과 set을 모두 구현해 주어야 배열 형식으로 사용 할 수 있다.
(set만 구현하고 get은 구현안하면 point[0]를 읽어 올 수 없음 : Unresolved reference )

### 7.3.2 in 관례

컬렉션에서 in은 객체가 컬렉션에 들어있는지 검사한다.
in 연산자와 대응하는 함수는 contains이다. a in c ---> a.contains(c)

```kotlin
//7.11 in 관례 구현하기
data class Point(val x : Int, val y : Int)

data class Rectangle(val upperLeft : Point, val lowerRight : Point)

operator fun Rectangle.contains(p : Point) : Boolean {
    return p.x in upperLeft.x until lowerRight.x && p.y in upperLeft.y until lowerRight.y
}

fun main() {
	val rect = Rectangle(Point(10, 20), Point(50, 50))
    println(Point(30, 40) in rect)
    println(Point(0, 0) in rect)
}
```

### 7.3.3 rangeTo 관례

범위를 만드려면 .. 구문을 사용하면 된다. ..연산자는 rangeTo를 간략하게 표현하는 방법이다.
start..end --> start.rangeTo(end)
코틀린 표준 라이브러리에는 모든 Comparable 객체에 대해 적용 가능한 rangeTo 함수가 들어 있다.

```kotlin
//ex)

public operator fun Int.rangeTo(n : Int) : IntRange{
        println(n)
        return 1..10
    }

fun main() {
	1.rangeTo(10)
	(1..10).forEach{println(it)}
}


```

### 7.3.4 for 루프를 위한 iterator 관례

코틀린은 iterator 메서드를 확장 함수로 정의 할 수 있다. ex) CharSequence에 대한 iterator 확장 함수

```kotlin
//7.13 날짜 범위에 대한 이터레이터 구현하기
import java.time.LocalDate

operator fun ClosedRange<LocalDate>.iterator() : Iterator<LocalDate> =
    object : Iterator<LocalDate>{
        var current = start
        override fun hasNext() = current <= endInclusive
        override fun next() = current.apply{
            current=plusDays(2)
        }
    }

fun main() {
	val newYear = LocalDate.ofYearDay(2017,1)
    val daysOff = newYear.minusDays(10)..newYear
    for (dayOff in daysOff) {println(dayOff)}

}
```

## 7.4 구조 분해 선언과 component 함수

구조 분해 : 복합적인 값을 분해 하여 여러 다른 변수를 초기화 할 수 있는 방법

```kotlin
val p = Point(10, 20)
val (x, y) = p

println(x)
println(y)
```

```kotlin
val (a, b) = p ---> val a = p.component(), val b = p.component()
```

data class의 주 생성자(constructor)에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 component'N' 함수를 만들어줌
data 타입이 아닌 클래스에서는 component'N'을 만들어 주어야 함

```kotlin

class Point(val x : Int, val y : Int){
    operator fun component1() = y
    operator fun component2() = x
}

fun main(){
val p = Point(10, 20)
val (x, y) = p

println(x)
println(y)
}

```

```kotlin
\\ 7.14 구조 분해 선언을 사용해 여러 값 반환하기

data class NameComponents1(val name : String, val extension : String)
class NameComponents2(val name : String, val extension : String)

fun main(){

    val comp1 = NameComponents1("박태민", "txt")
    val (name1, ext1) = comp1
    println(name1)
    println(ext1)

    val comp2 = NameComponents2("박태민", "txt")
//     val (name2, ext2) = comp2
//     println(name2) 				error! because it is not included Destructuring declation in general Class type...
//     println(ext2)

}

```

컬렉션에 대해서도 구조 분해를 사용 할 수 있다. 컬렉션이란 연관있는 자료들을 모아놓은 자료구조이다.

```kotlin
//7.16 컬렉션에 대해 구조 분해 선언 사용하기
data class NameComponents(
    val name : String,
    val extension : String){

    fun splitFilename(fullName : String) : NameComponents{
        val (name, extension) = fullName.split('.',  limit=2)//split collection
        return NameComponents(name, extension)
    }

}

fun main(){
    val k = NameComponents("kfc","먹고싶다")
    val c = k.splitFilename("hello.kt")
    println(c.name)
    println(c.extension)
}

```

코틀린 표준 라이브러리에는 맨 앞의 다섯 원소에 대한 componentN을 제공함.
즉, data class에서 default로 val (a, b, c, d, e, f) = g가 불가능. 오버라이딩으로는 가능한지 테스트해보자

```kotlin
class Seven(val one : Int,
                 val two : Int,
                 val three : Int,
                 val four : Int,
                 val five : Int,
                 val six : Int,
                 val seven : Int){
    operator fun component1() = one
    operator fun component2() = two
    operator fun component3() = three
    operator fun component4() = four
    operator fun component5() = five
    operator fun component6() = six
    operator fun component7() = seven
}

fun main(){
	val sevens = Seven(1, 2, 3, 4, 5, 6, 7)
    val (a, b, c, d, e, f, g) = sevens
    println(a)
    println(b)
    println(c)
    println(d)
    println(e)
    println(f)
    println(g)
}
//it works! but data class type not possible...
```

### 7.4.1 구조 분해 선언과 루프

변수 선언이 들어갈 수 있는 장소라면 어디든 구조 분해 선언을 사용 할 수 있음
(맵의 원소에 대한 이터레이션)

```kotlin
// 7.16 구조 분해 선언을 사용해 맵 이터레이션 하기
fun printEntries(map : Map<String, String>){
    for((key, value) in map)
    {
        println("$key->$value")
    }
}

fun main(){
	val map = mapOf("Oracle" to "Java", "JetBrains" to "Kotlin")
    printEntries(map)
}

```

위 예제에서 두가지 관례를 확인 할 수 있음. 1. 객체 이터레이션(in)/ 2. 구조 분해 선언( val (a, b) = c)

## 7.5 프로퍼티 접근자 로직 재활용 : 위임 프로퍼티

위임 프로퍼티(Delegated Property) : 코틀린의 강력한 관례 중 하나
위임 프로퍼티를 사용하면 값을 뒷받침하는 필드에 단순히 저장하는 것 보다 더 복잡한 방식으로 작동하는 프로퍼티를
쉽게 구현 할 수 있음

위임이란? : 객체가 직접 작업을 수행하지 않고 다른 도우미 객체가 그 작업을 처리하게 맡기는 디자인 패턴을 말함
위임객체(delegate) : 작업을 처리하는 도우미 객체

### 7.5.1 위임 프로퍼티 소개

```kotlin
class Foo{
    var p : Type by Delegate()
}
```

p 프로퍼티는 접근자 로직을 다른 객체에게 위임
