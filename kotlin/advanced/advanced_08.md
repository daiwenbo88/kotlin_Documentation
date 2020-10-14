#### 协程官方框架初步认识

#### **GlobalScope.launch**

```kotlin
fun main() {
    GlobalScope.launch {    // 在应用程序的生命周期内启动一个新的协程并继续
        delay(1000L)        // 非阻塞的等待1秒钟
        println("【${Thread.currentThread().name}】World!")
    }
    println("【${Thread.currentThread().name}】Hello,")
    Thread.sleep(2000L)    // 阻塞的等待2秒钟，因为协程的生命周期受应用程序生命周期限制，所以这里保证协程内部逻辑执行完
}
//【main】Hello,
//【DefaultDispatcher-worker-1】World!
```

- GlobalScope.launch用于启动了一个运行在子线程的顶层协程。GlobalScope继承于CoroutineScope（协程的生命周期），表示此协程的生命周期随应用程序的生命周期，所以如果上面代码中不对主线程进行Thread.sleep(2000L)的话，主线程main方法运行结束后，应用程序生命周期就会结果，然而协程中的代码未能及时得到响应。
- delay 是一个特殊的挂起函数 ，它不会造成线程阻塞，但是会挂起协程，并且只能在协程中使用。
  

#### **阻塞线程的协程**

```kotlin

fun main() {
    GlobalScope.launch {          // 在应用程序的生命周期内启动一个新的协程并继续
        delay(1000L)               // 非阻塞的等待1秒钟
        println("【${Thread.currentThread().name}】World!")
    }
    println("【${Thread.currentThread().name}】Hello,")
//    Thread.sleep(2000L)          // 阻塞的等待2秒钟，因为协程的生命周期受应用程序生命周期限制，所以这里保证协程内部逻辑执行完
    runBlocking {                  // 主线程中启动一个阻塞的协程
        delay(2000L)               // 使用非阻塞的等待2秒钟，这里仍然是阻塞
    }
}
//【main】Hello,
//【DefaultDispatcher-worker-1】World!
```

```kotlin
fun main() = runBlocking<Unit> {
    GlobalScope.launch {
        delay(1000L)
        println("【${Thread.currentThread().name}】World!")
    }
    println("【${Thread.currentThread().name}】Hello,")
    delay(2000L)
}
//【main】Hello,
//【DefaultDispatcher-worker-1】World!
```

- `runBlocking` 用于启动了一个运行在主线程的协程，这是会阻塞主线程的。
- 使有示例2，即`runBlocking` 来包装 main 函数的执行其运行效果是一样的。
- `runBlocking`  作用于最外层，也就是作用于协程的范围。可以建立一个阻塞当前线程的协程。然而它主要被用来在main函数中或者测试中使用，没有多大意义。

#### **等待协程（Job.join）**

```kotlin
fun main() = runBlocking<Unit> {
    val job = GlobalScope.launch {
        delay(1000L)
        println("【${Thread.currentThread().name}】World!")
    }
    println("【${Thread.currentThread().name}】Hello,")
    job.join() // 等待直到子协程执行结束
}
//【main】Hello,
//【DefaultDispatcher-worker-1】World!
```

- 协程通过GlobalScope.launch启动后可返回一个Job对象，通过Job对象的join函数可对协程进行等待。
- join函数只能运行在suspend函数中，或者说只能在协程内部使用，所以可见上述代码中是运行在runBlocking启动的协程中去等待GlobalScope.launch启动的协程

#### **取消协程（Job.cancel）**

```kotlin
fun main() = runBlocking {
    val job = GlobalScope.launch {
        repeat(1000) { i ->                 // 启动1000个协程
            println("【${Thread.currentThread().name}】协程工作中: $i ...")
            delay(500L)
        }
    }
    delay(1100L)
    println("【${Thread.currentThread().name}】准备取消协程")
    job.cancel()                            // 取消该作业
    job.join()                              // 等待作业执行结束，两个函数结合可用：job.cancelAndJoin()
    println("【${Thread.currentThread().name}】已经取消协程")
}

//【DefaultDispatcher-worker-1】协程工作中: 0 ...
//【DefaultDispatcher-worker-1】协程工作中: 1 ...
//【DefaultDispatcher-worker-1】协程工作中: 2 ...
//【main】准备取消协程
//【main】已经取消协程
```

- 同样在协程通过GlobalScope.launch启动后可返回一个Job对象，通过Job对象的cancel函数可对协程进行取消。
- 示例中通过repeat启动了1000个协程，然后每个协程里进行打印和耗时操作。
- 一般cancel和join一起使用，也可以使用cancelAndJoin函数来合并使用
  

###### **计算任务不能取消**



```kotlin
fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = GlobalScope.launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 10) {                        // 一个执行计算的循环，只是为了占用 CPU
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("协程工作中： ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }

    delay(1100L)
    println("【${Thread.currentThread().name}】准备取消协程")
    job.cancel()                                // 取消该作业
    job.join()                                  // 等待作业执行结束，两个函数结合可用：job.cancelAndJoin()
    println("【${Thread.currentThread().name}】已经取消协程")

}
```

- 协程代码必须协作才能被取消。 所有kotlinx.coroutines中的挂起函数都是可被取消的。它们检查协程的取消， 并在取消时抛出 CancellationException。 然而，如果协程正在执行计算任务，并且没有检查取消的话，那么它是不能被取消的。
- 运行结果中并没有发现CancellationException异常信息，那是因为CancellationException 被认为是协程执行结束的正常原因。

```kotlin

fun main() = runBlocking {
    val startTime = System.currentTimeMillis()
    val job = GlobalScope.launch(Dispatchers.Default) {
        var nextPrintTime = startTime
        var i = 0
        while (i < 10 && isActive) { // 一个执行计算的循环，只是为了占用 CPU
            if (System.currentTimeMillis() >= nextPrintTime) {
                println("【${Thread.currentThread().name}】协程工作中: ${i++} ...")
                nextPrintTime += 500L
            }
        }
    }
 
    delay(1100L)
    println("【${Thread.currentThread().name}】准备取消协程")
    job.cancel()                                // 取消该作业
    job.join()                                  // 等待作业执行结束，两个函数结合可用：job.cancelAndJoin()
    println("【${Thread.currentThread().name}】已经取消协程")
}
```

- 使用`isActive`可取消计算计算任务循环，使协程达到取消的效果。
- `isActive` 是一个可以被使用在` CoroutineScope` 中的扩展属性，它其实是协程上下文中的一个属性：`coroutineContext[Job]?.isActive `



#### **取消协程后释放资源（finally）**

```kotlin
fun main() = runBlocking {
    val job = GlobalScope.launch {
        try {
            repeat(1000) { i ->                 // 启动1000个协程
                println("【${Thread.currentThread().name}】协程工作中: $i ...")
                delay(500L)
            }
        } finally {
            println("【${Thread.currentThread().name}】协程已被取消，正在释放资源")
            delay(1000L)
            println("【${Thread.currentThread().name}】协程已被取消，已经释放完资源")
        }
    }
    delay(1100L)
    println("【${Thread.currentThread().name}】准备取消协程")
    job.cancel()                                // 取消该作业
    job.join()                                  // 等待作业执行结束，两个函数结合可用：job.cancelAndJoin()
    println("【${Thread.currentThread().name}】已经取消协程")
}

//【DefaultDispatcher-worker-1】协程工作中: 0 ...
//【DefaultDispatcher-worker-1】协程工作中: 1 ...
//【DefaultDispatcher-worker-1】协程工作中: 2 ...
//【main】准备取消协程
//【DefaultDispatcher-worker-1】协程已被取消，正在释放资源
//【main】已经取消协程
```

- 使用try {……} finally {……} 表达式可在协程取消抛出CancellationException时进行资源的释放工作。

- 请注意，上述示例中，finally中代码仅被执行了开头，delay后面并未被执行到

#### **指定协程上运行代码块（withContext）**

```
fun main() = runBlocking {
    val job = GlobalScope.launch {
        try {
            repeat(1000) { i ->                 // 启动1000个协程
                println("【${Thread.currentThread().name}】协程工作中: $i ...")
                delay(500L)
            }
        } finally {
            withContext(NonCancellable) {
                println("【${Thread.currentThread().name}】协程已被取消，正在释放资源")
                delay(1000L)
                println("【${Thread.currentThread().name}】协程已被取消，已经释放完资源")
            }
        }
    }
    delay(1100L)
    println("【${Thread.currentThread().name}】准备取消协程")
    job.cancel()                                // 取消该作业
    job.join()                                  // 等待作业执行结束，两个函数结合可用：job.cancelAndJoin()
    println("【${Thread.currentThread().name}】已经取消协程")
}
```

- 当需要挂起一个被取消的协程，可以将相应的代码包装在 withContext(NonCancellable) {……} 中，并使用 withContext 函数以及 NonCancellable 上下文，这里持续运行的代码就不会被取消。

#### **协程的超时（withTimeout / withTimeoutOrNull）**

```kotlin

fun main() = runBlocking {
    try {
        withTimeout(1100L) {
            repeat(1000) { i ->
                println("【${Thread.currentThread().name}】协程工作中: $i ...")
                delay(500L)
            }
        }
    } catch (e: TimeoutCancellationException) {
        println("【${Thread.currentThread().name}】协程超时了，${e.message}")
    }
}

//【main】协程工作中: 0 ...
//【main】协程工作中: 1 ...
//【main】协程工作中: 2 ...
//【main】协程超时了，Timed out waiting for 1100 ms
```

- `withTimeout` 抛出了 `TimeoutCancellationException`，它是` CancellationException` 的子类

```kotlin
fun main() = runBlocking {
    val result = withTimeoutOrNull(1100L) {
        repeat(1000) { i ->
            println("【${Thread.currentThread().name}】协程工作中: $i ...")
            delay(500L)
        }
        "Done"                  // 正常结束返回的结果
    }
    println("【${Thread.currentThread().name}】协程运行的结果是： $result")
}
//【main】协程工作中: 0 ...
//【main】协程工作中: 1 ...
//【main】协程工作中: 2 ...
//【main】协程运行的结果是： null
```

- 通过withTimeoutOrNull处理协程超时时不会抛出异常，而是返回一个null结果

#### **组合挂起函数（async/await并发使用）**

```kotlin
fun main() = runBlocking {
    val time = measureTimeMillis {
        val one =  doSomethingUsefulOne()
        val two =  doSomethingUsefulTwo()
        println("【${Thread.currentThread().name}】计算结果： ${one + two}")
    }
    println("【${Thread.currentThread().name}】共耗时： $time ms")
}
 
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L)
    println("【${Thread.currentThread().name}】doSomethingUsefulOne 计算中")
    return 2
}
suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L)
    println("【${Thread.currentThread().name}】doSomethingUsefulTwo 计算中")
    return 3
}
//【main】doSomethingUsefulOne 计算中
//【main】doSomethingUsefulTwo 计算中
//【main】计算结果： 5
//【main】共耗时： 2012 ms
```

- 上面示例代码中，doSomethingUsefulOne函数执行完毕后才到doSomethingUsefulTwo函数，它们是顺度执行的，所以它们的总耗时是两个函数执行时间之和。

```kotlin
fun main() = runBlocking {
    val time = measureTimeMillis {
        val one = async { doSomethingUsefulOne() }
        val two = async { doSomethingUsefulTwo() }
        println("【${Thread.currentThread().name}】计算结果： ${one.await() + two.await()}")
    }
    println("【${Thread.currentThread().name}】共耗时： $time ms")
}
 
suspend fun doSomethingUsefulOne(): Int {
    delay(1000L)
    println("【${Thread.currentThread().name}】doSomethingUsefulOne 计算中")
    return 2
}
suspend fun doSomethingUsefulTwo(): Int {
    delay(1000L)
    println("【${Thread.currentThread().name}】doSomethingUsefulTwo 计算中")
    return 3
}

//【main】doSomethingUsefulOne 计算中
//【main】doSomethingUsefulTwo 计算中
//【main】计算结果： 5
//【main】共耗时： 1020 ms
```



```kotlin
fun test() {
    val deferred1 = async(CommonPool) {
        "hello1"
    }
    val deferred2 = async(UI) {
        println("hello2")
        println(deferred1.await())
    }
}
//一个工作在主线程的协程，获取到了一个异步协程的返回值。
//这意味着，我们以后网络请求、图片加载、数据库、文件操作什么的，都可以丢到一个异步的协程中去，然后在同步代码中//直接取返回值，而不再需要去写回调了。这就是我们经常使用的一个最大特性。

fun test() {
    //每秒输出两个数字
    val job1 = launch(Unconfined, CoroutineStart.LAZY) {
        var count = 0
        while (true) {
            count++
            //delay()表示将这个协程挂起500ms
            delay(500)
            println("count::$count")
        }
    }

    //job2会立刻启动
    val job2 = async(CommonPool) {
        job1.start()
        "ZhangTao"
    }

    launch(UI) {
        delay(3000)
        job1.cancel()
        //await()的规则是：如果此刻job2已经执行完则立刻返回结果，否则等待job2执行
        println(job2.await())
    }
}

```



- 如果要计算的两个函数不需要依赖关系时，可使用 async进行执行并发来节省运行时间。
- async 类似于 launch。它启动了一个单独的协程，这是一个轻量级的线程并与其它所有的协程一起并发的工作。不同之处在于 launch 返回一个 Job 并且不附带任何结果值，而 async 返回一个Deferred（一个轻量级的非阻塞 future）， 这代表了一个将会在稍后提供结果的promise。你可以使用 .await() 在一个延期的值上得到它的最终结果。
- Deferred 也是一个 Job，所以如果需要的话，也可以通过cancel对它进行取消。
  

#### **协程上下文**

```kotlin
class ParameterContext(val isSuccess: Boolean) : AbstractCoroutineContextElement(Key) {
    companion object Key : CoroutineContext.Key<ParameterContext>
}
fun main() = runBlocking {
    launch(CoroutineName("HelloWorld") + ParameterContext(true)) {
        println("Job的上下文对象是：${coroutineContext[Job]}")
        println("传递的参数值是：${coroutineContext[ParameterContext]?.isSuccess}")
        println("协程的名字是：${coroutineContext[CoroutineName]?.name}")
    }
    println("Main函数执行完毕")
}

//Main函数执行完毕
//Job的上下文对象是：StandaloneCoroutine{Active}@369f73a2
//传递的参数值是：true
//协程的名字是：HelloWorld
```

- launch函数实质上是接收3个参数，其中第一个参数是CoroutineContext类型，也就是协程上下文。
  通过继承AbstractCoroutineContextElement接口来自定义上下文。
- CoroutineName是框架提供的上下文类，用于定义协程的名称。
  多个上下文可以使用“+”来进行add。
- 协程的 Job 是上下文的一部分，并且可以使用 coroutineContext [Job] 表达式在上下文中检索它。
  前面介绍取消协程时，取消计算任务中使用到的 isActive 其实本质上就是 coroutineContext[Job]?.isActive

#### **协程的生命周期**

`GlobalScope.launch`用于启动了一个运行在子线程的顶层协程，这个顶层的意思就是它的生命周期随应用程序的生命周期。虽然协程是轻量级的，但是它的运行仍然会消耗一些内存资源。所以我们完全可以在指定作用域内去启动协程

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch {
        delay(200L)
        println("【${Thread.currentThread().name}】在 runBlocking 里的任务")
    }
 
    coroutineScope { // 创建一个协程作用域
        launch {
            delay(500L)
            println("【${Thread.currentThread().name}】在 coroutineScope 里嵌套 launch 中的任务")
        }
 
        delay(100L)
        println("【${Thread.currentThread().name}】在 coroutineScope 里的任务")
    }
 
    println("【${Thread.currentThread().name}】在 Main 里的任务")
}
//【main】在 coroutineScope 里的任务
//【main】在 runBlocking 里的任务
//【main】在 coroutineScope 里嵌套 launch 中的任务
//【main】在 Main 里的任务
```

- 因为`runBlocking` 协程构建器将 main 函数转换为协程，包括` runBlocking` 在内的每个协程构建器都将 CoroutineScope 的实例添加到其代码块所在的作用域中，所以省略GlobalScope，直接使用launch就是等于在此作用域中启动协程。
- 使用` coroutineScope`构建器声明自己的作用域。它会创建一个协程作用域并且在里面所有已启动的协程执行完毕之前不会结束。
- `runBlocking` 与` coroutineScope`主要区别在于，`runBlocking` 方法会阻塞当前线程来等待， 而 `coroutineScope `只是挂起，会释放底层线程用于其他用途。 由于存在这点差异，`runBlocking` 是常规函数，而 `coroutineScope` 是挂起函数


#### **协程的嵌套关系**

```kotlin
fun main() = runBlocking {
 
    val job = launch {
        GlobalScope.launch {
            println("嵌套的顶层协程开始工作")
            delay(1000)
            println("嵌套的顶层协程结束工作")
        }
        launch {
            println("嵌套的子协程开始工作")
            delay(1000)
            println("嵌套的子协程结束工作")
        }
    }
    delay(500)
    job.cancel()
    delay(2000)
 
    println("Main函数执行完毕")
}
//嵌套的顶层协程开始工作
//嵌套的子协程开始工作
//嵌套的顶层协程结束工作
//Main函数执行完毕
```

- 当一个协程被一个协程嵌套启动时，它们就是父子关系，子协程会继承父协程的 CoroutineScope.coroutineContext，并且子协程Job也会成为父协程Job的子Job，当父协程被取消时，所以它的子协程也会被递归取消。
- 当我们使用GlobalScope来启动一个顶层协程时，被嵌套的顶层协程是没有父Job，因此它与这个“父协程”没有运作关系

```kotlin
fun main() = runBlocking {
    val job = launch {
        repeat(3) { i -> // 启动3个子协程
            launch  {
                delay((i + 1) * 200L)
                println("子协程 $i 执行完毕")
            }
        }
        println("父协程已经执行完毕")
    }
    job.join()
    println("Main函数执行完毕")
}
//父协程已经执行完毕
//子协程 0 执行完毕
//子协程 1 执行完毕
//子协程 2 执行完毕
//Main函数执行完毕
```

- 父协程一定会等待所有的子协程执行结束后它的生命周期才结束。

- 父协程并不显式的跟踪所有子协程的启动，所以不需要专门使用 Job.join 等待子协程

#### **协程调度器**

```kotlin
fun main() = runBlocking {
    launch {                            // 不传参数就是 EmptyCoroutineContext
        println("【${Thread.currentThread().name}】EmptyCoroutineContext 运行在父协程的所在线程的协程")
    }
    launch(Dispatchers.Unconfined) {
        println("【${Thread.currentThread().name}】Unconfined 运行在当前线程的协程1")
        delay(1000L)
        println("【${Thread.currentThread().name}】Unconfined 运行在当前线程的协程2")
    }
    launch(Dispatchers.Default) {       // GlobalScope.launch 就是使用了 Dispatchers.Default
        println("【${Thread.currentThread().name}】Default 运行在线程池的协程，用于处理CPU密集型工作")
    }
    launch(Dispatchers.IO) {
        println("【${Thread.currentThread().name}】IO 运行在线程池的协程，用于处理IO密集型工作")
    }
    launch(newSingleThreadContext("Myhread")) {
        println("【${Thread.currentThread().name}】运行在指定线程的协程")
    }
 
    println("【${Thread.currentThread().name}】Main函数执行完毕")
}

//【main】Unconfined 运行在当前线程的协程1
//【DefaultDispatcher-worker-1】Default 运行在线程池的协程，用于处理CPU密集型工作
//【DefaultDispatcher-worker-2】IO 运行在线程池的协程，用于处理IO密集型工作
//【main】Main函数执行完毕
//【main】EmptyCoroutineContext 运行在父协程的所在线程的协程
//【Myhread】运行在指定线程的协程
//【kotlinx.coroutines.DefaultExecutor】Unconfined 运行在当前线程的协程2
```

- Default和IO它们的背后都是一个线程池，区别在于IO使用到的线程池里线程队列是无限的，使用上Default适用于CPU密集型，也就是一些运算类型的操作；而IO适用于IO密集型，例如网络请求、文件读写等。
- 请注意Unconfined的运行结果，它的两次打印是在不同的线程中完成的。因为使用Unconfined启动协程首先会运行在当前线程上，但是只是在第一个挂起点之前是这样的，挂起恢复后运行在哪个线程完全由所调用的挂起函数决定。
- Unconfined属于非受限的调度器，它非常适用于执行不消耗 CPU 时间的任务，以及不更新局限于特定线程的任何共享数据（如UI）的协程。其实一般也不怎么使用。

#### **协程的启动模式**

协程的启动模式一共有4种， 我们从其源码可见它是一个枚举类

```kotlin
public enum class CoroutineStart {
    // 立即开始执行协程体，随时可以取消
    DEFAULT,
    // 只有在需要（start/join/await）时开始执行
    LAZY,
    // 立即开始执行协程体，且在第一个挂起点前不能被取消
    @ExperimentalCoroutinesApi
    ATOMIC,
    // 立即在当前线程执行协程体，直到遇到第一个挂起点
    @ExperimentalCoroutinesApi
    UNDISPATCHED;
}
```

```kotlin
fun main() = runBlocking {
    launch(start = CoroutineStart.DEFAULT) {
        println("【${Thread.currentThread().name}】协程开始")
        delay(100)
        println("【${Thread.currentThread().name}】协程结束")
    }
    println("【${Thread.currentThread().name}】Main函数执行完毕")
}
//【main】Main函数执行完毕
//【main】协程开始
//【main】协程结束
```

```kotlin
fun main() = runBlocking {
    launch(start = CoroutineStart.UNDISPATCHED) {
        println("【${Thread.currentThread().name}】协程开始")
        delay(100)
        println("【${Thread.currentThread().name}】协程结束")
    }
    println("【${Thread.currentThread().name}】Main函数执行完毕")
}
//【main】协程开始
//【main】Main函数执行完毕
//【main】协程结束
```



```kotlin
fun test() {
    //当启动类型设置成LAZY时，协程不会立即启动，而是手动调用start()后他才会启动。
    val job = launch(UI, CoroutineStart.LAZY) {
        println("hello")
    }
    job.start()
}
```



- 我们在启动协程通过使用lauch函数，通过指定它的第二个参数start来决定协程的启动模式。
- **LAZY**模式很好理解，就是我们在上一篇文章中介绍createCoroutine和startCoroutine的情况，createCoroutine是创建了协程但未执行，而startCoroutine是创建时就执行。
- **DEFAULT**模式和**ATOMIC**模式就是取消时机上的区别，一般我们常用**DEFAULT**模式，**ATOMIC**只有在涉及到cancel的时候才有意义。
- **UNDISPATCHED**模式就是协程体内代码会在当前线程先执行，直到遇到挂起点才跳出，如运行结果2。
  