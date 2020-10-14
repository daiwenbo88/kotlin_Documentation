#### 协程上下文和拦截器

#### 上下文（CoroutineContex）

上一篇文章中的Demo中创建和启动协程调用了startCoroutine 函数时需要传入了一个Continuation，Continuation里有两个成员，其中有一个属性叫CoroutineContext，前面提到 CoroutineContext也是一个接口，表示运行上下文或者叫协程上下文，当时因为我们不对它作处理所以给它赋于EmptyCoroutineContext。那么Continuation到底是什么，叫上下文的是不是很高级的东西？其实CoroutineContext就是一个在执行过程中携带数据的载体对象，或者你可以用最简单的理解，它就是一个用Key作索引，Element作元素的集合而已，一般用于数据从协程外层传递协程内部。自定义CoroutineContext一般需要继承AbstractCoroutineContextElement



假设现在我们需要在主线程中传递一个Boolean值到内部，然后在协程内部经过一系列的逻辑处理后返回相应的结果，那么我们上一篇文章的入门Demo可以这样改：

```kotlin
fun main() {
    log("Main函数开始")
    coroutineDo(ParameterContext(true)) {								// 传入一个自定义的Context
        val result = blockFun()
        log("异步方法返回结果：${result}")
        result
    }
    log("Main函数结束")
}
 
fun <T> coroutineDo(coroutineContext: CoroutineContext, block: suspend () -> T) {
    block.startCoroutine(object : Continuation<T> {
        override val context: CoroutineContext = EmptyCoroutineContext + coroutineContext        // 如果你需要多个CoroutineContext还可以使用加号进行添加
        override fun resumeWith(result: Result<T>) {
            log("收到异步结果：${result}")
        }
    })
}
 
 
suspend fun blockFun() = suspendCoroutine<String> { continuation ->
    Thread {
        val isSuccess = continuation.context[ParameterContext]!!.isSuccess			// 获取传入的context元素
        log("异步开始")
        Thread.sleep(2000)
        if (isSuccess) {
            continuation.resumeWith(Result.success("异步请求成功"))
        } else {
            continuation.resumeWith(Result.failure(Exception()))
        }
    }.start()
}
 
class ParameterContext(val isSuccess: Boolean) : AbstractCoroutineContextElement(Key) {	// 创建一个自定义的Context
    companion object Key : CoroutineContext.Key<ParameterContext>
}
 
fun log(msg: String) {
    println("【${Thread.currentThread().name}】$msg")
}
```

新建了一个ParameterContext，用于最外层在协程开始的时候传入，并附加一个isSuccess的值。Context经过startCoroutine里的Coroutine中可通过+来进行add，最后在suspend函数内部通过continuation.context[Key]?.Element的方式获取值

#### 拦截器（ContinuationInterceptor)

ContinuationInterceptor是一个接口，被称为协程控制拦截器，因为它可以对协程上下文所在的协程的Continuation进行拦截，所以它可以用来处理线程的切换。若要使用拦截器就需要在自定义CoroutineContext的基础上再进行继承ContinuationInterceptor接口并实现interceptContinuation函数

###### 拦截的时机

还记得startCoroutine的源码吗？它是接收一个Continuation，接着创建一个新的Continuation，然后再调用了一个intercepted函数

```
public fun <T> (suspend () -> T).startCoroutine(completion: Continuation<T>) {
    createCoroutineUnintercepted(completion).intercepted().resume(Unit)
}
```

intercepted函数会走到ContinuationImpl# intercepted：

```kotlin
internal abstract class ContinuationImpl(completion: Continuation<Any?>?, private val _context: CoroutineContext?) : BaseContinuationImpl(completion) {
    // ……
    public fun intercepted(): Continuation<Any?> = intercepted?: (context[ContinuationInterceptor]?.interceptContinuation(this) ?: this).also { intercepted = it }
    // ……
}
```

通过continuation.context[Key]?. Element便可以获到Context中Key对应的Element对象，上面源码中可见intercepted函数最后会调用到我们自定义的CoroutineContex里的interceptContinuation函数。所以调用startCoroutine函数来启动协程，实际上就是启动了我们自定义CoroutineContex里拦截后的Continuation


#### 拦截器Demo

我们一直在使用协程中，都是通过自己在协程内部去创建Thread进行实现异步逻辑，这样无异于自己切换线程。我们在开始介绍概念时就一直强调协程是没有异步能力，但是拥有切线程的能力，而这个切线程的能力就是通过拦截器来实现的。我们可以在开始协程后通过拦截原来主线程中的Continuation，然后返回一个新的Continuation，在新的Continuation里我们通过线程池来完成我们的异步逻辑。那么我们根据上一节的Demo1可以这样改


```kotlin
fun main() {
    log("Main函数开始")
    coroutineDo(ParameterContext(true)) {
        val result = blockFun()
        log("异步方法返回结果：${result}")
        result
    }
    log("Main函数结束")
}
 
fun <T> coroutineDo(coroutineContext: CoroutineContext, block: suspend () -> T) {
    block.startCoroutine(object : Continuation<T> {
//        override val context: CoroutineContext = EmptyCoroutineContext + coroutineContext
        override val context: CoroutineContext = AsyncContext() + coroutineContext
        override fun resumeWith(result: Result<T>) {
            log("收到异步结果：${result}")
        }
    })
}
 
suspend fun blockFun() = suspendCoroutine<String> { continuation ->
//    Thread {
        val isSuccess = continuation.context[ParameterContext]!!.isSuccess
        log("异步开始")
        Thread.sleep(2000)
        if (isSuccess) {
            continuation.resumeWith(Result.success("异步请求成功"))
        } else {
            continuation.resumeWith(Result.failure(Exception()))
        }
//    }.start()
}
 
class ParameterContext(val isSuccess: Boolean) : AbstractCoroutineContextElement(Key) {
    companion object Key : CoroutineContext.Key<ParameterContext>
}
 
fun log(msg: String) {
    println("【${Thread.currentThread().name}】$msg")
}
 
val singleThreadExecutor = ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, LinkedBlockingQueue<Runnable>())
 
class AsyncContext() : AbstractCoroutineContextElement(ContinuationInterceptor), ContinuationInterceptor {
    override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T> {
        return ThreadPoolContinuation(continuation)
    }
}
 
class ThreadPoolContinuation<T>(private val continuation: Continuation<T>) : Continuation<T> {
    override val context: CoroutineContext = continuation.context
    override fun resumeWith(result: Result<T>) {
        singleThreadExecutor.execute { continuation.resumeWith(result) }
    }
}
```

上面Demo2中前面部分跟上一节的Demo1几乎是一样的，区别在于在startCoroutine传入的Continuation中的CoroutineContext由EmptyCoroutineContext换成了我们自定义的AsyncContext，以及blockFun函数中注释了Thread的使用。从运行结果可见，其效果还是跟前面Demo是一样的。请往下看新增的代码。

- 新增了一个线程池对象singleThreadExecutor，异步逻辑就是通过它来完成的；
- 新增了一个AsyncContext类，上面我们了解到，继承AbstractCoroutineContextElement是为了自定义Context，继承ContinuationInterceptor是为了对Continuation进行拦截，里面的interceptContinuation函数就是接收原来的Continuation，然后返回一个新的Continuation。
- 新增ThreadPoolContinuation类，它就是新返回的Continuation，它在resumeWith函数中使用了线程池来完成工作逻辑。
  

#### 封装

```kotlin
fun main(){
    coroutineDo(ParameterContext(true)){
        try {
            val result = blockDo {
                val isSuccess = this[ParameterContext02]!!.isSuccess
                blockFun(isSuccess)
            }
            log("异步方法返回结果：${result}")
            result
        }catch (e:Exception){
            e.printStackTrace()
        }
    }
}
//传值到协程内部 使用 CoroutineContext
fun <T> coroutineDo(coroutineContext:CoroutineContext,block: suspend () -> T) {
    block.startCoroutine(object : Continuation<T> {
        //创建并启动协程
        override val context: CoroutineContext = AsyncContext()+coroutineContext //如果你需要多个CoroutineContext还可以使用加号进行添加

        override fun resumeWith(result: Result<T>) { // 协程结果统一处理
            log("收到异步结果：${result}")
        }
    })
}

suspend fun <T> blockDo(block:CoroutineContext.()->T)= suspendCoroutine<T> { continuation ->
    try{
        continuation.resumeWith(Result.success(block(continuation.context)))
    }catch (e:Exception){
        continuation.resumeWith(Result.failure(Exception()))
    }
}

fun blockFun(isSuccess: Boolean):String {
    log("异步开始")
    Thread.sleep(2000)
    if (isSuccess) {
        return "异步请求成功"
    } else {
        throw Exception()

    }
}

class ParameterContext(val isSuccess:Boolean):AbstractCoroutineContextElement(Key){
    companion object Key:CoroutineContext.Key<ParameterContext02>
}

val singleThreadException= ThreadPoolExecutor(1,1,0L, TimeUnit.MILLISECONDS,LinkedBlockingDeque<Runnable>())

//继承 AbstractCoroutineContextElement 是为了自定义Context 继承ContinuationInterceptor 是为了对Continuation 进行拦截
class AsyncContext():AbstractCoroutineContextElement(ContinuationInterceptor),ContinuationInterceptor{
    override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T> {
        return ThreadPoolContinuation(continuation)
    }
}
class ThreadPoolContinuation<T>(private val continuation: Continuation<T>):Continuation<T>{
    override val context: CoroutineContext = continuation.context
    override fun resumeWith(result: Result<T>) {
        singleThreadException.execute { continuation.resumeWith(result) }
    }
}
```

