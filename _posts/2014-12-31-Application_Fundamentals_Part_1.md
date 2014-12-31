---
layout: post
title: Application Fundamentals Part 1
date: 2014-12-31 17:00:00 +08:00
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

這篇先簡單介紹了 Android 系統運行應用程式的機制，接著，在閱讀全文後，你會了解 Android 的四個主要 component 的特性及分工、要如何啟動它們、以及 intent 在它們之間所扮演的角色。


## 寫在前面


Android 是以 Java 程式語言撰寫的。Android SDK 工具將程式碼（包括所有的資料或 resources 檔案）編譯後產生 APK (application package file)，也就是 .apk 檔。一個 APK 檔包含了一個 Android app 所有的檔案，Android 系統的行動裝置使用 APK 來安裝 app 。  

一旦安裝在 Android 裝置上以後，每個 Android app 都在各自的 security sandbox 裡運作：


*   Android 作業系統是個多人 (multi-user) 的 Linux 系統，每個 app 都是個不同的 user 。
*   在預設的情況下，系統會分配給每個 app 一個不重複的 Linux user ID；這個 ID 是系統所需要使用的，並不會讓 app 知道值是什麼。系統對一個 app 內的所有檔案設定權限，只有擁有 user ID 的 app (也就是自己) 可以獲取那些檔案。
*   在預設的情況下，每個 app 運作在獨立的 Linux process 裡，並且每個 process 都有自己的 virtual machine (VM)。當 app 有某個 component 需要被執行、且這個 app 目前沒有其他 component 運作在任何一個 process 的話，系統便會開啓一個 process 供這個 app 運作。當 app 的所有 component 都結束或是系統需要為其他 app 恢復更多記憶體時，系統便把 process 結束掉。


所以，Android 系統是遵循 principle of least privilege 運作的：在預設的情況下，每個 app 都只獲取其運作所需的資源。也就是說，app 並不能接觸系統中沒被賦予權限的檔案，因此保證了系統的安全性。


不過，一個 app 還是有辦法可以與其他 app 共享檔案或是獲取系統資源：

*   兩個 app 有辦法可以使用同一個 user ID，也就是說它們可以獲取彼此的檔案。使用相同 user ID 的 app 也可以在同一個 Linux process 運作以及共用 VM，不過 app 必須皆以同樣的憑證簽署。
*   App 可以要求權限以獲取裝置資料，像是通訊錄、SMS 訊息、SD 卡、相機等等，所有要求的權限都必須在安裝的時候獲得使用者的同意。


## App Components


App components 是建立 Android app 所必需的組件。每個 component 都是系統可以進入 app 的管道，不過對使用者而言不是每個 component 都可以用來開啓 app。每個 component 都有各自的特性、任務及生命週期，與其他 component 配合便能完成 app 的各種功能。


以下是 Android app 的四個主要 component：

*   Activity  

    Activity 是螢幕上表現出來的使用者介面。舉例來說，一個 email app 可能會有一個 activity 用來顯示信件匣、一個 activity 用來撰寫新的郵件、一個 activity 用來讀取信件。雖然這些 activity 要全部組在一起才能變成一個完整的 email app，但 activity 都是各自獨立的。如果 email app 允許的話，外部的其他 app 可以任意直接開啓任何一個 activity。舉例來說，一個照相的 app 可以直接開啟撰寫郵件的 activity 以分享照片。  
    在開發的過程中，建立一個 activity 需 <code>extends Activity</code> 並實作所需的內容。

*   Service

    Service 沒有使用者介面，可以在背景進行較長時間的運作或是與其他 process 溝通。舉例來說，一個 service 可以在背景播放音樂、讓使用者在同時使用其他 app，或是在下載檔案的同時避免將 app 的運作卡住。其他的 component (比如說 activity) 可以 start 一個 service 並交付任務讓其執行，或是 bind 住一個 service 與其互動。  
    在開發的過程中，建立一個 service 需 <code>extends Service</code> 並實作所需的內容。

*   Content provider

    Content provider 負責管理被分享的 app 資料，資料可以存在檔案系統、SQLite 資料庫、網路、或任何能被 app 訪問的永久儲存空間。透過 content provider，其他的 app 可以 query 甚至修改資料 (如果 content provider 允許的話)。舉例來說，Android 系統有一個負責管理通訊錄資料的 content provider，所以任何已獲得相對應權限的 app 都可以 query content provider 的部分資料 (像是 <code>ContactsContract.Data</code>) 以讀寫通訊錄的內容。  
    Content provider 也可以用來讀寫 app 內不公開、只供自己使用的資料。  
    在開發的過程中，建立一個 content provider 需 <code>extends ContentProvider</code> 並依照規定實作所需的 API 以供其他 app 進行讀寫資料等行為。

*   Broadcast receiver

    Broadcast receiver 會在接收到對全系統發出的廣播後作出回應。許多的廣播是由系統發出來的，比如說通知大家螢幕關了、電池的電量很低了、或是剛截完一張螢幕截圖。App 也可以自己發出廣播訊息通知大家，比如說廣播自己剛下載了一些資料可以供所有 app 使用。Broadcast receiver 並沒有使用者介面，不過它們可以在接收到廣播後產生在狀態欄的通知訊息、告知使用者有廣播事件。在較多的情況下，broadcast receiver 只負責接收訊息並轉告其他 component，比如說在接收廣播以後開啟一個 service 做後續的工作。  
    在開發的過程中，建立一個 broadcast receiver 需 <code>extends BroadcastReceiver</code> 並實作所需的內容，每則廣播則是以 <code>Intent</code> 傳遞。


Android 系統的特色之一是任何 app 都可以開啟其他 app 的 component。比如說，如果你希望你的 app 有拍照功能，那你可能可以直接使用現有某個 app 而不用自己撰寫開啟相機、拍照、將圖片存檔的 activity。你不必複製或知道該 app 的程式碼，你只需要開啟那個 app 與拍照相關的 activity。當完成的時候，照片的資料甚至可以傳回你自己的 activity，對使用者而言，看起來就像拍照功能也是你的 app 的一部份。


當系統啟動某個 component，如果那個 component 所屬的 app 沒有正在執行的話系統就會開一個新的 process 給該 app，並且 instantiate 那個 component 需要的 class。像上一段提到的開啟其他 app 的拍照 activity，那個 activity 會在原本 app 的 process 執行，而不是你的 app 的 process。所以，不同於大多數其他系統上的應用程式，Android app 並沒有一個單獨的程式開始執行點 (像 <code>main()</code> 之類的)。


不同的 app 在各自獨立的 process 中執行，而且 app 的資料都有權限的限制不讓其他 app 獲取，所以你的 app 無法直接開啟其他 app 的 component，不過系統就可以。因此，要啟動其他 app 的 component，你必須傳遞訊息給系統，並在你的 <code>intent</code> 註明要開啟哪個特定的 component，便可由系統來開啟。


## Activating Components
	

四種 component 中的三種：activity、service 和 broadcast receiver，是由 intent 這個非同步的訊息來啟動的。Intent 會在程式執行的 runtime 將獨立的 component 彼此 bind 住 (不論 component 是不是同屬同一個 app)，你可以把 intent 想像是負責發出由一個 component 送給另一個 component 的關於某個行為的請求。


一個 intent 是由一個 <code>Intent</code> 物件產生的，用來定義一則訊息以啟動某個特定 component 或是某種特定 component；一個 intent 的性質可以是明確的（explicit intent，傳給明確 class 對象）或是不明確的（implicit intent，只要是有相符的 intent-filter 的對象都可以接收）。


對 activity 和 service 而言，一個 intent 定義了某個要執行的行為，比如說開啟某個畫面或傳送某個東西，同時可能會包含所需資料的 URI；舉例來說，一個 intent 可能會傳遞一個顯示圖片或打開網頁的請求給一個 activity。你也可以要求接收 intent 訊息的對象再回傳訊息回來，回傳的訊息一樣是以 intent 的形式傳遞；比如說你的 app 要求開啟通訊錄的 activity，並由使用者挑選一筆資料之後將被挑選的聯絡人資料的 URI 以 intent 回傳。


對 broadcast receiver 而言，intent 只單純傳遞廣播的訊息，比如說一個告知裝置快沒電的廣播只包含一個已定義好、代表電量低的字串。


而 content provider 並不會由 intent 來啟動；Content provider 是在收到來自 <code>ContentResolver</code> 的請求時才會啟動。Content resolver 和 content provider 一起處理所有的資料交流，所以向 provider 要求資料的 component 便不用自己處理、只要呼叫 <code>ContentResolver</code> 物件的方法就好。這樣可以在 content provider 和要求資訊的 component 之間保持一層抽象層以維護系統資料的安全。


有幾種不同的方法可以啟動各種 component：  

*   你可以藉由傳遞 <code>Intent</code> 到 <code>startActivity()</code> 或是到 <code>startActivityForResult()</code> (如果你希望收到回傳的結果的話) 來啟動一個 activity 或是交付任務給一個 activity。
*   你可以藉由傳遞 <code>Intent</code> 到 <code>startService()</code> 來啟動一個 service 或是交付任務給一個正在運行的 service。你也可以藉由傳遞一個 <code>Intent</code> 到 <code>bindService()</code> 來 bind 住一個 service 與其互動。
*   你可以藉由傳遞 <code>Intent</code> 到 <code>sendBroadcast()</code>、 <code>sendOrderedBroadcast()</code>、 <code>sendStickyBroadcast()</code> 等方法來產生一則廣播以啟動 broadcast receiver。
*   你可以藉由呼叫一個 <code>ContentResolver</code> 的 <code>query()</code> 方法來 query 一個 content provider。



本篇主要翻譯自官方文件 [Application Fundamentals](https://developer.android.com/guide/components/fundamentals.html) 。
