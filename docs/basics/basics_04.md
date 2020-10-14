###### Kotlin - Lambda

><font color=#333333>SAM 单抽象方法 </font>

只有一个抽象方法的接口

```kotlin
interface Clickable {
    fun click()
}
```

函数试接口

```kotlin
fun postponeComputation(delay:Int,runnable:Runnable)

//对象表达式 作为函数式接口的实现传递 (每次使用都会新的实例)
postponeComputation(1000,object:Runnable{
  override fun run(){
    println(42)
  }
})

//如果lambda没有访问任何来着定义它的函数的变量 相应的匿名实例可以在多次调用之间重复使用
postponeComputation(1000){println(42)}//整个程序只会创建一个Runnable实例

val runnable=Runnable{println(42)}
postponeComputation(1000,runnable)

//lambda从包围它的作用域中捕获了变量，每次调用就不再可能用同一个实例了
fun handleComputation(id:String){
  postponeComputation(1000){println(id)}
}
```

> <font color=#333333>Lambda:作为函数参数的代码块 </font>

```kotlin
data class Person(val name: String, val age: Int)

val people = listOf<Person>(
        Person04("HanMeiMei", 29),
        Person04("LiLei", 20),
        Person04("Alice", 29),
        Person04("Bob", 31)
    )

println(people.maxBy ({ p:Person->p.age }))//Lambda代码块作为参数传递
println(people.maxBy (){ p:Person->p.age })//如果Lambda表达式是调用函数的最后一个参数，可以放到括号外面
println(people.maxBy { p:Person->p.age})//如果Lambda表达式是调用函数的唯一个参数，可以把参数括号去掉
println(people.maxBy { p -> p.age})//推导出类型参数
println(people.maxBy { it.age})//没有指定实参名称时 kotlin 默认生成it为参数名称
```

><font color=#333333>语法</font>

当lambda 赋值给一个变量时 必须显示的指定参数类型

```kotlin
//         参数部分        函数体
val sum={ x:Int, y:Int -> x + y }
//始终在花括号内 可以赋值给一个变量 
println(sum(1,2))
```

在函数内声明一个匿名内部类的时候，能够在匿名内部类引用这个函数的参数和局部变量 lambda 也可以做到

```kotlin
fun printMessageWithPrefix(messagess:Collection<String>,prefix:String){
    messagess.forEach { println("$prefix$it") }
}
val errors=listOf("403 Forbidden","404 Not Found")
printMessageWithPrefix(errors,"Error")


```

> <font color=#333333>集合api</font>

```kotlin
val list = listOf(1, 2, 3, 4)
println(list.filter { it % 2 == 0 })//filter过滤
println(list.map { it * it })//遍历

val person = listOf(Person04("Alice", 20), Person04("Bob", 31))
val canBeInClub27 = { p: Person04 -> p.age <= 27 }
//检查所有的年纪是否小于27岁
println(people.all(canBeInClub27))
//检查是否有的年纪小于27岁
println(people.any(canBeInClub27))
//检查有多少个的年纪小于27岁
println(people.count(canBeInClub27))
//找到第一个年纪小于27岁
println(people.find(canBeInClub27))
//进行年纪相同的人分组
println(people.groupBy { it.age })//groupBy 返回map age作为key
//使用序列 优化操作符
val reult = people.asSequence()//惰性集合操作
       .map(Person::name)//遍历姓名
       .filter { it.startsWith("A") }//筛选头字母为A的
       .toList()//末端操作被调用了 中间操作才会被执行 因为他们是惰性的
println(reult)

val strList = listOf("a", "ab", "b")
println(strList.groupBy(String::first))

val strs = listOf("abc", "def")
//flatMap 遍历每个元素
println(strs.flatMap { it.toList() })

val numbers = mapOf(0 to "zero", 1 to "one")
println(numbers.mapValues { it.value.toUpperCase() })

```

><font color=#333333>with 函数</font>

对同一个对象进行多次操作 with返回值是lambda代码的结果 也就是最后一行代码

```kotlin
fun alphabet(): String {
    val result = StringBuffer()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }
    result.append("\nNow I Know the alphabet!")
    return result.toString()
}

//with 可以对一个对象多次操作
fun alphabet(): String {
    return with(StringBuffer()) {
        for (letter in 'A'..'Z') {
            append(letter)
        }
        append("\nNow I Know the alphabet!")
        toString()
    }
}

fun alphabet()= with(StringBuffer()) {
        for (letter in 'A'..'Z') {
            append(letter)
        }
        append("\nNow I Know the alphabet!")
        toString()
    }

println(alphabet())
```

> <font color=#333333>apply 函数</font>

apply 始终返回作为实参传递给它的对象（）

```kotlin
//apply 返回操作的对象
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }
    append("\nNow I Know the alphabet!")
}.toString()

//创建TextView 对象
fun createTextView(context:Context)=
   TextView(context).apply{
       text="Sample Text"
       textSize=20.0
       setPadding(10,0,0,0)
   }
```

