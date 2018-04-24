# Android 目录结构

## 概述

在一个 android 项目中，最常使用编辑的文件夹是：

* `src` - 项目源码。它包括 Activity “Controller”，models 和 helpers 等等
* `res` - 项目资源文件。它包含图像文件，strings，布局文件，和其他资源文件
* `res/layout` -  布局 XML文件，声明每个 Activity 使用的 view和 布局 或者部分 view 如 list items
* `res/values` - 属性 XML 文件，包含[strings.xml](https://github.com/codepath/android_guides/wiki/Understanding-App-Resources#defining-a-string-resource), dimens.xml, [styles.xml](https://github.com/codepath/android_guides/wiki/Styles-and-Themes), colors.xml, [themes.xml](https://github.com/codepath/android_guides/wiki/Developing-Custom-Themes)等等
* `res/drawable` - 这里包含不同屏幕密度的图形文件

最常使用编辑的文件是：

* `res/layout/activity_foo.xml` - 该文件描述 Activity UI的布局。这意味着将每个View对象放置在一个应用程序屏幕上
* `src/.../FooActivity.java` - Activity  “Controller” 它使用 View 来构造，并处理一个应用程序屏幕的所有事件和逻辑。
* `AndroidManifest.xml` - 应用清单文件，它包含应用相关信息，如最小支持 android 版本，权限等

其余不常编辑的文件夹：

* `gen` - 生成 java 代码文件，仅仅供 android 内部使用
* `assets` - 项目不编译文件，较少使用
* `libs` - 在使用[Gradle](https://github.com/codepath/android_guides/wiki/Getting-Started-with-Gradle)之前,这个目录是用来放置和项目关联的jar包

## 参考

- <http://developer.android.com/tools/projects/index.html#ApplicationProjects>
- <http://www.codeproject.com/Articles/395614/Basic-structure-of-an-Android-project>
- <http://mobile.tutsplus.com/tutorials/android/android-sdk-app-structure/>