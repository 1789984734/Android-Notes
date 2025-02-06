#  Activity启动模式
1. standard

适用情况:
默认的启动模式，适用于大多数情况。
每次启动 Activity 时都会创建一个新的实例。
适用于不需要特殊任务栈行为的 Activity。
示例:
普通的页面跳转，例如从一个列表页面跳转到详情页面。

2. singleTop

适用情况:
当 Activity 已经位于任务栈的顶部时，避免创建新的实例，而是重用当前实例。
适用于希望在 Activity 位于顶部时避免创建新实例的情况。
示例:
消息通知点击后启动的 Activity，如果用户多次点击同一个通知，不希望创建多个相同的 Activity 实例。

3. singleTask

适用情况:
任务栈中只保留一个该 Activity 的实例。
如果实例已存在，则重用并调用 onNewIntent()，同时将其之上的所有 Activity 弹出栈。
适用于需要在任务栈中唯一存在的 Activity，如主页面。
示例:
应用的主界面（如主菜单或主页），不希望在导航过程中创建多个实例。

4. singleInstance

适用情况:
Activity 独占一个任务栈，不与其他 Activity 共用任务栈。
适用于需要独立任务栈的 Activity，通常是工具类应用。
示例:
独立的播放器、地图应用等，希望在后台运行时保持独立的任务栈。

# Activity生命周期顺序：

onCreate()

onStart()

onResume()

onPause()

onStop()

onRestart()

onStart()

onResume()

onDestroy()
