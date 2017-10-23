#Android网络请求——OkHttp的简单使用
OKHttp的项目地址：**https://github.com/square/okhttp**

> 使用前，添加OkHttp库的依赖：

```
dependencies {
    compile 'com.squareup.okhttp3:okhttp:3.5.0'
}
```
添加上述依赖后，会自动下载两个库：okHttp库与Okio库*(后者是前者的通信基础)*;

> OkHttp的使用步骤：

1.首先创建一个OkHttpClient的实例：

```
OkHttpClient client = new OkHttpClient();
```
2.接下来创建一个Request对象，用于发送Http请求：

```
Request request = new Request.Builder().build();
```
Ps:这只是一个空的Request对象，在这里并没有什么实际意义，但我们可以在build()之前连缀其他方法来丰富Request对象:
例如：

 1. url():Http请求的目的地址;
 2. method():Http请求的方法（例如GET、POST等等）；
 3. header():请求头（例如：Accept、Host等等）;
 4. addHeader():在header()后面继续追加请求头；
 5. removeHeader():移除所有的请求头； 
 6. post():将FromBody封装好的body,POST请求出去;
 7. build():创建Request对象;

例如：用OKHttp请求百度首页（GET请求）:

```
Request request = new Request.Builder().url("http://www.baidu.com").build();
```
POST请求会比GET请求复杂一点，主要是需要提交一些参数，提交这些参数时需要先构建出一个RequestBody对象用来存放待提交的参数：

```
FormBody fromBody = newFormBody.Builder()
.add("username","admin")
.add("password","123456")
.build();
```
然后在Request.Builder中调用post()方法，并将RequestBody对象传入：

```
Request request = new Request.Builder()
.url("http://www.baidu.com")
.post(formBody)
.build();
```
两种不同的请求方式简单介绍了一下，有兴趣的小伙伴可以研究一下OKHttp的Request源码了解更多不同的请求格式；
最后，我们调用OKHttpClient的newCall()来创建一个Call对象，并调用它的execute()方法来发送请求并获取服务器返回的数据：

```
Response response = client.newCall(request).execute();
```
Resp对象就是服务器返回的数据了，我们可以使用字符串得到返回的具体内容：

```
String data = response.body().string();
```
Ps：注意，这里是**string()**，而不是toString()。
这样，就完成了OKHttp的简单使用了；

----------
温馨提示：所有的网络请求都应在子线程中进行，因为**网络请求**属于**耗时操作**，如果所有耗时操作都在主线程中进行的话，可能会造成主线程阻塞，因为所有的网络请求操作都应**开启子线程**来进行；但是，**UI的更新不能在子线程中更新**，如果要更新UI，可以使用**Handler对象**或者使用**runOnUIThread（）**来进行UI更新；
