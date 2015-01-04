---
layout: post
title: Application Fundamentals Part 2
date: 2015-01-01 01:20:00 +08:00
description: ""
headline: ""
categories: android
tags: 
  - Android
  - Official API Guides
comments: true
mathjax: null
featured: false
share: true
published: true
---


這篇文章介紹了 AndroidManifest.xml，以及要如何在裡頭宣告 app 的 component、加入 intent-filter、宣告 app 需要的特殊功能。最後提到把各種規格的 resource 檔案放入以相對應的 qualifier 命名的資料夾，將使 Android 系統協助你在不同規格的裝置上最佳化你的 app 呈現。

	
## The Manifest File


在 Android 系統開始 app 的某個 component 以前，必須先透過 app 的 <code>AndroidManifest.xml</code> 檔案得知這個 component 的存在。<code>AndroidManifest.xml</code> 的檔案位置位於 app 專案的最底層，必須在這個檔案裡宣告 app 的 component。


除了宣告 component 以外，manifest 檔案還有不少其它功能，例如：


*   標明這個 app 所需要的所有使用者權限，像是連接網路或是獲取通訊錄檔案等。
*   根據這個 app 所使用的 API，標明對 API Level 的最低版本需求。
*   標明這個 app 所需要的軟體及硬體功能，比如說相機、藍牙、多點觸控螢幕。
*   標明除了 Android framework API 以外，這個 app 所需要使用的 API library，比如說 Google Maps library。


還有更多其它功能，就先不在此詳列。
	
## Declaring Components
	
首先的任務就是要告知 Android 系統這個 app 使用的 component。


以下面的程式碼為例，manifest 檔可以這樣宣告一個 activity (<code>...</code>為部分省略程式碼)：
	

    <?xml version="1.0" encoding="utf-8"?>
    <manifest ... >
        <application android:icon="@drawable/app_icon.png" ... >
            <activity android:name="com.example.project.ExampleActivity"
                    android:label="@string/example_label" ... >
            </activity>
            ...
        </application>
    </manifest>


在 <code>&lt;application&gt;</code> 元素裡，<code>android:icon</code> 屬性指向代表這個 app 的 icon 圖案的 resource。


在 <code>&lt;activity&gt;</code> 元素裡，<code>android:name</code> 屬性表示這個 activity (<code>Activity</code> 的subclass) 的完整 class 名稱，而 <code>android:label</code> 屬性表示使用者可看見的、代表這個 activity 的字串。


你必須用下列這些形式宣告 app 的 component：


*   <code>&lt;activity&gt;</code> 表示 activity
*   <code>&lt;service&gt;</code> 表示 service
*   <code>&lt;receiver&gt;</code> 表示 broadcast receiver
*   <code>&lt;provider&gt;</code> 表示 content provider

若你在程式碼中撰寫了 activity、service 或 content provider，卻沒有在 manifest 裡宣告，則系統不會曉得這些 component 的存在、程式也無法執行它們。不過，broadcast receiver 可以選擇在 manifest 宣告、或是在程式裡動態產生 (也就是 <code>BroadcastReceiver</code> 物件) 並呼叫 <code>registerReceiver()</code> 向系統註冊。


## Declaring Component Capabilities
	

在上一篇 <a href="/android/Application_Fundamentals_Part_1" target="_new">Application Fundamentals Part 1</a> 裡提到過，我們可以使用 <code>Intent</code> 來啟動 activity、service 及 broadcast receiver。除了可以用 explicit intent 明確指定 intent 接收對象 (標明 component 的 class 名稱) 來做到這件事之外，implicit intent 的使用範圍和用途相對更廣一些。一個 implicit intent 所指定的是想要達成的 action (同時你也可以選擇是否要將 action 所需要的資料放進 intent)，系統會在裝置上搜尋，找到能夠執行這個 action 的 component 並將其啟動，如果有不只一個 component 可以執行，則使用者可以從中選擇一個啟動。


那系統要怎麼知道有哪些 component 符合某個 intent 的需求呢？就是將收到的 intent 與裝置上的 app 在 manifest 檔案裡列出的 intent filter 做比較。


當你在你的 app 的 manifest 裡宣告一個 activity，你可以選擇加上一個 intent filter 將這個 activity 的功能列出來，這樣你的 activity 就可以在其他 app 發出對於特定功能的請求時啟動並作出回應。只要在你的 component 宣告的元素裡加上一個 <code>intent-filter</code> 元素便可完成 intent-filter 的宣告。


舉例來說，如果你建立了一個 email app 並且有一個撰寫新郵件的 activity，你可以宣告一個能回應寄信功能請求的 intent 的 intent-filter，如下列程式碼：
	
    <manifest ... >
        ...
        <application ... >
            <activity android:name="com.example.project.ComposeEmailActivity">
                <intent-filter>
                    <action android:name="android.intent.action.SEND" />
                    <data android:type="*/*" />
                    <category android:name="android.intent.category.DEFAULT" />
                </intent-filter>
            </activity>
        </application>
    </manifest>

這樣一來，如果有其他 app 產生了一個對外要求 <code>ACTION_SEND</code> 功能的 intent 並將其傳到 <code>startActivity()</code>，則系統就可以啟動你的 activity，供其他 app 撰寫新郵件。你的 activity 仍會在你的 app 的 process 執行，這點在上一篇也有稍作說明。


## Declaring App Requirements
	
運行 Android 系統的行動裝置有非常多種，並不是每台裝置擁有的軟體硬體功能都相同。為了避免其他使用者在安裝後才發現他們的行動裝置無法執行某些你的 app 需要的功能的窘境，強烈建議你在 manifest 將特殊的軟體硬體功能需求列出來。通常這些需求都只是提供告知的功能、系統本身不會檢查，但額外的服務商像是 Google Play 就會讀取這些資料，並在使用者瀏覽商店時將他們的裝置無法執行的 app 過濾掉。


舉例來說，如果你的 app 需要使用手機的拍照功能，以及使用 Android 2.1 (API Level 7) 的 API，你應該將這些需求宣告在 manifest 裡如下：


    <manifest ... >
        <uses-feature android:name="android.hardware.camera.any"
            android:required="true" />
        <uses-sdk android:minSdkVersion="7" android:targetSdkVersion="19" />
        ...
    </manifest>


這樣一來，沒有拍照功能或是 Android 版本小於 2.1 的裝置，便無法透過 Google Play 下載這個 app。


不過，你也可以列出你的 app 需要拍照功能，但不強求一定要有這個功能才能安裝。只要將 <code>android:required</code> 屬性設為 <code>false</code>，使用者就一樣可以下載安裝。但你應該要在程式執行時再次確認運行的手機有無這個功能，對不具有功能的手機要避開使用相機的情況並妥善處理。


## App Resources
	
一個 Android app 不只是由程式碼所組成的，還需要與程式碼分隔的 resource，像是圖片、聲音檔、以及任何與 app 的視覺呈現相關的資料。比如說，你需要以 XML 檔定義 app 需要的動畫、選單、樣式、顏色、activity 的佈局等。使用 app resource 讓你能不添加程式碼便能更改 app 的許多部分，並且，藉由提供不同規格的 resource 檔案，讓你可以在不同規格的裝置上 (像是不同的語系或是不同的螢幕大小呈現) 最佳化你的 app。


對於你加入到 Android 專案中的每一個 resource，SDK 工具都會給予一個 unique integer ID，讓你可以在程式碼或是其他 XML 檔裡頭用來指向該 resource。舉例來說，如果你的 app 包含一個圖片檔案 <code>logo.png</code>、儲存在 <code>res/drawable</code> 檔案路徑，則 SDK 工具會產生一組 ID 為 <code>R.drawable.logo</code>，讓你在建立使用者介面檔案時可以插入這個 ID 指向該圖片。


提供程式碼以外的 resource 最大的好處之一就是讓你的 app 能彈性地為不同規格的裝置提供適合的 resource。比如說，在 XML 檔裡定義好 UI 字串以後，你可以將 app 裡的字串翻譯成不同國家的語言並分別存放在不同的資料夾，接著，根據你的資料夾名稱中的語言 qualifier (例如 <code>res/values-zh-rTW/</code> 就代表台灣繁體中文的字串) 以及使用者的語言設定，Android 系統可以自動套用正確的語言字串到你的 UI。


為了彈性地為不同規格的裝置套用適合的 resource，Android 為 resource 提供了各種不同的 qualifier。Qualifier 是個簡短的字串，存放各種規格 resource 的資料夾以不同的 qualifier 命名，qualifier 字串會定義好代表的規格，因此就能供系統區分以及使用。



本篇主要翻譯自官方文件 [Application Fundamentals](https://developer.android.com/guide/components/fundamentals.html) 。

