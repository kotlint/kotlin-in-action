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
