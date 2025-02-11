# Tips:
1. 当长期存在的 lambda 或对象表达式引用在组合期间计算的参数或值时，您应使用 rememberUpdatedState，这在使用 LaunchedEffect 时可能很常见。
例：
![image](https://github.com/user-attachments/assets/7ef57297-0e30-4bf5-91f7-0b5d12f394f7)
```kotlin
@Composable
fun LandingScreen(onTimeout: () -> Unit, modifier: Modifier = Modifier) {
    Box(modifier = modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        // This will always refer to the latest onTimeout function that
        // LandingScreen was recomposed with
        val currentOnTimeout by rememberUpdatedState(onTimeout)

        // Create an effect that matches the lifecycle of LandingScreen.
        // If LandingScreen recomposes or onTimeout changes,
        // the delay shouldn't start again.
        LaunchedEffect(Unit) {
            delay(SplashWaitTime)
            currentOnTimeout()
        }
    }
}
```
2. 延迟列表回到顶部按钮的显示技巧
```kotlin
// Show the button if the first visible item is past
// the first item. We use a remembered derived state to
// minimize unnecessary recompositions
val showButton by remember {
    derivedStateOf {
        listState.firstVisibleItemIndex > 0
    }
}
```
3. 使用延迟列表时，注意要添加key在参数列表里，这会让延迟列表知道那些item在移动，并且只重组那些移动的item，从而提升性能。
