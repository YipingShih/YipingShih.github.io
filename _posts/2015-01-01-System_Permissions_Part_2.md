---
layout: post
title: System Permissions Part 2
date: 2015-01-01 16:15:00 +08:00
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


除了透過要求系統提供的權限以使用系統的特定功能以外，你也可以建立自己的權限，用來控管其他 app 與你的 app component 的互動。這篇文章簡單介紹了各種 component 的自訂權限要如何設定以及相關的機制。
	

## Declaring and Enforcing Permissions


如果你要建立自己的權限來控管別的 app 與你的 app 的互動，你必須要先在你的 <code>AndroidManifest.xml</code> 的 <code>&lt;permission&gt;</code> 標籤宣告這些權限。


舉例來說，如果你希望其他 app 必須具有權限才能開啟你的 app 的 activity，那你可以如下列程式碼這樣宣告供其他人使用的權限：


    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.me.app.myapp" >
        <permission android:name="com.me.app.myapp.permission.DEADLY_ACTIVITY"
            android:label="@string/permlab_deadlyActivity"
            android:description="@string/permdesc_deadlyActivity"
            android:permissionGroup="android.permission-group.COST_MONEY"
            android:protectionLevel="dangerous" />
        ...
    </manifest>


<code>android:name</code> 這個屬性是一定要的，其他人要求這個權限時需要使用這個名字字串。


<code>android:protectionLevel</code> 這個屬性是一定需要的 (沒寫的話預設值是 <code>normal</code>)，用來告訴系統說使用者會怎麼被告知需要的權限、誰可以持有這個權限等，可以設定的值以及各自代表的意義可以看 <a href="https://developer.android.com/reference/android/R.styleable.html#AndroidManifestPermission_protectionLevel" target="_new">AndroidManifestPermission_protectionLevel</a> 這份文件及表格的說明。


<code>android:permissionGroup</code> 不一定要註明，只是用來為權限做簡單的分組，你通常只需要從系統現成的分組中選擇就好 (列表在 <a href="https://developer.android.com/reference/android/Manifest.permission_group.html" target="_new">android.Manifest.permission_group</a> 這份文件)。


權限應該要提供說明用的 label 和 description 字串，在使用者被詢問是否要同意權限時顯示。<code>android:label</code> 是權限在列表中顯示的名字，<code>android:description</code> 是對於權限的說明文字。label 的長度不應太長，只需用簡短幾個字表達權限所用來執行的功能，description 則應該用幾句話說明有了權限以後可以做的事情。官方建議以兩句話撰寫 description，除了用一句說明要用權限做什麼事以外、另一句提醒使用者讓 app 有了這個權限可能會有什麼風險或影響。


下面的例子是系統的 CALL_PHONE 權限的 label 和 description 字串：


    <string name="permlab_callPhone">directly call phone numbers</string>
    <string name="permdesc_callPhone">Allows the application to call
        phone numbers without your intervention. Malicious applications may
        cause unexpected calls on your phone bill. Note that this does not
        allow the application to call emergency numbers.</string>


你可以從裝置的設定 app 以及 shell 指令 <code>adb shell pm list permissions</code> 查看裝置上現在設定的權限。要從設定 app 看的話，打開裝置的「設定」->「應用程式」，點選你要查看的 app，下滑至頁面最下方便可檢視 app 所要求的權限。開發者可如下利用 adb 加上 -s 指令，所顯示的權限及說明文字就跟使用者看到的差不多：


    $ adb shell pm list permissions -s
    All Permissions:

    Network communication: view Wi-Fi state, create Bluetooth connections, full
    Internet access, view network state

    Your location: access extra location provider commands, fine (GPS) location,
    mock location sources for testing, coarse (network-based) location

    Services that cost you money: send SMS messages, directly call phone numbers

    ...


### Enforcing Permissions in AndroidManifest.xml


你可以在你的 <code>AndroidManifest.xml</code> 裡設立高層級的權限來控管其他 app 對你的 app component 的使用。照前面的方法宣告完權限後，只要在 component 的標籤裡加上 <code>android:permission</code> 屬性，標明權限的名字就可以了。


<code>Activity</code> 的權限 (加在 <code>&lt;activity&gt;</code> 標籤裡) 限制誰可以開啟這個 activity。程式會在 <code>Context.startActivity()</code> 以及 <code>Activity.startActivityForResult()</code> 的時候檢查是否具有需要的權限，如果試圖開啟 activity 的人沒有權限，就會收到 <code>SecurityException</code> 的報錯。


<code>Service</code> 的權限 (加在 <code>&lt;service&gt;</code> 標籤裡) 限制誰可以 start 或 bind 這個 service。程式會在 <code>Context.startService()</code>、 <code>Context.stopService()</code> 和 <code>Context.bindService()</code> 的時候檢查是否具有需要的權限，如果試圖操作 service 的人沒有權限，就會收到 <code>SecurityException</code> 的報錯。


<code>BroadcastReceiver</code> 的權限 (加在 <code>&lt;receiver&gt;</code> 標籤裡) 限制誰可以發送廣播到這個 receiver。程式會在 <code>Context.sendBroadcast()</code> return 之後、系統試著要將廣播送給 receiver 時檢查是否具有需要的權限，不過如果傳送廣播的人沒有權限，因為都已經 return 了所以不會收到報錯，就只是不會傳送 intent 到 receiver。由於 <code>BroadcastReceiver</code> 不一定會在 manifest 裡宣告，在程式裡透過 <code>Context.registerReceiver()</code> 向系統註冊的 receiver 一樣可以設定權限來控管誰能發送廣播給它。反過來，也可以在 <code>Context.sendBroadcast()</code> 的時候設定權限，只有有權限的 app 的 <code>BroadcastReceiver</code> 可以收到廣播，下一個段落 (Enforcing Permissions when Sending Broadcasts) 會再說明這部分。


<code>ContentProvider</code> 的權限 (加在 <code>&lt;provider&gt;</code> 標籤裡) 限制誰可以獲取 <code>ContentProvider</code> 的資料。Content provider 另外有個重要的安全機制叫做 URI 權限，後面的段落會再說明。跟其他 component 有個不一樣的地方，content provider 有兩種不同的權限可以設定：<code>android:readPermission</code> 限制誰可以讀取 provider 的資料，<code>android:writePermission</code> 限制誰可以將資料寫入 provider。如果一個 provider 同時設定了讀和寫兩種權限，當你只有寫的權限的時候當然就無法讀取 provider 的資料。程式會在你一開始取得 provider 時檢查權限，如果你讀和寫的權限都沒有，這時就會收到 <code>SecurityException</code> 的報錯。接著在你進行對 provider 的操作時也會檢查權限，<code>ContentResolver.query()</code> 需要讀的權限，<code>ContentResolver.insert()</code>、 <code>ContentResolver.update()</code> 和 <code>ContentResolver.delete()</code> 需要寫的權限，如果在這些情況下沒有權限的話，就會收到 <code>SecurityException</code> 的報錯。


### Enforcing Permissions when Sending Broadcasts


除了像前面說的以權限限制誰可以送 intent 到一個已註冊的 <code>BroadcastReceiver</code> 以外，你也可以設立一個權限用來控管誰能收到廣播。在呼叫 <code>Context.sendBroadcast()</code> 時加上一個權限名稱字串，就可以限制只有擁有該權限的 app 的 receiver 可以收到你的廣播。


收發訊息的兩邊可以同時都要求權限，在這個情況下，必須要兩邊都有對方要求的權限，才能成功傳遞訊息。


### Other Permission Enforcement


對於任何對 service 的呼叫都可以使用 <code>Context.checkCallingPermission()</code> 檢查是否具有權限，在呼叫時傳入權限名稱字串，該方法會 return 一個 int 表示呼叫的 app 是否具有權限。要注意這個檢查應該只在對 service 的呼叫來自不同的 process 時執行 (通常會透過 IDL 介面或是其他跨 process 的方法)。


還有一些實用的方法可以用來檢查權限：如果你有另外一個 process 的 pid，你可以用 <code>Context</code> 的方法 <code>Context.checkPermission(String, int, int)</code> 來透過該 pid 檢查權限；如果你有另外一個 app 的 package name，你可以直接用 <code>PackageManager</code> 的方法 <code>PackageManager.checkPermission(String, String)</code> 來確認該 app 是否具有某個特定權限。


## URI Permissions


目前為止介紹的權限機制對於 content provider 來說還不夠完善：content provider 需要以讀寫的權限來保護自己，同時呼叫 provider 的人也需要將 content provider 的資料 URI 交給其他 app 來協助處理。舉個例子說明，今天如果開發一個 email app，對 email 資料的獲取應該要被權限保護，因為 email 對使用者來說是需要妥善保護處理的資料，但如果要打開某個郵件附件的照片檔案，就需要把照片的 URI 傳給相片瀏覽 app 來開啟，但相片瀏覽 app 並沒有權限可以獲取 email 的資料。


這個情況的解決辦法就是 per-URI 權限：當呼叫 content provider 的人在得到 URI 後開啟一個 activity 或是回傳結果給一個 activity 時，可以設定 <code>Intent.FLAG_GRANT_READ_URI_PERMISSION</code> 及 <code>Intent.FLAG_GRANT_WRITE_URI_PERMISSION</code> 來為收到 URI 的 activity 增加對該 URI 的讀寫權限，不必擔心該 activity 的 app 原本是否具有資料的 provider 要求的權限。


這個機制類似於往常的增加細分權限的做法，讓使用者的某個行為 (打開一個檔案附件、選擇通訊錄資料等) 獲得特定功能的權限，但有個好處是不必真的在 manifest 裡增加 <code>&lt;uses-permission&gt;</code>，讓 app 得以精簡地只需請求那些與主要功能相關的權限。


不過，增加 URI 權限這件事也需要 content provider 的合作，content provider 應該要在 manifest 的 <code>&lt;provider&gt;</code> 標籤內加上 <code>android:grantUriPermissions</code> 屬性和 <code>&lt;grant-uri-permissions&gt;</code> 標籤來宣告並實作這項功能。實作的細節可以參考 <a href="http://thinkandroid.wordpress.com/2012/08/07/granting-content-provider-uri-permissions/" target="_new">Granting Content Provider URI Permissions</a> 這篇文章，或是查詢關於 <code>Context.grantUriPermission()</code>、 <code>Context.revokeUriPermission()</code> 和 <code>Context.checkUriPermission()</code> 等方法的介紹。


本篇主要翻譯自官方文件 [System Permissions](https://developer.android.com/guide/topics/security/permissions.html) 。
