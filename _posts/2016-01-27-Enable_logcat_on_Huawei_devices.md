---
layout: post
title: Enable logcat on Huawei devices
date: 2016-01-27 18:45:00 +08:00
description: ""
headline: ""
categories: android
tags: 
  - Android
comments: true
mathjax: null
featured: false
share: true
published: true
---


這篇文章教你如何開啟華為裝置的 logcat 功能。


最近剛拿到一台華為的手機來測試，照往常一樣開啟開發人員模式、允許USB偵錯後，發現 logcat 只看得到系統的 log，app 的 log 不管怎麼試就是不會出現。查了以後才知道原來華為設計了這麼一個彩蛋給開發人員...


開啟華為手機 logcat 的方法如下：


1. 打開手機的電話 app（對，沒錯，就是用來打電話的那個電話 app）
2. 撥打電話號碼：<code>*#*#2846579#*#*</code>
3. 按完電話號碼以後不需要按播出鍵，手機會馬上自動跳出神秘的設定介面，這時依序至 ProjectMenu -> Background Setting -> Log setting，根據自己的需求打開 AP Log (app 的所有 log) 或 CP Log (app 的 debug 及 verbose log)。
4. 從網路上的討論看起來可能有的裝置需要重新開機才會生效，我自己的裝置是不需要。


神秘的設定介面還有其他的設定功能，我還沒有仔細研究。

工程師急著看 log 已經夠水深火熱了，設計這種東西是要逼死誰...
