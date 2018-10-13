---
title: "设置action bar、action bar tab 样式及高度"
date: 2014-08-23
description: "设置action bar、action bar tab 样式及高度"
categories: [ "android" ]
tags: ["android"]
aliases: [/android/2014/08/23/action-bar-tab-height/]
---

android 3.0 引入了action bar,supper library支持2.1以上版本。

本文主要说下action bar样式的设置。

首先我们要使用action bar须要使用theme `AppCompat`

应用到全局

```xml
<application android:allowBackup="true"
        android:icon="@drawable/app"
        android:label="@string/app_name"
        android:theme="@style/Theme.AppCompat.Light" >
    ....
</application>
```

应用指定的activity

```xml
<activity android:name=".PagerActivity"
        android:theme="@style/CustomActionBarTheme" >
        android:parentActivityName=".MMActivity" >
        <meta-data android:name="android.support.PARENT_ACTIVITY" android:value="com.mojidong.mnt.MMActivity" />
    ....
</activity>
```

### 自定义action bar

前面说了要使用action
bar必须用`AppCompat`这个theme,所以要修改样式我们就须要继承`AppCompat`，覆盖相应属性。

在styles.xml里加入

```xml
<!-- 继承AppCompat.Light，覆盖actionBarStyle -->
<style name="CustomActionBarTheme" parent="@style/Theme.AppCompat.Light">
    <item name="android:actionBarStyle">@style/BarStyle</item>

    <!-- Support library compatibility -->
    <item name="actionBarStyle">@style/BarStyle</item>
</style>

<!-- 继承AppCompat.ActionBar，覆盖background，titleTextStyle -->
<style name="BarStyle" parent="@style/Widget.AppCompat.ActionBar">
    <item name="android:background">@android:color/black</item>
    <item name="android:titleTextStyle">@style/TitleTextStyle</item>

    <!-- Support library compatibility -->
    <item name="titleTextStyle">@style/TitleTextStyle</item>
</style>

<!-- 继承AppCompat.Title，覆盖textColor -->
<style name="TitleTextStyle" parent="@style/TextAppearance.AppCompat.Widget.ActionBar.Title">
    <item name="android:textColor">#fff</item>
</style>
```

上面例子修改了默认的背景色和标题顔色，更多属性请参见[这里](http://developer.android.com/guide/topics/ui/actionbar.html#Style)。

### 自定义action bar tab

在上例中修改

```xml
<!-- 继承AppCompat.Light，覆盖actionBarStyle -->
<style name="CustomActionBarTheme" parent="@style/Theme.AppCompat.Light">
    <item name="android:actionBarStyle">@style/BarStyle</item>
    <item name="android:actionBarTabStyle">@style/TabStyle</item>
    <item name="android:actionBarTabTextStyle">@style/TabTextStyle</item>
    <item name="android:actionBarSize">50dp</item>

    <!-- Support library compatibility -->
    <item name="actionBarStyle">@style/BarStyle</item>
    <item name="actionBarTabStyle">@style/TabStyle</item>
    <item name="actionBarTabTextStyle">@style/TabTextStyle</item>
    <item name="actionBarSize">50dp</item>
</style>

<!-- 继承AppCompat.ActionBar，覆盖background，titleTextStyle -->
<style name="BarStyle" parent="@style/Widget.AppCompat.ActionBar">
    <item name="android:background">@android:color/black</item>
    <item name="android:titleTextStyle">@style/TitleTextStyle</item>

    <!-- Support library compatibility -->
    <item name="titleTextStyle">@style/TitleTextStyle</item>
</style>

<!-- 继承AppCompat.Title，覆盖textColor -->
<style name="TitleTextStyle" parent="@style/TextAppearance.AppCompat.Widget.ActionBar.Title">
    <item name="android:textColor">#fff</item>
</style>

<!-- 继承TabView，覆盖background -->
<style name="TabStyle" parent="@style/Widget.AppCompat.ActionBar.TabView">
    <item name="android:background">@android:color/black</item>
</style>

<!-- 继承TabText，覆盖textColor -->
<style name="TabTextStyle" parent="@style/Widget.AppCompat.ActionBar.TabText">
    <item name="android:textColor">#fff</item>
</style>
```

> 修改action bar tab高度须要设置`actionBarSize`，然而不幸的是这个属性具然是同时调整action
> bar和action bar tab的高度，没有办法做到分别调整。

### 加点效果
我们还可以给action bar tab 加上点效果
在drawable下创建文件action\_bar\_selector.xml,action\_bar\_tab.xml

action\_bar\_tab\_selector.xml内容如下:

```xml
<?xml version="1.0" encoding="utf-8"?>

<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_focused="false" android:state_selected="false" android:state_pressed="false" android:drawable="@color/tab_color"/>
    <item android:state_focused="false" android:state_selected="true" android:state_pressed="false" android:drawable="@drawable/action_bar_tab.xml"/>

    <item android:state_selected="true" android:state_pressed="true" android:drawable="@drawable/action_bar_tab"/>
</selector>
```

action\_bar\_tab.xml如下:

```xml
<?xml version="1.0" encoding="utf-8"?>

<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:top="-5dp" android:left="-5dp" android:right="-5dp">
        <shape android:shape="rectangle">
            <stroke android:color="#ff4ba587" android:width="5dp"/>
            <gradient android:startColor="@color/tab_color" android:endColor="@color/tab_color"/>
        </shape>
    </item>
</layer-list>
```

在values在创建colors.xml内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="tab_color">#dc222222</color>
</resources>
```

让我们的效果生效，修改之前style

```xml

......

<!-- 继承TabView，覆盖background -->
<style name="TabStyle" parent="@style/Widget.AppCompat.ActionBar.TabView">
    <item name="android:background">@drawable/action_bar_tab_selector</item>
</style>

......

```

### 使用自定义主题


应用到全局

```xml
<application android:allowBackup="true"
        android:icon="@drawable/app"
        android:label="@string/app_name"
        android:theme="@style/CustomActionBarTheme" >
    ....
</application>
```

应用指定的activity

```xml
<activity android:name=".PagerActivity"
        android:parentActivityName=".MMActivity" >
        android:theme="@style/CustomActionBarTheme" >
        <meta-data android:name="android.support.PARENT_ACTIVITY" android:value="com.mojidong.mnt.MMActivity" />
    ....
</activity>
```
