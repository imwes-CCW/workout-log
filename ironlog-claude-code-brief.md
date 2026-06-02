# IRONLOG — Claude Code 建置交辦文件

> 目的：把兩個已完成的靜態 HTML 頁面，部署成可用的 **GitHub Pages** 服務，並把兩頁用瀏覽器端儲存串起來（菜單頁存、訓練頁讀）。
> 重要：**不要更動既有視覺設計與互動**，只新增「資料層 + 導頁 + 部署」。

---

## 1. 既有資產（輸入）

兩個已完成的單一 HTML 檔（設計定稿、勿改外觀）：

- `ironlog-menu.html` — 菜單設定頁
- `ironlog-train.html` — 今日訓練頁

目前狀態：兩頁各自帶一份 **in-memory 示範資料**，彼此尚未連動。兩頁的資料結構已對齊（見第 4 節）。

設計重點（需保留）：深色 × 鐵橘色 `#ff6a1a`、字體 Archivo + JetBrains Mono（Google Fonts CDN）、卡片式 UI、打勾原地淡入、總量 count-up、部位長條等互動。

---

## 2. 限制（決定架構）

- **GitHub Pages 只能放靜態檔**：沒有後端、不能跑伺服器程式，所有邏輯必須在瀏覽器端。
- 採 **localStorage** 做持久化即可（單人、單機個人工具，足夠且零後端）。
- 不要引入打包工具 / 框架；維持單檔 HTML + 原生 JS 的形式。

---

## 3. 核心任務

1. 建立可部署到 GitHub Pages 的 repo 結構。
2. 用 localStorage 建立**兩頁共用的資料層**（schema 見第 4 節）。
3. **菜單頁**：`儲存`按鈕寫入 localStorage；開頁時讀回既有菜單。
4. **訓練頁**：「今天訓練選擇」改成讀 localStorage 的真實菜單（不要再用寫死的示範資料當來源）。
5. 補上兩頁之間的**導頁**。
6. 保留 CSV 匯出（檔名 `YYMMDD-workout-results.csv`、只匯出已勾選的組）。
7. 部署到 GitHub Pages 並驗收。

---

## 4. 資料結構（localStorage）

- Key：`ironlog.menus`
- Value：JSON 字串，內容為菜單陣列：

```json
[
  {
    "name": "上肢日",
    "ex": [
      { "name": "槓鈴臥推", "part": "胸", "w": 80, "sets": 4, "reps": 8 },
      { "name": "槓鈴划船", "part": "背", "w": 70, "sets": 4, "reps": 8 }
    ]
  }
]
```

規則：
- `part` ∈ 胸 / 背 / 肩 / 腿 / 臀 / 手臂 / 核心
- 上限：最多 5 份菜單，每份最多 10 個動作
- **首次開啟（key 不存在）**：用檔案內既有的示範菜單當種子寫入一次，避免空白。

> 兩個 HTML 檔目前的資料變數已是這個形狀（menu 頁 `menus`、train 頁 `MENUS`，皆為 `[{name, ex:[{name,part,w,sets,reps}]}]`），直接對應即可。

---

## 5. 兩頁要接的點

### 菜單頁（ironlog-menu.html）
- 開頁：讀 `ironlog.menus`，有就 render、沒有就用內建示範種子。
- `儲存菜單`按鈕：把目前 `menus` 陣列 `JSON.stringify` 寫入 localStorage；保留現有「已儲存」綠色狀態。
- 其餘編輯邏輯（新增/刪除菜單與動作、動作選擇器、部位）不變。

### 訓練頁（ironlog-train.html）
- 開頁：讀 `ironlog.menus` 填入「今天訓練選擇」。
- 若沒有任何菜單：顯示空狀態提示（例如「尚未建立菜單，先到菜單頁面設定」）＋前往菜單頁的連結。
- RPE 輸入、部位訓練量分解、CSV 匯出等邏輯不變。

### 導頁
- 訓練頁有「菜單頁面」按鈕（→ `ironlog-menu.html`）。
- 菜單頁有「訓練頁面」按鈕（→ `ironlog-train.html`，位於右上角）。
- 兩頁互連已具備，**直接沿用、確保相對連結有效即可，無需詢問**。
- 新增 `index.html`：作為入口，**導向訓練頁**（最常用）。可用簡單的 meta refresh 或 JS 轉址到 `ironlog-train.html`。

---

## 6. Repo 結構與部署

建議結構（檔名沿用，確保兩頁互連的相對連結 `ironlog-menu.html` / `ironlog-train.html` 仍有效）：

```
/
├─ index.html            # 轉址到 ironlog-train.html
├─ ironlog-train.html
└─ ironlog-menu.html
```

部署步驟：
1. 建立一個 public GitHub repo，把上述檔案放在 repo 根目錄。
2. Settings → Pages → Build and deployment → Source 選「Deploy from a branch」，Branch 選 `main` / `/root`。
3. 等待產生網址 `https://<帳號>.github.io/<repo>/`。
4. 本機可先用 `python3 -m http.server` 測試後再 push。

---

## 7. 驗收標準

- 打開 Pages 網址會進到訓練頁。
- 在菜單頁建立 / 編輯菜單並按「儲存」後，**重新整理仍在**。
- 訓練頁「今天訓練選擇」顯示的，正是已儲存的菜單。
- 完成訓練可下載 `YYMMDD-workout-results.csv`，**只含已勾選的組**，欄位：`日期, 菜單, 動作, 部位, 組, 重量, 次數, RPE`。
- 視覺與互動與原檔一致，無破版。

---

## 8. 注意事項 / 已知限制

- localStorage 是**單一瀏覽器、單一裝置**，不會跨裝置同步。若日後需要多裝置同步或雲端備份，就需要後端（GitHub Pages 做不到）——屆時可考慮 serverless API 或 Supabase 之類，屬另一階段。
- 全程資料留在瀏覽器，唯一離開裝置的是使用者自行下載的 CSV。
- 請勿為了「順便優化」而改動既有設計、版面或互動；本次只做資料層、導頁與部署。
