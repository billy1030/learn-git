# Level 04: 團隊多人協作與 PR 工作流 (Team Collaboration & PR Workflows)

本章涵蓋現代團隊多人協作開發的完整生命週期。透過**「多人合寫一本書」**與**「真實辦公室場景」**的比喻，讓您直觀理解每一個 Git 指令背後的意義與用法。

---

## 1. 核心思維模型比喻：多人合寫一本書 (The Book Analogy)

把 Git 儲存庫想像成團隊正在合力編寫一本巨作：

- **`master` (主幹)**：出版社正在印製發行中的**「最新官方定稿本」**。
- **`Branch` (功能分支)**：你從官方定稿本**複印了一份章節草稿**，帶回自己座位上寫作。
- **`Rebase` (變基/墊高)**：當你在座位寫草稿時，隊友已經把第 5 章正式定稿了；Rebase 就是把你的草稿**移動抽換到隊友最新定稿的頁碼之後**，維持章節順序是一條直線。
- **`Merge Conflict` (衝突)**：你和隊友在同一行寫了不同的標題，主編不知道該採納誰的，需要你們坐下來**「人眼確認裁決」**。
- **`Pull Request / PR` (拉取請求)**：你把寫好的草稿裝訂成冊，**遞交給主編與隊友進行審閱 (Code Review)**。
- **`Squash & Merge` (壓合合併)**：把你草稿裡的「改錯字、喝咖啡、重寫」等 10 個零碎塗鴉紀錄**精煉成一段完美的正式章節**，蓋章併入官方定稿本。

---

## 2. 五大實戰情境與比喻對照表 (Real-World Scenarios)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 10%; text-align: center;">情境</th>
      <th style="width: 20%;">日常情境比喻 (Analogy)</th>
      <th style="width: 22%;">團隊協作策略 (Strategy)</th>
      <th style="width: 48%;">標準 CLI 指令 (CLI Command)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>場景 A</strong></td>
      <td>領取新任務，複印最新定稿帶回座位寫</td>
      <td>確保 master 最新，開闢標準命名分支</td>
      <td style="white-space: nowrap;"><code>git checkout -b feat/user-auth</code></td>
    </tr>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>場景 B</strong></td>
      <td>座位寫到一半隊友交卷，把草稿墊到最新頁碼後</td>
      <td>用 Rebase 同步最新進度，維持線性歷史</td>
      <td style="white-space: nowrap;"><code>git fetch origin &amp;&amp; git rebase origin/master</code></td>
    </tr>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>場景 C</strong></td>
      <td>發現兩人改了同一行文字，主編亮紅燈</td>
      <td>手動刪除衝突標記，保留正確句子繼續</td>
      <td style="white-space: nowrap;"><code>git add . &amp;&amp; git rebase --continue</code></td>
    </tr>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>場景 D</strong></td>
      <td>草稿寫完，送交主管審核審批</td>
      <td>發起 Pull Request 請求隊友 Review</td>
      <td style="white-space: nowrap;"><code>gh pr create --fill --base master</code></td>
    </tr>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>場景 E</strong></td>
      <td>審查過關，把草稿塗鴉精簡成一章正式發布</td>
      <td>Squash-Merge 壓成單一乾淨節點並刪除草稿</td>
      <td style="white-space: nowrap;"><code>gh pr merge 42 --squash --delete-branch</code></td>
    </tr>
  </tbody>
</table>

---

## 3. 團隊分支命名標準規範 (Branch Naming Conventions)

比喻：就像公文夾上的**彩色分類標籤**，讓任何人一眼看出公文內容：

- **`feat/MDS-101-google-drive-sync`**：【綠色標籤】新功能開發 (Feature)
- **`fix/MDS-204-token-refresh-error`**：【紅色標籤】線上問題或 Bug 修復
- **`refactor/optimize-db-pool`**：【藍色標籤】架構重構（內部結構整理，外部功能不變）
- **`perf/fast-table-render`**：【黃色標籤】效能調校優化
- **`docs/update-api-guide`**：【白色標籤】技術文檔更新
- **`chore/upgrade-deps`**：【灰色標籤】套件庫與建置流程升級

---

## 4. 完整端到端 CLI 實戰流程 (Step-by-Step Walkthrough)

### 步驟 1：確保本機主幹最新，並領取新任務分支
> **比喻**：先從書架拿下最新版本的官方定稿本，並複印一份帶回座位。
```powershell
# 切換回主幹並快進拉取最新內容
git checkout master
git pull --ff-only

# 開闢專屬任務分支
git checkout -b feat/google-drive-backup
```

---

### 步驟 2：日常開發與標準化語意提交 (Conventional Commits)
> **比喻**：在草稿紙上作答，每寫完一個段落就貼一張便利貼說明「這一段完成了什麼」。
```powershell
# 1. 檢查目前動了哪些檔案
git status

# 2. 挑選要存檔的檔案加入暫存
git add backend/src/services/google-drive.service.ts
git add backend/src/routes/backup.routes.ts

# 3. 寫下具體的語意化備註
git commit -m "feat(backup): 新增 Google Drive OAuth2 自動排程上傳功能"
```

---

### 步驟 3：推送至雲端備份與共享
> **比喻**：把座位上的草稿副本上傳一份到雲端櫃子，避免電腦當機或給隊友查看。
```powershell
git push -u origin feat/google-drive-backup
```

---

### 步驟 4：開發中同步隊友最新進度 (Rebase 變基)
> **比喻**：隊友小明剛把第 4 章定稿了。你把自己的草稿「撕下來，重新貼在小明第 4 章的後面」，保持整本書頁碼順序是一條直線。
```powershell
# 1. 下載雲端所有最新進度
git fetch origin

# 2. 把自己的分支墊高到最新 origin/master 之上
git rebase origin/master
```

---

### 步驟 5：化解代碼衝突詳解 (Conflict Resolution Deep Dive)

> [!IMPORTANT]
> **關鍵觀念（修改只發生在你的電腦！）**：
> 1. **不需要去碰隊友的檔案**：隊友的代碼已經透過 `git fetch` 下載到你的本機中了。
> 2. **只有單一檔案**：Git 直接把兩人的代碼合併寫進你眼前的「這一個本機檔案」裡，並用三條標記線隔開。
> 3. **隊友完全不受影響**：你在本機解衝突時，隊友的電腦和雲端倉庫完全不會報錯，等你修好推上去時就是乾淨無衝突的最終版本！

#### A. 衝突當下，檔案內部具體長什麼樣子？
假設衝突檔案為 `backend/src/config.ts`，當你在自己的編輯器打開它時會看到：

```typescript
export const APP_CONFIG = {
  appName: "MDS System",
  port: 3000,
<<<<<<< HEAD
  backupInterval: 3600, // 隊友（主幹 master 最新定稿）寫的值：每小時
=======
  backupInterval: 86400, // 你（當前功能分支）寫的值：每日
>>>>>>> feat/google-drive-backup
  maxFileSizeMB: 50
};
```

#### B. 三條標記線是什麼意思？
- **`<<<<<<< HEAD`**：上方段落代表目前目標分支（主幹 `master`）現有的代碼。
- **`=======`**：中央分隔線（楚河漢界）。
- **`>>>>>>> feat/google-drive-backup`**：下方段落代表您（功能分支）寫的代碼。

---

#### C. 具體怎麼修改？（三種常見抉擇範例）

在你的本機編輯器手動將不想要的行數與標記符號**整行刪除 (Backspace / Delete)** 並存檔（`Ctrl + S`）：

##### 抉擇 1：決定採用「你的版本」（86400）
將 `<<<<<<<`、隊友那行、`=======`、`>>>>>>>` 刪除，只留下你的代碼：
```typescript
export const APP_CONFIG = {
  appName: "MDS System",
  port: 3000,
  backupInterval: 86400, // 保留你的修改
  maxFileSizeMB: 50
};
```

##### 抉擇 2：決定採用「隊友的版本」（3600）
將你的那行和所有標記線刪除，只留下隊友代碼：
```typescript
export const APP_CONFIG = {
  appName: "MDS System",
  port: 3000,
  backupInterval: 3600, // 保留隊友的修改
  maxFileSizeMB: 50
};
```

##### 抉擇 3：兩人的代碼「都要保留」（融合兩者）
若兩人各加了不同的新屬性，可以全部留下：
```typescript
export const APP_CONFIG = {
  appName: "MDS System",
  port: 3000,
  backupInterval: 86400, // 你的修改
  notifyEmail: "admin@example.com", // 隊友新增的通知信箱
  maxFileSizeMB: 50
};
```

---

#### D. 修改完成後的 CLI 流程
在自己的編輯器按 `Ctrl + S` 存檔後，在終端機輸入：

```powershell
# 1. 將修復好的檔案加入暫存（標記為衝突已化解）
git add backend/src/config.ts

# 2. 繼續執行變基流程
git rebase --continue

# 3. 安全推送到雲端 (因為 Rebase 重新計算了基底 Hash，需使用 force-with-lease)
git push --force-with-lease
```

> [!TIP]
> **VS Code 視覺化快捷鍵**：若使用 VS Code 打開衝突檔案，標記上方會出現 `Accept Current Change`（採納隊友版）、`Accept Incoming Change`（採納你的版）、`Accept Both`（兩者皆留）三個按鈕，點擊即可一鍵自動刪除標記線！

---

### 步驟 6：發起代碼審查請求 (Pull Request / PR 深度解析)

#### A. 什麼是 Pull Request (PR)？
很多人會疑惑：「我明明是要把代碼**推上去 (Push)**，為什麼叫 **Request to Pull (請求拉取)**？」

> **超直觀比喻**：
> - **個人隨意玩 (無 PR)**：你直接衝進辦公室，把策劃書直接蓋上公司大印發布（若有錯，全公司系統一起崩潰 💥）。
> - **團隊正規做法 (有 PR)**：你把寫好的策劃書放在會議桌上，向主編與隊友發送一封邀請函說：
>   > *「報告主管與隊友，我寫好了！**請你們幫我過目審查 (Review)**，如果確認沒問題，**請幫我『拉進 (Pull)』公司的正式版本庫中！**」*

**這就是 Pull Request 的本質：一封「請大家幫我看代碼、確認沒問題就批准合併」的公開審查邀請函！**

---

#### B. 團隊為什麼必須使用 PR？（三大核心價值）
1. **防止出包 (Code Review)**：隊友在網頁上能逐行比對差異，找出潛在 Bug、資安漏洞或效能隱患。
2. **自動化驗證守門員 (CI/CD)**：GitHub 在發出 PR 時會自動執行單元測試與排版檢查，測試全綠燈才放行。
3. **留下討論與決策紀錄**：任何人在特定代碼下方留下的討論，未來都會成為專案寶貴的歷史知識庫。

---

#### C. 如何發起 PR？（兩種方式）

##### 方式 1：GitHub 網頁上一鍵發起（最常用）
1. 本機推送分支：`git push -u origin feat/google-drive-backup`
2. 打開瀏覽器進入 GitHub 專案首頁。
3. 頂部會自動跳出綠色按鈕：**`Compare & pull request`（比較並建立 PR）**。
4. 填寫標題與功能摘要，點擊 **`Create pull request`**。

##### 方式 2：使用 GitHub CLI 命令列一秒發起（極速）
```powershell
# 自動讀取最新 Commit 訊息一鍵建立 PR
gh pr create --fill --base master
```

---

### 步驟 7：審查通過後正式合併 (Squash & Merge)
> **比喻**：審查過關！把你在草稿上的 5 次零碎修改「壓平精煉成 1 個完美的正式章節」，合併進官方主幹，並把過期的草稿紙碎掉。

```powershell
# 1. 執行 Squash 合併並自動刪除遠端分支
gh pr merge --squash --delete-branch

# 2. 切換回主幹，拉取最新合併結果
git checkout master
git pull --ff-only

# 3. 刪除本機已不需要的草稿分支
git branch -d feat/google-drive-backup
```

---

## 5. 團隊協作常見 CLI 指令速查表 (Quick Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 24%;">需求動作</th>
      <th style="width: 48%;">指令 (Command)</th>
      <th style="width: 28%;">生活化寫書比喻</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>查看所有分支 (本機+雲端)</td>
      <td style="white-space: nowrap;"><code>git branch -a</code></td>
      <td>查看所有人的草稿夾清單</td>
    </tr>
    <tr>
      <td>檢視所有待審查 PR</td>
      <td style="white-space: nowrap;"><code>gh pr list</code></td>
      <td>看待簽公文匣有哪些文件</td>
    </tr>
    <tr>
      <td>在本地測試隊友的 PR 分支</td>
      <td style="white-space: nowrap;"><code>gh pr checkout 42</code></td>
      <td>借閱第 42 號草稿到自己桌上測試</td>
    </tr>
    <tr>
      <td>放棄當前進行中的 Rebase</td>
      <td style="white-space: nowrap;"><code>git rebase --abort</code></td>
      <td>遇到嚴重混亂，瞬間退回墊高前的原點</td>
    </tr>
    <tr>
      <td>檢視直線圖形化提交樹</td>
      <td style="white-space: nowrap;"><code>git log --oneline --graph --decorate -10</code></td>
      <td>檢查章節脈絡是否保持漂亮的單一線路</td>
    </tr>
  </tbody>
</table>
