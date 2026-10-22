# 派車管理系統

這是一套每日派車系統，管理學校校車、學校補習班、旅遊團、進香團的派車。

- **儀表板**：上方是已派車的時間表，每台車一列；下方是未派車趟清單，可以直接選車派出。同一台車時段重疊時會用紅框標出。
- **車趟／客戶／車輛**：可以新增、編輯、刪除資料。也能把前一天的車趟複製到當天。
- **備份**：頁面最下方的「匯出備份」會把全部資料下載成一個 `.json` 檔，「匯入備份」可以還原資料。

整套系統只用到一個網頁檔案和 Google Firebase，不需要架設伺服器，也不依賴 Claude。

## 檔案說明

| 檔案 | 用途 |
|---|---|
| `index.html` | 系統本體（畫面與程式） |
| `firebase-config.js` | 連到你們 Firebase 專案的設定（步驟 2 要填） |
| `firestore.rules` | 資料庫權限：誰可以讀寫 |
| `firebase.json` | 用 Firebase CLI 部署時才需要 |

---

## 第一次設定（約 20 分鐘）

### 1. 建立 Firebase 專案
1. 用你的 Gmail 登入 <https://console.firebase.google.com>，按「新增專案」，取個名字（例如 `soic-dispatch`）。Google Analytics 可以關掉。
2. 左側選單「建構 → Firestore Database」→「建立資料庫」。
   - 位置選 **asia-east1（台灣）**。
   - 模式選「正式版模式」。
3. 左側「建構 → Authentication」→「開始使用」→「登入方式」→ 啟用 **Google**。

### 2. 取得網頁設定
1. 專案總覽旁的齒輪 →「專案設定」→ 最下方「你的應用程式」→ 按 `</>`（網頁）圖示，取個暱稱後註冊。
2. 畫面會出現一段 `const firebaseConfig = { ... }`，把裡面的值複製到 `firebase-config.js` 對應的位置。

### 3. 設定資料庫權限
1. 打開 Firestore Database →「規則」分頁。
2. 把 `firestore.rules` 的全部內容貼上去，然後按「發布」。
3. 只有 `allowList()` 名單裡的 Gmail 可以使用。把 `你的帳號@gmail.com` 換成你自己的信箱，同事的信箱也逐一加進去（要用英文雙引號包起來，每行最後加逗號），然後按「發布」。之後要加人或移除人，也是改這裡再發布。

### 4. 放上 GitHub 並開啟網址（GitHub Pages）
1. 在 GitHub 建一個新的 repository（例如 `dispatch`），把這個資料夾的所有檔案上傳上去。可以在網頁上用「Add file → Upload files」拖進去。
2. 到 repo 的 **Settings → Pages**，Source 選 `Deploy from a branch`，Branch 選 `main`、資料夾選 `/ (root)`，然後儲存。
3. 大約一分鐘後，網址會出現，格式是 `https://<你的帳號>.github.io/dispatch/`。
4. 回到 Firebase：Authentication →「設定」→「授權網域」→ 新增 `<你的帳號>.github.io`。少了這一步，Google 登入會失敗。

> 免費版 GitHub Pages 的 repo 必須是公開的。程式碼和 `firebase-config.js` 被看到沒有關係，資料本身由步驟 3 的權限保護。如果 repo 一定要設成私人，可以改用下面的 Firebase Hosting。

### 另一種做法：Firebase Hosting
需要先在電腦安裝 Node.js，然後在這個資料夾裡執行：
```bash
npm install -g firebase-tools
firebase login
firebase use --add          # 選擇步驟 1 建立的專案
firebase deploy             # 會同時部署網頁和資料庫權限
```
網址會是 `https://<專案ID>.web.app`，這個網域已經是預設的授權網域，不必另外新增。

---

## 日常使用
- 打開網址，用已加入名單的 Gmail 登入。
- 先到「車輛」和「客戶」分頁建立基本資料，再到「儀表板」新增車趟並派車。
- 校車、補習班這類每天固定的班次，用「複製前一日車趟」就能帶入。
- 建議每週按一次「匯出備份」，把備份檔存到公司的雲端硬碟。

## 費用
Firebase 免費方案（Spark）每天可以讀取 5 萬次、寫入 2 萬次，總共可存 1 GB 資料。一般車行每天幾十到幾百趟車用不到這個量。超過額度不會自動收費，只會暫停到隔天。

## 資料結構（Firestore）
- `vehicles`：`plate` 車牌、`type`（大車／中車）、`seats` 座位數、`status`（可派／維修／停用）、`driver` 司機、`phone` 電話、`note` 備註
- `customers`：`name` 名稱、`type`（school／cram／tour／pilgrim）、`contact` 聯絡人、`phone` 電話、`pickup` 上車地點、`note` 備註
- `trips`：`date`（YYYY-MM-DD）、`type`、`customerId`、`start`、`end`（HH:MM）、`pickup`、`destination`、`pax` 人數、`vehicleType` 需求車種、`vehicleId`（未派車時為 null）、`note`
