### Template Method Pattern

수식으로 정의하면 추상 클래스 + 시나리오 이다. 끝
즉 템플릿 메소드 패턴에는 추상 클래스를 상속받는 클래스들이 존재하고, 그 클래스들은 동일한 시나리오를 수행한다.
이 시나리오에 들어있는 메소드를 스켈레톤 메소드(스켈레톤=뼈대)라고 명명한다.
시나리오는 보통 2개이상의 스켈레톤 메소드를 호출한다.

### How to design Template Method in kotlin?

0. 추상 클래스 정의
1. 추상 클래스에 시나리오 메소드 구현 및 스켈레톤 메소드 정의
2. 추상 클래스를 상속 받는 메소드에 스켈레톤 메소드 구현

## CODE Sample

```kotlin

interface Beverage {
  	
    fun make(){
        front()
        middle()
        end()
    }
    
    fun front()
    
    fun middle()
    
    fun end()
}

class CafeLatte : Beverage {
    override fun front(){
        println("원두 갈기")
    }
    
    override fun middle(){
        println("우유 스팀")
    }
    
    override fun end(){
        println("에스프레소 + 우유스팀 섞기")
    }
}


class Lemonade : Beverage {
    override fun front(){
        println("레몬 착즙")
    }
    
    override fun middle(){
        println("얼음물 준비")
    }
    
    override fun end(){
        println("레몬 + 얼음물 섞기")
    }
}

fun main(){
 	var lemonade = Lemonade()
    lemonade.make()
    
    var cafelatte = CafeLatte()
    cafelatte.make()
}

```