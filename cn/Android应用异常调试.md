# 应用异常调试

## 概述

在构建Android应用程序时，您的应用程序必然会时不时崩溃或出现奇怪的现象。当您在模拟器或真机上看到以下弹窗时， 那么你需要知道你遇到了运行时异常：

![](https://camo.githubusercontent.com/1e2423da43686ee4d191d50e785d047dca29b36b/68747470733a2f2f692e696d6775722e636f6d2f6c4d545042666b2e706e67)

不要担心！这是完全正常的，您可以采取一些特定的步骤来解决这些问题。请参阅下面的指南或[幻灯片](https://docs.google.com/presentation/d/1DUigTm6Uh43vatHkB4rFkVVIt1zf7zB7Z5tpGTy2oFY/edit#slide=id.g38d66bc5a_0175)以获得更多关于调试崩溃和查找应用程序意外问题的详细信息。

### 调试心态

作为一名Android开发人员，您需要培养“调试思维”以及防御性编程实践，从而减少编写容易出错的代码。此外，您经常会发现自己担任编码调查员，以了解应用程序崩溃或未按预期工作的原因。下面介绍几个关于调试的关键原则：

- 一个程序能运行起来，**并不意味着它会按预期工作**。这类问题被称为**运行时错误**。这与编译时错误形成了对比，这些错误会阻止应用程序运行并且通常更容易被捕获。
- 将调试视为**填补知识空白**的**机会**。调试是一种可以比以前更好地了解您的应用程序，并通过防御性编程提高您未来编写正确代码能力的机会。
- 调试是软件开发过程中至关重要的一部分。通常你会发现自己在某些日子里花费**更多的时间来调试崩溃或意外的行为，然后编写新的代码**。作为一名移动工程师，这是完全正常的。

### 调试原则

在调试期间遇到意想不到的应用程序问题时，应采用以下优先原则：

- **复现**。相信自己的代码中存在问题并且可以通过相同的步骤复现。在假设代码被破坏之前，请尝试重新启动模拟器/真机，在设备上尝试启动应用程序或重新构建应用程序。
- **隔离**。尝试隔离或减少问题周围的代码，找出最简单的方法来复现问题。注释掉或删除可能使问题复杂化的无关代码。
- **研究**。如果您遇到一个重大意外问题，您可能并不孤单。使用Google搜索任何跟问题相关的描述并通过[stackoverflow](http://stackoverflow.com/)搜索遇到的问题。

有了这种思维和上述原则，我们来看看如何调试我们的应用程序出现的问题。

## 调试程序

当您看到应用程序崩溃并关闭时，诊断和解决此问题的基本步骤如下所示：

1. 在Android监视器（logcat）中查找最终的异常堆栈跟踪信息
2. 查看异常类型，信息和文件的对应行号
3. 在您的应用程序中打开文件并找到对应行号
4. 通过异常类型和信息来诊断问题
5. 如果对问题不熟悉，使用Google搜索答案
6. 根据建议的解决方案进行修复并重新运行应用程序
7. 重复以上步骤，直到崩溃不再发生

这个过程几乎总是以意外的应用程序崩溃开始，如下所示。

### 目睹崩溃

假设我们正在构建一个名为Flixster的简单电影预告片应用程序，该应用程序允许用户浏览Youtube上的新电影并观看预告片。想象一下，我们运行了应用程序，我们想要播放预告片，而我们看到了这个崩溃：

![](https://camo.githubusercontent.com/bc8b6e6b581548836cdf1cef6ab7978054d49029/68747470733a2f2f692e696d6775722e636f6d2f5a323247575a652e676966)



当您看到这个崩溃对话框时，直到**您已经完成以下步骤以确定堆栈跟踪详细信息**之前，**请不要**在该对话框中**按OK**，。

### 设置错误过滤器

首先，在Android Studio中，请务必设置您的Android监视器以仅过滤“Error”以减少不必要的信息：

![](https://camo.githubusercontent.com/62a82c0fb74939ae90da3dbaaadf593ee040ae52/68747470733a2f2f692e696d6775722e636f6d2f66625571396f6b2e676966)



1. 选择“Error”作为要显示的日志级别
2. 选择“Show only selected application ”来过滤消息

这会让你只看到出现的严重问题。

**注意：**可以参阅[博客](http://blog.adisonhyh.com/article/16/)修改`LogCat`的着色。

### 找到堆栈跟踪信息

现在让我们进入Android Studio并选择打开“Android Monitor”。展开监视器，以便您可以轻松读取日志消息。

![](https://camo.githubusercontent.com/7a3c1f2c30c6d2fa377e42b9c371f88fa0a6ef01/68747470733a2f2f692e696d6775722e636f6d2f316174474d31342e676966)



1. 滚动到**错误信息的底部，**寻找一条以`Caused by`开头始终显示在底部的行。这里是错误的重要部分，忽略堆栈跟踪信息的其余大部分。
2. 找到最底部的`Caused by`行，以及与您的activity名称即蓝色链接的行`VideoActivity.java:13`。将它们复制到剪贴板并将它们粘贴到单独的文本文件中以便于查看。

在这种情况下，最底部的“Caused by”行和相邻的蓝色文件链接复制在一起看起来像：

```java
Caused by: java.lang.NullPointerException: Attempt to invoke virtual method
 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' 
 on a null object reference 
      at com.codepath.flixster.VideoActivity.<init>(VideoActivity.java:13)

```

请注意，该行上面的堆栈跟踪块的顶部部分注释`FATAL EXCEPTION`以及第一部分带有`java.lang.RuntimeException`的通用错误，比上面捕获带有“Caused by”的异常更有用，该异常指向真正的罪魁祸首。

### 识别异常

接下来，我们需要确定这个堆栈跟踪信息警告我们的实际问题。为此，我们需要确定问题的以下要素：

1. 文件名
2. 行号
3. 异常类型
4. 异常信息

考虑我们上面复制的异常情况：

![](https://camo.githubusercontent.com/cc947fcc589bc2433d347ed5d54d897144d4849f/68747470733a2f2f692e696d6775722e636f6d2f78484f71356f452e676966)

这个异常需要识别转化为下列要素：

| 要素         | 识别出的罪魁祸首                                             |
| ------------ | ------------------------------------------------------------ |
| **文件名**   | VideoActivity.java                                           |
| **行号**     | 13行                                                         |
| **异常类型** | `java.lang.NullPointerException`                             |
| **异常信息** | `Attempt to invoke virtual method 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' on a null object reference` |

### 打开问题文件

接下来，我们需要打开有问题的文件并找到对应行号。在这种情况下，`VideoActivity.java`第13行看起来像这样：

```java
public class VideoActivity extends YouTubeBaseActivity{
    // -----> LINE 13 is immediately below <--------
    String videoKey = getIntent().getStringExtra("video_key");

    // ...
    protected void onCreate(Bundle savedInstanceState) {
       // ....
    }
}
```

因此，我们知道第13行发生了问题：

```java
String videoKey = getIntent().getStringExtra("video_key");
```

这是解决问题的第一步，因为我们需要确切地知道异常的触发位置。

### 诊断问题

接下来，我们需要使用异常类型和信息来诊断该行的问题。这种情况下的类型是`java.lang.NullPointerException`，信息是`Attempt to invoke virtual method 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' on a null object reference`。

类型是最重要的部分，因为类型数量有限。如果类型是`java.lang.NullPointerException`那么我们知道**某个对象不应该是null**。换句话说，我们在一个对象上调用一个方法或传递一个空值的对象（没有值）。这通常意味着**我们忘记了设置值**或者**错误地设置了值**。

上面的信息给出了方法`getStringExtra`在空对象上被调用的细节。这告诉我们`getIntent()`实际上是null，因为这`getStringExtra`是被调用的对象。这可能看起来很奇怪，为什么`getIntent()`这里是null？我们如何解决这个问题？

### google 搜索异常

即使在我们诊断出这个问题后，我们通常也不知道发生了什么问题。我们知道“ `getIntent()`不应该是 null”。但我们不知道为什么或如何解决。

在这个阶段，我们需要**聪明地利用Googl找解决方案**。你有任何问题，stackoverflow可能都有答案。我们需要确定一个可能找到我们答案的搜索查询。通常是查询类似`android [exception type] [partial exception message]`的关键字。这种情况下的类型是`java.lang.NullPointerException`，信息是`Attempt to invoke virtual method 'java.lang.String android.content.Intent.getStringExtra(java.lang.String)' on a null object reference`。

我们可能这样进行谷歌搜索：`android java.lang.NullPointerException android.content.Intent.getStringExtra(java.lang.String)`。[返回结果](https://www.google.com/#q=android+java.lang.NullPointerException+android.content.Intent.getStringExtra(java.lang.String))。

您通常希望查找“stackoverflow”链接，但在这种情况下，第一个结果[这个论坛](http://www.coderanch.com/t/654627/Android/Mobile/string-intent-null-object-reference)。

滚动到底部，您会发现原始作者的这条消息：

> 我现在意识到我的错误。我正在onCreate方法之外调用getIntent()方法。只要我在onCreate方法中调用getIntent()，它就可以正常工作。

试试这个解决方案吧！他似乎暗示这个问题是因为`getIntent()`在`onCreate`块之外被调用，我们需要在该方法中调用它。

### 处理问题

根据Google提供的stackoverflow或其他网站的建议，我们可以尝试一个可能的修复方法。在这种情况下，可以将`VideoActivity.java`更改为：

```
public class VideoActivity extends YouTubeBaseActivity{
    // ...move the call into the onCreate block
    String videoKey; // declare here
    protected void onCreate(Bundle savedInstanceState) {
       // ....
       // set the value inside the block
       videoKey = getIntent().getStringExtra("video_key");
    }
}
```

现在，我们可以运行应用程序，看看事情是否按预期工作！

### 验证错误是否已修复

让我们重新运行该应用程序并再次尝试：

![](https://camo.githubusercontent.com/4a0b7bcf421311d3c6ea1867aa1f40368842af6f/68747470733a2f2f692e696d6775722e636f6d2f71746378654c412e676966)



真棒！异常似乎已被修复！

### 重复尝试

有时我们在Google搜索后尝试的第一个修复方法无效，或者它让事情变得更糟。有时解决方案是错误的。有时我们必须尝试不同的查询。这里有几条准则：

- 在实际修复之前必须尝试3-4种解决方案是正常的。
- 尝试使用谷歌的第一个[stackoverflow](http://stackoverflow.com/)结果
- 打开多个stackoverflow页面并**查找重复多次的答案**
- 寻找有**绿色复选标记和许多赞同的** stackoverflow答案 。

不要灰心！仔细浏览一小部分网站，只要你**正确查询**，你一定会在80％的案例中找到解决方案。在您找到正确的解决方案之前，请尝试多个查询变体。

在某些情况下，我们需要进一步调查问题。下面概述了调试为什么某些事情出问题的方法。

## 调试方法

除了查找和使用Google搜索错误外，还必须应用其他方法来确定发生了什么问题。如果某些事情不符合我们的预期，这通常会变得很有帮助，我们需要弄清楚原因。Android中最常见的三种调试技术是：

1. **Toasts** -利用[toasts](https://github.com/codepath/android_guides/wiki/Displaying-Toasts)，提醒错误
2. **Logging** - 使用[日志系统](https://developer.android.com/studio/debug/index.html#systemLogWrite)输出日志
3. **断点** - 使用[断点系统](https://developer.android.com/studio/debug/index.html#breakPoints)调试

这些方法旨在让我们能够确定为什么值不如我们预期的那样。我们来看看如何使用它们。

让我们还是以上面的同一个应用为例，当单击电影详细信息页面顶部的图像时，电影预告片开始播放。

```java
public class InfoActivity extends YouTubeBaseActivity {
    private ImageView backdropTrailer;
    private String videoKey;

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_info);
        // ...
        backdropTrailer = (ImageView) findViewById(R.id.ivPoster);
        // Trigger video when image is clicked
        backdropTrailer.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) { 
                // Launch youtube video with key if not null
                if (videoKey != null) {
                    Intent i = new Intent(InfoActivity.this, VideoActivity.class);
                    i.putExtra("videoKey", videoKey);
                    startActivity(i);
                }
            }
        });
    }
}
```

不幸的是，当测试时，我们发现当我们运行该应用并且点击顶部图像时，预告片没有按预期出现：

为什么不按预期点击图像时启动视频？让我们来调试下。

![](https://camo.githubusercontent.com/55e0db5f87a823d17810c7671021f614b342ed62/68747470733a2f2f692e696d6775722e636f6d2f714c534377514b2e676966)



### 用Toasts通知

video activity不会在用户按下图像时启动，但让我们看看我们是否可以缩小问题范围。首先，我们添加 [toast ](https://github.com/codepath/android_guides/wiki/Displaying-Toasts)以确保我们可以开始了解问题。Toasts是在应用程序内弹出的消息。例如：

```
Toast.makeText(this, "Message saved as draft.", Toast.LENGTH_SHORT).show();
```

会产生：

![烤面包](https://camo.githubusercontent.com/bb48a1538b7e86fd42cd87388360d6ccc95a26be/687474703a2f2f646576656c6f7065722e616e64726f69642e636f6d2f696d616765732f746f6173742e706e67)

在中`InfoActivity.java`，我们将在`onClick`监听器方法中添加以下Toast消息：

``` java
public class InfoActivity extends YouTubeBaseActivity {
    // ...
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_info);
        // ...
        backdropTrailer = (ImageView) findViewById(R.id.ivPoster);
        backdropTrailer.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) { 
                // Launch youtube video with key if not null
                if (videoKey != null) {
                    Intent i = new Intent(InfoActivity.this, VideoActivity.class);
                    i.putExtra("videoKey", videoKey);
                    startActivity(i);
                } else {
                    // ----> ADD A TOAST HERE. This means video key is null
                    Toast.makeText(InfoActivity.this, "The key is null!", Toast.LENGTH_SHORT).show();
                } 
            }
        });
    }
}
```

随着toast添加，再次运行，我们看到问题得到证实：

问题是，**videoKey是空的**，因为它应该有一个值。我们尚未解决问题，但我们至少知道最初的问题是什么。

使用[Toast消息](https://github.com/codepath/android_guides/wiki/Displaying-Toasts)，当我们的应用出现问题时，我们可以轻松产生通知，我们甚至不必检查日志。当网络，意图，持久性等故障发生时，请务必添加toast，以便在出现问题时轻松发现问题。

### 使用日志进行调试

接下来，让我们解决toast提示的问题。我们现在知道问题是**videoKey是空的**，因为它应该有一个值。让我们来看看为我们提取视频密钥的方法：

```java
public void fetchMovies(int videoId) {
    // URL should be: https://api.themoviedb.org/3/movie/246655/videos?api_key=KEY
    String url = "https://api.themoviedb.org/3/movie" + movie_id + "/videos?api_key=" + KEY;
    client.get(url, new JsonHttpResponseHandler(){
        @Override
        public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
            JSONArray movieJsonResults = null;
            try {
                movieJsonResults = response.getJSONArray("results");
                JSONObject result = movieJsonResults.getJSONObject(0);
                // Get the key from the JSON
                videoKey = result.getString("key");
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    });
}
```

这种方法在某种程度上不会取得我们期望的key。让我们开始在`onSuccess`方法内部打日志，看看是否正如我们所期望的那样进入到那里。

[Android logger](https://developer.android.com/reference/android/util/Log.html) 很容易使用。日志选项如下：

| 级别    | 方法      |
| ------- | --------- |
| Verbose | `Log.v()` |
| Debug   | `Log.d()` |
| Info    | `Log.i()` |
| Warn    | `Log.w()` |
| Error   | `Log.e()` |

它们按相关性排序，`Log.i()`最不重要。这些方法的第一个参数是分类字符串（可以是我们选择的任何标签），第二个参数是要显示的实际消息。

我们可以通过在代码中添加两行代码来使用日志系统：一个在网络调用前的顶部，一个在内部`onSuccess`，查看它们是否都显示：

```java
Log.e("VIDEOS", "HELLO"); // <------------ LOGGING
client.get(url, new JsonHttpResponseHandler(){
    @Override
    public void onSuccess(int statusCode, Header[] headers, JSONObject response) {
        JSONArray movieJsonResults = null;
        try {
            // LOG the entire JSON response string below
            Log.e("VIDEOS", response.toString()); // <------------ LOGGING
            movieJsonResults = response.getJSONArray("results");
            JSONObject result = movieJsonResults.getJSONObject(0);
            videoKey = result.getString("key");
        } catch (JSONException e) {
            e.printStackTrace();
        }
    }
});
```

在运行应用程序时，第一个日志确实按预期显示，但在Android监视器中`onSuccess`的日志根本**没有显示**：

![](https://camo.githubusercontent.com/c3799c0e9aaf7489801469339a1859adb30c78a3/68747470733a2f2f692e696d6775722e636f6d2f4c464f557266352e676966)

注意我们看到`HELLO`但没有其他行日志。这意味着我们现在知道**onSuccess方法不会被调用**。这意味着我们发送的网络请求由于某种原因失败。我们距离解决这个问题更近了一步。

### 使用断点进行调试

为了查找网络请求失败的原因，我们将使用的最强大的调试工具，它是Android Studio中的调试引擎，它允许我们停止应用并彻底调查我们的环境。

首先，我们必须决定在哪些地方停止应用并进行调查。这是通过设置断点完成的。我们设置两个断点来调查网络请求：

![](https://camo.githubusercontent.com/975b565e406501dddd020a328bce576ca0be3cdd/68747470733a2f2f692e696d6775722e636f6d2f363365365a4d492e67696620)



现在，我们需要使用“debugger”而不是普通的运行命令来运行应用程序：

![](https://camo.githubusercontent.com/653fab480d21de7af047fcaef809b894c936aac6/68747470733a2f2f692e696d6775722e636f6d2f413467714b36722e676966)



一旦调试器连接，我们可以点击电影来触发代码运行。一旦代码碰到断点，整个代码就会暂停，让我们检查一切：

![](https://camo.githubusercontent.com/691b7e6fad0a7d70b50fc1d60d21eefc4293aa96/68747470733a2f2f692e696d6775722e636f6d2f544666704f53692e676966)



在这里，我们能够检查URL并将URL与预期值进行比较。我们的实际网址是“ https://api.themoviedb.org/3/movie246655/videos?api_key=KEY ”，而预期的网址是“ [https://api.themoviedb.org/3/movie/246655/videos?api_key = KEY](https://api.themoviedb.org/3/movie/246655/videos?api_key=KEY) “。非常微妙的差异。你能发现它吗？

然后我们可以点击“继续”（![img](https://camo.githubusercontent.com/24bbff869d9ed95155bc51084579c2b64441649b/68747470733a2f2f692e696d6775722e636f6d2f7368784655684a2e706e67)）继续，直到下一个断点或停止调试（![img](https://camo.githubusercontent.com/3d19501549235a0fcaad2f5804fe26b6d2962f97/68747470733a2f2f692e696d6775722e636f6d2f504554566677732e706e67)）结束会话。

断点非常强大，值得进一步学习。要了解关于断点的更多信息，请查看[Android官方调试指南](https://developer.android.com/studio/debug/index.html#breakPoints)或者[第三方断点指南](https://www.learnhowtoprogram.com/android/user-interface-basics-637d41b1-35dc-400a-bcc3-65794760474d/debugging-breakpoints-and-the-android-debugger)。（译者注：也可以参考我写的[调试技巧](http://blog.adisonhyh.com/article/11/)）

### 解决问题

现在我们知道问题是网址错误拼装导致，我们可以在原始代码中修复`InfoActivity.java`：

```
public void fetchMovies(int videoId) {
    // ADD trailing slash to the URL to fix the issue
    String url = "https://api.themoviedb.org/3/movie/" + // <----- add trailing slash
       movie_id + "/videos?api_key=" + KEY;
    client.get(url, new JsonHttpResponseHandler(){ 
      // ...same as before...
    });
}    
```

并删除我们不需要的日志语句。现在，我们可以尝试再次运行该应用程序：

![](https://camo.githubusercontent.com/4a0b7bcf421311d3c6ea1867aa1f40368842af6f/68747470733a2f2f692e696d6775722e636f6d2f71746378654c412e676966)



真棒！现在视频播放与预期一致！

### 总结

在本节中，我们考察了三种调试应用程序的重要方法：

1. **Toasts** - 在应用中显示消息。可以很好地通知你常见的失败原因。
2. **日志** - 在Android监视器中显示消息。很适合打印出值并查看代码是否正在运行。
3. **断点** - 暂停应用并使用断点执行代码，检查整个环境以确定哪些值不正确或哪些代码未运行。

借助这些工具和Google的强大功能，您应该能够很好地调试在开发应用程序时出现的任何错误或问题。祝你好运！

## 参考

- <http://andressjsu.blogspot.com/2016/07/android-debugging.html>
- <https://developer.android.com/studio/debug/index.html>
- <http://blog.strv.com/debugging-in-android-studio-as/>
- <https://www.youtube.com/watch?v=2c1L19ZP5Qg>
- <https://docs.google.com/presentation/d/1DUigTm6Uh43vatHkB4rFkVVIt1zf7zB7Z5tpGTy2oFY/edit?usp=sharing>