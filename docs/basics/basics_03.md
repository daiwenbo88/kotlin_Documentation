###### Kotlin - 对象和接口

> <font color=#333333>接口 </font>

```kotlin
//接口
interface Clickable {
    fun click()
  	//接口中可以有默认实现的方法
  	fun showOff() = println("I'm clickable")
}

//实现

//: 代表继承或者实现
class Button : Clickable{
    //override 重写父类的或者接口方法(强制要求)
    override fun click() {
        println("I was clicked")
    }
}
```

当有2个接口提供的默认实现方法一致时 会强制要求子类提供自己的实现

```kotlin
interface Clickable {
    fun click()
  	fun showOff() = println("I'm clickable")
}

interface Focusable {
    fun setFovus(b: Boolean) = println("I ${if (b) "got" else "lost"}focus")
    fun showOff() = println("I'm focusable")
}

//: 代表继承的实现 有多个用逗号分隔
class Button : Clickable, Focusable {
    override fun showOff() {
        //表示要调哪一个父类型的
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
    //override 重写父类的方法
    override fun click() {
        println("I was clicked")
    }
}

```

><font color=#333333>类 </font>

Kotlin中 类和方法默认都是final 的

被重写或者实现的方法默认是open的 可以将其标注为final的 阻止子类实现

```kotlin
//接口
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable")
}

//open 代表 RichButton可以被子类继承
open class RichButton : Clickable {
    //重写方法默认是open 声明为final 就不能被子类重写
    final override fun click() {
    }

    //这个函数是final 的 不能被子类重写
    fun disable() {}

    //这个函数是open 的 可以被子类重写
    open fun animate() {}

}
```

><font color=#333333>抽象类 </font>

abstract 关键字声明的类为抽象类 其中没有实现的抽象方法为 open的

```kotlin
//抽象类
abstract class Animated {
    //抽象函数 子类必须实现
    abstract fun animate()

    //这个函数是open 的 可以被子类重写
    open fun stopAnimating() {
    }

    //这个函数是final 的 不能被子类重写
    fun animateTwice() {}
}
```

><font color=#333333>可见性修饰符</font>

如果省略修饰符 默认声明就是publicde 

|    类成员     |    修饰符    |   顶层声明   |
| :-----------: | :----------: | :----------: |
| publice(默认) | 所有地方可见 | 所有地方可见 |
|   internal    |  模块中可见  |  模块中可见  |
|   protected   |  子类中可见  |      -       |
|    private    |   类中可见   |  文件中可见  |

><font color=#333333>嵌套类</font>

内部类添加inner关键字后 表示该内部类可以持有外部类的引用 反之该内部类就是静态的内部类

```kotlin
class Buttons : View {
    override fun getCurrentState(): State = ButtonState()
    override fun restoreState(state: State) {}
  
    //inner 修饰符 表示内部类可以持有外部类引用
    inner class ButtonState : State {
         //引用外部类实例方法 需要通过this@类名 
        fun getOuterReference(): Buttons = this@Buttons
    }
}

```

> <font color=#333333>密封类</font>

sealed 关键字 对可能创建的子类做成严格限制，所有子类必须嵌套在父类中 同时隐含表示这个类是open的

sealed 关键字不能用于声明接口类

```kotlin
//sealed 定义密封类
sealed class Exprs {
    class Num(val value: Int) : Exprs()
    class Sum(val left: Exprs, val right: Exprs) : Exprs()
}

fun eval(e:Exprs):Int=
    when(e){//不用去写else分支 同时如果类型没有判断完 编译不通过
        is Exprs.Num->e.value
        is Exprs.Sum->eval(e.left)+eval(e.right)
    }
```

><font color=#333333>主构造方法 初始化语句块</font>

constructor 关键字表示主构造方法或从构造方法的声明

init 关键字引入一个初始化语句块 在类被创建时要执行的代码 可以声明多个

```kotlin
class Users constructor(_nickname:String){
    val nickname:String
    init {//主构造方法执行后立马执行
        nickname=_nickname
    }
}
//val或var 可以加在参数前面的方式来 表示类中属性的定义
//构造方法参数也可以声明默认值
//所有参数都有默认值时 Kotlin 会默认创建一个无参数构造方法
class Users (val nickname:String,val value:Boolean =false)
```

初始化父类构造方法

```kotlin
open class B (val nickname:String)
//子类构造父类
class C (nickname: String ):B(nickname)

//默认有无参数构造
open class Button
class RadioButton : Button()
//必须显示的调用父类的构造方法 即使它没有任何参数

//接口
interface Clickable {
    fun click()
    fun showOff() = println("I'm clickable")
}
//接口没有构造方法
class Button : Clickable, Focusable {}
```

从构造方法

```kotlin
open class View{
		constructor(ctx:Context)
    constructor(ctx:Context,attr:AttributSet)
}

class MyButton : View{
  	//super 会调用父类的从构造方法
    constructor(ctx: Context):super(ctx){}//从构造函数
    constructor(cxt:Context,attr:AttributeSet):super(cxt,attr){}//从构造函数
}
```

实现在接口中的属性

override 关键字重写属性

```kotlin
//接口中声明属性
interface User {
    val nickname: String//接口中该属性不包含任何状态 子类有需要可以重写
    val age:Int //改属性可以被继承
        get()=18
}

//在主构造函数值实现属性赋值
class PrivateUser(override val nickname: String) : User //主构造

class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@')//截取email 之前字符串为用户名
}

class FacebookUser(val accounId: Int) : User {
    override val nickname = "text"
}

```

> <font color=#333333>属性的getter和setter</font>

```kotlin
//属性定义set 赋值
class User(val name: String) {
    var address: String = "unspecified"
        set(value: String) {//field
            println(
                """Address was Changed for $name 
           "$field"->"$value".""".trimIndent()
            )
            field = value
        }
}

val user = User("Alice")
user.address = "abcdfsg"


class LengthCounter {
    var counter: Int = 0
        private set //不能通过外部修改这个属性

    fun addWord(word: String) {
        counter += word.length
    }
}
```

><font color=#333333>对象的通用方法</font>

== 比较对象是否相等 equals

=== 比较对象的引用地址是否相等

```kotlin
//Any 是所有类的父类
class Client(val name: String, val postalCode: Int) {
   //重写toString
    override fun toString() = "Client(name=$name,postalCode=$postalCode)"
    
    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client)
            return false
        return name == other.name && postalCode == other.postalCode
    }

    override fun hashCode(): Int {
        return name.hashCode() * 31 + postalCode
    }
}

data class Client(val name: String, val postalCode: Int)
```

><font color=#333333>委托</font>

by

```kotlin
class CountingSet<T>(val innerSet: MutableCollection<T> = HashSet<T>()) :
    MutableCollection<T> by innerSet { //将MutableCollection 的实现 委托给 innerSet
    var objectsAdded = 0

    override fun add(element: T): Boolean {//不使用委托 自己实现
        objectsAdded++;
        return innerSet.add(element)
    }

    override fun addAll(elements: Collection<T>): Boolean {
        objectsAdded += elements.size
        return innerSet.addAll(elements)
    }
}

val cset = CountingSet<Int>()
cset.addAll(listOf(1,1,2))
println("${cset.objectsAdded} objects were added ${cset.size} remain")
```

><font color=#333333>Object 关键字</font>

object 实现单例

```kotlin
//object 关键字声明一个对象 单例的
object Payroll {
    val allEmployess = arrayListOf<Person>()
    fun calculateSalary() {
        for (person in allEmployess) {
					
        }
    }
}
//单例方法属性 用类名直接调用
Payroll.allEmployess.add(Person("123",false))
Payroll.calculateSalary()

//单例对象可以实现接口(前提是不包含任何状态)
object CaseInsensitiveFileComparator:Comparator<File>{
    override fun compare(o1: File, o2: File): Int {
        return o1.path.compareTo(o2.path,ignoreCase = true)
    }
}
println(CaseInsensitiveFileComparator.compare(File("User"),File("user")))

//单例对象单参数使用
val files=listOf(File("/Z"),File("/a"))
println(files.sortedWith(CaseInsensitiveFileComparator))
```

在内中实现单例  在这个类的实例中只有一个单例实例

```kotlin
data class Person(val name:String){
    //单例放到类中实现
    object NameComparator:Comparator<Person02>{
        override fun compare(o1: Person02, o2: Person02): Int {
            return o1.name.compareTo(o2.name)
        }
    }
}
//单例在类中实现
val persons=listOf(Person("123"),Person("456"))
println(persons.sortedWith(Person.NameComparator))
```

伴生对象(工厂方法)

companion object 关键字

```kotlin

class User private constructor(val nickname:String){
    //工厂构造方法
    companion object{
      	//可以访问 private 声明的构造方法
        fun newSubscribingUser(email: String)=User(email.substringBefore("@"))

        fun newFacebookUser(accoundId:Int)=User(getFacebookName(accoundId))
    }
}

//调用工厂方法 创建user对象 创建的对象也是单例的
val subUser= User.newSubscribingUser("Bob@gmail.com")
val facebookUser= User.newFacebookUser(123)
println(subUser.nickname)
println(facebookUser.nickname)
```

伴生对象可以实现接口

```kotlin
interface JSONFactory<T>{
    fun fromJSON(jsonText:String):T
}

class Person(val name:String){
    //伴生对象能实现接口
    companion object :JSONFactory<Person03>{
        override fun fromJSON(jsonText: String): Person03 {
            return  Person("name")
        }

    }
}
```

伴生对象可以声明成扩展函数

```kotlin
class Person(val name:String){
  companion object{
    
  }
}
//伴生对象扩展函数 对Person类进行扩展 同时Person类中必须有一个伴生对象 因为这个扩展函数时对Person中的伴生对象进行扩展
fun Person.Companion.fromJSON(jsonText: String):Person {
    return  Person("name")
}

val p=Person.fromJSON(json)
```

匿名对象(每次表达式被执行 都会创建一个新的对象) 

匿名对象可以访问该类的局部变量

```kotlin
interface OnclickListener {
    fun onClick()
}

class View {
  
    fun setclickListener(clickListener: OnclickListener) {
				
    }
}

//调用一
View().setclickListener(object :OnclickListener{
  override fun onClick() {
  }
})

//调用二
val listener=object :OnclickListener{
  override fun onClick() {
  }
}
View().setclickListener(listener)
```

