# Level 07: 資深工程師除錯與災難復原 (Debugging & Recovery)

本章介紹在手殘誤刪代碼時如何瞬間挽回 (Reflog)、如何用二分法瞬間找出引入 Bug 的提交 (Bisect)，以及精準追蹤代碼演變歷史。

---

## 1. 核心思維模型：黑盒子飛行記錄器與探案抓兇手 (The Black Box & Detective Analogy)

> **生活化比喻**：
> - **`git reflog`（飛機的黑盒子飛行記錄器 ✈️）**：一般 `git log` 只看得到目前留存的歷史；而 `reflog` 就像飛機的黑盒子，**記錄了你在本機敲下的「每一個按鍵動作」**（哪怕你手殘執行了 `git reset --hard` 把代碼全部刪光，黑盒子裡面依然完好保存著前一秒的時空座標，一秒穿越時空挽回！）。
> - **`git bisect`（福爾摩斯二分搜尋抓兇手 🕵️‍♂️）**：過去一個月有 100 筆 Commit，某天系統突然當機。你不需要肉眼逐行檢查 100 次，Git 會像猜數字遊戲一樣，第 1 次測第 50 筆、第 2 次測第 25 筆，**只要 6~7 次測試，精準揪出寫出 Bug 的兇手 Commit！**

---

## 2. 三大高頻災難復原與除錯實戰範例 (3 Real-World Use Cases)

### 場景 1：手滑執行 `git reset --hard` 搞丟了一整天的心血
- **實戰挽救**：
```powershell
# 1. 檢視黑盒子動作紀錄
git reflog
# 輸出：a1b2c3d HEAD@{0}: reset: moving to HEAD~5
#       e4f5g6h HEAD@{1}: commit: 完成一整天寫好的購物車功能

# 2. 一秒穿越時空還原到 HEAD@{1}：
git reset --hard HEAD@{1}
```

---

### 場景 2：二分搜尋法抓出神秘 Bug (Bisect Walkthrough)
- **實戰除錯**：
```powershell
# 1. 啟動二分偵探模式
git bisect start
git bisect bad            # 告訴 Git 目前最新版壞掉了
git bisect good v1.0.0    # 告訴 Git 一個月前的 v1.0.0 是好的

# 2. Git 自動切換到中間的 Commit，你執行測試後回報：
npm test                  # 測試若失敗
git bisect bad            # 告訴 Git 這版也是壞的

# 3. 重複測試 5 次後，Git 直接印出：
# 👉 "a1b2c3d is the first bad commit (由小明在 8/12 提交)"

# 4. 結束調查並回歸原本分支：
git bisect reset
```

---

### 場景 3：追查特定函數是誰在何時寫的 (`git log -S` 十字鎬搜尋)
- **實戰範例**：想找出 `calculateTaxRate()` 這個函數最早是在哪一次提交中被寫入專案庫的：
```powershell
git log -S "calculateTaxRate" --oneline -p
```

---

## 3. 除錯與歷史追溯指令速查表 (Debugging Command Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 16%;">除錯需求</th>
      <th style="width: 50%;">標準指令 (Command)</th>
      <th style="width: 34%;">中文作用說明與效益</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 災難復原</strong></td>
      <td style="white-space: nowrap;"><code>git reset --hard HEAD@{1}</code></td>
      <td>瞬間挽救誤刪的分支或誤執行的 hard reset。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 二分抓兇手</strong></td>
      <td style="white-space: nowrap;"><code>git bisect start &amp;&amp; git bisect bad &amp;&amp; git bisect good v1.0</code></td>
      <td>用二分搜尋法快速找出哪筆 Commit 引入了 Bug。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 字串異動搜尋</strong></td>
      <td style="white-space: nowrap;"><code>git log -S "verifyGoogleToken" -p</code></td>
      <td>搜尋哪筆 Commit 首次新增或刪除了特定函數或字串。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 正規表達式搜尋</strong></td>
      <td style="white-space: nowrap;"><code>git log -G "API_KEY_[0-9]+" -p</code></td>
      <td>使用 Regex 在全庫歷史中搜尋敏感字眼變更。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 逐行責任追溯</strong></td>
      <td style="white-space: nowrap;"><code>git blame -L 40,60 backend/src/services/backup.service.ts</code></td>
      <td>查看特定行數區間內每一行是由誰在何時修改的。</td>
    </tr>
  </tbody>
</table>
