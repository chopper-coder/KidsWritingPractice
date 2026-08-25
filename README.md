# 兒童書寫練習產生器 V2.3.1

一個可直接放在 GitHub Pages 使用的純前端兒童書寫練習工具。支援英文、注音、國字、數字、加減法、平板手寫、A4 練習紙、智慧弱項、作品相簿與家庭資料備份。

## 特色

- 👧 小朋友模式：英文、注音、數字、國字、加減法可直接選科。
- ✍️ 平板手寫：支援觸控筆、最大化書寫、逐題暫存。
- ➕➖ 智慧數學：題目去重、答案核對、錯題型複習、逐題續作。
- 🖨️ A4 練習紙：預覽與列印一致、指定頁列印、自訂內容、節省紙張版面。
- 🌱 成長紀錄：每日任務、徽章、弱項與作品相簿。
- 💾 本機資料：學習紀錄存於瀏覽器 localStorage / IndexedDB。
- 📴 PWA：部署到 HTTPS 後可加入主畫面並支援離線使用。

## GitHub Pages 部署

1. 建立新的 GitHub Repository，例如 `KidsWritingPractice`。
2. 將本專案所有檔案上傳到 `main` 分支根目錄。
3. 到 **Settings → Pages**。
4. **Build and deployment → Source** 選擇 **GitHub Actions**。
5. 推送完成後，到 **Actions** 等待 `Deploy GitHub Pages` 顯示綠色勾勾。
6. GitHub Pages 網址通常為：`https://你的帳號.github.io/儲存庫名稱/`。

本專案已附 `.github/workflows/deploy.yml`，推送到 `main` 後會自動部署。

## 隱私與資料安全

- 程式為純前端，沒有自建後端，也不會主動把孩子資料上傳到伺服器。
- 孩子名稱、點數、進度與手寫作品主要保存在目前裝置的瀏覽器。
- 匯出的家庭備份 JSON 可能含孩子名稱與手寫作品，**不要提交到公開 GitHub Repository**。
- 家長 PIN 主要用於防止小朋友誤觸，不等同高強度身分驗證。

詳細檢測可參考 `TEST_REPORT.md` 與 `SECURITY_REPORT.md`。
