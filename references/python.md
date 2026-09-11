# Python 出圖(matplotlib)

## 樣式底(每次開頭跑一次)

```python
import matplotlib as mpl, matplotlib.pyplot as plt

PALETTE = ['#2563EB','#0E9F6E','#F59E0B','#7C3AED','#DC6803','#64748B']
INK, MUTED, GRID, DIM = '#101828', '#667085', '#EEF0F3', '#CBD5E1'
POS, NEG = '#0E9F6E', '#DC2626'

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

## 常用尺寸

| 用途 | figsize |
|---|---|
| 主圖 / 投影片滿版 | `(10, 4.5)` |
| 並排輔圖 | `(5.5, 3.5)` |
| 橫向長條(依類別數) | `(8, 0.4 * n + 1.2)` |

## 兩個常用樣板

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
