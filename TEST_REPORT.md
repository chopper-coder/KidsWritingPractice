# V2.4.4 測試報告

## 靜態檢查
- `app.js`：`node --check` PASS。
- HTML ID：187 個，重複 0。
- JavaScript DOM ID 引用：171 個，缺漏 0。
- `manifest.webmanifest`：JSON 解析 PASS。

## ZIP 備份核心測試
- STORE ZIP 寫入／讀回：PASS。
- UTF-8 `backup.json`：PASS。
- 二進位圖片 entry 讀回：PASS。
- CRC32 損壞偵測：PASS，竄改資料會拒絕匯入。
- `../` 路徑穿越：PASS，會拒絕建立／讀取不安全路徑。
- 使用瀏覽器同款 ZIP writer 建立測試 ZIP，再以系統 `unzip -t` 驗證：PASS。

## 執行環境
- Chromium headless 在此容器環境因 D-Bus/程序結束限制無法完成完整 UI 自動化，因此沒有把該項標成 PASS；未觀察到 JavaScript console 的 Uncaught/ReferenceError/TypeError/SyntaxError。

## 封裝
- 最終發佈 ZIP 另以 `unzip -t` 驗證完整性。
