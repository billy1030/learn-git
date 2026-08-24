# Level 14: 超大型二進位檔案與 AI 模型管理 (Git LFS & Large Assets)

本章介紹在遊戲專案、機器學習 (ML/AI) 模型權重、4K 影音素材與大型資料集開發中，如何使用 **Git LFS (Large File Storage)** 避免儲存庫膨脹至數十 GB 導致下載卡死。

---

## 1. 核心思維模型：為什麼一般 Git 不能存大檔案？ (The Storage Locker Analogy)

> **寫書與光碟比喻**：
> - **傳統 Git 直接存大檔（災難做法）**：你在小說每一頁後面都黏一張 4K 原畫光碟（10GB）。每當你改一個錯字 Commit 一次，Git 就會**重新複製整張 10GB 光碟**存進歷史紀錄。才寫了 5 個版本，專案體積就變成 50GB，任何人 `git clone` 都會下載到電腦當機。
> - **Git LFS（超輕量置物櫃指標）**：
>   1. **實體大檔案上傳雲端置物櫃**：10GB 的模型檔案被直接送到專屬的雲端大型儲存庫（AWS S3 / GitHub LFS）。
>   2. **Git 內只存 1KB 的提貨券（Text Pointer）**：你的 Git 歷史中只保存短短 3 行文字（記錄 SHA256 校驗碼與檔案大小）。專案體積永遠保持數十 MB 極速下載！

---

## 2. Git LFS 常用核心操作指令表 (Git LFS Command Reference)

<table style="width: 100%; border-collapse: collapse;">
  <thead>
    <tr>
      <th style="width: 18%;">操作動作</th>
      <th style="width: 52%;">標準指令 (Command)</th>
      <th style="width: 30%;">中文作用說明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="white-space: nowrap;"><strong>1. 初始化 LFS 環境</strong></td>
      <td style="white-space: nowrap;"><code>git lfs install</code></td>
      <td>在當前電腦的 Git 環境中啟用 LFS 大檔攔截鉤子。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>2. 追蹤特定大檔案類型</strong></td>
      <td style="white-space: nowrap;"><code>git lfs track "*.onnx" "*.bin" "*.mp4" "*.zip"</code></td>
      <td>將指定副檔名納入 LFS 管理（會自動寫入 <code>.gitattributes</code>）。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>3. 提交設定檔</strong></td>
      <td style="white-space: nowrap;"><code>git add .gitattributes &amp;&amp; git commit -m "chore: track LFS models"</code></td>
      <td>將 LFS 規則檔案正式加入版本控制。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>4. 檢視目前被 LFS 追蹤的大檔</strong></td>
      <td style="white-space: nowrap;"><code>git lfs ls-files</code></td>
      <td>列出專案中所有被 LFS 提貨券指標接管的實體大檔。</td>
    </tr>
    <tr>
      <td style="white-space: nowrap;"><strong>5. 按需拉取大檔案實體</strong></td>
      <td style="white-space: nowrap;"><code>git lfs pull</code></td>
      <td>從雲端置物櫃下載真實的二進位大檔（未下前僅為 1KB 指標）。</td>
    </tr>
  </tbody>
</table>

---

## 3. Git LFS 底層提貨券指針檔案原理 (Pointer File Anatomy)

當你使用 LFS 追蹤一個 5GB 的模型檔案 `model.onnx` 時，Git 儲存庫內實際保存的內容僅有 130 位元組：

```text
version https://git-lfs.github.com/spec/v1
oid sha256:4b97a213e30f14da2e316d3fdf6c63e26466f9de58f1f8b652870428d0eb4b78
size 5368709120
```
*這就是為什麼儲存庫能永遠保持極速 Clone 與輕盈！*
