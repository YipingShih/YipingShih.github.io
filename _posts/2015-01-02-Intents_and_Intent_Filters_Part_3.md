---
layout: post
title: Intents and Intent Filters Part 3
date: 2015-01-02 02:00:00 +08:00
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


這篇文章先簡單介紹 pending intent，接著說明 intent 要通過 intent filter 必須經過哪些檢查以及詳細的規則，最後提到 intent 的比對可以不只用在尋找合適的執行 component。


## Using a Pending Intent

	
<code>PendingIntent</code> 物件是 <code>Intent</code> 物件的外層包裝，<code>PendingIntent</code> 的主要用途是用來擴增你使用其他 app 的 <code>Intent</code> 的權限、讓你使用時就好像是從自己 app 的 process 執行的一樣。


Pending intent 最常被使用的情況包括以下幾點：

*   宣告一個 intent 用來在使用者透過 app 的 <code>App Widget</code> 進行操作時執行（Android 系統的 <code>NotificationManager</code> 負責執行 <code>Intent</code>）。
*   宣告一個 intent 用來在使用者透過 app 的 <code>Notification</code> 進行操作時執行（Home screen app 負責執行 <code>Intent</code>）。
*   宣告一個 intent 用來在設定好的未來特定時間執行（Android 系統的 <code>AlarmManager</code> 負責執行 <code>Intent</code>）。


每個 <code>Intent</code> 物件都是設計要給特定類型的 component (像是 <code>Activity</code>、 <code>Service</code> 或 <code>BroadcastReceiver</code>) 來接收並處理，<code>PendingIntent</code> 也是如此。當使用一個 pending intent 時，你的 app 並不會使用像平時常用的 <code>startActivity()</code> 之類的呼叫來執行 intent；<code>PendingIntent</code> 會把 intent 封裝起來，其他 component 接收到之後就能執行裡面封裝的 intent。你必須根據 intent 要開啟的 component 類型來使用下列的方法建立 <code>PendingIntent</code>：


*   如果 <code>Intent</code> 會開啟一個 <code>Activity</code>，就呼叫 <code>PendingIntent.getActivity()</code>。
*   如果 <code>Intent</code> 會開啟一個 <code>Service</code>，就呼叫 <code>PendingIntent.getService()</code>。
*   如果 <code>Intent</code> 會開啟一個 <code>BroadcastReceiver</code>，就呼叫 <code>PendingIntent.getBroadcast()</code>。


除非你的 app 需要從其他 app 接收 pending intent，不然你應該只會需要用這些方法來建立 <code>PendingIntent</code>。


每個方法都要傳入當時的 app <code>Context</code>、要封裝的 <code>Intent</code> 以及一至數個聲明 intent 該如何使用的 flag (比如說 intent 能不能被使用不只一次)。


## Intent Resolution


當系統收到一個 implicit intent 時，會根據以下三點來比較 intent 以及裝置上各 app component 的 intent filter，來尋找最適合接收 intent 的 component：


*   Intent action
*   Intent data (包括 URI 和資料類型)
*   Intent category


這篇文章後面的段落會介紹 intent filter 要怎麼在 manifest 檔裡宣告，使得 intent 能被分配到合適的 component。


### Action test


一個 intent filter 可以宣告 0 或多個 <code>&lt;action&gt;</code> 元素來聲明接受的 intent action，如下例：


    <intent-filter>
        <action android:name="android.intent.action.EDIT" />
        <action android:name="android.intent.action.VIEW" />
        ...
    </intent-filter>


<code>Intent</code> 的 action 必須符合 filter 裡的某一個 action 才能通過這個 filter。


如果 filter 沒有列出任何 action，則所有 intent 都無法通過這項檢查。不過，若 <code>Intent</code> 沒有聲明任何 action、且 filter 包含至少一個 action，intent 則可以通過這項檢查。


### Category test


一個 intent filter 可以宣告 0 或多個 <code>&lt;category&gt;</code> 元素來聲明接受的 intent category，如下例：


    <intent-filter>
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        ...
    </intent-filter>


若一個 intent 要通過 category test，則 <code>Intent</code> 內的每個 category 都必須有在 filter 中被聲明。反過來則不一定，intent filter 有可能會宣告比 <code>Intent</code> 內的數量還多的 category 而 <code>Intent</code> 仍能通過 filter。因此，沒有任何 category 的 intent 一定可以通過 category test，不論 filter 的內容是什麼。


備註：Android 會自動為所有傳到 <code>startActivity()</code> 和 <code>startActivityForResult()</code> 的 implicit intent 都加上 <code>CATEGORY_DEFAULT</code> category，因此，如果你希望你的 component 能接收 implicit intent，就必須在 component 的 intent filter 加上 <code>"android.intent.category.DEFAULT"</code>，如前面的 <code>&lt;intent-filter&gt;</code> 程式碼裡添加的一樣。


### Data test


一個 intent filter 可以宣告 0 或多個 <code>&lt;data&gt;</code> 元素來聲明接受的 intent data，如下例：
	

    <intent-filter>
        <data android:mimeType="video/mpeg" android:scheme="http" ... />
        <data android:mimeType="audio/mpeg" android:scheme="http" ... />
        ...
    </intent-filter>


每個 <code>&lt;data&gt;</code> 元素可以聲明一個 URI 結構以及一個資料類型 (MIME media type)。URI 結構的每個部分有分別的屬性，包括 <code>scheme</code>、 <code>host</code>、 <code>port</code> 和 <code>path</code>：


<code>&lt;scheme&gt;://&lt;host&gt;:&lt;port&gt;/&lt;path&gt;</code>


舉個例子：


<code>content://com.example.project:200/folder/subfolder/etc</code>


在這個 URI 中，scheme 是 <code>content</code>，host 是 <code>com.example.project</code>，port 是 <code>200</code>，path 是 <code>folder/subfolder/etc</code>。


這些 <code>&lt;data&gt; 元素內的屬性都不一定要聲明，但是有些屬性間相互依賴的規則：</code>


*   如果沒有聲明 scheme，則忽略 host。
*   如果沒有聲明 host，則忽略 port。
*   如果 scheme 和 host 都沒有聲明，則忽略 path。


當比對 intent 裡的 URI 和 filter 裡聲明的 URI 時，只有 filter 裡提到的 URI 屬性會被拿來比較，舉例說明：


*   如果 filter 只有聲明 scheme，則所有擁有該 scheme 的 URI 都會通過 filter。
*   如果 filter 聲明了 scheme 和 authority 但沒有 path，則所有擁有相同 scheme 和 authority 的 URI 都會通過 filter，不論它們的 path 是什麼。
*   如果一個 filter 聲明了 scheme 和 authority 和 path，則 intent 的 URI 必須要三者都與 filter 相同才能通過。


備註：path 的聲明可以包括一個通配符星號 (*) 使 path 名稱只要部分相同就可以符合。


Data test 會同時比對 intent 和 filter 的 URI 及 MIME 類型，比對的規則如下：


*   同時沒有 URI 和 MIME 類型的 intent 只會在 filter 也沒有聲明任何 URI 和 MIME 類型時才會通過。
*   只有 URI 而沒有 MIME 類型 (既沒有明確列出也不能從 URI 推論出來) 的 intent 只會在 URI 與 filter 的 URI 格式相同且 filter 也沒有聲明 MIME 類型時通過。
*   只有 MIME 類型而沒有 URI 的 intent 只會在 filter 列出了相同的 MIME 類型且沒有聲明 URI 時通過。
*   如果是同時有 URI 及 MIME 類型 (有明確列出或是可以從 URI 推論出來) 的 intent，只會在 MIME 類型符合 filter 列出的某個類型時通過 MIME 部分的測試，在 URI 符合 filter 列出的某個 URI 時、或是 intent 有個 <code>content:</code> URI 或 <code>file:</code> URI 並且 filter 沒有聲明 URI 時通過 URI 部分的測試。換句話說，component 的 filter 如果只有列出 MIME 類型，則在預設情況下是能夠支援 <code>content:</code> 及 <code>file:</code> 資料的。


上述的規則 (d) 反映了 component 能夠從檔案或是 content provider 取得本地端資料的設計，因此它們的 filter 只需要列出資料類型而不用明確地寫出 <code>content:</code> 及 <code>file:</code> scheme，這也屬於很常見的做法。舉個例子，下面程式碼中的 <code>&lt;data&gt;</code> 元素告訴 Android 系統說自己所屬的 component 可以從 content provider 取得 image 資料並且顯示：


    <intent-filter>
        <data android:mimeType="image/*" />
        ...
    </intent-filter>


通常資料都是由 content provider 提供，因此 filter 聲明資料類型而沒有 URI 是較常見的情況。


另一個常見的架構是 filter 裡有一個 scheme 以及一項資料類型。比如說，下面程式碼裡的 <code>&lt;data&gt;</code> 元素告訴 Android 系統說這個 component 可以接收網路的影片資料：


    <intent-filter>
        <data android:scheme="http" android:type="video/*" />
        ...
    </intent-filter>


### Intent matching


將 intent 與 intent filter 做比對不只是為了找尋適合的目標 component 來執行，也可以用來判斷裝置上存在的 component，舉例來說，Home app 會找尋裝置上所有擁有聲明 <code>ACTION_MAIN</code> action 和 <code>CATEGORY_LAUNCHER</code> category 的 intent filter 的 activity 來放在 app 的 launcher，將裝置上的所有 app 列出來。


你的 app 也可以這樣使用 intent 的比對，<code>PackageManager</code> 有一系列 <code>query...()</code> 的方法可以找出所有能讓某個特定 intent 通過的 compontent，以及一個類似的系列 <code>resolve...()</code> 可以判斷最適合某個 intent 的 component。舉些例子，<code>queryIntentActivities()</code> 會回傳所有能處理傳入的 intent 的 activity 列表，類似的 <code>queryIntentServices()</code> 則是回傳 service 列表，以及 <code>queryBroadcastReceivers()</code> 用在 broadcast receiver，這些方法都不會真的去觸發 component。


本篇主要翻譯自官方文件 [Intents and Intent Filters](https://developer.android.com/guide/components/intents-filters.html) 。

