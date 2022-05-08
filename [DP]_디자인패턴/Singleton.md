### Singleton Pattern

싱글톤 패턴은 다음 두가지 특징을 가지고 있음.

1. 인스턴스 생성은 객체 내부에서 생성됨
2. 프로그램이 시작되고 끝나는 동안 단 하나의 객체만 유지함

### How to design Singleton in Kotlin?

Kotlin에는 object라는 키워드를 지원함. object 키워드를 통해 단 하나의 싱글톤 객체를 생성 할 수 있음
그러나 java와 다르게 object는 인스턴스 생성 시점을 프로그래머가 설정 할 수 없음.

#### object와 companinon object

companion object는 클래스의 특정 부분을 static으로 만들 수 있음

### CODE Sample

```

object Human{
    //object는 주생성자/부생성자 모두 사용 불가능
    fun speak(){
        println("Hello World!")
    }
}

fun main(){
 	Human.speak()
}

```
