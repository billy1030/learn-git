# 00 - Git Master Index & 學習路徑總覽 (Learning Roadmap)

歡迎來到完整的漸進式 Git 實戰指南。本課程共計 **18 大核心級別**，劃分為 **初階 (Beginner)**、**中階 (Intermediate)** 與 **進階/企業級 (Advanced & Enterprise)** 三大模組，帶您從零基礎一路進階至全方位軟體架構師。

---

## 📚 課程架構一覽表 (Curriculum Structure)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 10%; text-align: center;">級別 (Level)</th>
      <th style="width: 25%;">教學檔案 (File)</th>
      <th style="width: 22%;">學習階段 (Stage)</th>
      <th style="width: 43%;">核心關鍵字與概念 (Key Concepts)</th>
    </tr>
  </thead>
  <tbody>
    <!-- 初階模組 -->
    <tr>
      <td style="text-align: center;"><strong>Level 01</strong></td>
      <td><a href="./01-beginner-foundations.md"><code>01-beginner-foundations.md</code></a></td>
      <td>初階：個人本機基礎<br>(Foundations &amp; Loop)</td>
      <td>三大區域, <code>init</code>, <code>status</code>, <code>add</code>, <code>commit</code>, Commit ID 數位指紋, 8 大 Conventional Commits 字典, <code>.gitignore</code> 規則, Git Aliases</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 02</strong></td>
      <td><a href="./02-remote-github-sync.md"><code>02-remote-github-sync.md</code></a></td>
      <td>初階：遠端雲端同步<br>(Cloud Sync &amp; Remotes)</td>
      <td><code>remote</code>, <code>push -u</code>, <code>fetch --all --prune</code>, <code>pull --ff-only</code>, <code>git</code> vs <code>gh</code> 4 角色地圖, 寫書出版比喻</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 03</strong></td>
      <td><a href="./03-tokens-access-rights.md"><code>03-tokens-access-rights.md</code></a></td>
      <td>中階：憑證與安全存取<br>(Tokens &amp; Access Rights)</td>
      <td>飯店感應房卡比喻, 經典版 (Classic) vs 細粒度版 (Fine-grained), 常用 Scopes 字典, <code>gh auth login/status/refresh</code> 安全認證</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 04</strong></td>
      <td><a href="./04-branching-worktrees.md"><code>04-branching-worktrees.md</code></a></td>
      <td>中階：分支與多工並行<br>(Branching &amp; Worktrees)</td>
      <td>Feature Branching (功能分支), <code>git stash</code> (暫存置物櫃), <strong>Git Worktrees</strong> (工作樹：免切換、多資料夾並行 1.0/2.0 與 AI 沙盒)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 05</strong></td>
      <td><a href="./05-intermediate-team-collaboration.md"><code>05-intermediate-team-collaboration.md</code></a></td>
      <td>中階：團隊協作與 PR<br>(Team Collaboration &amp; PR)</td>
      <td>Merge, Rebase vs Merge (變基 vs 合併), 單檔案衝突 4 步驟化解, Pull Requests (PR 深度解析), Conventional Commits</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 06</strong></td>
      <td><a href="./06-surgical-tools-cherry-pick.md"><code>06-surgical-tools-cherry-pick.md</code></a></td>
      <td>中階：精準手術刀工具<br>(Precision Surgical Tools)</td>
      <td><code>git cherry-pick</code> (跨分支精準移植 Bug 修復與 0% 洩漏清單), <code>git add -p</code> (區塊暫存), <code>git rebase -i</code> (互動式整理提交)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 07</strong></td>
      <td><a href="./07-advanced-debugging-recovery.md"><code>07-advanced-debugging-recovery.md</code></a></td>
      <td>進階：除錯與災難復原<br>(Debugging &amp; Recovery)</td>
      <td><code>git reflog</code> (終極後悔藥), <code>git bisect</code> (二分搜尋除錯), <code>git log -S</code> (十字鎬字串搜尋), <code>git blame</code> (逐行責任標記)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 08</strong></td>
      <td><a href="./08-expert-enterprise-monorepo.md"><code>08-expert-enterprise-monorepo.md</code></a></td>
      <td>進階：企業級與 Monorepo<br>(Enterprise &amp; Monorepo)</td>
      <td><code>git rerere</code> (衝突自動重用), <code>sparse-checkout</code> (稀疏檢出/大型專案秒載), <code>git bundle</code> (離線封裝), Git Hooks, 金鑰抹除, 提交簽名</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 09</strong></td>
      <td><a href="./09-team-permissions-roles.md"><code>09-team-permissions-roles.md</code></a></td>
      <td>進階：成員權限管理<br>(Team RBAC &amp; Roles)</td>
      <td>五大權限層級 (Read, Triage, Write, Maintain, Admin), Web UI 邀請, <code>gh api</code> 命令列批次授權, 權限查詢與撤銷</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 10</strong></td>
      <td><a href="./10-branch-protection-rules.md"><code>10-branch-protection-rules.md</code></a></td>
      <td>進階：主幹防護與審批<br>(Branch Protection Rules)</td>
      <td>主幹上鎖保護, 強制 PR 審查 (Require Reviews), 強制 CI 綠燈 (Status Checks), 阻擋 <code>push -f</code> 強制覆蓋, <code>gh api</code> 上鎖</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 11</strong></td>
      <td><a href="./11-cicd-github-actions.md"><code>11-cicd-github-actions.md</code></a></td>
      <td>進階：CI/CD 自動化品檢<br>(GitHub Actions Pipelines)</td>
      <td>無人自動化工廠比喻, <code>.github/workflows/ci.yml</code> 實戰, 自動測試打包, <code>gh run list / watch</code> 命令列監控</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 12</strong></td>
      <td><a href="./12-git-submodules-subtrees.md"><code>12-git-submodules-subtrees.md</code></a></td>
      <td>進階：跨專案模組共用<br>(Submodules &amp; Subtrees)</td>
      <td>附錄字典比喻, <code>git submodule add</code>, 遞迴 Clone (<code>--recurse-submodules</code>), 子模組同步更新, Submodule vs Subtree 選型</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 13</strong></td>
      <td><a href="./13-release-management-semver.md"><code>13-release-management-semver.md</code></a></td>
      <td>進階：版本發布與 Release<br>(SemVer &amp; Releases)</td>
      <td>SemVer (Major.Minor.Patch), 附註標籤 (<code>git tag -a</code>), <code>gh release create</code> 一鍵發布正式公告與安裝檔資產</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 14</strong></td>
      <td><a href="./14-ai-agent-git-workflows.md"><code>14-ai-agent-git-workflows.md</code></a></td>
      <td>前沿：AI 人機協同工作流<br>(AI Agents &amp; Subagents)</td>
      <td>AI 獨立書桌沙盒比喻, AI Worktree 開發隔離, 提示詞生成規範 Commit, 人眼終審與原子合併 (Squash)</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 15</strong></td>
      <td><a href="./15-git-lfs-large-assets.md"><code>15-git-lfs-large-assets.md</code></a></td>
      <td>前沿：大檔與 AI 模型版控<br>(Git LFS &amp; Large Assets)</td>
      <td>雲端置物櫃與 1KB 提貨券指標比喻, <code>git lfs install / track</code>, <code>.gitattributes</code>, ONNX/模型/影音大檔極速拉取</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 16</strong></td>
      <td><a href="./16-github-gist-snippets.md"><code>16-github-gist-snippets.md</code></a></td>
      <td>工具：代碼片段與便利貼<br>(GitHub Gist &amp; Snippets)</td>
      <td>4 大生活比喻, 3 大實戰場景 (錯誤日誌短網址、一鍵安裝腳本、部落格嵌入), <code>gh gist create</code>, Public vs Secret</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 17</strong></td>
      <td><a href="./17-issues-projects-kanban.md"><code>17-issues-projects-kanban.md</code></a></td>
      <td>管理：工單與敏捷看板<br>(Issues &amp; Projects)</td>
      <td>讀者勘誤表與編輯部白板比喻, 魔法關鍵字自動結案 (<code>Fixes #42</code>), <code>gh issue create/list/close</code></td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>Level 18</strong></td>
      <td><a href="./18-secrets-dependabot-security.md"><code>18-secrets-dependabot-security.md</code></a></td>
      <td>資安：機密管理與自動防禦<br>(Secrets &amp; Dependabot)</td>
      <td>金庫保險箱與 24H 警報比喻, Repository Secrets 加密, Dependabot 自動漏洞修復 PR, Push Protection 密鑰攔截</td>
    </tr>
  </tbody>
</table>

---

## 快速導覽 (Quick Navigation)

請從第一章開始閱讀：**[Level 01: 基礎核心與個人工作流 (Beginner Foundations)](./01-beginner-foundations.md)**。
