---
title: "開發自己的 Composer package"
description: "用了那麼久的 Composer，也該自己寫一個了吧？"
summary: "開源的 Composer package 都會發布在 packagist 上，本文以一個最簡單的例子，走一次發布到使用的流程。"
featured_image: "/images/composer.png"
images: ["/images/composer.png"]
tags: [學習筆記,backend,心得分享,PHP]
date: 2021-05-08T15:03:37+08:00
toc: true
disable_share: true
---

## 前言
最近剛好有一些小工具的需求，本來是想要在 Local 用腳本處理就好，但既然都要寫了，不如寫成一個 Package，剛好填上一個文章的坑，之後如果要做成 web 也比較方便。

本篇的對象是已經了解什麼是 Composer，並且想要自己開發一個 Package 的朋友，如果你還不知道什麼是 Composer，[請移駕此處](https://szlforgithub.github.io/post/composer/)。

## Ok, Let's do it!

### Step 1. 建立你的 Package 資料夾
```bash
$ mkdir composer-practice
$ cd composer-practice
```

### Step 2. 初始化 Composer package
```bash
$ composer init
```
接下來會有一系列的交互式問答：

#### Package Name
```
This command will guide you through creating your composer.json config.

Package name (<vendor>/<name>) [suzhonglin/composer-practice]:
```
首先是 Package name，它會出現在完成頁面的這裡：
![](https://i.imgur.com/5gAn8db.png)
也就是別人 {{< inlineCode >}}composer require{{< /inlineCode >}} 時輸入的 Package 名稱。

#### Description
```
Description []:
```
Package 描述，會出現在完成頁面的這裡：
![](https://i.imgur.com/0iVgODc.png)

#### Author
```
Author [Louis.Su <i0989872540@gmail.com>, n to skip]:
```
維護者的姓名及連絡信箱。

#### Minimum Stability
```
Minimum Stability []:
```
這個值花了我一點時間才搞懂，[官方文件](https://getcomposer.org/doc/04-schema.md#minimum-stability)定義它由低到高有以下幾種值：{{< inlineCode >}}dev, alpha, beta, RC, and stable{{< /inlineCode >}}，並且標註了 {{< inlineCode >}}root-only{{< /inlineCode >}}。

首先它預設的是 {{< inlineCode >}}stable{{< /inlineCode >}}，意思是當 A package 依賴 B package 時，若 B package 都低於穩定版本(stable)時，就會出問題。而 {{< inlineCode >}}root-only{{< /inlineCode >}} 的意思則是對當前 A package 向下的依賴有效， B package 的 {{< inlineCode >}}composer.json{{< /inlineCode >}} 裡面同樣也可以設置 {{< inlineCode >}}Minimun Stability{{< /inlineCode >}}，這時的 root 就變成 B package 了。

講到這裡你應該可以明白，這個欄位做的是穩定性檢查，當你設為 {{< inlineCode >}}stable{{< /inlineCode >}} 時，你就可以確保你依賴的其他 Package 都會是 {{< inlineCode >}}stable{{< /inlineCode >}}。

#### Package Type:
```
Package Type (e.g. library, project, metapackage, composer-plugin) []:
```
看你的 Package 是哪一種類型就填哪一種，預設是 {{< inlineCode >}}library{{< /inlineCode >}}。

#### License
```
License []:
```
授權條款，我通常是用 MIT。

#### Dependencies
```
Define your dependencies.

Would you like to define your dependencies (require) interactively [yes]?
```
是否依賴其他的 Package，輸入 yes 之後：
```
Search for a package:
```
輸入 php
```
Enter the version constraint to require (or leave blank to use the latest version):
```
輸入依賴的 php 版本，這邊填的是 {{< inlineCode >}}^7.3 || ^8.0{{< /inlineCode >}}。

接下來會再問一次依賴的流程，但這次是在 {{< inlineCode >}}require-dev{{< /inlineCode >}} 的情況。

#### Generate
```
Do you confirm generation [yes]? yes
Would you like to install dependencies now [yes]? yes
```
最後會幫你安裝依賴的 Package，完成之後你應該要看到你的專案資料夾多了 {{< inlineCode >}}vendor{{< /inlineCode >}}、{{< inlineCode >}}composer.json{{< /inlineCode >}} 和 {{< inlineCode >}}composer.lock{{< /inlineCode >}}。

### Step 3. 完成 Package 內容
首先先來看一下我們的 {{< inlineCode >}}composer.json{{< /inlineCode >}}：
```json
{
    "name": "louissu/composer-practice",
    "description": "composer-practice is a package which I use to practice",
    "type": "library",
    "require": {
        "php": "^7.3 || ^8.0"
    },
    "require-dev": {
        "php": "^7.3 || ^8.0"
    },
    "license": "MIT",
    "authors": [
        {
            "name": "Louis.Su",
            "email": "i0989872540@gmail.com"
        }
    ],
    "minimum-stability": "dev"
}
```
還少了 autoload 的部分，在最後面加上：
```json
"autoload": {
    "psr-4": {
        "Louis\\": "src/test"
    }
}
```
{{< inlineCode >}}Louis{{< /inlineCode >}} 指的是 {{< inlineCode >}}namespace{{< /inlineCode >}}，{{< inlineCode >}}src/test{{< /inlineCode >}} 指的是對應的目錄。

接著新增我們剛剛指定的目錄，在專案根目錄新增 {{< inlineCode >}}src{{< /inlineCode >}} 資料夾和下一層的 test 資料夾，並新增一個 {{< inlineCode >}}Test.php{{< /inlineCode >}}：

```php
<?php
namespace Louis;
class Test 
{
    function test()
    {
        echo "This is my first composer package!";
    }
}
```
記得檔名跟類別名都要遵守 psr-4 的規範，到這裡已經基本完成整個 package 的內容了。

### Step 4. 上傳
到 [packagist](https://packagist.org/) 申請一個帳號，點擊右上方的 Submit：
![](https://i.imgur.com/DPQG7yE.png)
然後你會發現需要一個 Github Repository 的網址：
![](https://i.imgur.com/AxPjtaO.png)

將專案上傳到 Github 的部份就不用教學了吧，如果對 Git 不熟悉的話，[請移駕此處](https://szlforgithub.github.io/post/git/)。

將 Github Repository 的網址 Submit，應該就可以看到 Package 的畫面了。

![](https://i.imgur.com/s2GsKBf.png)

以上程式碼可以在這個 [Repository](https://github.com/SZLforGithub/composer-practice) 看到。

### 測試
新建一個專案資料夾，像一般在使用第三方套件一樣， {{< inlineCode >}}composer require louissu/composer-practice{{< /inlineCode >}}，記得換成自己的 Package name。

新建一個 {{< inlineCode >}}index.php{{< /inlineCode >}}：
```php
<?php
require "vendor/autoload.php";

use Louis\Test;

$test = new Test();
$test->test();

```

執行 {{< inlineCode >}}index.php{{< /inlineCode >}}，你應該會看到：
```
This is my first composer package!
```

以上程式碼可以在這個 [Repository](https://github.com/SZLforGithub/composer-practice-use) 看到。

## 結語
用了那麼久的 Composer，一直沒有真正的寫過一個 Package，在寫的過程也更理解 Composer 實際上是用什麼方式在管理 Package 之間的依賴，算是蠻有趣的收穫。

## References
1. https://packagist.org/
2. https://getcomposer.org/doc/04-schema.md