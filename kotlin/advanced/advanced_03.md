###### Kotlin-泛型

> <font color=#333333>泛型声明 </font>

```kotlin
val readers :MutableList<String>= mutableListOf()
val readers = mutableListOf<String>()
```

**泛型函数**

```kotlin
fun<T> List<T>.slice(indices:IntRange):List<T>
```

**泛型属性**

```kotlin
val <T> List<T>.penultimate: T
         get()=this[size-2]
      
println(listOf(1,2,3,4).penultimate)//3     
```

**泛型类**

```kotlin
interface List<T>{
 operator fun get(index:Int):T
}
//继承List 指定具体实参：String
class StringList:List<String>{
  override fun get(index:Int):String=....
}

class ArrayList<T> :List<T>{
  override fun get(index:Int):T=....
}

```

> <font color=#333333>类型参数约束 </font>

把一个类型指定为泛型类型形参的上界约束 在泛型类型具体的初始化中，其对应的类型参数就必须是这个具体类型或者它的子类型

```kotlin
fun <T:Number> List<T>.sum():T//上界是number
println(listOf(1,2,3,4).sum())//Int 是Number的子类型
```

 指定了类型参数的上界 就可以把类型T的值当作它的上界(类型)的值使用

```kotlin
fun <T :Number> oneHalf(value:T):Double{
   return value.toDouble()/2.0 //调用number的方法
}
println(3)
```

类型参数上指定多个约束

```kotlin
// 类型实际参数的类型必须实现 CharSequence 和 Appendable
fun <T> ensureTrailingPeriod(seq:T) where T :CharSequence, T : Appendable{
  if(!seq.endsWith('.')){seq.append('.')}
}
val hellWorld=StringBuilder("Hello World")
ensureTrailingPeriod(hellWorld)
println(hellWorld)// Hello World.
```

**让类型参数非空**

没有指定上界的类型形参将会使用`Any?` 这个默认上界

```kotlin
class Processor<T>{
  fun process(value:T){
     value?.hashCode() //value 是可空的
  }
}
val nullableStirngprocessor=Processor<String?>()
nullableStirngprocessor.process(null)
```

要实现替换类型形参始终非空类型 可以指定一个约束来实现 也可以用`Any`代替`Any?`作为上界

```kotlin
class Processor<T : Any>{
  fun process(value:T){
     value.hashCode() //类型T 是非空的
  }
}
```

><font color=#333333>运行时泛型 </font>

Kotlin的泛型在运行时也会被擦除

在比较一个值是否为集合时 要用**星号投影**

```kotlin
if(value is List<String>){}//编译不会通过 在运行时时 泛型类型会被擦除 所以不知道集合里面是什么类型
if(value is List<*>){}//使用星号投影 进行比较 主要检查value 是否为List类型 不检查集合元素类型
```

```kotlin
fun printSum(c:Collection<*>){
 val intList= c as? List<Int> ?: throw IlleagelArgumentException("List is expected")
 println(intList.sum()) 
}
printSum(listOf(1,2,3))//6
printSum(setOf(1,2,3)) //类型不匹配 会报错 List is expected
printSum(listOf("a","b","c"))//String cannot be cast to Number 类型转换异常
```

Kotlin可以对已知类型作类型转换 因为在编译期就确定了集合元素类型

```kotlin
fun printSum(c:Collection<Int>){
 if(c is List<Int>){//已知类型 可以进行类型转换
   println(intList.sum()) 
 }
}
printSum(listOf(1,2,3))//6
```

><font color=#333333>声明带实化类型参数的函数 </font>

把函数声明为`inline` 并且用`reified` (实化)标记类型参数 就可以检查该函数的T类型



```kotlin
inline fun <reified T> isA(value : Any)=value is T

println(isA<String>("abc")) //true
println(isA<String>(123)) //false
```

内联函数调用时 编译器会把函数字节码插入每一次调用的地方 每次调用带实化类型参数函数时 编译器都知道这次特定调用中用作类型时参的确切类型 因此编译器可以生成引用作为类型实参的具体类的字节码

带`reified`类型参数的`inline`函数不能在java被调用 普通的内联函数可以像常规函数那样在java中被调用

```kotlin
inline fun <reified T> Iterable<*>.filterIsInstance():List<T>{
 val destination=mutableListOf<T>()
  for(element in this){
    if(element is T){//在inline 内联函数中才可以这样写
        destination.add(element)
    }
    //生成的字节码引用了具体类型 而不是参数类型 不会被运行时发生的类型参数擦除影响
  }
  return destination
}
val items=listOf("one",2,"three")
println(items.filterIsInstance())//[one,three]
```

**实话类型参数代替类引用**

```kotlin
inline fun <reified T> loadService(){
  return ServiceLoader.load(T::class.java)
}

val serviceImpl=loadService<Service>()
```

><font color=#333333>变型</font>

**类、类型、 子类型、超类型**

```kotlin
//非空类型是可空类型的子类
val s : String = "abc"
val t : String？ = s
```

## Java 中的 ? extends

```java
List<Button> buttons = new ArrayList<Button>();
      👇
List<? extends TextView> textViews = buttons;

//这个 ? extends 叫做「上界通配符」，可以使 Java 泛型具有「协变性 Covariance」，协变就是允许上面的赋值是合法的
//在继承关系树中，子类继承自父类，可以认为父类在上，子类在下。extends 限制了泛型类型的父类型，所以叫上界
```

`? extends`它有两层意思：

- 其中 `?` 是个通配符，表示这个 `List` 的泛型类型是一个**未知类型**。
- `extends`限制了这个未知类型的上界，也就是泛型类型必须满足这个`extends`的限制条件，这里和定义`class`的`extends`关键字有点不一样：
  - 它的范围不仅是所有直接和间接子类，还包括上界定义的父类本身，也就是 `TextView`。
  - 它还有 `implements` 的意思，即这里的上界也可以是 `interface`。

这里 `Button` 是 `TextView` 的子类，满足了泛型类型的限制条件，因而能够成功赋值

```java
List<? extends TextView> textViews = new ArrayList<TextView>(); // 👈 本身
List<? extends TextView> textViews = new ArrayList<Button>(); // 👈 直接子类
List<? extends TextView> textViews = new ArrayList<RadioButton>(); // 👈 间接子类
```



```java
List<? extends TextView> textViews = new ArrayList<Button>();
TextView textView = textViews.get(0); // 👈 get 可以
textViews.add(textView);
//             👆 add 会报错，no suitable method found for add(TextView)
```

`List` 的泛型类型是个未知类型 `?`，编译器也不确定它是啥类型，只是有个限制条件

`? extends TextView` 的限制条件，所以 `get` 出来的对象，肯定是 `TextView` 的子类型，根据多态的特性，能够赋值给 `TextView`，赋值给 `View` 也是没问题。

到了 `add` 操作的时候，我们可以这么理解：

- `List<? extends TextView>` 由于类型未知，它可能是 `List<Button>`，也可能是 `List<TextView>`。
- 对于前者，显然我们要添加 TextView 是不可以的。
- 实际情况是编译器无法确定到底属于哪一种，无法继续执行下去，就报错了。

由于 `add` 的这个限制，使用了 `? extends` 泛型通配符的 `List`，只能够向外提供数据被消费，从这个角度来讲，向外提供数据的一方称为「生产者 Producer」

## **Java 中的 ? super**

```java
List<? super Button> buttons = new ArrayList<TextView>();
```

这个 `? super` 叫做「下界通配符」，可以使 Java 泛型具有「逆变性 Contravariance」。

与上界通配符对应，这里 super 限制了通配符 ? 的子类型，所以称之为下界。

它也有两层意思：

- 通配符 `?` 表示 `List` 的泛型类型是一个**未知类型**。
- `super`限制了这个未知类型的下界，也就是泛型类型必须满足这个`super`的限制条件。
  - `super` 我们在类的方法里面经常用到，这里的范围不仅包括 `Button` 的直接和间接父类，也包括下界 `Button` 本身。
  - `super` 同样支持 `interface`。

```java
List<? super Button> buttons = new ArrayList<Button>(); // 👈 本身
List<? super Button> buttons = new ArrayList<TextView>(); // 👈 直接父类
List<? super Button> buttons = new ArrayList<Object>(); // 👈 间接父类
```



```java
List<? super Button> buttons = new ArrayList<TextView>();
Object object = buttons.get(0); // 👈 get 出来的是 Object 类型
Button button = ...
buttons.add(button); // 👈 add 操作是可以的
```

首先 `?` 表示未知类型，编译器是不确定它的类型的。

虽然不知道它的具体类型，不过在 Java 里任何对象都是 `Object` 的子类，所以这里能把它赋值给 `Object`。

`Button` 对象一定是这个未知类型的子类型，根据多态的特性，这里通过 `add` 添加 `Button` 对象是合法的。

使用下界通配符 `? super` 的泛型 `List`，只能读取到 `Object` 对象，一般没有什么实际的使用场景，通常也只拿它来添加数据，也就是消费已有的 `List`，往里面添加 `Button`，因此这种泛型类型声明称之为「消费者 Consumer」



结下，Java 的泛型本身是不支持协变和逆变的。

- 可以使用泛型通配符 `? extends` 来使泛型支持协变，但是「只能读取不能修改」，这里的修改仅指对泛型集合添加元素，如果是 `remove(int index)` 以及 `clear` 当然是可以的。
- 可以使用泛型通配符 `? super` 来使泛型支持逆变，但是「只能修改不能读取」，这里说的不能读取是指不能按照泛型类型读取，你如果按照 `Object` 读出来再强转当然也是可以的。

## ** Kotlin 中的 out 和 in**

- 使用关键字 `out` 来支持协变，等同于 Java 中的上界通配符 `? extends`。
- 使用关键字 `in` 来支持逆变，等同于 Java 中的下界通配符 `? super`。

```kotlin
var textViews: List<out TextView>
var textViews: List<in TextView>
```

`out` 表示，我这个变量或者参数只用来输出，不用来输入，你只能读我不能写我；

`in` 就反过来，表示它只用来输入，不用来输出，你只能写我不能读我

```kotlin
//生产者
class Producer<out T> {
    fun produce(): T {
        ...
    }
}
val producer: Producer<TextView> = Producer<Button>()
val textView: TextView = producer.produce() // 👈 相当于 'List' 的 `get`
```



```kotlin
//消费者
class Consumer<in T> {
    fun consume(t: T) {
        ...
    }
}

val consumer: Consumer<Button> = Consumer<TextView>()
consumer.consume(Button(context)) // 👈 相当于 'List' 的 'add'
```

一个类可以在参数上协变 同时在另外一个类型参数上逆变

```kotlin
interface Function1<in P,out R>{
  operator fun invoke(p :P):R
}
```

## ** 点变型**

在类声明的时候就能够指定变型修饰符是很方便的，因为这些修饰符会应用到所有类被使用的地方，称为**声明点变型**

```kotlin
//来源元素是目标元素的子类
fun <T:R,R> copyData(source :MutableList<T>,destination:MutableList<T>){
  for(item in source){
    destination.add(item)
  }
}
val ints=mutableListOf(1,2,3)
val anyItems=mutableListOf<Any>()
copyData(ints,anyItems)//Int 是Any 的子类
println(anyItems)

//out 表示元素类型是 T 的子类
fun <T> copyData(source :MutableList<out T>,destination:MutableList<T>){
  for(item in source){
    destination.add(item)
  }
}
//in 目标元素类型是来源元素类型的超类型
fun <T> copyData(source :MutableList<T>,destination:MutableList<in T>){
  for(item in source){
    destination.add(item)
  }
}
```

**星号投影**

用星号代替类型参数 来表明你不知道关于泛型实参的任何消息 `List<*>`

```kotlin
val list : MutableList<Any?> =mutableListOf('a',1,"qwe")
val chars = mutableListOf('a','b','c')

val unKnownElements : MutableList<*> = if (Random.nextBoolean()) list else chars
unKnownElements.add(42)//禁止编译
//这个例子中 MutableList<*> 投影成了 MutableList<out Any?> 只能读取元素(消费者) 不能写入
```

