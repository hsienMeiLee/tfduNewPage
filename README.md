# 分頁管理前端｜Vue 3

最新版介面：左側選取「新增主選單」；「保險推廣」為初始分頁。保留行內新增名稱／新增／取消，右上方重複新增按鈕已移除。管理選單包含改名及刪除。

## 啟動
Node.js 22.12 以上，在專案資料夾執行：
```bash
npm ci
npm run dev
```
建置與預覽：
```bash
npm run build
npm run preview
```

## 已實作
- 新增分頁：空白、重複名稱、20 字長度驗證；成功後切換到新分頁。
- 改名：標題及頁籤同步更新，保留分頁 ID 與其內容。
- 刪除：確認視窗需輸入完整名稱，示範中一併移除該頁內容；自動切換相鄰分頁。刪除最後一頁顯示新增提示。
- 分頁切換、日期排序、逐筆勾選、手機側欄、對話框。
- localStorage 在本機保存頁籤與示範資料；不是正式後端。
- 三個初始分頁；只有保險推廣放四筆截圖示範內容，其他分頁留空。

## 整合
主要檔案 src/App.vue、src/style.css。請將 add、save、persist 與讀取資料改接 API。資料格式：Tab={id,name}；Article={id,tabId,title,date,image}。
建議 API：GET /tabs；POST /tabs；PATCH /tabs/:id；DELETE /tabs/:id；GET /tabs/:id/articles。正式删除須由後端在交易中處理分頁與其內容，並核對權限；目前不呼叫任何正式網站。

其他側欄選單沒有目標頁，暫停用。原圖的逐列省略選單沒有功能規格，未製作；頁碼依實際四筆顯示一頁，不假造 14 頁。
Logo 及縮圖以原截圖 CSS 定位呈現，正式環境請改用獨立圖檔；示範標題日期並非正式保險資料。
Nuxt 4 可將 App.vue 作為頁面、style.css 加入全域樣式，圖檔放入 public；改用 /reference.png 資源路徑。此交付為 Vue 3 + Vite 可獨立啟動專案。

## 免安裝預覽
雙擊 preview.html 即可使用（已內嵌編譯後程式與圖片）。本機檔案模式的儲存行為依瀏覽器而異；正式整合請使用 HTTP 網站服務。已通過 Vite 正式建置，未執行完整瀏覽器驗收。
