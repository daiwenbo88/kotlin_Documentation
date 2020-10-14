###### Kotlin - 函数的定义和调用



> <font color=#333333>创建集合 </font>

```kotlin
    //创建HashSet集合
    val set = hashSetOf(1, 2, 53)
    //创建ArrayList集合
    var list = arrayListOf<Int>(1, 7, 53)
    //创建map集合
    val map = hashMapOf(1 to "one", 7 to "seven", 53 to "fifty-three")
    println(set.javaClass)
    //HashSet
    println(list.javaClass)
    //ArrayList
    println(map.javaClass)
    //HashMap
```

><font color=#333333>函数之 命名参数 和 默认参数 </font>

必须按照函数中声明定义的参数顺序来给定参数

```kotlin
/**
 * collection 集合
 * separator 分割符
 * prefix 前缀 赋值默认值
 * postfix 后缀 赋值默认值
 */
fun <T> joinToString(collection:Collection<T>,separator:String,prefix:String="{",postfix:String="}"):String{
    val result= StringBuilder(prefix)
    for((index,element) in collection.withIndex()){
        if (index>0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
joinToString(list,separator=",",prefix=",",postfix=",")//命名方式调用
joinToString(list,",","","")
joinToString(list)
joinToString(list, ";")
```

><font color=#333333>顶层函数和属性 </font>

会把这个文件名编译成类名 对应的所有的顶层函数会编译为这个类的静态函数 顶层属性会被存储到一个静态变量中

```kotlin
@file:JvmName("StringUtils")//改变生成java文件顶层类名
package strings

//顶层属性 会被存储到一个静态变量中
var opCunt=0

fun performOperation(){
  opCunt++
}
fun performOperationCount(){
  println("Operation Count $opCunt")
}

//const 类似于静态常量
const  val UNIX_LINE_SEPARATOR="\n"
```

><font color=#333333>扩展函数和属性 </font>

为一个类添加成员函数 或者成员变量

```kotlin
//给String类添加扩展属性 lastChar
//因为没有字段支持存值 所以不支持初始化 val类型必须提供getter 
val String.lastChar:Char
      get()=get(length-1)
println("Kotlin".lastChar)



//String 接受者类型 this 接受者对象
fun String.lastChar():Char=this.get(this.length-1)
//调用 String是接受者类型 "Kotlin"是接受者对象
println("Kotlin".lastChar())



//给StringBuilder类添加扩展属性
//因为没有字段支持存值 所以不支持初始化 var类型必须提供getter 和setter
var StringBuilder.builderLastChar:Char
    get()=get(length-1)
    set(value:Char) {
      this.setCharAt(length-1,value)
}

//调用
var sb: StringBuilder = StringBuilder("Kotlin?")
sb.builderLastChar = '!'
println(sb)

/**
 * collection 集合
 * separator 分割符
 * prefix 前缀 赋值默认值
 * postfix 后缀 赋值默认值
 */
fun <T> Collection<T>.joinToString(separator:String,prefix:String="{",postfix:String="}"):String{
    val result= StringBuilder(prefix)
    for((index,element) in this.withIndex()){
        if (index>0) result.append(separator)
        result.append(element)
    }
    result.append(postfix)
    return result.toString()
}
//调用
list.joinToString(separator="(",prefix=",",postfix=")")//命名方式调用
```

><font color=#333333>扩展函数不能重写 </font>

扩展函数会编译为静态函数

扩展函数和成员函数同名时  会调用扩展函数

```kotlin
open class View{
  open fun click()=println("View click")
}
class Buttons : View {
  override fun click()=println("Buttons click")
}
fun View.showOff()=println("View showOff")
fun Buttons.showOff()=println("Buttons showOff")

val view:View=Button()
view.click()
view.showOff()
//Buttons click 根据对象决定调哪一个
//View showOff 根据类型觉定掉哪一个

```

><font color=#333333>可变参数 、中缀调用、解构声明 </font>

可变参数

```kotlin
val list=listOf(2,3,5,7)
//vararg 可变参数关键字
fun listOf<T>(vararg values:T):List<T>{
  ...
}

//当数组传递给可变参数函数时 需要显示添加*(展开运算符)
fun main(args:Array<String>){
  //println 也支持多参数传递
  println("args":*args)
}
```

中缀调用

```kotlin
val map=mapOf(1 to "one",7 to "seven",53 to "fifty-three")
//1 to "one" 使用中缀符号调用to函数
//infix 关键字声明改函数可以使用中缀符号调用
infix fun Any.to(other:Any)=Pair(this,other)
```

解构声明 

```kotlin
val(number,name)=1 to "one"
```

> <font color=#333333>局部函数 </font>

在函数中添加函数 在函数中调用 这种写法通常使用在 `会在某些条件下触发递归的方法内`或者是`不希望外部其他函数访问到的函数`，在一般情况下是不推荐使用嵌套函数的

```kotlin
class User(val id: Int, val name: String, val address: String)

//扩展函数
fun User.validateBeforeSave() {
    //局部函数 
    fun validate(value: String, fieldName: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("Can't save user ${id}:empty $fieldName")
        }
    }
  	//扩展函数可以直接访问 对象类型的属性
    validate(name, "Name")
    validate(address, "Address")
}
//调用
val user = User(1, "LiLei", " ")
saveUser(user)

fun saveUser(user: User) {
    user.validateBeforeSave()
}
```

