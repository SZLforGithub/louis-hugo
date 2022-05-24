---
title: "Docker🐳 - 簡介 (1/2)"
description: "偷懶了很久終於願意寫的 Docker 筆記"
summary: "從分享會簡報整理過來的筆記，簡單地記錄了自己的 Docker 學習歷程。"
featured_image: "/images/docker-banner.jpeg"
images: ["/images/docker-banner.jpeg"]
tags: [學習筆記,backend,心得分享]
date: 2022-05-24T19:34:43+08:00
toc: true
disable_share: true
---

## 前言

這篇是從一個每月固定舉辦的分享會簡報整理過來的，紀錄了自己從 {{< inlineCode >}}初學 Docker{{< /inlineCode >}} -> {{< inlineCode >}}實際使用別人的容器{{< /inlineCode >}} -> {{< inlineCode >}}自己做容器給別人用{{< /inlineCode >}} 的心路歷程。  

如果你是了解在沒有 Docker 的情況下如何開發部署的工程師的話，這篇會非常適合你(因為我就是)，當然，完全是小白也沒問題。  

Ok. Let' do it!  

## 簡介
Docker 是一個開源、標準化、虛擬化的軟體工具，使用者可以透過 API 操作指令來建置、執行、部署容器(Container)。  

Docker 想解決的是傳統系統中 {{< inlineCode >}}環境難以保持一致性{{< /inlineCode >}} 與 {{< inlineCode >}}難以遷移{{< /inlineCode >}}，透過容器隔離出獨立的執行空間，並將應用程式與執行環境共同打包的方式，解決「在我的機器上沒問題啊」的問題。  

![](https://i.imgur.com/llo6WES.png)

## 演進

在學習一項技術時，知道它的演進脈絡會更有助於理解我們為什麼需要它，所以在進入 Docker 之前，我們先聊聊在沒有 Docker 的時候，Infrastructure 是怎麼運作的：  

> 以下皆為去脈絡化的簡易架構，僅提供說明，並不適用任何專案架構。

### 在沒有 Docker 的蠻荒時代

![](https://i.imgur.com/qyMuYXV.png)

一個非常簡(ㄙㄨㄟˊ)單(ㄅㄧㄢˋ)的架構圖，專注在 Backend 的部份，可以想像我們會有一台 Web Server (如 Nginx、Apache 等)，它會把流量導進 Application (如 Laravel、Gin 等)，然後或許會有跟 Database (如 MySQL、PostgreSQL 等) 的互動，以上就是一個傳統的單體式應用程式(Monolithic application) 的長相。  

接著如果有一顆隕石打過來，我們可能需要把整個 Service 從自架主機移動到雲端，或者要更換雲端主機的平台，要做 {{< inlineCode >}}遷移{{< /inlineCode >}} 這件事時，我們會需要考慮：  
- 語言環境的設定？
    - PHP5? PHP7?
- Web Server 上的設定檔與套件？
    - nginx.conf and Nginx Module?
- Database 的版本？
    - MySQL 5.7? MySQL 8.0?
- 加班會按照勞基法給薪給假嗎？
    - 週六週日哪一天是例日？哪一天是假日？給不給特休？

以上這些東西，除了最後一項是 HR 負責，其他幾項都需要一項一項排查，然後在新環境中一項一項安排好，然後跑跑看 Service 起不起得來？跑起來後功能是不是都正常？會不會有個外掛套件沒裝到，然後剛好沒寫測試或是測試沒覆蓋到？

先不談 scale out，光是遷移本身就是一個讓人非常煩躁的工程了。

那麼容器化可以拯救我們的加班時間嗎？

### Docker 的文明之火

![](https://i.imgur.com/NkSdWSI.png)

這張圖其實有些問題，不過姑且先這樣理解吧。  

我們將個別的服務包在容器裡面，形成各自獨立的執行個體，彼此之間藉由 docker network 互相溝通，這樣可以帶來幾項好處：  
- 將所有的版本、套件、設定檔都封裝在容器中，實現良好的可遷移性，替未來的可擴充的架構、微服務、Serverless 等打下基礎
- 所有容器皆可透過指令(或者是 Dockerfile or docker compose) 來操作，替後續自動化的持續整合與持續部署 (CI/CD) 打下基礎

以上就是有沒有使用容器化技術比較簡單的比較，兩相比較之下，應該可以比較理解 Docker 所說的 {{< inlineCode >}}Build and Ship any Application Anywhere{{< /inlineCode >}} 是什麼意思。

這時候比較有 sense 的讀者可能已經想到另一個隔離的技術，沒錯，就是 Virtual Machines，所以接下來我們再多聊聊兩者的差異。  

### Docker vs Virtual Machines (VMs)

Docker 和 VM 都是虛擬化的技術，差別在於對 OS 的使用方式不同。  

#### Virtual Machines (VMs)

![](https://i.imgur.com/3bomg57.png)

VM 需要藉由 Hypervisor 來模擬出軟體、韌體、硬體，並且在原有的 OS (Host OS) 上實際安裝一套新的作業系統 (Guest OS)。  

#### Docker

![](https://i.imgur.com/ynvL53C.png)

Docker 則是直接運行在原有的 OS (Host OS) 上，Container 中的環境 Base on kernel of Host OS，並利用 Linux 的 namespace 功能隔離資源。  

相較於 VM 省去了等待 Guest OS 開機的時間，所以更輕量化，執行與啟動的時間也更快。  

## 小結

有點太長了，我想去吃宵夜了，分成兩篇好了，第一篇我們介紹了 Docker 的前世今生，Docker 的實際操作就留給下一篇吧！