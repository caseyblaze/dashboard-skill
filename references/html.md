# HTML 輸出

## 選 SVG 還是 Chart.js

- 靜態、資料點固定、只要好看 → **手寫 inline SVG**。沒有相依、載入快、改起來直覺。
- 要 hover tooltip、切換序列、資料超過 50 點 → **Chart.js**(`https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js`)。
- 對話裡直接呈現的互動儀表板 → 單一 HTML,資料寫死在 `<script>` 裡的常數,不要 fetch 外部 API。
- 注意:對話內的產出不能用 localStorage / sessionStorage,狀態放 JS 變數就好。

## 樣式底(直接複製)

```html
<style>
:root{
  --bg:#fff; --surface:#F9FAFB; --border:#E5E7EB; --grid:#EEF0F3;
  --ink:#101828; --muted:#667085;
  --c1:#2563EB; --c2:#0E9F6E; --c3:#F59E0B; --c4:#7C3AED; --c5:#DC6803; --c6:#64748B;
  --pos:#0E9F6E; --neg:#DC2626; --dim:#CBD5E1;
  --pos-text:#047857;   /* --pos 對白底只有 3.39:1,寫字一律用這個 */
}
*{box-sizing:border-box}
body{margin:0;padding:24px;background:var(--bg);color:var(--ink);
  font:14px/1.5 -apple-system,"Segoe UI","Noto Sans TC",system-ui,sans-serif}
h1{font-size:20px;font-weight:600;margin:0 0 4px}
.sub{color:var(--muted);font-size:13px;margin-bottom:24px}
.kpis{display:grid;grid-template-columns:repeat(auto-fit,minmax(160px,1fr));gap:16px;margin-bottom:24px}
.card{background:var(--surface);border:1px solid var(--border);border-radius:8px;padding:20px}
.card .label{font-size:13px;color:var(--muted);margin-bottom:6px}
.card .value{font-size:28px;font-weight:600;letter-spacing:-.02em}
.card .delta{font-size:13px;margin-top:6px}
.up{color:var(--pos-text)} .down{color:var(--neg)} .flat{color:var(--muted)}
.panel{background:var(--bg);border:1px solid var(--border);border-radius:8px;padding:20px;margin-bottom:24px}
.panel h2{font-size:15px;font-weight:600;margin:0 0 16px}
.row2{display:grid;grid-template-columns:1fr 1fr;gap:24px}
.small-multiples{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:16px}
.small-multiples .cell{height:160px}
.small-multiples .cell-title{font-size:13px;color:var(--muted);margin-bottom:4px}
table{width:100%;border-collapse:collapse;font-size:13px}
th{text-align:left;color:var(--muted);font-weight:500;padding:8px 12px;border-bottom:1px solid var(--border)}
td{padding:8px 12px;border-bottom:1px solid var(--grid)}
td.num{text-align:right;font-variant-numeric:tabular-nums}
@media(max-width:640px){.row2{grid-template-columns:1fr}body{padding:16px}}
</style>
```

## 結構

```html
<h1>Q3 營收概況</h1>
<div class="sub">2026/07–09 ・ 單位:新台幣千元 ・ 更新於 9/11</div>

<div class="kpis">
  <!-- 有上一期資料:才可以放 ▲▼ -->
  <div class="card">
    <div class="label">總營收</div>
    <div class="value">12,480</div>
    <div class="delta up">▲ 18.2% vs Q2</div>
  </div>

  <!-- 沒有上一期資料:灰字寫限定範圍的事實,不要編一個比較 -->
  <div class="card">
    <div class="label">交易筆數</div>
    <div class="value">298</div>
    <div class="delta flat">全月 30 天</div>
  </div>
  <!-- 3–5 張 -->
</div>

<div class="panel">
  <h2>營收連 3 個月成長,9 月為今年新高</h2>
  <svg viewBox="0 0 800 320" style="width:100%;height:auto">…</svg>
</div>

<div class="row2">
  <div class="panel">…</div>
  <div class="panel">…</div>
</div>
```

## Chart.js 共用設定

```js
Chart.defaults.font.family = '-apple-system,"Segoe UI","Noto Sans TC",sans-serif';
Chart.defaults.color = '#667085';
Chart.defaults.borderColor = '#EEF0F3';
const opts = {
  responsive:true, maintainAspectRatio:false,
  plugins:{ legend:{display:false}, tooltip:{backgroundColor:'#101828',padding:10,cornerRadius:6,displayColors:false} },
  scales:{
    x:{ grid:{display:false}, border:{color:'#E5E7EB'} },
    y:{ beginAtZero:true, grid:{color:'#EEF0F3'}, border:{display:false},
        ticks:{ callback:v=>v.toLocaleString() } }
  }
};
```

折線要收斂一點:`borderWidth:2, pointRadius:0, pointHoverRadius:4, tension:0`(不要 spline,會扭曲數值)。長條 `borderRadius:4, barPercentage:.65`。

**一頁多張圖時,把 `opts` 改成工廠函式**,不要共用同一個物件。Chart.js 會持有你傳進去的 options,之後改 A 圖的 `scales.y.max` 會連帶改到 B 圖:

```js
const baseOpts = () => ({ /* 同上 */ });
const o = baseOpts(); o.scales.y.max = 100;   // 只影響這張
```

**small multiples**:每張小圖各自 `new Chart()`,但 `scales.y.max` 要**算好全體最大值後寫死同一個數**,不能讓它各自 autoscale。小圖不放軸標題和圖例,標題用 `.cell-title` 的純文字。

## 軸刻度的數字縮寫

預設的 `toLocaleString()` 遇到大數字會把 y 軸撐爆。一張圖只能用一種寫法:

```js
const fmtTW = v => Math.abs(v) >= 1e8 ? (v/1e8).toFixed(1) + '億'
               : Math.abs(v) >= 1e4 ? (v/1e4).toFixed(1) + '萬'
               : v.toLocaleString();
const fmtEN = v => Math.abs(v) >= 1e9 ? (v/1e9).toFixed(1) + 'B'
               : Math.abs(v) >= 1e6 ? (v/1e6).toFixed(1) + 'M'
               : Math.abs(v) >= 1e3 ? (v/1e3).toFixed(1) + 'K'
               : v.toLocaleString();

o.scales.y.ticks.callback = fmtTW;
o.scales.y.title = { display:true, text:'營收(新台幣)', color:'#667085' };
```

Tooltip 裡要給完整數字,縮寫只用在刻度。

## 多序列不要只靠顏色

顏色之外要再給一個線索,折線用 `borderDash`:

```js
const DASH = [[], [6,4], [2,3], [8,3,2,3]];   // 實線 / 虛線 / 點線 / 點劃線
datasets.forEach((d, i) => { d.borderDash = DASH[i % DASH.length]; });
```

## 無障礙

- 圖的容器給 `role="img"` 和一句寫結論的 `aria-label`,不要寫「長條圖」:

```html
<div class="chart h-main" role="img"
     aria-label="6/20 單日 121 筆,佔全月四成,其餘日期多在 20 筆以下">
  <canvas id="daily"></canvas>
</div>
```

  `<canvas>` 對讀螢幕的人是空白,沒有 `aria-label` 等於整張圖不存在。結論式標題已經寫好了,直接拿去用。

- 明細表就是資料表備援,`<details>` 可以收合但不要藏。
- 最小字級 13px,`--dim` (`#CBD5E1`,1.48:1) 只能填色不能寫字。

## 手寫 SVG 提醒

- `viewBox` 固定、寬度給 100%,才會跟著版面縮放。
- 文字直接用 `<text fill="#667085" font-size="12">`,不要靠 CSS class 傳到匯出環境會掉。
- 座標軸自己算:留 `padding-left 56 / bottom 32` 給刻度。
- 長條圖的值直接標在長條末端,可以省掉 y 軸。
