# dashboard

一個 Claude Skill:用固定配色與固定版型產出儀表板,支援互動式 HTML 與靜態圖檔兩種輸出。

## 為什麼

每次叫模型畫圖,配色、版型、圖表類型都要重講一次,結果還不一致。這個 skill 把決定寫死:

- 淺色商務配色,6 色資料色序 + 正負向語意色
- 固定版型:標題 → KPI 卡 → 主圖 → 雙欄輔圖 → 明細表
- 選圖邏輯表:先問「這張圖要回答什麼問題」,再對應圖表類型
- 呈現規則:長條一律排序、y 軸從 0、不用雙 Y 軸、標題寫結論句
- KPI 一定要有比較基準,**但沒有基準時留白,不准編一個**

規則背後還放了一組**判斷基準**(誠實 > 感知準確度 > 前注意處理 > 精簡 > 敘事),用途是遇到規則表沒寫到的狀況時有得裁決,而且衝突時知道誰讓誰。來源是 Cleveland & McGill 的感知精度排序、完形法則、Tufte 的 lie factor 與 data-ink、Cairo 與 Knaflic 的敘事、Few 的儀表板主張。

## 結構

```
dashboard/
├── SKILL.md              # 判斷基準 / 配色 / 版型 / 選圖邏輯 / 呈現規則(觸發時載入)
└── references/
    ├── html.md           # HTML 輸出:CSS tokens、結構、Chart.js 設定
    └── python.md         # 靜態出圖:matplotlib rcParams、常用樣板
```

參考檔只在需要對應輸出格式時才讀,平常不佔 context。

## 安裝

**Claude Code** —— clone 到 skills 目錄就生效,不用打包:

```bash
git clone https://github.com/caseyblaze/dashboard-skill ~/.claude/skills/dashboard
```

更新:`git -C ~/.claude/skills/dashboard pull`。要開新的 session 才會載入。

目錄名建議就叫 `dashboard`,跟 `SKILL.md` 的 `name:` 對齊,之後找起來不會困惑。

**claude.ai 網頁版** —— 需要打包成 `.skill` 上傳:

```bash
git clone https://github.com/caseyblaze/dashboard-skill dashboard
zip -r dashboard.skill dashboard/ -x "*.git*"
```

上傳 `dashboard.skill`,點 Save skill。第二行的 `dashboard/` 是第一行 clone 出來的目錄名,兩行要對得起來。

## 調整

配色改 `SKILL.md` 的色票區塊與兩個參考檔裡的 `PALETTE` / CSS 變數(三處要一起改)。版型與規則直接改 `SKILL.md`。
