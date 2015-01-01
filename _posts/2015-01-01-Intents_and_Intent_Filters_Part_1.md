---
layout: post
title: Intents and Intent Filters Part 1
date: 2015-01-01 16:50:00 +08:00
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


Intent 負責在 component 之間傳遞訊息，可以概分為 explicit intent 和 implicit intent。這邊文章先簡單介紹 intent ，然後一一介紹 Intent 物件內的屬性，最後示範使用 explicit intent 開啟 component 的例子。


## 寫在前面

	
<code>Intent</code> 是負責傳遞訊息的物件，你可以用來向其他的 app component 傳遞請求。Intent 在不少方面都使得 component 之間的溝通更加方便，最常見的使用情況是以下這三種：


*   開啟一個 activity  
    <code>Activity</code> 是螢幕上表現出來的單一使用者介面。你可以藉由傳遞一個 <code>Intent</code> 到 <code>startActivity()</code> 來開始一個 <code>Activity</code>，<code>Intent</code> 會聲明要開始的 activity 以及包含需要用到的資料。
    如果你希望在 activity 結束的時候，前一個呼叫它的 activity 能夠收到 result 來做後續的事情，那就用 <code>startActivityForResult()</code> 來開始一個 activity，在後面的 activity 結束後，前一個呼叫的 activity 會在自己的 <code>onActivityResult()</code> callback 收到另外一個 <code>Intent</code> 物件。
*   開始一個 service  
    <code>Service</code> 能在背景進行長時間的運作並且沒有使用者介面。你可以藉由傳遞一個 <code>Intent</code> 到 <code>startService()</code> 來 start 一個 service 以進行單次的操作 (比如說下載一個檔案)，<code>Intent</code> 會聲明要開始的 service 以及包含需要用到的資料。
    如果 service 需要跟其他 component 溝通運作的話，你可以藉由傳遞一個 <code>Intent</code> 到 <code>bindService()</code> 來將一個 service 與其他的 component 一起 bind 住。
*   傳遞一則 broadcast  
    Broadcast 是任何 app 都能收到的廣播訊息，系統會傳遞各種廣播來通知系統事件，像是系統啟動或是裝置開始充電等。你也可以發送廣播訊息到其他 app，藉由傳遞一個 <code>Intent</code> 到 <code>sendBroadcast()</code>、 <code>sendOrderedBroadcast()</code> 或 <code>sendStickyBroadcast()</code>。


## Intent Types


Intent 可概分為兩種：


*   Explicit intents  
    Explicit intent 會明確指明要開始的 component 的名字 (包含完整 class)，通常是用在開始自己的 app 內的某個 component (比如說在 user 做了某個動作後開始一個新的 activity，或是開始一個 service 在背景下載檔案)，畢竟你一定會知道自己的 app 內的 activity 或 service 的 class 名稱。
*   Implicit intents  
    Implicit intent 不需要明確指明要開啟的對象名稱，而是聲明一個你想要執行的功能，請求其他具有這項功能的 app 來處理。舉個例子，如果你想要開啟一個地圖並顯示某個地址，你可以用一個 implicit intent 開啟其他有地圖功能的 app 來顯示，而不用自己撰寫這項功能。


當你建立了一個 explicit intent 來開始一個 activity 或 service，系統會直接開啓 <code>Intent</code> 物件所聲明的 app component 對象。


當你建立了一個 implicit intent，Android 系統會將 intent 的內容與裝置上所有 app 的 manifest 內宣告的 intent filter 做比對，如果找到一個相符的 intent filter，系統會開啟該 component 並將 <code>Intent</code> 物件傳給它，如果有多個相符的 intent filter，系統會顯示一個視窗讓使用者挑選要使用哪個 app 的 component。


Intent filter 在 manifest 裡負責註明各個 component 願意收到的 intent 類型。舉個使用的例子，在 activity 內宣告一個 intent filter，表示其他 app 能夠使用這個 intent 來開啟這個 activity；反過來說就是，如果一個 activity 沒有宣告任何 intent filter，就表示只能用 explicit intent (通常在同個 app 內) 來開啟它。


有件要注意的事，為了確保你的 app 的安全性，永遠用 explicit intent 來 start 一個 <code>Service</code> 而不要宣告 intent filter，用 implicit intent 來 start service 會帶來安全上的風險，因為你無法確定、使用者也看不到哪個 service 會因應 intent 而開啟。


## Building an Intent


一個 <code>Intent</code> 物件會包含供 Android 系統判斷要開啟哪個 component 的資訊 (像是確切的 component 名字或是應該收到 intent 的 component 類別)，以及被開啟的 component 在執行接下來的任務所需要的資訊 (像是要做哪件事以及需要用到的資料)。


一個 <code>Intent</code> 主要包含的資訊為下列幾項：


*   Component name  
    要開啟的 component 的名字。
    Intent 裡不一定要有這項資訊，但是一定要有這項才能建立一個 explicit intent，代表 intent 應該被傳遞到這個名字 (包含完整 class 名稱) 的 component。沒有 component 名字的話，intent 就會是 implicit intent 並且系統會根據 intent 包含的其他資訊 (像是 action、 data 或 category 等，下面會詳述) 來決定要開啟哪個 component。如果你要開啟某個特定的 component，你就應該要在 intent 包含完整的 component 名字。
    注意：前幾段有提到過，如果要開啟的 component 是 <code>Service</code>，那你就必須註明 component 名字、使用 explicit intent，不然就會有安全方面的風險。
    <code>Intent</code> 的這項屬性是 <code>ComponentName</code> 物件，也就是目標 component 的完整 class 名稱 (前面包含 app 的 package name，比如說 <code>com.example.ExampleActivity</code>)。你可以使用 <code>setComponent()</code>、 <code>setClass()</code>、 <code>setClassName()</code> 或 <code>Intent</code> 的 constructor 來設 component 的名字。
*   Action  
    一個字串，用來表示要執行的 action (像是 view 或 pick)。
    用在 broadcast intent 的時候，這就代表發生且被廣播通知的事件。Action 很大部分地決定了 intent 其他資訊的結構，尤其是 data 和 extras 的內容。
    你可以建立你自訂的 action，供你 app 內的 intent 使用、或是讓其他 app 可以調用你 app 內的 component ，不過大部分情況下，你還是應該要用 <code>Intent</code> 類別或其他框架類別定義好的 action constant。下面舉幾個用來開啟 activity 的常見 action：
    *   <code>ACTION_VIEW</code>  
        如果你有某些資訊要顯示給使用者看，比如說開啟一個照片瀏覽 app 顯示照片、開啟一個地圖 app 顯示地址，那就在 intent 裡加入這項 action 並且 <code>startActivity()</code>。
    *   <code>ACTION_SEND</code>  
        也就是「分享」的 intent，如果你希望使用者透過其他的 app (比如說 email 或 facebook 之類的社群 app) 對外分享某些東西，那就在 intent 裡加入這項 action 並且 <code>startActivity()</code>。

    你可以在 <a href="https://developer.android.com/reference/android/content/Intent.html" target="_new"><code>Intent</code> 的文件</a>查看更多定義好的 action，不過有些 action 會定義在 Android framework 的其他地方，比如說 <code>Settings</code> 類別裡就有用來開啟設定 app 內的各種設定頁面的 action。
    你可以用 <code>setAction()</code> 或是 <code>Intent</code> 的 constructor 來聲明一個 intent 的 action。
    如果你定義了你自己的 action，要記得在 action 的名字前面加上 app 的 package name，像這樣：  
    <code>static final String ACTION_TIMETRAVEL = "com.example.action.TIMETRAVEL";</code>
*   Data  
    要傳送給 intent 使用的資料的 URI (<code>Uri</code> 物件) 以及資料的 MIME 類型。可以傳送的資料類型要視 intent 的 action 而定，舉例來說，如果 action 是 <code>ACTION_EDIT</code>，那 data 就該包含可以編輯的文件的 URI。
    當建立一個 intent 的時候，除了 data 的 URI 以外，通常也都需要聲明 data 的 MIME 類型。一個 activity 不一定能處理所有類型的資料 (有可能在拿到資料後只打算作為圖片顯示、而沒有播放音樂的功能)，但不同資料的 URI 有可能長得一樣，所以聲明資料的 MIME 類型可以協助 Android 系統找到正確的 component 來接收 intent。不過 MIME 類型有時候可以從 URI 判斷，尤其當 data 是 「content: URI」的時候，表示 data 位於裝置內並且是由 ContentProvider 所管理，因此系統便會得知資料的 MIME 類型。
    如果要為 intent 設 data URI，就呼叫 <code>setData()</code>，如果只要設 MIME 類型，就呼叫 <code>setType()</code>，如果有需要的話你也可以直接用 <code>setDataAndType()</code> 來設定兩者。
    注意：如果你要同時設定 URI 和 MIME 類型，不要分別呼叫 <code>setData()</code> 和 <code>setType()</code>，因為這兩個方法有可能會把對方的值變成 null，記得永遠使用 <code>setDataAndType()</code> 來同時設定 URI 和 MIME 類型。
*   Category  
    一個字串，告知該使用什麼樣的 component 來處理 intent。一個 intent 可以有很多的 category 敘述，但其實大部分情況下的 intent 都用不到 category。下面舉幾個常見的 category：
    *   <code>CATEGORY_BROWSABLE</code>  
        要開啟的 activity 可以開啟網頁瀏覽器來顯示收到的 link 的 data，比如說圖片或電子郵件。
    *   <code>CATEGORY_LAUNCHER</code>  
        要開啟的 activity 是一個 task 的初始 activity 並且是系統的應用程式 launcher 之一。
        在 <a href="https://developer.android.com/reference/android/content/Intent.html" target="_new">Intent 的文件</a>可以找到 category 的列表。</p>
        要為 intent 加上 category 敘述的話，使用 <code>addCategory()</code>。


上面列舉的屬性 (component name、action、data 和 category) 都是用來定義 intent 的功能及特性，藉由參考這些資訊，Android 系統就可以判斷該開啟什麼 app component。


不過，除此之外，一個 intent 還可以包含額外的、與該開啟什麼 component 無關的資訊。一個 intent 可以包含以下幾樣東西：


*   Extras  
    為了讓 component 完成接下來的任務所需的資訊，為鍵值對 (Key-value pairs) 的形式。有的 action 會需要接收特定形式的 URI，有的 action 則會需要用到特定的 extra 資料。
    你可以透過傳入各種形式的資料給 <code>putExtra()</code> 來加入 extra data，每種形式都一樣是兩個參數：key的名字以及對應的值。你也可以建立一個 <code>Bundle</code> 物件來包住所有 extra data，然後再用 <code>putExtras()</code> 將 <code>Bundle</code> 放進 <code>Intent</code>。
    舉個例子，當要建立一個 ACTION_SEND 的 intent 用來發送電子郵件，你可以將收信對象的地址與 <code>EXTRA_EMAIL</code> key 相對、將信件主旨與 <code>EXTRA_SUBJECT</code> key 相對，將資料傳遞給接收 intent 的 activity。
    <code>Intent</code> 類別裡聲明了 <code>EXTRA_*</code> 開頭的 constant 作為系統的標準 data 類別，如果你要宣告你自己的 extra key (給自己 app 內的 intent 使用)，為了避免與系統的重複，建議使用 app 的 package name 作為 key 名稱的開頭，宣告方式像是這樣：  
    <code>static final String EXTRA_GIGAWATTS = "com.example.EXTRA_GIGAWATTS";</code>
*   Flags  
    定義在 <code>Intent</code> 類別內，用來作為 intent 的 metadata。Flag 會指示 Android 系統要怎麼開啟一個 activity (比如說 activity 該被分配到哪個 task) 以及開啟後如何處理 activity (比如說該不該將其列入最近使用的 activity 列表)。不同的 flag 的功能可以查看 <a href=https://developer.android.com/reference/android/content/Intent.html#setFlags(int)"" target="_new"><code>setFlags()</code></a> 的文件說明。
### Example explicit intent


你可以使用 explicit intent 來開啟特定的 app component，像是你的 app 內的特定 activity 或 service。要建立一個 explicit intent，<code>Intent</code> 物件唯一需要的屬性是 component name，其他屬性都不一定要註明 (視你的需要而定)。


舉個例子，如果你的 app 內有個叫做 <code>DownloadService</code> 的 service，要用來下載某個網址的檔案，你可以像下面的程式碼這樣 start 你的 service：


    // 在一個 Activity 內執行，所以 'this' 是 Context
    // fileUrl 是一個 string URL，比如說 "http://www.example.com/image.png"
    Intent downloadIntent = new Intent(this, DownloadService.class);
    downloadIntent.setData(Uri.parse(fileUrl));
    startService(downloadIntent);


上面程式碼的 <code>Intent(Context, Class)</code> constructor 提供了 app 的 <code>Context</code> 以及要開啟的 component 的 class，這樣一來，這個 explicit intent 就能開啟 app 內的 <code>DownloadService</code> class。


本篇主要翻譯自官方文件 [Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html) 。

