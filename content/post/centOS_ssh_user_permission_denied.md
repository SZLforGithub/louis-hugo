---
title: "CentOS 的 Permission denied (publickey,gssapi-keyex,gssapi-with-mic). 問題"
description: "為什麼內容農場的 SEO 這麼強可惡"
summary: "新建 user ssh 時遇到的權限問題。"
featured_image: "/images/linux.jpg"
images: ["/images/linux.jpg"]
tags: [學習筆記,Linux,心得分享]
date: 2020-10-11T14:02:31+08:00
toc: true
disable_share: true
---

## 問題描述
開好一個新的伺服器之後，我希望新建一個 user louis 讓他可以用 ssh 的方式登入，所以我建完帳號之後在該用戶的 home 目錄 {{< inlineCode >}}mkdir .ssh{{< /inlineCode >}} ，然後在 .ssh 裡面 {{< inlineCode >}}vim autorized_keys{{< /inlineCode >}}，最後把 local 的 ssh public key 放進 autorized_keys 裡面：

```bash=
$ adduser louis
$ passwd louis
$ usermod -aG wheel louis
$ su louis
$ cd
$ mkdir .ssh
$ vim autorized_keys
```

回到我自己的機器後 {{< inlineCode >}}ssh louis@serverIP{{< /inlineCode >}} 之後出現：
```bash=
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

## 解決方法
Google 一直搜到一個內容農場的文章，完全沒有幫助，這 SEO 到底為什麼可以做這麼好內容又可以這麼爛。

![](https://i.imgur.com/w3B4m4L.png)


最後是從錯誤訊息上判斷應該是跟權限有關的問題，多加了一些關鍵字才找到解法。

主要是 user 目錄底下的 .ssh 的權限必須是 700 而 autorized_keys 的權限必須是 600：

```bash=
$ chmod 700 .ssh
$ chmod 600 .ssh/authorized_keys
```

離開後重新 ssh 應該就可以成功登入了