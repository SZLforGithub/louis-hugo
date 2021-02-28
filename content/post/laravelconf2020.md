---
title: "LaravelConf Taiwan - Less is More 參加心得"
description: "我就是跟著大家進來看看熱鬧"
summary: "Laravel Conf 是目前台灣最大的 Laravel 社群。在疫情影響下仍然如期舉辦，今年是第四屆的 Laravel Conf。"
featured_image: "/images/laravelconf2020.jpg"
images: ["/images/laravelconf2020.jpg"]
tags: [學習筆記,PHP,心得分享]
date: 2020-07-26T13:18:23+08:00
toc: true
disable_share: true
---
Laravel Conf 是目前台灣最大的 Laravel 社群。在疫情影響下仍然如期舉辦，今年是第四屆的 Laravel Conf。

## 要起飛了！快速了解 PHP 8 大進化
在令人驚豔的 PHP7 發布後的五年，PHP8 即將發佈，他會帶給我們更多驚喜嗎？

### JIT (Just In Time)
![](https://i.imgur.com/6fPiJxS.jpg)

在一般的流程中， PHP 不論有沒有使用 OPCache ，最後都會需要經過 Zend VM，而 JIT 可以藉由在執行時編譯產出機器碼來越過 Zend VM ，理論上可以帶給我們效能上的提升。

但真的是這樣嗎？

> It's no secret that the performance jump of PHP 7 was originally initiated by attempts to implement JIT for PHP. We started these efforts at Zend (mostly by Dmitry) back in 2011 and since that time tried 3 different implementations. We never moved forward to propose to release any of them, for three main reasons: {{< color color="#FF6600">}}They resulted in no substantial performance gains for typical Web apps;{{< /color >}} They were complex to develop and maintain; We still had additional directions we could explore to improve performance without having to use JIT.

以上引用自 [PHP 官方的 RFC](https://wiki.php.net/rfc/jit)，其中提到 JIT 並不會對一般的 Web apps 帶來明顯的效能提升，因為 PHP 效能主要受到 I/O-bound 的約束，而 JIT 帶來的 CPU-bound 方面的優勢並不明顯。

另一方面若是用強型別的寫法，在使用 JIT 的情況下，相對於弱型別也會有一定的效能提升，原因是如果是弱型別， Zend VM 會對型別做判斷，而在動態編譯時如果也做這樣的邏輯判斷，性能並不會提升很多。


對於這樣的改動，首先可以看出的就是往強型別靠攏的趨勢，這從 PHP7 就能看到一些端倪了。

講師在現場有做了一個簡單的小 Demo ，可以看出強型別的確是會提升效能沒錯，而這次既然有線上直播和聊天室，當然要好好利用，我當下就在聊天室 ~~刷了一波愛心~~ 問了一波大家對於強型別的使用習慣：
![](https://i.imgur.com/KB0W5dv.png)

可以看出社群的開發習慣也的確開始朝著這個方向調整，講師也建議大家要開始習慣強型別，我認為未來的 PHP 開發者勢必是需要熟悉強型別開發的。

另外還有一點值得紀錄的是圓桌會議時 ANT 大大提到的，當今技術潮潮追求的 tech trends (ABCDEMQ[^1])，有 PHP 的一席之地嗎？

當下講師沒有直接說出來，不過個人認為應該就是沒有啦，但講師同時也說到現在的網頁仍然有 79% 是 PHP 寫出來的，甚至舉了 Cobol 的例子，我當下腦子裡面就浮現大象變成長毛象的畫面...

當然 PHP 的開發團隊自然也有想到這一點，[同一篇 RFC](https://wiki.php.net/rfc/jit) 也有提到：

> Secondly - using JIT may open the door for PHP being more frequently used in other, non-Web, CPU-intensive scenarios - where the performance benefits will actually be very substantial - and for which PHP is probably not even being considered today.

可以看出開發團隊的野心，或許未來 PHP 在非 Web、高計算密集的場景也能有一席之地也說不定 ~~一統江湖，指日可待~~。


## 大象也能飛翔！聊 Laravel 效能調校
這一堂相對來說比較基本，大多是對 PHP 本身語法的效能測試，也提出一個想法： PHP 原生函式的 overhead 相對大，減少原生函式呼叫，可能會提升效能。

Laravel 的部分則是提出了 {{< inlineCode >}}php artisan optimize{{< /inlineCode >}}、移除掉多餘的 ServiceProvider 和 Middleware 這幾種方法，並提醒 Eager loading 的重要性，最近開發的專案剛好有使用到，蠻有感觸的。

沒有超出我預想外的優化方案，但也中規中矩的做了基本的介紹和解釋。

## Severless

最後幾堂(Severless PHP Case : Agile Dashboard via GitLab Board & API、Serverless PHP Applications、Serverless Laravel with Bref、我們與 Serverless 的距離)都是關於 Serverless 的講座，我必須致上十二萬分的歉意，因為 ~~我就爛~~ 工作上還沒能進階到使用這個部分，我聽的當下有努力聽並試圖理解，無奈自己觀念太差，只能隱約掌握到一點輪廓，貿然寫下心得不僅有誤人子弟之嫌，也是對演講者的不尊重。

講座內容都非常精彩且豐富，之後若是能力足夠，再回來重看肯定能有更多收穫，只可惜自己在大會前沒能做好 Serverless 相關的功課。

## 結語
去年 LaravelConf 時從苗栗上來台北，是第一個正式參加的 Conference ，看各路大神秀操作，算是大開眼界，更重要的是也讓一直閉門造車的自己看到外面的大大們是如何使用這個工具。

當時也默默替自己設定了一個目標：明年的 LaravelConf 時要以一個正式的後端工程師身份再次參加。

我總是會在在做白日夢的時候想，一年後的自己會在哪裡，在什麼樣的地方工作，認識些什麼樣的人，看到怎麼樣的世界，現在也算是勉強達成目標了吧，希望沒有讓一年前的我失望。

參考資料
---
1. [PHP 官方手冊](https://wiki.php.net/rfc/jit)
2. https://segmentfault.com/a/1190000021947001

[^1]:AI, Blockchain, Cloud, Data, Edge computing, Mobile, Quantum

