# Git 基本操作

## 初始設定

```bash
git config --global user.name "Jason Wu"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

## 建立 / 取得 repo

```bash
git init                      # 在現有目錄建立 repo
git clone <url>               # 複製遠端 repo
```

## 日常工作流程

```bash
git status                    # 查看工作區狀態
git add <file>                # 將檔案加入 staging area
git add -p                    # 互動式選擇 hunk
git commit -m "message"       # 提交 staging area
git commit -am "message"      # 僅對已追蹤檔案 add + commit
```

## 檢視紀錄與差異

```bash
git log --oneline --graph --all
git diff                      # working tree vs. staging
git diff --staged             # staging vs. HEAD
git show <commit>
git ls-tree --name-only origin/main # main下檔案跟資料夾名稱
```

## 分支

```bash
git branch                    # 列出本地分支
git switch -c feature/x       # 建立並切換分支
git switch main
git merge feature/x           # 合併至目前分支
git branch -d feature/x       # 刪除已合併分支
```

## 遠端

```bash
git remote -v
git remote add origin <url>
git fetch                     # 僅下載，不合併
git pull                      # fetch + merge（--rebase 可改用 rebase）
git push -u origin main       # 首次推送並設定 upstream
git push
```

## 復原

```bash
git restore <file>            # 捨棄 working tree 的修改
git restore --staged <file>   # 從 staging 移除
git commit --amend            # 修改最近一次 commit
git revert <commit>           # 建立反向 commit（安全，適用於已推送）
git reset --soft HEAD~1       # 撤銷 commit，保留 staging
git reset --hard HEAD~1       # 撤銷 commit 並丟棄修改（不可逆）
```

## 暫存

```bash
git stash                     # 暫存未提交修改
git stash pop                 # 還原並移除 stash
git stash list
```

## .gitignore

```
node_modules/
.env
__pycache__/
*.log
```

已被追蹤的檔案需先 `git rm --cached <file>` 才會被忽略。

## 衝突處理

`merge` / `pull` 出現衝突時，檔案內會有 `<<<<<<<`、`=======`、`>>>>>>>` 標記。手動編輯後：

```bash
git add <file>
git commit                    # 完成 merge（rebase 則用 git rebase --continue）
```

需要更深入的部分（rebase、cherry-pick、reflog、GitHub Pages 部署等）可以指定。