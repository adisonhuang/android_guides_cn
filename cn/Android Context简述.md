# Android Context 简述

## 概述

context提供对应用程序状态信息的访问。它提供 Activitys，Fragments，Services对资源文件，图片，主题/样式和外部目录位置的访问。它还可以访问Android的内置服务，例如那些用于布局填充，键盘的服务和查找 content provider。

## Context 使用场景

下面是几个使用 context 的场景

### 显式启动组件

```java
// 如果MyActivity是应用内部 Activity需要提供 context 
Intent intent = new Intent(context, MyActivity.class);
startActivity(intent);
```

当显式启动一个组件时，两部分信息是必须的：

* 包名，它标识包含组件的应用程序。
* 组件的Java类全路径名称。

如果启动一个应用内组件，可以传入 context，因为可以通过`context.getPackageName()`提取当前应用程序的包名

### 创建View

```java
TextView textView = new TextView(context);
```

context包含View需要的以下信息： 

* 将dp，sp转换为像素需要的设备屏幕大小和尺寸 
* 样式属性 
* 持有`onClick`属性的activity 引用

### 解析XML文件

我们使用context 获取`LayoutInflater,`，它可以将XML布局填充到内存中：

```java
LayoutInflater inflater = LayoutInflater.from(context);
inflater.inflate(R.layout.my_layout, parent);
```

### 发送本地广播

在发送或注册广播接收器时，我们使用context获取`LocalBroadcastManager`：

```java
Intent broadcastIntent = new Intent("custom-action");
LocalBroadcastManager.getInstance(context).sendBroadcast(broadcastIntent);
```

### 查找系统服务

要从应用程序发送通知，需要使用`NotificationManager`系统服务：

```java
//Context 对象可以获取或者启动系统服务
NotificationManager notificationManager = 
    (NotificationManager) getSystemService(NOTIFICATION_SERVICE);

int notificationId = 1;

// Context 是构造RemoteViews的必要条件
Notification.Builder builder = 
    new Notification.Builder(context).setContentTitle("custom title");

notificationManager.notify(notificationId, builder.build());
```

通过 Context 可以查找所有[有效的系统服务](http://developer.android.com/reference/android/content/Context.html#getSystemService(java.lang.String))



## Application VS Activity Context

主题和样式一般在应用层面使用，当然它们也可以在 Activity 层面定制。通过这种方式，Activity可以具有与应用程序的其余部分不同的主题或样式（例如，如果某些页面需要禁用ActionBar）。你会注意到`AndroidManifest.xml`文件中通常有一个针对应用程序的`android:theme`,我们也可以为一个Activity指定一个不同的`android:theme`：

```java
<application
       android:allowBackup="true"
       android:icon="@mipmap/ic_launcher"
       android:label="@string/app_name"
       android:theme="@style/AppTheme" >
       <activity
           android:name=".MainActivity"
           android:label="@string/app_name"
           android:theme="@style/MyCustomTheme">
```

因此，明白Application Context和Activity Context 的区别是很重要的，它们有各自的生命周期。大多数View应该通过Activity Context来访问该应用的主题，样式，尺寸。如果没有明确指定Activity的主题，则默认使用应用程序指定的主题。

在大多数情况下，您应该使用Activity Context 。通常，Java中的关键字`this`引用了类的实例，只要在Activity中需要Context的地方可以使用它来代替。以下示例显示了如何使用此方法显示Toast消息：

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);  
        Toast.makeText(this, "hello", Toast.LENGTH_SHORT).show();
    }
}
```

### 匿名方法

在使用匿名函数时，例如监听器，Java中的关键字`this` 使用使用直接的类声明。在这些情况下，必须指定外部类`MainActivity`来引用Activity实例。

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {

        TextView tvTest = (TextView) findViewById(R.id.abc);
        tvTest.setOnClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View view) {
                  Toast.makeText(MainActivity.this, "hello", Toast.LENGTH_SHORT).show();
              }
          });
        }
    }
```

### Adapter

#### Array Adapter

在为ListView构造Adapter时，通常在布局填充过程中调用`getContext()`。此方法使用context实例化ArrayAdapter：

```java
 if (convertView == null) {
        convertView = 
            LayoutInflater
                .from(getContext())
                .inflate(R.layout.item_user, parent, false);
     }
```

但是，如果使用Application 实例化ArrayAdapter，则主题/样式可能不生效。因此，在这种情况，要确保传递的是 Activity Context。

#### RecyclerView Adapter

```java
public class MyRecyclerAdapter extends RecyclerView.Adapter<MyRecyclerAdapter.ViewHolder> {

    @Override 
    public ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View v = 
            LayoutInflater
                .from(parent.getContext())
                .inflate(R.layout.itemLayout, parent, false);
        
        return new ViewHolder(v);
    }

    @Override
    public void onBindViewHolder(ViewHolder viewHolder, int i) {
        Context context = viewHolder.itemView.getContext();
        if(i == 0) {
            TextView tvMessage = new TextView(context);
            tvMessage.setText("Only displayed for the first item.")

            viewHolder.customViewGroup.addView(tvMessage);
        }
    }

   public static class ViewHolder extends RecyclerView.ViewHolder {
       public FrameLayout customViewGroup;

       public ViewHolder(view imageView) {
           super(imageView);
           customViewGroup = (FrameLayout) imageView.findById(R.id.customViewGroup);
       }
   }
}
```

虽然`ArrayAdapter`需要将Context传递给它的构造函数，但`RecyclerView.Adapter`不需要。相反，当需要填充布局时，可以从父视图中推断出正确的上下文。

关联的`RecyclerView`始终将其自身作为父视图传递给`RecyclerView.Adapter.onCreateViewHolder()`调用。

如果需要在`onCreateViewHolder()`外部使用 context，只要`ViewHolder`实例有效，可以通过`viewHolder.itemView.getContext()`获取 Context。`itemView`在 ViewHolder基类里是一个 public，非空、`final`字段

### 避免内存泄漏

Application Context 通常在创建单例实例时使用，例如需要context来访问系统服务但在多个Activity中复用的自定义管理器类。由于保留对Activity context的引用会导致在Activity不再运行后内存无法回收，因此改用Application Context很重要。

在下面的例子中，如果持有的Context 是一个Activity或Service，当它被Android系统销毁，它将无法被垃圾回收处理，因为CustomManager类持有它的静态引用。

```java
public class CustomManager {
    private static CustomManager sInstance;

    public static CustomManager getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new CustomManager(context);
        }

        return sInstance;
    }

    private Context mContext;

    private CustomManager(Context context) {
        mContext = context;
    }
}
```

### 正确持有context：使用Application Context

为了避免内存泄漏，不要在超出其生命周期的范围内引用Context。检查任何后台线程，待处理handlers或可能持有Context对象的内部类。

在`CustomManager.getInstance()`中正确的做法是持有Application Context的引用。Application Context是一个单例，并且与应用程序进程的生命周期相关联，因此可以安全地持有它的引用。

当需要超出组件生命周期的Context引用时，或者独立于传入的Context的生命周期时，使用Application Context。

```java
public static CustomManager getInstance(Context context) {
    if (sInstance == null) {
        sInstance = new CustomManager(context.getApplicationContext());
    }

    return sInstance;
}
```

## 参考

- [What is a context - Simple Code Stuffs](http://www.simplecodestuffs.com/what-is-context-in-android/)
- [Context, what context - Possible Mobile](https://possiblemobile.com/2013/06/context/)
- [Context: What, where, & how? - 101 Apps](http://www.101apps.co.za/index.php/articles/all-about-using-android-s-context-class.html)
- [What is a context - Stack Overflow](http://stackoverflow.com/questions/3572463/what-is-context-in-android)
- [Avoiding memory leaks](http://android-developers.blogspot.com.tr/2009/01/avoiding-memory-leaks.html)