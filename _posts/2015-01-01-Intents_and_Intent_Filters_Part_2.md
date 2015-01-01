---
layout: post
title: Intents and Intent Filters Part 2
date: 2015-01-01 17:00:00 +08:00
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


這篇文章使用例子示範要如何建立 implicit intent、並且在送出前先確認裝置上有能夠接收的 component，接著介紹用來接收 implicit intent 的 intent filter 以及說明 intent filter 所使用的元素。

	
### Example implicit intent

	
Implicit intent 會聲明一個 action，裝置上任何可以執行該 action 的 app 都可以被調用。當你的 app 沒有某項功能、其他的 app 卻有的時候，你可以使用 implicit intent 讓使用者選擇要由哪個 app 代你執行這項功能。


舉個例子，如果你希望 user 透過電子郵件或社群服務等方式跟其他人分享你的 app 的某些內容，建立一個 action 是 <code>ACTION_SEND</code> 的 intent 並把要分享的內容放進 extras，然後再將 intent 傳入 <code>startActivity()</code>，使用者便可以從跳出的選單選擇要使用哪個 app 分享。


要小心的是，有可能使用者的裝置上完全沒有任何 app 可以接收你傳給 <code>startActivity()</code> 的 implicit intent，如果這個情況發生，這個 call 就會失敗並且你的 app 會 crash。為了確認存在能夠接收 intent 的 activity，對你的 <code>Intent</code> 物件呼叫 <code>resolveActivity()</code>，如果回傳的結果不是 null，那就表示至少有一個 app 可以接收這個 intent、呼叫 <code>startActivity()</code> 是安全的，而如果回傳的結果是 null，你就不應該送出這個 intent，並且視情況拿掉與這個 intent 相關的功能並妥善處理。


    // Create the text message with a string
    Intent sendIntent = new Intent();
    sendIntent.setAction(Intent.ACTION_SEND);
    sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
    sendIntent.setType(HTTP.PLAIN_TEXT_TYPE); // "text/plain" MIME type

    // Verify that the intent will resolve to an activity
    if (sendIntent.resolveActivity(getPackageManager()) != null) {
        startActivity(sendIntent);
    }


注意：在這個例子中並沒有使用 URI，不過有聲明 intent 的 data 類型以供系統分辨 extras 攜帶的是什麼樣的資料。


當呼叫 <code>startActivity()</code> 時，系統會查看所有安裝在裝置上的 app，判斷哪些人可以接收這種 intent (action 是 <code>ACTION_SEND</code> 且攜帶 "text/plain" 資料)，如果只有一個 app 可以處理，該 app 就會直接被開啟並且接收 intent，如果有多個 activity 可以接收 intent，系統會顯示一個選單讓使用者決定要開啟哪個 app。


### Forcing an app chooser


如果有不只一個 app 能處理你的 implicit intent，使用者可以選擇要開啟哪個 app，並且將該 app 設為這種 action 的預設選擇。這對使用者來說是蠻貼心的考量，畢竟使用者有可能在同樣的情況下每次都會選擇一樣的 app，比如說選擇開啟網頁的 app 的時候 (使用者通常只習慣用某種網頁瀏覽器)。


不過，如果使用者每次都可能會選擇不同的 app，那你就應該提供選單給使用者。選單每次都會詢問使用者，不會讓使用者勾選預設 app。比如說，當你的 intent 是 <code>ACTION_SEND</code> action、要執行「分享」的功能，使用者每次可能會根據情況選擇不同的 app 分享資訊，所以你就應該要每次都顯示選單，如下圖。


<img border="0" src="/images/20150101_post_intent-chooser.png" />


要顯示選單的話，使用 <code>createChooser()</code> 建立一個 <code>Intent</code> 並將其傳給 <code>startActivity()</code>，如下面程式碼：


    Intent intent = new Intent(Intent.ACTION_SEND);
    // Omitted. Construct the intent here

    // Always use string resources for UI text.
    // This says something like "Share this photo with"
    String title = getResources().getString(R.string.chooser_title);
    // Create intent to show chooser
    Intent chooser = Intent.createChooser(intent, title);

    // Verify the intent will resolve to at least one activity
    if (chooser.resolveActivity(getPackageManager()) != null) {
        startActivity(chooser);
    }


如果有至少一個 app 可以處理這個傳給 <code>createChooser()</code> 的 intent，就會顯示一個選單視窗將 app 列出來，視窗的標題可以由你決定。


## Receiving an Implicit Intent


如果要讓你的 app 能夠接收 implicit intent，在 manifest 檔裡面的 app component 標籤內使用 <code>&lt;intent-filter&gt;</code> 宣告一或多個 intent filter。每個 intent filter 會利用 action、 data 和 category 等資料聲明自己可以接收的 intent 類型。只有在 intent 可以通過某個 filter 時，系統才會將 implicit intent 傳給你的 app component。


注意：一個 explicit intent 永遠會被傳遞給其聲明的目標對象，不論有沒有符合該 component 宣告的 intent filter。


App component 應該要對於自己能做的各種不同工作宣告各自的 filter，比如說，一個在相片瀏覽器 app 內的 activity 可能會有兩種 filter：一個用來瀏覽照片、另外一個用來編輯照片。當 activity 開始的時候，它會檢查傳入的 <code>Intent</code> 並根據其資訊決定要執行什麼功能 (像是要不要顯示相片編輯器)。


Intent filter 在 manifest 檔裡面由 <code>&lt;intent-filter&gt;</code> 定義，是 app component 內的巢狀資料 (比如說放在 <code>&lt;activity&gt;</code> 標籤內)。在 <code>&lt;intent-filter&gt;</code> 標籤內，你可以利用以下三種元素聲明這個 intent-filter 能夠接收的 intent：

*   &lt;action&gt;  
    宣告接受的 intent action 的名字。這個屬性的值不是 class 裡定義的 constant，而是 constant 對應到的 string 的值。
*   &lt;data&gt;  
    宣告接受的 data 類型，使用一個或多個屬性來指定各種 data URI (scheme、 host、 port、 path 等等) 和 MIME 類型。
*   &lt;category&gt;  
    宣告接受的 intent category 的名字。這個屬性的值不是 class 裡定義的 constant，而是 constant 對應到的 string 的值。
    注意：如果要接收 implicit intent，你就必須在 intent filter 裡加上 <code>CATEGORY_DEFAULT</code> 這個 category。<code>startActivity()</code> 和 <code>startActivityForResult()</code> 這兩個方法認定所有 intent 都會宣告 <code>CATEGORY_DEFAULT</code> category，如果你沒有在你的 intent filter 宣告這個 category，你的 activity 就不會收到 implicit intent。


舉個例子，下面的程式碼宣告了一個 activity，擁有一個 intent filter，可以接收 action 是 <code>ACTION_SEND</code>、data 類型是 text 的 intent：


    <activity android:name="ShareActivity">
        <intent-filter>
            <action android:name="android.intent.action.SEND"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="text/plain"/>
        </intent-filter>
    </activity>


一個 filter 可以同時包含多個的 <code>&lt;action&gt;</code>、 <code>&lt;data&gt;</code> 或 <code>&lt;category&gt;</code>，如果你這樣做的話，要確保你的 component 可以處理任何通過 filter 的組合。


當你希望處理多種特定的 intent，但只限於特定排列組合的 action、 data 和 category 時，你就必須建立多個 intent filter。


一個 implicit intent 會跟 intent filter 的元素比對來判斷有沒有符合 filter 的條件，intent 必須通過三種檢查才能被傳遞給 component。不過，一個 component 有可能同時有多個 intent filter，intent 只要通過任一個就可以被傳遞給 component。下一篇文章會詳細說明系統是怎麼認定 intent 符不符合 intent filter。


注意：對所有的 activity 而言，intent filter 都應該要在 manifest 裡宣告，不過 broadcast receiver 的 filter 就可以透過呼叫 <code>registerReceiver()</code> 來動態註冊 filter，你也可以在之後用 <code>unregisterReceiver()</code> 取消註冊 receiver，這樣做可以讓你的 app 只在特定執行期間接收特定的 broadcast。


### Restricting access to components


使用 intent filter 不能安全地避免其他 app 開啟你的 component，雖然 intent filter 可以設立條件、限制特定種類的 implicit intent 才能通過，但其他 app 還是可以用 explicit intent 來開啟你的 component (如果他們想辦法得知了你的 component 的完整名稱)。假如你希望只有你自己的 app 才能夠開啟你的 component，那就將 manifest 裡面的 component 的 <code>exported</code> 屬性設為 <code>"false"</code>。


## Example filters


看些例子會更好理解關於 intent filter 的用法及行為，下面的程式碼是個社群分享 app 的 manifest：


    <activity android:name="MainActivity">
        <!-- This activity is the main entry, should appear in app launcher -->
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>

    <activity android:name="ShareActivity">
        <!-- This activity handles "SEND" actions with text data -->
        <intent-filter>
            <action android:name="android.intent.action.SEND"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="text/plain"/>
        </intent-filter>
        <!-- This activity also handles "SEND" and "SEND_MULTIPLE" with media data -->
        <intent-filter>
            <action android:name="android.intent.action.SEND"/>
            <action android:name="android.intent.action.SEND_MULTIPLE"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <data android:mimeType="application/vnd.google.panorama360+jpg"/>
            <data android:mimeType="image/*"/>
            <data android:mimeType="video/*"/>
        </intent-filter>
    </activity>


第一個 activity，<code>MainActivity</code>，是 app 的入口，當使用者點擊 app 的 icon 後會最先開始的 activity：

*   <code>ACTION_MAIN</code> 這個 action 表示這是 app 的入口並且預期不會收到任何 intent 資料。
*   <code>CATEGORY_LAUNCHER</code> 這個 category 表示這個 activity 的 icon 應該要被放在系統的 app launcher 裡，如果這個 <code>&lt;activity&gt;</code> 元素沒有聲明 <code>icon</code> 的話，系統會使用 <code>&lt;application&gt;</code> 元素裡的 <code>icon</code>。


這兩項必須要放在一起，activity 才能出現在 app launcher 供使用者點擊開啟。


第二個 activity，<code>ShareActivity</code>，提供了分享 text 以及 media 內容的功能。使用者除了從 <code>MainActivity</code> 導到這個 activity 以外，也可以從其他 app 直接進入 <code>ShareActivity</code>，只要建立一個 implicit intent 並且符合 <code>ShareActivity</code> 的兩個 intent filter 的任一者即可。


註：程式碼中的 MIME 類型，<code>application/vnd.google.panorama360+jpg</code>，是表示環景照片的特殊資料類別，你可以使用 Google 的環景照片 API 來處理這類型的資料。


本篇主要翻譯自官方文件 [Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html) 。

