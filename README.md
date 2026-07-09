# Career Blueprint · Mike 主任職涯諮詢系統

純靜態 HTML 網站，透過 Firebase Firestore 儲存資料，部署在 Netlify。

## 檔案結構

| 檔案 | 用途 |
|---|---|
| `index.html` | 客戶填寫的職涯諮詢表單（含 6 題 Wealth Dynamics 天賦測驗） |
| `report.html` | 職涯策略診斷報告書（依 `index.html` 的填答自動產生） |
| `disc-test.html` | **新增** — DISC 行為風格測驗（20 題），獨立流程 |
| `disc-report.html` | **新增** — DISC 行為風格報告書（依 `disc-test.html` 的填答自動產生） |
| `admin-upgraded.html` | 後台管理系統，同時管理職涯諮詢客戶與 DISC 測驗客戶 |

所有頁面共用同一個 Firebase 專案（`career-b5a20`），但使用兩個不同的 Firestore collection：

- `bookings` — 職涯諮詢客戶（index.html → report.html）
- `discClients` — DISC 測驗客戶（disc-test.html → disc-report.html）

後台 `admin-upgraded.html` 會同時讀取這兩個 collection，並在側邊欄以「所有客戶」與「DISC 測驗客戶」分開顯示。

## Firestore 安全規則

新增的 `discClients` collection 需要跟 `bookings` 一樣的讀寫權限。到 Firebase Console → Firestore Database → 規則，確認類似以下設定（依你現有規則調整）：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /bookings/{docId} {
      allow read, write: if true;
    }
    match /discClients/{docId} {
      allow read, write: if true;
    }
  }
}
```

> 目前規則是開放讀寫（方便前台送出資料），正式上線後建議收斂權限，例如限制 write 只能新增不能刪改，並搭配 Firebase App Check。

## 部署方式：改用 GitHub + Netlify 持續部署

以後不用再手動把檔案拖進 Netlify，改成本地 git push 就會自動部署。

### 第一次設定

1. 到 [github.com](https://github.com) 建立一個新的 **private repository**（例如 `career-blueprint`），不要勾選自動加 README。
2. 在這個資料夾（也就是這次收到的檔案所在目錄）打開終端機，執行：

   ```bash
   git init
   git add .
   git commit -m "Initial commit: career blueprint + DISC 測驗系統"
   git branch -M main
   git remote add origin https://github.com/<你的帳號>/career-blueprint.git
   git push -u origin main
   ```

   （如果 push 時要求登入，GitHub 現在需要用 Personal Access Token 取代密碼，或改用 `gh auth login` 搭配 GitHub CLI。）

3. 回到 Netlify 後台，找到現有的網站（`unrivaled-paprenjak-08c650`）：
   - **Site configuration → Build & deploy → Continuous deployment**
   - 點 **Link repository**，選擇剛剛建立的 GitHub repo
   - Build command 留空，Publish directory 填 `.`（因為是純靜態檔案，不需要 build）
   - 存檔後 Netlify 會自動抓取 GitHub 上的內容部署

### 之後的日常部署流程

改完檔案後，在終端機執行：

```bash
git add .
git commit -m "說明這次改了什麼"
git push
```

Netlify 偵測到 GitHub 有新的 commit 就會自動重新部署，通常 1 分鐘內完成，不需要再手動上傳。

## 待辦提醒

- 目前 `disc-test.html`、`disc-report.html`、`admin-upgraded.html` 都是全新／更新版本，第一次部署前請先用瀏覽器打開確認外觀與 Firebase 讀寫正常。
- `report.html` 與 `index.html` 內容跟你原本的版本一致（只修正了 `index.html` 結尾一段重複貼上、位於 `<script>` 標籤外、實際不會執行的殘留程式碼）。
