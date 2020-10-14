# Kotlin

> <font color=#333333>配置 </font>

**项目根目录下的：<font color=#4CAF50>build.gradle</font>**

```groovy
buildscript {
    👇
    ext.kotlin_version = '1.3.71'
    repositories {
        ...
    }
    dependencies {
        classpath 'com.android.tools:r8:1.6.84'
        classpath 'com.android.tools.build:gradle:3.6.3'
        👇
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
    }
}
```

**Module目录下的下的：<font color=#4CAF50>build.gradle</font>**

```groovy
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-kapt'//项目中有使用 apt的时候添加


dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    👇
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    ...
}
```

> **协程配置**

**项目根目录下的：<font color=#4CAF50>build.gradle</font>**

```groovy
buildscript {
    ...
    // 👇指定版本
    ext.kotlin_coroutines = '1.3.1'
    ...
}
```

**Module目录下的下的：<font color=#4CAF50>build.gradle</font>**

```groovy
dependencies {
    ...
    //            👇 依赖协程核心库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:$kotlin_coroutines"
    //            👇 依赖当前平台所对应的平台库
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$kotlin_coroutines"

    ...
}
```

**官网：**[https://www.kotlincn.net/](https://www.kotlincn.net/)

**版本更新文档：**[https://www.kotliner.cn/](https://www.kotliner.cn/)

**官方文档：**[https://www.kotlincn.net/docs/reference/](https://www.kotlincn.net/docs/reference/)

**反射jar包下载：**[kotlin-reflect.jar](https://www.mvnjar.com/org.jetbrains.kotlin/kotlin-reflect/jar.html)

**官方协程学习Demo：**[Kotlin协程官方框架]( https://github.com/Kotlin/kotlinx.coroutines)

**官方协程文档：**[协程文档](https://www.kotlincn.net/docs/reference/coroutines/coroutines-guide.html)

**官方对Android的支持：**[Android KTX](https://developer.android.com/kotlin/ktx)

**Kotlin博客：**[Kotlin博客](https://www.bennyhuo.com/2018/10/02/kotlin-community-cn/#more)

