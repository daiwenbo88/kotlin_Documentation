###### Kotlin - 类型

> <font color=#333333>可空类型 </font>

问号可以加在任何类型的后面来表示这个类型的变量可以存储null引用

所有的检查都发送在编译期 并不会在运行时带来额外的开销

```kotlin
//变量为可以是null
val x:String?=null
val y:Int?

//参数可以为null
fun strLenSafe(s:String?):Int{
  //一旦进行和null的比较 编译器都会记住
  if(null!=s) s.length else 0
  //等同
  s?.length
}
//  java          Kotlin
@Nullable+ Type = Type?
@NotNull + Type = Type
```

><font color=#333333>安全调用符：?. </font>

?. 安全调用符允许你把一次null检查和一次方法调用合并成一个操作

```kotlin
s?.length//等同 if(null!=s) s.length else null

fun printAllCaps(s:String?){
  //如果s为null 返回的是 String?
  val allCaps : String? = s?.toUpperCase()
}
```

> <font color=#333333>Elvis运算符 ?:</font>

```kotlin
fun foo(s:String?){
  val t: String=s ?: ""  //如果s为null 就返回空字符串
}
//Elvis运算符 和安全调用符 一起使用
fun strLenSafe(s:String?):Int=s?.length ?: 0 

fun Person.countryName()= company?.address?.county ?: "Unknown"

//抛出有意义的错误
fun printShippingLabel(person:Person){
 val address = person.company?.address ?: throw IllegalArgumentException("No address")
}
```

><font color=#333333>安全转换 "as?"</font>

as?运算符尝试把值转换为指定类型 如果值不是适合类型就返回null

```kotlin
class Person(val firstName: String, val lasetName: String) {
    override fun equals(other: Any?): Boolean {
        //如果不匹配就返回false
        val otherPerson = other as? Person ?: return false

        return otherPerson.firstName == firstName && otherPerson.lasetName == lasetName;
    }
}
```

><font color=#333333>非空断言"!!"</font>

可以把任何值转换为非空类型

```kotlin
fun ignoreNulls(s:String?){
  printlin(s!!.length)
}
```

> <font color=#333333>let 函数</font>

对表达式求值,检查结果是否为null

```kotlin
fun sendEmailTo(email:String){

}
val email:String?=null
if(null!=email)sendEmailTo(email)

//只有email非空时 let函数才会被调用
email?.let{email->sendEmailTo(email)}

//使用lambda 生成的默认名字
email?.let{sendEmailTo(it)}

fun getTheBestPersonInThWorld():Person?{
  
}
//函数返回为null 永远不会执行
getTheBestPersonInThWorld()?.let{sendEmailTo(it.email)}
```

><font color=#333333>延迟初始化 lateinit</font>

延迟初始化属性都是var

```kotlin
class MyTest{
 private lateinit var myServer:MyServer
 fun setUp(){
     myServer=MyServer()
 }
}
```

><font color=#333333>可空的扩展类型</font>

```kotlin
//可空类型定义扩展函数
fun CharSequence?.isNullOrEmpty(): Boolean {
    return this == null || this.length == 0
}

fun verifyUserInput(input:String?){
  if(input.isNullOrEmpty()){
    .......
  }
}
```

><font color=#333333>参数类型的可空性</font>

Kotlin 中所有泛型类和泛型函数的类型参数默认都是可空的， 任何类型 包括空在内 都可以替换类型参数 这种情况下 使用类型参数作为类型的声明都允许为null 尽管类型参数T并没有用问号结尾

```kotlin
fun <T> printHashCode(t:T){
  printlin(t?.hashCode())
}
printHashCode(null) //T 会推导成Any?

//指定泛型类型为非null上界 参数就不能传null了
fun <T:Any> printHashCode(t:T){
  printlin(t.hashCode())
}
```

