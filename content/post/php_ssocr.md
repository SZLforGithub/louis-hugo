---
title: "SSOCR 在 PHP 中的實作"
description: "找不到套件那就自己幹一個吧"
summary: "因為在工作上需要 SSOCR 的功能，但發現 PHP 只有一個跟 C library 的 bridge，所以決定自己實作一個 pure PHP 的版本。"
featured_image: "/images/php_ssocr.png"
tags: [心得分享,backend,php]
date: 2021-09-18T18:56:42+08:00
toc: true
disable_share: true
---

## 前言
Hi there，這次要分享的是最近工作上用到的 SSOCR (Seven Segment Optical Character Recognition) 功能，就是類似以下這樣的圖片：  

![](https://i.imgur.com/Zdp0IpY.png)

這樣的顯示方式被稱為 Seven Segment，因為它是由七個柱狀體來組合成數字，常見的中文名稱是七字節或是七段顯示器。  

而需要實作的功能，初步的想像是這樣的：會由前端傳來一張這樣的圖片，然後我(後端)要對這張圖片做簡單的處理和辨識，並回傳 {{< inlineCode >}}73{{< /inlineCode >}}。  


## 有哪些工具可以使用？
我第一步先嘗試了一些視覺辨識的套件，比如大名鼎鼎的 {{< inlineCode >}}Google Cloud Vision API{{< /inlineCode >}} 和 {{< inlineCode >}}Tesseract{{< /inlineCode >}}，但效果很出我意料的不甚理想，與其說是不甚理想，不如說根本完全無法辨識。  

過程中做了一些 survey，主要的原因是一般的視覺辨識套件的流程是：  
1. 灰階  
2. 二值化  
3. 根據二值化後的黑點分佈來做字體的判斷  

但是在七段顯示器中，每個柱狀體之間是有空間的，再加上每個柱狀體又奇形怪狀的，所以很容易被視為雜訊。  
> 這一篇心得文主要會著重在演算法的實作，若是對嘗試過程和套件之間的比較有興趣的話，可以移駕[我在公司內部分享會的簡報](https://docs.google.com/presentation/d/1--NM50IYO74ubPAd5qJz3QPYG_KRzoJimyJneJjMnkI/edit?usp=sharing) (已移除產品相關的敏感資訊)，裡面記錄了我撞牆的過程。
> 
> 其實這種場景目前比較主流的解決方案都是 train 一個 AI model 去辨識，因為七段顯示器也算是相對單純的 input，但礙於時間壓力和我本人的經驗不足，所以並沒有在產品上選用這個方案。

## 解決方案

接著我進一步搜尋在特化情況下的視覺辨識套件，找到了這一個用 C 寫的 [ssocr](https://github.com/auerswal/ssocr)，稍微嘗試了一下確實可以正確辨識，在[它的文件](https://www.unix-ag.uni-kl.de/~auerswal/ssocr/index.html)也把辨識的演算法寫得非常清楚，我心想真是太棒了，應該可以找到 PHP 版本吧，等下就解完可以去吃公司的零食了，結果...

![](https://i.imgur.com/bnlJzCl.png)

哭啊，只找到一個 call C library 的 bridge，但又覺得這種解法很不乾淨，可能還要考慮 Command injection 和接踵而來的 Remote Code Execution (雖然最後還是採用這個方案)，但我越想越不舒服，怎麼可以沒有呢？  

[![php_ssocr](/images/php_ssocr.png)](https://github.com/SZLforGithub/ssocr-php)

登愣！所以我自己寫了一個，所以現在有了！  

## 演算法實作
其實只是一個很簡單的小套件，我基本上就是參考了[ssocr 文件](https://www.unix-ag.uni-kl.de/~auerswal/ssocr/index.html)中的做法：  

![](https://i.imgur.com/e6pKCZr.png)

以下僅擷取部分程式碼：  

### step 1. 灰階與二值化
首先一樣是做圖片處理，我也提供從外部傳入參數對圖片做 resize 的功能，因為有些過大的圖片會導致下一個步驟 timeout  
```php
...

$im->thresholdimage($this->threshold * Imagick::getQuantum());

list($width, $height) = $this->getWidthAndHeight($im);

if ($this->scale) {
    $im->resizeImage(
        $width / $this->scale,
        $height / $this->scale,
        Imagick::FILTER_LANCZOS,
        1
    );
}

list($width, $height) = $this->getWidthAndHeight($im);

...
```

### step 2. 取得所有像素點與每個數字的邊界
第二步則是針對像素點的判斷和擷取，基本上 function 所做的事就如同它們的名字，要是有興趣請到 [repository](https://github.com/SZLforGithub/ssocr-php/blob/master/src/Main/ssocr.php) 內觀看。  
```php
...

$numberPixels = $this->getNumberPixels($im, $width, $height);

$numberPositions = $this->getXY($numberPixels);

...
```

{{< inlineCode >}}getXY{{< /inlineCode >}} 會回傳一個像這樣的二維陣列，代表每一個數字的邊界的四個值：  

```php
array(2) {
  [0] =>
  array(4) {
    'x1' =>
    int(17)
    'x2' =>
    int(46)
    'y1' =>
    int(12)
    'y2' =>
    int(71)
  }
  [1] =>
  array(4) {
    'x1' =>
    int(56)
    'x2' =>
    int(84)
    'y1' =>
    int(12)
    'y2' =>
    int(67)
  }
}
```
### step 3. 利用值來切割數字並辨識

```php
...

$im->writeImage('tmp.' . $this->image);

$result = $this->getResult($numberPositions);

unlink('tmp.' . $this->image);

return $result;
```

核心 function 就是這個 {{< inlineCode >}}getResult{{< /inlineCode >}}：  

```php
private function getResult($numberPositions)
{
    $result = '';

    foreach ($numberPositions as $key => $numberPosition) {
        $im = new Imagick();
        $im->readImage('tmp.' . $this->image);

        $im->cropImage(
            $numberPosition['x2'] - $numberPosition['x1'],
            $numberPosition['y2'] - $numberPosition['y1'],
            $numberPosition['x1'],
            $numberPosition['y1']
        );

        $size   = $im->getImageGeometry();
        $width  = $size['width'];
        $height = $size['height'];

        /**
         * Since the algorithm cannot recognize the digit one,
         * a digit that has a width of less than one quarter of it's height is recognized as a one.
         */
        if ($width < $height / 4) {
            $result .= '1';
            break;
        }

        /**
         * A vertical scan is started in the center top pixel of the digit to find the three horizontal segments.
         * Any foreground pixel in the upper third is counted as part of the top segment,
         * those in the second third as part of the middle and those in the last third as part of the bottom segment.
         */
        $numberPixels = $this->getNumberPixels($im, $width, $height);
        $line         = [0, 0, 0, 0, 0, 0, 0];
        $centerTop    = floor($width / 2);
        $thirdHeight  = floor($height / 3);
        $quarterLeft  = floor($height / 4);
        $halfWidth    = floor($width / 2);

        foreach ($numberPixels as $key => $numberPixel) {
            if ($numberPixel['x'] == $centerTop) {
                if ($numberPixel['y'] <= $thirdHeight) {
                    $line[0] = 1;
                } elseif ($numberPixel['y'] <= $thirdHeight * 2) {
                    $line[1] = 1;
                } elseif ($numberPixel['y'] <= $thirdHeight * 3) {
                    $line[2] = 1;
                }
            }

            if ($numberPixel['y'] == $quarterLeft) {
                if ($numberPixel['x'] <= $halfWidth) {
                    $line[3] = 1;
                } elseif ($numberPixel['x'] >= $halfWidth) {
                    $line[4] = 1;
                }
            }

            if ($numberPixel['y'] == $quarterLeft * 3) {
                if ($numberPixel['x'] <= $halfWidth) {
                    $line[5] = 1;
                } elseif ($numberPixel['x'] >= $halfWidth) {
                    $line[6] = 1;
                }
            }
        }

        foreach ($this->lineNumbers as $key => $lineNumber) {
            if ($line == $lineNumber) {
                $result .= $key;
                break;
            }
        }
    }

    return $result;
}
```

上面這段註解的部分都是擷取自 SSOCR 的文件中一些比較特殊的判斷或是作法，基本上就是判斷每一個切出來的數字中，有黑色像素點的區域，並且把可能性條列出來，最後再比較：  

```php
/**
 *  --0--
 * |3    |4
 * |     |
 *  --1--
 * |5    |6
 * |     |
 *  --2--
 */
protected $lineNumbers = [
    '2' => [1, 1, 1, 0, 1, 1, 0],
    '3' => [1, 1, 1, 0, 1, 0, 1],
    '4' => [0, 1, 0, 1, 1, 0, 1],
    '5' => [1, 1, 1, 1, 0, 0, 1],
    '6' => [1, 1, 1, 1, 0, 1, 1],
    '7' => [1, 0, 0, 1, 1, 0, 1],
    '8' => [1, 1, 1, 1, 1, 1, 1],
    '9' => [1, 1, 1, 1, 1, 0, 1],
    '0' => [1, 0, 1, 1, 1, 1, 1],
];
```

註解上的數字代表的是第二層的 index，以 2 這個數字為例，按照比劃順序是 0, 4, 1, 5, 2，對應的 index 拿到的值就會是 1，沒走到則是 0，所以就會得到 [1, 1, 1, 0, 1, 1, 0] 這樣的陣列。  

然後不得不自誇一下，我這個註解畫得真有靈性。  

## 結論
以上的文字搭配程式碼應該可以理解整個演算法的邏輯，我寫完之後有丟上 [Packagist](https://packagist.org/packages/louissu/ssocr-php) 了，相關的安裝與使用方式也可以在[文件](https://github.com/SZLforGithub/ssocr-php)裡面看到，希望這個套件能幫助到你，如果有任何功能也歡迎發 issue 或 pr，感謝你看到這，掰啦！