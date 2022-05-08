### Factory Method Pattern

객체 생성을 다른 클래스로 위임하는 패턴.

### How to design Factory Method in Kotlin?

0. enum Type을 정의한다. (getInstance()시 타입을 간편하게 입력하기 위함)
1. 관련 클래스를 그룹화 할 수 있는 추상 클래스를 정의한다. (실제 getInstance() 호출 시, 세부 클래스 추상 클래스로 캡슐화하기 위함)
2. 객체 생성을 위임 할 수 있는 Factory 클래스를 만들고, 인스턴스를 반환 할 수 있는 getInstance() 메소드를 작성한다.

### CODE Sample

```kotlin
enum class HumanType{
    ASIAN, AMERICAN
}

interface Human {
  	fun speak()
}

class AsianHuman : Human {
    override fun speak(){
        println("안녕하세요!")
    }
}

class AmericanHuman : Human {
    override fun speak(){
        println("Hello!")
    }
}

class HumanFactory {
    companion object {
        fun getInstance(type : HumanType) : Human {
            when(type){
                HumanType.ASIAN -> return AsianHuman()
                HumanType.AMERICAN -> return AmericanHuman()
            }
        }
    }
}


fun main(){
 	var taeminPark = HumanFactory.getInstance(HumanType.ASIAN)
    var justinBiber = HumanFactory.getInstance(HumanType.AMERICAN)

    taeminPark.speak()
    justinBiber.speak()
}
```
