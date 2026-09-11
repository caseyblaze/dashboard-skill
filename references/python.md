# Python 出圖(matplotlib)

## 樣式底(每次開頭跑一次)

```python
import matplotlib as mpl, matplotlib.pyplot as plt

PALETTE = ['#2563EB','#0E9F6E','#F59E0B','#7C3AED','#DC6803','#64748B']
INK, MUTED, GRID, DIM = '#101828', '#667085', '#EEF0F3', '#CBD5E1'
POS, NEG = '#0E9F6E', '#DC2626'
POS_TEXT = '#047857'      # POS 對白底只有 3.39:1,要寫字一律換這個(5.48:1)

# 有序資料用漸層,不要拿 PALETTE 去畫
SEQ = ['#DBEAFE','#93C5FD','#3B82F6','#1D4ED8','#1E3A8A']                    # 單向
DIV = ['#B45309','#F59E0B','#FDE68A','#F1F5F9','#BFDBFE','#3B82F6','#1E40AF']  # 有中點

from matplotlib.colors import LinearSegmentedColormap
CMAP_SEQ = LinearSegmentedColormap.from_list('seq', SEQ)
CMAP_DIV = LinearSegmentedColormap.from_list('div', DIV)   # 記得 vmin=-x, vmax=+x 對稱

mpl.rcParams.update({
    'figure.facecolor':'white', 'axes.facecolor':'white',
    'figure.dpi':110, 'savefig.dpi':200, 'savefig.bbox':'tight',
    'font.family':'sans-serif',
    'font.sans-serif':['Noto Sans CJK TC','Noto Sans TC','DejaVu Sans'],
    'font.size':11, 'text.color':INK,
    'axes.edgecolor':'#E5E7EB', 'axes.linewidth':1,
    'axes.spines.top':False, 'axes.spines.right':False, 'axes.spines.left':False,
    'axes.titlesize':13, 'axes.titleweight':'600', 'axes.titlelocation':'left',
    'axes.titlepad':14, 'axes.labelcolor':MUTED, 'axes.labelsize':11,
    'axes.prop_cycle': mpl.cycler(color=PALETTE),
    'axes.grid':True, 'axes.axisbelow':True,
    'grid.color':GRID, 'grid.linewidth':1,
    'xtick.color':MUTED, 'ytick.color':MUTED,
    'xtick.direction':'out', 'ytick.direction':'out',
    'xtick.major.size':0, 'ytick.major.size':0,
    'legend.frameon':False, 'legend.fontsize':11,
})
```

只留水平格線:每張圖畫完加 `ax.xaxis.grid(False)`。

中文字型如果缺,先 `apt-get install -y fonts-noto-cjk`,再 `mpl.font_manager._load_fontmanager(try_read_cache=False)`。裝不起來就退回英文標籤,不要讓圖上出現豆腐方塊。

## 補齊時間軸時不要順手補 0

`reindex(full_range, fill_value=0)` 會把「沒有資料」變成「值是 0」,平均數和總數跟著被污染。先分清楚兩者:

```python
s = raw.reindex(full_range)              # 缺的地方留 NaN,不要 fill_value=0
真的是零 = s.fillna(0)                    # 確認過該期有在運作、只是沒交易,才補 0
ax.plot(s.index, s.values)               # 折線遇到 NaN 會自然斷開,這是對的
```

長條圖畫 NaN 會是空缺(正確),畫 0 會是一根貼地的短棒(看起來像「有在跑只是很低」)。分不清楚就在圖下方寫一句說明。

## 常用尺寸

| 用途 | figsize |
|---|---|
| 主圖 / 投影片滿版 | `(10, 4.5)` |
| 並排輔圖 | `(5.5, 3.5)` |
| 橫向長條(依類別數) | `(8, 0.4 * n + 1.2)` |

## 三個常用樣板

**橫向長條(排序 + 強調第一名)**

```python
d = df.sort_values('value')                      # 由小到大,畫出來最大在最上面
colors = [DIM] * len(d); colors[-1] = PALETTE[0]

fig, ax = plt.subplots(figsize=(8, 0.4*len(d)+1.2))
ax.barh(d['name'], d['value'], color=colors, height=.62)
ax.set_title('北區貢獻 38% 營收,是第二名的兩倍')
ax.xaxis.grid(False); ax.set_xticks([])          # 直接標值就不需要軸
for y, v in enumerate(d['value']):
    ax.text(v*1.01, y, f'{v:,.0f}', va='center', color=MUTED, fontsize=10)
ax.margins(x=.12)
```

**折線(末端直接標註,不用圖例)**

```python
fig, ax = plt.subplots(figsize=(10, 4.5))
for i, (name, s) in enumerate(series.items()):
    ax.plot(s.index, s.values, lw=2, color=PALETTE[i], solid_capstyle='round')
    ax.annotate(name, (s.index[-1], s.values[-1]), xytext=(8, 0),
                textcoords='offset points', va='center',
                color=PALETTE[i], fontsize=11, fontweight='600')
ax.xaxis.grid(False)
ax.set_ylim(bottom=0)                             # 折線可不從 0,但要刻意決定
ax.yaxis.set_major_formatter(lambda v, p: f'{v:,.0f}')
ax.margins(x=.14)                                 # 留空間給末端標註
```

**small multiples(超過 4 條線就改用這個)**

```python
n = len(groups)
ncol = min(4, n); nrow = -(-n // ncol)
fig, axes = plt.subplots(nrow, ncol, figsize=(3*ncol, 2.2*nrow),
                         sharex=True, sharey=True)      # sharey 是重點,不能省
ymax = max(s.max() for s in groups.values()) * 1.1      # 全體共用刻度

for ax, (name, s) in zip(axes.flat, groups.items()):
    ax.plot(s.index, s.values, lw=1.8, color=PALETTE[0])
    ax.set_title(name, fontsize=11, color=MUTED, pad=6)
    ax.set_ylim(0, ymax)
    ax.xaxis.grid(False)
for ax in axes.flat[n:]:
    ax.set_visible(False)                               # 多出來的格子藏掉
fig.suptitle('六個站點裡只有南港在成長', x=.06, ha='left', fontsize=14, fontweight='600')
```

要凸顯其中一個就把它上 `PALETTE[0]`、其餘 `DIM`,並在每張小圖畫一條全體平均的灰虛線當共同基準。

## 軸刻度的數字縮寫

大數字不縮寫,y 軸會被 `12,480,000` 這種刻度撐爆。**一張圖只能用一種寫法。**

```python
import matplotlib.ticker as mticker

def fmt_tw(v, _=None):                  # 中文報表:萬 / 億
    for div, unit in ((1e8, '億'), (1e4, '萬')):
        if abs(v) >= div:
            return f'{v/div:,.1f}{unit}'
    return f'{v:,.0f}'

def fmt_en(v, _=None):                  # 技術指標 / 國際場合:K / M / B
    for div, unit in ((1e9, 'B'), (1e6, 'M'), (1e3, 'K')):
        if abs(v) >= div:
            return f'{v/div:,.1f}{unit}'
    return f'{v:,.0f}'

ax.yaxis.set_major_formatter(mticker.FuncFormatter(fmt_tw))
ax.set_ylabel('營收(新台幣)')          # 單位寫在軸標題,不要每個刻度都寫
```

## 誤差線與不確定性

抽樣、預測、外推的數字只畫一個點,等於宣稱精確度比實際高。

```python
# 長條 + 誤差線
ax.bar(d['name'], d['mean'], yerr=d['ci'], color=PALETTE[0],
       error_kw={'ecolor': MUTED, 'elinewidth': 1, 'capsize': 3})

# 折線 + 信賴區間 / 預測帶
ax.plot(x, mu, lw=2, color=PALETTE[0])
ax.fill_between(x, lo, hi, color=PALETTE[0], alpha=.15, linewidth=0)

# 預測段落改虛線,跟實際值分開
ax.plot(x_fcst, y_fcst, lw=2, ls='--', color=PALETTE[0])
```

樣本數小就標出來:`ax.text(..., f'n={n}', color=MUTED, fontsize=10)`。

## 黑白與色盲

六色色序的相對亮度是 紫 0.134 / 藍 0.153 / 灰 0.171 / 橘 0.251 / 綠 0.260 / 黃 0.439 —— 紫藍灰三個幾乎同色,橘綠只差 0.009。**印出來就分不出來。**

所以多序列的圖,顏色之外一定要再給一個線索:

```python
STYLES = ['-', '--', ':', '-.']
for i, (name, s) in enumerate(series.items()):
    ax.plot(s.index, s.values, lw=2,
            color=PALETTE[i], ls=STYLES[i % len(STYLES)])
```

檢查方式:存檔後轉灰階看還讀不讀得懂。

```python
from PIL import Image
Image.open(path).convert('L').save(path.replace('.png', '-gray.png'))
```

## 多圖拼成一張儀表板

```python
fig = plt.figure(figsize=(12, 9))
gs = fig.add_gridspec(2, 2, height_ratios=[1.3, 1], hspace=.45, wspace=.25)
ax_main = fig.add_subplot(gs[0, :])      # 主圖佔滿上排
ax_l, ax_r = fig.add_subplot(gs[1,0]), fig.add_subplot(gs[1,1])
fig.suptitle('Q3 營收概況', x=.09, y=.97, ha='left', fontsize=17, fontweight='600')
fig.text(.09, .935, '2026/07–09 ・ 單位:新台幣千元', ha='left', color=MUTED, fontsize=11)
```

KPI 數字在 matplotlib 裡硬做很醜。要 KPI 卡就改走 HTML;真的得出成圖檔,就把關鍵數字寫進各圖的結論式標題。

存檔:`fig.savefig(path, dpi=200, bbox_inches='tight', facecolor='white')`,存完用 `present_files` 交給使用者。
