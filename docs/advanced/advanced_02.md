###### Kotlin - 运算符重载



> <font color=#333333>运算符重载 </font>

- 重载二元运算符

  operator： 所有重载运算符都要改关键字标记

  

  ```kotlin
  //operator 是重载运算符 关键字 
  data class Point(val x: Int, val y: Int) {
      // plus (加法)
      operator fun plus(other: Point): Point {
          return Point02(x + other.x, y + other.y)
      }
  }
  
  var p1 = Point(10, 20)
  val p2 = Point(10, 20)
  println(p1 + p2)//调用的是plus() 函数 
  
  //扩展函数 乘法
  operator fun Point.times(scale: Double): Point02 {
      return Point((x * scale).toInt(), (y * scale).toInt())
  }
  println(p1 * 1.5)
  
  
  operator fun Char.times(count:Int):String{
    return toString().repeat(count)
  }
  println('a'*3)
  
  
  ```

  

  

  | First Header | 函数名 |
  | :----------: | :----: |
  |     a*b      | times  |
  |     a/b      |  div   |
  |     a%b      |  mod   |
  |     a+b      |  plus  |
  |     a-b      | minus  |

- 重载复合运算符

  +=、-= 等运算符被称为复合运算符 

  区别  plus和 plusAssign都能实现+=这样的逻辑 单变量不可变时(val) 就要用 plus 变量可变时(var) 用plusAssign

  

  ```kotlin
  data class Point(val x: Int, val y: Int)
  //plusAssign
  operator fun Point.plusAssign(scale: Double): Point02 {
      return Point((x * scale).toInt(), (y * scale).toInt())
  }
  
  var p1 = Point(10, 20)
  p1 += Point(5, 5)// 只对 var(可变变量)有效
  println(p1)
  
  //+ -运算符总是返回一个新的集合
  //+= -=用于可变集合时 始终在一个地方修改他们 用于只读集合时 返回一个修改过的副本
  val list=arrayListOf(1,2)
  list+=3//修改list
  println(p1)//[1,2,3]
  
  val newList=list+listOf(4,5)
  println(newList)//[1,2,3,4,5]
  ```

- 重载一元运算符

  |  表达式  | 函数名     |
  | :------: | ---------- |
  |    +a    | unaryPlus  |
  |    -a    | unaryMinus |
  |    !a    | not        |
  | ++a、a++ | inc        |
  | --a、a-- | dec        |



```kotlin
data class Point(val x: Int, val y: Int)
//重载一元运算符
operator fun Point.unaryMinus(): Point02 {
    return Point(-x, -y)
}
var p1 = Point(10, 20)
//一元运算符
println(-p1)//Point(-10,-20)

operator fun BigDecimal.inc() = this + BigDecimal.ONE

var bd = BigDecimal.ZERO
println(bd++)//0 后加加
println(++bd)//2 前加加

```



- 重载比较运算符

   ==、!=  会被转换为equals 比较的调用 同时也可以用于空运算数 (a==b) 会检查a是否为null

   `a==b`--->`a?.equals(b) ?:(b==null)`

  ===恒等运算符 检查2个对象是否同一对象引用（基本数据类型 检查他们的值是否相同）同时该运算符不能被重载

  ```kotlin
class Point(val x: Int, val y: Int) {
  override fun equals(obj:Any?):Boolean{
    if(obj===this)return true
    if(obj !is Point)return false
    return obj.x==x&&obj.y==y
  }
}
println(Point(10,20)==Point(10,20))
  ```

- 重载排序运算符

  (<、>、<=、>=)的使用将被转换为compareTo

  `a>b`--->`a.compareTo(b)>=0`

  

  ```kotlin
  class Person(val firstName:String,val lastName:String):Comparable<Person>{
  override fun compareTo():Int{
    //先比较 lastName (相同)再比较 firstName
    return compareValuesBy(this, other, Person::lastName, Person::firstName)
  }
  }
  //实现Comparable 接口
  val pe01 = Person("Alice", "Smith")
  val pe02 = Person("Bob", "Johnson")
  println(pe01 < pe02)//false
  //所有java中实现了 Comparable接口的类 都可以在Kotlin 中做比较
  println("abc">"bac")//true
  ```



> <font color=#333333>集合与区间的约定 </font>

- 通过下标来访问元素  `get`和`set`  比如map



```kotlin
data class Point(val x: Int, val y: Int)
// 标记operator 后 get可以像数组下标 方式调用 get方法参数可以是任意类型
operator fun Point.get(index:Int){
  return when(index){
    0->x
    1->y
    else->throw IndexOutofBoundsException("Invalid coordinate $index")
  }
}

operator fun Point.set(index:Int,value:Int){
  return when(index){
    0->x=value
    1->y=value
    else->throw IndexOutofBoundsException("Invalid coordinate $index")
  }
}
val p=Point(10,20)
println(p[1])//p.get(1)
p[1]=100     //p.set(1,1000)
println(p[1])
```



- **in 约定**

  in：用于检查某个对象是否属于集合 相应的函数叫作 `contains`

  until：建立一个开区间

  

  ```kotlin
  data class Point(val x: Int, val y: Int)
  
  data class Rectangle(val upperLeft:Point,val lowerRight:Point)
  
  operator fun Rectangle.contains(p:Point){
    return p.x in upperLeft.x until lowerRight.x &&
           p.y in upperLeft.y until lowerRight.y
  }
  
  val rect = Rectangle(Point(10, 20), Point(50, 50))
  //检查摸个点是否属于 这个矩形区域
  println(Point(15, 15) in rect)
  ```



-  **rangeTo**

   rangeTo: 返回一个区间

  `start..end`--->`start.rangeTo(end)`

  ```kotlin
   operator fun<T:Comparable<T>>T.rangeTo(that:T):ClosedRange<T>
  
   val now=LocalDate.now()
   val vacation=now..now.plusDats(10)// 编译为 now.rangeTo(now.plusDats(10))
   println(now.plusWeeks(1) in vacation)
  ```

- **解构声明和组件函数**

   展开单个复合值，并使用它来初始化多个单独变量 

```kotlin
data class Point(val x: Int, val y: Int)

val p=Point(10,20)
val(x,y)=p // x, y 值使用p组件来初始化
println(x)//10
println(y)//20

//解构声明编译如下  在解构声明中初始化每个变量 将调用componentN()的函数 N是声明变量的位置 从一个函数返回多个值的场景使用
class Point(val x:Int,val y:Int){
  operator fun component1()=x
  operator fun component2()=y
}

data class NameComponents(val name:String,val extension:String)

fun splitFilename(fullName:String):NameComponents{
  //val result=fullName.split(".",limit=2)
  val (name,extension)=fullName.split(".",limit=2)
  return NameComponents(name,extension)
}

val(name,ext)=splitFilename("example.kt")
println(name)//example
println(ext)//kt

```

- **委托属性**

 by：将访问器的逻辑委托给另外一个对象

```kotlin
class Foo{
   var p:Type by Delegate()
}

class Foo{
  private val delegate=Delegate()
  
  var p :Type
    set(value:Type)=delegate.setValue(...,value)
    get()= delegate.getValue(...)
}
 val foo=Foo()
 val oldValue=foo.p
 foo.p=newValue
```

by lazy：惰性初始化

直到第一次访问该属性的时候，才根据需要创建对象的一部分 只被初始化一次

```kotlin
class Person(val name:String){
  val emails by lazy{loadEmails(this)}
} 

open class PropertyChangeAware {
    protected val changeSupport = PropertyChangeSupport(this)

    fun addPropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.addPropertyChangeListener(listener)
    }

    fun removePropertyChangeListener(listener: PropertyChangeListener) {
        changeSupport.removePropertyChangeListener(listener)
    }
}
//使用Delegates.observable 来实现属性修改通知
class Person(val name: String, age: Int, salary: Int) : PropertyChangeAware() {
    private val observer = {
        prop:KProperty<*>, oldValue:Int, newValue:Int->
        changeSupport.firePropertyChange(prop.name,oldValue,newValue)
    }
    var age:Int by Delegates.observable(age,observer)
    var salary:Int by Delegates.observable(salary,observer)
}

val p3 = Person("Dmity", 34, 2000)
//添加监听器
p3.addPropertyChangeListener(PropertyChangeListener { event ->
   println("Property ${event.propertyName} change" + "from${event.oldValue}to${event.newValue}")
    })
p3.age = 80
p3.salary = 9000

```

