---
layout: post
title: Device Compatibility
date: 2015-01-01 15:20:00 +08:00
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

Android 系統可運行在各種不同類型的行動裝置上，而各種裝置的軟硬體功能以及系統所使用的 API 版本也不盡相同，因此在不同螢幕大小、不同版本的各種裝置上，你的 app 運行的情況可能會有所不同，再加上 app 內容的考量，可能不是所有地區的使用者都能使用。這篇文章告訴你能在 manifest 或 Google Play 開發者後台裡如何宣告各種規格及條件，確保只有合適並能正確執行程式的裝置下載使用你的 app。


## 寫在前面


Android 系統可運行在各種不同的裝置上，例如手機、平板電腦、電視等，讓開發者的應用程式得以擁有極大的市場。為了讓你的 app 能成功在各種裝置上執行，你的程式必須能處理各種裝置的軟硬體功能不一致的情況、並為不同規格的螢幕都提供合適的使用者介面。


聽起來很麻煩嗎？別擔心，Android 提供了一套框架來協助你完成這件事。你只需要提供各種規格的 resource 靜態檔案 (像是為不同的螢幕大小準備的 XML 佈局)，Android 系統會根據裝置的規格自動載入適合的 resource 檔案。因此，只要妥善規劃好你的 app、並準備好各種規格的檔案，你只需要發佈一個 APK 便能在許多的裝置上都最佳化你的 app 的使用者體驗。


即使如此，如果有需要的話，你可以標明你的 app 對於裝置的軟硬體功能需求，並控制哪些類型的裝置可以從 Google Play Store 下載你的 app。這篇文章接下來將說明你該如何做到這件事、以及如何確保你的 app 在商店上被正確的客群所看見。

	
## What Does "Compatibility" Mean?


在閱讀與 Android 相關的開發文件時，三不五時會在各種情況看到 "compatibility" 這個字。關於 compatibility(相容性) 通常指的有兩種可能：device compatibility 和 app compatibility。


關於 device compatibility，因為 Android 是個開放原始碼專案，任何的硬體製造商都可以製作出運行 Android 作業系統的裝置。不過，一個裝置只有在能正確地執行為了 Android execution environment 所寫的 app 時，才能被視作是 "Android compatible"。關於 Android execution environment 的定義可參考官方的 <a href="http://source.android.com/compatibility/overview.html" target="_new">Android compatibility program</a>，同時裝置必須通過 Compatibility Test Suite (CTS) 才能被視作是有相容性的。


上一段的細節其實可以不必深究，身為 Android app 開發者，基本上你不太需要管使用者的裝置是不是 Android compatible，因為只有 Android compatible 的裝置才能夠透過 Google Play Store 下載 app，所以你可以認定所有從 Google Play Store 安裝使用你的 app 的裝置都是 Android compatible的。


不過 app compatibility 就是你需要關心的事情了。你必須考慮到你的 app 是否與各種規格的裝置都能相容，因為能運行 Android 系統的裝置種類非常多樣，有些功能並不是所有裝置都有支援。舉例來說，有些裝置可能沒有方向的感應器，假如你的 app 的核心功能需要有方向感應器才能運作，則你的 app 只跟配備方向感應器的裝置相容。

	
## Controlling Your App's Availability to Devices
	

Android 讓你的 app 可以透過 platform API 使用裝置的各種特殊功能，有些功能是硬體方面的，像是方向感應器，有些如 app widget 則是軟體方面的。不是所有功能都能在所有裝置上使用，得看軟硬體分別是否有支援、有些功能則得視 platform 版本而定，所以你可能得根據你的 app 需要的功能，決定要讓哪些裝置能下載你的 app。


為了觸及盡可能多的使用者，你的 app 能支援的裝置應該要越多越好，在多數的情況下，如果有裝置無法支援 app 的特定功能，你可以選擇在程式執行的時候關閉部分功能並提供各種不同的 resource 檔案 (像是不同的螢幕規格使用不同的佈局)。不過，如果有必要的話，你可以根據下列幾項因素限制裝置從 Google Play 下載你的 app：


*   Device features
*   Platform version
*   Screen configuration

	
### Device features
	

為了讓你能根據裝置的特殊功能決定某些裝置是否能安裝你的 app，Android 為不是所有裝置都會配備的軟硬體特殊功能定義了 ID。比如說，方向感應器的 ID 是 <code>FEATURE_SENSOR_COMPASS</code>、app widget 的 ID 則是 <code>FEATURE_APP_WIDGETS</code>。


有需要的話，你可以在 manifest 檔案裡標明必要的 <code>uses-feature</code>，避免不具有某些功能的裝置下載安裝你的 app。


舉例來說，如果你的 app 無法在沒有方向感應器的裝置上運行，你可以像下面的範例這樣在 manifest 裡面把方向感應器列為必要的功能：


    <manifest ... >
        <uses-feature android:name="android.hardware.sensor.compass"
                  android:required="true" />
        ...
    </manifest>


Google Play Store 會將裝置和你列出的功能需求做比對，以得知你的 app 和使用者的裝置是否相容。如果裝置缺少了你列出的功能，就無法下載你的 app。


不過如果你的 app 的主要功能並不需要那些功能，你應該把 <code>android:required</code> 屬性設為 <code>false</code>，並在程式執行期間檢查裝置是否有那些功能。如果檢查發現使用者的裝置沒有那些功能，就妥善地隱藏相關的 app 操作。你可以像下面的程式碼這樣呼叫 <code>hasSystemFeature()</code> 來檢查裝置是否具有某項功能：


    PackageManager pm = getPackageManager();
    if (!pm.hasSystemFeature(PackageManager.FEATURE_SENSOR_COMPASS)) {
        // 這台裝置沒有方向感應器，關掉 app 裡相關的功能
        disableCompassFeature();
    }

關於你可以列舉的功能，可參考 <a href="https://developer.android.com/google/play/filters.html" target="_new">Filters on Google Play</a> 這份文件。

注意：有的系統權限會默認強致要求某些裝置功能，比如說如果你的 app 在 manifest 要求使用藍牙的權限，則系統就會默認一定需要 <code>FEATURE_BLUETOOTH</code> 這項裝置功能，那沒有藍牙的裝置就會無法從 Google Play 下載你的 app；你可以藉由在 &lt;uses-feature&gt; 將 <code>android:required</code> 屬性設為 <code>false</code> 取消這項要求。關於更多會被默認要求的裝置權限，可以參考 <a href="https://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions" target="_new">Permissions that Imply Feature Requirements</a> 這份文件。

	
### Platform version


不同的裝置會裝載不同版本的 Android 系統 (比如說 Android 4.0 或 Android 4.4)，每個版本通常都會加上新的 API，為了區分不同版本的 API，每個系統版本都有一個用來辨別的 API level。舉例來說，Android 1.0 的 API level 是 1、Android 4.4 的 API level 是 19。


藉由 manifest 裡的 <code>&lt;uses-sdk&gt;</code> 標籤的 <code>minSdkVersion</code> 屬性，你可以標明你的 app 最小相容到哪個 API level。


舉例來說，Calendar Provider 的 API 是從 Android 4.0 (API level 14) 以後才有的，如果你的 app 不使用這部分 API 就無法運作，那麼你應該要像下列程式碼這樣將你的 app 最低支援的裝置版本標為 API level 14：


    <manifest ... >
        <uses-sdk android:minSdkVersion="14" android:targetSdkVersion="19" />
        ...
    </manifest>


<code>&lt;minSdkVersion&gt;</code> 這項屬性宣告了你的 app 最小相容的版本，版本小於這個數字的裝置將無法從 Google Play Store 下載你的 app。另外，由於每個版本的 API 不盡相同，在遇到編譯 app 的 API 版本小於裝置本身的系統版本的情況，系統可能會啟動某些相容性的檢查及處理，以避免舊版本的 API 在新版本系統出錯，雖然比較安全但相對也可能會影響程式執行的效能，若宣告了 <code>&lt;targetSdkVersion&gt;</code> 這項屬性，表示你確定最多到這個版本都不會有相容性問題、不需要系統檢查，因此程式執行效率便會高一些。


每個版本的 API 都會顧及舊版本 API 的相容性，比如說如果某個 API 從某個版本開始不再支援，通常就會有另外的 API 及方法可以達到相同的功能，你在使用系統的 API 的過程中要檢查並確保與之後的新版本都能相容。


宣告了 <code>&lt;targetSdkVersion&gt;</code> 屬性後，高於這個版本的裝置還是可以下載安裝你的 app (如果你要禁止某個版本以上的裝置安裝，你可以宣告 <code>&lt;maxSdkVersion&gt;</code>，不過通常不會這樣做)，只是版本高於這個數字的系統就會在執行程式時進行相容性檢查。


舉個例子，Android 4.4 系統做了一些改變 (你可以參考這篇關於 Android 4.4 的 <a href="https://developer.android.com/about/versions/android-4.4.html#Behaviors" target="_new">Important Behavior Changes</a>)，利用 <code>AlarmManager</code> API 設定的提醒不會準確地在你設定的時間響起、而是會由系統將時間相近的各種提醒綁在一起一次通知，以節省系統的電力與資源。但如果你的 target API level 設定小於 19 (Android 4.4) 的話，系統對於你的 app 設定的提醒就會依照原本的 API 的行為準時響起。


如果你的 app 在某個小地方使用了較新的 API，你又不想因為一點小功能就提高 app 的 <code>&lt;minSdkVersion&gt;</code>，那你可以在程式執行期間進行檢查並在遇到較低版本的裝置時取消相關的功能。根據你的 app 需要的核心功能設定 <code>&lt;minSdkVersion&gt;</code> 的版本，然後在程式執行期間對於那些使用較新的 API 的小功能進行檢查，把裝置的系統版本 (<code>SDK_INT</code>) 跟你要檢查的 API 版本 (Build.VERSION_CODES 裡的 codename 常數) 進行比較，像下面的程式碼這樣：


    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB) {
        // Running on something older than API level 11, so disable
        // the drag/drop features that use ClipboardManager APIs
        disableDragAndDrop();
    }

	
### Screen configuration


Android 在各種不同螢幕規格的裝置上運行，包括手機、平板電腦、電視等，為了根據螢幕的類型區分這些裝置，Android 定義了兩個用來區分的特徵：螢幕大小 (螢幕實際的物理尺寸) 與 螢幕解析度 (density，螢幕上單位物理面積的像素密度，通常稱作 DPI)。為了簡化各種不同的規格，Android 為這些特徵制定了一些變數來將它們分類以方便區分：


*   螢幕大小區分成四種：small、normal、large 和 xlarge.
*   螢幕解析度區分成數種：mdpi (medium)、hdpi (hdpi)、xhdpi (extra high)、xxhdpi (extra-extra high) 等


預設的情況下，你的 app 和所有的螢幕大小及解析度都能相容，因為系統會為你的 UI 佈局以及圖片 resource 做合適的調整。即使如此，你還是應該要為各種螢幕規格提供各自適合的佈局以及圖片，以確保最佳化各種裝置的使用者體驗。


## Controlling Your App's Availability for Business Reasons


除了根據裝置規格限制使用者安裝你的 app 以外，你可能會因為商業考量或法律因素等原因來決定不同的使用者能否安裝。比如說，假如你的 app 提供的是台灣的高鐵時刻表，那對其他國家的使用者來說可能就不太實用，在這種非技術考量的情況，你可以從 Google Play 的開發者後台設定你的 app 可以被哪些條件的使用者看見。


總結就是，在你的 APK 檔案裡根據規格相容的考量來選擇可以安裝的使用者，然後在 Google Play 開發者後台根據非技術因素 (像是國家、某些特定廠牌的裝置等) 來選擇可以安裝的使用者。

  
## References


*   <a href="http://site.douban.com/128911/widget/notes/5269368/note/177080440/" target="_new">minSdkVersion、targetSdkVersion、targetApiLevel的区别</a>
*   <a href="http://meebox.blogspot.tw/2012/08/androidminsdkversionandroidtargetsdkver.html" target="_new">android:minSdkVersion、android:targetSdkVersion、Project Build Target 的區別</a>


本篇主要翻譯自官方文件 [Device Compatibility](https://developer.android.com/guide/practices/compatibility.html) 。
