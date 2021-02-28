---
title: "Git 常見問題"
description: "你遇到這樣的問題了嗎？"
summary: "Git 到底怎麼用？怎麼修改剛剛的 Commit？想重來要怎麼做？Git 踩坑及用法紀錄"
featured_image: "/images/git.png"
images: ["/images/git.png"]
tags: [學習筆記,版本控制]
date: 2020-06-14T20:47:43+08:00
toc: true
disable_share: true
---

## 正確的使用姿勢？
{{< inlineCode >}}git init{{< /inlineCode >}} 或是 {{< inlineCode >}}git clone{{< /inlineCode >}} 之後，你可能會手癢先 {{< inlineCode >}}git status{{< /inlineCode >}} 看一下，你會看到一個乾淨的目錄，然後開始編寫程式碼，一邊寫一邊 {{< inlineCode >}}git diff{{< /inlineCode >}} 看自己到底改了哪些東西，改了一個段落在 {{< inlineCode >}}git add{{< /inlineCode >}} 之前或許會 {{< inlineCode >}}git branch{{< /inlineCode >}} 確定一下自己在哪個分支，有錯的話趕快 {{< inlineCode >}}git checkout branchName{{< /inlineCode >}} 到正確的分支，接著 {{< inlineCode >}}git add{{< /inlineCode >}} 把修改存到暫存區，{{< inlineCode >}}git status{{< /inlineCode >}} 看一下是不是正確的狀態，然後 {{< inlineCode >}}git commit -m "This is my new git description"{{< /inlineCode >}}，最後 {{< inlineCode >}}git push{{< /inlineCode >}} 。

---
## 這行誰寫的！
一覺醒來發現網站掛了，怎麼多了這一行扣！{{< inlineCode >}}git blame index.php{{< /inlineCode >}} 可以看到 index.php 裡每一行的作者是誰。

---
## 這個 commit 到底修改了啥？
先 {{< inlineCode >}}git log{{< /inlineCode >}} 找到那次修改的 commit ID，你或許會用到 
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
最後{{< inlineCode >}}git show commitID{{< /inlineCode >}}，就可以看到修改記錄了。

---
## 我想修改剛剛的 commit
想修改最後一次的 commit，可以輸入
```git=
git commit --amend -m "This is right description"
```
對人類來說只是修改 commit 的訊息，但對 git 來說，這其實是一個全新的 commit 了，可以從 {{< inlineCode >}}git log --oneline{{< /inlineCode >}} 中看到 commit 的 SHA-1 值是不同的。

---
## 我可以修改更之前的 commit 嗎？
這通常發生在自己開的 branch 裡 commit 雜亂無章的情況，想做好修剪之後再 push。盡量不要在已經 push 到中央伺服器的 commit 進行這種操作，除非你和你的團隊非常清楚自己在做什麼，這會造成協作者的混亂，因為同樣的程式碼有不同的版本，而且它會改變歷史 (這也是 rebase 屬於危險指令的原因)。

Git 本身並沒有修改歷史的工具。但可以使用 rebase 來做到對過去 commit 的修改。
<br>
```git=
git rebase -i
```
可以開啟對話模式，要給他一個參數，這個參數可以是 {{< inlineCode >}}git rebase -i HEAD~3{{< /inlineCode >}} 這樣從當前分支往前算的形式，也可以是 {{< inlineCode >}}git rebase -i 6ee9b6b{{< /inlineCode >}} 這樣的 SHA-1 值，意思是「從這一個 commit 往後算到現在位子的所有 commit 」。

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
存檔離開後， git 會 checkout 到你想修改的 commit ，當你進行完你想做的修改之後， {{< inlineCode >}}git add [你有修改的檔案]{{< /inlineCode >}} ，之後再 {{< inlineCode >}}git commit --amend {{< /inlineCode >}} 修改 commit message ，最後再輸入 {{< inlineCode >}}git rebase --continue{{< /inlineCode >}} 將 rebase 完成。

另外的 squash 和 fixup 則是用來合併 commit，不同的地方是當你用 squash 要合併時，會把多個要合併的 commit message 放進編輯器裡面讓你修改最後合併的 commit message ；而 fixup 則是把這個 commit 合併到上一個 commit 中，並直接捨棄這個 commit message。

---
## 挫賽！我可以重來嗎？
git 的強大之處在於他大部分的指令都是可逆的，但它同樣也是危險的，因為並不是所有操作都可逆，一不小心就會搞得整個環境亂七八糟，在實戰時執行指令前都要謹慎思考，清楚知道自己在幹嘛，強烈建議可以搭配 Github 開一個自己來玩玩看。

這一 part 會先介紹如何反悔，再說明 reset 這個指令的意義，建議看完再操作。

### 如何將暫存區的檔案移出
這樣的場景會出現在你修改了兩個檔案，想要分別提交，但你手速太快不小心打了 {{< inlineCode >}}git add -A{{< /inlineCode >}} 全部加到暫存區了。

```git=
git reset HEAD <filename>
```

### 如何將剛剛的 commit 拆回來
這樣的場景則出現在後悔剛剛那個 commit 的內容，想拆回來重做，你可以先下 {{< inlineCode >}}git log{{< /inlineCode >}} 或 {{< inlineCode >}}git log --oneline{{< /inlineCode >}} 來看到 SHA-1 值，然後：
```git=
git reset 6ee9b6b
```
也可以用相對的方式：
```git=
git reset HEAD^
```

### 哇塞我整個專案都玩壞了，怎麼回復成原本的樣子
```git=
git reset --hard HEAD~2
```
這行指令的意思是最後的兩個版本(HEAD、HEAD^)我都不要了，我也不想讓人看見他，專案會直接回到 HEAD^^ 的狀態，所以在下指令前請確保你的 HEAD、HEAD^ 都是不要的。

### 我還想要 HEAD 或 HEAD^，能回復嗎？
其實還是救的回來，先下：
```git=
git reflog
```
reflog 裡面存的是「HEAD移動的紀錄」，你會看到你剛剛 reset 的紀錄，所以要取消 reset 的話，就要「reset 到 reset 前的那個 commit 」：
```git=
git reset --hard <commitID>
```
講到這裡聰明的你一定已經隱約感覺到 reset 的意義了，在中文會比較接近「前往」。

而 {{< inlineCode >}}--hard{{< /inlineCode >}} 是參數，有三種常見的： {{< inlineCode >}}--mixed{{< /inlineCode >}} 、 {{< inlineCode >}}--soft{{< /inlineCode >}} 、 {{< inlineCode >}}--hard{{< /inlineCode >}} ，這個參數決定的是「那些我不要的東西最後要去哪裡」
`{{< inlineCode >}}--mixed`{{< /inlineCode >}} 是預設，這個模式下的 reset commit 會把 commit 內容丟進工作目錄裡面
`{{< inlineCode >}}--soft`{{< /inlineCode >}} 則會把 commit 內容丟進暫存區
`{{< inlineCode >}}--hard`{{< /inlineCode >}} 則是工作目錄和暫存區都不會留

所以前文說的「拆」 commit 其實並不準確，而是「回到」那個 commit 的狀態，不同參數則決定了路過的那幾個檔案將何去何從。

參考資料
---
1. [Git官方文件](https://git-scm.com/book/zh-tw/v2)
2. [為你自己學Git](https://gitbook.tw/)
3. [連猴子都能懂的Git入門指南](https://backlog.com/git-tutorial/tw/)