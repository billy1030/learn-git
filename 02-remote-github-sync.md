# Level 02: 遠端同步與 GitHub 工作流 (Remote Synchronization & GitHub Workflow)

本章介紹如何將本機版本庫連接到遠端託管平台（如 GitHub、GitLab），推送代碼並安全拉取團隊更新，並透過「合寫一本書」的比喻徹底釐清 `git` 與 `gh` 的核心分工以及各角色（開發者、審查者、架構師、DevOps）的專屬單行指令集。

---

## 1. 核心觀念澄清：`git` vs `gh` 的本質差別（寫書比喻）

在「多人合寫一本書」的出版流程中，`git` 與 `gh` 分別扮演完全不同的角色：

- **`git` 是你的「打字機與裝訂機」** ✍️：
  - 放在你的個人書桌上，完全不需要網路。
  - 你用它在座位上寫章節、修改草稿、撕掉重寫、把草稿裝訂成小冊子（`commit`、`branch`）。
  - 你可以用它把寫好的章節郵寄給任何出版社（GitHub、GitLab 或私人印刷廠）。

- **`gh` 是出版社的「雲端公文系統專用對講機」** 📡：
  - 必須連上網路，而且**專門對接「GitHub 這家特定出版社」的總部大樓**。
  - 你在自己座位上按對講機，就能直接向出版社總編「發起審稿請求（PR）」、「登記勘誤表（Issue）」、「發布新書出版（Release）」，完全不必親自跑去出版社網站填表單。

---

### `git` vs `gh` 核心對比表

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 15%;">比較維度</th>
      <th style="width: 42.5%;"><code>git</code> (版本控制核心)</th>
      <th style="width: 42.5%;"><code>gh</code> (GitHub 專用對講機)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>寫書角色比喻</strong></td>
      <td><strong>書桌上的打字機與裝訂夾</strong> ✍️<br>（管草稿怎麼寫、怎麼裝訂、怎麼備份）</td>
      <td><strong>出版社的總部公文對講機</strong> 📡<br>（管審稿排程、總編蓋章、勘誤登記）</td>
    </tr>
    <tr>
      <td><strong>核心本質</strong></td>
      <td>分散式版本控制系統 (VCS)</td>
      <td>GitHub 網站平台的命令列包裝工具</td>
    </tr>
    <tr>
      <td><strong>連網依賴</strong></td>
      <td><strong>完全離線可用</strong>（書桌前自己寫稿存檔不需網路）</td>
      <td><strong>必須連網</strong>（需與 GitHub 出版社伺服器通訊）</td>
    </tr>
    <tr>
      <td><strong>適用平台</strong></td>
      <td>通用於任何印刷廠（GitHub、GitLab、Bitbucket、自建庫）</td>
      <td><strong>專屬於 GitHub 出版社</strong></td>
    </tr>
    <tr>
      <td><strong>主要負責範疇</strong></td>
      <td>草稿快照、分支、暫存、比對、郵寄/收取章節<br>(<code>add</code>, <code>commit</code>, <code>branch</code>, <code>push</code>, <code>pull</code>)</td>
      <td>出版社審稿、問題回報、雲端自動校對排版<br>(<code>pr create</code>, <code>issue list</code>, <code>release</code>, <code>run</code>)</td>
    </tr>
  </tbody>
</table>

---

## 2. 團隊各角色職責與專屬單行指令地圖 (Role-Based Command Matrix)

在專業團隊中，不同角色在專案生命週期中會使用不同的指令集：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 12%;">角色 (Role)</th>
      <th style="width: 16%;">職責比喻</th>
      <th style="width: 56%;">主要常用指令 (Primary CLI)</th>
      <th style="width: 16%;">日常典型動作</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 功能開發者<br>(Developer)</strong></td>
      <td><strong>草稿撰寫作者</strong> ✍️</td>
      <td style="white-space: nowrap;">
        <code>git checkout -b feat/user-auth</code><br>
        <code>git add . &amp;&amp; git commit -m "feat: login form"</code><br>
        <code>git push -u origin feat/user-auth</code><br>
        <code>gh pr create --fill --base master</code>
      </td>
      <td>開分支、寫代碼存檔、推上雲端發 PR 請求審查。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 代碼審查者<br>(Reviewer)</strong></td>
      <td><strong>資深責任編輯</strong> 🧐</td>
      <td style="white-space: nowrap;">
        <code>gh pr list</code><br>
        <code>gh pr checkout 42</code><br>
        <code>gh pr diff 42</code><br>
        <code>gh pr review 42 --approve -b "LGTM, looks good!"</code>
      </td>
      <td>拉取隊友 PR 分支測試、比對代碼差異、線上給予 Approve。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 技術主編 / 架構師<br>(Tech Lead)</strong></td>
      <td><strong>出版社總編輯</strong> 👨‍💼</td>
      <td style="white-space: nowrap;">
        <code>gh pr merge 42 --squash --delete-branch</code><br>
        <code>git cherry-pick a1b2c3d</code><br>
        <code>git tag -a v2.0.0 -m "Release version 2.0.0"</code><br>
        <code>gh release create v2.0.0 --notes "Initial v2.0 release"</code>
      </td>
      <td>執行 Squash 合併至主幹、跨版本移植修復、打 Tag 與發布 Release。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 維運 / CI/CD 工程師<br>(DevOps)</strong></td>
      <td><strong>印刷廠廠長</strong> ⚙️</td>
      <td style="white-space: nowrap;">
        <code>gh run list &amp;&amp; gh run watch</code><br>
        <code>git clone --depth 1 --filter=blob:none https://github.com/org/repo.git</code><br>
        <code>git reflog</code><br>
        <code>git filter-repo --path secret.env --invert-paths</code>
      </td>
      <td>監控 GitHub Actions 建置、極速拉取代碼、挽救誤刪、抹除洩漏金鑰。</td>
    </tr>
  </tbody>
</table>

---

## 3. 這兩行 PR 指令「由誰來執行？」(Who Runs What?)

```powershell
gh pr create --fill                  # 誰執行？👉 寫這段代碼的作者 (你 🙋‍♂️)
gh pr merge --squash --delete-branch # 誰執行？👉 審查者/主編 (團隊 👨‍💼) 或 自己 (個人專案 🙋‍♂️)
```

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 8%; text-align: center;">步驟</th>
      <th style="width: 46%;">執行指令 (Command)</th>
      <th style="width: 23%;">團隊環境執行者 (Team)</th>
      <th style="width: 23%;">個人專案執行者 (Solo)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center;"><strong>步驟 1</strong></td>
      <td style="white-space: nowrap;"><code>gh pr create --fill --base master</code></td>
      <td><strong>代碼作者 (你 🙋‍♂️)</strong><br>功能寫完並推上 GitHub 後發起審查。</td>
      <td><strong>你自己 🙋‍♂️</strong><br>建立 PR 觸發雲端自動測試。</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>步驟 2</strong></td>
      <td><em>(人眼代碼審查與 CI 自動化驗證)</em></td>
      <td><strong>隊友或架構師 (Reviewer 👨‍💼)</strong><br>逐行檢查並點擊 <strong>Approve</strong>。</td>
      <td><strong>你自己 🙋‍♂️</strong><br>在網頁上做最後自我審視。</td>
    </tr>
    <tr>
      <td style="text-align: center;"><strong>步驟 3</strong></td>
      <td style="white-space: nowrap;"><code>gh pr merge --squash --delete-branch</code></td>
      <td><strong>資深架構師 / Tech Lead 👨‍💼</strong><br>（企業主幹防護，禁止球員兼裁判）。</td>
      <td><strong>你自己 🙋‍♂️</strong><br>看到 CI 綠燈 ✅ 後自己執行合併。</td>
    </tr>
  </tbody>
</table>

---

## 4. 這是給團隊還是個人用的？（團隊與個人雙重價值）

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 16%;">應用維度</th>
      <th style="width: 42%;">多人團隊協作 (Teamwork - 必備守則)</th>
      <th style="width: 42%;">個人獨立開發 (Solo Dev - 資深習慣)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>防呆與把關</strong></td>
      <td><strong>同行代碼審查 (Peer Review)</strong>：由隊友或架構師逐行抓出潛在 Bug 與效能漏洞，防止主幹被改壞。</td>
      <td><strong>自我最後審視 (Self-Review)</strong>：在發出 PR 前做最後檢查，常能即時發現忘記刪除的測試密碼或 <code>console.log</code>。</td>
    </tr>
    <tr>
      <td><strong>自動化檢驗</strong></td>
      <td><strong>CI/CD 雲端守門員</strong>：GitHub Actions 自動執行所有單元測試與安全掃描，全綠燈才允許合併。</td>
      <td><strong>乾淨環境驗證</strong>：確保程式碼在雲端乾淨環境中也能成功打包編譯，避免「我電腦跑得動但別人跑不動」。</td>
    </tr>
    <tr>
      <td><strong>歷史與紀錄</strong></td>
      <td><strong>責任歸屬與討論串</strong>：誰批准合併、為什麼這樣設計，都在 PR 討論串中永久留存。</td>
      <td><strong>完美發布節點</strong>：主幹 <code>master</code> 上永遠只有一條乾淨的直線，每個節點都是一項完工驗證過的大功能。</td>
    </tr>
  </tbody>
</table>

```
[ 個人本機自由寫作 ]  ---> 用 git  (在座位上隨意打草稿、commit、切換分支)
         |
[ 準備併入正式主幹 ]  ---> 用 PR / gh  (透過審查機制與雲端自動化，確保達到生產上線品質)
```

---

## 5. 寫書實戰中的端到端「接力合作」流程

```powershell
# --- [前半段：在座位上用打字機 (git) 寫作並寄出草稿 (由你/作者執行)] ---
git checkout -b feat/chapter-3      # [git] 拿出新章節草稿紙
git add .                           # [git] 將寫好的這幾頁夾進資料夾
git commit -m "feat: 完成第三章初稿" # [git] 蓋上個人時間戳記存檔
git push -u origin feat/chapter-3   # [git] 透過郵局把草稿寄送到 GitHub 出版社總部

# --- [後半段：拿起對講機 (gh) 指示出版社總部進行審查與印製] ---
gh pr create --fill                 # [你/作者執行] 呼叫對講機：「第三章已寄達，請開始審稿與跑自動校對！」
# (等待隊友審核通過 / 個人專案測試綠燈...)
gh pr merge --squash --delete-branch# [主編/審查者執行] 呼叫對講機：「審核通過！請將第三章精煉後正式印入定稿本，並把舊草稿銷毀。」
```

---

## 6. 遠端架構思維模型 (Remote Architecture)

遠端儲存庫（Remote）本質上就是指向雲端出版社伺服器 URL 的別名指針（慣例預設名稱為 `origin`）。

```
[ 本機書桌儲存庫 (Local) ]  <=== (git push 郵寄 / git pull 收取) ===>  [ 雲端出版社總庫 (origin) ]
```

---

## 7. 連接與管理遠端庫 (Remote Management)

### 查看遠端出版社位址 (`git remote -v`)
```powershell
git remote -v
# 輸出範例：
# origin  https://github.com/billy1030/mds.git (fetch 下載通道)
# origin  https://github.com/billy1030/mds.git (push 上傳通道)
```

### 新增遠端庫 (`git remote add`)
```powershell
git remote add origin https://github.com/billy1030/mds.git
```

### 修改遠端庫位址 (`git remote set-url`)
```powershell
# 切換為 SSH 安全通訊協定
git remote set-url origin git@github.com:billy1030/mds.git
```

---

## 8. 推送章節至雲端出版社 (`git push`)

### 初次寄送並綁定上游章節 (Upstream Tracking `-u`)
`-u`（`--set-upstream`）會將你書桌上的草稿分支與出版社同名分支進行持續綁定追蹤：
```powershell
git push -u origin master
```

### 後續常規推送
一旦綁定後，後續只需輸入：
```powershell
git push
```

---

## 9. 抓取與拉取出版社更新 (Fetch vs Pull)

### A. 抓取 (Fetch) vs 拉取 (Pull) 的差異
- **`git fetch`**：只負責從出版社下載最新目錄與隊友進度清單，**完全不會改動**你書桌上正在寫的草稿紙（安全預覽）。
- **`git pull`**：等同於 `git fetch` 加上 `git merge`，會立刻把出版社最新定稿合併進你手邊的檔案中。

### B. 推薦的安全同步工作流 (Safe Pull Workflow)
```powershell
# 1. 抓取所有最新進度，並清理已作廢的章節指標 (Prune)
git fetch --all --prune

# 2. 快進合併拉取 (Fast-Forward Only，若有衝突會安全中止，不弄髒本地草稿)
git pull --ff-only
```

---

## 10. 個人基礎分支操作 (Basic Branching)

即使是個人寫作，也建議為新章節建立獨立草稿紙，保護穩定的出版定稿本 `master`：

```powershell
# 拿出新的草稿紙並開始寫作 (Create & Switch Branch)
git checkout -b feature-dark-mode
# (現代 Git 語法推薦): git switch -c feature-dark-mode

# 寫作、暫存並蓋章存檔...
git commit -m "feat: 完成夜間深色閱讀模式章節"

# 切換回主幹定稿本
git checkout master
# (現代 Git 語法推薦): git switch master

# 將寫好的章節併入定稿本 (Merge)
git merge feature-dark-mode

# 合併完成後將草稿紙碎掉 (Delete Branch)
git branch -d feature-dark-mode
```
