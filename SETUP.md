# 有我在療癒所 · 預約系統 部署指南

整個系統由三塊組成，全部免費（除非 LINE Pay 手續費）：

```
index.html（LIFF 預約頁，放 Vercel）
   ↓ 送出預約
Apps Script Web App（後端）── 寫 → Google Sheet（預約總表）
   ↓ LINE Messaging API 推播
業主／客人的 LINE
admin.html（業主審核頁，放 Vercel）
```

> ⚙️ 目前程式都寫好了，只差「填憑證」。下面照順序做。等跟業主約好、綁他官方帳號後再執行。

---

## 步驟 1：Google 試算表 + 後端 Apps Script
1. 用要管預約的 Google 帳號開一個新 **Google 試算表**，命名「有我在預約」。記下網址裡的 ID：`docs.google.com/spreadsheets/d/`**`這段`**`/edit`。
2. 試算表 → 擴充功能 → **Apps Script**。
3. 把 `backend/Code.gs` 全部貼進去，存檔。
4. 上方 `CFG` 區填：`SHEET_ID`、`ADMIN_KEY`（自訂密碼）、`BRAND`。（LINE 相關先留空，步驟 3 再回填）
5. 部署 → **新增部署作業** → 類型「網頁應用程式」→ 執行身分＝**我**、存取權＝**任何人** → 部署。複製產生的**網頁應用程式網址**。
6. 把這個網址填到：
   - `index.html` 的 `CONFIG.apiUrl`
   - `admin.html` 最上方的 `API_URL`
   - `Code.gs` 的 `CFG.ADMIN_URL`（admin 上線網址）、`CFG.BOOKING_URL`

## 步驟 2：每日 20:00 提醒（時間觸發器）
Apps Script → 左側「觸發條件」⏰ → 新增觸發條件：
- 函式 `sendDailyReminders`、事件來源「時間驅動」、類型「日計時器」、時段 **晚上 8–9 點**。

## 步驟 3：LINE 官方帳號 + 推播（綁業主帳號後做）
1. 用業主的 LINE 商用帳號登入 **LINE Developers Console**，建立／取得 **Messaging API channel**。
2. 複製 **Channel access token（長期）** → 填 `Code.gs` 的 `CFG.LINE_TOKEN`。
3. 取得**業主自己的 userId**（加自己的 OA 為好友後，用 webhook 或工具查）→ 填 `CFG.OWNER_USER_ID`（接「新預約通知」）。
4. 客人的 userId 由 LIFF 自動取得（步驟 4），不用手動。

## 步驟 4：LIFF（讓預約頁在 LINE 內開、綁定身分）
1. 同一個 Provider 下，LINE Login channel → 新增 **LIFF**。
2. Endpoint URL 填 index.html 的正式網址（Vercel）；Size 選 Full；勾選 `profile`。
3. 複製 **LIFF ID** → 填 `index.html` 的 `CONFIG.liffId`。
4. 把預約入口做成 LINE 圖文選單，連到 `https://liff.line.me/<LIFF_ID>`。
   - 填了 liffId 後，頁面會要求在 LINE 內開啟並登入，才能取得 userId（沒登入就收不到通知）。

## 步驟 5：部署前端到 Vercel
```
cd ~/Projects/healing-booking
vercel --prod
```
（純靜態，index.html + admin.html + assets。沿用得群慣用的 Vercel CLI 部署。）

## 步驟 6：LINE Pay（之後，需商家帳號）
> ⚠️ LINE Pay 收款要先申請 **LINE Pay 商家帳號**（需商業登記、LINE Pay 台灣審核），不是填程式就能用。
- 過渡：可先用 LINE Pay 個人收款連結/QR（免審核、無自動對帳）。
- 正式：拿到 **Channel ID / Channel Secret** 後，在 Apps Script 加 LINE Pay Request/Confirm API；付款時機（先付訂金／同意後付／全額預付）見規格文件，待業主決定。
- 程式已預留接口，補上即可。

---

## 設定檢查清單
- [ ] Code.gs：SHEET_ID、ADMIN_KEY、LINE_TOKEN、OWNER_USER_ID、ADMIN_URL、BOOKING_URL
- [ ] index.html：CONFIG.apiUrl、CONFIG.liffId、DIY 三項 price、hours（營業時間）
- [ ] admin.html：API_URL
- [ ] Apps Script：部署為網頁應用程式、裝 20:00 觸發器
- [ ] LINE：Messaging API token、業主 userId、LIFF endpoint
- [ ] Vercel：index.html + admin.html 上線

## 預約流程（提醒）
送出＝**待確認** → 通知業主 → 業主在 admin 按**同意** → 推「預約成功」給客人 → 預約**前一天 20:00** 自動提醒。
