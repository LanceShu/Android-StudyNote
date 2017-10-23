>定义一个服务：
    public class MyService extends Service {

    public MyService(){
        
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

onBind方法是Service中唯一的一个抽象方法,所以必须在子类里实现；

每个服务中常用的三个方法：

onCreate():在服务**第一次**创建时调用；

onStartCommad():在服务**每次**启动时调用；

onDestroy():在服务销毁时调用；

>启动和停止服务

在Activity中启动一个活动时，先构建一个Intent对象，并调用startService()方法来启动一个服务；

停止一个服务同样先构建出一个Intent对象，并调用stopService()方法来停止服务；

在Activity中，完全是由活动来决定服务合适停止；

在Service中可以调用stopSelf()方法来停止服务；

	public class MyService extends Service {

    public MyService(){

    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("Service","onCreate");
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e("Service","onStartCommad");
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e("Service","onDestroy");
    }
	}

>活动与服务进行通信

借助服务中的onBind()方法来实现控制服务的行为;

在服务中专门创建一个Binder对象，如：

    ` private DownloadBinder downloadBinder = new DownloadBinder();

    class DownloadBinder extends Binder{
        public void startDownload(){
            Log.e("MyService","startDownload executed");
        }

        public int getProgress(){
            Log.e("MyService","getProgress executed");
            return 0;
        }
    }

    public MyService(){

    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return downloadBinder;
    }`

接下里，当一个活动与服务绑定之后，就可以调用服务里面的Binder方法了；

首先创建一个ServiceConnection的匿名类，在里面重写onServiceConnected()方法与onServiceDisconnected()方法，这两个方法分别会在活动与服务成功绑定以及活动与服务断开连接时候调用，随后调用bindService()方法进行绑定、unbindService()进行取绑；**一个服务可以与多个活动进行绑定;**

    `private MyService.DownloadBinder downloadBinder;

    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            downloadBinder = (MyService.DownloadBinder) service;
            downloadBinder.getProgress();
            downloadBinder.startDownload();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    `case R.id.bindService:
          	Intent bindintent = new Intent(this,MyService.class);
            bindService(bindintent,serviceConnection,BIND_AUTO_CREATE);
            break;
     case R.id.unbindService:
            Intent unbindintent = new Intent(this,MyService.class);
            unbindService(serviceConnection);
            break;

>服务的生命周期

实际上每个服务只存在一个实例，无论调用多少次startService()方法，只需要调用一次stopService()或stopSelf()方法，服务就会停止下来；

根据Android系统的机制，一个服务只要被启动或被绑定之后，就会一直处于运行状态，必须要让以上两种条件同时不满足，服务才能被销毁；

**即当调用了startService()方法，又调用了bindService()方法后，要同时调用stopService()和unbindService()方法，onDestroy()才会被执行;**

>服务的更多技巧

1.使用前台服务：

在服务的onCreate()方法中，调用startForeground()方法就会让服务变成一个前台服务，并在系统状态栏显示出来；

    `
    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("Service","onCreate");

        Intent intent = new Intent(this,SignatureVerificationActivity.class);
        PendingIntent pi = PendingIntent.getActivity(this,0,intent,0);
        Notification notification = new Notification.Builder(this)
                .setContentTitle("this is content title")
                .setContentText("this is content text")
                .setWhen(System.currentTimeMillis())
                .setSmallIcon(R.mipmap.ic_launcher)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher))
                .setContentIntent(pi)
                .build();
        startForeground(1,notification);
    }`


2.IntentService：

首先要提供一个无参的构造方法，并且必须在其内部**调用父类的有参构造方法**；onHandleIntent（）实在**子线程**中运行，所以不用担心ANR问题；onDestroy（）会在**运行结束后自动停止**；

    `public class MyIntentService extends IntentService {

    public MyIntentService(){
        super("MyIntentService");
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        Log.e("MyIntentService","onHandleIntent thread id:"+Thread.currentThread().getId());
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e("MyIntentService","onDestroy");
    }
}`