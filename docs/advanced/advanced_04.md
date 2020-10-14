###### Kotlin - 注解和反射

## 应用注解

注解只能有如下类型：基本数据类型、字符串、枚举、类引用、其他的注解类、以及前面这些类型的数组。

- 要把一个类指定为注解实参，在类名后加上`::class`  如`@MyAnnotation(MyClass::class)`

- 要把一个注解指定为一个实参，去掉注解名称前面的@, 如 `ReplaceWith`也是注解 指定为`Deprecated`注解的参数

  `@Deprecated("Use removateAt(index) instead.",ReplaceWith("removeAt(index)"))`

- 要把一个数组指定为一个实参，使用arrayOf函数

​      `@RequestMapping(path=arrayOf("/foo","/bar"))`

- 注解实参需要在编译期就是已知 所以不能引用任意的属性作为实参 要把属性当注解实参使用 只能是编译期常量

  ```kotlin
  const val TEST_TIMEOUT=100L
  @Test(timeout=TEST_TIMEOUT)
  fun testMethod(){
    .....
  }
  ```

  ## 注解声明

  在`class`前 添加 `annotation`

  ```kotlin
  annotation class JsonName(val name:String) //注解参数是 val
  
  //使用 @JsonName(name="first_name")和@JsonName("first_name") 一样
  ```

  元注解

  ```kotlin
  @Target(AnnotationTarget.PROPERTY)//Target 元注解说明注解可以被运用的元素类型
  annotation class JsonExclude
  
  @Target(AnnotationTarget.ANNOTATION_CLASS)//说明改注解可以被运用在class 上
  annotation class BindingAnnotation
  
  @BindingAnnotation
  class MyBinding
  ```

  使用类做注解参数

  ```kotlin
  interface Company{
    val name:String
  }
  data class CompanyImpl(override val name : String):Company
  
  data class Person(val name:String,
                    @DeserializeInterface(CompanyImpl::class) val company : Company)
  
  //KClass 在java中对应 java.lang.Class 类型 说明指向那个类 KClass<CompanyImpl>
  annotation class DeserializeInterface(val targetClass: KClass<out Any>)
  
  //KClass<Any> 没有用out 修饰符 就不能传递CompanyImpl::class out 关键字说明允许引用那些继承Any的 的类 而不是Any自己
  ```

  使用泛型类作为注解参数

  ```kotlin
  interface ValueSerializer<T>{
    fun toJsonValue(value T):Any?
    fun fromJsonValue(jsonValue:Any?):T
  }
  //接受任何实现 ValueSerializer接口的类型 
  annotation class CustomSerializer(val serializerClass:KClass<out ValueSerializer<*>>)
  
  data class Person(
            val name :String,
            @CustomSerializer(DateSerializer::class) val birthData: Date
   )
  
  ```

  

  ## 反射：在运行时对Kotlin对象进行自省

  反射：一种在运行时动态的访问对象属性的方法和方式 而不需要事先确定这些属性是什么

  反射api: KClass(表示类或者对象)、KCallable、KFunction、KProperty(可以表示任何属性)

  反射只能访问定义在最外层或者类中的属性 不能访问局部变量

  ```kotlin
  class Person(val name:String,val age:Int)
  val person=Person("Alice",29)
  val kClass=person.javaClass.kotlin//获取一个KClass<Person> 实例
  println(kClass.simpleName)//Person
  
  //memberProperties 获取这个类和所有超类中定义的全部非扩展属性
  kClass.memberProperties.forEach{println(it.name)}
  //age
  //name
  ```

  `KCallable`时函数和属性接口的超类, 可以使用`call`方法调用对应的函数或者对应属性` getter`  

  在知道具体的`KFunction`和它的形参类型合返回值类型情况下 优先使用这个具体类型的`invoke`  `call`方法是对所有类型都有效的通用手段 

  ```kotlin
  fun foo(x:Int)=println(x)
  val kFunction:KFunction1<Int,Unit>=::foo //::返回一个反射api KFunction1<Int,Unit>的实例 包含形参和返回值
  kFunction.call(42)//调用foo 函数
  //kFunction.call() 运行会报错
  
  fun sum(x:Int,y:Int)=x+y
  val kFunction:KFunction2<Int,Int,Int>=::sum
  println(kFunction.invoke(1,2)+kFunction(3,4))//10 通过invoke 方法来调用函数
  
  //kFunction(10) 编译报错 需要2个形参
  
  ```

  顶层属性的赋值和取值

  ```kotlin
  var counter=0
  val kProperty=::counter//顶层属性返回KProperty0接口实例
  kProperty.setter.call(21)//调用属性的setter 赋值
  println(kProperty.get())//21
  ```

  类属性的访问

  ```kotlin
  class Person(val name:String,val age:Int)
  val person=Person("Alice",29)
  val memberProperty:KProperty1<Person,Int>=Person::age//返回成员属性 KProperty1的实例 第一个类型参数表示接受者类型，第二个类型参数代表属性类型
  println(memberProperty.get(person))//29 KProperty1实例指向Person的age属性 
  ```

  

  ```kotlin
  fun StringBuilder.serializeObject(obj : Any){
      val kClass=obj.javaClass.kotlin//获取对象的KClass
      val properties=kClass.memberProperties //获取类的所有属性
    
      properties.joinToStringBuilder(this,prefix = "{",postfix = "}"){prop->
        serializeString(prop.name)//属性名
        append(":")
        serializePropertyValue(prop.get(obj))//属性值                                                           
      }
  }
  ```

  `KAnnotatedElement`定义了属性`annotations` 是元素上所有注解的实例组成的集合

  `KProperty` 继承了`KAnnotatedElement` 所以`KProperty.annotations`能获取一个属性的所有注解

  ```kotlin
  
  inline fun <reified T> KAnnotatedElement.findAnnotation():T?=
  			//查找这个元素上的<T>注解是否为null
        annotations.filterIsInstance<T>().firstOrNull()
  
  
  fun StringBuilder.serializeObject(obj : Any){
      val kClass=obj.javaClass.kotlin//获取对象的KClass
    	//过滤掉有带@JsonExclude注解的元素
      val properties=kClass.memberProperties.filter{it.findAnnotation<JsonExclude>==null}
    
      properties.joinToStringBuilder(this,prefix = "{",postfix = "}"){prop->
        serializeString(prop.name)//属性名
        append(":")
        serializePropertyValue(prop.get(obj))//属性值                                                           
      }
  }
  ```

  取属性值进行序列化

  ```kotlin
  annotation class JsonName(val name :String)
  //接受任何实现 ValueSerializer接口的类型 
  annotation class CustomSerializer(val serializerClass:KClass<out ValueSerializer<*>>)
  
  inline fun <reified T> KAnnotatedElement.findAnnotation():T?=
  			//查找这个元素上的<T>注解是否为null
        annotations.filterIsInstance<T>().firstOrNull()
  
  data class Person(
                   @JsonName("alias")val firstName:String,//序列化的时候用别名
                   val age:Int,
                   @CustomSerializer(DateSerializer::class) val birthData: Date
                   //只序列化DateSerializer的子类
          )
  
  fun KProperty<*>.getSerializer():ValueSerializer<Any?>?{
    //查找该元素是否有@CustomSerializer注解
    val customSerializerAnn=findAnnotation<CustomSerializer>()?:return null
    //获取注解中的KClass 实例
    val serializerClass=customSerializerAnn.serializerClass
    //如果注解类有声明object单例 可以通过serializerClass.objectinstance 获取单例 否则 createInstance 创建一个实例对象
    val valueSerializer =serializerClass.objectinstance ?: serializerClass.createInstance()
    @Suppress("UNCHECKED_CAST")
    //实例对象转换为 ValueSerializer<Any?>类型
    return valueSerializer as ValueSerializer<Any?>
  }
  
  fun StringBuilder.serializeObject(obj : Any){
      val kClass=obj.javaClass.kotlin//获取对象的KClass
    	//过滤掉有带@JsonExclude注解的元素
      val properties=kClass.memberProperties.filter{it.findAnnotation<JsonExclude>==null}
    
      properties.joinToStringBuilder(this,prefix = "{",postfix = "}"){prop->
        val jsonNameAnn=prop.findAnnotation<JsonName>()
        val propName=jsonNameAnn?.name ?: prop.name //获取注解中的值 没有就用属性名替代                                                            
        serializeString(propName)//属性名
        append(":")
                                                                    
        val value=prop.get(obj)
      //先获取有注解属性值的的序列化，没有注解的属性 返回属性值                                                              
        val jsonValue = prop.getgetSerializer()?.toJsonValue(value) ?: value                                                      
      serializePropertyValue(jsonValue)//属性值                                                           
      }
}
  ```
  
  ```kotlin
  inline fun TextView.doAfterTextChanged(crossinline action: (text: Editable?) -> Unit) = addTextChangedListener(afterTextChanged = action)
  ```
  
  
  
  
  
  
  
  