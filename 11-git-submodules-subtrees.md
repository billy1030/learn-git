# Level 11: 跨專案模組共用與子儲存庫 (Git Submodules & Subtrees)

本章介紹如何在一個 Git 儲存庫中乾淨地引用另一個獨立的 Git 儲存庫（例如前後端共用型別庫、共用 UI 元件庫或第三方核心外掛），並保持獨立更新。

---

## 1. 核心思維模型：寫書中的「附錄字典」比喻 (Submodule Analogy)

> **寫書比喻**：
> - **傳統複製貼上（災難做法）**：你在寫小說時，直接把一本 500 頁的「常用英漢字典」影印貼進你的每一本書後面。一旦字典修訂版改了錯字，你必須手動去改所有的小說，而且小說檔案變得無比臃腫。
> - **Git Submodule（優雅引用）**：你的小說末頁只附帶一張**「條碼借書卡」**，上面記錄著：「請向字典出版社借閱 `v2.1` 版字典，放置於 `./lib/dictionary` 資料夾」。
>   1. **獨立版本庫**：字典有自己的 Git 歷史與維護者。
>   2. **精準鎖定 Commit**：主專案明確記錄引用子模組的特定 Commit Hash，不會因對方隨意改動而意外壞掉。

---

## 2. Submodule 常用核心操作指令表 (Submodule Command Reference)

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
      <td style="white-space: nowrap;"><strong>1. 引入外部子模組</strong></td>
      <td style="white-space: nowrap;"><code>git submodule add https://github.com/org/shared-utils.git lib/utils</code></td>
      <td>將外部儲存庫掛載到專案的指定路徑中。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 初次 Clone 包含子模組</strong></td>
      <td style="white-space: nowrap;"><code>git clone --recurse-submodules https://github.com/org/main-app.git</code></td>
      <td>下載主專案的同時，自動遞迴拉取所有子模組代碼。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 既有專案補拉子模組</strong></td>
      <td style="white-space: nowrap;"><code>git submodule update --init --recursive</code></td>
      <td>初始化並下載專案中尚未拉取的子模組檔案。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 同步遠端子模組最新代碼</strong></td>
      <td style="white-space: nowrap;"><code>git submodule update --remote --merge</code></td>
      <td>抓取子模組遠端的最新更新並合併進主專案。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 移除子模組</strong></td>
      <td style="white-space: nowrap;"><code>git submodule deinit -f lib/utils &amp;&amp; git rm -f lib/utils</code></td>
      <td>乾淨解除註冊並刪除子模組目錄與 <code>.gitmodules</code> 紀錄。</td>
    </tr>
  </tbody>
</table>

---

## 3. Submodule vs Subtree 技術選型對比

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 20%;">比較維度</th>
      <th style="width: 40%;">Git Submodule (指針引用型)</th>
      <th style="width: 40%;">Git Subtree (實體融入型)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>代碼存放方式</strong></td>
      <td>主專案只記錄一個 Commit Hash 指標，檔案按需下載。</td>
      <td>直接將外部代碼完整合入主專案歷史，像原生代碼一樣存在。</td>
    </tr>
    <tr>
      <td><strong>Clone 複雜度</strong></td>
      <td>需要帶 <code>--recurse-submodules</code> 否則資料夾會是空的。</td>
      <td>隊友正常 <code>git clone</code> 即可看到完整檔案，零心智負擔。</td>
    </tr>
    <tr>
      <td><strong>適用場景</strong></td>
      <td>大型獨立套件庫、私有閉源核心庫、微服務模組共用。</td>
      <td>第三方開源庫小幅修補、只想單純引入不想管子模組設定。</td>
    </tr>
  </tbody>
</table>
