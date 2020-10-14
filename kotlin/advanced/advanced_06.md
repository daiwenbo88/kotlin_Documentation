###### Kotlin - DSL

## DSL构建

**领域特定语言(DSL)：**构建API,让代码具有最佳的可读性和可维护性 减少代码中的语法噪声

**DSL完全静态类型：**

|             常规语法              |               整洁语法                |    用到的功能    |
| :-------------------------------: | :-----------------------------------: | :--------------: |
|     StringUtil.capitalize(s)      |            s.capitalize()             |     扩展函数     |
|            1.to("one")            |              1 to "one"               |     中缀调用     |
|            set.add(2)             |                set+=1                 |    运算符重载    |
|          map.get("key")           |              map["key"]               |  get的方法约定   |
|    file.use({ f -> f.read() })    |         file.use{ it.read() }         |  括号外的lambda  |
| sb.append("yes")  sb.append("no") | with(sb){ append("yes") append("no")} | 带接受者的lambda |

扩展函数类型(不需要显示的修饰符就可以访问一个外部类型的成员) 一个可以被当作扩展函数来调用的代码块

```kotlin
//函数参数为 函数类型
fun buildString(builderAction:(StringBuilder)->Unit):String{
    val sb= StringBuilder()
    builderAction(sb)
    return sb.toString()
}

val s = buildString{
            it.append("Hello,")
            it.append("World!")
                       }

//进一步优化 StringBuilder.()->Unit 定义接受者的函数类型的参数
//使用扩展函数类型取代普通函数类型来声明参数类型
fun buildString(builderAction:StringBuilder.()->Unit):String{
    val sb= StringBuilder()
    sb.builderAction()
    return sb.toString()
}

val s = buildString{
            this.append("Hello,")//this 指向StringBuilder的实例
            append("World!")//也可以省略 this
                       }

//扩展函数类型(不需要显示的修饰符就可以访问一个外部类型的成员)
String.(Int,Int)->Unit
//接受者是String 两个参数类型是Int 返回类型是Unit
```

扩展函数变量

```kotlin
//扩展函数类型赋值给一个变量
val appendExcl :StringBuilder.()->Unit={this.append("!")}
val stringBuilder=StringBuilder("Hi")
stringBuilder.appendExcl()//和调用扩展函数一样调用
println(StringBuilder)
//Hi!
println(buildString(appendExcl))//扩展函数类型 作为参数一样传递
//!
fun buildString(builderAction:StringBuilder.()->Unit):String=
    StringBuilder().apply(builderAction).toString()
```

```kotlin
inline fun <T> T.apply(block:T.()->Unit):T{
       block()
       return this//返回函数接受者本身
}

inline fun <T,R> with(receiver:T,block:T.()->R):R=receiver.block()//返回值是block函数的执行结果

val map= mutableMapOf(1 to "one")
map.apply{this[2]="two"}
with(map){this[3]="three"}
println(map)
//不关心结果的情况下 2个可以相互调用
```

`invoke` 约定

```kotlin
class Greeter(val greeting:String){
  operator fun invoke(name:String){// invoke约定
    println("$greeting,$name")
  }
}
val bavarianGreeter=Greeter("Servus")
bavarianGreeter("Dmitry")//bavarianGreeter.invoke("Dmitry")
//Servus,Dmitry

interface Function2<in P1,in p2,out R>{
  operator fun invok(p1,:P1 , p2:P2):R
}
```

将函数类型做为基类

```kotlin
data class Issue(
    val id:String,
    val project:String,
    val type:String,
    val priority:String,
    val description:String
)

class ImportantIssuesPredicate(val project:String):(Issue)->Boolean{

    override fun invoke(issue: Issue): Boolean {
        return issue.project==project&&issue.isImportant()
    }

    private  fun Issue.isImportant():Boolean{
        return type=="Bug"&&(priority=="Major"||priority=="Critical")
    }
}

val i1=Issue("IDEA-154446","IDEA","Bug","Major","Save settings failed")
val i2=Issue("KT-12183","Kotlin","Feature","Normal","Intention: convert several calls on the same receiver to with/apply")
val predicate=ImportantIssuesPredicate("IDEA")
for( issue in listOf<Issue>(i1,i2).filter(predicate)){
     println(issue.id)
}
//IDEA-154446
//ImportantIssuesPredicate 继承的 函数类型  filter 会调用invoke
public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
    return filterTo(ArrayList<T>(), predicate)
}
```

DSL中的`invoke`约定：在Gradle中声明依赖

```kotlin
class DependendyHandler{
    fun compile(coordinate:String){
        println("Added dependency on $coordinate")
    }
    operator fun invoke(body:DependendyHandler.()->Unit){
        body()
    }
}


val dependencies=DependendyHandler()
//直接调用compile 方法
dependencies.compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
//调用 invoke 类似 dependencies.invoke{compile(xxx)}
dependencies{
  	//this.compile(xxx)
    compile("org.jetbrains.kotlin:kotlin-stdlib:1.0.0")
}
```





