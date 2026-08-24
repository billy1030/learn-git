# Level 09: 主幹分支保護與合規審批規則 (Branch Protection & Quality Gates)

本章介紹如何在 GitHub 中為核心分支（`master` / `main`）建立不可撼動的安全防護罩，防止任何人手滑覆蓋線上正式環境，並強制執行代碼審查 (Code Review) 與 CI 自動驗證。

---

## 1. 核心思維模型：為什麼需要分支保護？ (The Safety Vault Analogy)

> **寫書與出版比喻**：
> - **無保護的主幹**：就像出版社的「官方定稿印刷機」沒有上鎖，任何實習生都能隨便走進去按下印刷按鈕，甚至不小心把整本書撕掉（`git push -f` 災難）。
> - **上鎖保護的主幹**：印刷機裝上防彈玻璃與雙重鑰匙孔。
>   1. **嚴禁直接修改**：禁止任何人直接對定稿本動筆。
>   2. **雙人會簽 (PR Review)**：必須至少有另一位責任編輯審閱簽名（Approve）。
>   3. **自動校對機綠燈 (Status Checks)**：自動校對機檢查無錯字後，閘門才會打開允許印刷。

---

## 2. 三大核心保護機制 (3 Core Protection Rules)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 25%;">保護機制 (Rule)</th>
      <th style="width: 45%;">具體防護效果</th>
      <th style="width: 30%;">寫書生活化比喻</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>1. Require PR Reviews</strong><br>(強制代碼審查)</td>
      <td>禁止直接 <code>git push origin master</code>。所有修改必須透過 PR 發起，且需達到指定人數（如 1 人以上）Approve 才能合併。</td>
      <td>稿件必須至少有 1 位編輯看過並蓋章。</td>
    </tr>
    <tr>
      <td><strong>2. Require Status Checks</strong><br>(強制 CI 測試通過)</td>
      <td>GitHub Actions 自動化測試（如 <code>npm test</code>、<code>build</code>）必須全部顯示為綠燈勾勾 ✅ 才能解鎖合併按鈕。</td>
      <td>排版機必須跑完自動校對無誤才准印。</td>
    </tr>
    <tr>
      <td><strong>3. Block Force Pushes</strong><br>(禁止強制覆蓋與刪除)</td>
      <td>嚴格禁止 <code>git push --force</code> 與刪除分支，防止不小心覆蓋整個團隊的歷史紀錄。</td>
      <td>禁止任何人拿碎紙機銷毀定稿本。</td>
    </tr>
  </tbody>
</table>

---

## 3. 方式 A：透過 GitHub 網頁介面設定保護規則 (Web UI)

1. 開啟 GitHub 專案首頁，點擊 **⚙️ Settings（設定）**。
2. 左側選單點擊 **Branches（分支）**。
3. 點擊 **`Add branch protection rule`（新增分支保護規則）**。
4. **Branch name pattern** 輸入：`master`（或 `main`）。
5. 勾選以下推薦防護項目：
   - ✅ **Require a pull request before merging**（合併前必須發起 PR）
     - ✅ *Require approvals: 1*（至少 1 位審查者批准）
     - ✅ *Dismiss stale pull request approvals when new commits are pushed*（作者推新代碼時，舊審批自動作廢需重新審）
   - ✅ **Require status checks to pass before merging**（合併前必須通過 CI 測試）
   - ✅ **Do not allow bypassing the above settings**（連最高管理員 Admin 也必須遵守此規則，不可硬闖）
6. 點擊頁面底部的 **`Save changes`**。

---

## 4. 方式 B：透過 GitHub CLI (`gh api`) 命令列一鍵上鎖

使用命令列快速為 `master` 分支套用嚴格的保護規則：

```powershell
# 1. 查詢目前 master 分支的保護狀態
gh api /repos/billy1030/learn-git/branches/master/protection

# 2. 一鍵為 master 分支啟用嚴格保護（強制 1 人審批 + 禁止 Force Push）
gh api --method PUT /repos/billy1030/learn-git/branches/master/protection `
  --input - <<< '{
    "required_status_checks": null,
    "enforce_admins": true,
    "required_pull_request_reviews": {
      "dismiss_stale_reviews": true,
      "require_code_owner_reviews": false,
      "required_approving_review_count": 1
    },
    "restrictions": null,
    "allow_force_pushes": false,
    "allow_deletions": false
  }'
```

---

## 5. 常見分支保護 CLI 查詢速查表 (Quick Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 25%;">需求動作</th>
      <th style="width: 50%;">GitHub CLI 指令 (Command)</th>
      <th style="width: 25%;">說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>查看主幹保護狀態</td>
      <td style="white-space: nowrap;"><code>gh api /repos/.../branches/master/protection</code></td>
      <td>檢視目前啟用的安全規則</td>
    </tr>
    <tr>
      <td>測試強制推送 (會被擋)</td>
      <td style="white-space: nowrap;"><code>git push origin master --force</code></td>
      <td>應回傳 <code>Protected branch hook declined</code></td>
    </tr>
    <tr>
      <td>解除分支保護 (謹慎)</td>
      <td style="white-space: nowrap;"><code>gh api --method DELETE /repos/.../branches/master/protection</code></td>
      <td>將 master 恢復為無保護狀態</td>
    </tr>
  </tbody>
</table>
