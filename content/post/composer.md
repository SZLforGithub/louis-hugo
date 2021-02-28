---
title: "Composer 介紹與使用"
description: "還在 include 一大堆檔案？"
summary: "Composer 是 PHP 用來管理套件的工具，是現代 PHPer 必備的武器之一。"
featured_image: "/images/composer.png"
images: ["/images/composer.png"]
date: 2020-06-07T19:33:01+08:00
tags: [學習筆記,PHP]
toc: true
disable_share: true
---

什麼是 Composer？
---

![](https://i.imgur.com/nHqr8I5.jpg)
<br>

Composer 是 PHP 用來管理套件的工具，是現代 PHPer 必備的武器之一。

為什麼我們需要 Composer？
---
在沒有使用 Composer 的情況下，每使用一個套件就必須 include 一次，套件又可能相依其他套件，於是我們又要再 include 那個套件。

若是自己開發的小專案還沒關係，要是專案一大起來，可能會有兩種情況，一種是不同的情況需要不同的套件，於是 include 就散落在各個檔案中，難以管理；另一種就是在程式進入點一次 include 一大堆，include 相連到天邊。


![](https://i.imgur.com/hS0gMsy.jpg)
<br>

但有了 Composer 之後，我們只需要一行
```php
require 'vendor/autoload.php';
```

怎麼樣？是不是有種把衣櫃整理乾淨的爽感？這就是我們使用 Composer 的原因。

怎麼開始？
---

### if (your_system == "windows")
[照著這份文件做](https://getcomposer.org/doc/00-intro.md#installation-windows)，下載  Composer-Setup.exe 後執行，他會下載最新版的 Composer 並修改你的 PATH 環境變數，但要記得重新啟動 terminal ，才會重新 load PATH。

---

### if (your_system == "Linux / Unix / macOS")
[照著這份文件做](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos)。  

> 分為 Locally 和 Globally，下載方式大同小異，差別在於把 Composer 放在專案內 (Locally) 或是全局可訪問 (Globally)，[這裡有一些討論](https://stackoverflow.com/questions/35405851/composer-installation-global-vs-local)。

首先請務必從 [Composer 正式的官網](https://getcomposer.org/download/)下載，依據執行下面四行指令：
```bash=
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e0012edf3e80b6978849f5eff0d4b4e4c79ff1609dd1e613307e16318854d24ae64f26d17af3ef0bf7cfb710ca74755a') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

分別做的事情為  
1. 下載 composer-setup.php 安裝程式  
2. 對下載的檔案做檢查，確保來源正確  
3. 執行 composer-setup.php 開始安裝  
4. 安裝完成後刪除 composer-setup.php  

---

按照上面的方法安裝完後，你會發現你的目錄中多了一個 composer.phar 檔案，執行 {{< inlineCode >}}php composer.phar{{< /inlineCode >}} ，出現 Composer 圖案就代表成功了。

![composer_success](https://i.imgur.com/ZsbNXIi.png)

想要 Globally 使用 Composer ，還需要把你的 composer.phar 移動到 /usr/local/bin 中，並確定自己有把 /usr/local/bin 加入 $PATH 參數中

```bash=
mv composer.phar /usr/local/bin/composer
```

直接執行 {{< inlineCode >}}composer{{< /inlineCode >}} ，如果看到一樣的 Composer 圖案就代表成功了。


如何使用？
---

### 初始化
```
composer -t
```
可以看到當前 Composer 的版本號。

切換到專案目錄執行
```
composer init
```
一陣問答之後會幫你產生一個 composer.json ，裡面會有 Composer 相關的設定，包含使用的套件以及版本等。

### 安裝套件
接著假設你想安裝的是 [phpUnit](https://phpunit.de/index.html)，按照官方文件執行
```
composer require --dev phpunit/phpunit ^9.1
```

{{< inlineCode >}}composer require{{< /inlineCode >}} 這個指令會安裝你指定的套件，並新增一個 composer.lock (如果沒有的話)，然後把這個套件的資訊記錄在 composer.lock 和 composer.json 中。

--dev 的意思則是相關設定會寫在 composer.json 和 composer.lock 的 require-dev 中，這樣的區分是因為有些套件只需要在測試環境中執行 (development or testing)，在正式環境中並不需要 (production)，在正式環境中只需要執行 {{< inlineCode >}}composer install --no-dev{{< /inlineCode >}} 就可以排除掉 require-dev 中的套件 ([詳見此](https://stackoverflow.com/questions/19117871/what-is-the-difference-between-require-and-require-dev-sections-in-composer-json))。

最後的 ^9.1 則代表版本。

### composer install
{{< inlineCode >}}composer install{{< /inlineCode >}} 指令會去找專案內的 composer.lock，並按照裡面記錄的套件和版號安裝。如果找不到 composer.lock ，就會按照 composer.json 的紀錄安裝，並產生一個 composer.lock。

### composer update
{{< inlineCode >}}composer update{{< /inlineCode >}} 則會根據 composer.json 中指定的版本下載。

也就是說，如果有 composer.lock，就乖乖用 {{< inlineCode >}}composer install{{< /inlineCode >}} 來確保自己的套件版本與原開發者是一模一樣的。若是用 {{< inlineCode >}}composer update{{< /inlineCode >}} 的話，就有套件的版本可能不同導致的不相容問題。另外，在沒有 composer.lock 的情況下， {{< inlineCode >}}composer install{{< /inlineCode >}} 和 {{< inlineCode >}}composer update{{< /inlineCode >}} 做的事情是一樣的 (讀取 composer.json 並安裝套件)。

以上就是基本的 Composer 使用方式，想了解更多請至參考資料。

參考資料
---
1. https://getcomposer.org/
2. http://blog.tonycube.com/2016/12/composer-php.html
3. https://medium.com/@shengyou/php-%E4%B9%9F%E6%9C%89-day-28-composer-%E5%BE%9E%E5%85%A5%E9%96%80%E5%88%B0%E5%AF%A6%E6%88%B0-4d3b34a91946


