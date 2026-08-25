# V2.3｜嚴格測試報告

測試日期：2026-08-25

## 結論

本版完成靜態檢查、DOM 執行測試、惡意資料測試、數學組合測試與多尺寸版面測試。此次測試範圍內未發現阻斷性 JavaScript 錯誤，V2.3 新增功能均通過測試。

> 測試環境限制：目前執行環境會阻擋 Chromium 直接瀏覽 `127.0.0.1`，因此互動測試以 Chromium DevTools `Page.setDocumentContent` 載入完整 DOM/CSS/JS 進行。Service Worker 的實際 HTTP 註冊流程無法在此環境做端到端註冊測試，改以語法、快取清單與更新流程做靜態驗證。

## 1. 靜態完整性檢查

- 35 / 35 項：PASS
- HTML ID：167 個，全部唯一
- JavaScript `#id` 引用：151 個，缺漏 0
- 導覽目標：全部存在
- `node --check app.js`：PASS
- `node --check sw.js`：PASS
- Manifest JSON：PASS
- PWA 192×192 / 512×512 圖示尺寸：PASS
- 無 `eval()`、`new Function()`、`document.write()`
- 無 HTML inline event handler
- 無外部 runtime CDN / script / stylesheet
- `app.js` 已加入 Service Worker 核心快取
- CSP 已設定 `script-src 'self'`，未允許 `unsafe-inline` JavaScript

## 2. A4 預覽一致性

100% 預覽實測：

- 寬：793.6875 px
- 高：1122.515625 px
- 高寬比：1.414304
- A4 理論比例 297 / 210 ≈ 1.414286
- 結果：PASS

V2.3 使用固定 210mm × 297mm A4 內容建立預覽，再對整頁縮放；不再為了符合右側欄寬重新排列格子。

另發現並修正一個預覽縮放競態：切換成 100% 後，先前排程的「適合寬度」可能再次覆蓋使用者選擇。現已在延遲重新計算前再次確認目前模式。

## 3. 指定頁列印

以 0～20 數字練習產生 4 頁測試：

- 預覽頁數：4
- 頁面選擇器：4
- 僅勾第 2 頁後，列印前檢查顯示「已選 1 頁」
- 建立列印 Portal 後實際保留頁數：1
- 結果：PASS

## 4. 數學智慧練習

完整跑過：

- 題型：加法 / 減法 / 混合，共 3 種
- 範圍：0～10 / 20 / 50 / 100，共 4 種
- 題數：10 / 20 / 30 / 40 / 50，共 5 種
- 合計：3 × 4 × 5 = 60 組

60 / 60 組全部通過：

- 題數正確
- 同份算式不重複
- 加法結果不超過設定範圍
- 減法不產生負數
- 混合模式加減題數差不超過 1 題
- 題目導航數量與設定題數一致
- Runtime error：0

數學導航另測：

- 20 題 → 20 個導航按鈕
- 答錯後導航正確顯示 wrong 狀態
- 可直接跳至第 2 題
- 快速連按「完成這一題」兩次，只增加 5 點，不會重複計分

## 5. 惡意資料 / XSS 動態測試

將孩子名稱、ID、禮物名稱刻意放入 HTML / event-handler 字串：

- `window.__xss`：0
- 惡意孩子 ID 被重新產生為安全 token
- 名稱只以文字顯示，未建立惡意 `<img>` / `<svg>` DOM
- `Object.prototype` 未被污染
- 惡意作品圖片字串在備份匯入時被拒絕
- 匯入測試後 `window.__xss` 仍為 0
- Runtime error：0

## 6. 多尺寸版面測試

| Viewport | 整頁水平溢出 | 左側工作台可到最底 |
|---|---|---|
| 1440×900 | PASS | PASS |
| 1024×768 | PASS | PASS |
| 820×1180 | PASS | 使用整頁自然捲動 |
| 390×844 | PASS | 使用整頁自然捲動 |

桌機 1440×900 左側工作台：clientHeight 682、scrollHeight 1517，捲至 scrollTop 835 後最後說明仍完整可見。

## 7. PWA / 更新流程

- Service Worker cache：`kids-writing-v2.3-20260825-1`
- 僅核心檔案進入 Cache Storage
- 非核心同源 GET 不再被 Service Worker 無限制快取
- install 階段不再自動 `skipWaiting()`
- 新版會進入 waiting，使用者按「立即更新」後才送出 `SKIP_WAITING`
- 舊 cache 於 activate 清理
- `app.js` 已納入離線核心資源

## 8. 已知環境差異

不同作業系統的字型實際 glyph metrics 仍可能略有差異，但 A4 容器、格線、頁面尺寸及換頁邏輯已統一。實際印表機仍可能受到印表機驅動程式「縮放至可列印範圍」設定影響，建議列印對話框使用 100% / 實際大小。
