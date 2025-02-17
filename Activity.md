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
# 6. Android中有哪些动画？

---

### **1. 视图动画（View Animation / Tween Animation）**
- **作用对象**: 仅适用于 `View`。
- **类型**:
  - **平移动画** (`TranslateAnimation`): 移动View的位置。
  - **缩放动画** (`ScaleAnimation`): 改变View的尺寸。
  - **旋转动画** (`RotateAnimation`): 围绕中心点旋转。
  - **透明度动画** (`AlphaAnimation`): 调整透明度实现淡入淡出。
- **特点**:
  - 通过XML或代码定义，但**只能控制视觉表现**，实际属性未改变（如点击位置仍在原处）。
- **示例代码**（XML）:
  ```xml
  <!-- res/anim/translate.xml -->
  <translate 
      xmlns:android="http://schemas.android.com/apk/res/android"
      android:duration="300"
      android:fromXDelta="0%"
      android:toXDelta="100%" />
  ```
  ```kotlin
  val animation = AnimationUtils.loadAnimation(context, R.anim.translate)
  view.startAnimation(animation)
  ```

---

### **2. 属性动画（Property Animation）**
- **作用对象**: 任何对象的属性（不仅仅是View）。
- **核心类**:
  - **ValueAnimator**: 通过插值器计算数值变化，适用于自定义属性动画。
  - **ObjectAnimator**: 直接修改目标对象的属性值（如 `alpha`、`translationX`）。
  - **AnimatorSet**: 组合多个动画，支持顺序或并行执行。
- **特点**:
  - **真实改变属性值**（如移动后点击区域同步更新）。
  - 支持自定义插值器和估值器，可控制动画速率（如加速、弹跳效果）。
- **示例代码**:
  ```kotlin
  // 平移 + 透明度变化（2秒）
  ObjectAnimator.ofFloat(view, "translationX", 0f, 200f).apply {
      duration = 2000
      start()
  }

  // 组合动画：先缩放后旋转
  val scaleX = ObjectAnimator.ofFloat(view, "scaleX", 1f, 2f)
  val rotate = ObjectAnimator.ofFloat(view, "rotation", 0f, 180f)
  AnimatorSet().apply {
      playSequentially(scaleX, rotate)
      start()
  }
  ```

---

### **3. 过渡动画（Transition Animation）**
#### **(1) 场景过渡（Scene Transition）**
- **用途**: 同一布局中不同状态的转换（如折叠/展开效果）。
- **实现方式**:
  - 使用 `TransitionManager.beginDelayedTransition()` 或定义不同 `Scene`。
  ```kotlin
  TransitionManager.beginDelayedTransition(rootView, Slide())
  view.visibility = View.VISIBLE // 自动触发动画
  ```

#### **(2) 活动过渡（Activity Transition）**
- **用途**: Activity/Fragment 之间的切换动画。
  - **共享元素过渡**: 两个界面共享的View平滑过渡（如图片从列表项到详情页）。
- **使用步骤**:
  1. **开启窗口特性**（在 `styles.xml`）:
      ```xml
      <item name="android:windowActivityTransitions">true</item>
      ```
  2. **定义共享元素**:
      ```kotlin
      val intent = Intent(this, DetailActivity::class.java)
      val options = ActivityOptionsCompat.makeSceneTransitionAnimation(
          this,
          imageView, // 共享的View
          "image_transition" // 唯一标识（需与目标Activity的transitionName一致）
      )
      startActivity(intent, options.toBundle())
      ```
  3. **目标Activity接收**:
      ```kotlin
      imageView.transitionName = "image_transition" // 设置与跳转时一致的标识
      ```

---

### **4. 矢量动画（Vector Animation）**
- **作用对象**: 矢量图（SVG转为 `VectorDrawable`）。
- **实现方式**:
  - 使用 `AnimatedVectorDrawable`，在XML中定义路径变化。
- **使用场景**: 实现路径变形（如汉堡菜单转箭头、加载图标变形）。
- **示例步骤**:
  1. 定义 `VectorDrawable`（`res/drawable/ic_arrow.xml`）。
  2. 定义动画路径变化（`res/animator/path_morph.xml`）:
      ```xml
      <objectAnimator
          android:duration="300"
          android:propertyName="pathData"
          android:valueFrom="M0,0 L100,0 L50,100 Z" 
          android:valueTo="M0,0 L100,0 L50,100 L50,50 Z"
          android:valueType="pathType"/>
      ```
  3. 组合矢量和动画（`res/drawable/anim_vector.xml`）:
      ```xml
      <animated-vector xmlns:android="..." 
          android:drawable="@drawable/ic_arrow">
          <target
              android:name="箭头路径"
              android:animation="@animator/path_morph"/>
      </animated-vector>
      ```
  4. 代码中使用:
      ```kotlin
      val drawable = imageView.drawable as AnimatedVectorDrawable
      drawable.start()
      ```

---

### **5. 帧动画（Frame Animation）**
- **作用对象**: 一系列静态Drawable（如 `png` 序列）。
- **实现方式**: 使用 `AnimationDrawable`，逐帧播放图像。
- **示例**:
  ```xml
  <!-- res/drawable/loading_animation.xml -->
  <animation-list 
      android:oneshot="false"
      android:visible="true">
      <item android:drawable="@drawable/frame1" android:duration="100"/>
      <item android:drawable="@drawable/frame2" android:duration="100"/>
      ...
  </animation-list>
  ```
  ```kotlin
  imageView.setImageResource(R.drawable.loading_animation)
  (imageView.drawable as AnimationDrawable).start()
  ```

---

### **6. Jetpack Compose 动画**
- **声明式API**: 通过状态驱动动画，简化代码。
- **常用组件**:
  - `animate*AsState`: 自动处理属性过渡（如 `val alpha by animateFloatAsState(targetValue)`）。
  - `Transition`: 管理多个动画状态。
  - `AnimatedVisibility`: 控制组件的显示/隐藏动画。
- **示例**:
  ```kotlin
  var visible by remember { mutableStateOf(false) }
  AnimatedVisibility(
      visible = visible,
      enter = fadeIn() + expandVertically(),
      exit = shrinkHorizontally() + fadeOut()
  ) {
      Text("Hello, Compose!")
  }
  ```

---

### **选择建议**
- **简单视图效果**: View动画（如按钮点击反馈）。
- **属性动态变化**: 属性动画或Jetpack Compose动画（如拖动元素的实时反馈）。
- **界面切换流畅性**: 过渡动画（如共享元素提升用户体验）。
- **复杂矢量图形变化**: 矢量动画（如路径变形图标）。
- **逐帧播放**: 帧动画（如加载进度）。

---

### **性能优化Tips**
- **避免过度绘制**: 减少动画覆盖区域的复杂布局。
- **使用硬件加速**: 开启 `android:hardwareAccelerated="true"`。
- **释放资源**: 动画完成后调用 `animator.cancel()` 防止内存泄漏。

# 7. Fragment的生命周期：
1. **初始化阶段**（从创建到视图加载前）：
    - `onAttach()`: Fragment 与 Activity 相关联时调用。
    - `onCreate()`: Fragment 创建时调用，用于初始化数据（但还未加载界面）。
    - `onCreateView()`: 创建并返回 Fragment 的视图布局。
    - `onViewCreated()`: 创建视图后调用，可以在这里完成视图的相关初始化操作。
    - `onActivityCreated()`: 宿主 Activity 创建完成时调用，表示 Fragment 已经与 Activity 完全结合。

2. **活跃阶段**（从视图显示到用户可见）：
    - `onStart()`: Fragment 由不可见变为可见，进入 "已开始" 状态。
    - `onResume()`: Fragment 进入用户可交互的状态。

3. **退场与销毁阶段**（从视图不可见到销毁）：
    - `onPause()`: Fragment 进入后台（部分或完全不可见）。
    - `onStop()`: Fragment 完全不可见。
    - `onDestroyView()`: Fragment 的视图被销毁，但仍然与 Activity 绑定。
    - `onDestroy()`: Fragment 即将销毁，可用于释放资源。
    - `onDetach()`: Fragment 与 Activity 解除绑定。

---

### **Fragment 生命周期方法的详细说明**

1. **`onAttach(Context context)`**
   - **调用时机**: Fragment 被添加到 Activity 时调用。
   - **功能**: 获取与 Fragment 绑定的 `Context`（通常是 Fragment 所隶属的 Activity）。
   - **示例**:
     ```kotlin
     override fun onAttach(context: Context) {
         super.onAttach(context)
         Log.d("FragmentLifecycle", "onAttach() called")
     }
     ```

2. **`onCreate(Bundle)`**
   - **调用时机**: Fragment 被创建时调用。
   - **功能**: 用于保存和恢复基础状态或初始化非界面数据。
   - **说明**: 此时 Fragment 的 UI 尚未初始化。
   - **示例**:
     ```kotlin
     override fun onCreate(savedInstanceState: Bundle?) {
         super.onCreate(savedInstanceState)
         Log.d("FragmentLifecycle", "onCreate() called")
     }
     ```

3. **`onCreateView(LayoutInflater, ViewGroup, Bundle)`**
   - **调用时机**: Fragment 创建其视图时调用。
   - **功能**: 加载 UI 布局，并返回根视图。
   - **示例**:
     ```kotlin
     override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View? {
         Log.d("FragmentLifecycle", "onCreateView() called")
         return inflater.inflate(R.layout.fragment_layout, container, false)
     }
     ```

4. **`onViewCreated(View, Bundle?)`**
   - **调用时机**: `onCreateView()` 执行完毕后立即调用。
   - **功能**: 在视图已经被创建后执行进一步的初始化工作（如绑定数据和设置监听器）。
   - **示例**:
     ```kotlin
     override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
         super.onViewCreated(view, savedInstanceState)
         Log.d("FragmentLifecycle", "onViewCreated() called")
     }
     ```

5. **`onStart()`**
   - **调用时机**: Fragment 从不可见变为可见时调用。
   - **功能**: 启动动画、刷新数据。
   - **示例**:
     ```kotlin
     override fun onStart() {
         super.onStart()
         Log.d("FragmentLifecycle", "onStart() called")
     }
     ```

6. **`onResume()`**
   - **调用时机**: Fragment 被交互调用后，进入前台可见状态。
   - **功能**: 开始更新 UI 或执行动画。
   - **示例**:
     ```kotlin
     override fun onResume() {
         super.onResume()
         Log.d("FragmentLifecycle", "onResume() called")
     }
     ```

7. **`onPause()`**
   - **调用时机**: 在 Fragment 失去焦点时调用（例如跳转到另一个 Fragment）。
   - **功能**: 暂停交互行为（如暂停播放视频）。
   - **示例**:
     ```kotlin
     override fun onPause() {
         super.onPause()
         Log.d("FragmentLifecycle", "onPause() called")
     }
     ```

8. **`onStop()`**
   - **调用时机**: Fragment 完全不可见时调用。
   - **功能**: 停止可能消耗资源的任务（如后台线程）。
   - **示例**:
     ```kotlin
     override fun onStop() {
         super.onStop()
         Log.d("FragmentLifecycle", "onStop() called")
     }
     ```

9. **`onDestroyView()`**
    - **调用时机**: Fragment 视图即将被销毁时调用。
    - **功能**: 清理视图相关资源（如解除绑定的监听器）。
    - **示例**:
      ```kotlin
      override fun onDestroyView() {
          super.onDestroyView()
          Log.d("FragmentLifecycle", "onDestroyView() called")
      }
      ```

10. **`onDestroy()`**
    - **调用时机**: Fragment 被销毁时调用。
    - **功能**: 清理所有与 Fragment 生命周期无关的资源。
    - **示例**:
      ```kotlin
      override fun onDestroy() {
          super.onDestroy()
          Log.d("FragmentLifecycle", "onDestroy() called")
      }
      ```

11. **`onDetach()`**
    - **调用时机**: Fragment 从 Activity 中分离时调用。
    - **功能**: 最后清理环境，确保没有外部引用。
    - **示例**:
      ```kotlin
      override fun onDetach() {
          super.onDetach()
          Log.d("FragmentLifecycle", "onDetach() called")
      }
      ```

---

### **Fragment 生命周期和 Activity 生命周期的关系**

Fragment 的生命周期很大程度上依赖其宿主 Activity 的生命周期，当 Activity 的状态发生变化时，Fragment 的生命周期会随之改变：
1. 当 Activity 进入 **创建状态**，Fragment 会依次执行：
   - `onAttach()` → `onCreate()` → `onCreateView()` → `onViewCreated()` → `onActivityCreated()` → `onStart()` → `onResume()`。
2. 当 Activity 被 **停止/销毁** 时，Fragment 的对应方法会依次被调用：
   - `onPause()` → `onStop()` → `onDestroyView()` → `onDestroy()` → `onDetach()`。

---

### **可见性相关生命周期**

Fragment 可见性的变化还与 FragmentTransaction 的操作有关，例如 `show`、`hide`、`replace` 以及 `addToBackStack()` 对生命周期的影响，这些操作可能会使 Fragment 进入某种 *“不可见但未销毁”* 的状态。

- **`setUserVisibleHint()`（废弃）**: 可用于懒加载逻辑（早期版本）。
- **`onHiddenChanged(boolean)`**: Fragment 显示或隐藏时调用。
- **Fragment 的 `isVisible()`** 和 `getUserVisibleHint()` 用于判断当前可见状态。

---

### **Fragment 生命周期图示**

以下生命周期流程更清晰地描述了调用顺序：

```
onAttach() → onCreate() → onCreateView() → onViewCreated() → onActivityCreated() → onStart() → onResume()
↓ (Fragment 可见，用户可操作)
onPause() → onStop() → onDestroyView() → onDestroy() → onDetach()
```

---

### **总结**
- **Fragment 的生命周期与其视图的生命周期不是完全一致**。
- 在合适的生命周期方法中完成对应的任务：
  - 数据初始化：`onCreate()`。
  - 绑定视图：`onViewCreated()`。
  - 启动任务：`onStart()` 或 `onResume()`。
  - 清理资源：`onDestroyView()` 或 `onDestroy()`。
