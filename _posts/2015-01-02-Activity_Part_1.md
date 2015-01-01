---
layout: post
title: Activity Part 1
date: 2015-01-02 02:15:00 +08:00
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


Activity 是幾乎每個 app 都不可或缺的重要 component，讓使用者可以透過螢幕上的介面進行特定的操作。這篇文章簡單介紹 activity 的基本概念與功能、建立使用者介面的方法、在 manifest 內的宣告，以及開始和結束 activity 的方法。


## 寫在前面

	
Activity 是應用程式的四大 component 之一，讓使用者可以透過螢幕上的介面進行某些特定的操作，像是撥打電話、照相、發送電子郵件或是查看地圖等。每個 activity 都會被分配一個視窗用來顯示使用者介面，通常視窗會占滿螢幕，不過也可以小於螢幕尺寸並顯示於其他視窗之上。


一個應用程式通常由多個彼此之間不太緊密相關的 activity 所組成。通常來說，一個應用程式會擁有一個被聲明為 "main activity" 的 activity，在使用者一打開應用程式時顯示。每個 activity 可以接著 start 其他的 activity 用來進行其他不同的操作。每當有一個新的 activity 開啟時，前一個 activity 就會停止，但系統會將之前的 activity 保存在 back stack 裡面。當一個新的 activity start 時，會被 push 進 back stack 並且取得使用者的關注。Back stack 遵守 stack 的基本原則 "last in, first out"，所以當使用者結束目前 activity 的操作並且按下手機的返回鍵時，activity 就會從 stack pop 並且 destroy、前一個 activity 就會 resume。


當一個 activity 因為新的 activity start 而被 stop 時，會透過 activity 的 lifecycle callback 方法被通知。Activity 在狀態改變時 (被系統 create、 stop、 resume、 destroy 等等) 會分別觸發不同的 callback 方法，你可以在每個方法裡根據你的需求來執行合適的動作。舉個例子，當你的 activity stop 的時候，你應該要釋放大型的物件，像是網路或是資料庫的連接等；而當 activity resume 時，你可以再重新送出請求並繼續原本中斷的工作。這些狀態的改變即構成了 activity 的 lifecycle。


這系列的文章介紹了如何建立及使用 activity 的基礎，包括 activity lifecycle 如何運作，讓你在 activity 狀態改變時能妥善地控制及管理。


## Creating an Activity

	
要建立一個 activity，你必須先建立一個 <code>Activity</code> 的 subclass、或是使用一個現有的 subclass。在你的 subclass 裡，你需要實作系統在 activity 的不同 lifecycle 狀態間會呼叫的 callback 方法，像是當 activity 被 create、 stop、 resume、 destroy 的時候。最重要的兩個 callback 方法為：

	
*   <code>onCreate()</code>  
    一定得實作，系統在建立 activity 時會呼叫這個方法。在你的實作裡，你應該要初始化 activity 必需的一些元件。最重要的是你要在這個方法裡呼叫 <code>setContentView()</code> 來定義 activity 使用者介面的佈局。
*   <code>onPause()</code>  
    這是當使用者要離開你的 activity 時系統會呼叫的第一個方法 (不過並不表示 activity 接著一定會被 destroy)。這個時候你應該要儲存在使用者離開 app 後還應該被保存的資料 (因為在 <code>onPause</code> 之後使用者有可能就不會再回到 app 了)。


除此之外還有其他幾個不同的 lifecycle callback 方法，你應該妥善在這些方法裡處理你的 app，以在不同 activity 之間提供流暢的使用者體驗，並且處理任何 activity 預期或非預期的 stop 甚至 destroy 的情況。所有的 activity lifecycle callback 方法都會在後面的 Managing the Activity Lifecycle 段落繼續介紹。


### Implementing a user interface

	
一個 activity 的使用者介面是由一層層 <code>View</code> 類別衍生的 view objects 所組成。每個 view 都控制一塊 activity 視窗內的長方形空間，並且可以與使用者互動。舉例來說，一個 view 可以是一個按鈕，在使用者按下時執行動作。


Android 提供了各種已經建立好的 view 讓開發者可以快速地用來設計及建立 app 的佈局。"Widget" 類型的 view 提供一個螢幕上可看見且可互動的元件，像是按鈕、輸入框、勾選框、或是一張圖片等。"Layout" 類型的 view 由 <code>ViewGroup</code> 所衍生，提供一個特定的佈局模型用來在內部放置 child view，這種佈局 view 有 linear layout、 grid layout、 relative layout 等。你也可以繼承 <code>View</code> 或 <code>ViewGroup</code> 類別或它們的子類別來建立你自己的 widget 及 layout 並在你的 layout 中使用。


以 view 來建立 app 佈局最常見的方式是利用存放在應用程式的 resource 裡的 XML 佈局檔。這樣做的話，你就可以將使用者介面的設計與 app 運作的程式碼分開來維護。你可以用 <code>setContentView()</code> 方法將佈局檔設為你的 activity 的 UI，傳入方法的參數為該佈局檔的 resource ID。你也可以在 activity 的程式裡建立新的 <code>View</code> 並且將新的 <code>View</code> 插入 <code>ViewGroup</code> 來建立 view 階層，並藉由將該 root <code>ViewGroup</code> 傳到 <code>setContentView()</code> 來使用這個佈局。


### Declaring the activity in the manifest

	
你必須在 manifest 檔裡宣告你的 activity，系統才能接觸到你的 activity。宣告的方法是在 manifest 檔的 <code>&lt;application&gt;</code> 元素內加上 <code>&lt;activity&gt;</code> 元素，如下面的例子：


    <manifest ... >
      <application ... >
          <activity android:name=".ExampleActivity" />
          ...
      </application ... >
      ...
    </manifest >


除了 <code>android:name</code> 以外還有其他的 activity 屬性，像是定義 activity 的標籤、圖示或是定義 activity UI 的佈局主題等。唯一必須標明的屬性只有 <code>android:name</code>，用來聲明 activity 的 class 名稱。一旦你的應用程式對外發表以後，這個名稱就不應該再被更改，因為更改之後可能會使某些與你應用程式相關的運作出錯，比如說裝置上開啟你的應用程式的捷徑等。


### Using intent filters

	
一個 <code>&lt;activity&gt;</code> 元素內可以加上 <code>&lt;intent-filter&gt;</code> 來聲明多個 intent filter，用來定義其他應用程式的 component 可以怎樣啟動這個 activity。


當你使用 Android SDK 工具建立新的應用程式時，通常會預先幫你建好 (或是在詢問你之後建立) 一個 activity，包含一個 intent filter 宣告這個 activity 會回應 "main" action 並且應該被放在 launcher 類別，也就是會顯示在裝置的 app 清單中。該 intent filter 會像這樣：


    <activity android:name=".ExampleActivity" android:icon="@drawable/app_icon">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>


<code>&lt;action&gt;</code> 元素聲明這是應用程式的主要開始點，<code>&lt;category&gt;</code> 元素聲明這個 activity 應該被列在裝置的系統應用程式 launcher (讓使用者可以開啟這個 activity)。


如果你不打算讓你的應用程式的 activity 被其他應用程式啟動，則你不需要再使用任何其他的 intent filter，只需要如上面的例子讓 app 最一開始的 activity 有 main action 和 launcher category 就好。如果你不希望讓其他應用程式可以開啟你的 activity，那就不應該有任何的 intent filter，你自己則是使用 explicit intent 來開啟你的 activity (後面會再說明)。

不過，如果你希望你的 activity 可以回應從你的或其他人的應用程式傳來的 implicit intent，那你就必須為你的 activity 定義額外的 intent filter。對於每種你希望能回應的 intent，你都必須在 manifest 的 <code>&lt;activity&gt;</code> 標籤內包含一個 <code>&lt;intent-filter&gt;</code>，該 intent filter 要包含一個 <code>&lt;action&gt;</code> 元素以及視需要而定是否要包含的 <code>&lt;category&gt;</code> 及 <code>&lt;data&gt;</code> 元素，這些元素讓你的 activity 得以區分要接受哪些類型的 intent。


## Starting an Activity

	
你可以呼叫 <code>startActivity()</code> 來開啟另外一個 activity，將描述目標 activity 的 <code>Intent</code> 傳入。Intent 會聲明確切的 activity 名稱或是描述你希望執行的 action 類型然後由系統來選擇裝置上合適的 activity。Intent 同時也可以攜帶小量的資料供接下來要開啟的 activity 使用。


大部份的時候你只需要開啟你的應用程式內的 activity，做法是建立一個 intent 並注明要開啟的 activity 的 class name。下面的例子是如何在 activity 內開啟另一個叫做 <code>SignInActivity</code> 的 activity：


    Intent intent = new Intent(this, SignInActivity.class);
    startActivity(intent);


不過，假如你的 app 需要用到某些特定功能如寄 email、傳簡訊等，你不一定要自己撰寫這些功能的 activity，而可以尋找裝置上其他具有這些功能的應用程式的 activity 來幫你執行。這時就可以體現 intent 的功能強大之處，你可以建立一個 intent、描述你想要執行的功能，系統便會為你搜尋裝置上能夠執行的應用程式。如果有多個擁有特定功能的應用程式，則使用者可以選擇要開啟哪個應用程式來執行你所需要的功能。舉個例子，如果你希望讓使用者寄一封電子郵件，你可以建立一個這樣的 intent：


    Intent intent = new Intent(Intent.ACTION_SEND);
    intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
    startActivity(intent);


加在 intent 裡的 <code>EXTRA_EMAIL</code> 這項資料是一個存著收件者郵件地址的 string array，當一個 email 應用程式收到這個 intent，會讀取 extra 資料內的 string array 並將資料放入郵件的收件者欄位。在這個情況時，email 應用程式的 activity 被 start、而當使用者寄完信後，你的 activity 便會 resume。


### Starting an activity for a result


有時候你可能會需要讓前一個 activity 接收後來 start 的 activity 傳回來的結果資料，這樣的話就呼叫 <code>startActivityForResult()</code> 來開啟新的 activity。在原本的 activity 加上 <code>onActivityResult()</code> callback 方法來接收並處理結果資料。當後來的 activity 結束後，會傳回一個包含結果資料的 <code>Intent</code> 到 <code>onActivityResult()</code> 內。


舉個例子，如果你希望讓使用者從他們的通訊錄選擇一個聯絡人並回傳給你的 activity、讓你的 activity 可以取得該聯絡人的資料來使用，你可以像下面的範例程式碼這樣建立一個 intent 以及處理回傳回來的結果：


    private void pickContact() {
        // Create an intent to "pick" a contact, as defined by the content provider URI
        Intent intent = new Intent(Intent.ACTION_PICK, Contacts.CONTENT_URI);
        startActivityForResult(intent, PICK_CONTACT_REQUEST);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        // If the request went well (OK) and the request was PICK_CONTACT_REQUEST
        if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
            // Perform a query to the contact's content provider for the contact's name
            Cursor cursor = getContentResolver().query(data.getData(),
            new String[] {Contacts.DISPLAY_NAME}, null, null, null);
            if (cursor.moveToFirst()) { // True if the cursor is not empty
                int columnIndex = cursor.getColumnIndex(Contacts.DISPLAY_NAME);
                String name = cursor.getString(columnIndex);
                // Do something with the selected contact's name...
            }
        }
    }


這個例子示範了你在 <code>onActivityResult()</code> 方法內應該使用的處理流程與基本邏輯：一開始要先像第 10 行一樣，確認對於資料的請求有成功 (回傳的 <code>resultCode</code> 會是 RESULT_OK)、以及確認這是對於哪項資料的請求 (在這個例子，比對後確認傳入 <code>startActivityForResult()</code> 的第二個參數與傳給 <code>onActivityResult()</code> 的 <code>requestCode</code> 是一樣的)，接著就可以藉由 query <code>Intent</code> 傳回的資料來進行後續的處理。


## Shutting Down an Activity


你可以藉由呼叫 activity 的 <code>finish()</code> 方法來結束一個 activity，也可以藉由呼叫 <code>finishActivity()</code> 來結束另一個先前開啟的 activity。


注意：在多數情況下，你不應該使用這些方法來結束一個 activity。下一篇文章會討論關於 activity 的 lifecycle，裡頭提到 Android 系統會替你管理 activity 的生命週期，所以你不需要自己結束你的 activity。主動藉由這些方法結束 activity 可能會影響預期的使用者體驗，只能在你相當確定不希望使用者再看到這些 activity 時使用。


## References

*   <a href="http://android-developers.blogspot.tw/2011/06/things-that-cannot-change.html" target="_new">Things That Cannot Change</a>


本篇主要翻譯自官方文件 [Activities](https://developer.android.com/guide/components/activities.html) 。

