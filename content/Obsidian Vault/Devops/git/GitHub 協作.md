主要有兩種模式，取決於你有沒有目標 repo 的寫入權限。

## 模式一：有寫入權限（同一個團隊/組織內）

1. **Clone**
    
    ```bash
    git clone https://github.com/<owner>/<repo>.gitcd <repo>
    ```
    
2. **開 feature branch**（不要直接改 main）
    
    ```bash
    git switch -c feature/xxx
    ```
    
3. **修改、commit**
    
    ```bash
    git add .git commit -m "feat: 描述這次改動"
    ```
    
4. **同步遠端最新的 main，避免衝突**
    
    ```bash
    git fetch origingit rebase origin/main    # 或 git merge origin/main
    ```
    
5. **push 分支**
    
    ```bash
    git push -u origin feature/xxx
    ```
    
6. **在 GitHub 開 Pull Request**（base: main ← compare: feature/xxx），指定 reviewer，等 review 通過後 merge。
7. merge 後清理：
    
    ```bash
    git switch main && git pullgit branch -d feature/xxx
    ```
    

## 模式二：沒有寫入權限（開源專案、外部 repo）— Fork 流程

1. 在 GitHub 頁面按 **Fork**，產生你自己的副本。
2. Clone 你的 fork，並加上原 repo 為 `upstream`：
    
    ```bash
    git clone https://github.com/<你的帳號>/<repo>.gitcd <repo>git remote add upstream https://github.com/<原作者>/<repo>.git
    ```
    
3. 同步上游：
    
    ```bash
    git fetch upstreamgit switch maingit merge upstream/main
    ```
    
4. 開分支、commit、push 到**你的 fork**（`origin`）。
5. 在 GitHub 從你的 fork 對原 repo 開 Pull Request。

## 共同要點

- **新增協作者**（模式一）：repo → Settings → Collaborators（個人 repo）或用 Organization 的 Teams 管理權限。
- **Branch protection**：Settings → Branches，可要求 PR review、通過 CI 才能 merge，並禁止直接 push main。
- **衝突處理**：rebase/merge 時出現衝突，編輯標記 `<<<<<<<` / `>>>>>>>` 的區塊，解決後：
    
    ```bash
    git add <檔案>git rebase --continue    # 或 git commit（merge 情況）
    ```
    
- **Commit 慣例**：小而聚焦的 commit；訊息可用 Conventional Commits（`feat:`、`fix:`、`docs:`）。
- **Issue**：用來記錄任務與 bug，PR 描述寫 `Closes #12` 可在 merge 時自動關閉對應 issue。
- **`.gitignore`**：避免提交 `node_modules/`、`.env`、金鑰等；已誤 push 的 secret 需視為外洩並更換。
- **認證**：HTTPS 需使用 Personal Access Token 或 `gh auth login`；也可設定 SSH key。

## 補充

- 若想在協作時保持線性歷史，用 `git rebase`；若團隊偏好保留完整分支紀錄，用 merge。已 push 且他人也在用的分支，不要 rebase 後 force push。
- 若要，我可以針對你的情境（例如課程小組專案、Quartz 網站、實驗室程式碼交接）給具體的分支策略與設定步驟。