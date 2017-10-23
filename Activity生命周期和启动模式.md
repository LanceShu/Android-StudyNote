#Activity的生命周期和启动模式

##重点

1. 生命周期
2. 启动模式
3. IntentFilter匹配规则

##Activity的生命周期

###典型情况下的生命周期：

1. onCreate：Activity正在**被创建**，可以做一些**初始化**的工作，例如：加载布局；
2. onRestart：表示Activity正在重新启动，当当前Activity从**不可见变为可见状态**时，onRestart就会被调用；
3. onStart：此时Activity已经**可见**了，但还**未出现在前台**，**无法与用户交互**；
4. onResume：Activity已经可见了，并且**出现在前台并开始活动**，注意与onStart的区别；
5. onPause：表示正在停止，在特殊情况下，如果这个时候快速地回到当前的Activity时，那么onResume就会被调用；
6. onStop：即将停止；
7. onDestroy：即将被销毁，可以做一些回收的工作和资源释放;

###问题

1. onStart和onResume、onPause和onStop从描述上来看差不多，对我们来说有什么实质的不同呢？

答：onStart和onStop是从Activity**是否可见**这个角度来回掉的，onResume和onPause是从Activity**是否位于前台**这个角度来回调的；

2. 假设当前的Activity为A，如果这时用户打开一个新的Activity B，那么B的onResume和A的onPause哪个先执行？

答：启动Activity的请求会**由Instrumentation来处理**，然后它**通过Binder向AMS发请求**，AMS内部维护着一个**ActivityStack**并负责栈内的Activity的**状态同步**，AMS通过**ActivityThread**去**同步Activity的状态**从而完成生命周期方法的调用；**旧的Activity先onPause，然后新的Activity再启动**；