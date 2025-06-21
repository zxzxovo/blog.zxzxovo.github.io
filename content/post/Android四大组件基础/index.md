+++
date = '2025-06-21T14:54:32+08:00'
draft = false
title = 'Android四大组件基础'
image = 'nav.svg'
description = "Android应用开发中的四大组件"
categories = ["Code", "Android"]
tags = ["Android", "Code", "Kotlin", "学习"]
+++

# Android应用开发四大组件详解

Android应用开发中常使用系统提供的 **四大组件** ： *Activity, Service, Broadcast Receiver, Content Provider*。

近期学习 《Android 第一行代码》，在这里总结一下四大组件的基础用法，用以巩固。

> 部分代码使用AI生成。

## Android应用工作原理

### 应用启动流程
Android应用程序 **启动** 的过程与PC略有不同。在Android设备上，当用户在Launcher（桌面）点击应用图标时：

1. **Launcher发起请求**：Launcher会将启动该应用所需的相关信息发送给 **AMS**(ActivityManagerService)
2. **AMS处理请求**：AMS经过一系列检查和处理后，会根据该应用的进程是否已经存在来决定要创建一个新的应用进程还是在已有进程中直接启动Activity
3. **进程创建**：如需要新进程，由`Zygote`进程fork出新的应用进程
4. **Activity启动**：在应用进程中创建Activity实例并启动

### Android系统架构
Android系统采用分层架构：
- **应用层(Application)**：各种应用程序
- **应用框架层(Framework)**：提供四大组件等API
- **系统运行库层**：核心库和ART虚拟机
- **Linux内核层**：驱动程序和系统服务

# Activity

`Activity` 是Android应用中 **最基本** 的组件，负责提供用户界面和处理用户交互。每个Activity都代表一个单独的屏幕页面。

## Activity生命周期

Activity拥有完整的生命周期：

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        // onCreate在 A ctivity被创建时调用，用于初始化工作
        Log.d("Activity", "onCreate")
    }
    
    override fun onStart() {
        super.onStart()
        // Activity变为可见时调用
        Log.d("Activity", "onStart")
    }
    
    override fun onResume() {
        super.onResume()
        // Activity获得焦点，用户可以与之交互
        Log.d("Activity", "onResume")
    }
    
    override fun onPause() {
        super.onPause()
        // Activity失去焦点，但仍可见
        Log.d("Activity", "onPause")
    }
    
    override fun onStop() {
        super.onStop()
        // Activity不可见时调用
        Log.d("Activity", "onStop")
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // Activity被销毁时调用，释放资源
        Log.d("Activity", "onDestroy")
    }
    
    override fun onRestart() {
        super.onRestart()
        // Activity从停止状态重新启动时调用
        Log.d("Activity", "onRestart")
    }
}
```

## Activity创建和使用

### 1. 创建Activity类
```kotlin
class SecondActivity : AppCompatActivity() {
    
    companion object {
        const val REQUEST_CODE = 1001
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        
        // 获取传递的数据
        val data = intent.getStringExtra("key_data")
        val number = intent.getIntExtra("key_number", 0)
        
        Log.d("SecondActivity", "接收到数据: $data, 数字: $number")
        
        // 设置返回结果
        findViewById<Button>(R.id.btn_return).setOnClickListener {
            val resultIntent = Intent().apply {
                putExtra("result_data", "返回的数据")
            }
            setResult(RESULT_OK, resultIntent)
            finish()
        }
    }
}
```

### 2. 在AndroidManifest.xml中注册
```xml
<activity 
    android:name=".SecondActivity"
    android:label="第二个页面"
    android:theme="@style/AppTheme" />
```

### 3. Activity间跳转和数据传递

**显式Intent启动：**
```kotlin
class MainActivity : AppCompatActivity() {
    
    companion object {
        const val REQUEST_CODE = 1001
    }
    
    private fun basicJump() {
        // 基本跳转
        val intent = Intent(this, SecondActivity::class.java)
        startActivity(intent)
    }
    
    private fun jumpWithData() {
        // 携带数据跳转
        val intent = Intent(this, SecondActivity::class.java).apply {
            putExtra("key_data", "传递的字符串")
            putExtra("key_number", 123)
        }
        startActivity(intent)
    }
    
    private fun jumpForResult() {
        // 期望返回结果的跳转
        val intent = Intent(this, SecondActivity::class.java)
        startActivityForResult(intent, REQUEST_CODE)
    }
}
```

**隐式Intent启动：**
```kotlin
private fun implicitIntents() {
    // 启动浏览器
    val webIntent = Intent(Intent.ACTION_VIEW).apply {
        data = Uri.parse("https://www.google.com")
    }
    startActivity(webIntent)
    
    // 拨打电话
    val dialIntent = Intent(Intent.ACTION_DIAL).apply {
        data = Uri.parse("tel:10086")
    }
    startActivity(dialIntent)
    
    // 发送邮件
    val emailIntent = Intent(Intent.ACTION_SENDTO).apply {
        data = Uri.parse("mailto:example@gmail.com")
        putExtra(Intent.EXTRA_SUBJECT, "邮件主题")
        putExtra(Intent.EXTRA_TEXT, "邮件内容")
    }
    startActivity(emailIntent)
}
```

### 4. 处理Activity返回结果

**传统方式：**
```kotlin
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    
    when (requestCode) {
        REQUEST_CODE -> {
            if (resultCode == RESULT_OK) {
                val result = data?.getStringExtra("result_data")
                Log.d("MainActivity", "收到返回结果: $result")
                // 处理返回的数据
            }
        }
    }
}
```

**使用Activity Result API（推荐）：**
```kotlin
class MainActivity : AppCompatActivity() {
    
    // 注册Activity Result Launcher
    private val startActivityLauncher = registerForActivityResult(
        ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val data = result.data?.getStringExtra("result_data")
            Log.d("MainActivity", "收到返回结果: $data")
        }
    }
    
    private fun launchSecondActivity() {
        val intent = Intent(this, SecondActivity::class.java)
        startActivityLauncher.launch(intent)
    }
}
```

## Activity启动模式

在AndroidManifest.xml中可以设置Activity的启动模式：

```xml
<activity 
    android:name=".MainActivity"
    android:launchMode="singleTop" />
```

- **standard**：默认模式，每次启动都创建新实例
- **singleTop**：如果Activity在栈顶，则复用实例
- **singleTask**：栈内只保留一个实例
- **singleInstance**：单独占用一个任务栈

# Service

`Service` 是Android中用于执行 **后台任务** 的组件，即应用程序界面并没有显示时，Service仍可以在后台持续运行(Android 8.0之后对后台的限制进一步加强，可以使用Jetpack Workmanager进行管理)。
需要注意的是Service默认还是 *在主线程* 执行的。
Srvice主要有两种启动方式：

## 启动式Service (Started Service)
通过`startService()`启动，适用于执行单一操作且不需要返回结果的场景。

**基本使用步骤：**
1. 创建Service类，继承自Service
2. 重写`onStartCommand()`方法
3. 在AndroidManifest.xml中注册
4. 使用`startService()`启动

```kotlin
class MyService : Service() {
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // 执行后台任务
        Thread {
            try {
                // 模拟耗时操作
                Thread.sleep(5000)
                Log.d("MyService", "任务执行完成")
                stopSelf() // 任务完成后停止服务
            } catch (e: InterruptedException) {
                e.printStackTrace()
            }
        }.start()
        
        return START_STICKY // 服务被杀死后会重新创建
    }
    
    override fun onBind(intent: Intent?): IBinder? {
        return null // 启动式服务返回null
    }
    
    override fun onCreate() {
        super.onCreate()
        Log.d("MyService", "服务被创建")
    }
    
    override fun onDestroy() {
        super.onDestroy()
        Log.d("MyService", "服务被销毁")
    }
}

// 在Activity中启动Service
class MainActivity : AppCompatActivity() {
    
    private fun startMyService() {
        val intent = Intent(this, MyService::class.java)
        startService(intent)
    }
    
    private fun stopMyService() {
        val intent = Intent(this, MyService::class.java)
        stopService(intent)
    }
}
```

## 绑定式Service (Bound Service)
通过`bindService()`启动，适用于需要与Activity进行交互的场景。

```kotlin
class MyBoundService : Service() {
    
    inner class LocalBinder : Binder() {
        fun getService(): MyBoundService = this@MyBoundService
    }
    
    private val binder = LocalBinder()
    
    override fun onBind(intent: Intent?): IBinder = binder
    
    // 提供给客户端调用的方法
    fun getCurrentTime(): String = Date().toString()
    
    fun performCalculation(a: Int, b: Int): Int = a + b
    
    override fun onCreate() {
        super.onCreate()
        Log.d("MyBoundService", "绑定式服务创建")
    }
    
    override fun onDestroy() {
        super.onDestroy()
        Log.d("MyBoundService", "绑定式服务销毁")
    }
}

// 在Activity中绑定Service
class MainActivity : AppCompatActivity() {
    
    private var boundService: MyBoundService? = null
    private var isBound = false
    
    private val connection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            val binder = service as MyBoundService.LocalBinder
            boundService = binder.getService()
            isBound = true
            
            // 可以调用服务的方法
            val time = boundService?.getCurrentTime()
            val result = boundService?.performCalculation(10, 20)
            Log.d("MainActivity", "当前时间: $time, 计算结果: $result")
        }
        
        override fun onServiceDisconnected(name: ComponentName?) {
            isBound = false
            boundService = null
        }
    }
    
    private fun bindMyService() {
        val intent = Intent(this, MyBoundService::class.java)
        bindService(intent, connection, BIND_AUTO_CREATE)
    }
    
    private fun unbindMyService() {
        if (isBound) {
            unbindService(connection)
            isBound = false
        }
    }
    
    override fun onDestroy() {
        super.onDestroy()
        unbindMyService()
    }
}
```

## 前台Service（Foreground Service）
用于执行用户可感知的任务，需要显示持续通知：

```kotlin
class ForegroundService : Service() {
    
    companion object {
        const val CHANNEL_ID = "ForegroundServiceChannel"
        const val NOTIFICATION_ID = 1
    }
    
    override fun onCreate() {
        super.onCreate()
        createNotificationChannel()
    }
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val notification = createNotification()
        startForeground(NOTIFICATION_ID, notification)
        
        // 执行长时间运行的任务
        performLongRunningTask()
        
        return START_STICKY
    }
    
    override fun onBind(intent: Intent?): IBinder? = null
    
    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "前台服务通道",
                NotificationManager.IMPORTANCE_LOW
            )
            val manager = getSystemService(NotificationManager::class.java)
            manager.createNotificationChannel(channel)
        }
    }
    
    private fun createNotification(): Notification {
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle("前台服务运行中")
            .setContentText("正在执行后台任务...")
            .setSmallIcon(R.drawable.ic_notification)
            .build()
    }
    
    private fun performLongRunningTask() {
        Thread {
            // 执行耗时任务
            for (i in 1..100) {
                Thread.sleep(1000)
                Log.d("ForegroundService", "任务进度: $i%")
            }
            stopSelf()
        }.start()
    }
}
```

# Broadcast Receiver

`Broadcast Receiver`用于接收来自系统或其他应用的 **广播消息**，是Android中实现组件间通信的重要方式。

## 广播类型
- **标准广播**：异步执行，所有接收器几乎同时收到
- **有序广播**：同步执行，按优先级依次传递，可被拦截
- **本地广播**：仅在应用内部传播，更安全高效

## 注册方式

### 静态注册
在AndroidManifest.xml中注册，应用未启动也能接收广播：

```xml
<receiver android:name=".MyBroadcastReceiver">
    <intent-filter android:priority="100">
        <action android:name="android.intent.action.BOOT_COMPLETED"/>
        <action android:name="com.example.MY_BROADCAST"/>
    </intent-filter>
</receiver>
```

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context?, intent: Intent?) {
        when (intent?.action) {
            Intent.ACTION_BOOT_COMPLETED -> {
                Log.d("Receiver", "系统启动完成")
            }
            "com.example.MY_BROADCAST" -> {
                val data = intent.getStringExtra("data")
                Log.d("Receiver", "收到自定义广播: $data")
            }
            Intent.ACTION_BATTERY_LOW -> {
                Log.d("Receiver", "电量低")
            }
            Intent.ACTION_SCREEN_ON -> {
                Log.d("Receiver", "屏幕点亮")
            }
        }
    }
}
```

### 动态注册
在代码中注册，生命周期与注册组件相同：

```kotlin
class MainActivity : AppCompatActivity() {
    
    private lateinit var receiver: MyBroadcastReceiver
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 创建广播接收器
        receiver = MyBroadcastReceiver()
        
        // 注册广播接收器
        val filter = IntentFilter().apply {
            addAction("com.example.MY_BROADCAST")
            addAction(Intent.ACTION_BATTERY_CHANGED)
            priority = 100 // 设置优先级
        }
        registerReceiver(receiver, filter)
        
        // 设置发送广播按钮
        findViewById<Button>(R.id.btn_send_broadcast).setOnClickListener {
            sendCustomBroadcast()
        }
    }
    
    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(receiver) // 记得取消注册
    }
    
    private fun sendCustomBroadcast() {
        val intent = Intent("com.example.MY_BROADCAST").apply {
            putExtra("data", "来自MainActivity的数据")
        }
        sendBroadcast(intent)
    }
}
```

## 发送广播

```kotlin
class BroadcastSender : AppCompatActivity() {
    
    private fun sendBroadcasts() {
        // 发送标准广播
        val standardIntent = Intent("com.example.MY_BROADCAST").apply {
            putExtra("data", "标准广播数据")
            putExtra("timestamp", System.currentTimeMillis())
        }
        sendBroadcast(standardIntent)
        
        // 发送有序广播
        val orderedIntent = Intent("com.example.ORDERED_BROADCAST").apply {
            putExtra("data", "有序广播数据")
        }
        sendOrderedBroadcast(orderedIntent, null)
        
        // 发送本地广播（仅应用内部接收）
        val localIntent = Intent("com.example.LOCAL_BROADCAST").apply {
            putExtra("data", "本地广播数据")
        }
        LocalBroadcastManager.getInstance(this).sendBroadcast(localIntent)
    }
    
    // 发送带权限的广播
    private fun sendProtectedBroadcast() {
        val intent = Intent("com.example.PROTECTED_BROADCAST")
        sendBroadcast(intent, "com.example.CUSTOM_PERMISSION")
    }
}
```

## 有序广播处理

```kotlin
class OrderedBroadcastReceiver : BroadcastReceiver() {
    
    override fun onReceive(context: Context?, intent: Intent?) {
        when (intent?.action) {
            "com.example.ORDERED_BROADCAST" -> {
                val data = intent.getStringExtra("data")
                Log.d("OrderedReceiver", "接收到有序广播: $data")
                
                // 可以修改结果数据
                resultData = "处理后的数据"
                
                // 可以终止广播传递（阻止后续接收器接收）
                // abortBroadcast()
                
                // 设置结果给下一个接收器
                setResult(Activity.RESULT_OK, "新的数据", null)
            }
        }
    }
}
```

# Content Provider

`Content Provider`是Android中实现 **跨应用数据共享** 的标准方式，当你想访问其他应用程序提供的数据(如通讯录中的联系人)时，就需要使用该组件。

## 主要特点
- 提供统一的数据访问接口
- 支持增删改查(CRUD)操作
- 使用URI来标识数据
- 自动处理跨进程通信

## 基本使用

### 创建Content Provider

```kotlin
class MyContentProvider : ContentProvider() {
    
    companion object {
        private const val AUTHORITY = "com.example.provider"
        private const val USER_DIR = 0
        private const val USER_ITEM = 1
        
        private val uriMatcher = UriMatcher(UriMatcher.NO_MATCH).apply {
            addURI(AUTHORITY, "users", USER_DIR)
            addURI(AUTHORITY, "users/#", USER_ITEM)
        }
    }
    
    private lateinit var dbHelper: MyDatabaseHelper
    
    override fun onCreate(): Boolean {
        dbHelper = MyDatabaseHelper(context, "MyApp.db", null, 1)
        return true
    }
    
    override fun query(
        uri: Uri,
        projection: Array<out String>?,
        selection: String?,
        selectionArgs: Array<out String>?,
        sortOrder: String?
    ): Cursor? {
        val db = dbHelper.readableDatabase
        
        return when (uriMatcher.match(uri)) {
            USER_DIR -> {
                db.query("users", projection, selection, 
                        selectionArgs, null, null, sortOrder)
            }
            USER_ITEM -> {
                val userId = uri.pathSegments[1]
                db.query("users", projection, "id = ?", 
                        arrayOf(userId), null, null, sortOrder)
            }
            else -> null
        }
    }
    
    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        val db = dbHelper.writableDatabase
        
        return when (uriMatcher.match(uri)) {
            USER_DIR -> {
                val newUserId = db.insert("users", null, values)
                val newUri = Uri.parse("content://$AUTHORITY/users/$newUserId")
                
                // 通知数据变化
                context?.contentResolver?.notifyChange(uri, null)
                newUri
            }
            else -> null
        }
    }
    
    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<out String>?): Int {
        val db = dbHelper.writableDatabase
        
        val deletedRows = when (uriMatcher.match(uri)) {
            USER_DIR -> {
                db.delete("users", selection, selectionArgs)
            }
            USER_ITEM -> {
                val userId = uri.pathSegments[1]
                db.delete("users", "id = ?", arrayOf(userId))
            }
            else -> 0
        }
        
        if (deletedRows > 0) {
            context?.contentResolver?.notifyChange(uri, null)
        }
        return deletedRows
    }
    
    override fun update(
        uri: Uri,
        values: ContentValues?,
        selection: String?,
        selectionArgs: Array<out String>?
    ): Int {
        val db = dbHelper.writableDatabase
        
        val updatedRows = when (uriMatcher.match(uri)) {
            USER_DIR -> {
                db.update("users", values, selection, selectionArgs)
            }
            USER_ITEM -> {
                val userId = uri.pathSegments[1]
                db.update("users", values, "id = ?", arrayOf(userId))
            }
            else -> 0
        }
        
        if (updatedRows > 0) {
            context?.contentResolver?.notifyChange(uri, null)
        }
        return updatedRows
    }
    
    override fun getType(uri: Uri): String? {
        return when (uriMatcher.match(uri)) {
            USER_DIR -> "vnd.android.cursor.dir/vnd.com.example.provider.users"
            USER_ITEM -> "vnd.android.cursor.item/vnd.com.example.provider.users"
            else -> null
        }
    }
}
```

### 数据库帮助类

```kotlin
class MyDatabaseHelper(
    context: Context?,
    name: String?,
    factory: SQLiteDatabase.CursorFactory?,
    version: Int
) : SQLiteOpenHelper(context, name, factory, version) {
    
    companion object {
        private const val CREATE_USERS = """
            CREATE TABLE users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                age INTEGER NOT NULL,
                email TEXT
            )
        """
    }
    
    override fun onCreate(db: SQLiteDatabase?) {
        db?.execSQL(CREATE_USERS)
    }
    
    override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
        db?.execSQL("DROP TABLE IF EXISTS users")
        onCreate(db)
    }
}
```

### 在AndroidManifest.xml中注册
```xml
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.provider"
    android:exported="true" />
```

### 访问Content Provider

```kotlin
class ContentProviderClient : AppCompatActivity() {
    
    private val userUri = Uri.parse("content://com.example.provider/users")
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_content_provider_client)
        
        setupButtons()
    }
    
    private fun setupButtons() {
        findViewById<Button>(R.id.btn_query).setOnClickListener { queryUsers() }
        findViewById<Button>(R.id.btn_insert).setOnClickListener { insertUser() }
        findViewById<Button>(R.id.btn_update).setOnClickListener { updateUser() }
        findViewById<Button>(R.id.btn_delete).setOnClickListener { deleteUser() }
    }
    
    private fun queryUsers() {
        val cursor = contentResolver.query(userUri, null, null, null, null)
        
        cursor?.use {
            while (it.moveToNext()) {
                val id = it.getInt(it.getColumnIndexOrThrow("id"))
                val name = it.getString(it.getColumnIndexOrThrow("name"))
                val age = it.getInt(it.getColumnIndexOrThrow("age"))
                val email = it.getString(it.getColumnIndexOrThrow("email"))
                
                Log.d("ContentProvider", "用户: ID=$id, 姓名=$name, 年龄=$age, 邮箱=$email")
            }
        }
    }
    
    private fun insertUser() {
        val values = ContentValues().apply {
            put("name", "张三")
            put("age", 25)
            put("email", "zhangsan@example.com")
        }
        
        val newUri = contentResolver.insert(userUri, values)
        Log.d("ContentProvider", "插入用户成功，URI: $newUri")
    }
    
    private fun updateUser() {
        val values = ContentValues().apply {
            put("age", 26)
            put("email", "zhangsan_new@example.com")
        }
        
        val updatedRows = contentResolver.update(
            userUri,
            values,
            "name = ?",
            arrayOf("张三")
        )
        Log.d("ContentProvider", "更新了 $updatedRows 行数据")
    }
    
    private fun deleteUser() {
        val deletedRows = contentResolver.delete(
            userUri,
            "name = ?",
            arrayOf("张三")
        )
        Log.d("ContentProvider", "删除了 $deletedRows 行数据")
    }
    
    // 监听数据变化
    private val contentObserver = object : ContentObserver(Handler(Looper.getMainLooper())) {
        override fun onChange(selfChange: Boolean) {
            super.onChange(selfChange)
            Log.d("ContentProvider", "数据发生变化")
            queryUsers() // 重新查询数据
        }
    }
    
    override fun onStart() {
        super.onStart()
        // 注册内容观察者
        contentResolver.registerContentObserver(userUri, true, contentObserver)
    }
    
    override fun onStop() {
        super.onStop()
        // 取消注册内容观察者
        contentResolver.unregisterContentObserver(contentObserver)
    }
}
```

## 访问系统Content Provider

```kotlin
class SystemContentProviderExample : AppCompatActivity() {
    
    private fun accessContacts() {
        // 访问联系人
        val contactsUri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI
        val projection = arrayOf(
            ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME,
            ContactsContract.CommonDataKinds.Phone.NUMBER
        )
        
        val cursor = contentResolver.query(contactsUri, projection, null, null, null)
        
        cursor?.use {
            while (it.moveToNext()) {
                val name = it.getString(it.getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME))
                val number = it.getString(it.getColumnIndexOrThrow(ContactsContract.CommonDataKinds.Phone.NUMBER))
                Log.d("Contacts", "联系人: $name, 电话: $number")
            }
        }
    }
    
    private fun accessMediaFiles() {
        // 访问媒体文件
        val mediaUri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI
        val projection = arrayOf(
            MediaStore.Images.Media._ID,
            MediaStore.Images.Media.DISPLAY_NAME,
            MediaStore.Images.Media.SIZE
        )
        
        val cursor = contentResolver.query(mediaUri, projection, null, null, null)
        
        cursor?.use {
            while (it.moveToNext()) {
                val id = it.getLong(it.getColumnIndexOrThrow(MediaStore.Images.Media._ID))
                val name = it.getString(it.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME))
                val size = it.getLong(it.getColumnIndexOrThrow(MediaStore.Images.Media.SIZE))
                Log.d("Media", "图片: ID=$id, 名称=$name, 大小=$size")
            }
        }
    }
}
```

# 总结

Android四大组件构成了Android应用的核心架构：

- **Activity**：负责用户界面的展示和交互，是用户能看到和操作的页面
- **Service**：处理后台任务和长时间运行的操作，即使应用不在前台也能工作
- **Broadcast Receiver**：实现组件间的消息通信，监听系统和应用事件
- **Content Provider**：提供跨应用的数据共享机制，统一数据访问接口

## 组件间协作示例

在实际开发中，四大组件经常需要协同工作：

```kotlin
class ComponentCollaborationExample : AppCompatActivity() {
    
    private lateinit var downloadReceiver: BroadcastReceiver
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_collaboration)
        
        setupBroadcastReceiver()
        
        findViewById<Button>(R.id.btn_start_download).setOnClickListener {
            startDownload()
        }
    }
    
    private fun setupBroadcastReceiver() {
        // 注册下载完成广播接收器
        downloadReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context?, intent: Intent?) {
                when (intent?.action) {
                    "com.example.DOWNLOAD_COMPLETE" -> {
                        val fileName = intent.getStringExtra("file_name")
                        val filePath = intent.getStringExtra("file_path")
                        
                        // 更新UI
                        updateDownloadUI(fileName)
                        
                        // 通过ContentProvider保存文件信息
                        saveFileInfo(fileName, filePath)
                    }
                }
            }
        }
        
        val filter = IntentFilter("com.example.DOWNLOAD_COMPLETE")
        registerReceiver(downloadReceiver, filter)
    }
    
    private fun startDownload() {
        // Activity启动Service执行下载任务
        val serviceIntent = Intent(this, DownloadService::class.java).apply {
            putExtra("download_url", "https://example.com/file.zip")
        }
        startService(serviceIntent)
    }
    
    private fun updateDownloadUI(fileName: String?) {
        // 更新UI显示下载完成
        findViewById<TextView>(R.id.tv_status).text = "下载完成: $fileName"
    }
    
    private fun saveFileInfo(fileName: String?, filePath: String?) {
        // 通过ContentProvider保存下载的文件信息
        val values = ContentValues().apply {
            put("file_name", fileName)
            put("file_path", filePath)
            put("download_time", System.currentTimeMillis())
        }
        
        val uri = Uri.parse("content://com.example.provider/files")
        contentResolver.insert(uri, values)
    }
    
    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(downloadReceiver)
    }
}

// 下载服务
class DownloadService : Service() {
    
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        val downloadUrl = intent?.getStringExtra("download_url")
        
        Thread {
            // 模拟下载过程
            Thread.sleep(3000)
            
            // 下载完成后发送广播
            val broadcastIntent = Intent("com.example.DOWNLOAD_COMPLETE").apply {
                putExtra("file_name", "downloaded_file.zip")
                putExtra("file_path", "/storage/downloads/downloaded_file.zip")
            }
            sendBroadcast(broadcastIntent)
            
            stopSelf()
        }.start()
        
        return START_NOT_STICKY
    }
    
    override fun onBind(intent: Intent?): IBinder? = null
}
```

