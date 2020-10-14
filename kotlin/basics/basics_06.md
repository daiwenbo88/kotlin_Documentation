###### Kotlin - 数据类型

> <font color=#333333>基本数据类型 Int 、Boolean 及其他 </font>

Kotlin 不区分基本数据类型合包装类型

大多数情况下 对于变量 属性 参数 返回类型 Kotlin 的Int类型会被编译成java 基本数据类型 int  

泛型会编译成对于包装类型

只要使用了基本数据类型的可空版本,它就会编译成对应的包装类型

```kotlin
val i : Int = 10
val list : List<Int> =listOf(1,2,3)
```

- 整数类型--Byte、Short、Int、Long
- 浮点类型--Float、Double
- 字符类型--Char
- 布尔类型--Boolean

```kotlin
//定义可空的基本数据类型
data class Person(val name:String,val age:Int?=null){
  
  fun isOlderThan(other:Person):Boolean?{
    if(age==null||null==other.age)return null
    return age>other.age
  }
}
```

><font color=#333333>数字转换</font>

Kotlin不会自动把数字从一种类型转换成另外一种

```kotlin
val i: Int  = l
val l: Long = i.toLong //显示的进行转换
```

><font color=#333333>Any 和  Any?  跟类型</font>

Any 是所有非空类型的根类型 Any类型变量不可以持有null 

Any? 可以持有任何类型的变量 包含null

```kotlin
val answer:Any=42 // Any 是引用类型变量 42会被进行装箱
```

><font color=#333333>Nothing类型：这个函数永远不返回</font>

```kotlin
fun fail(message:String):Nothing{
   throw IllegalStateException(message)
}
fail("Error occurred")
```

><font color=#333333>集合</font>

- 可空性集合

  ```kotlin
  //List<Int?> 集合元素可以为null
  val result:ArrayList<Int?> = ArrayList<Int?>()
  //集合可以为null
  val result:ArrayList<Int>? = ArrayList<Int?>()
  //集合可以为null 元素也可以为null
  val result:ArrayList<Int?>? = ArrayList<Int?>()
  
  //List<Int?> 集合元素可以为null
  fun readNumbers(reader: BufferedReader): List<Int?> {
      val result = ArrayList<Int?>()
      for (line in reader.lineSequence()) {
          try {
              val num = line.toInt()//读取一行转换为number类型
              result.add(num)
          } catch (e: NumberFormatException) {
              result.add(null)//添加空元素
          }
      }
      return result
  }
  ```

  ```kotlin
  fun addValidNumbers(numbers: List<Int?>) {
    //filterNotNull 去掉集合空元素
      val validNumbers=numbers.filterNotNull()
  }
  ```

  

- 只读集合与可变集合

​       Kotlin.collections.Collection 这个接口 可以遍历集合中的元素 获取集合大小 判断集合是否包含某个元素 以及执行从集合读取数据的操作 但是没有任何添加或移除的方法(只读集合)

​      Kotlin.collections.MutableCollection 可以修改集合数据 继承 Kotlin.collections.Collection

| 集合类型 |  只读  | 可变                                              |
| :------: | :----: | :------------------------------------------------ |
|   List   | listOf | mutableListOf、arrayListOf                        |
|   Set    | setOf  | mutableSetOf、hashSetOf、linkedSetOf、sortedSetOf |
|   Map    | mapOf  | mutableMapOf、hashMapOf、linkedMapOf、sortedMapOf |



```kotlin
fun <T> copyElements(source:Collection<T>,traget:MutableCollection<T>){
  for(item in source){
    traget.add(item)
  }
}
val source:Collection<Int> =arrayListOf(3,5,7)
val target:MutableCollection<Int>=arrayListOf(1)
copyElements(source,target)

val target_02:Collection<Int>=arrayListOf(1)
copyElements(source,target_02)//报错 
```



​    对于集合类型的选型(只读/可变) 与java 相互调用的时候要考虑：

​    集合是否可空  集合元素是否可空 你的方法是否会修改集合

   ```kotlin
List<String> //Java List 

//对应kotlin 的集合
List<String>? //集合是否为null
MutableList<String?>//集合元素是否可空
   ```



> <font color=#333333>数组</font>

Kotlin 数组是一个带有类型参数的类型 其元素类型被指定为相应的类型参数

创建数组方法：

   arrayOf、  arrayOfNulls(可以包含null 元素)、Array(创建指定大小的数组)  -->装箱类型

```kotlin
fun main(array: Array<String>) {
  for(i in args.indices){
    println("Argument $i is :${args[i]}")
  }
}

var letters=Array<String>(26){i->('a'+i).toString()}
println(letters.joinToString())

val strings=listOf("a","b","c")
//vararg 关键字 可变参数  *(展开运算符)  toTypedArray()将集合转换为数组
println("%s/%s/%s".format(*strings.toTypedArray()))

val squares =Array<Int>(5){i->(i+1)*(i+1)}
val source=squares.toIntArray()
```

  IntArray、ByteArray、ChatArray、BooleanArray 对应基本数据类型  int[]、 byte[]、 char[]

```kotlin
//创建5个0的整型数组
val fiveZeros=IntArray(5)
val fiveZerosToo=intArrayOf(0,0,0,0,0)

//使用lambda 创建整型数组
val squares =IntArray(5){i->(i+1)*(i+1)}
println(squares.joinToString())
//数组操作api 和集合类似
squares.forEachIndexed{index,element->
  println("Argument $index is :$ element")
}
```

