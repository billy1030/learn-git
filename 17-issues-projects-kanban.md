# Level 17: 專案需求與 Issue/Bug 自動化追蹤 (Issues, Projects & Magic Keywords)

本章介紹如何在 GitHub 中以工程化方式管理產品需求、Bug 勘誤表、自訂範本，並利用「魔法關鍵字 (Magic Keywords)」在 PR 合併時自動關閉關聯的 Issue。

---

## 1. 核心思維模型：讀者勘誤表與編輯部看板 (The Issue & Kanban Analogy)

> **寫書比喻**：
> - **GitHub Issues（讀者勘誤與需求表 📋）**：讀者發現第 5 章有錯字，或希望下一版增加「課後習題」，向出版社提交一張工單。
> - **GitHub Projects（編輯部敏捷看板 📊）**：主編桌上的磁吸白板，將所有工單劃分為 **Todo（待辦）** ➔ **In Progress（寫作中）** ➔ **Done（已發布）**。
> - **Magic Keywords（自動結案公文印章 🪄）**：你在寫草稿 PR 時附註 `Fixes #42`，當主編批准合併的那一秒，第 42 號勘誤工單**自動被蓋上「已解決」印章並歸檔**！

---

## 2. 魔法關鍵字字典 (Magic Keywords Reference)

在 Commit 訊息或 PR 說明中寫入以下關鍵字 + `#Issue編號`，當 PR 合併進預設分支 (`master`) 時，GitHub 會自動關閉該 Issue：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 22%;">魔法關鍵字 (Keyword)</th>
      <th style="width: 48%;">標準 Commit / PR 寫法範例</th>
      <th style="width: 30%;">自動化結案行為</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong><code>close / closes / closed</code></strong></td>
      <td style="white-space: nowrap;"><code>git commit -m "feat: complete login flow, closes #12"</code></td>
      <td>PR 合併時自動關閉 Issue #12</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>fix / fixes / fixed</code></strong></td>
      <td style="white-space: nowrap;"><code>git commit -m "fix(auth): fix token expiry bug (fixes #45)"</code></td>
      <td>PR 合併時自動關閉 Bug #45</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong><code>resolve / resolves</code></strong></td>
      <td style="white-space: nowrap;"><code>git commit -m "refactor(db): optimize query, resolves #88"</code></td>
      <td>PR 合併時自動解決 Issue #88</td>
    </tr>
  </tbody>
</table>

---

## 3. 透過 GitHub CLI (`gh issue`) 命令列管理 Issue

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
      <td style="white-space: nowrap;"><strong>1. 建立新 Issue</strong></td>
      <td style="white-space: nowrap;"><code>gh issue create --title "OAuth 登入逾時" --body "需排查 Token 機制"</code></td>
      <td>一秒在雲端建立新工單</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 列出當前開啟的 Issue</strong></td>
      <td style="white-space: nowrap;"><code>gh issue list --assignee "@me"</code></td>
      <td>查看指派給自己的任務清單</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 檢視特定工單詳情</strong></td>
      <td style="white-space: nowrap;"><code>gh issue view 42</code></td>
      <td>在終端機直接閱讀討論串</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 手動關閉 Issue</strong></td>
      <td style="white-space: nowrap;"><code>gh issue close 42 --comment "已於 v2.0 修復"</code></td>
      <td>手動結案並留下備註</td>
    </tr>
  </tbody>
</table>
