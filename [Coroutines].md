# Coroutines

코루틴은 컴퓨터 프로그램 구성 요소 중 하나로 비선점형 멀티태스킹(non-preemptive multitasking)을 수행하는 일반화한 서브루틴(subroutine)이다. 코루틴은 실행을 일시 중단(suspend) 하고, 재개(resume)할 수 있는 여러 진입 지점(entry point)을 허용한다.

## Subroutines
 
> 서브루틴은 여러 명령어를 모아 이름을 부여해서 반복 호출할 수 있게 정의한 프로그램 구성요소로 함수라고 부르기도 한다. 객체지향 언어에서는 메서드도 서브루틴이라 할 수 있다.
>
> 서브루틴에 진입하는 방법은 한 가지(해당 함수를 호출하면 서브루틴의 맨 처음부터 실행이 시작됨) 뿐이며, 그때 마다 __활성 레코드(activation record)__ 라는 것이 __스택(stack)__ 에 할당 되고, 서브루틴 내부의 로컬 변수 등이 초기화 된다.
> 
> 서브루틴에서 반환되고 나면 활성 레코드가 스택에서 사라지기 때문에 실행 중이던 모든 상태를 잃어버린다. 따라서 여러 번 반복 실행해도(전역 변수나 다른 부수 효과가 있지 않는 한) 항상 같은 결과를 반복해서 얻게 된다.

## Multitasking

> 멀티태스킹은 여러 작업을 동시에 수행하는 것처럼 보이거나 실제로 동시에 수행하는 것이다. 비선점형이란 멀티태스킹의 각 작업을 수행하는 참여자들의 실행을 운영체제가 강제로 일시 중단시키고 다른 참여자를 실행하게 만들 수 없다는 뜻이다. 따라서 각 참여자들이 서로 자발적으로 협력해야만 비선점형 멀티태스킹이 제대로 작동할 수 있다.

__따라서, 코루틴이란 서로 협력해서 실행을 주고 받으면서 작동하는 여러 서브루틴을 의미한다.__

## Coroutines Thread ?

- One can think of coroutines as a light-weight thread.
- The biggest difference is that coroutines are very cheap, almost free: we can create thousands of them, and pay very little in terms of performance.
- __Light-weight thread__

## suspend

코루틴 안에서 delay(), yield() 는 일시 중단(suspending) 함수라고 불린다. 코루틴이 아닌 일반 함수 속에서 일시 중단 함수를 사용하게 되면 __Suspend function 'yield' should be called only from a coroutine or another suspend function__ 이라는 오류가 표시된다. 즉, 일시 중단 함수를 코루틴이나 일시 중단 함수가 아닌 함수에서 호출하는 것은 컴파일러 수준에서 금지된다.

### Continuation passing style

일시 중단 함수는 어떻게 만들어질까? 일시 중단 함수 안에서 yield() 를 해야 하는 경우 어떤 동작이 필요할까?

- 코루틴에 진입할 때와 코루틴에서 나갈 때 __코루틴이 실행 중이던 상태를 저장하고 복구하는 등의 작업__ 을 할 수 있어야 한다.
- 현재 실행 중이던 위치를 저장하고 다시 코루틴이 재개될 때 해당 위치부터 실행을 재개할 수 있어야 한다.
- 다음에 어떤 코루틴을 실행할지 결정한다.

마지막 동작은 코루틴 컨텍스트에 있는 디스패처에 의해 수행된다. 일시 중단 함수를 컴파일하는 컴파일러는 앞의 두 가지 작업을 할 수 있는 코드를 생성해 내야 한다. 이때  코틀린은 __CPS(Continuation passing style) 변환과 상태 기계(state machine)__ 를 활용해 코드를 생성해낸다.

CPS 변환은 프로그램의 실행 중 특정 시점 이후에 진행해야 하는 내용을 별도의 함수로 뽑고(이런 함수를 `Continuation` 이라 함), 그 함수에게 현재 시점까지 실행한 결과를 넘겨서 처리하게 만드는 소스코드 변환 기술이다.

CPS 를 사용하는 경우 프로그램이 다음에 해야 할 일이 항상 컨티뉴에이션이라는 함수 형태로 전달된다.

```kotlin
suspend fun example(v: Int): Int {
    return v*2
}
```

코틀린 컴파일러는 이 함수를 컴파일하면서 뒤에 Continuation 을 인자로 만들어 붙여준다.

```kotlin
public static final Object example(int v, @NotNull Continuation var1)
```

그리고 이 함수를 호출할 때는 함수 호출이 끝난 후 수행해야 할 작업을 var1 에 Continuation 으로 전달하고, 함수 내부에서는 필요한 모든 일을 수행한 다음에 결과를 var1 에 넘기는 코드를 추가한다. (이 예제에서는 v*2 를 인자로 Continuation 을 호출하는 코드가 들어간다.)

## Links

- [kotlinx.coroutines](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/index.html)
- [wikipedia coroutines](https://en.wikipedia.org/wiki/Coroutine)
- [taehwandev kotlin coroutines](https://speakerdeck.com/taehwandev/kotlin-coroutines)

## 참고 문헌

- Kotlin In Action / Dmitry Jemerov, Svetlana Isakova 공저 / 에이콘
