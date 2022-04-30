# 연산자 오버로딩과 기타 관례

## 7장에서 다루는 내용

### 연산자 오버로딩

### 관례 : 여러 연산을 지원하기 위해 특별한 이름이 붙은 메서드

### 위임 프로퍼티

## 연산자 오버로딩

Overloading : 파라미터의 타입, 갯수와 상관 없이 어떤 파라미터를 받아도 유사한 기능을 수행하도록 함수를 구현한 것.

Convention : 어떤 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법

### 7.1 산술 연산자 오버로딩

```
data class Point(val x : Int, val y : Int){
    operator fun plus(other: Point) : Point{
        return Point(x + other.x, y + other.y)

    }
}
```

연산자를 오버로딩 하는 함수 앞에는 반드시 operator가 있어야 함
코틀린에서는 프로그래머가 직접 연산자를 만들어 사용할 수 없고 언어에서 미리 정해둔 연산자만 오버로딩 할 수 있다

### 7.3.1 인덱스로 원소에 접근 : get과 set

```
mutableMap[key] = newValue
```

인덱스 연산자를 사용해 원소를 읽는 연산은 get 연산자 메서드로 변환됨
x[a] ---> x.get(a)

인덱스 연산자를 사용해 원소를 쓰는 연산은 set 연산자 메서드로 변환됨
x[b] = c ---> x.set(b, c)

```
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

```

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

```
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

```
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

```
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

```
val p = Point(10, 20)
val (x, y) = p

println(x)
println(y)
```

```
val (a, b) = p ---> val a = p.component(), val b = p.component()
```

data class의 주 생성자(constructor)에 들어있는 프로퍼티에 대해서는 컴파일러가 자동으로 component'N' 함수를 만들어줌
data 타입이 아닌 클래스에서는 component'N'을 만들어 주어야 함

```

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

```
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
