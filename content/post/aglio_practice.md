---
title: "用 Aglio 做出漂亮的 API 文件"
description: "欸，你可以寫一個超屌超漂亮超方便看的 API 文件嗎"
summary: "Aglio 的簡單介紹，並透過一個 CRUD 的 API 作為例子。"
featured_image: "/images/aglio.jpg"
tags: [學習筆記,backend,心得分享]
date: 2020-11-13T17:54:10+08:00
toc: true
disable_share: true
---

## 前言
身為一名後端工程師，開 API 是工作中很重要的一個部分，而寫文件絕對是不想做但又不得不做排行榜的前五名，前端工程師要看、App 工程師也要看、PM 偶爾也想看，要是寫得爛肯定是被群起攻之。

![](https://i.imgur.com/tXHy0ob.png)

## 簡介
這次要介紹的工具就是 [Aglio](https://github.com/danielgtaylor/aglio)，根據在 Github 上的介紹，它是一種 [API Blueprint](https://apiblueprint.org/) 的渲染器(renderer)。而 API Blueprint 是一種語法，可以在基於 Markdown 的語法上增加一些簡單的 tag 來描述 API 行為，並以 .apib 作為副檔名。

> 白話文就是：用 Markdown 語法寫一個 .apib 檔的 API 文件，再用 Aglio 的指令把它編譯成一個 html 檔。

## 懶人流程

1. 確認你有安裝 Node.js

```bash
$ node -v
v14.4.0
```

2. 安裝 Aglio

```bash
$ npm install -g aglio
```

3. 開始撰寫你的文件並以 .apib 做為副檔名，你可能會需要用到 {{< inlineCode >}}aglio -i your_file_name.apib -s{{< /inlineCode >}}，讓你可以 hot reload ，不必每次修改都要重新編譯一次。

4. 完成之後使用 {{< inlineCode >}}aglio -i your_file_name.apib -o your_file_name.html{{< /inlineCode >}} 就可以成功編譯了

> {{< inlineCode >}}aglio -h{{< /inlineCode >}} 可以看到 aglio 相關的全部指令，上述用到的參數：  
> {{< inlineCode >}}-i, --input Input file{{< /inlineCode >}}  
> {{< inlineCode >}}-o, --output Output file{{< /inlineCode >}}  
> {{< inlineCode >}}-s, --server Start a local live preview server{{< /inlineCode >}}

## 實際範例

接著我會實際完成一個簡易的 API 文件，並附上圖片和指令的簡易說明

### 建立一個空的 api_doc.apib 檔

![](https://i.imgur.com/h5TvZ0Q.png)

### 輸入指令 {{< inlineCode >}}aglio -i api_doc.apib -s{{< /inlineCode >}}
```bash
$ aglio -i api_doc.apib -s
Server started on http://127.0.0.1:3000/
Rendering api_doc.apib
Socket connected
```
前往它給我們的 http://127.0.0.1:3000/ 你應該會看到一個空白的文件

![](https://i.imgur.com/GOLML6M.png)

接著對你的 api_doc.apib 做任何的編輯後儲存，你會發現它可以即時呈現在畫面上，接下來就可以開始完善你的 API 文件了。
    
### 語法介紹

![](https://i.imgur.com/8MKMbC8.png)

大致上會長得像這樣，完整的文件可以到[這裡](https://github.com/SZLforGithub/aglio-practice/blob/master/api_doc_all.apib)看到。

#### 版本號
```
FORMAT: 1A
```

#### 文件網址
會自動在下面所有的 API 添加這個網址
```
HOST: https//aglio.practice
```

#### Resource Group
一組 API Resource，在這裡用文章 (Post) 當做例子
```
# Group 文章
```

#### Resource
在文章這個 Group 裡面，我們首先定義一個創建文章的 API Resource (/api/post/)

```
## Create Post
```

#### Actions
最後再進一步定義對這個 API Resource 的操作，後面用中括號([])表示這個 API 的 HTTP method

```
### Create Post [POST]
```


> 在進入下一個語法前，你如果覺得疑惑：「奇怪，我為什麼不把 Actions 寫在 Resource 那一層就好，這樣不就可以少一層嗎？」
> 
> 那你 ~~真的是要打屁股~~ 可以跳到文件的第41行，這裡開始定義了一個 /api/post/{id} 的 API Resource ，並用三種 (GET、PUT、DELETE) Actions 去操作它，這樣的分層語法方便我們創造出符合 Rest 風格的 API 文件。

#### Request & Response
接著就是 API 實際的內容了

```
+ Request (application/json)
    + Body

            {
                "title": "test",
                "content": "This is a test post."
            }

- Response 200 (application/json)
    - Body

            {
                "id": 1,
                "title": "test",
                "content": "This is a test post.",
                "created_at": "2020/11/11"
            }
```

可以用 {{< inlineCode >}}+{{< /inlineCode >}} 、 {{< inlineCode >}}-{{< /inlineCode >}} 或 {{< inlineCode >}}*{{< /inlineCode >}}  來開頭，在 Request 後面可以指定你的 Content-Type，Aglio 會在渲染時自動幫你加上 Header，而 Response 後面則需要多加一個 HTTP Status Code ，你可以根據你的 API 來調整。如果需要 Header 也可以自己新增，語法和 Body 是一樣的。

#### include

最後再介紹一個 include 指令，將原本文章 Group 的內容移到 post_api_doc.md，再 include 進原本的 api_doc.apib

```
<!-- include(post_api_doc.md) -->
```

![](https://i.imgur.com/UxQurxA.png)

所以未來如果有更多的 API：

![](https://i.imgur.com/sLulVT3.png)

除了讓文件更清楚以外，也方便多人協作。

    
### 輸出為 html 檔

終於到了最後一步了
```
$ aglio -i api_doc.apib -o index.html
```

aglio 會在目錄下產生一個根據 api_doc.apib 內容渲染而成的 index.html 檔案，長得大概像這樣：

![](https://i.imgur.com/J4SjJSm.png)

如果你覺得預設的 template 不好看，也可以在輸出時指定別的 template：
```
aglio -i api_doc.apib --theme-variables Cyborg -o index.html
```

![](https://i.imgur.com/hjCoDG9.png)

最後輸出的樣子長[這樣](https://szlforgithub.github.io/aglio-practice/)

## 結語
以上針對撰寫 API 文件的工具 - Aglio 做了簡單的介紹，能耐心地看到這裡，相信你肯定也可以做出超屌超漂亮超方便看的 API 文件了。

如果想知道更進階的用法，可以到參考資料的官方文件查閱，如果有人問要怎麼做出超屌超漂亮超方便看的 API 文件的話，請大方地把這篇文章甩在他臉上

![](https://i.imgur.com/MqNY1L6.png)

## 參考資料
1. [API Blueprint 官方文件](https://apiblueprint.org/documentation/)
2. [Aglio Github](https://github.com/danielgtaylor/aglio)
3. [網路上前輩的文章](https://github.com/twtrubiks/aglio_tutorial)