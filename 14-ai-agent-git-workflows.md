# Level 13: AI 時代的協同 Git 工作流 (AI Agents & Pair Programming)

本章介紹在現代 AI 程式助理（Antigravity, Cursor, Claude Code, Aider）普及的時代，如何建立人機協同的最佳 Git 實踐，避免 AI 覆蓋工程師代碼、實現自動生成高水準 Commit 與審批隔離。

---

## 1. 核心思維模型：AI 助教的「獨立書桌」比喻 (AI Sandbox Analogy)

> **寫書比喻**：
> - **混亂做法（同桌共寫）**：你和一位反應極快但有時會幻覺的「AI 助教」坐在同一張桌子上共用同一張稿紙。你剛寫完第一段，AI 突然大筆一揮把整頁塗改，你很難分清哪些字是你寫的、哪些是 AI 改的。
> - **專業做法（Worktree 獨立沙盒）**：你在隔壁為 AI 安排一張**「獨立書桌（AI Worktree / AI 分支）」**。
>   1. **隔離開發**：AI 在 `ai/feat-oauth` 分支與專屬目錄下瘋狂編寫。
>   2. **工程師驗收**：完成後，工程師使用 `git diff` 逐行審查 AI 的成果，確認安全無誤後才併入主分支。

---

## 2. AI 協同開發三大標準黃金法則 (3 Golden Rules)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 22%;">黃金法則</th>
      <th style="width: 48%;">標準指令 (Command)</th>
      <th style="width: 30%;">核心效益說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 任務開闢獨立沙盒</strong></td>
      <td style="white-space: nowrap;"><code>git worktree add ../ai-workspace -b ai/refactor-auth</code></td>
      <td>將 AI 隔離在獨立資料夾，完全不干擾工程師當前視窗。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 讓 AI 生成規範 Commit</strong></td>
      <td style="white-space: nowrap;"><code>git diff --staged | prompt-ai "generate conventional commit"</code></td>
      <td>要求 AI 分析暫存差異，自動生成符合 <code>feat/fix</code> 標準的訊息。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 人眼終審與原子合併</strong></td>
      <td style="white-space: nowrap;"><code>git diff master..ai/refactor-auth &amp;&amp; git merge --squash</code></td>
      <td>工程師擁有最後審核權，將 AI 的多筆零碎提交壓合成乾淨節點。</td>
    </tr>
  </tbody>
</table>

---

## 3. 實戰流程：指揮 AI 代理進行代碼重構的完整 CLI

```powershell
# 1. 為 AI 開闢獨立工作樹與任務分支
git worktree add ../ai-task-worker -b ai/optimize-billing

# 2. 指示 AI 代理在 ../ai-task-worker 目錄下重構並執行測試
# (AI 完成代碼修改與單元測試...)

# 3. 在 AI 分支上提交成果
cd ../ai-task-worker
git add .
git commit -m "refactor(billing): optimize tax calculation loops and add tests"

# 4. 工程師回到主工作目錄進行差異驗收
cd ../mds
git diff master..ai/optimize-billing

# 5. 確認無誤後，Squash 合併入主幹並清理 AI 工作樹
git merge --squash ai/optimize-billing
git commit -m "refactor(billing): apply AI-assisted billing optimization"
git worktree remove ../ai-task-worker
git branch -D ai/optimize-billing
```
