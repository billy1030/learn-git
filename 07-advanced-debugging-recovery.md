# Level 06: 資深工程師除錯與災難復原 (Debugging & Recovery)

本章介紹在手殘誤刪代碼時如何瞬間挽回 (Reflog)、如何用二分法瞬間找出引入 Bug 的提交 (Bisect)，以及精準追蹤代碼演變歷史。

---

## 1. 引用日誌 (`git reflog` — 終極時光機與後悔藥)

Git 本機會記錄您每一次的指標變更動作（無論是 commit、checkout、hard reset 甚至刪除分支）。

```powershell
# 1. 查看本機完整的指標異動日誌 (Reflog)
git reflog

# 2. 直接 Hard Reset 重設回犯錯前的一瞬間：
git reset --hard HEAD@{1}
```

---

## 2. 二分搜尋法抓出 Bug (`git bisect` — Binary Search Debugging)

當某個功能本週壞了，但你不知道在過去 100 筆 Commit 中究竟是誰改壞的：

```powershell
# 1. 啟動二分搜尋模式
git bisect start

# 2. 標記目前版本是壞的
git bisect bad

# 3. 標記某個歷史版本是正常的
git bisect good v1.0.0

# 4. Git 會自動將歷史折半切出中間的 Commit 給你測試。告訴 Git：
git bisect good   # (若該版本測試正常)
# 或
git bisect bad    # (若該版本就已經壞了)

# 5. 重複大約 5~6 次折半測試後，Git 會明確指出出錯的 Commit！

# 6. 結束除錯並回到原點：
git bisect reset
```

---

## 3. 除錯與歷史追溯指令速查表 (Debugging Command Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 16%;">除錯需求</th>
      <th style="width: 50%;">標準指令 (Command - 單行完整)</th>
      <th style="width: 34%;">中文作用說明與比喻</th>
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
