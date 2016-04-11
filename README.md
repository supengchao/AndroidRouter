# ActivityRouter

### [English README.md here](https://github.com/mzule/ActivityRouter/blob/master/README-en.md)

## 功能

支持给`Activity`定义URL，这样可以通过URL跳转到`Activity`，支持在浏览器以及app中跳入。

![image](https://raw.githubusercontent.com/mzule/ActivityRouter/master/gif/router.gif)

![image](https://raw.githubusercontent.com/mzule/ActivityRouter/master/gif/http.gif)

## 集成
根目录build.gradle

``` groovy
buildscript {
  dependencies {
    classpath 'com.neenbedankt.gradle.plugins:android-apt:1.7'
  }
}
```

项目app/build.gradle
``` groovy
apply plugin: 'android-apt'

dependencies {
	compile 'com.github.mzule.activityrouter:activityrouter:1.1.1'
	apt 'com.github.mzule.activityrouter:compiler:1.1.1'
}

```

在`AndroidManifest.xml`配置

``` xml
<activity
    android:name="com.github.mzule.activityrouter.router.RouterActivity"
    android:theme="@android:style/Theme.NoDisplay">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="mzule" /><!--改成自己的scheme-->
    </intent-filter>
</activity>
```
在需要配置的`Activity`上添加注解

``` java
@Router("main")
public class MainActivity extends Activity {
	...
}
```
这样就可以通过`mzule://main`来打开`MainActivity`了。

## 进阶

### 支持配置多个地址

``` java
@Router({"main", "root"})
```

`mzule://main`和`mzule://root`都可以访问到同一个`Activity`


### 支持获取url中`?`传递的参数

``` java
@Router("main")
```
上面的配置，可以通过`mzule://main?color=0xff878798&name=you+are+best`来传递参数，在`MainActivity#onCreate`中通过`getIntent().getStringExtra("name")`的方式来获取参数，所有的参数默认为`String`类型，但是可以通过配置指定参数类型，后面会介绍。

### 支持在path中定义参数

``` java
@Router("main/:color")
```

通过`:color`的方式定义参数，参数名为`color`，访问`mzule://main/0xff878798`，可以在`MainActivity#onCreate`通过`getIntent().getStringExtra("color")`获取到color的值`0xff878798`

### 支持多级path参数

``` java
@Router("user/:userId/:topicId/:commentId")

@Router("user/:userId/topic/:topicId/comment/:commentId")
```

上面两种方式都是被支持的，分别定义了三个参数，`userId`,`topicId`,`commentId`


### 支持指定参数类型

``` java
@Router(value = "main/:color", intExtra = "color")
```
这样指定了参数`color`的类型为`int`，在`MainActivity#onCreate`获取color可以通过`getIntent().getIntExtra("color", 0)`来获取。支持的参数类型有`int`,`long`,`short`,`byte`,`char`,`float`,`double`,`boolean`，默认不指定则为`String`类型。

### 支持优先适配

``` java
@Router("user/:userId")
public class UserActivity extends Activity {
	...
}

@Router("user/statistics")
public class UserStatisticsActivity extends Activity {
	...
}
```
假设有上面两个配置，

不支持优先适配的情况下，`mzule://user/statistics`可能会适配到`@Router("user/:userId")`，并且`userId=statistics`

支持优先适配，意味着，`mzule://user/statistics`会直接适配到`@Router("user/statistics")`，不会适配前一个`@Router("user/:userId")`

### 支持Callback

``` java
public class App extends Application implements RouterCallbackProvider {
    @Override
    public RouterCallback provideRouterCallback() {
        return new SimpleRouterCallback() {
            @Override
            public void beforeOpen(Context context, Uri uri) {
                context.startActivity(new Intent(context, LaunchActivity.class));
            }

            @Override
            public void afterOpen(Context context, Uri uri) {
            }

            @Override
            public void notFound(Context context, Uri uri) {
                context.startActivity(new Intent(context, NotFoundActivity.class));
            }
        };
    }
}
```
在`Application`中实现`RouterCallbackProvider`接口，通过`provideRouterCallback()`方法提供`RouterCallback`，具体API如上。

### 支持Http(s)协议

``` java
@Router({"http://mzule.com/main", "main"})
```

AndroidManifest.xml

``` xml
<activity
    android:name="com.github.mzule.activityrouter.router.RouterActivity"
    android:theme="@android:style/Theme.NoDisplay">
    ...
    <intent-filter>
    	<action android:name="android.intent.action.VIEW" />
    	<category android:name="android.intent.category.DEFAULT" />
    	<category android:name="android.intent.category.BROWSABLE" />
    	<data android:scheme="http" android:host="mzule.com" />
	</intent-filter>
</activity>
```

这样，`http://mzule.com/main`和`mzule://main`都可以映射到同一个Activity，值得注意的是，在`@Router`中声明`http`协议地址时，需要写全称。

### 支持参数transfer

``` java
@Router(value = "item", longExtra = "id", transfer = "id=>itemId")
```
这里通过`transfer = "id=>itemId"`的方式，设定了url中名称为`id`的参数会被改名成`itemId`放到参数`Bundle`中，类型为`long`. 值得注意的是，这里，通过`longExtra = "id"`或者`longExtra = "itemId"`都可以设置参数类型为`long`.

### 支持应用内调用

``` java
Routers.open(context, "mzule://main/0xff878798")
Routers.open(context, Uri.parse("mzule://main/0xff878798"))
```

通过`Routers.open(Context, String)`或者`Routers.open(Context, Uri)`可以直接在应用内打开对应的Activity，不去要经过RouterActivity跳转，效率更高。

## 混淆配置

``` groovy
-keep class com.github.mzule.activityrouter.router.** { *; }
```
