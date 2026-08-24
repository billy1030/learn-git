# Level 01: 基礎核心與個人工作流 (Beginner Foundations)

本章介紹 Git 的核心思維模型（Mental Model）以及在個人本機環境追蹤檔案變更所需的基礎指令。

---

## 1. 核心思維模型：Git 的三大區域 (The 3 States of Git)

Git 並非單純拷貝檔案，而是在本機的三個工作區域之間流轉快照：

```
[ 工作目錄 Working Directory ]  --- (git add) --->  [ 暫存區 Staging Area / Index ]  --- (git commit) --->  [ 版本庫 Repository (.git) ]
   (日常實際編輯的檔案)                               (準備存檔的草稿清單)                                    (永久保存的歷史快照)
```

---

## 2. 初次設定個人身分 (Identity Setup)

```powershell
# 設定提交者姓名與電子郵件（會記錄在每一筆 commit 中）
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 設定預設初始分支名稱為 'master'（或 'main'）
git config --global init.defaultBranch master

# Windows 與 Linux 換行符號自動轉換 (CRLF vs LF)
git config --global core.autocrlf true
```

---

## 3. 建立或複製儲存庫 (Repository Setup)

```powershell
# 初始化 (Initialize)：在當前目錄建立全新的 Git 儲存庫
git init

# 複製 (Clone)：從遠端 URL 下載既有專案與完整歷史
git clone https://github.com/billy1030/mds.git
```

---

## 4. 日常循環操作指令速查 (The Daily Edit Loop Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 15%;">操作階段</th>
      <th style="width: 48%;">標準指令 (Command - 單行完整)</th>
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
      <td style="white-space: nowrap;"><code>git commit -m "feat: 完成登入表單"</code></td>
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

## 5. 忽略檔案清單 (`.gitignore`)

在專案根目錄建立 `.gitignore` 檔案，避免將密鑰、暫存快取或編譯產出誤加入版本庫：

```gitignore
# 依賴套件庫 (Dependencies)
node_modules/
venv/
__pycache__/

# 環境變數與機密金鑰 (Secrets & Credentials)
.env
.env.local
*.pem
*.key

# 建置輸出與日誌 (Build outputs & Logs)
dist/
build/
*.log

# 作業系統暫存檔 (OS temporary files)
.DS_Store
Thumbs.db
```

---

## 6. 個人工作流核心準則 (Best Practices)

1. **頻繁小步提交 (Commit Often)**：按邏輯拆分小提交，避免累積一週才做一次巨大的 Commit。
2. **語意化清晰備註 (Clear Messages)**：以動詞開頭（如 `feat: 新增搜尋過濾`、`fix: 修復日期計算`）。
3. **絕不提交機密 (Never Commit Secrets)**：務必在 `git status` 確認 `.env` 等敏感檔案已被忽略。
