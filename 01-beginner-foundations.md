# Level 01: 基礎核心與個人工作流 (Beginner Foundations)

本章介紹 Git 的核心思維模型（Mental Model）、提交前置準備 (Pre-flight Setup)、Commit ID 數位指紋原理、業界標準語意化規範 (Conventional Commits)、個人本機操作全景，以及 `.gitignore` 深度防護規則與快取陷阱解藥。

---

## 1. 核心思維模型：Git 的三大區域 (The 3 States of Git)

Git 並非單純拷貝檔案，而是在本機的三個工作區域之間流轉快照：

```
[ 工作目錄 Working Directory ]  --- (git add) --->  [ 暫存區 Staging Area / Index ]  --- (git commit) --->  [ 版本庫 Repository (.git) ]
   (日常實際編輯的檔案)                               (準備存檔的草稿清單)                                    (永久保存的歷史快照)
```

---

## 2. 什麼是 Commit ID (Hash)？怎麼來的？（數位指紋身分證 🪪）

> **寫書比喻**：
> 當你在某個章節蓋章存檔時，出版社的機器會把：**「你改動的所有文字 + 你的姓名 Email + 蓋章時間戳 + 前一頁的印章編號」** 全部丟進一台不可逆的數學碎紙機（SHA-1 / SHA-256 演算法），瞬間吐出一串 **40 位的唯一十六進位數位指紋**（例如 `a1b2c3d4e5f6...`）。

### 為什麼日常只用「前 7 碼 (Short SHA)」？
在全世界數十億筆提交中，前 7 碼（例如 `a1b2c3d`）碰撞重複的機率已經低於數億分之一，因此在所有 CLI 指令（如 `git checkout a1b2c3d` 或 `git cherry-pick a1b2c3d`）中，**只需要輸入前 7 碼即可精準識別**！

---

## 3. 標準化語意提交規範 (Conventional Commits 1.0)

團隊中最忌諱的 Commit 訊息是：`update`、`fix bug`、`111`、`aaa`。
業界統一採用 **Conventional Commits** 結構：

```text
<type>(<scope>): <subject>

[optional body - 詳細修改原因與背景]
[optional footer - 關聯的 Issue 或 Breaking Change]
```

### 推薦的 8 大 Type 動詞字典：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 15%;">前綴 (Type)</th>
      <th style="width: 18%;">中文分類</th>
      <th style="width: 45%;">標準範例 (Example)</th>
      <th style="width: 22%;">適用情境</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong><code>feat</code></strong></td>
      <td><strong>新功能</strong></td>
      <td style="white-space: nowrap;"><code>feat(auth): 新增 Google OAuth2 登入按鈕</code></td>
      <td>使用者可感知的新功能</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>fix</code></strong></td>
      <td><strong>問題修復</strong></td>
      <td style="white-space: nowrap;"><code>fix(billing): 修復閏年二月利息計算錯誤</code></td>
      <td>修復生產或測試中的 Bug</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>docs</code></strong></td>
      <td><strong>文件更新</strong></td>
      <td style="white-space: nowrap;"><code>docs(api): 補充會員註冊 API 參數說明</code></td>
      <td>純 Markdown、註解、文件</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>refactor</code></strong></td>
      <td><strong>架構重構</strong></td>
      <td style="white-space: nowrap;"><code>refactor(db): 重構資料庫連線池為單例模式</code></td>
      <td>不影響外部功能的代碼整理</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>perf</code></strong></td>
      <td><strong>效能優化</strong></td>
      <td style="white-space: nowrap;"><code>perf(table): 虛擬滾動降低 DOM 渲染耗時</code></td>
      <td>提升執行速度或降低記憶體</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>test</code></strong></td>
      <td><strong>測試補充</strong></td>
      <td style="white-space: nowrap;"><code>test(order): 新增購物車結帳單元測試</code></td>
      <td>新增或修改自動化測試</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>chore</code></strong></td>
      <td><strong>雜項建置</strong></td>
      <td style="white-space: nowrap;"><code>chore(deps): 升級 TypeScript 至 5.4 版本</code></td>
      <td>更新依賴套件、修改建置流程</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>ci</code></strong></td>
      <td><strong>流水線</strong></td>
      <td style="white-space: nowrap;"><code>ci(actions): 新增自動發布 Docker 映像檔</code></td>
      <td>修改 GitHub Actions 設定</td>
    </tr>
  </tbody>
</table>

---

## 4. 提交前必備「起飛檢查清單 (Pre-flight Checklist)」

在按下 `git commit` 或 `git push` 前，花 5 秒鐘在腦海中跑一遍：
- [ ] **1. `git status` 檢查**：確認沒有把 `.env`、敏感金鑰或暫存垃圾檔加入。
- [ ] **2. `git diff` 檢查**：確認沒有留下測試用的 `console.log`、`print()` 或硬編碼密碼。
- [ ] **3. 單元測試**：確認本地測試全部跑通。
- [ ] **4. 語意化命名**：確認以 `feat:`、`fix:` 等動詞開頭，訊息明確。

---

## 5. 初次設定個人身分與提交範本 (Identity Setup)

```powershell
# 設定提交者姓名與電子郵件（會記錄在每一筆 commit 中）
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 設定預設初始分支名稱為 'master'（或 'main'）
git config --global init.defaultBranch master

# Windows 與 Linux 換行符號自動轉換 (CRLF vs LF)
git config --global core.autocrlf true

# (選用) 設定全域 Commit 訊息範本
git config --global commit.template ~/.gitmessage
```

---

## 6. 日常循環操作指令速查 (The Daily Edit Loop Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 15%;">操作階段</th>
      <th style="width: 48%;">標準指令 (Command)</th>
      <th style="width: 37%;">中文作用說明與比喻</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 檢查狀態</strong></td>
      <td style="white-space: nowrap;"><code>git status</code></td>
      <td>查看有哪些檔案被新增、修改或已加入暫存。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 檔案暫存</strong></td>
      <td style="white-space: nowrap;"><code>git add .</code></td>
      <td>將當前目錄下所有修改夾進準備存檔的資料夾。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 提交快照</strong></td>
      <td style="white-space: nowrap;"><code>git commit -m "feat(auth): 完成登入表單"</code></td>
      <td>將暫存區的內容永久蓋章寫入版本歷史。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 差異比對</strong></td>
      <td style="white-space: nowrap;"><code>git diff</code></td>
      <td>逐行檢視工作目錄中尚未暫存的具體修改。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 暫存比對</strong></td>
      <td style="white-space: nowrap;"><code>git diff --staged</code></td>
      <td>檢視已經加入暫存區、準備提交的修改行數。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>6. 歷史日誌</strong></td>
      <td style="white-space: nowrap;"><code>git log --oneline --graph -n 10</code></td>
      <td>以單行精簡與圖形化線路檢視最近 10 筆提交。</td>
    </tr>
  </tbody>
</table>

---

## 7. 忽略檔案防護罩 (`.gitignore` 深度指南)

### A. 核心思維模型：出境海關黑名單
> - **沒有 `.gitignore`**：Git 就像過度熱心的搬家工人，把你家客廳的垃圾、甚至你皮夾裡的密碼（`.env`）通通裝箱打包推上雲端 💥。
> - **有 `.gitignore`**：在門口貼上**「海關黑名單清單」**，嚴格命令 Git：「這些目錄與檔案一律當作看不見，絕對不准打包！」

---

### B. 萬用 `.gitignore` 語法規則表

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 24%;">語法格式</th>
      <th style="width: 38%;">範例</th>
      <th style="width: 38%;">具體忽略效果</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>副檔名通配符 (<code>*</code>)</strong></td>
      <td style="white-space: nowrap;"><code>*.log</code></td>
      <td>忽略所有以 <code>.log</code> 結尾的檔案</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>特定資料夾 (<code>/</code>)</strong></td>
      <td style="white-space: nowrap;"><code>node_modules/</code></td>
      <td>忽略專案中任何層級名為 <code>node_modules</code> 的目錄</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>根目錄限定 (<code>/</code> 開頭)</strong></td>
      <td style="white-space: nowrap;"><code>/config.json</code></td>
      <td>只忽略根目錄的 <code>config.json</code>，不影響子目錄</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>任意多層目錄 (<code>**</code>)</strong></td>
      <td style="white-space: nowrap;"><code>**/temp/*.tmp</code></td>
      <td>忽略任何層級 <code>temp</code> 資料夾下的 <code>.tmp</code> 檔案</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>例外反向保留 (<code>!</code>)</strong></td>
      <td style="white-space: nowrap;"><code>!important.log</code></td>
      <td>即使上面忽略了 <code>*.log</code>，唯獨此檔案<strong>不忽略</strong></td>
    </tr>
  </tbody>
</table>

---

### C. 為什麼加了 `.gitignore` 卻無效？（快取陷阱與後悔解藥）
如果某個檔案在寫入 `.gitignore` 之前就已經被 `git add` 追蹤過，Git 就會**永久記住它**，導致 `.gitignore` 失效。

```powershell
# 1. 移除所有快取追蹤（不會刪除本機實體檔案！）
git rm -r --cached .

# 2. 重新讀取最新的 .gitignore 並暫存
git add .

# 3. 提交生效
git commit -m "chore: refresh .gitignore cache"
```

---

### D. 現代全端標準萬用範本 (`.gitignore`)

```gitignore
# 1. 環境變數與安全金鑰 (絕對禁止上傳！)
.env
.env.local
.env.*.local
*.pem
*.key
*.cert
credentials.json

# 2. 相依套件庫 (Dependencies)
node_modules/
vendor/
venv/
__pycache__/
*.pyc

# 3. 建置輸出與暫存檔 (Build & Cache)
dist/
build/
out/
.next/
.cache/
*.tsbuildinfo

# 4. 日誌檔案 (Logs)
*.log
npm-debug.log*

# 5. 作業系統與 IDE 暫存
.DS_Store
Thumbs.db
.vscode/*
!.vscode/settings.json
!.vscode/extensions.json
.idea/
```
