---
layout: post
title: Activity Part 3
date: 2015-10-09 15:45:00 +08:00
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


這篇文章告訴你 activity 可能會在哪些非你或使用者控制的情況下結束並重新 create，以及，當這樣的情形發生時，你應該如何儲存及恢復 activity 的資料。最後會講到當一個 activity start 了另外一個 activity 時，兩個 activity 的 lifecycle callback 會以怎樣的順序被呼叫，你可以學習到如何在 activity 轉換的過程中妥善處理你的資料。


### Saving activity state


[前一篇](http://www.yipingshih.com/android/Activity_Part_2/){:target="_blank"} 提到當 activity 被 pause 或是 stop 的時候，activity 的狀態會被系統保留住，因為在 pause 或 stop 的狀態下 <code>Activity</code> 物件（包含所有的狀態以及成員的資訊）會被保留在記憶體內。因此，使用者在 activity 所做的變動也會保留住，當 activity 回到前景（resume）時，activity 看起來就和離開前一樣。


但是，當系統記憶體不足、為了取得更多資源而將背景的 activity destroy 時，<code>Activity</code> 物件也同時被 destroy，此時系統便沒辦法直接完整地恢復原本的狀態。當使用者重新把 activity 叫回前景時，系統便必須重新 create <code>Activity</code> 物件。但當使用者重新把背景的 activity 叫到前景時，並不會知道 activity 被 destroy 及 recreate，因此會期待 activity 要跟被叫到背景前的狀態一樣。在這個情況下，你可以藉由實做一個額外的 callback 方法：<code>onSaveInstanceState()</code>，來儲存 activity 的狀態及重要資訊，使 activity 可以在 recreate 的過程恢復成 destroy 前的狀態。


系統在銷毀 activity 前會呼叫 <code>onSaveInstanceState()</code>，傳入一個 <code>Bundle</code> 讓你將資料以 name-value pair 的形式存進去（使用 <code>putString()</code> 或 <code>putInt()</code> 之類的方法）。接著，如果系統 kill 了你的 application process 然後使用者又將背景的 activity 打開，系統會 recreate activity 並將 <code>Bundle</code> 傳給 <code>onCreate()</code> 以及 <code>onRestoreInstanceState()</code>。你可以在這兩個方法的任一之中從 <code>Bundle</code> 取回儲存的資料並恢復 activity 的狀態。如果沒有可以恢復的資料，傳入的 <code>Bundle</code> 會是 <code>null</code>（也就是 activity 是第一次被建立而非重新恢復）。


<img border="0" src="/images/20150928/restore_instance.png" />

上圖標示了 activity 重新回到前景獲得 user focus 並保留原先完整狀態的兩種方式（中間向下的箭頭為 activity 進入背景的過程）：第一種方式為 activity 被 destroy、接著被 recreate 並且從儲存的資料恢復原本的狀態，第二種方式為 activity 在 stop 後接著便 resume、原本的狀態都還完整保留著。


注意：<code>onSaveInstanceState()</code> 並不保證在 activity 被 destroy 前一定會呼叫，因為在某些使用情境下並不需要暫時儲存 activity 的狀態（例如使用者在 activity 按了手機的 back 鍵，表示他們真的想要結束這個 activity）。如果系統呼叫了 <code>onSaveInstanceState()</code>，時機會在 <code>onStop()</code> 前，可能也會在 <code>onPause()</code> 前。


不過，即使你沒有自己 implement <code>onSaveInstanceState()</code>，<code>Activity</code> class 原本的 <code>onSaveInstanceState()</code> 內其實就有預設會主動儲存某些資料。具體來說，<code>Activity</code> 預設的行為是去呼叫 layout 裡的每個 view 各自的 <code>onSaveInstanceState()</code>、讓每個 view 自己提供該被儲存的資訊。Android framework 裡的幾乎每個 widget 都 implement 了這個方法，讓 UI 的改變被自動儲存並在 activity 被 recreate 時恢復。舉兩個例子：<code>EditText</code> 會儲存使用者輸入的內容、<code>CheckBox</code> 也會儲存是否有勾選的狀態。不過讓這些 widget 自動儲存的前提是你必須要為這些 view 設 unique ID（<code>android:id</code> 屬性）；如果一個 widget 沒有被給予 ID，系統便不能儲存它的狀態。


即使預設的 <code>onSaveInstanceState()</code> 已經會自動儲存 UI 的重要資訊，你依然可能需要 override 來儲存額外的資料。舉例來說，你可能需要存 activity 活動的過程中對內部成員變數的修改（這些資料可能會影響到 UI，但是並不會被自動儲存）。


你可以藉由設定 <code>android:saveEnabled</code> 屬性為 <code>false</code>、或是呼叫 <code>setSaveEnabled()</code> 方法來停止 layout 內特定 view 的自動儲存。大部分的情況下你不需要這麼做，除非你想要用不同的方式來恢復 activity UI 的狀態。


因為 <code>onSaveInstanceState()</code> 的預設實作會協助儲存 UI 的狀態，如果你要 override 這個方法來儲存額外資訊的話，你應該要在方法內先呼叫 super 才接著做你想做的儲存動作。同樣的道理，如果你要 override <code>onRestoreInstanceState()</code>，你也必須要呼叫 super，這樣原本預設會自動儲存 UI 資料的行為才能正確運作。


注意：<code>onSaveInstanceState()</code> 並不保證會被呼叫，你應該只用這個方法來儲存 activity 暫時的資料（UI 的狀態）、永遠不能用這個方法來儲存長時間存在的資料。你應該在 <code>onPause</code> 或是其他使用者離開 activity 的 callback 方法內儲存長時間存在的資料（比如說寫入資料庫的資料）。


如果要測試你的 app 能不能正確恢復 activity 的狀態，你可以藉由旋轉手機、讓螢幕方向改變來測試。當螢幕的方向改變，系統會 destroy 並且 recreate activity 來為新的螢幕配置重新套用相對應的設定及資源。這意味著在 activity recreate 時恢復原本的狀態是很重要的事，因為使用者在使用手機時可能會很頻繁地旋轉螢幕方向。


### Handling configuration changes


裝置的某些配置會在執行過程中改變（像是螢幕方向、鍵盤開關、語言）。當這些配置改變時，Android 會 recreate 執行中的 activity（系統呼叫 <code>onDestroy()</code>、接著馬上呼叫 <code>onCreate()</code>）。這樣的設計是為了協助你的應用程式重新套用相對應的設定（像是不同的螢幕方向分別對應的 layout）。如果你照上述所說的來處理 activity 在 recreate 時的狀態恢復，你的應用程式在面對 activity lifecycle 過程中的各種突發狀況時都會有更好且更有彈性的處理。最好的處理方式就是照上面所說的，在 <code>onSaveInstanceState()</code> 內儲存狀態、在 <code>onRestoreInstanceState()</code> 或 <code>onCreate()</code> 內恢復狀態。


這篇官方的 [Handling Runtime Changes](https://developer.android.com/guide/topics/resources/runtime-changes.html){:target="_blank"} 有更多關於應用程式執行過程中可能會發生的裝置配置改變以及要如何處理的說明。


### Coordinating activities


當一個 activity start 了另外一個 activity，他們同時會經歷 lifecycle 的狀態改變。第一個 activity 會先 pause 然後 stop（如果它還有任何部分是可以在螢幕上被看見的，就不會 stop），另外一個 activity 則是會被 create。由於不同的 activity 可能會共用 disc 或其他地方所儲存的資料，你必須知道第一個 activity 在另一個 activity create 前並不會 stop。第二個 activity 在開始過程的 lifecycle 會跟前一個 activity 停止過程的 lifecycle 交叉。


當兩個 activity 在同一個 process 下並且由其中一個去 start 另外一個時，他們的 lifecycle callback 的順序是有被好好定義的。以下是當 <code>Activity</code> A start <code>Activity</code> B 時，他們的 lifecycle callback 被呼叫的順序：


1. <code>Activity</code> A 的 <code>onPause()</code>
2. <code>Activity</code> B 的 <code>onCreate()</code>
3. <code>Activity</code> B 的 <code>onStart()</code>
4. <code>Activity</code> B 的 <code>onResume()</code> （<code>Activity</code> B 現在擁有 user focus）
5. 如果 <code>Activity</code> A 在螢幕上完全看不到了，則 <code>Activity</code> A 的 <code>onStop()</code> 被呼叫


這個固定的順序使你可以在不同 activity 轉換的過程中管理你的資料。舉例來說，如果你必須在第一個 activity stop 以前將資料寫入資料庫使下一個 activity 可以讀取資料，那你就應該在 <code>onPause()</code> 內寫入而不是 <code>onStop()</code>。


本篇主要翻譯自官方文件 [Activities](https://developer.android.com/guide/components/activities.html){:target="_blank"} 。

