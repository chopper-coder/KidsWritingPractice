# V2.3｜資安風險檢測報告

檢測日期：2026-08-25

## 總體評估

V2.3 為純前端、本機資料為主的兒童書寫練習工具。此次檢測針對 DOM XSS、惡意備份 JSON、原型污染、資料外傳、PWA / Service Worker、IndexedDB / localStorage、重複計分與資料完整性進行檢查。

本輪發現的可修正風險均已處理。此次測試範圍內，未發現尚未修正的高風險遠端程式碼執行或主動資料外傳機制；但仍有「本機 PIN 並非真正安全邊界」與「備份檔未加密」等設計限制，詳見下方。

## 已修正項目

### 1. 匯入資料造成 DOM XSS 的可能性 — 已修正

**原風險：中～高**

舊版部分由備份 / localStorage 還原的 ID 或作品圖片資料會進入 `innerHTML` / HTML attribute。若使用者匯入刻意製作的惡意 JSON，理論上可能形成 attribute injection / event handler 注入。

**V2.3 修正：**

- 孩子 ID、禮物 ID、作品 ID 等必須通過安全 token 白名單。
- 顯示文字統一 HTML escape。
- 手寫圖片只允許 `data:image/png`、`data:image/jpeg`、`data:image/webp` 的 Base64 格式。
- 單張匯入圖片資料限制約 2.5 MB 字串大小。
- JavaScript 從 `index.html` 拆為同源 `app.js`。
- CSP 改為 `script-src 'self'`，不再允許 inline JavaScript / inline event handler 執行。
- 動態惡意資料測試中 XSS 觸發次數為 0。

### 2. 未信任 JSON 的 Prototype Pollution 風險 — 已修正

**原風險：中**

舊版曾使用 `Object.assign(base, src)` / 類似方式合併還原資料。對完全不可信的 JSON 而言，`__proto__`、`constructor`、`prototype` 等特殊鍵不應直接合併。

**V2.3 修正：**

- 改成欄位白名單 normalization，不再把 raw state 直接 `Object.assign` 進正式狀態。
- Map 型欄位忽略 `__proto__` / `constructor` / `prototype`。
- 數值、陣列數量、模式值、格線值、孩子 ID 都重新驗證。
- 動態測試：`({}).polluted` 為 null / undefined，未出現 prototype pollution。

### 3. 惡意 / 過大備份造成資源耗盡 — 已修正

**原風險：中**

JSON 匯入原本缺乏檔案大小及大量紀錄限制，極大的 JSON 或大量 Base64 圖片可能造成平板記憶體壓力甚至頁面卡死。

**V2.3 修正：**

- 備份檔上限：50 MB。
- 小朋友：最多 30 位。
- 禮物：最多 100 筆。
- 作品 / 數學暫存：各最多 1000 筆匯入資料。
- 圖片格式與大小逐筆驗證。
- 匯入前先顯示版本、備份時間、孩子數、作品數、數學暫存數，再要求確認。

### 4. 完成按鈕快速連點造成重複計分 — 已修正

**風險類型：資料完整性**

完成流程含 async 作品保存。在舊流程中，極快速連點可能在 `awarded` 寫入前進入第二次完成流程。

**V2.3 修正：**

- 新增 `completionBusy` 鎖。
- async 保存期間停用完成按鈕。
- await 後再次檢查是否已完成。
- 實測快速連按兩次，只增加 5 點。

### 5. Service Worker 更新流程 — 已修正

舊版 install 期間直接 `skipWaiting()`，會讓「發現新版 → 立即更新」的 waiting 流程失去意義。

**V2.3 修正：**

- install 只建立 cache，不主動 `skipWaiting()`。
- 使用者按「立即更新」才送出 `SKIP_WAITING`。
- Service Worker 只處理核心靜態檔，不再把所有同源 GET 任意加入快取。
- Cache response 僅在 `resp.ok` 時寫入。

### 6. 預覽縮放狀態競態 — 已修正

這不是資安漏洞，但屬資料 / UI 狀態一致性問題。原本排程中的 fit 計算可能在使用者已切換 100% 後再覆蓋設定。現在延遲執行前會再次確認 `previewZoom === 'fit'`。

## 隱私與資料流檢查

- App 主程式沒有外部 API 呼叫。
- HTML 沒有外部 CDN、第三方 JavaScript 或第三方 CSS。
- CSP `connect-src 'self'`。
- Service Worker 僅處理同源資源。
- Referrer policy：`no-referrer`。
- 孩子資料、學習紀錄主要儲存在 localStorage / IndexedDB。
- 本版沒有把孩子名稱、手寫內容或學習紀錄主動上傳到伺服器的程式碼。

## 仍存在的設計限制

### A. 家長 PIN 不是密碼學安全機制

家長 PIN 儲存在本機資料中，其定位是「避免小朋友誤進家長介面」，不是用來抵抗擁有裝置、DevTools 或檔案存取權限的攻擊者。熟悉瀏覽器開發工具的人仍可修改本機資料。

**建議：** 不要把 PIN 當成保護真正敏感資料的密碼。如果未來需要多人帳號或雲端同步，必須改成真正的登入 / 驗證架構。

### B. 完整備份 JSON 未加密

備份可能包含孩子名稱、練習紀錄與手寫作品。拿到備份檔的人可以直接閱讀內容。

**目前措施：** 家長管理頁已增加明確警告。

**建議：** 不要把完整備份放到公開 GitHub、公開雲端分享或社群貼文附件中。

### C. 瀏覽器本機儲存未加密

localStorage / IndexedDB 由瀏覽器管理，本工具沒有額外加密。共用電腦的其他可存取同一瀏覽器設定檔的人，可能取得這些資料。

### D. CSP 的 style 仍允許 `unsafe-inline`

因現有 UI 大量使用 inline style 與單檔 CSS 結構，`style-src` 仍需允許 inline style；但 JavaScript 已拆出，因此 `script-src` 已不需要 `unsafe-inline`。這是目前安全性上較重要的改善。

### E. GitHub Pages 的 HTTP Security Headers 不由本 HTML 完整控制

Meta CSP 可以提供一部分瀏覽器防護，但 `X-Content-Type-Options`、`Permissions-Policy`、完整 `frame-ancestors` 等最好由 HTTP response header 設定。若未來用於正式多人環境，建議部署到可自訂安全標頭的主機。

## 最終結論

V2.3 已針對本輪發現的 DOM XSS、原型污染、惡意備份資源耗盡、重複計分、PWA 更新一致性等問題完成修補。以「單機 / 家庭使用 / GitHub Pages 靜態部署」的定位而言，目前沒有檢測到阻斷性高風險問題；但家長 PIN、本機資料與未加密備份仍應視為便利功能，而不是高安全等級的存取控制。
