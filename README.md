# 我紀 · MyLog

「我的紀律」——單人健身記錄工具，純靜態（HTML + 原生 JS），部署於 GitHub Pages。資料以 `localStorage` 持久化於瀏覽器本機，無後端。

暖米紙基底，點左上角的 **W** 圖標可輪替重點色（陶土磚紅 → 橄欖軍綠 → 焦糖芥末 → 墨黛藍），選擇存於 `localStorage`（key `mylog.accent`），兩頁共用、重新整理後維持。

## 頁面
- `index.html` — 入口，轉址到訓練頁
- `ironlog-train.html` — 今日訓練：勾選組數、輸入 RPE、看總量與部位分解、匯出 CSV
- `ironlog-menu.html` — 菜單設定：建立／編輯菜單與動作

## 資料層
- localStorage key：`ironlog.menus`
- 形狀：`[{ name, ex:[{ name, part, w, sets, reps }] }]`
- 菜單頁按「儲存菜單」寫入；訓練頁開頁讀回。首次開啟（key 不存在）以內建示範菜單種子寫入一次。

## 本機測試
```
python -m http.server 8000
# 開 http://localhost:8000
```

## 部署（GitHub Pages）
1. 推到 public repo 的 `main` 分支根目錄。
2. Settings → Pages → Source 選「Deploy from a branch」，Branch 選 `main` / `/ (root)`。
3. 網址：`https://<帳號>.github.io/<repo>/`
