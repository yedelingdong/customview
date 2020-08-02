引用自《Android开发艺术探索》第15章

Android性能优化

一些有效的性能优化方法，主要内容包括布局优化、绘制优化、内存泄露优化、响应速度优化、ListView优化、
Bitmap优化、线程优化以及一些性能优化建议，同时在介绍响应速度优化的同时还介绍了ANR日志的分析方法。

Android设备作为一种移动设备，不管是内存还是CPU的性能都受到了一定的限制.
Android程序不可能无限制地使用内存和CPU资源，过多地使用内存会导致程序内存溢出，即OOM。而过多地
使用CPU资源，一般是指做大量的耗时任务，会导致手机变得卡顿甚至出现程序无法响应的情况，即ANR。


---布局优化？
A:减少布局文件的层级。
删除布局中无用的控件和层级，其次有选择地使用性能较低的ViewGroup，比如RelativeLayout。
如果布局中既可以使用LinearLayout也可以使用RelativeLayout，那么就采用LinearLayout，因为
RelativeLayout的功能比较复杂，它的布局过程需要花费更多的CPU时间。
FrameLayout和LinearLayout一样都是一种简单高效的ViewGroup。
如果产品效果比较复杂需要通过嵌套的方式来完成，建议使用RelativeLayout,因为ViewGroup的嵌套就
相当于增加了布局的层级，同样会降低程序的性能。

布局优化的另外一种手段是采用<include>标签、<merge>标签和ViewStub。<include>标签主要用于布局重用，
<merge>标签一般和<include>配合使用，它可以降低减少布局的层级，而ViewStub则提供了按需加载的功能，
当需要时才会将ViewStub中的布局加载到内存，这提高了程序的初始化效率，

<include>标签
<include>标签可以将一个指定的布局文件加载到当前的布局文件中
如果<include>指定了这个id属性，同时被包含的布局文件的根元素也指定了id属性，那么以<include>指定的
id属性为准。需要注意的是，如果<include>标签指定了android:layout_*这种属性，那么要求
android:layout_width和android:layout_height必须存在，否则其他android:layout_*形式的属性
无法生效，下面是一个指定了android:layout_*属性的示例。
<include android:id="@+id/new_title"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  layout="@layout/title"/>

<merge>标签
<merge>标签一般和<include>标签一起使用从而减少布局的层级。

ViewStub
ViewStub继承了View，它非常轻量级且宽/高都是0，因此它本身不参与任何的布局和绘制过程。
ViewStub的意义在于按需加载所需的布局文件,提高了程序初始化时的性能。
<ViewStub
  android:id="@+id/stub_import"
  android:inflatedId="@+id/panel_import"
  android:layout="@layout/layout_network_error"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  android:layout_gravity="bottom"
  />
其中stub_import是ViewStub的id，而panel_import是layout/layout_network_error这个布局的
根元素的id。如何做到按需加载呢？在需要加载ViewStub中的布局时，可以按照如下两种方式进行：
(ViewStub)findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
或
View importPanel = (ViewStub)findViewById(R.id.stub_import)).inflate();
当ViewStub通过setVisibility或者inflate方法加载后，ViewStub就会被它内部的布局替换掉，
这个时候ViewStub就不再是整个布局结构中的一部分了。另外，目前ViewStub还不支持<merge>标签???


---绘制优化？
绘制优化是指View的onDraw方法要避免执行大量的操作。
1.onDraw中不要创建新的局部对象，这是因为onDraw方法可能会被频繁调用，这样就会在一瞬间产生
大量的临时对象，这不仅占用了过多的内存而且还会导致系统更加频繁gc，降低了程序的执行效率。
2.onDraw方法中不要做耗时的任务，也不能执行成千上万次的循环操作，尽管每次循环都很轻量级，
但是大量的循环仍然十分抢占CPU的时间片，这会造成View的绘制过程不流畅。Google官方给出的性能优化
典范中的标准，View的绘制帧率保证60fps是最佳的，这就要求每帧的绘制时间不超过16ms（16ms = 1000 / 60），
尽量降低onDraw方法的复杂度总是切实有效的。

Android 4.1（API 级别 16）或更高版本的设备,
设置 -> 开发者选项 -> 监控 -> GPU 渲染模式分析 -> 在屏幕上显示为竖条

可视化 GPU 过度绘制
设置 -> 开发者选项 -> 调试 GPU 过度绘制 -> 显示过度绘制区域
Android 将按如下方式为界面元素着色，以确定过度绘制的次数：
真彩色：没有过度绘制
蓝色：过度绘制 1 次
绿色：过度绘制 2 次
粉色：过度绘制 3 次
红色：过度绘制 4 次或更多次

解决过度绘制问题
1.移除布局中不需要的背景。
2.使视图层次结构扁平化。
3.降低透明度。
https://developer.android.com/topic/performance/rendering/overdraw?hl=zh-cn

使用布局检查器调试布局
打开 Layout Inspector
在连接的设备或模拟器上运行应用。
依次点击 Tools > Layout Inspector。
在随即显示的 Choose Process 对话框中，选择想要检查的应用进程，然后点击 OK。


---内存泄漏优化？
内存泄露的优化分为两个方面，
一方面是在开发过程中避免写出有内存泄露的代码，
另一方面是通过一些分析工具比如Profiler（SDK低版本使用MAT）来找出潜在的内存泄露继而解决。

场景1：静态变量导致的内存泄露
下面的代码将导致Activity无法正常销毁，因为静态变量sContext和mView引用了它。
public class TestActivity extends AppCompatActivity {
    private static Context mContext;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mContext = this;
    }
}

public class TestActivity extends AppCompatActivity {
    private static View mView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mView = new View(this);
    }
}

场景2：单例模式导致的内存泄露
提供一个单例模式的TestManager, TestManager可以接收外部的注册并将外部的监听器存储起来。
public class TestManager {

    public interface OnDataArrivedListener {
        public void onDataArrived(Object data);
    }

    private List<OnDataArrivedListener> mOnDataArrivedListener = new ArrayList<OnDataArrivedListener>();

    private static class SingletonHolder {
        public static final TestManager INSTANCE = new TestManager();
    }

    private TestManager() {}

    public static TestManager getInstance() {
        return SingletonHolder.INSTANCE;
    }

    public synchronized void registerListener(OnDataArrivedListener listener) {
        if (!mOnDataArrivedListener.contains(listener)) {
            mOnDataArrivedListener.add(listener);
        }
    }

    public synchronized void unregisterListener(OnDataArrivedListener listener) {
        mOnDataArrivedListener.remove(listener);
    }
}
接着再让Activity实现OnDataArrivedListener接口并向TestManager注册监听.
下面的代码由于缺少解注册的操作所以会引起内存泄露，泄露的原因是Activity的对象被单例模式的TestManager
所持有，而单例模式的特点是其生命周期和Application保持一致，因此Activity对象无法被及时释放。
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    TestManager.getInstance().registerListener(this);
}
解决方法：
在onDestroy里面调用TestManager.getInstance().unregisterListener(this);解注册

场景3：属性动画导致的内存泄露
从Android 3.0开始，Google提供了属性动画，属性动画中有一类无限循环的动画，如果在Activity中
播放此类动画且没有在onDestroy中去停止动画，那么动画会一直播放下去，尽管无法在界面上看到动画效果了，
并且这个时候Activity的View会被动画持有，而View又持有了Activity，最终引起Activity无法释放。
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    mButton = findViewById(R.id.btn_slide_card);
    ObjectAnimator animator = ObjectAnimator.ofFloat(mButton,"rotation",0,360).setDuration(2000);
    animator.setRepeatCount(ValueAnimator.INFINITE);
    animator.start();
}
解决方法：
在Activity的onDestroy中调用animator.cancel()停止动画。

场景4：Hanlder使用引起的内存泄漏
使用Handler更新UI。因为Handler持有当前Activity的引用导致Activity无法被回收，应该被回收的对象
不能被回收而驻留在堆内存中产生内存泄漏。最终造成OOM。
主线程的ThreadLocal -> Looper -> MessageQueue -> Message -> Handler -> Activity
APP存活，所以主线程一直存在，Looper一直存在，MessageQueue一直存在。发送延迟消息时，如果Activity销毁，
很可能会引起内存泄漏。示例如下：
Handler mHandler = new Handler() {
    @Override
    public void handleMessage(@NonNull Message msg) {
        super.handleMessage(msg);
    }
};
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    mHandler.postDelayed(new Runnable() {
        @Override
        public void run() {
            System.out.println("run 逻辑");
        }
    }, 50 * 1000);
}
进入当前Activity,被延迟的消息会在被处理之前存在于主线程消息队列中50s,而这个消息中又包含了Handler引用，
而mHandler又是一个匿名内部类的实例并持有外部Activity引用，会引起Activity无法回收，导致Activity持有
的资源无法回收引起内存泄漏。
解决方法：
在Activity销毁时将Handler手动置null,或将messagequeue清空，或将Handler改为静态内部类，内部通过弱引用
持有Activity对象。
private static class MyHanlder extends Handler {
    private final WeakReference<TestActivity> mActivity;

    private MyHanlder(TestActivity activity) {
        this.mActivity = new WeakReference<TestActivity>(activity);
    }

    @Override
    public void handleMessage(@NonNull Message msg) {
        TestActivity activity = mActivity.get();
        super.handleMessage(msg);
        if (activity != null) {

        }
    }
}
private static final Runnable myRunnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("run 逻辑");
    }
};

在onCreate方法里面调用
new MyHanlder(this).postDelayed(myRunnable, 50 * 1000);

场景5：多线程引起的内存泄漏
A:可以使用弱引用解决类似问题。


Profiler使用
将堆转储另存为 HPROF 文件
要使用其他 HPROF 分析器（如 jhat），您需要将 HPROF 文件从 Android 格式转换为 Java SE HPROF 格式。
可以使用 android_sdk/platform-tools/ 目录中提供的 hprof-conv 工具执行此操作。运行包含两个参数
（即原始 HPROF 文件和转换后 HPROF 文件的写入位置）的 hprof-conv 命令。例如：
hprof-conv heap-original.hprof heap-converted.hprof

分析内存的技巧
使用 Memory Profiler 时，您应对应用代码施加压力并尝试强制内存泄露。

您还可以通过以下某种方式来触发内存泄露：
1.在不同的 Activity 状态下，先将设备从纵向旋转为横向，再将其旋转回来，这样反复旋转多次。旋转设备经常会使应用
泄露 Activity、Context 或 View 对象，因为系统会重新创建 Activity，而如果您的应用在其他地方保持对这些对象
其中一个的引用，系统将无法对其进行垃圾回收。
2.在不同的 Activity 状态下，在您的应用与其他应用之间切换（导航到主屏幕，然后返回到您的应用）。

Android 3.0及更高版本使用Android Profiler来分析应用的 CPU、内存和网络使用情况。
DDMS -> Android Profiler
Traceview -> Android Studio CPU Profiler
Systrace -> 在命令行中使用 systrace 或在 CPU Profiler 中使用经过简化的系统跟踪。
OpenGL ES 跟踪器 -> 使用 Graphics API Debugger。
Hierarchy Viewer -> Layout Inspector布局检查器
Pixel Perfect -> 布局检查器
网络流量工具 -> Network Profiler。


MAT工具的使用


---响应速度优化和ANR日志分析
响应速度优化的核心思想是避免在主线程中做耗时操作，但是有时候的确有很多耗时操作，怎么办呢？
可以将这些耗时操作放在线程中去执行，即采用异步的方式执行耗时操作。响应速度过慢更多地体现在Activity
的启动速度上面，如果在主线程中做太多事情，会导致Activity启动时出现黑屏现象，甚至出现ANR。
Android规定，Activity如果5秒钟之内无法响应屏幕触摸事件或者键盘输入事件就会出现ANR，
而BroadcastReceiver如果10秒钟之内还未执行完操作也会出现ANR。
怎么定位ANR问题呢？
当一个进程发生ANR以后，系统会在/data/anr目录下创建一个文件traces.txt，通过分析这个文件就能定位出ANR的原因。

场景1：在onCreate方法中sleep20s,应用一定会出现ANR
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    SystemClock.sleep(20 * 1000);
}

为了分析ANR的原因，可以导出traces文件，如下所示，其中．表示当前目录：
adb pull /data/anr/traces.txt .

场景2：onCreate中开启了一个线程，在线程中执行testANR()，而testANR()和initView()都被加了同一个锁，
为了百分之百让testANR()先获得锁，特意在执行initView()之前让主线程休眠了10ms，这样一来initView()
肯定会因为等待testANR()所持有的锁而被同步住，这样就产生了一个稍微复杂些的ANR。要注意子线程和主线程抢占
同步锁的情况。
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    new Thread(new Runnable() {
        @Override
        public void run() {
            testANR();
        }
    });
    SystemClock.sleep(10);
    initView();
}
private synchronized void initView() {
}
private synchronized void testANR() {
    SystemClock.sleep(30 * 1000);
}
首先要有意识地避免出现ANR，其次出现ANR了也不要着急，通过分析traces文件即可定位问题。


---ListView优化和Bitmap优化？
ListView的优化主要分为三个方面：
1.采用ViewHolder并避免在getView中执行耗时操作；
2.根据列表的滑动状态来控制任务的执行频率，比如当列表快速滑动时显然是不太适合开启大量的异步任务的；
3.尝试开启硬件加速来使Listview的滑动更加流畅。
注意Listview的优化策略完全适用于GridView。
Bitmap的优化主要是通过BitmapFactory.Options来根据需要对图片进行采样，采样过程中主要用到了
BitmapFactory.Options的inSampleSize参数。


---线程优化？
线程优化的思想是采用线程池，避免程序中存在大量的Thread。线程池可以重用内部的线程，从而避免了线程的创建
和销毁所带来的性能开销，同时线程池还能有效地控制线程池的最大并发数，避免大量的线程因互相抢占系统资源导致
阻塞现象的发生。在实际开发中，我们要尽量采用线程池，而不是每次都去创建一个Thread对象。


---一些性能优化建议？
· 避免创建过多的对象；
· 不要过多使用枚举，枚举占用的内存空间要比整型大；（根据业务需求和逻辑来判定，如果枚举让信息描述更清晰和易于维护就使用枚举）
· 常量请使用static final来修饰；
· 使用Android系统特有的数据结构，比如SparseArray,SparseBooleanArray,SparseIntArray,SparseLongArray,Pair,
  ArrayMap,ArraySet等，它们在一定条件下具有较好的性能；
· 适当使用弱引用和软引用；
· 采用内存缓存(LruCache)和磁盘缓存(DiskLruCache)；
· 尽量采用静态内部类，避免内部类隐式持有外部类的引用引起内存泄露。


---内存泄露分析:
1.MAT工具(Android Studio 3.0以前)
2.Android Profiler在Android Studio 3.0或者更高版本替代Android Monitor，使用Profiler
分析内存,CPU,network,energy等的使用情况。


---提高程序的可维护性
如何提高代码的可维护性和可扩展性？
1.规范命名，要能正确地传达出变量或者方法的含义，少用缩写，参考Android源码的命名方式，比如私有成员以m开头，
静态成员以s开头，常量则全部用大写字母表示，等等。
2.规范排版，代码的排版上需要留出合理的空白来区分不同的代码块。
3.仅为关键代码添加注释，其他地方不写注释，这就对变量和方法的命名风格提出了很高的要求

代码的层次性是指代码要有分层的概念，对于一段业务逻辑，不要试图在一个方法或者一个类中去全部实现，
而要将它分成几个子逻辑，然后每个子逻辑做自己的事情，这样既显得代码层次分明，又可以分解任务从而
实现简化逻辑的效果。

在写程序的过程中要时刻考虑扩展性，如果这个逻辑发生了改变需要做哪些修改，怎样才能降低修改的工作量，
面向扩展编程会使程序具有较好的扩展性。

合理地使用设计模式可以提高代码可维护性和可扩展性，Android操作系统主要用于移动设备的开发，容易产生
性能瓶颈，因此要控制设计的度，设计不能太牵强，否则就是过度设计了。

设计模式需要理解后灵活运用，才能发挥更好的效果，切记切记！！


