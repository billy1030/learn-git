# Level 12: 語意化版本控制與 Release 正式發布 (SemVer & Release Management)

本章介紹現代軟體工程中的語意化版本命名標準 (Semantic Versioning)、Git 標籤 (Tag) 管理，以及如何使用 GitHub Releases 進行自動化打包與發布公告。

---

## 1. 核心思維模型：什麼是語意化版本 (SemVer: X.Y.Z)？

> **版本號格式**：`MAJOR.MINOR.PATCH`（例如：`v2.1.4`）

```text
       v 2 . 1 . 4
         │   │   └── PATCH (修訂版/補丁) : 向下相容的 Bug 修復 (例如修正計算溢位)
         │   └────── MINOR (次版本/新功能) : 向下相容的新功能 (例如新增 Google Drive 備份功能)
         └────────── MAJOR (主版本/大改版) : 包含重大破壞性變更 (Breaking Changes)，不相容舊版 API
```

---

## 2. Git 標籤管理 (Git Tagging Reference)

標籤 (Tag) 就像給特定歷史節點貼上一張「永久唯讀的金色書籤」，代表此處為正式發行版本：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 18%;">操作需求</th>
      <th style="width: 52%;">標準指令 (Command)</th>
      <th style="width: 30%;">中文作用說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 建立附註標籤</strong></td>
      <td style="white-space: nowrap;"><code>git tag -a v1.0.0 -m "Release version 1.0.0 正式版上線"</code></td>
      <td>建立帶有作者簽名、時間與備註的正式標籤。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 查看所有標籤</strong></td>
      <td style="white-space: nowrap;"><code>git tag -l -n3</code></td>
      <td>列出目前本機所有標籤及其說明文字。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 推送單一標籤</strong></td>
      <td style="white-space: nowrap;"><code>git push origin v1.0.0</code></td>
      <td>將指定的版本標籤上傳到 GitHub。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 推送所有本機標籤</strong></td>
      <td style="white-space: nowrap;"><code>git push origin --tags</code></td>
      <td>一口氣將本機所有新建標籤全部推上雲端。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 刪除標籤 (本機+雲端)</strong></td>
      <td style="white-space: nowrap;"><code>git tag -d v1.0.0 &amp;&amp; git push origin :refs/tags/v1.0.0</code></td>
      <td>若打錯版本號，徹底從本機與遠端刪除該 Tag。</td>
    </tr>
  </tbody>
</table>

---

## 3. 使用 GitHub CLI (`gh`) 一鍵發布正式 Release

GitHub Release 允許你附帶自動生成的更新日誌 (Changelog) 與執行檔資產 (Assets)：

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 20%;">發布需求</th>
      <th style="width: 54%;">GitHub CLI 指令 (Command)</th>
      <th style="width: 26%;">說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>自動生成更新日誌發布</td>
      <td style="white-space: nowrap;"><code>gh release create v1.0.0 --generate-notes</code></td>
      <td>自動分析所有 PR 並產生發布公告</td>
    </tr>
    <tr>
      <td>附帶二進位安裝檔發布</td>
      <td style="white-space: nowrap;"><code>gh release create v1.0.0 app-setup.exe --notes "正式上線"</code></td>
      <td>上傳安裝檔以供使用者直接下載</td>
    </tr>
    <tr>
      <td>發布預覽測試版 (Pre-release)</td>
      <td style="white-space: nowrap;"><code>gh release create v2.0.0-beta.1 --prerelease</code></td>
      <td>標記為 Beta 測試版，不干擾穩定版</td>
    </tr>
    <tr>
      <td>檢視最近發布清單</td>
      <td style="white-space: nowrap;"><code>gh release list</code></td>
      <td>列出雲端目前已發布的所有版本</td>
    </tr>
  </tbody>
</table>
