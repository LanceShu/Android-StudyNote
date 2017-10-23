#Kotlin实战——LacWeather

>##简介

该APP是基于**Kotlin语言**以及**借鉴《Android第一行代码》中的酷欧天气**进行编写的，这也算是小猿学习Kotlin的时候，写的第一款Kotlin小demo。

>##主要功能

1. 我的城市：用户收藏的城市（正在实现中...）以及用户当前所在的城市的天气信息；
2. 查询城市：用户可以查询全国各地的城市的天气信息；

>##技术实现

1. LitePal：利用LitePal数据库来存储城市的省市县信息，实现简单的插入与查询；
2. SharePreference：用户存用户查询到的城市的天气信息以及有关的图片背景的Url；
3. Okhttp：用于网络请求接口信息；
4. Json：用于解析城市的省市县的数据；
5. Gson：用于解析城市的天气信息；
6. MaterialDesign：利用DraweLayout+NavigationView来用户展示有关的个性化设置；
7. Service+AlarmManager：利用后台的服务加上Alarm定时技术来实现天气APP在后台一直保持着天气数据的更新；
8. Glide：用于天气图片的加载；

>##依赖库

1. kotlin：

	compile 'org.jetbrains.kotlin:kotlin-stdlib:1.1.1'

    compile 'org.jetbrains.anko:anko-sdk25:0.10.0-beta-1'

    compile 'org.jetbrains.anko:anko-appcompat-v7:0.10.0-beta-1'

2. MaterialDesign：

	compile 'com.android.support:design:25.3.1'

3. LitePal：

	compile 'org.litepal.android:core:1.4.1'

4. Okhttp：

	compile 'com.squareup.okhttp3:okhttp:3.4.1'

5. Gson：

	compile 'com.google.code.gson:gson:2.7'

6. Glide：

	compile 'com.github.bumptech.glide:glide:3.7.0'

>##**Kotlin小知识**

###基本用法：

####1.var/val：var为变量，val为常量（类似于Java中的final，不能更改）

基本类型写法（常量）：

    /*基本常量写法：val 常量名: 数据类型(首字母需大写) = 初始值
    * 基本变量写法：var 常量名: 数据类型(首字母需大写) = 初始值
    * 数据类型后带上"?"，表示该数据可以为null；否则，不能为null；*/
    val a:Int = 0          //整数类型
    val b:Float = 0.0f     //单精度浮点数
    val c:Double = 0.0     //双精度浮点数
    val d:Boolean = false  //布尔类型
    val e:String? = null    //字符串类型
    val f:Char? = null      //字符类型

####2.方法fun的写法：

**kotlin的类class与方法fun，默认都为private**（如果需要继承，则需要使用**open**关键字）；而Java的类class与方法，默认为public；

父类Parent：

    open class Parent{

    /*open class表示该类可以被子类继承；
    * open fun表示该方法可以被重写*/

    open fun eat(){
        System.out.print("i can eat")
    }

	}	

子类Son：

    class Son: Parent() {
    
    /*重写父类的方法，需使用override关键字*/

    override fun eat(){
		System.out.print("i do not want to eat")
    }

	}

kotlin方法的重载与Java一样，知识语法不同：

    class Son: Parent() {

    /*重写父类的方法，需使用override关键字*/

    override fun eat(){
        System.out.print("i do not want to eat")
    }

    fun eat(a: Int){
		//逻辑实现
    }

    fun eat(b: String){
        //逻辑实现
    }

	}

####3.":"的不同用途：

1. 在变量名后使用，":"表示该变量是属于什么数据类型的；例如：var i: Int = 0,表示i属于int类型的;
2. 在类名后使用，":"表示该类继承某类，作用与extends关键字一样；

####4.companion object{}的作用：

在kotlin中使用companion object{},等于在Java中使用public static；例如：

父类中定义一个fun drink()：

    open class Parent {

    /*open class表示该类可以被子类继承；
    * open fun表示该方法可以被重写*/

    open fun eat(){
        System.out.print("i can eat")
    }

    companion object{
        fun drink(){
            System.out.print("i can drink")
        }
    }
	}

在子类中实现父类的drink()方法：

    class Son : Parent() {

    /*重写父类的方法，需使用override关键字*/

    override fun eat(){
        System.out.print("i do not want to eat")
    }

    fun drink(){
        Parent.drink()
    }

	}

###高级用法：

####1.Okhttp请求：

    fun sendOkHttpRequest(address : String,callback: okhttp3.Callback){

        /*address为请求的Url，callback为请求后的回调接口*/

        val client = OkHttpClient()
        val request = Request.Builder()
                .url(address)
                .build()

        /*也可这样写：
        * val client: OkHttpClient = OkHttpClient()
        * val request: Request = Request.Builder()
        *       .url(address)
        *       .build()
        * 这里就体现的kotlin语言的简洁了*/
        
        client.newCall(request).enqueue(callback)
    }

####2.this的用法：

在kotlin的方法中使用this，类似于在Java的方法中使用this，调用本类中的成员方法或成员变量；

**在kotlin的companion object{}中使用this小技巧：**

父类Parent：

    open class Parent {

    /*open class表示该类可以被子类继承；
    * open fun表示该方法可以被重写*/

    open fun eat(){
        System.out.print("i can eat")
    }

    companion object{
        
        fun eat(){
			/*调用本类中的成员方法eat()*/
            this.eat()
        }

    }
	}

子类Son可通过调用类方法来实现父类Parent中的eat()方法：

    class Son : Parent() {

    /*重写父类的方法，需使用override关键字*/

    override fun eat(){
        System.out.print("i do not want to eat")
		//output:	i do not want to eat
    }

    fun drink(){
        
        Parent.eat()
		//output:	i can eat
    }

	}
    
####3.when用法：

在kotlin中，提供了一种新的语法when，用于取代Java语法中的switch，when的语法规则如下：

    fun text1(){
        
        val s = "lance"
        when(s){
            "lance"->{
                System.out.print("lance")
            }
            "shu"->{
                System.out.print("shu")
            }
        }
    }

而在Java中，则是：

    public void text1(){
        
        String s = "lance";
        
        switch (s){
            case "lance":
                System.out.print("lacne");
                break;
            case "shu":
                System.out.print("shu");
                break;
        }
    }

####4有关Android中的控件点击监听事件：

kotlin中每个匿名内部类重写方法的时候，都需**在类的前面加个object**，这样才能重写匿名内部类中的方法；

拿NavigationView的OnNavigationItemSelectedListener（）举个例子（结合when语法）：

    navView!!.setNavigationItemSelectedListener(object : NavigationView.OnNavigationItemSelectedListener{
            navView!!.setNavigationItemSelectedListener(object : NavigationView.OnNavigationItemSelectedListener{
            override fun onNavigationItemSelected(item: MenuItem): Boolean {
                when(item.itemId){
                    R.id.nav_my_city->{...}
                    R.id.nav_change_city->{...}
                    R.id.nav_about->{...}
                    R.id.nav_help->{...}
                }
                return true
            }
        })

普通的控件点击监听事件可以这样写（Button按钮）：

     nav_button.setOnClickListener(object : View.OnClickListener{
            override fun onClick(v: View?) {
				//实现具体逻辑;
               	...
            }
     })

