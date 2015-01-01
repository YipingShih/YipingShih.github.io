---
layout: post
title: System Permissions Part 1
date: 2015-01-01 16:00:00 +08:00
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


Android 有一套權限許可的機制，用來控管 app 是否能夠進行特定的操作或是獲取特定的資料。如果你的 app 所使用的功能需要相對應的權限，就必須在 manifest 裡頭宣告要求的權限，系統會先確認 app 的簽署並取得使用者對於所有要求權限的許可後再安裝 app。這篇文章簡單解釋權限的機制、說明要如何在 manifest 裡加上需要的權限，並提醒你如果 <code>targetSdkVersion</code> 版本不夠高，Android 為了保險起見可能會自動為你的 app 加上你並不需要的權限要求。


## 寫在前面


Android 是個權限分散的作業系統，每個 app 在系統中都有一個獨立的系統身份 (Linux user ID 和 group Id)，就連系統的某些部分也一樣會區分成獨立的身份，因此每個 app 和其他 app 以及系統之間都是分隔開的。


Android 提供了一套「權限許可」的機制來控管 app 對於額外細分的 security feature 的使用，像是某個 process 能否執行某項操作、是否同意 app 對於特定 URI 資料的獲取等。


這篇文章說明應用程式開發者可以如何使用 Android 提供的 security feature，更多細節可以參考 Android Open Source Project 裡的 <a href="http://source.android.com/devices/tech/security/" target="_new">Android Security Overview</a>。


## Security Architecture
	
在預設的情況下，沒有 app 有權限可以執行任何可能危害其他應用程式或作業系統或使用者的操作，這是 Android 系統的安全機制設計的核心概念之一。包括像是讀寫使用者的私人資料 (例如通訊錄或電子郵件)、讀寫其他 app 的檔案、連結網路、保持裝置螢幕開啓等，都是這類需要考慮安全性的操作。


因為每個 Android 應用程式都在各自的 process sandbox 裡執行，app 間分享 resource 和資料的過程必須都是很明確的：當 app 需要 sandbox 沒有提供的額外功能時，必須宣告相對應的權限。App 靜態宣告需要的權限，Android 系統會在使用者安裝程式時先取得使用者對權限的同意再安裝。Android 不提供動態增加權限 (也就是在執行程式時才增加權限) 的機制，以確保安全及避免影響使用者體驗。


程式的 sandbox 的運作不會受到建立程式所使用的技術影響：Dalvik VM 並不能用來當作區分安全性的邊界，任何 app 都可以執行不在 VM 內運作的 native code (native code 指的是 C 和 C++ 等語言所撰寫的程式碼，透過 Android NDK 在 app 中使用)。所有類型的 app (Java、native 或混合的) 都被分配相同的安全等級以及使用相同的 sandbox 機制。


## Application Signing
	

所有 APK (.apk 檔) 都必須要使用憑證簽署，由開發者持有憑證的私密金鑰。持有憑證的身份不需要是公司或機關，任何 Android app 都可以使用個人的憑證簽署，憑證的目的只是為了區別 app 的開發者，讓系統用來判斷要同意或拒絕一個 app 對另外一個 app 要求的簽署層級的權限、以及是否要同意一個 app 要求和另一個 app 使用相同 Linux 身份的請求。


## User IDs and File Access


在安裝程式的時候，Android 會給程式的 package 一個獨特的 Linux user ID (註：這裡的 package 指的是 app 本身的 package name，而不是程式裡的多個 java package)，在程式被解安裝以前，這個 ID 都是不會變的。同一個 package name 的 app 在不同裝置上的 ID 不一定相同，獨特的 user ID 的作用只是為了在同一個裝置上區分每個 package。


安全性的檢查是屬於 process 層級的，預設情況下，由於不同 package 的程式的 Linux user 身份不同，所以不能在同一個 process 裡執行。你可以藉由 <code>AndroidManifest.xml</code> 裡的 <code>manifest</code> 標籤的 <code>sharedUserId</code> 屬性來讓不同的 app 分配到同樣的 user ID，這樣做的話，兩個 package 就會被當作同樣的 app、使用相同的 user ID 及共享檔案權限。由於安全的考量，所以只有簽署相同並且設定相同 <code>sharedUserId</code> 的 app 才會分配到相同的 user ID。


一個 app 所儲存的任何資料都會被標記 app 的 user ID，在一般情況下不能被其他 app 獲取。當使用 <code>getSharedPreferences(String, int)</code>、 <code>openFileOutput(String, int)</code> 或 <code>openOrCreateDatabase(String, int, SQLiteDatabase.CursorFactory)</code> 等方法建立一個新的檔案，你可以使用 <code>MODE_WORLD_READABLE</code> 和 <code>MODE_WORLD_WRITEABLE</code> 等 flag 來允許 package name 和你不同的其他 app 讀寫這個檔案。當設定了這些 flag 後，檔案一樣是由你的 app 所擁有，不過已經能讓任何其他 app 使用。


## Using Permissions

一個基本的 Android app 在預設情況下不必多要求任何權限，也就是說 app 並不會做任何可能給使用者體驗或裝置資料帶來不好影響的行為。如果要使用裝置上被保護的功能 (像是連接網路、獲取手機通訊錄、讀寫裝置資料等)，你必須在 <code>AndroidManifest.xml</code> 裡加上 <code>&lt;uses-permission&gt;</code> 標籤來宣告你的 app 所需要的權限。


舉個例子，如果一個 app 需要監控裝置收到的 SMS 訊息，就必須像下面的程式碼這樣在 manifest 裡頭加上相對應的權限：
	

    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
        package="com.android.app.myapp" >
        <uses-permission android:name="android.permission.RECEIVE_SMS" />
        ...
    </manifest>


在 app 安裝的時候，系統的 package installer 會在確認過 app 的簽署和使用者的同意後，為 app 增加它所要求的權限。app 不可能會在程式執行的過程向使用者要求增加權限，所以如果一個 app 要使用某個需要增加權限的功能，一定得在安裝的時候增加 app 權限並得以執行該功能，如果沒要求增加權限，執行該功能的時候就會出錯、不會事先提醒使用者。


如果 app 沒獲得需要的權限就執行某個動作的話，通常會產生 <code>SecurityException</code> 並向 app 報錯，不過並不是所有情況都一定會這樣。舉例來說，<code>sendBroadcast(Intent)</code> 是在資料送到接收端的時候檢查權限，而那時發送端的 method 都已經 return 了，所以發送端的 app 就不會收到關於權限的報錯。不過大部份的時候系統的 log 都是會印出缺少權限的報錯讓開發者知道的。


話又說回來，在絕大部分的使用情況下，使用者都是從 Google Play 商店下載安裝 app，使用者一定得在安裝前就同意 app 要求的所有權限，所以你如果 manifest 裡頭沒漏掉需要的權限宣告，就不太需要擔心因為缺少權限而在程式執行的過程中出錯。


所有 Android 系統所提供的權限都可以在 <a href="https://developer.android.com/reference/android/Manifest.permission.html" target="_new"><code>Manifest.permission</code></a> 裡找到，不過 manifest 裡能宣告的權限並不只這些，因為每個 app 也都可以自行定義並提供與其互動所需的權限。


程式執行期間有可能會在下列情況確認 app 是否有得到需要的權限：


*   執行一個系統功能的時候
*   開始一個 activity 的時候，根據權限確認是否可以開啟其他 app 的 activity
*   發出或接收 broadcast 的時候，根據權限確認誰可以收到你的 broadcast 或是誰可以發送 broadcast 給你
*   使用及操作 content provider 的時候
*   Bind 或 start 一個 service 的時候

要注意的是，新版本的 platform 可能會增加權限的限制，原本不需要權限的 API 有可能會在新版本時改為需要有權限才能執行。因為舊版本的 app 不需要要求權限，在新版本的裝置上執行時便可能出錯，因此 Android 會自動根據 <code>targetSdkVersion</code> 來判斷，如果有的權限是在更新的版本被建立的，為了保險起見，那些權限就都會自動被加在這個 app 所要求的權限清單裡。


舉個例子，<code>WRITE_EXTERNAL_STORAGE</code> 這個權限是在 API level 4 以後才加入、用來限制讀寫裝置內儲存的資料，如果你的 <code>targetSdkVersion</code> 的值是 3 以下，表示你只保證你的 app 在 3 以下的裝置執行不會出錯，那你的 app 就會在 4 以上的系統版本的裝置上自動增加這項權限的要求，即使你的 app 根本不需要這項功能。如果這種情況發生了，那使用者從 Google Play 下載你的 app 時，Google Play 列出的權限要求列表便會多出你並沒有在 manifest 裡宣告的權限。


為了避免突然被加上更多權限要求的情況，建議把 <code>targetSdkVersion</code> 設定得越高越好，像上面 <code>WRITE_EXTERNAL_STORAGE</code> 的例子，只要把 <code>targetSdkVersion</code> 改成 4 或以上，表示你保證版本為 4 或以上的裝置是不會出錯的，所以沒宣告 <code>WRITE_EXTERNAL_STORAGE</code> 權限就表示你不會用到，就不會冒出這項多餘的權限要求。你可以從 <a href="https://developer.android.com/reference/android/os/Build.VERSION_CODES.html" target="_new">Build.VERSION_CODES</a> 這份文件查看每個權限分別是在哪個版本加入的。



## Related Posts
*   <a href="http://java.dzone.com/articles/depth-android-package-manager" target="_new">In Depth: Android Package Manager and Package Installer</a>


本篇主要翻譯自官方文件 [System Permissions](https://developer.android.com/guide/topics/security/permissions.html) 。

