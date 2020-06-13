---
title: "Git 簡單介紹與常用指令"
description: "Git 初心者大全，開著這篇用 Git 就對了！"
featured_image: "/images/git.png"
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

而只要將專案 clone 到自己的設備，你不需要連網，就可以在自己的版本庫中查看歷史紀錄(git log)、創建分支(git branch)、切換分支(git checkout)、甚至是提交(git commit)。也就是說假設你遇難隻身一人漂流到孤島，突然有一股修改專案的衝動，打開筆電就可以開始工作。

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
直接輸入 `git diff` 可以看到當前工作目錄和暫存區的差別。
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
`git rebase` 是一個很有趣的功能，也有其他不同功能的用法，這裡主要介紹它在合併上的使用，進階的使用請見下方「我可以修改以前的 commit 嗎？」。

base 是基準點，也就是分支從哪裡長出來，所以 rebase 的意思接近於「重新定義分支的參考基準」。

假設我們從 master 分出了兩條 branch ，分別是 feature01 跟 feature02。我如果在 feature02 這個分支輸入：
```git
git rebase feature01
```
意思就是「我 (feature02) 要重新定義 (re) 我的參考基準 (base)，那個參考基準就是 feature01 」。

以結果來說，feature02 的 commit 會直接接在 feature01 的後面，看起來很像剪下貼上，但對 Git 來說 feature02 的那些 commit 都是新的 commit ，可以從 `git log --oneline`看到他們和原本的 SHA-1 值不一樣。

至於在 feature01 時 rebase feature02 ，和在 feature02 時 rebase feature01 的差別，只在最後 log 上 commit 的順序不同。

---

### git push
```git=
git push origin master
```
將分支 master 推上遠端數據庫 origin
若有設定 remote，不帶參數執行 `git push` 相當於 `git push <remote>`
若沒有設定 remote ，不帶參數執行 `git push` 相當於 `git push origin`

---
### git pull
```git=
git pull origin master
```
將遠端數據庫 origin 的 master 分支拉下並合併到本地數據庫
若有設定 remote，不帶參數執行 `git pull` 相當於 `git pull <remote>`
若沒有設定 remote ，不帶參數執行 `git pull` 相當於 `git pull origin`

`git pull` 做的事情實際上等於 `git fetch` + `git merge`，`git fetch` 會比對本地與遠端的差別，並在本地形成分支，`git merge` 則會將分支合併。

---
你遇到這樣的問題了嗎？
---

### 正確的使用姿勢？
`git init` 或是 `git clone` 之後，你可能會手癢先 `git status` 看一下，你會看到一個乾淨的目錄，然後開始編寫程式碼，一邊寫一邊 `git diff` 看自己到底改了哪些東西，改了一個段落在 `git add` 之前或許會 `git branch` 確定一下自己在哪個分支，有錯的話趕快 `git checkout branchName` 到正確的分支，接著 `git add` 把修改存到暫存區，`git status` 看一下是不是正確的狀態，然後 `git commit -m "This is my new git description"`，最後 `git push` 。

---
### 這行誰寫的！
一覺醒來發現網站掛了，怎麼多了這一行扣！`git blame index.php` 可以看到 index.php 裡每一行的作者是誰。

---
### 這個 commit 到底修改了啥？
先 `git log` 找到那次修改的 commit ID，你或許會用到 
```git=
git log --author="Louis.Su"
```
來篩選出 commit 作者，或者是
```git=
git log --grep="Hey"
```
來找到 description 裡面包含 Hey 的 commit，又或者
```git=
git log -S "functionA"
```
來找到上傳檔案有包含 functionA 的 commit。
最後`git show commitID`，就可以看到修改記錄了。

---
### 我想修改剛剛的 commit
想修改最後一次的 commit，可以輸入
```git=
git commit --amend -m "This is right description"
```
對人類來說只是修改 commit 的訊息，但對 git 來說，這其實是一個全新的 commit 了，可以從 `git log --oneline` 中看到 commit 的 SHA-1 值是不同的。

---
### 我可以修改更之前的 commit 嗎？
這通常發生在自己開的 branch 裡 commit 雜亂無章的情況，想做好修剪之後再 push。盡量不要在已經 push 到中央伺服器的 commit 進行這種操作，除非你和你的團隊非常清楚自己在做什麼，這會造成協作者的混亂，因為同樣的程式碼有不同的版本，而且它會改變歷史 (這也是 rebase 屬於危險指令的原因)。

Git 本身並沒有修改歷史的工具。但可以使用 rebase 來做到對過去 commit 的修改。
<br>
```git=
git rebase -i
```
可以開啟對話模式，要給他一個參數，這個參數可以是 `git rebase -i HEAD~3` 這樣從當前分支往前算的形式，也可以是 `git rebase -i 6ee9b6b` 這樣的 SHA-1 值，意思是「從這一個 commit 往後算到現在位子的所有 commit 」。

之後會出現像這樣的列表：
```
pick f7f3f6d fix some bugs
pick 310154e add some files
pick a5f4a0d feat: complete a feature

# Rebase 710f0f8..a5f4a0d onto 710f0f8
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```
前幾行 pick 的意思是保留 commit 不做修改，如果你是想修改 commit message 的話，把 pick 改成 reword ：
```
reword f7f3f6d fix some bugs
pick 310154e add some files
pick a5f4a0d feat: complete a feature
```
表示我要修改這個 commit ，存檔離開後會再跳出另一個編輯器畫面，將 fix some bugs 改成你想要的 commit message 之後存檔離開， git 就會處理完剩下的事情。

如同上文中對 rebase 的描述，聰明如你肯定也猜到了，是的，這個 commit 同樣是全新的一個 commit ，而在這裡要注意的是，因為修改了過去的歷史，在這個 commit 之後的所有 commit 因為歷史的改變，所以整串 commit 都會變成新的 commit 。

因為過去被改變了，所以未來也會產生相應的變化，~~這就是平行時空的概念~~。

而 edit 和 reword 不同的地方在於： edit 同時還能修改 commit 內容，他會在 rebase 的過程中暫停等待你修改，所以當你：
```
edit f7f3f6d fix some bugs
pick 310154e add some files
pick a5f4a0d feat: complete a feature
```
存檔離開後， git 會 checkout 到你想修改的 commit ，當你進行完你想做的修改之後， `git add [你有修改的檔案]` ，之後再 `git commit --amend ` 修改 commit message ，最後再輸入 `git rebase --continue` 將 rebase 完成。

另外的 squash 和 fixup 則是用來合併 commit，不同的地方是當你用 squash 要合併時，會把多個要合併的 commit message 放進編輯器裡面讓你修改最後合併的 commit message ；而 fixup 則是把這個 commit 合併到上一個 commit 中，並直接捨棄這個 commit message。

---
### 挫賽！我可以重來嗎？
git 的強大之處在於他大部分的指令都是可逆的，但它同樣也是危險的，因為並不是所有操作都可逆，一不小心就會搞得整個環境亂七八糟，在實戰時執行指令前都要謹慎思考，清楚知道自己在幹嘛，強烈建議可以搭配 Github 開一個自己來玩玩看。

這一 part 會先介紹如何反悔，再說明 reset 這個指令的意義，建議看完再操作。

#### 如何將暫存區的檔案移出
這樣的場景會出現在你修改了兩個檔案，想要分別提交，但你手速太快不小心打了 `git add -A` 全部加到暫存區了。

```git=
git reset HEAD <filename>
```

#### 如何將剛剛的 commit 拆回來
這樣的場景則出現在後悔剛剛那個 commit 的內容，想拆回來重做，你可以先下 `git log` 或 `git log --oneline` 來看到 SHA-1 值，然後：
```git=
git reset 6ee9b6b
```
也可以用相對的方式：
```git=
git reset HEAD^
```

#### 哇塞我整個專案都玩壞了，怎麼回復成原本的樣子
```git=
git reset --hard HEAD~2
```
這行指令的意思是最後的兩個版本(HEAD、HEAD^)我都不要了，我也不想讓人看見他，專案會直接回到 HEAD^^ 的狀態，所以在下指令前請確保你的 HEAD、HEAD^ 都是不要的。

#### 我還想要 HEAD 或 HEAD^，能回復嗎？
其實還是救的回來，先下：
```git=
git reflog
```
reflog 裡面存的是「HEAD移動的紀錄」，你會看到你剛剛 reset 的紀錄，所以要取消 reset 的話，就要「reset 到 reset 前的那個 commit 」：
```git=
git reset --hard <commitID>
```
講到這裡聰明的你一定已經隱約感覺到 reset 的意義了，在中文會比較接近「前往」。

而 `--hard` 是參數，有三種常見的： `--mixed` 、 `--soft` 、 `--hard` ，這個參數決定的是「那些我不要的東西最後要去哪裡」
`--mixed` 是預設，這個模式下的 reset commit 會把 commit 內容丟進工作目錄裡面
`--soft` 則會把 commit 內容丟進暫存區
`--hard` 則是工作目錄和暫存區都不會留

所以前文說的「拆」 commit 其實並不準確，而是「回到」那個 commit 的狀態，不同參數則決定了路過的那幾個檔案將何去何從。

參考資料
---
Git官方文件：https://git-scm.com/book/zh-tw/v2
為你自己學Git：https://gitbook.tw/
連猴子都能懂的Git入門指南：https://backlog.com/git-tutorial/tw/
