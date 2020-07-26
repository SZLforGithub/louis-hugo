---
title: "使用 PHP 獲取使用者 IP 以及漏洞分析"
description: "你獲取 IP 的方式真的安全嗎？"
featured_image: "/images/security.jpg"
tags: [學習筆記,資安,PHP]
date: 2020-06-06T18:21:38+08:00
toc: true
disable_share: true
---

工作上有這樣的需求，需要在使用者訪問網站之後獲取 IP 並儲存，花了一些時間研究，也發現很多有趣的地方，特地記錄下來。

市面上常見的方法
---
其實要得到使用者 IP 這件事情本身不難，拿磚頭往大學資工系隨便砸，砸到的那個學生應該都有辦法回答這個問題，以 PHP 來說不外乎是去讀取 {{< inlineCode >}}$_SERVER{{< /inlineCode >}} 這個預定義變數，包含了一大堆的參數可以對應到不同的值，這些值可能有可能沒有，有些可以偽造有些無法偽造。

主要是透過 {{< inlineCode >}}$_SERVER['HTTP_CLIENT_IP']{{< /inlineCode >}} 、 {{< inlineCode >}}$_SERVER['HTTP_X_FORWARDED_FOR']{{< /inlineCode >}} 、 {{< inlineCode >}}$_SERVER['REMOTE_ADDR']{{< /inlineCode >}} 等等的值來取到 IP。

問題
---
接下來就是有趣的地方了，這些值除了 $_SERVER['REMOTE_ADDR'] 以外**都是可以偽造的**，但唯一可靠的 $_SERVER['REMOTE_ADDR'] 得到的未必是使用者真實的 IP ，例如使用者有代理伺服器、或自己本身有使用反向代理伺服器，這樣的情況通常會去讀取 $_SERVER['HTTP_X_FORWARDED_FOR'] ，這個欄位是特地用來辨識有透過 HTTP 代理的使用者 IP ，在正常情況下他會儲存每一個經過的 IP (格式為 clientIP, proxy1, proxy2...)。

麻煩的地方在於：我們雖然可以透過 {{< inlineCode >}}$_SERVER['REMOTE_ADDR']{{< /inlineCode >}} 取得安全可靠的 IP ，但卻無法確定這是使用者真正的 IP ；當我們想透過 {{< inlineCode >}}$_SERVER['HTTP_X_FORWARDED_FOR']{{< /inlineCode >}} 來取得真正的 IP 時，卻無法確認這個 IP 是安全可靠的。

這個問題可大可小，小一點只是無法取得正確的 IP ，大一點別有用心的使用者可以透過偽造 {{< inlineCode >}}$_SERVER['HTTP_X_FORWARDED_FOR']{{< /inlineCode >}} 的方式達成 SQL injection 。

一方面再一次的驗證「所有來自客戶端的資料都是不可信任的」，另一方面也說明了 IP 無法成為辨別使用者身份的銀彈，至於如何辨別使用者身份，我們下一期會特地發一篇文章給各位介紹。

Talk is cheap. Show me the code.
---

要偽造 IP 不需要什麼神秘的工具，我這裡簡單用兩個 PHP 檔案就可以模擬出偽造 IP 的場景。

第一個檔案是 server.php ，裡面有一個 function getRequestIp() ，最後會消毒得到的IP後輸出，用來模擬在伺服器端獲取 IP
```php=
<?php
function getRequestIp()
{
    $ip_keys = [
        'HTTP_CLIENT_IP',
        'HTTP_X_FORWARDED_FOR',
        'HTTP_X_FORWARDED',
        'HTTP_X_CLUSTER_CLIENT_IP',
        'HTTP_FORWARDED_FOR',
        'HTTP_FORWARDED',
        'REMOTE_ADDR'
    ];

    foreach ($ip_keys as $key) {
        if (array_key_exists($key, $_SERVER) === true) {
            foreach (explode(',', $_SERVER[$key]) as $ip) {
                $ip = trim($ip);
                if ((bool) filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4 | FILTER_FLAG_NO_PRIV_RANGE | FILTER_FLAG_NO_RES_RANGE)) {
                    return $ip;
                }
            }
        }
    }

    return isset($_SERVER['REMOTE_ADDR']) ? $_SERVER['REMOTE_ADDR'] : '-';
}

$ip = getRequestIp();
echo 'UserIp: ' . $ip;
```
然後我們看一下直接訪問這個 server.php 得到的畫面：
![](https://i.imgur.com/b5xIxwZ.png)
嗯，就本機，沒毛病。

---

另一個檔案是 badMan.php ，裡面只是很基礎的用 curl 去打這個 server.php ，並在 HTTPHEADER 中帶上想偽造的參數，這裡以 X-FORWARDED-FOR 為例：
```php=
$ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, "server.php");
    curl_setopt($ch, CURLOPT_HTTPHEADER, array(
        'X-FORWARDED-FOR:1.2.3.4',
    ));

    curl_setopt($ch, CURLOPT_POST, 1);

    $output = curl_exec($ch);
    curl_close($ch);
```
接著執行 badMan.php ：
![](https://i.imgur.com/t9gttlY.png)
顯然 server.php 乖乖的被我們騙了。

這還是 server 端有乖乖消毒的情況，要是沒有消毒，我們把 X-FORWARDED-FOR 改成 {{< inlineCode >}}"Robert');DROP TABLE students;--"{{< /inlineCode >}} 之類的，就會聽到DB BOMB 的一聲跟大家說再見了。

參考資料
---
1. [如何正確的取得使用者 IP？](https://devco.re/blog/2014/06/19/client-ip-detection/)
2. [PHP 官方文件 ($_SERVER)](https://www.php.net/manual/es/reserved.variables.server.php)