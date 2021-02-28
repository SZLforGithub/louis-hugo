---
title: "Singleton pattern (單例模式) for PHP"
description: ""
summary: "單例模式是一種 design pattern，要求一個 Class 只能有一個實例，通常會使用在系統中需要一個全局物件的情況。"
featured_image: "/images/php.jpg"
images: ["/images/php.jpg"]
tags: [學習筆記,PHP,design pattern]
date: 2020-07-26T14:15:24+08:00
toc: true
disable_share: true
---

什麼是單例模式
---
單例模式是一種 design pattern，要求一個 Class 只能有一個實例，通常會使用在系統中需要一個全局物件的情況。

在實作時的邏輯就是：「一個 Class 只能有一個儲存自身實例化的靜態變數 {{< inlineCode >}}$instance{{< /inlineCode >}} 、和一個獲得這個變數的靜態方法 {{< inlineCode >}}getInstance(){{< /inlineCode >}} 」
實際在使用時會 call {{< inlineCode >}}getInstance(){{< /inlineCode >}} 這個靜態方法，他會檢查 {{< inlineCode >}}$instance{{< /inlineCode >}} 是否為空，若不為空則回傳，若為空則實例化 Class 賦予 {{< inlineCode >}}$instance{{< /inlineCode >}} 並回傳。

以PHP為例
---
```php=
<?php
final class Singleton
{
    private static $instance;
    
    public static function getInstance()
    {
        if (self::$instance === null) {
            self::$instance = new Singleton;
        }
        
        return self::$instance;
    }
    
    public $_somethingWeWantToKeep = null;
}
```
我們用 {{< inlineCode >}}final{{< /inlineCode >}} 宣告 class 來確保不會被繼承，並如同上面所說的，建立 static function 來 return 一個存了 class 實體化後物件的 static 變數，最後在 class 中宣告一個變數 {{< inlineCode >}}$_somethingWeWantToKeep{{< /inlineCode >}}。

實際上在使用的情況是這樣的：
```php=
Singleton::getInstance()->_somethingWeWantToKeep = 'I\'m smart!';
```
之後我們在任何地方都可以引用 {{< inlineCode >}}Singleton::getInstance()->_somethingWeWantToKeep{{< /inlineCode >}} ，並且都會得到一樣的 {{< inlineCode >}}I'm smart!{{< /inlineCode >}}

但這樣就夠了嗎？

在上面的例子中， Singleton 大致上完成了，但還不夠嚴謹。

### 防止被外部調用

舉一個簡單的例子：
```php=
Singleton::getInstance()->_somethingWeWantToKeep = 'I\'m smart!';
var_dump(Singleton::getInstance()->_somethingWeWantToKeep);    //I'm smart!

$stupidGuy = new Singleton;
$stupidGuy->getInstance()->_somethingWeWantToKeep = 'I\'m stupid!';
var_dump(Singleton::getInstance()->_somethingWeWantToKeep);    //I'm stupid!
```
上述的情況簡單來說就是在 Singleton 之後，被外部調用並重新賦值，所以我們可以在 class 中新增：
```php=
private function __construct() {}
```
覆寫 {{< inlineCode >}}__construct(){{< /inlineCode >}} 來避免 Singleton 被外部調用。

### 防止被 clone

還有可能出現另一種情況：
```php=
Singleton::getInstance()->_somethingWeWantToKeep = 'I\'m smart!';
var_dump(Singleton::getInstance()->_somethingWeWantToKeep);    //I'm smart!

$stupidGuy = clone Singleton::getInstance();
$stupidGuy->getInstance()->_somethingWeWantToKeep = 'I\'m stupid!';
var_dump(Singleton::getInstance()->_somethingWeWantToKeep);    //I'm stupid!
```

這個地方也是發生類似的事情，我們的 Singleton 被 clone 之後調用了 {{< inlineCode >}}getInstance(){{< /inlineCode >}} ，同樣的我們
```php=
private function __clone() {}
```
覆寫 {{< inlineCode >}}__clone{{< /inlineCode >}} 來避免 Singleton 被 clone 。

### 防止被序列化

最後還有一個可能是序列化(serialize)
```php=
Singleton::getInstance()->_somethingWeWantToKeep = 'I\'m smart!';
var_dump(Singleton::getInstance()->_somethingWeWantToKeep);    //I'm smart!

$stupidGuy = serialize(Singleton::getInstance());
$stupidGuy = unserialize($stupidGuy);
$stupidGuy->getInstance()->_somethingWeWantToKeep = 'I\'m stupid!';
var_dump(Singleton::getInstance()->_somethingWeWantToKeep);    //I'm stupid!
```

相對應的防範措施是
```php=
private function __wakeup() {}
```
所以完整的 Singleton 如下：
```php=
<?php
final class Singleton
{
    private static $instance;
    
    public static function getInstance()
    {
        if (self::$instance === null) {
            self::$instance = new Singleton;
        }
        
        return self::$instance;
    }
    
    public $_somethingWeWantToKeep = null;
    
    private function __construct() {}

    private function __clone() {}

    private function __wakeup() {}
}
```

結語
---
總之我們做的事情就是防止這位 stupidGuy 對這個應該要是單例的 Class 建了物件，然後還呼叫 static function 來修改我們想要保持不變的東西，也就是**在其他開發者在不知情的情況下，建立了副本並修改時，噴 error 給他**。

事實上 Singleton pattern 因為隱藏依賴和難以測試，被廣泛地認為是一種 anti-pattern，可以參考[在 stackoverflow 的討論](https://stackoverflow.com/questions/137975/what-is-so-bad-about-singletons)。