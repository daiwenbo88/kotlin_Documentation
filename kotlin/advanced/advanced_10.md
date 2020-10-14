#### 协程案例

#### 弹出对话框示例

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
 
        val showDialogButton = findViewById<Button>(R.id.btn_show_dialog)
        showDialogButton.setOnClickListener { view ->
            lifecycle.coroutineScope.launch {
                val result = showDialog()
                Toast.makeText(applicationContext, "选择了${result}", Toast. LENGTH_SHORT).show()
            }
        }
    }
    private suspend fun Context.showDialog() = suspendCancellableCoroutine<Int> { continuation ->
        AlertDialog.Builder(this)
            .setIcon(R.mipmap.ic_launcher)
            .setTitle("dialog的标题")
            .setMessage("dialog的内容")
            .setPositiveButton("确定") { dialog, which ->
                dialog.dismiss()
                continuation.resumeWith(Result.success(1))
            }
            .setNegativeButton("取消") { dialog, which ->
                dialog.dismiss()
                continuation.resumeWith(Result.success(2))
            }
            .setOnCancelListener {
                continuation.resumeWith(Result.success(0))
            }
            .create()
            .also { dialog ->
                continuation.invokeOnCancellation {
                    dialog.dismiss()
                }
            }
            .show()
    }
}
```

- 在界面中配置了一个id为btn_show_dialog的按钮，并通过熟悉的findViewById方法获取到一个Button对象,请留意IDE上其类型是Button!，表示是一个平台类型，它有可能为空。
- 给Button对象的showDialogButton变量设置点击回调，setOnClickListener方法通过SAM转换后可直接以表达式的形式来进行。
- lifecycle.coroutineScope.launch{…} 用于声明一个当 LifeCycle 回调 onDestroy() 时变自动取消的协程。
  点击的处理是要弹出一个Android原生的对话框，我们定义了一个Context的扩展方法showDialog用于对话框的生成。
- showDialog函数用suspend修饰，表示它还是一个挂起函数，它指向于一个suspendCancellableCoroutine函数。
- suspendCancellableCoroutine是suspendCoroutine的高级版，它支持如果函数在挂起时取消或完成协同，就会引发CancellationException，表示支持协程的取消。
- AlertDialog内部存在确定、取消和关闭三种结果的正确形式返回。
  also当协程取消时关闭对话框。
- 最后调用处通过同步的写法得到三种不同结果的返回值并通过Toast显示。

#### 网络请求示例

```kotlin
class MainActivity : AppCompatActivity() {
    private val mOkHttpClient = OkHttpClient()
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
 
        val networkRequestButton = findViewById<Button>(R.id.btn_network_request)
        networkRequestButton.setOnClickListener {
            lifecycle.coroutineScope.launch {
                networkRequest("https://www.baidu.com")
                    .flowOn(Dispatchers.IO)
                    .collect {
                        Toast.makeText(applicationContext,"【${Thread.currentThread().name}】网络请求结果$it", Toast.LENGTH_SHORT).show()
                    }
            }
        }
    }
    private fun networkRequest(url: String): Flow<String> {
        return flow {
            val request = Request.Builder().url(url).get().build()
            val response = mOkHttpClient.newCall(request).execute()
            if (response.isSuccessful) {
                emit(response.code.toString())
                emit(inputStreamToString(response.body!!.byteStream()))
            } else {
                throw IOException("request error!")
            }
        }.catch { t: Throwable ->
            emit("error $t")
        }
    }
    private fun inputStreamToString(inputStream: InputStream): String {
        var resultBuffer = StringBuffer()
        BufferedReader(InputStreamReader(inputStream)).use {
            while (true) {
                var temp = it.readLine() ?: break
                resultBuffer.append(temp)
            }
        }
        return resultBuffer.toString()
    }
}
```

- 示例使用了Okhttp3进行了网络请求，所以别忘记添加Okhttp3的依赖（implementation "com.squareup.okhttp3:okhttp:4.6.0"）和联网权限的声明`<uses-permission android:name="android.permission.INTERNET" />`。
- 网络请求最络会在协程中返回请求码和请求结果两个值，所以选择使用了Flow的协程形式进行。
- 因为协程可通过调度器运行在IO线程池，所以使用了okhttp的同步请求函数execute省去回调的嵌套。
- Okhttp3成功返回的结果最终通过使用了高阶函数use进行辅助将其转化成String。use函数内部封装了异常捕获和自动关闭资源。
  