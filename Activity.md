#  1. Activity启动模式
## (1). standard
### 适用情况:
默认的启动模式，适用于大多数情况。
每次启动 Activity 时都会创建一个新的实例。
适用于不需要特殊任务栈行为的 Activity。
### 示例:
普通的页面跳转，例如从一个列表页面跳转到详情页面。
## (2). singleTop
### 适用情况:
当 Activity 已经位于任务栈的顶部时，避免创建新的实例，而是重用当前实例。
适用于希望在 Activity 位于顶部时避免创建新实例的情况。
### 示例:
消息通知点击后启动的 Activity，如果用户多次点击同一个通知，不希望创建多个相同的 Activity 实例。
## (3). singleTask
### 适用情况:
任务栈中只保留一个该 Activity 的实例。
如果实例已存在，则重用并调用 onNewIntent()，同时将其之上的所有 Activity 弹出栈。
适用于需要在任务栈中唯一存在的 Activity，如主页面。
### 示例:
应用的主界面（如主菜单或主页），不希望在导航过程中创建多个实例。
## (4). singleInstance
### 适用情况:
Activity 独占一个任务栈，不与其他 Activity 共用任务栈。
适用于需要独立任务栈的 Activity，通常是工具类应用。
### 示例:
独立的播放器、地图应用、闹钟提醒等，希望在后台运行时保持独立的任务栈。

# 2. Activity生命周期顺序：
### onCreate()->onStart()->onResume()->onPause()->onStop()->onRestart()->onStart()->onResume()->onDestroy()

# 3. LinearLayout、RelativeLayout、FrameLayout三种常用布局的特性，他在布局的时候是怎么计算的？效率如何？
### LinearLayout: 适用于简单的线性排列布局，布局效率较高。
### RelativeLayout: 适用于需要复杂相对位置的布局，布局效率较低。
### FrameLayout: 适用于简单的层叠布局，布局效率最高。
在 Android 开发中，页面跳转慢和页面崩溃是常见的性能与稳定性问题，以下是可能的原因及解决方案：

---

# 4. 页面跳转慢的常见原因及优化
1. **Intent 数据过大**  
   - **现象**：通过 `Intent` 传递大量数据（如 Bitmap、复杂对象）时，跳转会明显卡顿。  
   - **解决方案**：  
     - 使用 `Bundle` 传递轻量数据（如 ID、Key），通过全局变量、数据库或 `ViewModel` 共享数据。  
     - 对大文件使用 `FileProvider` 传递 URI，或通过持久化存储（如 Room、SharedPreferences）中转。

2. **目标 Activity 初始化耗时**  
   - **现象**：目标页面的 `onCreate()` 或 `onResume()` 中执行了耗时操作（如网络请求、复杂计算）。  
   - **解决方案**：  
     - 将耗时操作移到子线程（如 `AsyncTask`、`Coroutine`、`RxJava`）。  
     - 延迟加载非关键资源（如使用 `ViewStub` 或分步加载布局）。

3. **布局渲染过慢**  
   - **现象**：复杂布局（如嵌套多层 `LinearLayout`）导致测量/绘制时间过长。  
   - **优化方法**：  
     - 使用 `ConstraintLayout` 减少嵌套层级。  
     - 通过 Android Studio 的 **Layout Inspector** 或 **Profile GPU Rendering** 工具分析布局性能。  
     - 启用 `merge` 标签或 `include` 复用布局。

4. **过渡动画卡顿**  
   - **现象**：自定义动画复杂或未优化。  
   - **优化方法**：  
     - 简化动画，优先使用系统默认动画（如 `overridePendingTransition`）。  
     - 使用 `Property Animation` 替代 `补间动画`，避免主线程阻塞。

---

# 5. 页面崩溃的常见原因及修复
1. **TransactionTooLargeException**  
   - **触发场景**：通过 `Intent` 传递的数据超过 1MB（不同系统版本阈值不同）。  
   - **解决方案**：  
     - 避免直接传递大对象，改用持久化存储或全局单例。  
     - 使用 `Parcelable` 替代 `Serializable`（更高效）。

2. **空指针异常（NPE）**  
   - **常见场景**：未判空的 `Intent` 数据、Fragment 参数传递错误。  
   - **修复方法**：  
     ```kotlin
     val value = intent?.getStringExtra("key") ?: "" // Kotlin 非空处理
     // Java 中需显式判空
     if (getIntent().hasExtra("key")) {
         String value = getIntent().getStringExtra("key");
     }
     ```

3. **内存泄漏导致 OOM**  
   - **常见原因**：静态引用 `Activity`、未解注册监听器（如 `BroadcastReceiver`）、匿名内部类持有外部引用。  
   - **解决方案**：  
     - 使用 `WeakReference` 或 `ViewModel` 管理生命周期敏感对象。  
     - 在 `onDestroy()` 中解注册监听器、取消网络请求。  
     - 通过工具（如 **LeakCanary**）检测内存泄漏。

4. **主线程阻塞（ANR）**  
   - **现象**：页面跳转时主线程执行耗时操作（如数据库查询、文件 IO）。  
   - **解决方案**：  
     - 使用 `AsyncTask`、`Coroutine` 或 `WorkManager` 异步处理任务。  
     - 避免在 `onCreate()` 中同步加载大量数据。

5. **多线程并发问题**  
   - **场景**：子线程更新 UI 或操作已销毁的 `Activity`。  
   - **修复方法**：  
     - 使用 `runOnUiThread` 或 `Handler` 切换回主线程更新 UI。  
     - 通过 `Lifecycle` 组件（如 `LiveData`）确保操作仅在活跃生命周期执行。
