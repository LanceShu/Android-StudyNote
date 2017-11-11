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

问1： onStart和onResume、onPause和onStop从描述上来看差不多，对我们来说有什么实质的不同呢？

答：onStart和onStop是从Activity**是否可见**这个角度来回掉的，onResume和onPause是从Activity**是否位于前台**这个角度来回调的；

问2： 假设当前的Activity为A，如果这时用户打开一个新的Activity B，那么B的onResume和A的onPause哪个先执行？

答：启动Activity的请求会**由Instrumentation来处理**，然后它**通过Binder向AMS发请求**，AMS内部维护着一个**ActivityStack**并负责栈内的Activity的**状态同步**，AMS通过**ActivityThread**去**同步Activity的状态**从而完成生命周期方法的调用；**旧的Activity先onPause，然后新的Activity再启动**；

###异常情况下的生命周期：

####资源相关的系统配置发生改变导致Activity被杀死并重新创建

1. 由于Activity是在**异常情况下终止**的，系统会调用**onSaveInstanceState**来保存当前的Activity的状态，这个方法的**调用时机是在onStop之前**，**可以在onPause之前或者之后**；
2. onSaveInstanceState**只会出现在Activity被异常终止的情况下调用**，正常情况下系统不会回调这个方法；
3. 当Activity**被重新创建后**，系统会调用onRestoreInstanceState，并且**把Activity销毁时的所保存的Bundle对象**作为参数同时传递给onRestoreInstanceState和onCreate方法，**onRestoreInstanceState的调用时机在onStart之后**；
4. 和Activity一样，**每个View都有onSaveInstanceState和onRestoreInstanceState**这两个方法；
5. 关于保存和恢复View层的数据，系统的工作流程是这样的：首先Activity**被意外终止**时，Activity会调用onSaveInstanceState去保存数据，然后Activity会**委托Window**去保存数据，接着Window**再委托上面的顶级容器**去保存数据，最后顶级容器**再去一一通知它的子元素去保存数据**；这是一种典型的**委托思想**，上层委托下层、父容器委托子元素去处理事情；
6. onSaveInstanceState方法只会在Activity**即将被销毁**并且**有机会重新显示**的情况下去调用它；系统只会在Activity异常终止的时候才会调用onSaveInstanceState和onRestoreInstanceState来存储和恢复数据；

####资源内存不足导致低优先级的Activity被杀死

1. Activity按照优先级从高到低，可以分为：
		1）.前台Activity——正在和用户交互的Activity，优先级最高；
		2）.可见但非前台的Activity——比如Activity中弹出一个对话框，导致Activity可见但是位于后台不可与用户交互；
		3）.后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低；
2. 如果一个进程没有四大组件在执行，那么这个进程很快被系统杀死；所以一般后台工作都在Service中执行；

##Activity的启动模式：

###Activity的LaunchMode

1. standard（标准模式）：系统的默认模式，每次启动一个Activity都会创建一个新的实例，不管该实例是否存在；**当我们使用ApplicationContext去启动standard模式的Activity时会报错，因为非Activity类型的Context并没有所谓的任务栈**；
2. singleTop（栈顶复用模式）：如果新的Activity位于任务栈的**栈顶**，那么此Activity不会被重新创建，同时它的**onNewIntent**方法会被回调，通过此方法的参数我们可以取出当前的请求的信息；
3. singleTask（栈内复用模式）：这是一种**单例模式**，在这种模式下，只要Activity在一个栈中存在，那么多次启动此Activity都不会重新创建实例，系统也会回调onNewInstent方法；
4. singleInstance（单实例模式）：是一种**加强的singleTask模式**，加强了一点，那就是具有**该模式的Activity只能单独地位于一个任务栈中**；

###Activity的Flags
1. 设定Activity的启动模式：**FLAG_ACTIVITY_NEW_TASK**（singleTask） 与 **FLAG_ACTIVITY_SINGLE_TOP**（singleTop）
2. 标记位：**FLAG_ACTIVITY_CLEAR_TOP**（将位于它上面的activity都出栈，standard模式下连同自己以及自己上面的activity出栈，然后创建新的实例入栈） 和 **FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS**（不会出现在activity的历史列表中）；

##IntentFilter的匹配规则
1. 启动Activity的方法有两种：显式调用与隐式调用；
2. 隐式调用需要明确地指定组件信息，隐式调用需要Intent能够匹配目标组件的IntentFilter中所设置的过滤信息；IntentFilter中的过滤信息有action（行为）、category（类别）、data（数据）；

###匹配规则：
1. action的匹配规则：action是一个**字符串**，**Intent中的action必须能够和过滤规则中的action匹配**，这里的匹配是指**action字符串的值完全一样**；一个或类别规则中有多个action，只要Intent中的action能够和过滤规则中的**任何一个action相同**即可匹配成功；action的匹配规则要求Intent中的**action存在**且**必须和过滤规则中的其中一个action相同**，区分大小写；
2. category匹配规则：它的匹配规则与action的匹配规则不同，它要求Intent中**如果含有category**，那么**所有的category都必须和过滤规则中的其中一个category相同**；**Intent中可以没有category**；系统在调用startActivity或者startActivityForResult的时候会默认为Intent加上DEFAULT这个category，为了我们的activity能够接受隐式调用，就必须在intent-filter中指定DEFAULT这个category；
3. data的匹配规则：data的匹配规则与action的类似，如果**过滤规则中定义了data**，那么**Intent中必须也要定义可匹配的data**；
