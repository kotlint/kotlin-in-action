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

## launch

launch 는 코루틴을 Job 으로 반환하며 만들어진 코루틴은 기본적으로 즉시 실행된다. Job 의 cancel() 을 통해서 코루틴 실행을 중단 시킬 수 있다.

launch 가 동작하기 위해서는 CoroutineScope 객체가 블록의 this 로 지정돼야 한다.

```kotlin
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyStandaloneCoroutine(newContext, block) else
        StandaloneCoroutine(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```

다른 suspend 함수 내부라면 해당 함수가 사용 중인 CoroutineScope 가 있겠지만, 그렇지 않은 경우에는 GlobalScope 를 이용하면 된다.

```kotlin
fun now() = ZoneDateTime.now().toLocalTime().truncatedTo(ChronoUnit.MILLIS)

fun log(msg: String) = println("${now()}: ${Thread.currentThread()}: ${msg}")

fun launchInGlobalScope() {
    GlobalScope.launch {
        log("coroutine started")
    }
}

fun main() {
    log("main() started")
    launchInGlobalScope()
    log("launchInGlobalScope() executed")
    Thread.sleep(5000L)
    log("main() terminated")
}
```

실행 결과는 다음과 같다.

```idle
Thread[main] main() started 
Thread[main] launchInGlobalScope() executed
Thread[DefaultDispatcher-worker-2] coroutine started
Thread[main] main() terminated
```

GlobalScope.launch 가 만들어낸 코루틴은 메인 함수와 다른 스레드에서 실행된다. GlobalScope 는 메인 스레드가 실행 중인 동안만 코루틴의 동작을 보장해준다. 따라서 Thread.sleep(5000L) 이 없더라면 코루틴이 실행되지 않을 것이다.

이를 방지하기 위해서는 비동기적으로 launch 를 실행하거나, launch 가 모두 다 실행될 때 까지 기다려야 한다. 코루틴의 실행이 끝날 때까지 현재 스레드를 블록시키는 함수 `runBlocking()` 이 있다. runBlocking() 은 일반 함수이기 때문에 별도의 코루틴 스코프 객체 없이 사용 가능하다.

```kotlin
expect fun <T> runBlocking(context: CoroutineContext = EmptyCoroutineContext, block: suspend CoroutineScope.() -> T): T
```

```kotlin
fnn runblockingExample() {
    runBlocking {
        launch {
            log("GlobalScope.launch started")
        }
    }
}
```

실행 결과는 다음과 같다.

```idle
Thread[main] main() started 
Thread[main] coroutine started
Thread[main] runBlockingExample() executed
Thread[main] main() terminated
```

__스레드가 모두 main() 에서 동작하기 때문에 코루틴들이 서로 yield() 를 해주면서 협력할 수 있다.__

```kotlin
fun yieldExample() {
    runBlocking {
        launch {
            log("1")
            yield()
            log("3")
            yield()
            log("5")
        }
        log("after first launch")
        launch {
            log("2")
            delay(1000L)
            log("4")
            delay("1000L")
            log("6")
        }
    }
}
```

실행 결과는 다음과 같다.

```idle
Thread[main] main() started 
Thread[main] after first launch
Thread[main] after second launch
Thread[main] 1
Thread[main] 2
Thread[main] 3
Thread[main] 5
Thread[main] 4
Thread[main] 6
Thread[main] after runBlocking
Thread[main] yieldExample() executed
Thread[main] main() terminated
```

결과를 통해서 다음과 같은 특징을 알 수 있다.

- launch 는 즉시 반환된다.
- runBlocking 은 내부 코루틴이 모두 끝난 다음 반환된다.
- delay() 를 사용한 코루틴은 그 시간이 지날 때 까지 다른 코루틴에게 실행을 양보한다. 만약, delay() 대신 yield() 를 사용했다면 1,2,3,4,5,6 의 결과가 나왔을 것이다.

## async

```kotlin
public fun <T> CoroutineScope.async(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> T
): Deferred<T> {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyDeferredCoroutine(newContext, block) else
        DeferredCoroutine<T>(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```

async 는 launch 와 같은 일을 한다. 차이는 async 는 Deferred 를 반환한다. Deferred 는 Job 을 상속했기 때문에 launch 대신 async 를 써도 문제가 없다.

```kotlin
interface Deferred<out T> : Job
```

> All functions on this interface and on all interfaces derived from it are thread-safe and can be safely invoked from concurrent coroutines without external synchronization.
> 
> Deferred interface and all its derived interfaces are not stable for inheritance in 3rd party libraries, as new methods might be added to this interface in the future, but is stable for use.

Deferred 의 타입 파라미터는 Deferred 코루틴이 계산을 하고 돌려주는 값의 타입이다. Job 은 Unit 을 돌려주는 `Deferred<Unit>` 이라고 생각할 수 있다.

async 는 코드 블록을 비동기로 실행할 수 있고, async 가 반환하는 Deferred 의 `await` 을 사용해서 코루틴이 결과 값을 내놓을 때까지 기다렸다가 결과 값을 얻어낼 수 있다.

```kotlin
/**
 * Awaits for completion of this value without blocking a thread and resumes when deferred computation is complete, returning the resulting value or throwing the corresponding exception if the deferred was cancelled.
 */
abstract suspend fun await(): T
```

async/await 을 사용한 코드는 다음과 같다.

```kotlin
val result = runBlocking {
    orders.map { order ->
        async {
            val approveResult = orderService.approve(order)
            order.update(status = approveResult.status)
            approveResult
        }
    }.awaitAll()
}
```

async 로 코드를 실행하는 데는 시간이 거의 걸리지 않는다. 병렬 처리와의 가장 큰 차이는 모든 작업이 main() 스레드 안에서 일어난다는 점이다. 비동기 코드가 늘어남에 따라 async/await 을 사용한 비동기가 빛을 발한다. 실행하려는 작업이 시간이 얼마 걸리지 않거나 I/O 에 의한 대기 시간이 크고, CPU 코어 수가 작아 동시에 실행할 수 있는 스레드 개수가 한정된 경우에는 특히 코루틴과 일반 스레드를 사용한 비동기 처리 사이에 차이가 커진다.

## CoroutineContext

> Defines a scope for new coroutines. Every coroutine builder (like launch, async, etc.) is an extension on CoroutineScope and inherits its coroutineContext to automatically propagate all its elements and cancellation.

```kotlin
interface CoroutineScope
```

프로퍼티로는 CoroutineContext 를 가지고 있다.

```kotlin
abstract val coroutineContext: CoroutineContext
```

사실 CoroutineScope 는 CoroutineContext 를 coroutine builder(Ex. launch 등) 확장 함수 내부에서 사용하기 위한 매개체 역할만을 담당한다.

> CoroutineContext - Persistent context for the coroutine. It is an indexed set of Element instances. An indexed set is a mix between a set and a map. Every element in this set has a unique Key.

CoroutineContext 는 Element 와 Key 라는 두 가지 타입이 있다.

> Element - An element of the CoroutineContext. An element of the coroutine context is a singleton context by itself.
>
> Key - Key for the elements of CoroutineContext. E is a type of element with this key.

CoroutineContext 는 실제로 코루틴이 실행 중인 여러 작업(Job 타입)과 디스패처를 저장하는 `Persistence Context` 라고 할 수 있다. 코틀린 런타임은 CoroutineContext 를 사용해서 다음에 실행할 작업을 선정하고, 어떻게 스레드에 배정할지에 대한 방법을 결정한다.

```kotlin
launch { // 부모 컨텍스트를 사용
    
}

launch(Dispatchers.Unconfined) { // 특정 스레드에 종속되지 않음. 메인 스레드 사용
    
}

launch(Dispatchers.Default) { // 기본 디스패처 사용
    
}

launch(newSingleThreadContext("Async-Thread")) { // 새 스레드를 사용
    
}
```

같은 launch 를 사용하더라도 전달하는 컨텍스트에 따라 서로 다른 스레드 상에서 코루틴이 실행된다.

## CoroutineDispatcher

```kotlin
abstract class CoroutineDispatcher : AbstractCoroutineContextElement, ContinuationInterceptor
```

> The following standard implementations are provided by kotlinx.coroutines as properties on the Dispatchers object:
> 
> Dispatchers.Default — is used by all standard builders if no dispatcher or any other ContinuationInterceptor is specified in their context. It uses a common pool of shared background threads. This is an appropriate choice for compute-intensive coroutines that consume CPU resources.
> 
> Dispatchers.IO — uses a shared pool of on-demand created threads and is designed for offloading of IO-intensive blocking operations (like file I/O and blocking socket I/O).
> 
> Dispatchers.Unconfined — starts coroutine execution in the current call-frame until the first suspension, whereupon the coroutine builder function returns. The coroutine will later resume in whatever thread used by the corresponding suspending function, without confining it to any specific thread or pool. The Unconfined dispatcher should not normally be used in code.
>
> Private thread pools can be created with newSingleThreadContext and newFixedThreadPoolContext.
>
> An arbitrary java.util.concurrent.Executor can be converted to a dispatcher with the asCoroutineDispatcher extension function.

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
