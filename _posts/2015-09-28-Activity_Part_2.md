---
layout: post
title: Activity Part 2
date: 2015-09-28 18:15:00 +08:00
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


了解 Activity 的 lifecycle callback method 是非常重要的事。你必須知道每個方法被呼叫的時機以及順序，才能在 activity 的不同 lifecycle 階段執行合適的動作。這篇文章介紹了 Activity 的 lifecycle 以及你可能需要在各個 callback method 內執行的動作。


## Managing the Activity Lifecycle


透過實作 lifecycle callback 方法來管理你的 activity 的 lifecycle 是建立一個穩定且靈活的應用程式的關鍵。一個 activity 的 lifecycle 會直接受到自己的 task、back stack 以及與其他 activity 的關聯所影響。


一個 activity 本質上而言可以以下列三種狀態存在：


*   <code>Resumed</code>  
    Activity 正在螢幕的前景並且擁有 user focus，也就是正在執行 (running) 的狀態。
*   <code>Paused</code>  
    有別的 activity 在前景並且擁有 focus，但是這個 activity 本身還是看得見。也就是說，有另外一個 activity 蓋在這個 activity 上，但是那個 activity 可能是半透明的或是沒有占滿整個螢幕。一個 paused 的 activity 依舊是 alive 的（Activity 物件被保留在記憶體裡，保持所有的狀態以及成員的資訊，並且持續 attached to window manager），但是在系統的記憶體超級無敵低的時候還是有可能會被系統結束掉。
*   <code>Stopped</code>  
    Activity 完全被另一個 activity 遮住（這個 activity 現在已經在背景了）。一個 stopped 的 activity 也依舊是 alive 的（Activity 物件被保留在記憶體裡，保持所有的狀態以及成員的資訊，但是沒有 attached to window manager）。此時 activity 已經不能被使用者看見，並且在系統需要記憶體資源時便可能被結束掉。
    如果一個 activity 是 paused 或 stopped，系統可以藉由要求它結束 (呼叫其 <code>finish()</code> 方法)或是直接 kill 其 process 來將 activity 從記憶體中移除。當一個 activity 在結束後 (被 finish 或 kill) 被重新開啟，它必須整個從頭 create 。
    
    
### Implementing the lifecycle callbacks


當一個 activity 在上述的不同狀態間轉換，會透過各種 callback 方法被通知。你可以 override 所有 callback 方法來在 activity 狀態轉換時進行各種適當的應對。下列是一個 activity 的簡單架構，列出了幾個主要的 lifecycle callback 方法：

{% highlight java %}
    public class ExampleActivity extends Activity {
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            // The activity is being created.
        }
        @Override
        protected void onStart() {
            super.onStart();
            // The activity is about to become visible.
        }
        @Override
        protected void onResume() {
            super.onResume();
            // The activity has become visible (it is now "resumed").
        }
        @Override
        protected void onPause() {
            super.onPause();
            // Another activity is taking focus (this activity is about to be "paused").
        }
        @Override
        protected void onStop() {
            super.onStop();
            // The activity is no longer visible (it is now "stopped")
        }
        @Override
        protected void onDestroy() {
            super.onDestroy();
            // The activity is about to be destroyed.
        }
    }
{% endhighlight %}


注意：官方文件裡曾提到 "Your implementation of these lifecycle methods must always call the superclass implementation before doing any work"，也就是應該要先呼叫 <code>super.onXX()</code>，下面再接其他你需要執行的工作。不過我查了一下 Android 原始碼和網路上的其他討論，覺得官方這裡有點把話說太死了XD。在原始碼裡面，<code>onPause()</code> 或 <code>onResume()</code> 其實都沒做什麼事，所以原則上 super 放在哪裡不會有差別，至於其他 callback 方法的部分，可以參考 <a href="http://stackoverflow.com/questions/18821481/what-is-the-correct-order-of-calling-superclass-methods-in-onpause-onstop-and-o" target="_new">這篇文章</a> 以及裡面引用文章的說明，有的觀念覺得跟建立 component 有關的方法 (如 <code>onCreate()</code> 、 <code>onStart()</code> 及 <code>onResume()</code>) 應該要先呼叫 super，讓該初始化的流程先做完，跟結束 component 有關的方法 (如 <code>onPause()</code> 、 <code>onStop()</code> 及 <code>onDestroy()</code>) 則應該要最後呼叫 super，確保你需要使用的資源不會先被清除掉。文章裡也有一些其他開發者遇到、必須最先或最後呼叫 super 否則會有差別的例子。


總而言之，這些 callback 方法定義了一個 activity 的整個 lifecycle，藉由實作這些方法，你可以監控由以下三組 callback methods 組成的 activity lifecycle 巢狀迴圈：


*   整個 activity 的 lifecycle 介於對 <code>onCreate()</code> 的呼叫以及對 <code>onDestroy()</code> 的呼叫。你的 activity 應該要在 <code>onCreate()</code> 內進行與整體有關的設定 (像是定義佈局) ，並且在 <code>onDestroy()</code> 內釋放所有還保留著的資源。舉例來說，如果你的 activity 有一個 thread 在背景從網路下載資料，應該要在 <code>onCreate()</code> 建立該 thread 並在 <code>onDestroy()</code> 停止該 thread。
*   如果一個 activity 處於可被使用者看見的狀況，則 lifetime 正介於對 <code>onStart()</code> 的呼叫以及對 <code>onStop()</code> 的呼叫之間。在這段期間，使用者可以在螢幕上看見 activity，並且與其互動。例如 <code>onStop()</code> 會在另一個 activity 開始、且自己這個 activity 已經被遮住不能被看見時呼叫。在這兩個方法之間，你可以管理顯示 activity 所需的資源。比如說，你可以在 <code>onStart()</code> 註冊一個 <code>BroadcastReceiver</code> 來監控會影響你的 UI 的改變，並且在 <code>onStop()</code> 結束註冊(unregister)，因為使用者這時已經看不到 activity 的顯示。系統在 activity 的整個 lifetime 可能會呼叫 <code>onStart()</code> 和 <code>onStop()</code> 數次，因為 activity 可能會數次在裝置的前景後景之間切換。
*   一個 activity 的前景期間介於對 <code>onResume()</code> 和 <code>onPause()</code> 的呼叫之間。在這段期間，該 activity 在螢幕上的位置會位於所有其他 activity 之前，並且擁有使用者的 input focus。一個 activity 可以頻繁地進出前景，比如說，<code>onPause()</code> 會在裝置進入睡眠或是有個對話視窗跳出時被呼叫。因為這個狀態可以被頻繁地切換，這兩個方法裡的程式運作必須足夠輕量以避免使用者在每次切換時等待過久。


下圖說明了 activity 在 lifecycle 的不同狀態間切換的順序及循環，長方形表示開發者可以在 activity 的各狀態變化時實作的方法。

<img border="0" src="/images/20150928/post_activity_lifecycle.png" />

下面列出了圖中的 lifecycle callback 方法並詳加介紹。


|   方法名稱  | 介紹 | app 在此方法被呼叫後是否會被系統 kill |    下一個 callback 方法    |
|:-----------:|:----:|:-------------------------------------:|:--------------------------:|
|  onCreate() |   在 activity 被 create 時呼叫。<br>你應該在這個方法內執行所有必須的初始化動作：建立 view、將資料與 list 連結等等。<br>這個方法會傳入一個 Bundle 物件，包含 activity 先前的狀態（如果有的話）。<br>這個方法後會跟著執行 onStart()。   |                  不會                 |          onStart()         |
| onRestart() |   在 activity 被 stop 以後、再次 start 以前呼叫。<br>這個方法後會跟著執行 onStart()。   |                  不會                 |          onStart()         |
|  onStart()  |   在 activity 即將要被 user 看見以前呼叫。<br>如果 activity 接下來變成前景，這個方法後會跟著執行 onResume()；如果還沒完全出現在前景就被收起來，則會跟著執行 onStop()。   |                  不會                 |   onResume() 或 onStop()   |
|  onResume() |   在 activity 即將要可以跟 user 互動以前呼叫。<br>這時 activity 會在 activity stack 的最上面，並接收 user 的 input。<br>這個方法後會跟著執行 onPause()。   |                  不會                 |          onPause()         |
|  onPause()  |   在系統即將要轉到另一個 activity 以前呼叫。<br>這個方法通常會用來儲存尚未儲存的重要資料、停止動畫或其他較消耗 CPU 的動作。<br>這部分不能執行 block 太久的動作，因為下一個 activity 會等到這個方法 return 才執行其 resume 的程序。<br>如果 activity 接下來又回到前景，這個方法後會跟著執行 onResume()；如果對 user 來說變成完全看不見，則會跟著執行 onStop()。   |                   會                  |   onResume() 或 onStop()   |
|   onStop()  |   在 activity 對 user 來說變成完全看不到時呼叫。<br>看不到的原因有可能是因為被 destroy 或另一個 activity 來到前景並完全覆蓋此 activity。<br>如果 activity 接下來又準備回到前景，這個方法後會跟著執行 onRestart()；如果 activity 被 destroy 則會跟著執行 onDestroy()。   |                   會                  | onRestart() 或 onDestroy() |
| onDestroy() |   在 activity 即將被 destroy 時呼叫。<br>這是 activity 最後一個會被呼叫的 lifecycle callback。<br>被呼叫的原因有兩種，一個是因為 activity 正在 finishing（有人呼叫了 activity 的 finish()），或是因為系統為了獲取更多資源而將這個 activity destroy。<br>你可以使用 isFinishing() 來區分是哪一種被 destroy 的原因。   |                   會                  |             無             |



「app 在此方法被呼叫後是否會被系統 kill」表示系統能否在此方法 return 後直接 kill host activity 的 process、且不會執行任何一行 activity 的 code。有三個方法被標為「會」：<code>onPause()</code>、<code>onStop()</code> 和 <code>onDestroy()</code>。由於 <code>onPause()</code> 的執行順序是三個方法中最先的，從 activity 被 create 以後，<code>onPause()</code> 是 process 被 kill 前保證會被執行的最後一個方法——如果系統在緊急的情況下必須恢復記憶體，那 <code>onStop()</code> 和 <code>onDestroy()</code> 有可能不會在 kill 前被呼叫到。因此，你可能需要在 <code>onPause()</code> 內儲存重要的資料（例如使用者編輯的東西或是對 app 設定的更動）。要注意的是你必須自己判斷怎樣的資訊必須在 <code>onPause()</code> 內儲存，因為任何在這個方法內會 block 住 app 的動作就可能會使連往下一個 activity 的過程停頓並影響使用者體驗。


上面所說「 app 在被呼叫後不會被系統 kill 」的 callback 方法，從他們被呼叫開始，便保護 activity 所在的 process 不被系統 kill。所以，一個 activity 從 <code>onPause()</code> return 開始，到 <code>onResume()</code> 被呼叫為止都是可被 kill 的。然後從 <code>onResume()</code> 被呼叫到下次 <code>onPause()</code> 再次被呼叫並 return 為止都是不會被 kill 的。


注意：一個 activity 就算處於上面所說不會被 kill 的狀態，在極特殊的、系統完全沒有足夠資源的情況下，還是會被系統 kill。


本篇主要翻譯自官方文件 [Activities](https://developer.android.com/guide/components/activities.html) 。

