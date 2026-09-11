# dashboard

一個 Claude Skill:用固定配色與固定版型產出儀表板,支援互動式 HTML 與靜態圖檔兩種輸出。

## 為什麼

每次叫模型畫圖,配色、版型、圖表類型都要重講一次,結果還不一致。這個 skill 把決定寫死:

- 淺色商務配色,6 色資料色序 + 正負向語意色
- 固定版型:標題 → KPI 卡 → 主圖 → 雙欄輔圖 → 明細表
- 選圖邏輯表:先問「這張圖要回答什麼問題」,再對應圖表類型
- 呈現規則:長條一律排序、y 軸從 0、不用雙 Y 軸、標題寫結論句

## 結構

```
dashboard/
├── SKILL.md              # 配色 / 版型 / 選圖邏輯 / 呈現規則(觸發時載入)
└── references/
    ├── html.md           # HTML 輸出:CSS tokens、結構、Chart.js 設定
    └── python.md         # 靜態出圖:matplotlib rcParams、常用樣板
```

參考檔只在需要對應輸出格式時才讀,平常不佔 context。

## 安裝

打包成 `.skill` 後在 Claude 裡安裝:

```bash
cd .. && zip -r dashboard.skill dashboard/ -x "*.git*"
```

把產出的 `dashboard.skill` 上傳到 Claude,點 Save skill。

## 調整

配色改 `SKILL.md` 的色票區塊與兩個參考檔裡的 `PALETTE` / CSS 變數(三處要一起改)。版型與規則直接改 `SKILL.md`。
