---
title: "PHP 關於 zero、false、null 和 isset、empty、is_null 之間的恩怨糾葛"
description: "下次再搞錯我就從這裡跳下去！"
summary: "PHP 混亂的空值判斷和隱藏在其後的自動轉型，惡名昭彰的程度應該只在 JS 之下，這是一篇自己梳理過一遍的紀錄。"
featured_image: "/images/php.jpg"
images: ["/images/php_best_practices.png"]
tags: [學習筆記,backend,心得分享]
date: 2021-05-01T13:08:19+08:00
toc: true
disable_share: true
---

## 前言
![](https://i.imgur.com/CLlEicj.png)
開個玩笑大家別當真啊，PHP 真香。  

之所以會有這篇比較基礎的文章，是因為去年在面試時，曾經被問到 isset、empty、is_null 三者的差別，但當時因為太依賴 [PHP 官方的比對表](https://www.php.net/manual/en/types.comparisons.php)，一時之間答得亂七八糟，但當時其實也怎麼沒放在心上，心裡想著實際遇到時再查表就好。  

但是，人生總是有個但是，後來工作上某個專案紮紮實實的踩到坑，浪費了我一個下午的美好時光。這個故事告訴我們，遇到坑不填，早晚會再摔進去一次。  

文中所有程式碼可以在這個 [repository](https://github.com/SZLforGithub/php-wtf) 看到。

---

## Environment
PHP 8.0.5 ([official docker image php:8.0.5-cli](https://hub.docker.com/layers/php/library/php/8.0.5-cli/images/sha256-6dbc7603c99016c013eb26ebb0f3898448a9d97670718d5cb11382750122d7ee?context=explore))

---

## 填坑囉

### [isset()](https://www.php.net/manual/en/function.isset.php)

字面上的意思就是判斷變數 {{< inlineCode >}}是否被設置{{< /inlineCode >}}，這裡的尚未被設置包含 {{< inlineCode >}}undefined{{< /inlineCode >}}、{{< inlineCode >}}尚未給予初始值{{< /inlineCode >}}、{{< inlineCode >}}null{{< /inlineCode >}} 這三種狀態，簡單來說就是 {{< inlineCode >}}檢查變數已經設置且值不為null{{< /inlineCode >}}

```php
<?php

// undefined
var_dump(isset($x)); // bool(false)

// 尚未給予初始值
$x;
var_dump(isset($x)); // bool(false)

// null
$x = null;
var_dump(isset($x)); // bool(false)

```

> 容易混淆的部分：  
> 1. 給 0 算是有設置，這點算合理，之所以會做這個檢查是因為 {{< inlineCode >}}0 == null{{< /inlineCode >}} 會回傳 true，看看我都被 PHP 嚇成什麼樣子了  
> 2. 如果對一個變數 {{< inlineCode >}}unset{{< /inlineCode >}}，那他自然會變成 isset false 的狀態  
```php
$x = 0;
var_dump(isset($x)); // bool(true)

unset($x);
var_dump(isset($x)); // bool(false)

```

---

### [empty()](https://www.php.net/manual/en/function.empty.php)

{{< inlineCode >}}empty{{< /inlineCode >}} 是一個比較不容易識別的詞，什麼叫空？0 算空嗎？null 算空嗎？我的錢包算...咳，直接看程式吧

```php
<?php

// undefined
var_dump(empty($x)); // bool(true)

// 尚未給予初始值
$x;
var_dump(empty($x)); // bool(true)

// null
$x = null;
var_dump(empty($x)); // bool(true)

// 0
$x = 0;
var_dump(empty($x)); // bool(true)

// 空字串
$x = '';
var_dump(empty($x)); // bool(true)

// 空陣列
$x = array();
var_dump(empty($x)); // bool(true)

// 字串 0
$x = '0';
var_dump(empty($x)); // bool(true)

// false
$x = false;
var_dump(empty($x)); // bool(true)

```

除了剛剛 {{< inlineCode >}}isset(){{< /inlineCode >}} 檢查的{{< inlineCode >}}undefined{{< /inlineCode >}}、{{< inlineCode >}}尚未給予初始值{{< /inlineCode >}}、{{< inlineCode >}}null{{< /inlineCode >}} 三種狀態以外，{{< inlineCode >}}0{{< /inlineCode >}}、{{< inlineCode >}}空字串{{< /inlineCode >}}、{{< inlineCode >}}空陣列{{< /inlineCode >}}、{{< inlineCode >}}'0'{{< /inlineCode >}}、{{< inlineCode >}}false{{< /inlineCode >}} 這幾種也算 empty

> 容易混淆的部分：  
> 1. 後兩種 {{< inlineCode >}}'0'{{< /inlineCode >}} 和 {{< inlineCode >}}false{{< /inlineCode >}}，比較容易搞混，因為 {{< inlineCode >}}'1'{{< /inlineCode >}} 和 {{< inlineCode >}}true{{< /inlineCode >}} 並不算空

---

### [is_null()](https://www.php.net/manual/en/function.is-null.php)

語意上是判斷是否為 {{< inlineCode >}}null{{< /inlineCode >}}，但 {{< inlineCode >}}undefined{{< /inlineCode >}} 和 {{< inlineCode >}}尚未給予初始值{{< /inlineCode >}} ，雖然 PHP 會丟一個 Warning 級別的錯誤，但 {{< inlineCode >}}is_null(){{< /inlineCode >}} 仍然會回 {{< inlineCode >}}true{{< /inlineCode >}}。


```php
<?php

// undefined
var_dump(is_null($x)); // Warning: Undefined variable $x bool(true)

// 尚未給予初始值
$x;
var_dump(is_null($x)); // Warning: Undefined variable $x bool(true)

// null
$x = null;
var_dump(is_null($x)); // bool(true)

// 0 不算是 null
$x = 0;
var_dump(is_null($x)); // bool(false)

// unset 之後 is_null false
unset($x);
var_dump(is_null($x)); // Warning: Undefined variable $x  bool(true)

```

簡單來說，如果無視錯誤的話， {{< inlineCode >}}is_null(){{< /inlineCode >}} 的行為是 {{< inlineCode >}}isset(){{< /inlineCode >}} 的相反

### PHP 的 Loose comparisons (==)
順便紀錄同樣很常搞錯的鬆散比較

```php
<?php

// 如果是 int 或者其中一個是 int ，會自動轉為 int 比對
var_dump(1 == '1'); // bool(true)
var_dump(2 == '1'); // bool(false)
var_dump('2' == '1'); // bool(false)

// null 會等於 0，但若是字串跟數字則都不等於
var_dump(null == 0); // bool(true)
var_dump('null' == 0); // bool(false)
var_dump('null' == '0'); // bool(false)
var_dump(null == '0'); // bool(false)

// false 會等於 0，字串的 false 相當正常，但 bool 的 false 會等於字串的 0
var_dump(false == 0); // bool(true)
var_dump('false' == 0); // bool(false)
var_dump('false' == '0'); // bool(false)
var_dump(false == '0'); // bool(true)

// 空字串則是都不等於 0
var_dump('' == 0); // bool(false)
var_dump('' == '0'); // bool(false)

```
這還只是一小部份，完成內容還是請參考[官方](https://www.php.net/manual/en/types.comparisons.php)，是不是看得頭很痛？那就對了，請愛用 {{< inlineCode >}}==={{< /inlineCode >}} 的嚴格比較，他會先檢查型別才檢查值，所以絕對不會出現意外。

## 結語
花了一些時間爬官方文件，也自己寫了[範例](https://github.com/SZLforGithub/php-wtf)，梳理了長久以來不太在意的問題，這次算是學到一個教訓，也希望未來遇到所有的坑都要記得盡量探究到底。

## References
1. https://www.php.net/manual/en/types.comparisons.php
2. https://www.php.net/manual/en/language.operators.comparison.php
