# 1. BroadcastReceiver和LocalBroadcastReceiver的区别？
### (1). 应用场景
#### BroadcastReceiver用于应用之间的传递消息；
#### LocalBroadcastManager用于应用内部传递消息，比broadcastReceiver
更加高效。
### (2). 安全
#### BroadcastReceiver使用的ContentAPI，所以本质上它是跨应用的，所以在使用它时必须要考虑到不要被别的应用滥用；
#### LocalBroadcastManager不需要考虑安全问题，因为它只在应用内部有效。
### (3) 原理方面
#### BroadcastReceiver是以Binder通讯方式为底层实现的机制不同，
#### LocalBroadcastManager 的核心实现实际还是Handler，只是利用到了IntentFilter的match功能，至于BroadcastReceiver换成其他接口也无所谓，顺便利用了现成的类和概念而已。LocalBroadcastManager因为是Handler 实现的应用内的通信，自然安全性更好，效率更高。
