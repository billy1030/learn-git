# 00 - Git Master Index & 學習路徑總覽 (Learning Roadmap)

歡迎來到完整的漸進式 Git 實戰指南。本課程將帶您從零基礎逐步進階至企業級版本控制架構。

---

## 📚 課程架構一覽表 (Curriculum Structure)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 10%; text-align: center;">級別 (Level)</th>
      <th style="width: 25%;">教學檔案 (File)</th>
      <th style="width: 22%;">適用對象 (Audience)</th>
      <th style="width: 43%;">核心關鍵字與概念 (Key Concepts)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center;"><strong>Level 01</strong></td>
      <td><a href="./01-beginner-foundations.md"><code>01-beginner-foundations.md</code></a></td>
      <td>初學者與個人開發者<br>(Beginners &amp; Solo Devs)</td>
      <td><code>init</code> (初始化), <code>status</code> (狀態查詢), <code>add</code> (暫存/追蹤), <code>commit</code> (提交快照), <code>log</code> (日誌歷史), <code>diff</code> (差異比對), <code>.gitignore</code> (忽略檔案清單)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 02</strong></td>
      <td><a href="./02-remote-github-sync.md"><code>02-remote-github-sync.md</code></a></td>
      <td>雲端同步與個人遠端協作<br>(Cloud Sync &amp; Remotes)</td>
      <td><code>remote</code> (遠端庫), <code>push</code> (推送), <code>fetch</code> (抓取更新), <code>pull --ff-only</code> (快進拉取), Upstream Tracking (上游追蹤分支), Branch Switching (分支切換)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 03</strong></td>
      <td><a href="./03-branching-worktrees.md"><code>03-branching-worktrees.md</code></a></td>
      <td>多工並行與功能分支開發<br>(Multi-tasking &amp; Worktrees)</td>
      <td>Feature Branching (功能分支), <code>git stash</code> (暫存儲藏/草稿夾), <strong>Git Worktrees</strong> (工作樹：免切換、同時多資料夾並行 1.0 與 2.0)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 04</strong></td>
      <td><a href="./04-intermediate-team-collaboration.md"><code>04-intermediate-team-collaboration.md</code></a></td>
      <td>團隊多人協作開發<br>(Team Collaboration)</td>
      <td>Merge (合併), Merge Conflicts (衝突解決), Rebase vs Merge (變基 vs 合併), Pull Requests / PR (拉取請求), Conventional Commits (標準化提交規範)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 05</strong></td>
      <td><a href="./05-surgical-tools-cherry-pick.md"><code>05-surgical-tools-cherry-pick.md</code></a></td>
      <td>專業精準操作工具<br>(Precision Surgical Tools)</td>
      <td><code>git cherry-pick</code> (挑選特定提交移植 Bug 修復), <code>git add -p</code> (互動式區塊暫存), <code>git rebase -i</code> (互動式變基/合併整理提交紀錄)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 06</strong></td>
      <td><a href="./06-advanced-debugging-recovery.md"><code>06-advanced-debugging-recovery.md</code></a></td>
      <td>資深工程師除錯與災難復原<br>(Debugging &amp; Recovery)</td>
      <td><code>git reflog</code> (引用日誌/終極後悔藥), <code>git bisect</code> (二分搜尋法抓出出錯提交), <code>git log -S</code> (十字鎬字串搜尋), <code>git blame</code> (逐行責任標記)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 07</strong></td>
      <td><a href="./07-expert-enterprise-monorepo.md"><code>07-expert-enterprise-monorepo.md</code></a></td>
      <td>企業級架構與大型 Monorepo<br>(Enterprise &amp; DevOps)</td>
      <td><code>git rerere</code> (衝突解決自動重用), <code>sparse-checkout</code> (稀疏檢出/超大型專案秒載), <code>git bundle</code> (離線封裝), Git Hooks (提交前自動檢查), 敏感金鑰抹除, GPG/SSH 簽名認證</td>
    </tr>
  </tbody>
</table>

---

## 快速導覽 (Quick Navigation)

請從第一章開始閱讀：**[Level 01: 基礎核心與個人工作流 (Beginner Foundations)](./01-beginner-foundations.md)**。
