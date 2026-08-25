# GitHub Pages 上線步驟

## 建議儲存庫名稱
`KidsWritingPractice`

## 上傳方式
將 ZIP 解壓縮後，把內容放在 Repository 根目錄。根目錄應直接看到：

- `index.html`
- `app.js`
- `manifest.webmanifest`
- `sw.js`
- `icon-192.png`
- `icon-512.png`
- `.nojekyll`
- `.github/workflows/deploy.yml`

## 啟用 Pages
GitHub → Repository → Settings → Pages → Source → GitHub Actions。

之後每次更新 `main` 分支，GitHub Actions 都會重新部署網站。

## 注意
請勿把程式匯出的家庭備份 JSON 上傳到公開 Repository。
