# Level 06: 精準手術刀工具：Cherry-Pick、區塊暫存與互動式變基 (Surgical Tools)

本章介紹如何在不污染分支的前提下移植特定提交 (Cherry-Pick)、挑選單一檔案內部特定行數暫存 (Patch Staging)，以及整理重構提交歷史 (Interactive Rebase)。

---

## 1. 核心思維模型：果園採櫻桃與外科手術比喻 (The Orchard & Surgery Analogy)

> **生活化比喻**：
> - **一般 Merge（整棵樹連根拔起 🌳）**：就像你只想吃一顆甜櫻桃，卻把整座果園連同泥土、爛葉子與雜草全部倒進家裡客廳（把 2.0 未完成的半成品新功能一起倒進 1.0 正式版）。
> - **Git Cherry-Pick（精準摘取最甜的一顆櫻桃 🍒）**：你拿著剪刀，**只剪下「修復 Bug 的那一個 Commit」**，精準帶回 1.0 正式版貼上，其餘 2.0 的新功能 100% 留在原地！
> - **Git Add -p（外科手術刀局部縫合 🩺）**：同一個檔案寫了 100 行，但有 20 行是測試代碼、80 行是正式功能。你像外科醫生一樣**只把這 80 行縫進暫存區存檔**，不留下多餘痕跡。

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
      <th style="width: 48%;">執行指令 (Command)</th>
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

## 3. 三大高頻實戰場景範例 (3 Real-World Use Cases)

### 場景 1：線上緊急熱修復移植 (Hotfix Backporting)
- **實戰範例**：在 `v2.0-beta` 上修復了一個記憶體溢位問題（Commit `f7e8d9c`），需要立刻同步到生產線上的 `v1.0-prod`：
```powershell
git checkout v1.0-prod
git cherry-pick f7e8d9c
git push origin v1.0-prod
```

---

### 場景 2：不小心在錯誤的分支上寫了 Code (Commit on Wrong Branch)
- **實戰範例**：你本該在 `feat/auth` 分支寫作，卻手滑在 `master` 提交了 Commit `b2c3d4e`：
```powershell
# 1. 切換到正確的分支並摘取該 Commit
git checkout feat/auth
git cherry-pick b2c3d4e

# 2. 切回 master 並將指標倒退一步
git checkout master
git reset --hard HEAD~1
```

---

### 場景 3：發 PR 前將 10 個雜亂 Commit 壓成 1 個乾淨提交
- **實戰範例**：本機寫功能時提交了 `fix typo`、`wip`、`test` 5 次，發 PR 前整理為 1 個：
```powershell
git rebase -i HEAD~5
# 將第 2~5 行的 'pick' 改為 'squash' 或 'fixup'，存檔後自動合併！
```

---

## 4. 互動式代碼區塊暫存 (`git add -p` — Patch Staging)

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

## 5. 互動式變基指令速查表 (`git rebase -i`)

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
