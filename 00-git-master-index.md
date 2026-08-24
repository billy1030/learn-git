# 00 - Git Master Index & 學習路徑總覽 (Learning Roadmap)

歡迎來到完整的漸進式 Git 實戰指南。本課程共計 **16 大核心級別**，帶您從零基礎個人開發，逐步進階至團隊協作、企業架構、分支防護、CI/CD 自動化、版本發布、AI 協同、大型模型版控、Gist 便利貼與 Token 安全權限體系。

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
      <td><code>init</code>, <code>status</code>, <code>add</code>, <code>commit</code>, <code>log</code>, <code>diff</code>, <code>.gitignore</code> 深度規則, Commit ID 數位指紋, 8 大 Conventional Commits, Git Aliases</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 02</strong></td>
      <td><a href="./02-remote-github-sync.md"><code>02-remote-github-sync.md</code></a></td>
      <td>雲端同步與個人遠端協作<br>(Cloud Sync &amp; Remotes)</td>
      <td><code>remote</code> (遠端庫), <code>push</code> (推送), <code>fetch</code> (抓取更新), <code>pull --ff-only</code> (快進拉取), Upstream Tracking (上游追蹤分支), <code>git</code> vs <code>gh</code> 4 角色地圖</td>
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
    <tr>
      <td style="text-align: center;"><strong>Level 08</strong></td>
      <td><a href="./08-team-permissions-roles.md"><code>08-team-permissions-roles.md</code></a></td>
      <td>團隊成員管理與權限控制<br>(Team Permissions &amp; Roles)</td>
      <td>五大權限層級 (Read, Triage, Write, Maintain, Admin), Web UI 邀請, <code>gh api</code> 命令列批次授權, 權限查詢與撤銷</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 09</strong></td>
      <td><a href="./09-branch-protection-rules.md"><code>09-branch-protection-rules.md</code></a></td>
      <td>主幹分支保護與合規審批<br>(Branch Protection Rules)</td>
      <td>強制 PR 審批 (Require Reviews), 強制 CI 測試通過 (Status Checks), 阻擋 <code>push -f</code> 強制覆蓋, <code>gh api</code> 命令列上鎖</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 10</strong></td>
      <td><a href="./10-cicd-github-actions.md"><code>10-cicd-github-actions.md</code></a></td>
      <td>CI/CD 自動化與 GitHub Actions<br>(Pipelines &amp; Automation)</td>
      <td>無人自動化工廠比喻, <code>.github/workflows/ci.yml</code> 實戰, 自動測試與打包, <code>gh run list / watch</code> 命令列即時監控</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 11</strong></td>
      <td><a href="./11-git-submodules-subtrees.md"><code>11-git-submodules-subtrees.md</code></a></td>
      <td>跨專案模組共用與子儲存庫<br>(Submodules &amp; Subtrees)</td>
      <td>附錄字典比喻, <code>git submodule add</code>, 遞迴 Clone (<code>--recurse-submodules</code>), 子模組同步更新, Submodule vs Subtree 選型</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 12</strong></td>
      <td><a href="./12-release-management-semver.md"><code>12-release-management-semver.md</code></a></td>
      <td>語意化版本與 Release 發布<br>(SemVer &amp; Releases)</td>
      <td>SemVer (Major.Minor.Patch), 附註標籤 (<code>git tag -a</code>), <code>gh release create</code> 一鍵發布正式公告與安裝檔資產</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 13</strong></td>
      <td><a href="./13-ai-agent-git-workflows.md"><code>13-ai-agent-git-workflows.md</code></a></td>
      <td>AI 時代協同 Git 工作流<br>(AI Agents &amp; Subagents)</td>
      <td>AI 獨立書桌沙盒比喻, AI Worktree 開發隔離, 提示詞生成規範 Commit, 人眼終審與原子合併 (Squash)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 14</strong></td>
      <td><a href="./14-git-lfs-large-assets.md"><code>14-git-lfs-large-assets.md</code></a></td>
      <td>超大二進位檔與 AI 模型管理<br>(Git LFS &amp; Large Assets)</td>
      <td>雲端置物櫃與 1KB 提貨券指標比喻, <code>git lfs install / track</code>, <code>.gitattributes</code>, ONNX/模型/影音大檔極速拉取</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 15</strong></td>
      <td><a href="./15-github-gist-snippets.md"><code>15-github-gist-snippets.md</code></a></td>
      <td>輕量程式碼片段與雲端便利貼<br>(GitHub Gist &amp; Snippets)</td>
      <td>4 大生活比喻, 3 大實戰場景 (日誌短網址、一鍵安裝腳本、部落格嵌入), <code>gh gist create</code>, Public vs Secret</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 16</strong></td>
      <td><a href="./16-tokens-access-rights.md"><code>16-tokens-access-rights.md</code></a></td>
      <td>安全存取憑證與細粒度權限<br>(Tokens, PAT &amp; Scopes)</td>
      <td>飯店感應房卡比喻, 經典版 (Classic) vs 細粒度版 (Fine-grained), 常用 Scopes 字典, <code>gh auth login/status/refresh</code> 安全認證</td>
    </tr>
  </tbody>
</table>

---

## 快速導覽 (Quick Navigation)

請從第一章開始閱讀：**[Level 01: 基礎核心與個人工作流 (Beginner Foundations)](./01-beginner-foundations.md)**。
