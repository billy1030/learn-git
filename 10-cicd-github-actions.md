# Level 10: CI/CD 自動化與 GitHub Actions 實戰 (Automated Testing & Pipelines)

本章介紹如何在代碼推送到 GitHub 或發起 PR 時，由雲端伺服器自動接手執行單元測試、代碼風格檢查與自動化打包上線。

---

## 1. 核心思維模型：什麼是 GitHub Actions？ (The Automated Factory Analogy)

> **寫書與印刷廠比喻**：
> - **傳統手動時代**：你寫完新章節，必須自己一字一句肉眼檢查拼字、手動確認排版尺寸、自己抱著稿子跑到印刷廠送件。只要哪天你精神不好漏看了，錯字就直接印成十萬本書發行。
> - **GitHub Actions 自動化時代**：你在專案中安插了一座「24 小時無人自動化品檢工廠」。
>   1. **觸發器 (Event / Trigger)**：只要你一把草稿推上雲端（Push / PR）。
>   2. **自動開機 (Runner)**：GitHub 瞬間在雲端免費開一台全新乾淨的 Linux 虛擬機。
>   3. **流水線作業 (Jobs & Steps)**：自動安裝 Node.js / Python、自動跑 `npm test` 抓錯字、自動編譯打包。
>   4. **回報結果**：全部通過就亮綠燈 ✅ 放行 PR，有任何錯誤立刻發警報 ❌ 阻止合併。

---

## 2. GitHub Actions 核心四大元素架構

```
.github/workflows/
└── ci.yml (自動化工作流設定檔)
     ├── 1. Trigger (觸發時機: push 到 master 或發起 PR)
     └── 2. Job (任務: 執行在 ubuntu-latest 雲端機器上)
          ├── Step 1: 下載代碼 (actions/checkout)
          ├── Step 2: 設定環境 (actions/setup-node)
          ├── Step 3: 安裝依賴 (npm install)
          └── Step 4: 執行測試 (npm test)
```

---

## 3. 實戰建立第一份自動化流水線 (`.github/workflows/ci.yml`)

只要在專案根目錄建立這個檔案，推上 GitHub 後自動化立刻生效：

```yaml
name: Continuous Integration (CI 核心品檢流水線)

# 觸發條件：當有人推送代碼到 master，或對 master 發起 PR 時觸發
on:
  push:
    branches: [ master, main ]
  pull_request:
    branches: [ master, main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest # 在 GitHub 免費提供的最新 Ubuntu 雲端機器上執行

    steps:
      # 步驟 1：下載專案儲存庫代碼
      - name: 檢出專案代碼 (Checkout repository)
        uses: actions/checkout@v4

      # 步驟 2：設定 Node.js 執行環境
      - name: 設定 Node.js 環境 (Setup Node.js)
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      # 步驟 3：安裝依賴套件
      - name: 安裝專案依賴 (Install Dependencies)
        run: npm ci

      # 步驟 4：執行代碼排版與語法檢查
      - name: 代碼檢查 (Run Linter)
        run: npm run lint --if-present

      # 步驟 5：執行單元測試
      - name: 執行自動化測試 (Run Unit Tests)
        run: npm test --if-present

      # 步驟 6：測試專案是否能成功編譯打包
      - name: 測試建置打包 (Build Project)
        run: npm run build --if-present
```

---

## 4. 透過 GitHub CLI (`gh`) 監控與管理流水線

不用打開瀏覽器，直接在終端機監控雲端自動化建置：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 25%;">需求動作</th>
      <th style="width: 48%;">GitHub CLI 指令 (Command)</th>
      <th style="width: 27%;">說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>檢視最近運行的流水線清單</td>
      <td style="white-space: nowrap;"><code>gh run list</code></td>
      <td>查看綠燈 ✅ 或紅燈 ❌ 狀態</td>
    </tr>
    <tr>
      <td>即時監控當前正在跑的建置</td>
      <td style="white-space: nowrap;"><code>gh run watch</code></td>
      <td>終端機即時捲動日誌輸出</td>
    </tr>
    <tr>
      <td>手動觸發特定工作流</td>
      <td style="white-space: nowrap;"><code>gh workflow run ci.yml</code></td>
      <td>立即在雲端手動啟動品檢</td>
    </tr>
    <tr>
      <td>檢視出錯的詳細日誌</td>
      <td style="white-space: nowrap;"><code>gh run view &lt;run-id&gt; --log-failed</code></td>
      <td>直接抓出失敗的報錯行數</td>
    </tr>
  </tbody>
</table>
