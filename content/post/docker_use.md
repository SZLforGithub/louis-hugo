---
title: "Docker🐳 - 使用 (2/2)"
description: "偷懶了很久終於願意寫的 Docker 筆記"
summary: "從分享會簡報整理過來的筆記，簡單地記錄了自己的 Docker 學習歷程。"
featured_image: "/images/docker-banner.jpeg"
images: ["/images/docker-banner.jpeg"]
tags: [學習筆記,backend,心得分享]
date: 2022-05-24T22:11:33+08:00
toc: true
disable_share: true
---

## 前言

這篇是從一個每月固定舉辦的分享會簡報整理過來的，紀錄了自己從 {{< inlineCode >}}初學 Docker{{< /inlineCode >}} -> {{< inlineCode >}}實際使用別人的容器{{< /inlineCode >}} -> {{< inlineCode >}}自己做容器給別人用{{< /inlineCode >}} 的心路歷程。  

這是第二篇，還沒看過前篇的，[請往這裡走](https://szlforgithub.github.io/post/docker_intro)。  

在前一篇，我們聊的是 Docker 的簡介，這一篇我們會著重在 Docker 的使用上。  

走起！  

## Docker 三大元素

Docker 有一些基本的觀念和使用的元件，主要可以分為 {{< inlineCode >}}Image{{< /inlineCode >}}、 {{< inlineCode >}}Container{{< /inlineCode >}}、 {{< inlineCode >}}Registry{{< /inlineCode >}} 這三種，以下我們一個一個介紹：  

### Image (鏡像)

Image 是一個唯讀的模板，由一組用於創建 Container 的指令組成，由一層一層的 Layer 堆疊而成：

![](https://i.imgur.com/KDGhvIN.gif)
> Reference: [https://www.youtube.com/watch?v=8vXoMqWgbQQ](https://www.youtube.com/watch?v=8vXoMqWgbQQ)

### Container

Container 是根據 Image 的定義創建的可執行個體，可以想像 Image 內一層一層的 Layer 會依序執行，最後成為我們的 Container。  

Container 內包含了程式碼本身與所有執行所需的依賴項，執行時相當於在 Image 上新增一個可寫的存儲層，生命週期結束後清空。  

![](https://i.imgur.com/2DU6Bcf.png)

### Registry

用於存放 Images，可從 Registry pull or push Images，相當於開源程式碼中 Github 的角色，在容器中最常用的是 [Dockerhub](https://hub.docker.com/)  

## 使用流程
有了這三大元素，就可以有一個簡單的容器使用流程了：Developer 從 Registry 上 pull Images，接著開始做一些 Custom Layer，最後 Run 成我們專案需要的 Container：  

![](https://i.imgur.com/AnAsqBe.png)

## Dockerfile

### 如果沒有 Dockerfile

接著我們要進入 Dockerfile 這個工具的介紹，老樣子，在介紹之前，我們先看看如果沒有 Dockerfile，我們是怎麼使用 Docker 的：
1. 搜尋我想要的 Docker Image
2. Pull Image
3. 啟動並進入 Container
4. 安裝相關環境

轉換成指令的話就是：
1. docker search php -f is-official=true
2. docker pull php
3. docker run -ti php /bin/bash
4. maybe apt-get or other command in linux

這種做法如果遇到了這樣的情境：  
```
欸欸你那個 image 怎麼建的，我想要起一台測試機  
欸欸你那個 image 怎麼建的，我想要換到另一台機器  
欸欸你那個 image 怎麼建的，我想要 blablabla ...  
```

我們需要一個工具來讓這些只會伸手的傢伙閉嘴，所以 Dockerfile 就應運而生了。

### Dockerfile 如何拯救我們
Dockerfile 是一個純文字檔，包含了一連串的指令，可以對應到 Image 中介紹的 Layer。執行 docker run 時會根據 Dockerfile 中的指令 build 出 image。

以上的使用流程就可以簡化為以下的 Dockerfile：  
一個相對簡單的例子：
```Dockerfile
FROM php:7.4-cli
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD [ "php", "./your-script.php" ]
```

一個來自 [Dockerhub php image 官方文件](https://hub.docker.com/_/php) 的例子：
```Dockerfile
FROM php:7.4-fpm
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd
```

有了這樣的工具，當別人又開始在欸欸我想幹嘛幹嘛的時候，我們反手一個 Dockerfile 甩在他臉上就可以完美解決問題。  

## docker compose

### 如果沒有 docker compose
我們可能會需要在同一個容器內安裝語言環境、資料庫、web server… 等，但這種做法是[可行但不被推薦的](https://docs.docker.com/config/containers/multi-service_container)，因為那相當於把 Docker 當成 VM 使用。  
> It’s ok to have multiple processes, but to get the most benefit out of Docker, avoid one container being responsible for multiple aspects of your overall application. You can connect multiple containers using user-defined networks and shared volumes.

所以我們勢必得讓一個 Service 對應到一個 Container，也就是會有多個容器、多個 dockerfile、多行指令，並且在每一行指令中定義 port、DB 帳號密碼、volume local file… 等。  

這光聽就很蠢，所以我們需要一個工具來讓自己不用幹蠢事。  

### docker compose 如何拯救我們

docker compose 是為了定義和共用多容器應用程式而開發的工具，可以使用單一命令，控制整個專案的所有容器。  

將 docker compose 保留在專案的根目錄的版本控制，其他人可以透過簡單的指令啟動並參與專案，這也是開源專案中很常見的解決方案，很有效的避免的不同環境的問題和建置環境的冗余時間。  

這邊還是提供一個例子，來源自 PHP framework - Laravel 的 sail：
```yaml
# For more information: https://laravel.com/docs/sail
version: '3'
services:
    laravel.test:
        build:
            context: ./vendor/laravel/sail/runtimes/8.0
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        image: sail-8.0/app
        ports:
            - '${APP_PORT:-80}:80'
        environment:
            WWWUSER: '${WWWUSER}'
            LARAVEL_SAIL: 1
        volumes:
            - '.:/var/www/html'
        networks:
            - sail
        depends_on:
            - mysql
    mysql:
        image: 'mysql:8.0'
        ports:
            - '${FORWARD_DB_PORT:-3306}:3306'
        environment:
            MYSQL_ROOT_PASSWORD: '${DB_PASSWORD}'
            MYSQL_DATABASE: '${DB_DATABASE}'
            MYSQL_USER: '${DB_USERNAME}'
            MYSQL_PASSWORD: '${DB_PASSWORD}'
            MYSQL_ALLOW_EMPTY_PASSWORD: 'yes'
        volumes:
            - 'sailmysql:/var/lib/mysql'
        networks:
            - sail
        healthcheck:
          test: ["CMD", "mysqladmin", "ping", "-p${DB_PASSWORD}"]
          retries: 3
          timeout: 5s
networks:
    sail:
        driver: bridge
volumes:
    sailmysql:
        driver: local
```

可以看到，我們在裡面定義了兩個 service，並對不同的 service 都有不同的 image、ports、environment、volumes、networks 等等的設定，一個指令可以 run 多個 service，完美解決我們原本擔心的問題。

## 小結

這篇我們簡單的過了一下 Docker 一些基本的元件，以及 Dockerfile 和 docker compose 兩個工具，著重介紹他們分別解決了什麼問題，和工具解決的問題場景、發展脈絡，我認為會比介紹文件裡面已經有的一堆指令還要來得有幫助，畢竟哪個工程師不會查文件，是吧。

其實 Docker 這個系列很早就想寫，但一直難產，拖到今天總算是寫完了，感謝你看到這，希望有幫到你，掰啦！