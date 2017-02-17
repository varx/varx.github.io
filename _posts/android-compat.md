---
title: Android 兼容性处理
date: 2017-02-17 09:16:37
tags:
---


### 样式

`Activity`的 theme 继承自 `Theme.AppCompat`，并及时更新AppCompat 的版本。

AppCompat 对不同版本的Android 系统进行的重载，可以确保在不同设备上都使用最适合的样式。

{% asset_img  WX20170217-092326@2x.png %}

### API
在Gradle 里对API level 有三个值，分别是`compileSdkVersion`,`minSdkVersion`和`targetSdkVersion`. 其中`minSdkVersion`和`targetSdkVersion`会在打包时写入`AndroidManifest.xml`.	历史上(eclipse 和ADT的时代)曾经存在`maxSdkVersion`,现在已经弃用了.

`minSdkVersion` 表示能兼容的最低Android版本,低于这个版本的系统会拒绝安装.

`compileSdkVersion` 意味着编译时 android.jar 的版本,所有api 提示都会按照这个版本来.

`targetSdkVersion` 表示经过兼容测试的版本.

Android Studio(下面简称AS) 会检查 `minSdkVersion`和`compileSdkVersion`之间的兼容性,如果存在兼容性问题, 比如调用了低版本不存在的API,会提示添加版本判断条件.

``` java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP){
	//
}
```

值得注意的是,如果没有经过完善的测试,不要随意变动`targetSdkVersion`. Android系统的向后兼容机制识别的是`targetSdkVersion`, 比如众所周知 API23(6.0) 增加了运行时权限检查, 那么不支持运行时权限的应用在 6.0 更新以后岂不是都要挂? 然而并没有.因为在适配6.0前的应用`targetSdkVersion`只要<23,即使运行在6.0系统上,也是按照`targetSdkVersion`版本的行为运行的.即运行时权限不会作用在这些应用上.

总结:

`compileSdkVersion`越新越好,即使你的应用还不能兼容那么高也没关系.带来的好处是可以知道新版本API的变动,比如删除/废弃的方法,新添加的推荐用法,AS也会帮你检查和解决兼容性问题.

当解决和测试完之后(一般要跨几个版本),再修改`targetSdkVersion`,这时在新版系统上运行就会按照新版的特性运行.至此完成一次新版系统的适配.


