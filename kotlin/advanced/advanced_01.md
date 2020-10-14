###### Kotlin - 高阶函数

> <font color=#333333>声明高阶函数 </font>

高阶函数：是以另外一个函数作为参数或者返回值的函数,  Kotlin中函数可以使用lambda或者函数引用来表示

- **函数类型**

```kotlin
//函数引用 
val sum={x:Int,y:Int->x+y}
val action={println(42)}

//类型推导后
val sum:(Int,Int)->Int={x,y->x+y}
val action:()->Unit={println(42)}

//返回值为null的函数
var canReturnNull:(Int,Int)->Int?={null}
//函数类型的可空变量
var funOrNull:((Int,Int)->Int)?=null


```



- **调用作为参数的函数**

```kotlin
fun twoAndThree(operation:(Int,Int)->Int){
  val result=operation(2,3) //调用函数类型参数
  println("The result is $result")
}
twoAndThree{a,b->a+b}
//5
twoAndThree{a,b->a*b}
//6

//扩展函数
fun String.fiter(predicate: (Char) -> Boolean): String {
    var stringBuilder = StringBuilder()
    for (index in 0 until length) {
        var element = get(index)
        if (predicate(element)) {
            stringBuilder.append(element)
        }
    }
    return stringBuilder.toString()
}
println("ab1c".fiter{ it in 'a'..'z' })//abc
```

- **与java互调**

```kotlin
//kotlin
fun processTheAnswer(f:(Int)->Int){
 println(f(42))
}
//java 调用
processTheAnswer(number->number+1)
//编译后的代码
processTheAnswer(new Function1<Integer,Integer>(){
  @Override
  public Integer invoke(Integer number){
    System.out.println(number);
    return number+1;
  }
});
```

- **函数参数的默认值和null值**

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ",",
    prefix: String = "",
    postfix: String = "",
    transfoem: ((T) -> String)? = null //默认值为null
): String {
    var result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) {
        if (index > 0) result.append(separator)
        var str = transfoem?.invoke(element) ?: element.toString()
        result.append(str)
    }
    result.append(postfix)
    return result.toString()
}

val letters = listOf("Alpha", "Beta")
println(letters.joinToString(separator="{",prefix="!",postfix="}"transfoem={ it.toUpperCase()})
//{ALPHA!BETA}        

```

- **返回函数的函数**

从函数中返回另外一个函数

```kotlin
enum class Delivery { STANDARD, EXPEDITED }

class Order(val item: Int) 

//返回一个函数
fun getShippingCostCalculator(delivery: Delivery): (Order) -> Double {
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.item }
    }
    return { order -> 2.1 * order.item }
}

var shippingCostCalculator = getShippingCostCalculator(Delivery.EXPEDITED)
println("shipping costs ${shippingCostCalculator(Order(3))}")//调用返回函数
```

- **内联函数消除Lambda带来的运行时性能开开销**

Lambda 在运行时会编译为匿名类，没调用一次Lambda表达式一个额外的就会被创建，如果Lambda捕捉了某个变量 每次调用都会创建一个新的对象 带来额外的运行开销

inline：使用inline修饰符标记一个函数 在函数被使用时编译器并不会生成函数调用代码 而是使用函数实现的真实代码替换每一次函数的调用

```kotlin
// inline 标记该函数为内联函数
inline fun<T> synchronized(lock:Lock,action:()->T):T{
 lock.lock()
  try{
    return action()
  }finally{
    lock.unlock()
  }
}
```



```kotlin

fun foo(l:Look){
 println("Before sync")
 synchronized(l){
   println("Action")
 } 
 println("After sync")
}

// 编译后的代码 （函数实现的真实代码替换每一次函数的调用）
fun _foo_(l:Lock){
  println("Before sync")
  // 内联代码块
  l.lock()
  try{
    println("Action")
  }finally{
    l.unlock()
  }
  // 内联代码块
  println("After sync")
}
```

传递函数类型的变量作为参数

```kotlin
class LockOwner(val lock:Lock){
  fun runUnderLock(body:()->Unit){
   synchronized(lock,body)
  }
}

```



- **内联集合操作**

大部分集合标准库函数都被声明为内联函数

```kotlin
class Person(val name: String, age: Int)

val people=listOf(Person("Alice",29),Person("Bob",29))

println(people.filter(it.age<30))

//编译成如下代码
val result=mutableListOf<Person>()
for(person in people){
  if(person.age<30)result.add(person)
}
println(result)
```

- **高阶函数控制流**

在lambda中使用`return`关键字，它会从<font color=#4CAF50 >调用lambda的函数</font>返回 并不是只从lambda中返回 这样的`return`语句叫作<font color=#4CAF50 >非局部返回</font> 前提是在内联函数中使用lambda

在一个非内联函数的lambda中使用`return`是不允许的 因为可以把非内联函数的lambda保存在一个变量中，以便在函数返回后可以继续使用 这个时候lambda想去影响函数的结果返回已经太晚

```kotlin
class Person(val name: String, age: Int)

val people=listOf(Person("Alice",29),Person("Bob",29))

fun lookForAlice(people:List<Person>){
  for(person in people){
     if(person.name=="Alice"){
        println("Found")
        return
     }
  }
   println("Alice is not found")
}

fun lookForAlice02(people:List<Person12>){
    people.forEach {
        if (it.name=="Alice"){
            println("Found!")
            return //从调用的函数返回 也就是lookForAlice02
        }
    }
    println("Alice is not found")
}

lookForAlice(people)//Found
lookForAlice02(people)//Found

```

- **从Lambda 返回：使用标签返回**

```kotlin
fun lookForAlice(people:List<Person>){
  //定义Lambda 标签 lable@
    people.forEach lable@{
      //只从Lambda 返回
        if (it.name=="Alice")return@lable
    }
    println("Alice is not found")
}

//如果指定了Lambda表达式标签 再用函数名作为标签没有任何效果
fun lookForAlice02(people:List<Person>){
  //函数名作为return 标签
    people.forEach{
      //只从Lambda 返回
        if (it.name=="Alice")return@forEach
    }
    println("Alice is not found")
}

lookForAlice(people) //Alice is not found
```

- **匿名函数 :默认使用局部返回**

匿名函数中的`return` 从 匿名函数返回

Lambda表达式的`return`从调用它的函数返回

```kotlin
fun lookForAlice02(people:List<Person>){
  //使用匿名函数替代Lambda
    people.forEach(fun (person){
     		if (person.name=="Alice")return //return 只向最近一个函数 匿名函数
        println("${person.name} is not Alice")
    })
}
lookForAlice(people)//Bob is not Alice

//匿名函数需要显示指定 返回值类型
people.fiflter(fun (person):Boolean{
  return person.age<30
})
//Lambda 不需要显示指定
people.fiflter(fun (person)= person.age<30)
```

