---
title: android内存泄漏笔记
date: 2016-09-01 09:58:52
tags: android
---

### 第三方库

``` bash
https://github.com/square/leakcanary
```

### gradle 集成

``` bash
dependencies {
  debugCompile 'com.squareup.leakcanary:leakcanary-android:1.4-beta2'
  releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.4-beta2'
  testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.4-beta2'
}
```

### In your Application class:
``` bash
public class ExampleApplication extends Application {

  @Override public void onCreate() {
    super.onCreate();
    LeakCanary.install(this);
  }
}
```

### 说明
下次编译安装好了apk后，会多出一个leak的应用，且每有一个项目集成了该库，就会有一个一样的leak应用出现在launch中，打开可以看到项目中，泄漏的部分。

### 实战

#### Activity finish后，但是对他的引用还未结束
最常见的是多线程的应用，简单来说，就是当在activity的生命周期内，发送了一个异步请求，当activity结束，该异步请求还未结束返回，但是对activity的context还是有引用，导致activity无法正常被回收。

解决方法：一般如Handle采用弱引用，或者在activity的onStop中，结束异步请求。

合理分配activity的context，即时释放对该context的引用。

#### Context leak in android.net.ConnectivityManager
解决方法：

由

``` bash
ConnectivityManager cm = (ConnectivityManager)context.getSystemService(Context.CONNECTIVITY_SERVICE);
```

改成

``` bash
ConnectivityManager cm = (ConnectivityManager)context.getApplicationContext().getSystemService(Context.CONNECTIVITY_SERVICE);
```

原因我猜测和上面的context的引用一样的情况。

#### Realm的不及时关闭

解决方法：

在BaseActivity中定义
``` bash
private Realm realm;

@Override
protected void onDestroy() {
    super.onDestroy();
    if (realm != null) {
        realm.close();
    }
}

public Realm getRealm() {
    if (realm == null) {
        realm = Realm.getDefaultInstance();
    }
    return realm;
}
```

在BaseFragment中定义

``` bash
private Realm realm;
@Override
public void onPause() {
    super.onPause();
    if (realm != null){
        realm.close();
    }
}

@Override
public void onResume() {
    super.onResume();
    if (realm == null){
        realm = Realm.getDefaultInstance();
    }
}

public Realm getRealm() {
    if (realm == null){
        realm = Realm.getDefaultInstance();
    }
    return realm;
}
```

未完待续。。。
