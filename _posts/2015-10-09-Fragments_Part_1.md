---
layout: post
title: Fragments Part 1
date: 2015-10-09 16:45:00 +08:00
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


### 寫在前面


一個 <code>Fragment</code> 代表一個 <code>Activity</code> 內的運作或 ui 的一部分。你可以在一個 activity 內結合多個 fragment 來建立一個多重面板（multi-pane）介面、或是在多個 activity 內重複使用某個 fragment。你可以把 fragment 想像成是一個 activity 內獨立的模組，擁有自己的 lifecycle、自己獨立接收 input 事件、在 activity 執行的過程中可以任意添加或移除（像是一個 "sub activity" 讓你可以在不同 activity 重複使用）。


一個 <code>Fragment</code> 永遠只能附屬於 activity 內並且 fragment 的 lifecycle 會被自己附屬的 activity 的 lifecycle 影響。舉個例子，當一個 activity pause 時，所有附屬於它的 fragment 也會一起 pause；當一個 activity 被 destroy 時，所有附屬於它的 fragment 也都會 destroy。不過，當 activity 在 resumed 的狀態（正在前景執行），你可以獨立操作每個 fragment，比如說新增或移除它們。當你執行類似的 fragment 操作，你也可以把 fragment 加到一個由 activity 管理的 back stack，activity 的 back stack 裡的每個 entry 都是一筆已被執行的 fragment transaction。這個 back stack 讓使用者可以藉由按手機的 back 按鍵來返回某個 fragment 。


當你在 activity 的 layout 內加入一個 fragment，這個 fragment 會存在於 activity 的 view hierarchy 的一個 <code>ViewGroup</code> 內、並且會擁有自己的 view layout。在 activity 內加入一個 fragment 有兩個方法：在 activity 的 layout 檔案內宣告 <code>&lt;fragment&gt;</code> 元素，或是在程式碼內直接新增一個 <code>Fragment</code> 並加入現有的 <code>ViewGroup</code> 內。要注意的是，fragment 並沒有被規定一定要是 activity 介面的一部分，你可以新增一個沒有 UI 的 fragment，讓它單純作為一個看不見的元件來為 activity 執行任務。


接下來關於 fragment 的文章會介紹要如何在 app 內使用 fragment，包括 fragment 在被加進 activity 的 back stack 後如何管理狀態、如何與 activity 和其他 fragment 共享事件、對 activity 的 action bar 狀態的處理等等。


### Design Philosophy


Android 從 3.0 (API level 11) 開始加入 fragment，以支援平板電腦等大尺寸裝置上更靈活的 UI 設計。由於平板電腦的螢幕尺寸比一般手機大很多，因此有更多的空間可以重新組合和安排 UI 元件。Fragment 讓你不用處理 view hierarchy 的複雜改變也能做到這樣的設計。將 activity 的 layout 分隔成 fragment 後，你就可以在 activity 執行的期間去改變 activity 的組成並且將所做的改變儲存在 activity 管理的 back stack 中。


舉個例子，一個新聞 app 可以用一個 fragment 在畫面的左邊顯示文章列表、另一個 fragment 在畫面的右邊顯示文章內容，兩個 fragment 同時出現在一個 activity 內，擁有各自的一組 lifecycle callback、各自處理自己收到的 input 事件。這樣一來，使用者就可以在同一個 activity 內選擇並讀取文章，而不用每次選擇完文章後都被帶到新的 activity，如下面圖一所說明的平板 layout 一樣。


你應該把 fragment 設計成一個模型、可多次使用的 activity 元件。因為每個 fragment 都有自己的 layout 以及根據自己的 lifecycle callback 所決定的行為，你可以在多個 activity 內含有同樣的 fragment，所以在設計 fragment 時，你應該要考慮到重複使用性、並且避免用一個 fragment 去操作另一個 fragment。這件事格外重要是因為模組化的 fragment 使得你可以為不同的螢幕尺寸重新安排 fragment 之間的組合。

This is especially important because a modular fragment allows you to change your fragment combinations for different screen sizes. When designing your application to support both tablets and handsets, you can reuse your fragments in different layout configurations to optimize the user experience based on the available screen space. For example, on a handset, it might be necessary to separate fragments to provide a single-pane UI when more than one cannot fit within the same activity.


<img border="0" src="/images/post_imgs/20151009_fragments.png" />


Figure 1. An example of how two UI modules defined by fragments can be combined into one activity for a tablet design, but separated for a handset design.

For example—to continue with the news application example—the application can embed two fragments in Activity A, when running on a tablet-sized device. However, on a handset-sized screen, there's not enough room for both fragments, so Activity A includes only the fragment for the list of articles, and when the user selects an article, it starts Activity B, which includes the second fragment to read the article. Thus, the application supports both tablets and handsets by reusing fragments in different combinations, as illustrated in figure 1.

For more information about designing your application with different fragment combinations for different screen configurations, see the guide to Supporting Tablets and Handsets.





本篇主要翻譯自官方文件 [Fragments](https://developer.android.com/guide/components/fragments.html){:target="_blank"} 。

