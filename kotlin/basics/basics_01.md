###### Kotlin 基础

> <font color=#333333> hello world </font>

```kotlin
fun main(args:array<String>){
	println("Hello world!")
}
```

> <font color=#333333>函数</font>

```kotlin
fun max(a:Int,b:Int):Int{
  return if(a>b) a esle b
}

//参数列表后面跟着返回值类型
// if 在kotlin中作为表达式 表达式有返回值 一般最后一句作为返回值

fun max(a:Int,b:Int)=if(a>b) a else b
//表达式函数体 通过类型推导出返回值类型
```

><font color=#333333> 变量</font>

```kotlin
val question="String"
var answer :Int =42
//类型可以不用显示指定  
//val(value) --不可变引用 不能在初始化后再次赋值
//var(variable) --可变引用 允许改变自己的值 不能改变类型
```

><font color=#333333> 字符串</font>

  可以使用EL 表达式 在字符串中进行拼接

```kotlin
val question="String"
println("this is $question")
println("this is ${question}")
```

><font color=#333333> 类和属性</font>

```kotlin
class Person(var name:String,val age:Int)
//var 属性 会生成 getter 和 setter 
//val 属性 只会生成 getter
//属性名 is开头 setter会被替换成set  getter不会增加任何前缀

class Rectangle(val height:Int,val width:Int){
  val isSquare:Boolean
       get(){
         return height==width
       }
  		//声明属性的getter
}
val person=Person("Alice", 30)
//:: 成员引用
val createPerson = ::Person04 //引用构造方法
val p = createPerson("Alice", 30)
println(p)
```

><font color=#333333> 枚举</font>

```kotlin
/**声明代属性的枚举*/
enum class Color(val r: Int, val g: Int, var b: Int) {
    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0)
}
```

><font color=#333333> 智能转换</font>

检查过一个变量是某种类型，后面就不需要转换了，可以把它当做检查过的类型使用

when 匹配到的代码块中的最后一个表达式就是结果

```kotlin
interface Expr

class Num(val value: Int) : Expr

class Sum(val left: Expr, val right: Expr) : Expr

fun eval(e: Expr): Int {
    if (e is Num) {//is检查 e 是否为Num类型
        val n = e as Num //多余转换
        return n.value
    }
    if (e is Sum) {
        return eval(e.left) + eval(e.right)
    }
    throw IllegalAccessException("这不是一个数")
}

println(eval(Sum(Sum(Num(1), Num(2)), Num(4))))
//val n = e as Num 显示的转换

//使用 when 替代if
fun eval03(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval03(e.left) + eval03(e.right)
        else -> throw IllegalAccessException("这不是一个数")
    }
```

><font color=#333333> 遍历</font>

in ：检查区间

downTo：递减

step ：步长

```kotlin
while(condition){
	/*循环体*/
}

do{
  /*循环体*/
}while(condition)

for(x in 0 until size)//0到size-1
for(x in 100 downTo 1 step 2) //downTo递减 step 步长

val binaryReps = TreeMap<Char, String>()
for (c in 'A'..'F') {
    val binary = Integer.toBinaryString(c.toInt())
    //map赋值
    binaryReps[c] = binary
}
//map遍历
for ((letter, binary) in binaryReps) {
    println("$letter=$binary")
}

val list = arrayListOf<String>("10", "11", "1001")
//遍历集合
for ((index, element) in list.withIndex()) {
     println("$index:$element")
}

// in 检查区间
fun isLetter(c: Char): Boolean {
    return c in 'a'..'z' || c in 'A'..'F'
}
// in 用在 when 场景
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "it"
    in 'a'..'z',
    in 'A'..'Z' -> "letter"
    else -> "I dot know"
}

fun isNotDigit(c: Char): Boolean {
    return c !in '0'..'9'
}
```

> <font color=#333333> try  Catch  finally</font>

在声明函数时可以不抛出异常

```kotlin
//try 作为表达式 使用
fun readNumber02(reader: BufferedReader){
   var number= try {
        var line = reader.readLine()
              Integer.parseInt(line)//最后一句作为返回值
    } catch (e: NumberFormatException) {
        return
    } finally {
        reader.close()
    }
}
```

