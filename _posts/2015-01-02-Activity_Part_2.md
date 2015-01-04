---
layout: post
title: Activity Part 2
date: 2015-01-02 14:45:00 +08:00
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
published: false
---

<div class="content">
  <div id="post">
    <h1>{{ page.title }}</h1>
    <p class="excerpt">

{{ page.excerpt }}

	</p>
	
	<h2>Managing the Activity Lifecycle</h2>
	
<p>透過實作 lifecycle callback 方法來管理你的 activity 的 lifecycle 是建立一個穩定且靈活的應用程式的關鍵。一個 activity 的 lifecycle 會直接受到自己的 task、back stack 以及與其他 activity 的關聯所影響。</p>

<p>一個 activity 本質上而言可以以下列三種狀態存在：</p>

<li><code>Resumed</code></li>
<p style="margin-left:20px">
Activity 正在螢幕的前景並且擁有 user focus，也就是正在執行 (running) 的狀態。
</p>
<li><code>Paused</code></li>
<p style="margin-left:20px">
有別的 activity 在前景並且擁有 focus，但是這個 activity 本身還是看得見。也就是說，有另外一個 activity 蓋在這個 activity 上，但是那個 activity 可能是半透明的或是沒有占滿整個螢幕。一個 paused 的 activity 依舊是 alive 的（Activity 物件被保留在記憶體裡，保持所有的狀態以及成員的資訊，並且持續 attached to window manager），但是在系統的記憶體超級無敵低的時候還是有可能會被系統結束掉。
</p>
<li><code>Stopped</code></li>
<p style="margin-left:20px">
Activity 完全被另一個 activity 遮住（這個 activity 現在已經在背景了）。一個 stopped 的 activity 也依舊是 alive 的（Activity 物件被保留在記憶體裡，保持所有的狀態以及成員的資訊，但是沒有 attached to window manager）。此時 activity 已經不能被使用者看見，並且在系統需要記憶體資源時便可能被結束掉。
</p>
<p>如果一個 activity 是 paused 或 stopped，系統可以藉由要求它結束 (呼叫其 <code>finish()</code> 方法)或是直接 kill 其 process 來將 activity 從記憶體中移除。當一個 activity 在結束後 (被 finish 或 kill) 被重新開啟，它必須整個從頭 create 。</p>

	<h3>Implementing the lifecycle callbacks</h3>
	
<p>當一個 activity 在上述的不同狀態間轉換，會透過各種 callback 方法被通知。你可以 override 所有 callback 方法來在 activity 狀態轉換時進行各種適當的應對。下列是一個 activity 的簡單架構，列出了幾個主要的 lifecycle callback 方法：</p>

<p class="codeblock">
{% highlight java linenos=table %}
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
</p>

<p>注意：官方文件裡曾提到 "Your implementation of these lifecycle methods must always call the superclass implementation before doing any work"，也就是應該要先呼叫 <code>super.onXX()</code>，下面再接其他你需要執行的工作。不過我查了一下 Android 原始碼和網路上的其他討論，覺得官方這裡有點把話說太死了XD。在原始碼裡面，<code>onPause()</code> 或 <code>onResume()</code> 其實都沒做什麼事，所以原則上 super 放在哪裡不會有差別，至於其他 callback 方法的部分，可以參考 <a href="http://stackoverflow.com/questions/18821481/what-is-the-correct-order-of-calling-superclass-methods-in-onpause-onstop-and-o" target="_new">這篇文章</a> 以及裡面引用文章的說明，有的觀念覺得跟建立 component 有關的方法 (如 <code>onCreate()</code> 、 <code>onStart()</code> 及 <code>onResume()</code>) 應該要先呼叫 super，讓該初始化的流程先做完，跟結束 component 有關的方法 (如 <code>onPause()</code> 、 <code>onStop()</code> 及 <code>onDestroy()</code>) 則應該要最後呼叫 super，確保你需要使用的資源不會先被清除掉。文章裡也有一些其他開發者遇到、必須最先或最後呼叫 super 否則會有差別的例子。</p>

<p>總而言之，這些 callback 方法定義了一個 activity 的整個 lifecycle，藉由實作這些方法，你可以監控由以下三組 callback methods 組成的 activity lifecycle 巢狀迴圈：</p>

<li>整個 activity 的 lifecycle 介於對 <code>onCreate()</code> 的呼叫以及對 <code>onDestroy()</code> 的呼叫。你的 activity 應該要在 <code>onCreate()</code> 內進行與整體有關的設定 (像是定義佈局) ，並且在 <code>onDestroy()</code> 內釋放所有還保留著的資源。舉例來說，如果你的 activity 有一個 thread 在背景從網路下載資料，應該要在 <code>onCreate()</code> 建立該 thread 並在 <code>onDestroy()</code> 停止該 thread。</li>

      <li>如果一個 activity 處於可被使用者看見的狀況，則 lifetime 正介於對 <code>onStart()</code> 的呼叫以及對 <code>onStop()</code> 的呼叫之間。在這段期間，使用者可以在螢幕上看見 activity，並且與其互動。例如 <code>onStop()</code> 會在另一個 activity 開始、且自己這個 activity 已經被遮住不能被看見時呼叫。在這兩個方法之間，你可以管理顯示 activity 所需的資源。比如說，你可以在 <code>onStart()</code> 註冊一個 <code>BroadcastReceiver</code> 來監控會影響你的 UI 的改變，並且在 <code>onStop()</code> 結束註冊(unregister)，因為使用者這時已經看不到 activity 的顯示。系統在 activity 的整個 lifetime 可能會呼叫 <code>onStart()</code> 和 <code>onStop()</code> 數次，因為 activity 可能會數次在裝置的前景後景之間切換。</li>
      
<li>一個 activity 的前景期間介於對 <code>onResume()</code> 和 <code>onPause()</code> 的呼叫之間。在這段期間，該 activity 在螢幕上的位置會位於所有其他 activity 之前，並且擁有使用者的 input focus。一個 activity 可以頻繁地進出前景，比如說，<code>onPause()</code> 會在裝置進入睡眠或是有個對話視窗跳出時被呼叫。因為這個狀態可以被頻繁地切換，這兩個方法裡的程式運作必須足夠輕量以避免使用者在每次切換時等待過久。</li>
      
      <p>
      下圖說明了 activity 在 lifecycle 的不同狀態間切換的順序及循環，長方形表示開發者可以在 activity 的各狀態變化時實作的方法。
      </p>

<p>
	<img border="0" src="/images/post_activity_lifecycle.png" />
</p>

      <p>
        下面列出了圖中的 lifecycle callback 方法並詳加介紹。
      </p>
      
      <p>
      
      <li>
          <code>onCreate()</code>
      </li>      
          <span style="margin:20px;line-height: 180%">介紹：<br/></span>
          <span style="margin:40px">介紹介紹<br/></span>
          <span style="margin:20px;line-height: 180%">app 在此方法被呼叫後是否會被系統 kill：<br/></span>
          <span style="margin:40px">不會<br/></span>
          <span style="margin:20px;line-height: 180%">下一個 callback 方法：<br/></span>
          <span style="margin:40px"><code>onStart()</code><br/></span>
      
    <li>
          <code>onRestart()</code>
      </li>      
          <span style="margin:20px;line-height: 180%">介紹：<br/></span>
          <span style="margin:40px">介紹介紹<br/></span>
          <span style="margin:20px;line-height: 180%">app 在此方法被呼叫後是否會被系統 kill：<br/></span>
          <span style="margin:40px">不會<br/></span>
          <span style="margin:20px;line-height: 180%">下一個 callback 方法：<br/></span>
          <span style="margin:40px"><code>onStart()</code><br/></span>
          
          <li>
          <code>onStart()</code>
      </li>      
          <span style="margin:20px;line-height: 180%">介紹：<br/></span>
          <span style="margin:40px">介紹介紹<br/></span>
          <span style="margin:20px;line-height: 180%">app 在此方法被呼叫後是否會被系統 kill：<br/></span>
          <span style="margin:40px">不會<br/></span>
          <span style="margin:20px;line-height: 180%">下一個 callback 方法：<br/></span>
          <span style="margin:40px"><code>onResume()</code> 或 <code>onStop()</code><br/></span>
          
    <li>
          <code>onResume()</code>
      </li>      
          <span style="margin:20px;line-height: 180%">介紹：<br/></span>
          <span style="margin:40px">介紹介紹<br/></span>
          <span style="margin:20px;line-height: 180%">app 在此方法被呼叫後是否會被系統 kill：<br/></span>
          <span style="margin:40px">不會<br/></span>
          <span style="margin:20px;line-height: 180%">下一個 callback 方法：<br/></span>
          <span style="margin:40px"><code>onPause()</code><br/></span>
          
          <li>
          <code>onPause()</code>
      </li>      
          <span style="margin:20px;line-height: 180%">介紹：<br/></span>
          <span style="margin:40px">介紹介紹<br/></span>
          <span style="margin:20px;line-height: 180%">app 在此方法被呼叫後是否會被系統 kill：<br/></span>
          <span style="margin:40px">會<br/></span>
          <span style="margin:20px;line-height: 180%">下一個 callback 方法：<br/></span>
          <span style="margin:40px"><code>onResume()</code> 或 <code>onStop()</code><br/></span>
          
          <li>
          <code>onStop()</code>
      </li>      
          <span style="margin:20px;line-height: 180%">介紹：<br/></span>
          <span style="margin:40px">介紹介紹<br/></span>
          <span style="margin:20px;line-height: 180%">app 在此方法被呼叫後是否會被系統 kill：<br/></span>
          <span style="margin:40px">會<br/></span>
          <span style="margin:20px;line-height: 180%">下一個 callback 方法：<br/></span>
          <span style="margin:40px"><code>onRestart()</code> 或 <code>onDestroy()</code><br/></span>
          
          <li>
          <code>onDestroy()</code>
      </li>      
          <span style="margin:20px;line-height: 180%">介紹：<br/></span>
          <span style="margin:40px">介紹介紹<br/></span>
          <span style="margin:20px;line-height: 180%">app 在此方法被呼叫後是否會被系統 kill：<br/></span>
          <span style="margin:40px">會<br/></span>
          <span style="margin:20px;line-height: 180%">下一個 callback 方法：<br/></span>
          <span style="margin:40px">無<br/></span>
          
      </p>

      
The column labeled "Killable after?" indicates whether or not the system can kill the process hosting the activity at any time after the method returns, without executing another line of the activity's code. Three methods are marked "yes": (onPause(), onStop(), and onDestroy()). Because onPause() is the first of the three, once the activity is created, onPause() is the last method that's guaranteed to be called before the process can be killed—if the system must recover memory in an emergency, then onStop() and onDestroy() might not be called. Therefore, you should use onPause() to write crucial persistent data (such as user edits) to storage. However, you should be selective about what information must be retained during onPause(), because any blocking procedures in this method block the transition to the next activity and slow the user experience.

<p>

    
      
</p>
      
<p>
   上面所說「 app 在被呼叫後不會被系統 kill 」的 callback 方法，從他們被呼叫開始，便保護 activity 所在的 process 不被系統 kill。所以，一個 activity 從 <code>onPause()</code> return 開始，到 <code>onResume()</code> 被呼叫為止都是可被 kill 的。然後從 <code>onResume()</code> 被呼叫到下次 <code>onPause()</code> 再次被呼叫並 return 為止都是不會被 kill 的。
</p>
      
<p>
   注意：一個 activity 就算處於上面所說不會被 kill 的狀態，在極特殊的、系統完全沒有足夠資源的情況下，還是會被系統 kill。
</p>

</div>