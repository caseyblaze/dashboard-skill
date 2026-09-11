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

### Claude Code

clone 到 skills 目錄就生效,不用打包:

```bash
git clone https://github.com/caseyblaze/dashboard-skill ~/.claude/skills/dashboard
```

更新:`git -C ~/.claude/skills/dashboard pull`。要開新的 session 才會載入。

### Claude Desktop / claude.ai 網頁版

**先開權限。** Settings → Capabilities,把 **Code execution and file creation** 打開。沒開的話 skill 上傳得進去但不會動。

打包:

```bash
git clone https://github.com/caseyblaze/dashboard-skill dashboard
zip -r dashboard.zip dashboard/ -x "*.git*"
```

上傳:

| | 路徑 |
|---|---|
| Claude Desktop | Settings → Capabilities → Skills → 上傳 `dashboard.zip` |
| claude.ai 網頁版 | Customize → Skills → `+` → Create skill → Upload a skill |

傳完確認清單裡那個開關是**開**的,然後**開一個新對話** —— skill 清單是在對話開始時載入的,已經開著的對話不會追加。

**兩個最容易失敗的地方:**

1. **zip 裡要包一層資料夾。** `SKILL.md` 不能躺在 zip 根目錄,要在 `dashboard/` 底下。上面的指令 zip 的是目錄本身,結構是對的。
2. **資料夾名要跟 `SKILL.md` 的 `name:` 一致**,兩邊都是 `dashboard`。所以 clone 那行才要手動指定目錄名 —— 直接 `git clone <url>` 會得到 `dashboard-skill`,跟 `name: dashboard` 對不起來,上傳會被退件。

## 調整

配色改 `SKILL.md` 的色票區塊與兩個參考檔裡的 `PALETTE` / CSS 變數(三處要一起改)。版型與規則直接改 `SKILL.md`。
