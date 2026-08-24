# Level 09: 團隊成員管理、權限控制與安全稽核 (Team Permissions & Access Control)

本章介紹如何在 GitHub 中建立團隊架構、為不同職位（唯讀、PM、工程師、架構師、管理員）配置五大細粒度權限，以及透過 Web UI 與 GitHub CLI (`gh`) 自動化管理團隊成員。

---

## 1. 核心思維模型：五大權限層級比喻 (The 5 Permission Levels)

在「出版社出版書籍」的體系中，不同成員擁有不同等級的鑰匙與通行證：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 14%;">權限角色 (Role)</th>
      <th style="width: 16%;">中文稱呼</th>
      <th style="width: 44%;">具體權限範圍 (Permissions)</th>
      <th style="width: 26%;">適用對象與比喻</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. Read (Pull)</strong></td>
      <td><strong>唯讀成員</strong> 👁️</td>
      <td>可查看代碼、Clone / Pull 下載、提 Issue 與發起 PR，<strong>嚴禁直接 Push 代碼</strong>。</td>
      <td><strong>書店讀者/實習生</strong><br>只能閱讀與提建議，不能在原稿上畫畫。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. Triage</strong></td>
      <td><strong>問題分流員</strong> 📋</td>
      <td>擁有 Read 權限，並可指派 Issue / PR 負責人、修改標籤與里程碑，<strong>不能 Push 代碼</strong>。</td>
      <td><strong>專案經理 (PM) / QA 測試</strong><br>負責分發公文與貼標籤，不直接寫代碼。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. Write (Push)</strong></td>
      <td><strong>核心開發者</strong> ✍️</td>
      <td>可直接建立與推送分支、編輯 Wiki、上傳 Release 資產、直接合併無保護分支。</td>
      <td><strong>簽約主力作家 (Developers)</strong><br>擁有書桌鑰匙，負責實際撰寫代碼。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. Maintain</strong></td>
      <td><strong>專案維護者</strong> 🛠️</td>
      <td>擁有 Write 權限，並可管理儲存庫部分設定、鎖定討論串、管理分支保護規則。</td>
      <td><strong>資深責任編輯 / Team Lead</strong><br>把關各章節流程與審核規範。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. Admin</strong></td>
      <td><strong>最高管理員</strong> 👑</td>
      <td>擁有完全控制權（刪除專案庫、增刪成員與修改權限、管理金鑰與 Webhooks）。</td>
      <td><strong>出版社社長 / CTO / 創辦人</strong><br>掌管整家出版社的生殺大權。</td>
    </tr>
  </tbody>
</table>

---

## 2. 三大團隊成員指派實戰範例 (3 Real-World Permission Scenarios)

### 場景 1：邀請外部審計顧問（唯讀 Read 權限）
- **需求**：資安稽核團隊需要查驗代碼是否符合 ISO 規範，但絕對不能修改任何檔案。
- **CLI 實戰**：
```powershell
gh api --method PUT /repos/billy1030/learn-git/collaborators/auditor_john -f permission=pull
```

---

### 場景 2：邀請敏捷專案經理 (PM - Triage 權限)
- **需求**：PM 需要在 GitHub Issues 上指派工程師、修改 Milestone 里程碑與排定優先級，但不直接寫 code。
- **CLI 實戰**：
```powershell
gh api --method PUT /repos/billy1030/learn-git/collaborators/pm_sarah -f permission=triage
```

---

### 場景 3：專案結束一秒撤銷外包存取權 (Revoke Access)
- **需求**：專案驗收完成，合約結束，立刻收回外包工程師的存取權限。
- **CLI 實戰**：
```powershell
gh api --method DELETE /repos/billy1030/learn-git/collaborators/contractor_mike
```

---

## 3. 方式 A：透過 GitHub 網頁介面管理 (Web UI)

適合直觀的人眼確認與單一成員授權：

1. 開啟瀏覽器進入 GitHub 專案首頁。
2. 點擊頂部導覽列的 **⚙️ Settings（設定）**。
3. 在左側選單點擊 **Collaborators（協作者）**（組織專案為 **Collaborators and teams**）。
4. 點擊綠色按鈕 **`Add people`（新增成員）**。
5. 輸入對方的 **GitHub 使用者名稱 (Username)** 或 **Email**。
6. 在下拉選單選擇賦予的角色權限（`Read` / `Triage` / `Write` / `Maintain` / `Admin`）。
7. 點擊 **`Add <username> to this repository`** 送出邀請函。
   *(受邀者確認 Email 或登入 GitHub 點擊接受後即刻生效)*

---

## 4. 方式 B：透過 GitHub CLI (`gh`) 命令列自動化管理

適合 DevOps 工程師、團隊主管或批次腳本快速授權：

### A. 邀請新成員並設定權限
```powershell
# 語法：gh api --method PUT /repos/<擁有者>/<專案>/collaborators/<帳號> -f permission=<權限名稱>

# 1. 邀請為「唯讀成員 (Read-only)」
gh api --method PUT /repos/billy1030/learn-git/collaborators/alex123 -f permission=pull

# 2. 邀請為「問題分流員 (Triage)」
gh api --method PUT /repos/billy1030/learn-git/collaborators/pm_sarah -f permission=triage

# 3. 邀請為「核心開發者 (Write)」
gh api --method PUT /repos/billy1030/learn-git/collaborators/dev_david -f permission=push

# 4. 邀請為「資深維護者 (Maintain)」
gh api --method PUT /repos/billy1030/learn-git/collaborators/lead_michael -f permission=maintain

# 5. 邀請為「最高管理員 (Admin)」
gh api --method PUT /repos/billy1030/learn-git/collaborators/co_founder -f permission=admin
```

> [!NOTE]
> 在 GitHub API 底層參數中：
> - `permission=pull` 代表 **Read (唯讀)**
> - `permission=push` 代表 **Write (寫入)**
> - `permission=admin` 代表 **Admin (管理員)**

---

### B. 查詢現有成員名單與權限等級
```powershell
# 列出專案中所有協作者帳號
gh api /repos/billy1030/learn-git/collaborators --jq ".[].login"

# 檢視特定成員（例如 alex123）的詳細權限等級
gh api /repos/billy1030/learn-git/collaborators/alex123/permission --jq ".permission"
```

---

## 5. 團隊權限管理常見指令速查表 (Quick Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 20%;">需求動作</th>
      <th style="width: 52%;">GitHub CLI 指令 (Command)</th>
      <th style="width: 28%;">說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>邀請唯讀成員</td>
      <td style="white-space: nowrap;"><code>gh api --method PUT .../collaborators/user -f permission=pull</code></td>
      <td>只能 Clone 與看 Code，不能 Push</td>
    </tr>
    <tr>
      <td>邀請開發工程師</td>
      <td style="white-space: nowrap;"><code>gh api --method PUT .../collaborators/user -f permission=push</code></td>
      <td>可建立分支、推送與發 PR</td>
    </tr>
    <tr>
      <td>查詢成員名單</td>
      <td style="white-space: nowrap;"><code>gh api .../collaborators --jq ".[].login"</code></td>
      <td>輸出所有具存取權之帳號</td>
    </tr>
    <tr>
      <td>移除成員權限</td>
      <td style="white-space: nowrap;"><code>gh api --method DELETE .../collaborators/user</code></td>
      <td>立刻收回鑰匙與存取權限</td>
    </tr>
  </tbody>
</table>
