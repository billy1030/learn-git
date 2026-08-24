# Level 03: 功能分支、暫存儲藏與工作樹 (Branches, Stashes & Worktrees)

本章介紹如何在多個任務與版本之間靈活切換，保持工作區整潔而不丟失進度。

---

## 1. 暫存儲藏 (Git Stash — 臨時置物櫃)

當您正在開發新功能（寫了一半、尚未完成、不能 Commit），卻必須緊急切換去修 Bug 時：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 15%;">操作動作</th>
      <th style="width: 48%;">指令 (Command - 單行完整)</th>
      <th style="width: 37%;">中文作用說明與比喻</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 快速儲藏</strong></td>
      <td style="white-space: nowrap;"><code>git stash</code></td>
      <td>將所有未提交的修改掃進臨時抽屜，還原乾淨工作區。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 標籤儲藏</strong></td>
      <td style="white-space: nowrap;"><code>git stash push -m "WIP: 登入表單排版"</code></td>
      <td>加上備註標籤存入，方便日後識別。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 查看清單</strong></td>
      <td style="white-space: nowrap;"><code>git stash list</code></td>
      <td>查看目前抽屜內所有的存檔清單與編號。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 還原並刪除</strong></td>
      <td style="white-space: nowrap;"><code>git stash pop</code></td>
      <td>取出最新一筆暫存並套用，同時將其自抽屜中刪除。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 預覽差異</strong></td>
      <td style="white-space: nowrap;"><code>git stash show -p</code></td>
      <td>檢視抽屜中最新存檔的具體代碼差異而不套用。</td>
    </tr>
  </tbody>
</table>

---

## 2. Git 工作樹 (Git Worktree — 現代多工並行神器)

### 什麼是 Worktree？
一般切換分支時，同一個資料夾內的檔案會被整批覆寫替換；
而 **Git Worktree** 允許您在硬碟上**同時開闢多個獨立資料夾**，各自分別指向不同的分支，但**共享底層同一個 `.git` 歷史紀錄庫**（完全不浪費磁碟空間下載重複紀錄）。

```
c:\ai\
├── mds/                  <-- 主工作區 (分支：master / 穩定 1.0 正式版)
├── mds-v2/               <-- 工作樹 1 (分支：v2.0 實驗重構版)
└── mds-hotfix/           <-- 工作樹 2 (分支：hotfix-auth 緊急修復)
```

---

## 3. Worktree 常用核心指令表 (Worktree Command Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 16%;">操作動作</th>
      <th style="width: 48%;">指令 (Command - 單行完整)</th>
      <th style="width: 36%;">中文作用說明與效果</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 建立新分支工作樹</strong></td>
      <td style="white-space: nowrap;"><code>git worktree add ../mds-v2 -b v2.0</code></td>
      <td>開闢新資料夾 <code>mds-v2</code> 並直接切換到全新 <code>v2.0</code> 分支。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 建立既有分支工作樹</strong></td>
      <td style="white-space: nowrap;"><code>git worktree add ../mds-hotfix master</code></td>
      <td>從現有分支開闢獨立工作資料夾。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 列出所有工作樹</strong></td>
      <td style="white-space: nowrap;"><code>git worktree list</code></td>
      <td>列出目前本機所有掛載的實體資料夾與綁定分支。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 移除工作樹</strong></td>
      <td style="white-space: nowrap;"><code>git worktree remove ../mds-v2</code></td>
      <td>刪除指定工作樹資料夾並解除登記。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 清理過期參考</strong></td>
      <td style="white-space: nowrap;"><code>git worktree prune</code></td>
      <td>清理已手動刪除資料夾的殘留指標。</td>
    </tr>
  </tbody>
</table>

---

## 4. 實戰場景與 AI 協同開發優勢

1. **並行開發 1.0 與 2.0**：同時開啟兩個終端機與開發伺服器（Port `3000` 與 `3001`），左右螢幕即時對比畫面與功能差異。
2. **AI Subagent 獨立沙盒**：將特定工作樹目錄交給 AI 代理進行代碼重構或寫測試，完全不會干擾您在主視窗中正在敲鍵盤編輯的檔案。
3. **零中斷熱修復 (Hotfix)**：生產環境出狀況時，在獨立 hotfix 工作樹中快速修復、提交並推送，完全不必動用 `git stash`。
