# Level 16: 安全存取憑證、Token 與細粒度權限控制 (Tokens, PAT & Scopes)

本章介紹 GitHub 的現代安全身分驗證機制：個人存取權杖 (Personal Access Token, PAT)、權限範圍 (Scopes)、經典權杖與細粒度權杖的差異，以及如何透過 GitHub CLI (`gh auth`) 安全管理金鑰。

---

## 1. 核心思維模型：飯店電子感應房卡比喻 (Hotel Keycard Analogy)

自 2021 年起，GitHub 已全面廢除「使用帳號密碼進行 `git push`」，必須改用 Token 或 SSH Key：

> **生活化比喻**：
> - **帳號密碼（萬能大鑰匙 🗝️）**：一旦洩漏，別人就能登入你的帳號、刪除所有專案庫、盜刷綁定的信用卡，災難不可挽回。
> - **GitHub Token / PAT（電子感應房卡 💳）**：
>   1. **精準存取 (Scopes)**：可限制這張房卡「只能開 302 號房門（只存取特定 Repo）」，且「只能看電視（唯讀），不能動保險箱（不能刪庫）」。
>   2. **自動過期 (Expiration)**：可設定 30 天或 90 天後自動失效。
>   3. **一秒掛失 (Instant Revoke)**：一旦發現 Token 意外流出，至後台點擊 Delete，該房卡立即作廢，原本的帳號依然 100% 安全！

---

## 2. 兩種 Token 類型比較：經典版 vs 細粒度版 (Classic vs Fine-Grained)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 18%;">比較維度</th>
      <th style="width: 41%;">經典權杖 (Tokens Classic - <code>ghp_...</code>)</th>
      <th style="width: 41%;">細粒度權杖 (Fine-Grained - <code>github_pat_...</code>)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>專案庫範圍</strong></td>
      <td><strong>全部存取</strong>（帳號下所有公開與私有專案皆可碰）</td>
      <td><strong>精準指定</strong>（可限定「只允許存取 <code>learn-git</code> 專案」）</td>
    </tr>
    <tr>
      <td><strong>權限控制</strong></td>
      <td>粗粒度勾選（例如勾了 <code>repo</code> 就代表讀寫刪全部開放）</td>
      <td><strong>細粒度拆分</strong>（Contents 讀取/寫入、Issues 讀取、Secrets 禁止）</td>
    </tr>
    <tr>
      <td><strong>過期時間限制</strong></td>
      <td>允許設定「永不過期 (No expiration)」（不推薦）</td>
      <td><strong>強制必須設定過期日</strong>（最多 1 年，預設 30~90 天）</td>
    </tr>
    <tr>
      <td><strong>安全評級</strong></td>
      <td>⭐⭐⭐（適合個人本機 CLI 腳本）</td>
      <td>⭐⭐⭐⭐⭐（<strong>業界推薦標準</strong>，適合 CI/CD 與外包開發）</td>
    </tr>
  </tbody>
</table>

---

## 3. 常用權限範圍字典 (Common Token Scopes Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 22%;">權限名稱 (Scope)</th>
      <th style="width: 48%;">允許動作</th>
      <th style="width: 30%;">生活化權限說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong><code>repo</code></strong></td>
      <td>完整控制私有與公開專案庫的代碼、Commit、分支、PR</td>
      <td>核心開發者必勾（寫代碼與 Push）</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>read:org</code></strong></td>
      <td>讀取組織架構、團隊名單與個人資料</td>
      <td>查看團隊與邀請成員</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>workflow</code></strong></td>
      <td>更新 GitHub Actions CI/CD 工作流檔案 (<code>.github/</code>)</td>
      <td>修改或觸發自動化建置腳本</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>gist</code></strong></td>
      <td>建立與管理 Gist 代碼便利貼</td>
      <td>允許 <code>gh gist create</code></td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>write:packages</code></strong></td>
      <td>上傳與發布 Docker 映像檔或 npm 套件</td>
      <td>發布套件庫至 GitHub Packages</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>admin:repo_hook</code></strong></td>
      <td>建立或修改 Webhooks 伺服器回調鉤子</td>
      <td>第三方服務即時連動通知</td>
    </tr>
  </tbody>
</table>

---

## 4. 如何生成 Token 並在本地安全使用？

### 步驟 1：在 GitHub 生成 Token
1. 登入 GitHub，點擊右上角頭像 ➔ **Settings（個人設定）**。
2. 左側選單滑到最底部 ➔ 點擊 **Developer Settings（開發者設定）**。
3. 點擊 **Personal access tokens** ➔ 選擇 **Fine-grained tokens**（或 Tokens (classic)）。
4. 點擊 **`Generate new token`**，輸入名稱（如 `Work-Laptop-2026`）、設定 90 天過期、勾選所需權限。
5. 點擊生成後，**立刻複製 `ghp_...` 或 `github_pat_...`**（*注意：關閉網頁後就再也看不到了！*）。

---

### 步驟 2：在本地終端機安全登入與管理 (GitHub CLI)

```powershell
# 1. 推薦做法：使用 GitHub CLI 一鍵登入並安全保存至系統憑證庫 (Credential Manager)
gh auth login

# 2. 查詢目前登入身分與 Token 擁有之權限範圍：
gh auth status

# 3. 重新整理或更新權限 Scope（例如追加 gist 權限）：
gh auth refresh -s gist,workflow

# 4. 登出當前帳號並清除本地快取：
gh auth logout
```
