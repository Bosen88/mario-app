# Mario App — Firebase + Vercel 部署指南

## 概覽
此 App 使用 Firebase Firestore 儲存資料，所有裝置即時同步。
資料結構：`appdata/mv2_users`（用戶）、`appdata/mv2_subs`（任務記錄）。

---

## 步驟 1：建立 Firebase 專案

1. 前往 [https://console.firebase.google.com](https://console.firebase.google.com)
2. 點「新增專案」→ 輸入名稱（例：`mario-app-prod`）→ 建立
3. 在左側選單點「建置 > Firestore Database」
4. 點「建立資料庫」→ 選擇地區（建議 `asia-east1` 台灣最近）
5. **先選「測試模式」**（30 天後需更新安全性規則）

### Firestore 安全性規則（上線後請改為此版本）
在 Firebase Console > Firestore > 規則，貼上：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // 只允許讀寫 appdata 集合（此 App 唯一使用的集合）
    match /appdata/{docId} {
      allow read, write: if true;  // 開放所有人（適合內部小型團隊）
    }
  }
}
```

> 若日後要加入身份驗證，改為 `allow read, write: if request.auth != null;`

---

## 步驟 2：取得 Firebase 設定值

1. Firebase Console 左上角點齒輪 > 「專案設定」
2. 滑到「您的應用程式」區塊 > 點「</> 新增應用程式（Web）」
3. 輸入暱稱（例：`mario-web`）→ 不勾選 Hosting → 點「註冊應用程式」
4. 複製 `firebaseConfig` 物件中的各項值：

```
apiKey            → VITE_FIREBASE_API_KEY
authDomain        → VITE_FIREBASE_AUTH_DOMAIN
projectId         → VITE_FIREBASE_PROJECT_ID
storageBucket     → VITE_FIREBASE_STORAGE_BUCKET
messagingSenderId → VITE_FIREBASE_MESSAGING_SENDER_ID
appId             → VITE_FIREBASE_APP_ID
```

---

## 步驟 3：本機測試（選用）

```bash
# 在 mario-app/ 目錄下
cp .env.example .env
# 編輯 .env 填入上方取得的實際值
npm install
npm run dev
```

> **注意：** `.env` 已在 `.gitignore` 中，不會被 commit。

---

## 步驟 4：推送到 GitHub

```bash
cd mario-app
git init
git add .
git commit -m "feat: migrate localStorage to Firebase Firestore"
git branch -M main
git remote add origin https://github.com/bosen88/mario-app.git
git push -u origin main
```

> 若 repo 已存在，跳過 `git init` 和 `git remote add`，直接 push。

---

## 步驟 5：部署到 Vercel

1. 前往 [https://vercel.com](https://vercel.com) → 用 GitHub 帳號登入
2. 點「Add New > Project」→ 選擇 `mario-app` repo → 點「Import」
3. Framework Preset 選「**Vite**」（Vercel 通常自動偵測）
4. 展開「**Environment Variables**」，逐一新增：

| 變數名稱 | 值（從 Firebase 複製） |
|----------|----------------------|
| `VITE_FIREBASE_API_KEY` | `AIzaSy...` |
| `VITE_FIREBASE_AUTH_DOMAIN` | `your-project.firebaseapp.com` |
| `VITE_FIREBASE_PROJECT_ID` | `your-project-id` |
| `VITE_FIREBASE_STORAGE_BUCKET` | `your-project.appspot.com` |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | `123456789` |
| `VITE_FIREBASE_APP_ID` | `1:123...` |

5. 點「**Deploy**」→ 等待約 1 分鐘 → 取得 `.vercel.app` 網址

---

## 重新部署（後續更新）

```bash
git add .
git commit -m "your update message"
git push
```
Vercel 會自動偵測 GitHub push 並重新部署。

---

## 資料結構說明

| Firestore 路徑 | 內容 |
|---------------|------|
| `appdata/mv2_users` | `{ data: [ ...用戶陣列 ] }` |
| `appdata/mv2_subs` | `{ data: [ ...任務記錄陣列 ] }` |

第一次開啟 App 時，Firestore 中尚無資料，會使用程式碼中的 `SEED_MEMBERS` 作為初始用戶清單（僅顯示，不自動寫入）。管理員首次操作（新增/編輯成員）後資料才會寫入 Firestore。

---

## 常見問題

**Q: 多人同時使用會衝突嗎？**
A: 不會。`onSnapshot` 即時監聽，任何人的更新都會立刻同步到其他裝置。

**Q: Firestore 費用？**
A: 免費方案（Spark Plan）提供每天 50,000 次讀取 / 20,000 次寫入，小型團隊完全夠用。

**Q: 圖片（imgData）儲存在哪？**
A: 目前仍以 base64 字串存在 Firestore 的 `subs` 文件中。若圖片很大，建議日後改用 Firebase Storage。
