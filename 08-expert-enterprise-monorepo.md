# Level 08: 企業級架構與大型 Monorepo (Enterprise & DevOps)

本章介紹衝突重用記錄 (`rerere`)、數 GB 超大型 Monorepo 稀疏檢出 (`sparse-checkout`)、離線封裝 (`bundle`)、Git Hooks 自動防護，以及資安金鑰抹除。

---

## 1. 核心思維模型：超商條碼掃描器與貨櫃船比喻 (Enterprise Analogies)

> **生活化比喻**：
> - **`git rerere`（肌肉記憶與重用錄音機 📼）**：全名為 *Reuse Recorded Resolution*。就像你每次遇到相同的代碼衝突，Git 自動在背後「錄音存檔」。下一次當你又遇到一模一樣的衝突時，Git **自動播放錄音帶一秒幫你解好**，不必重複手動改 10 次！
> - **`sparse-checkout`（只買一包洋芋片而不是搬空整座好市多 🛒）**：在 100GB 的超大型 Monorepo 中，前端工程師只需要改 `frontend/` 目錄。透過稀疏檢出，你不需要把整個後端與大數據倉庫下載下來，**只下載 50MB 的前端資料夾**即可極速開工。
> - **`git bundle`（無網路無人島的「時空膠囊隨身碟」💾）**：在嚴格斷網的極密國防或銀行機房環境中，無法連接 GitHub。`git bundle` 把整個專案與所有 Commit 歷史打包成單一檔案，帶進機房直接還原成完整 Git 庫！

---

## 2. 三大企業級高頻實戰範例 (3 Real-World Enterprise Scenarios)

### 場景 1：開啟衝突自動重用記憶 (`rerere`)
- **實戰配置**：
```powershell
# 1. 開啟全域衝突記憶功能
git config --global rerere.enabled true

# 2. 下次遇到相同分支衝突時，Git 自動輸出：
# 👉 "Resolved 'backend/src/config.ts' using previous resolution."
```

---

### 場景 2：100GB Monorepo 稀疏檢出 (Sparse Checkout)
- **實戰流程**：只拉取 `frontend/` 資料夾，下載時間從 30 分鐘縮短至 5 秒：
```powershell
# 1. 複製時不下載實體檔案 (Blobless Clone)
git clone --filter=blob:none --no-checkout https://github.com/org/huge-monorepo.git
cd huge-monorepo

# 2. 初始化並指定只檢出前端目錄
git sparse-checkout init --cone
git sparse-checkout set frontend/
git checkout master
```

---

### 場景 3：斷網機房離線備份與遷移 (`git bundle`)
- **實戰流程**：
```powershell
# 1. 在有網路的電腦上打包整個 master 分支歷史為單一檔案
git bundle create project-backup.bundle master

# 2. 將 .bundle 檔案透過隨身碟拷貝至斷網機房電腦，直接還原：
git clone project-backup.bundle local-repo
```

---

## 3. 企業級進階技術指令總表 (Enterprise Command Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 16%;">技術領域</th>
      <th style="width: 52%;">標準指令 (Command)</th>
      <th style="width: 32%;">中文作用說明與效益</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 衝突自動重用</strong></td>
      <td style="white-space: nowrap;"><code>git config --global rerere.enabled true</code></td>
      <td>自動記憶衝突解法，下次遇到相同衝突秒速自動化解。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 稀疏檢出初始化</strong></td>
      <td style="white-space: nowrap;"><code>git clone --depth 1 --filter=blob:none --no-checkout &lt;url&gt;</code></td>
      <td>複製超大型 Monorepo 時不下載實體檔案內容。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 設定檢出子目錄</strong></td>
      <td style="white-space: nowrap;"><code>git sparse-checkout init --cone &amp;&amp; git sparse-checkout set frontend/</code></td>
      <td>指定只下載並檢出特定子目錄（10GB 專案秒變 50MB）。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 離線封裝導出</strong></td>
      <td style="white-space: nowrap;"><code>git bundle create repo-backup.bundle master</code></td>
      <td>將整個分支與歷史打包成單一二進位檔案以供離線傳輸。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 離線封裝還原</strong></td>
      <td style="white-space: nowrap;"><code>git clone repo-backup.bundle new-repo</code></td>
      <td>在無網路環境直接從 bundle 檔案還原出完整版本庫。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>6. 徹底抹除敏感檔案</strong></td>
      <td style="white-space: nowrap;"><code>git filter-repo --path sensitive-credentials.env --invert-paths</code></td>
      <td>從所有歷史 Commit 與分支中徹底抹除指定洩漏檔案。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>7. SSH 提交簽名</strong></td>
      <td style="white-space: nowrap;"><code>git config --global commit.gpgsign true</code></td>
      <td>對每次 Commit 進行數位簽名，獲得 GitHub Verified 標章。</td>
    </tr>
  </tbody>
</table>

---

## 4. Git 自動防護鉤子 (Git Hooks & Pre-commit)

存放於 `.git/hooks/` 中的腳本，可在提交或推送前自動強制執行品質檢查：

### 範例：`.git/hooks/pre-commit` (防止誤將密鑰提交)
```bash
#!/bin/sh
# 檢查是否有副檔名為 .env、.key、.pem 的機密檔案被加入暫存
if git diff --cached --name-only | grep -E '\.(env|key|pem)$'; then
    echo "【錯誤】：禁止將敏感金鑰檔案 (.env / .key) 提交進版本庫！"
    exit 1
fi

# 自動執行代碼排版與檢查
npm run lint
```
