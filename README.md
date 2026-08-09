# EPUB / TXT 繁簡轉換工具 — 整合說明

整合了以下兩個 repo：
- `Vincent981010/epub-simplified-to-traditional-chinese`（下稱 A）
- `Vincent981010/epub-tools`（下稱 B）

依照需求，**EPUB 的簡轉繁核心處理邏輯完全採用 A 的寫法**，B 的其他功能（TXT 支援、
OpenCC 模式選擇、自訂字詞替換規則）則整合進來。

## 這次整合做了什麼

### 完全沿用 A 的部分（EPUB 處理核心）
- `processSingleEpub` 的整體流程原封不動：
  - `mimetype` 檔案原樣保留，不經過轉換
  - 會處理的文字類副檔名：`html / xhtml / htm / ncx / opf / xml / txt / css`
    （比 B 原本的範圍更廣，B 少了 `xml` 和 `css`）
  - 轉換前先對 `href` / `src` 屬性做 `decodeURIComponent`，避免轉換破壞
    百分號編碼的網址字元
  - 轉換後自動把 `<?xml encoding=...?>`、`<meta charset=...>` 修正為 `utf-8`
  - **檔名與資料夾名稱也會一併翻譯**（B 原本沒有這個行為）
- 拖曳區＋深度資料夾遞迴掃描（`webkitGetAsEntry` / `createReader`）也是沿用 A，
  B 原本完全沒有拖曳功能

### 整合自 B 的部分
- 支援 `.txt` 檔案（沿用 B 的簡單讀取/寫入邏輯）
- OpenCC 轉換模式可選：`s2twp / s2tw / s2t / t2s / 不轉換`
  （A 原本寫死只有 `s2twp` 一種）
- 可自訂多組「找字詞 → 換字詞」規則，於 OpenCC 轉換**之後**套用

### 我做的整合設計決策（原本兩個 repo 都沒有明確處理）
1. **自訂字詞替換規則只套用在檔案內容，不套用在檔名／路徑上**
   　避免使用者填的替換規則不小心把檔名弄壞（例如產生過長或非法路徑）。
   　檔名/路徑只會套用 OpenCC 轉換。
2. **檔名後綴依轉換方向動態決定**：
   - 簡轉繁（`s2twp` / `s2tw` / `s2t`）→ 加 `_TW`
   - 繁轉簡（`t2s`）→ 加 `_CN`
   - 不轉換（`none`，只套字詞替換規則）→ 加 `_converted`
   　（A 原本寫死一律加 `_TW`，因為它只支援簡轉繁一個方向）
3. **資料夾模式保留所有檔案以維持結構完整**（沿用 B 的行為）：
   　非 `.epub` / `.txt` 的檔案（圖片、字型等）會原封不動放回輸出的 zip，
   　只有檔案本身是 epub/txt 時才會翻譯該層檔名。
   　拖曳單一/多個「零散檔案」（非資料夾）時則只保留 `.epub` / `.txt`。

## 檔案結構
整合後只有一個 `index.html`，純前端、無需後端伺服器。相依套件改為**本地檔案引入**
（不再用 CDN）：

```html
<script src="jszip.min.js"></script>
<script src="opencc-js.min.js"></script>
```

請把 A repo（`epub-simplified-to-traditional-chinese`）裡原本就有的
`jszip.min.js` 和 `opencc-js.min.js` 這兩個檔案，複製到跟這份 `index.html`
**同一個資料夾**底下，三個檔案放在一起即可離線使用，不再依賴外部 CDN。

## 使用方式
1. 選擇 OpenCC 轉換模式（預設「簡體 ➔ 臺灣正體，含常用詞彙修正」）
2. 視需要加入自訂字詞替換規則
3. 拖曳檔案/資料夾，或用按鈕選擇「多個檔案」/「整個資料夾」
4. 單一/多檔案模式會逐一觸發下載；資料夾模式會打包成一個 zip 下載

## 建議後續動作
- 我沒有這兩個 repo 的寫入權限，所以無法直接幫你建立新 repo 或發 PR。
　你可以把 `index.html` 直接放進 A 或 B 任一個 repo 覆蓋原檔案，或建立一個新 repo。
- 如果你想要保留 A、B 兩個 repo 分開維護，也可以把這個整合版另外開一個新 repo
　（例如 `epub-tools-merged`）。
