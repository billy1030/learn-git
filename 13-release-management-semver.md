# Level 13: 語意化版本控制與 Release 正式發布 (SemVer & Release Management)

本章介紹現代軟體工程中的語意化版本命名標準 (Semantic Versioning)、Git 標籤 (Tag) 管理，以及如何使用 GitHub Releases 進行自動化打包與發布公告。

---

## 1. 核心思維模型：新書正式印刷與封條條碼比喻 (The Book Release Analogy)

> **生活化比喻**：
> - **語意化版本 (SemVer 條碼 🏷️)**：就像圖書館與書店的書籍修訂版號：
>   - **`PATCH` (第三位數字 `v1.0.1`)**：只修復了第 42 頁的一個錯字，讀者完全不需要重新學習如何閱讀這本書（向下相容修復）。
>   - **`MINOR` (第二位數字 `v1.1.0`)**：新增了「附錄習題章節」，舊讀者能照樣讀，新功能隨取隨用（向下相容新功能）。
>   - **`MAJOR` (第一位數字 `v2.0.0`)**：整本書架構大翻新、章節徹底重寫，舊讀者必須依照全新手冊重新學習（破壞性大改版 Breaking Change）。
> - **GitHub Release（新書發表會與隨書光碟 🎁）**：不僅在 Git 歷史上打上金色標籤，還自動在雲端附帶「編譯好的 `.exe` 安裝檔」與「自動生成的詳細更新日誌 (Changelog)」。

```text
       v 2 . 1 . 4
         │   │   └── PATCH (修訂版/補丁) : 向下相容的 Bug 修復 (例如修正計算溢位)
         │   └────── MINOR (次版本/新功能) : 向下相容的新功能 (例如新增 Google Drive 備份功能)
         └────────── MAJOR (主版本/大改版) : 包含重大破壞性變更 (Breaking Changes)，不相容舊版 API
```

---

## 2. 三大版本發布實戰範例 (3 Real-World Release Scenarios)

### 場景 1：修復重大 Bug，發布補丁修訂版 (Patch Release)
- **實戰需求**：修復了用戶登入 Session 提前逾時問題，需發布 `v1.2.1`：
```powershell
git tag -a v1.2.1 -m "fix(auth): fix session timeout calculation"
git push origin v1.2.1
gh release create v1.2.1 --title "v1.2.1 緊急修復版" --generate-notes
```

---

### 場景 2：新增功能，發布次版本 (Minor Release)
- **實戰需求**：完成了 Google Drive 自動備份功能，發布 `v1.3.0`：
```powershell
git tag -a v1.3.0 -m "feat(backup): add Google Drive cloud backup integration"
git push origin v1.3.0
gh release create v1.3.0 --title "v1.3.0 雲端備份功能上線" --generate-notes
```

---

### 場景 3：附帶桌面安裝檔的正式發布 (Release with Installer Assets)
- **實戰需求**：將打包好的 Windows 桌面安裝檔 `MDS-Setup.exe` 一併上傳至 GitHub Release 供使用者下載：
```powershell
gh release create v2.0.0 ./dist/MDS-Setup.exe --title "MDS v2.0.0 正式版" --notes "全新 UI 介面與十倍效能提升！"
```

---

## 3. Git 標籤管理指令速查表 (Git Tagging Reference)

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

## 4. GitHub CLI (`gh release`) 發布指令速查表

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
