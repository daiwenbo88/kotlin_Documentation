###### Kotlin - 协程

####  简介

协程（Coroutine）并非什么新产物，它是几十年前就已存在的概念，但兴起于近些年。Kotlin作为一门朝阳语言，它跟其它近些年新兴语言如：go、Lua、python等，一样都引入了协程的语法支持。

在Java并不存在协程的语法，我们在过去使用Java开发过程中，若想要使用异常逻辑一般需要传入一个回调接口，待异步逻辑执行完后通过回调接口进行结果返回。

而协程可以认为它是传统线程模型的进化版，它可以由程序自行控制<font color=#FF0004>挂起</font>和<font color=#FF0004>恢复</font>；可以实现多任务的协作执行；可以解决异步任务控制流的灵活协移从而降低异步程序的设计复杂度。

还是不明白协程是什么？那我们用最简单明了的一句话来总结协程吧，<font color=#FF0004>**协程没有异步的能力，但能让异步逻辑使用同步写法**</font>。


#### 协程与线程的对比

<font color=#333333>**线程：**</font>

1.一个进程可以拥有多个线程；

2.线程由系统调度，线程切换或阻塞开销较大；

3.线程看起来像是语言层次，但实质上和进程一样是操作系统级的产物，被操作系统内核所管理，只是通过API暴露给开发者使用；

4.线程之间是抢占式的调度，线程一旦开始执行，从任务的角度来看，就不会被暂停，直到任务结束这个过程都是连续的；

5.线程阻塞是会阻塞当前线程，并且空耗CPU时间而不能执行其它任务。

<font color=#333333>**协程：**</font>

1.一个线程可以拥有多个协程；

2.协程依赖于线程，协程挂起时不会阻塞线程，几乎不存在开销；

3.协程是编译器级的魔术，是语言层次的语法，通过插入相关的代码使得代码段能够实现分段式的执行，完全是由程序所控制；

4.协程是非抢占式，是协作式的，所以需要自己释放使用权来切换到其他协程；

5.协程挂起不会阻塞线程，可以去执行其它计算任务，比如其它协程，这也是我们看到协程实现异步的效果。

```kotlin
fun main() {
    log("Main函数开始")
    coroutineDo{
        val result=blockFun()
        log("异步方法返回结果：${result}")
        result
    }
}

fun log(msg: String) {
    println("【${Thread.currentThread().name}】${msg}")
}

fun <T> coroutineDo(block: suspend () -> T) {
    block.startCoroutine(object : Continuation<T> {
        //创建并启动协程
        override val context: CoroutineContext =
            EmptyCoroutineContext// 协程上下文，如不作处理使用EmptyCoroutineContext即可

        override fun resumeWith(result: Result<T>) { // 协程结果统一处理
            log("收到异步结果：${result}")
        }
    })
}
//关键字suspend修饰，表示是一个挂起函数
suspend fun blockFun() = suspendCoroutine<String> { continuation ->
    Thread {  // 协程没有异步的能力，所以耗时操作依然在线程中完成
        log("异步开始")
        Thread.sleep(2000)
        try {
            continuation.resumeWith(Result.success("异步请求成功")) // 异步成功的返回
        } catch (e: Exception) {
            continuation.resumeWith(Result.failure(e))  // 异步失败的返回
        }
    }.start()
}
```



coroutineDo函数是一个高阶函数，因为它的接收参数也是一个函数，而且这个参数是一个“suspend () -> T”类型，代表接收的是一个挂起函数，该挂起函数又返回了T类型。coroutineDo函数内接收到一个挂起函数后调用其startCoroutine函数，该函数接收一个Continuation对象。

Continuation是协程里一个关键的接口，源码如下。

```kotlin
public interface Continuation<in T> {
    public val context: CoroutineContext
    public fun resumeWith(result: Result<T>)
}
```

它是用于运行控制，负责正确的结果和异常的返回。它有两个成员：CoroutineContext也是一个接口，表示运行上下文，用于资源持有、运行调度，如果不对它作处理可以给它赋于EmptyCoroutineContext ； resumeWith函数就是用于接收协程里返回成功或失败的结果了。

startCoroutine用于进行协程的创建和启动，源码如下：

```kotlin
public fun <T> (suspend () -> T).createCoroutine(completion: Continuation<T>): Continuation<Unit> =
    SafeContinuation(createCoroutineUnintercepted(completion).intercepted(), COROUTINE_SUSPENDED)
 
public fun <T> (suspend () -> T).startCoroutine(completion: Continuation<T>) {
    createCoroutineUnintercepted(completion).intercepted().resume(Unit)
}
```

startCoroutine相当于createCoroutine+ resume。createCoroutine函数是创建协程，它接收了一个Continuation,然后再返回了一个Continuation，resume就是启动协程。



bloclFun函数前面有关键字suspend修饰，表示是一个挂起函数，挂起函数只能在其它挂起函数或协程中被调用（因为它实际会转化成需要一个Continuation<T>参数的函数）。挂起函数调用的地方叫作挂起点，表示协程挂起，在IDE中代码行号旁边会出现这个符号，函数里通过Continuation.resumeWith（或者使用Continuation.resume和Continuation.resumeWithException）函数来返回结果，表示协程恢复。

suspend修饰的挂起函数实质上在转化过程中会多出一个Continuation<T>的参数，但是我们看不到，也不需要自己传入，这个Continuation参数对象，便是我们在startCoroutine中创建的（注意createCoroutine是接收一个Coroutine返回一个Coroutine，这个Continuation对象是返回的那个），然后再通过这个Continuation对象再转化成一个SafeContinuation对象。其实我们从使用代码中也能发现suspend函数实际是指向于suspendCoroutine函数，suspendCoroutine接收“(Continuation<T>) -> Unit”类型的函数参数，返回了T。源码如下。

```kotlin
public suspend inline fun <T> suspendCoroutine(crossinline block: (Continuation<T>) -> Unit): T =
    suspendCoroutineUninterceptedOrReturn { c: Continuation<T> ->
        val safe = SafeContinuation(c.intercepted())
        block(safe)
        safe.getOrThrow()
}
```

也就是我们通过suspendCoroutine函数便能获取到Continuation（SafeContinuation）对象，并对函数内进行Continuation. resumeWith的调用

请注意和记往：suspendCoroutine这里有一个很有意思的情况，如果suspend函数中并没存在线程的切换，则表示并没有真正的挂起，则会直接返回T，而如果存在挂起情况时，就会通过Continuation来返回。所以上面的Demo中，如果你尝试将Thread注释的话，程序的运行结果会按正常代码顺序执行。

```kotlin

fun main() {
    log("Main函数开始")
    suspend {
        val result = suspendCoroutine<String> { continuation ->
            Thread {
                log("异步开始")
                Thread.sleep(2000)
                try {
                    continuation.resumeWith(Result.success("异步请求成功"))
                } catch (e: Exception) {
                    continuation.resumeWith(Result.failure(e))
                }
            }.start()
        }
        log("异步方法返回结果：${result}")
        result
    }.startCoroutine(object : Continuation<String> {
        override val context: CoroutineContext = EmptyCoroutineContext
        override fun resumeWith(result: Result<String>) {
            log("收到异步结果：${result}")
        }
    })
    log("Main函数结束")
}
fun log(msg: String) {
    println("【${Thread.currentThread().name}】$msg")
}
```



#### 协程的分类

######  协程按调用栈分类

有栈协程：每个协程会分配单独的调用栈，类似线程的调用栈，可以通过栈来保存局部变量。可以在任意函数嵌套中挂起。例如 Lua的协程就是有栈式协程

无栈协程：不会分配单独的调用栈，挂起点状态通过闭包或对象保存。只能在当前函数中挂起。例如 Kotlin和Python都是一种无栈协程

######  协程按调用关系分类

对称协程：调度权可以转移给任意协程，协程之间是对等关系。对称协程的方式更像在执行多个相互独立的任务并发。

非对称协程：调度权只能转移给调用自己的协程，协程存在父子关系。大多实现都是非对称协程



