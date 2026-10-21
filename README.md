# 今晚吃什麼轉盤（GitHub Pages + Firebase 版）

兩個人共用的晚餐口袋名單，依地區或類型篩選後用轉盤抽。
網頁放在 GitHub Pages，名單存在 Firebase，只有規則裡列出的兩個 Google 帳號能讀寫。

整個設定大約 15 到 20 分鐘，全程免費、不用綁信用卡。

---

## 步驟 1：把網頁放上 GitHub

1. 到 <https://github.com/new> 建一個新的 repository，名稱例如 `dinner-wheel`，選 **Public**。
2. 在新 repo 頁面按 **Add file → Upload files**，把 `index.html` 拖進去，按 **Commit changes**。
3. 進 repo 的 **Settings → Pages**：
   - Source 選 **Deploy from a branch**
   - Branch 選 **main**，資料夾選 **/ (root)**，按 **Save**
4. 等一到兩分鐘，頁面上方會出現網址，長得像 `https://你的帳號.github.io/dinner-wheel/`。先記下來。

這時打開網址已經能用了，只是會顯示「還沒設定 Firebase，目前只存在這台裝置」。

## 步驟 2：建立 Firebase 專案

1. 到 <https://console.firebase.google.com/> 用 Google 帳號登入。
2. 按 **建立專案**，名稱例如 `dinner-wheel`。
3. 問要不要啟用 Google Analytics 時可以關掉，用不到。

## 步驟 3：開啟 Google 登入

1. 左側選單 **建構（Build）→ Authentication**，按 **開始使用**。
2. 在 **登入方式（Sign-in method）** 分頁選 **Google**，打開「啟用」，選一個支援用的電子郵件，按 **儲存**。
3. 切到 **設定（Settings）→ 授權網域（Authorized domains）**，按 **新增網域**，填入：
   ```
   你的帳號.github.io
   ```
   （只填網域，不要加 `https://` 或後面的 `/dinner-wheel/`）

## 步驟 4：取得設定，貼進 index.html

1. 左上角齒輪 → **專案設定**，往下找到「你的應用程式」，按 **`</>`（網頁）** 圖示。
2. 應用程式暱稱隨便取，**不要**勾 Firebase Hosting，按註冊。
3. 畫面會出現一段 `const firebaseConfig = { ... }`，把大括號裡的內容複製起來。
4. 回到 GitHub，打開 `index.html`，按鉛筆圖示編輯，找到這一段：
   ```js
   window.FIREBASE_CONFIG = {
     apiKey: "貼上你的 apiKey",
     ...
   };
   ```
   把大括號裡整段換成你剛剛複製的內容，按 **Commit changes**。

> `apiKey` 放在公開的 repo 裡是正常的，它不是密碼。真正擋住外人的是下一步的資料庫規則。

## 步驟 5：建立資料庫並設定規則

1. 左側選單 **建構（Build）→ Firestore Database**，按 **建立資料庫**。
2. 位置選 **asia-east1（台灣）**。這個之後不能改。
3. 選 **以正式版模式啟動（production mode）**，按建立。
4. 建好後切到 **規則（Rules）** 分頁，把內容整個換成下面這段，**把兩個信箱改成你們自己的 Google 帳號**，按 **發布**：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isUs() {
      return request.auth != null
        && request.auth.token.email in ['你的信箱@gmail.com', '她的信箱@gmail.com'];
    }
    match /{document=**} {
      allow read, write: if isUs();
    }
  }
}
```

## 步驟 6：打開來用

1. 打開 GitHub Pages 的網址（剛 commit 完可能要等一分鐘才更新，可以重新整理）。
2. 按右上角 **用 Google 登入**。
3. 如果之前在這台裝置上加過選項，名單上方會出現「搬到共用名單」，按一下就會同步上去。
4. 把網址傳給女朋友，她用自己的 Google 帳號登入就能一起用。

---

## 常見問題

- **登入後顯示「沒有權限」**：規則裡的信箱拼錯，或登入的不是規則裡的帳號。回到步驟 5 檢查。
- **按登入沒反應或出現 `auth/unauthorized-domain`**：步驟 3 的授權網域沒加，或拼錯。
- **出現 `auth/popup-blocked`**：瀏覽器擋了彈出視窗，允許這個網站的彈出視窗後再按一次。
- **想加第三個人**：在步驟 5 的規則清單多加一個信箱，再發布一次就好。
