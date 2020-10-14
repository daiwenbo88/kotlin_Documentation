#### 协程官方框架Channel、Select、Flow

#### **Channel**

Channel我们一般翻译成叫通道，用于多个协程之间进行数据相互传输,多个协程允许发送和接收同一个Channel的数据。它类似于线程任务队列BlockingQueue  + 挂起函数的支持，因为如果通道支持缓存的话，那么它实质上就是一个队列。我们发消息和接收消息都是挂起函数，挂起取决于Channel的状态，如果Channel已经满了，那么Send时就会被挂起，如果Channel里什么都没有的话，那么Receive时也会被挂起。

#### Channle的分类

```kotlin
public fun <E> Channel(capacity: Int = RENDEZVOUS): Channel<E> =
    when (capacity) {
        RENDEZVOUS -> RendezvousChannel()
        UNLIMITED -> LinkedListChannel()
        CONFLATED -> ConflatedChannel()
        BUFFERED -> ArrayChannel(CHANNEL_DEFAULT_CAPACITY)
        else -> ArrayChannel(capacity)
    }
```

**RENDEZVOUS**

表示约会形式的等待，send调用后就会一直挂起，直到receive到达。

**UNLIMITED**

表示执行缓存无限容量，send调用后就存放在channel里直接返回，不管是否有receive。但是我们在使用时还是需要注意内存情况。

**CONFLATED**

表示保留最新，send调用后就存放在channel里直接返回，但是channel里只能存放最近一次send的值。

**BUFFERED**

表示执行缓存使用默认容量，默认是64。

**FIXED**

表示执行缓存使用固定容量，跟BUFFERED一样，只是容量值是通过参数自己传入

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>(Channel.RENDEZVOUS)
    val producer = GlobalScope.launch {
        for (i in 0..3) {
            println("【${Thread.currentThread().name}】准备发送 $i")
            channel.send(i)
            println("【${Thread.currentThread().name}】发送完毕 $i")
        }
        channel.close()
    }
    val consumer = GlobalScope.launch {
        while (!channel.isClosedForReceive) {           // 还可以继续接收
            println("【${Thread.currentThread().name}】准备接收")
            val value = channel.receiveOrNull()   // 跟receive的区别在于使用receive话channel如果被close会抛出异常
            println("【${Thread.currentThread().name}】接收完毕 $value")
        }
    }
    producer.join()
    consumer.join()
}
```

- 有些情况下打印顺序不一致，那是因为两个协程是运行在两个线程中，它们的执行顺序取决于CPU对线程的调度，但是能保证的是，每次发送后，如果没有接收定会被挂起，如果没有发送，那么接收地方一定会被挂起

- 调用Channel的close函数后，通道就会被关闭。

- 通道关闭后，isClosedForSend函数就会返回true，此时如果继续调用send函数就会抛出ClosedSendChannelException异常。

- 通道关闭后，调用receive仍可接收缓存的数据，直到缓存数据消费完后，isClosedForReceive函数就会返回true，此时继续调用receive函数就会抛出ClosedReceiveChannelExceptoin异常

#### **Channel的迭代**

`Channe`类似于`BlockingQueue` ，所以它可以进行循环迭代也是情理之中， 它内部实现了`ChannelIterator`接口的挂起函数`hasNext`。

**hasNext在有缓存的数据时会返回true**；

**hasNext在未关闭且缓存为空时挂起**；

**hasNext在正常关闭且缓存为空时返回false**。

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>(Channel.RENDEZVOUS)
    val producer = GlobalScope.launch {
        for (i in 0..3) {
            println("【${Thread.currentThread().name}】准备发送 $i")
            channel.send(i)
            println("【${Thread.currentThread().name}】发送完毕 $i")
        }
        channel.close()
    }
    val consumer = GlobalScope.launch {
       for (i in channel) {
            println("【${Thread.currentThread().name}】接收 $i")
        }
    }
    producer.join()
    consumer.join()
}
```

#### **Channel的协程Buidler（SendChannel / ReceiveChannel）**

通过一个生产者协程producer和一个消费者协程consumer进行了数据的send和receive，而在官方框架中也专门为生产者协程和消费者协程提供了两个函数来构建出协程，它们就是produce和actor。而且通过produce和actor函数启动的协程结束后都会自动关闭对应的Channel。

- **produce**：启动一个生产者协程，返回ReceiveChannel。

- **actor**：启动一个消息者协程，返回SendChannel**（注意，actor函数目前框架中是被标为废弃）**

**ReceiveChannel**

```kotlin
fun main() = runBlocking {
    val receiveChannel = GlobalScope.produce(capacity = Channel.RENDEZVOUS) {
        for (i in 0..3) {
            println("【${Thread.currentThread().name}】准备发送 $i")
            send(i)		// 等价于channel.send(i)
            println("【${Thread.currentThread().name}】发送完毕 $i")
        }
    }
    val consumer = GlobalScope.launch {
        for (i in receiveChannel) {
            println("【${Thread.currentThread().name}】接收 $i")
        }
    }
    consumer.join()
}
```

**SendChannel**

```kotlin
fun main() = runBlocking {
    val sendChannel = GlobalScope.actor<Int>(capacity = Channel.RENDEZVOUS ) {
        for (i in this) {
            println("【${Thread.currentThread().name}】接收 $i")
        }
    }
 
    val producer = GlobalScope.launch {
        for (i in 0..3) {
            println("【${Thread.currentThread().name}】准备发送 $i")
            sendChannel.send(i)
            println("【${Thread.currentThread().name}】发送完毕 $i")
        }
    }
    producer.join()
}
```

**BroadcastChannel**

需要一对多的话那就需要`BroadcastChannel`，它会像我们平时使用广播一样进行分发给所有订阅者。另外需要注意的是，`BroadcastChannel`不支持**RENDEZVOUS**。

```kotlin
fun main() = runBlocking {
    val broadcastChannel = GlobalScope.broadcast {
        for (i in 0..3) {
            println("【${Thread.currentThread().name}】准备发送 $i")
            send(i)
            println("【${Thread.currentThread().name}】发送完毕 $i")
        }
    }
    List(3){index ->
        GlobalScope.launch{
            for (i in broadcastChannel.openSubscription()) {
                println("【${Thread.currentThread().name}】协程$index 接收 $i")
            }
        }
 
    }.joinAll()
}
```

## Select

Select一般是IO多路复用的概念，而在协程的Select则是用于挂起函数的多路复用。通俗一点表达就是可以同时进行多个挂起函数的调用，但最后只选择执行最快的挂起函数的返回结果

```kotlin
fun main() = runBlocking {
    val one = async(Dispatchers.Default) { doOne() }
    val two = doTwo()
    select<Unit> { // Unit表示 select 表达式不返回任何结果
        one.onAwait { value ->
            println("【${Thread.currentThread().name}】one -> $value")
        }
        two.onReceive { value ->
            println("【${Thread.currentThread().name}】two -> $value")
        }
    }
}
suspend fun doOne(): Int {
    delay(1000)
    println("【${Thread.currentThread().name}】doOne 计算中")
    return 1
}
fun doTwo() = GlobalScope.produce<Int> {
    delay(500)
    println("【${Thread.currentThread().name}】doTwo 计算中")
    send(2)
}

//【DefaultDispatcher-worker-2】doTwo 计算中
//【main】two -> 2
//【DefaultDispatcher-worker-2】doOne 计算中
```



```kotlin
fun main() = runBlocking {
    val one = async(Dispatchers.Default) { doOne() }
    val two = doTwo()
    select<Unit> { // Unit表示 select 表达式不返回任何结果
        one.onAwait { value ->
            println("【${Thread.currentThread().name}】one -> $value")
        }
        two.onReceive { value ->
            println("【${Thread.currentThread().name}】two -> $value")
        }
    }
}
suspend fun doOne(): Int {
    delay(500)
    println("【${Thread.currentThread().name}】doOne 计算中")
    return 1
}
fun doTwo() = GlobalScope.produce<Int> {
    delay(1000)
    println("【${Thread.currentThread().name}】doTwo 计算中")
    send(2)
}

//【DefaultDispatcher-worker-1】doOne 计算中
//【main】one -> 1
```

- 示例1和示例2仅仅是两个挂起函数delay时长的区别，在示例1中，doTwo比doOne函数快，所以打印出2，示例2中，它们的delay时长刚才相反，所以打印出的值是1。
- 我们从两个运行结果中还发现，运行结果1打印了3行，表示需然最终采纳的结果是onOne的值，但是onTwo还是坚持执行完了。而运行结果2打印了2行，表示最终采纳的结果是onTwo的值，但是onOne被中止了。
- 在调用doOne函数使用了async返回了一个Deferred，所以我们可以使用.await()对它进行结果的等待，而在select中变成相应的onAwait()。
- 在调用doTwo函数时，因为它是一个Channle，所以在select中使用了onReceive对其进行结果接收。

## Flow

**Flow**异步流，它就可以在挂起函数处理异步计算时返回多个结果。它在使用上跟`sequence`（序列）非常像，`sequence`是协程语言级的API，`sequence`不能使用delay，它只会阻塞当前线程

```kotlin
fun main() = runBlocking {
    val foo = flow {
        for (i in 1..3) {
            println("【${Thread.currentThread().name}】flow $i")
            emit(i)
            delay(100)
        }
    }
    foo.collect{ value -> println("【${Thread.currentThread().name}】collect $value") }
}

//【main】flow 1
//【main】collect 1
//【main】flow 2
//【main】collect 2
//【main】flow 3
//【main】collect 3
```

- 使用flow需要import kotlinx.coroutines.flow.*。

- 使用的**collect**函数触发flow里代码的执行从而读flow内发射回来的值。

### Flow的创建

```kotlin
fun main() = runBlocking {
    listOf(1, 2, 3).asFlow()
            .onEach {
                delay(100)
            }.collect {
                println("通过 asFlow 创建的 Flow $it")
            }
 
    flowOf(1, 2, 3)
            .onEach {
                delay(100)
            }
            .collect {
                println("通过 flowOf 创建的 Flow $it")
            }
 
    channelFlow {
        for (i in 1..3) {
            delay(100)
            send(i)
        }
    }.collect {
        println("通过 channelFlow 创建的 Flow $it")
    }
}
//通过 asFlow 创建的 Flow 1
//通过 asFlow 创建的 Flow 2
//通过 asFlow 创建的 Flow 3
//通过 flowOf 创建的 Flow 1
//通过 flowOf 创建的 Flow 2
//通过 flowOf 创建的 Flow 3
//通过 channelFlow 创建的 Flow 1
//通过 channelFlow 创建的 Flow 2
//通过 channelFlow 创建的 Flow 3
```

- 通过**asFlow**和**flowOf**对集合进行创建Flow。

- 通过**channelFlow**可以从Channel创建Flow

### Flow使用调度器切换线程

```kotlin
fun main() = runBlocking {
    val foo = flow {
        for (i in 1..3) {
            println("【${Thread.currentThread().name}】flow $i")
            emit(i)
            delay(100)
        }
    }
    foo.flowOn(Dispatchers.IO).collect{ value -> println("【${Thread.currentThread().name}】collect $value") }
}
//【DefaultDispatcher-worker-1】flow 1
//【main】collect 1
//【DefaultDispatcher-worker-1】flow 2
//【main】collect 2
//【DefaultDispatcher-worker-1】flow 3
//【main】collect 3
```

- 通过**flowOn**函数，跟使用launch一样传入相应的调度器就可以进行线程的切换。

### Flow的异常处理

```kotlin
fun main() = runBlocking {
    val foo = flow {
        emit(1)
        throw ArithmeticException("计算异常了")
        emit(2)
    }.catch { t:Throwable->
        println("【${Thread.currentThread().name}】catch error: $t")
        emit(-1)
    }.onCompletion { t:Throwable?->
        println("【${Thread.currentThread().name}】onCompletion: $t")
    }
 
    foo.collect{ value -> println("【${Thread.currentThread().name}】collect $value") }
}
//【main】collect 1
//【main】catch error: java.lang.ArithmeticException: 计算异常了
//【main】collect -1
//【main】onCompletion: null
```

- flow表达式后可以直接通过.catch进行异常的捕获，但不包括取消异常，因为取消操作属于正常逻辑并不算真正意义上的异常。
- onCompletion类似于我们平时异常捕获中的finally，它是一定会执行的，t是否为null取决于是否有异常和是否前面catch是否有将异常捕获。
- 如果我们在flow { ... } 构建器内部的 try/catch来捕获异常也是可以的，但是我们不建议这样做，因为会违反异常透明性的，而且这样做我们并不能在catch中继续使用emit来发射值。

### Flow的取消

Flow本身并没有取消的API，因为Flow的运行依赖于协程，Flow的取消取决于collect所在的协程的取消，collect作为挂起函数可以响应所在协程的取消状态

```kotlin
fun main() = runBlocking {
    val foo = flow {
        emit(1)
        delay(1000)
        emit(2)
    }
 
    withTimeoutOrNull(200) {
        foo.collect{ value -> println("【${Thread.currentThread().name}】collect $value") }
    }
    println("【${Thread.currentThread().name}】Main函数结束")
}
//【main】collect 1
//【main】Main函数结束
```



### Flow元素并发问题

在创建一个Flow后想在里面进行通过调度器切换线程是不允许的，因为emit本身并不是线程安全的。如果你非要这样做的话，可以选择使用channelFlow来创建Flow，因为Channel是一个并发安全的消息通道，send本身是线程安全的

```kotlin
fun main() = runBlocking {
    channelFlow {
        send(1)
        withContext(Dispatchers.IO) {
            send(2)
        }
    }.collect{ value -> println("【${Thread.currentThread().name}】collect $value") }
}
//【main】collect 1
//【main】collect 2
```

### Flow的缓冲

当发射太快而消费太慢的时候，由于消费的速度跟不上发射的速度，这时就会影响到后面结果的发射

```kotlin
fun main() = runBlocking {
    flow {
        for (i in 1..3) {
            delay(100)
            println("【${Thread.currentThread().name}】flow $i")
            emit(i)
        }
    }.collect { value ->
        delay(1000)
        println("【${Thread.currentThread().name}】collect $value")
    }
}
//【main】flow 1
//【main】collect 1
//【main】flow 2
//【main】collect 2
//【main】flow 3
//【main】collect 3

fun main() = runBlocking {
    flow {
        for (i in 1..3) {
            delay(100)
            println("【${Thread.currentThread().name}】flow $i")
            emit(i)
        }
    }.buffer().collect { value ->
        delay(1000)
        println("【${Thread.currentThread().name}】collect $value")
    }
}
//【main】flow 1
//【main】flow 2
//【main】flow 3
//【main】collect 1
//【main】collect 2
//【main】collect 3
```

- 加入了**buffer**后，当消费未完成时，先由buffer来缓冲发射项，这样往后需要发射的结果就无需等待。

### Flow的背压问题

buffer仅能缓解发射太快而消费太慢的问题，但是它还是会存在buffer满了的情况。这类背压的问题还可以使用conflate或者collectLatest来进行解决。

````kotlin
fun main() = runBlocking {
    flow {
        for (i in 1..3) {
            delay(100)
            println("【${Thread.currentThread().name}】flow $i")
            emit(i)
        }
    }.conflate().collect { value ->
        delay(1000)
        println("【${Thread.currentThread().name}】collect $value")
    }
}
//【main】flow 1
//【main】flow 2
//【main】flow 3
//【main】collect 1
//【main】collect 3
````

- 使用**conflate**合并是加快处理速度的一种方式。它通过删除发射值来实现。 另一种方式就是使用**collectLatest**取消缓慢的收集器，并在每次发射新值的时候重新启动它

### Flow的组合

**zip组合两个流的值**

```kotlin
fun main() = runBlocking {
    val nums = (1..3).asFlow()
    val strs = flowOf("one", "two", "three")
 
    nums.zip(strs) { a, b ->
        "$a -> $b"
    }.collect { value ->
        println("【${Thread.currentThread().name}】$value")
    }
}

//【main】1 -> one
//【main】2 -> two
//【main】3 -> three
```

**combine结合计算**

```kotlin
fun main() = runBlocking {
    val nums = (1..3).asFlow()
            .onEach {
                delay(300)
                println("【${Thread.currentThread().name}】nums: $it")
            }
    val strs = flowOf("one", "two", "three")
            .onEach {
                delay(400)
                println("【${Thread.currentThread().name}】strs: $it")
            }
 
    nums.combine(strs) { a, b ->
        "$a -> $b"
    }.collect { value ->
        println("【${Thread.currentThread().name}】$value")
    }
}
//【main】nums: 1
//【main】strs: one
//【main】1 -> one
//【main】nums: 2
//【main】2 -> one
//【main】strs: two
//【main】2 -> two
//【main】nums: 3
//【main】3 -> two
//【main】strs: three
//【main】3 -> three
```

- 假如两个流执行的时间并非一致，将zip换成**combine**后，每当流产生值的时候都需要重新计算

