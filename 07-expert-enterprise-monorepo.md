# Level 07: 企業級架構與大型 Monorepo (Enterprise & DevOps)

本章介紹衝突重用記錄 (`rerere`)、數 GB 超大型 Monorepo 稀疏檢出 (`sparse-checkout`)、離線封裝 (`bundle`)、Git Hooks 自動防護，以及資安金鑰抹除。

---

## 1. 企業級進階技術指令總表 (Enterprise Command Reference)

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

## 2. Git 自動防護鉤子 (Git Hooks & Pre-commit)

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
