# Level 18: 機密管理與安全自動化防禦 (GitHub Secrets, Dependabot & Push Protection)

本章介紹如何在 CI/CD 流水線中安全使用 API Key 與資料庫密碼 (Repository Secrets)、如何使用 Dependabot 自動修復漏洞依賴，以及開啟 Push Protection 阻擋密鑰外洩。

---

## 1. 核心思維模型：金庫保險箱與 24H 防盜警報 (The Security Vault Analogy)

> **寫書比喻**：
> - **硬編碼密鑰（災難做法）**：你把出版社的金庫密碼直接用原子筆寫在小說封面寄出去。全世界拿到書的人都能走進金庫搬錢。
> - **GitHub Secrets（加密保險箱 🔐）**：密碼鎖在 GitHub 雲端保險箱。Actions 執行時只借給它「看不到明文的代幣」，且終端機日誌中自動全部打碼成 `***`。
> - **Push Protection（24 小時防盜警報器 🚨）**：只要工程師手滑 Commit 了含 AWS 金鑰的檔案並按 `git push`，GitHub 在接收到封包的瞬間**直接強制攔截拒絕上傳**！

---

## 2. GitHub Secrets 安全管理實戰

在 GitHub 儲存庫設定環境變數（**Settings ➔ Secrets and variables ➔ Actions**）：

```yaml
# 在 .github/workflows/deploy.yml 中安全呼叫：
steps:
  - name: 部署至生產環境伺服器
    env:
      DATABASE_URL: ${{ secrets.PROD_DB_URL }}
      API_KEY: ${{ secrets.STRIPE_SECRET_KEY }}
    run: |
      npm run deploy
```

---

## 3. 自動化安全漏洞修復 (`.github/dependabot.yml`)

在專案中加入 Dependabot 設定檔，每週自動掃描並發 PR 升級有安全漏洞的套件：

```yaml
version: 2
updates:
  # 每週一自動檢查 npm 套件是否有安全更新
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 5
```

---

## 4. 安全管理 GitHub CLI 指令速查表

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 22%;">操作需求</th>
      <th style="width: 52%;">GitHub CLI 指令 (Command)</th>
      <th style="width: 26%;">說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 設定加密 Secret</strong></td>
      <td style="white-space: nowrap;"><code>gh secret set PROD_API_KEY --body "sk_live_123456"</code></td>
      <td>將機密金鑰加密存入儲存庫</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 列出已設定之 Secret</strong></td>
      <td style="white-space: nowrap;"><code>gh secret list</code></td>
      <td>檢視金鑰名稱清單（不顯示明文）</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 刪除 Secret</strong></td>
      <td style="white-space: nowrap;"><code>gh secret delete PROD_API_KEY</code></td>
      <td>立即抹除該加密變數</td>
    </tr>
  </tbody>
</table>
