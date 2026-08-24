# Level 05: 精準手術刀工具：Cherry-Pick、區塊暫存與互動式變基 (Surgical Tools)

本章介紹如何在不污染分支的前提下移植特定提交 (Cherry-Pick)、挑選單一檔案內部特定行數暫存 (Patch Staging)，以及整理重構提交歷史 (Interactive Rebase)。

---

## 1. 挑選提交 (`git cherry-pick` — 跨分支精準移植 Bug 修復)

### 核心概念 (The Concept)
當你在 `2.0` 分支修復了一個 Bug，但 `1.0`（`master`）也同樣存在該 Bug 時；若使用 `merge` 會把 `2.0` 未完成的新功能一起帶進 `1.0` 造成災難。
而 **`git cherry-pick`** 允許您只「摘取」該筆修復 Commit 的代碼變更，精準移植至 `1.0`。

```
[v2.0 分支]  --- (Bug 修復 Commit: a1b2c3d) ---> (繼續開發 2.0 新功能...)
                                |
                        git cherry-pick a1b2c3d
                                |
                                v
[master 分支] ------------------ (僅將該 Bug 修復精準套入 1.0)
```

---

## 2. 避免 2.0 功能洩漏的 4 步驟防護清單 (Leak Prevention)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 12%; text-align: center;">步驟</th>
      <th style="width: 48%;">執行指令 (Command - 單行完整)</th>
      <th style="width: 40%;">中文防護動作說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>步驟 1</strong></td>
      <td style="white-space: nowrap;"><code>git add backend/src/services/billing.ts &amp;&amp; git commit -m "fix(billing): fix tax calc"</code></td>
      <td><strong>建立獨立原子提交</strong>：切勿將修復與 2.0 新功能一起打包。</td>
    </tr>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>步驟 2</strong></td>
      <td style="white-space: nowrap;"><code>git show a1b2c3d</code></td>
      <td><strong>事前檢視差異</strong>：確認該 Commit 內 100% 只有修復代碼。</td>
    </tr>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>步驟 3</strong></td>
      <td style="white-space: nowrap;"><code>git cherry-pick -n a1b2c3d</code></td>
      <td><strong>無提交預覽模式</strong>：套用至本機工作區但不自動 Commit，方便人眼檢查。</td>
    </tr>
    <tr>
      <td style="text-align: center; white-space: nowrap;"><strong>步驟 4</strong></td>
      <td style="white-space: nowrap;"><code>git diff &amp;&amp; git commit -m "fix: cherry-picked from v2.0"</code></td>
      <td><strong>測試後正式提交</strong>；若發現夾帶不想要的代碼可執行 <code>git cherry-pick --abort</code>。</td>
    </tr>
  </tbody>
</table>

---

## 3. 互動式代碼區塊暫存 (`git add -p` — Patch Staging)

當同一個檔案內有 10 處修改，但您只想把其中 3 處納入當次 Commit 時：

```powershell
git add -p backend/src/index.ts
```

Git 會逐一展示每個修改區塊 (Hunk) 並詢問：
- **`y`** (Yes)：暫存此區塊。
- **`n`** (No)：跳過此區塊（留在工作目錄，不暫存）。
- **`s`** (Split)：將當前區塊進一步細分成更小的行數片段。
- **`q`** (Quit)：結束暫存。

---

## 4. 互動式變基整理歷史 (`git rebase -i` — Interactive Rebase)

在發出 Pull Request 前，將本機零碎的暫存 Commit 整理成乾淨優雅的提交：

```powershell
# 整理最近 4 筆提交
git rebase -i HEAD~4
```

編輯器會列出清單：
```text
pick e1a2b3c feat: 新增驗證路由
squash d4e5f6g 修復驗證拼字錯誤
squash a7b8c9d 補充測試
pick 3d2e1f0 docs: 更新 API 文件
```

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 15%;">指令動作</th>
      <th style="width: 25%;">簡寫與語法</th>
      <th style="width: 60%;">作用說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>pick</strong></td>
      <td><code>pick &lt;hash&gt;</code> / <code>p</code></td>
      <td>保留該筆提交原貌不變。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>squash</strong></td>
      <td><code>squash &lt;hash&gt;</code> / <code>s</code></td>
      <td>將該提交<strong>壓入上一筆提交</strong>，並保留/合併兩者的備註訊息。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>fixup</strong></td>
      <td><code>fixup &lt;hash&gt;</code> / <code>f</code></td>
      <td>將該提交<strong>壓入上一筆提交</strong>，並直接捨棄本筆備註（最常用於小修補）。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>reword</strong></td>
      <td><code>reword &lt;hash&gt;</code> / <code>r</code></td>
      <td>修改該筆提交的備註訊息。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>drop</strong></td>
      <td><code>drop &lt;hash&gt;</code> / <code>d</code></td>
      <td>徹底刪除該筆提交。</td>
    </tr>
  </tbody>
</table>
