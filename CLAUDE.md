# 每日派車管理系統：專案說明（給 Claude 與維護者）

## 這是什麼
這是一家北海岸遊覽車公司的每日派車系統。車趟類型有學校校車、學校補習班、旅遊團、進香團，類型可以在「設定」分頁自訂；車種分大車、中車。
擁有者用個人 Gmail 管理，與公司帳號無關。

## 架構
- **前端**：單一 `index.html`，原生 JS（`<script type="module">`），沒有 build 步驟。
- **資料庫**：Firebase Firestore，專案 `nbbus-683cc`，地區 asia-east1。設定值在 `firebase-config.js`，這些值本來就公開在網頁上，不是密碼。
- **登入**：Firebase Auth 的 Google 登入。
- **網站**：GitHub Pages，branch `main`、資料夾 `/ (root)`。推到 main 後約 1～2 分鐘自動更新。
- **SDK**：Firebase 10.12.2，從 gstatic CDN 載入。程式裡有一層小 adapter（`makeDb()`），提供 `db.collection(x).doc(id).set()/delete()`、`.where().get()/onSnapshot()`。

## 資料表（Firestore collections）
- `vehicles`：plate、type（大車/中車）、seats、status（可派/維修/停用）、driver、phone、note
- `customers`：name、type（tripTypes 的 id）、contact、phone、pickup、note
- `trips`：date（YYYY-MM-DD）、type、customerId、start、end（HH:MM）、pickup、destination、pax、vehicleType、vehicleId（未派車時為 null）、note、updatedAt
- `tripTypes`：label、color（#rrggbb）、order、active。預設 id 為 school / cram / tour / pilgrim；資料表是空的時，第一位登入者會自動寫入預設值。
- `members/{email 小寫}`：email、name、roleId、groupId、active
- `roles`：name、order、canManage（可管理權限）、modules（可瀏覽功能 key 陣列）。預設角色：admin 系統管理員（固定全部）、manager 經理、assistant 特助、leader 組長、staff 組員
- `groups`：name、order、note（部門／小組，只用來分類）

## 權限設計
- 人員名單、角色、群組都在系統內管理（「系統設定」→ 權限管理／角色管理／群組管理）。
- 權限只控制「能不能看到某個功能」；依角色設定。功能 key：dash、trips、customers、vehicles、tripTypes；members/roles/groups 只給 canManage 的角色。
- `firestore.rules` 的 `ownerList()` 是最高管理員（一定能登入）。**repo 是公開的，檔案裡只放預留字，真實 Gmail 只填在 Firebase 主控台。**
- 擁有者第一次登入時，若沒有 members 文件，系統會自動建立預設角色並把他設為 admin。
- 新增 collection 或功能時，必須同步修改規則，並提醒擁有者到 Firebase 主控台的「規則」分頁貼上後按「發布」。Claude 無法代為發布規則。

## 已確定的設計（不要隨意更動）
- 標題「每日派車管理系統」，副標題「DAILY DISPATCH · 北海之真情 · · 與您共享旅遊的歡愉」。**中間的兩個點是刻意的。**
- 標題和副標題用芫荽體（Google Fonts: Iansui）；內文用思源黑體（Noto Sans TC）；日期用思源宋體（Noto Serif TC）；數字、車牌用 IBM Plex Mono。
- 三隻海鷗（inline SVG）放在標題右側（方案 A）。手機版的海鷗縮小、移到右上角，副標題分成英文和中文兩行。
- 北海岸配色：深海藍綠色的頁首、波浪邊、淺海霧色的底色。支援深色模式。
- 儀表板：上方是已派車時間表（每台車一列，依大車/中車分組，同車時段重疊時用紅框標示）；下方是未派車清單（可直接選車派出）。

## 版面
- 左側可收合選單（大項：派車作業、基本資料、系統設定；底下有子項），手機版改成 ☰ 滑出選單。
- 登入者資訊（人頭圖示＋姓名＋角色·群組）放在頁面標題列最右側；點開可登出。手機版只顯示頭像。

## 已有功能
- 新增、編輯、刪除車趟、客戶、車輛（刪除要按兩次確認）
- 複製前一日車趟（保留原派車）
- 派車時檢查時段衝突和座位數
- 匯出、匯入 JSON 備份
- 「設定」分頁：自訂車趟類型（名稱、顏色、排序、停用）

## 工作方式（擁有者的要求）
1. 改畫面前，**先給擁有者看截圖或預覽，確認後才改正式檔案。**
2. 推上 GitHub 前要取得同意。
3. 回覆使用繁體中文，用詞要讓非工程背景的人看得懂。
4. 改完後告訴擁有者：要不要更新 Firebase 規則，以及網站多久會生效。
