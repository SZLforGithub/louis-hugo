---
title: "Git 簡單介紹與常用指令"
description: "Git 初心者大全，開著這篇用 Git 就對了！"
summary: Git 初心者大全，開著這篇用 Git 就對了！
featured_image: "/images/git.png"
images: ["/images/git.png"]
tags: [學習筆記,版本控制]
date: 2020-06-13T16:24:11+08:00
toc: true
disable_share: true
---

什麼是 Git？
---
Git 是一種分散式版本控制系統。

### 版本控制系統？
未導入版本控制系統之前，我們對專案進行修改或整合時，必須耗費大量的人力來對程式碼做備份和比對的工作。這在專案規模逐漸擴大或是多人協作的場景時，每個人的程式碼進度不一，哪段程式碼被修改或覆蓋？這段程式碼和上一次的版本有什麼不同？導致程式碼混亂不堪，難以維護，而版本控制系統就是要解決這樣的問題。

### 分散式？
版本控制主要分為集中式(Centralized Version Control Systems，CVCSs)和分散式(Distributed Version Control Systems, DVCSs)兩種，前者如 SVN，後者如 Git。

在 Git 中，每個協作者都會擁有一個自己完整的版本庫，你可以在自己的版本庫中盡情的開 branch、修改程式碼，只要不 push 到主要的版本庫，你不會影響任何一個與你協作的夥伴。

而只要將專案 clone 到自己的設備，你不需要連網，就可以在自己的版本庫中查看歷史紀錄({{< inlineCode >}}git log{{< /inlineCode >}})、創建分支({{< inlineCode >}}git branch{{< /inlineCode >}})、切換分支({{< inlineCode >}}git checkout{{< /inlineCode >}})、甚至是提交({{< inlineCode >}}git commit{{< /inlineCode >}})。也就是說假設你遇難隻身一人漂流到孤島，突然有一股修改專案的衝動，打開筆電就可以開始工作。

若您在公司使用的是SVN，但又想體驗 Git 自由自在的感覺，Git 也有和 SVN 溝通的 bridge，請 Google git svn。

概念
---
![Git](https://i.imgur.com/94TZTAv.png "Git")
Workspace 是工作目錄，index 或者 stage 是暫存區(所以也有一些文章將暫存區翻譯為索引)，local repository 是本地數據庫，remote repository 是遠端數據庫。

基本的開發流程是在 workspace 修改程式碼完成功能或修改之後，分批的將修改 add 進 index(stage) 中，再將這部分的修改 commit 到 local repository 並寫好符合規定的 description。

或許 stage 對於初使用 Git 的人來說有點不直覺，可能會認為每一次 commit 之前還要先 add 有點多此一舉。但在工作目錄(workspace)和數據庫(local repository)多一個暫存區(stage)的設計讓 Git 有更多彈性。

實際使用時可能會有很多需要分批上傳的情景，例如目前的 workspace 有兩個 feature 的修改，我可以將 featureA 先 add 後 commit，再做 featureB 部份的上傳，這樣的好處是不同的修改都可以擁有自己的 commit description，在 commit tree 中可以明確地看出來。

另外有一個有趣的地方需要注意，如果先對 hello.php 做了 functionA 部份的修改後 add，再對同樣在 hello.php 中的 functionB 部份做修改，這時候 commit 的話只會對第一次修改做上傳，如果在上傳之前 git status 會看見 hello.php 同時出現在 Changes to be committed 和 Changes not staged for commit 中，這是因為 add 是對修改做快照，而不是檔案。


常用指令
---
### git init
切換到專案目錄並輸入，會在該目錄新增一個 .git目錄，代表 Git 開始對該目錄進行版控。

---
### git clone
```git=
git clone git://XXXXXXXX
```
從目標專案複製到本地，這個複製幾乎包含了該專案的所有資料，包含歷史紀錄、分支、標籤等，而不僅僅是檔案，並會自動設定此遠端數據庫為 origin。

---
### git remote
查看遠端數據庫，若目前的本地數據庫是從遠端數據庫 clone 來的，則會看到預設的 origin。
<br>
```git=
git remote -v
```
參數 -v 會在名稱後方顯示 URL
<br>
```git=
git remote add name git://XXXXXXXX
```
add 指令可以新增一個遠端數據庫，並取名為 name。

---

### git status
查看當前專案的狀態，是否有新增刪除檔案或修改程式碼。
<br>
```git=
git status -s
```
參數 -s 可以看到比較簡潔的輸出格式
M = modified
D = deleted
R = renamed
C = copied
U = updated but unmerged

---
### git diff
直接輸入 {{< inlineCode >}}git diff{{< /inlineCode >}} 可以看到當前工作目錄和暫存區的差別。
```git=
git diff <commitId1> <commitId2>
```
可以比較兩次 commit 的差別。
<br>
```git=
git diff --staged
```
參數 --staged 則顯示暫存區和上次 commit 的差別

---
### git log
從新到舊列出所有提交的歷史紀錄，包含 Commit 的作者、時間和description。
<br>
```git=
git log -p -2
```
參數 -p 可以列出提交的內容，-2則限制最近的兩筆更新
<br>
```git=
git log --stat
```
參數 --stat 則顯示更新內容的簡略內容，包含被更動的檔案、更動多少檔案、有多少行被修改
<br>
```git=
git log --graph
```
參數 --graph 會在 log 旁用 ASCII 畫出分支的分歧和合併

---
### git add
```git=
git add hey.php
```
可以將尚未被追蹤或有修改的檔案加入暫存區
<br>
```git=
git add *.php
```
也可以使用萬用字元
<br>
```git=
git add -A
```
參數 -A 可以將所有修改加入暫存區，等效於 `git add .` 或 `git add --all`

---
### git rm
刪除檔案並將刪除記錄存進暫存區
<br>
```git=
git rm --cached
```
參數 --cached 將檔案從 git 中移除，並沒有真的刪除檔案，而是將檔案變成 untracked。

---
### git mv
修改檔名並將修改記錄存進暫存區
```git=
git mv oldName newName
```

---
### git commit
將暫存區提交到儲存庫，這個指令會打開編輯器，輸入這次的 commit description 後離開編輯器，git 便會完成提交。
<br>
```git=
git commit -m "This is commit descriptions"
```
參數 -m 可以直接輸入 commit description
<br>
```git=
git commit -a
```
參數 -a 可以省略 `git add` 的步驟，直接將修改存入暫存區並上傳

---
### git branch
顯示分支列表，有*代表當前分支。
<br>
```git=
git branch myfeature
```
建立 myfeature 分支
<br>
```git=
git branch -m oldName newName
```
將 oldName 這個分支的名稱改為 newName
<br>
```git=
git branch -d myfeature
```
刪除 myfeature 分支

---
### git merge
```git=
git merge myfeature
```
會將 myfeature 這個分支合併到當前分支，所以在 merge 前記得先 checkout 到(通常是) master 分支

---
### git rebase
{{< inlineCode >}}git rebase{{< /inlineCode >}} 是一個很有趣的功能，也有其他不同功能的用法，這裡主要介紹它在合併上的使用，進階的使用請見 [Git 常見問題](https://szlforgithub.github.io/post/gitproblem/#我可以修改更之前的-commit-嗎)。

base 是基準點，也就是分支從哪裡長出來，所以 rebase 的意思接近於「重新定義分支的參考基準」。

假設我們從 master 分出了兩條 branch ，分別是 feature01 跟 feature02。我如果在 feature02 這個分支輸入：
```git
git rebase feature01
```
意思就是「我 (feature02) 要重新定義 (re) 我的參考基準 (base)，那個參考基準就是 feature01 」。

以結果來說，feature02 的 commit 會直接接在 feature01 的後面，看起來很像剪下貼上，但對 Git 來說 feature02 的那些 commit 都是新的 commit ，可以從 {{< inlineCode >}}git log --oneline{{< /inlineCode >}}看到他們和原本的 SHA-1 值不一樣。

至於在 feature01 時 rebase feature02 ，和在 feature02 時 rebase feature01 的差別，只在最後 log 上 commit 的順序不同。

---

### git push
```git=
git push origin master
```
將分支 master 推上遠端數據庫 origin  
若有設定 remote，不帶參數執行 {{< inlineCode >}}git push{{< /inlineCode >}} 相當於 {{< inlineCode >}}git push <remote>{{< /inlineCode >}}  
若沒有設定 remote ，不帶參數執行 {{< inlineCode >}}git push{{< /inlineCode >}} 相當於 {{< inlineCode >}}git push origin{{< /inlineCode >}}

---
### git pull
```git=
git pull origin master
```
將遠端數據庫 origin 的 master 分支拉下並合併到本地數據庫  
若有設定 remote，不帶參數執行 {{< inlineCode >}}git pull{{< /inlineCode >}} 相當於 {{< inlineCode >}}git pull <remote>{{< /inlineCode >}}  
若沒有設定 remote ，不帶參數執行 {{< inlineCode >}}git pull{{< /inlineCode >}} 相當於 {{< inlineCode >}}git pull origin{{< /inlineCode >}}  

{{< inlineCode >}}git pull{{< /inlineCode >}} 做的事情實際上等於 {{< inlineCode >}}git fetch{{< /inlineCode >}} + {{< inlineCode >}}git merge{{< /inlineCode >}}，{{< inlineCode >}}git fetch{{< /inlineCode >}} 會比對本地與遠端的差別，並在本地形成分支，{{< inlineCode >}}git merge{{< /inlineCode >}} 則會將分支合併。

參考資料
---
1. [Git官方文件](https://git-scm.com/book/zh-tw/v2)
2. [為你自己學Git](https://gitbook.tw/)
3. [連猴子都能懂的Git入門指南](https://backlog.com/git-tutorial/tw/)
