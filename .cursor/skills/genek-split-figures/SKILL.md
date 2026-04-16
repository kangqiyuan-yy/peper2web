---
name: genek-split-figures
description: 识别科研论文 Figure 中子图(panel)的边界并裁剪。采用 LLM 语义分组 + 像素精修的混合方案，输出每个 panel 组的精确矩形坐标并生成可视化验证图。当用户提及"子图边界"、"panel 识别"、"figure 拆分"、"提取子图"、"split figure"时使用此技能。
---

# 科研 Figure 子图边界识别

支持两种模式：
- **自动模式**：LLM 看图自行判断分组，**限制为 2–3 组（不超过 3 个）**
- **用户指定模式**：用户给出分组方案（如 `A, DEFG, BCH`），LLM 据此定位，组数不受限制

采用混合方案：语义理解（确定每个 panel 位置）+ 像素级白色间隙分析（精确定位边界）。

## 目录约定

与 `genek-extract-figures` 共用 `papers/{Author}{Year}/` 结构：

- **输入**：`papers/{Author}{Year}/figures/fig{N}.png`（原图）
- **输出**：`papers/{Author}{Year}/figures/panels/fig{N}_{panels}.png`（拆分后的 panel）

```
papers/Li2022/
├── paper.pdf
├── figures/
│   ├── fig1.png                  ← 输入（extract-figures 产出）
│   ├── fig2.png
│   └── panels/                   ← 输出目录
│       ├── fig1_A.png            ← 单 panel
│       ├── fig1_DEFG.png         ← 多 panel 合并为一组
│       ├── fig2_BC.png
│       └── ...
```

Panel 文件命名：`fig{N}_{panels}.png`
- 单 panel：`fig1_A.png`
- 多 panel 合为一组：`fig1_BC.png`、`fig1_DEFG.png`（按原 panel 标签排列拼接）

**严格要求（禁止规避）**：
- 必须按下方流程**完整执行**：确定分组 → LLM 看图定位 → 像素精修 → 裁剪保存 → 验证图检查。不得仅将原图复制或重命名为「整图当一 panel」作为输出。
- **禁止**使用 `_whole`、`_full`、`_all` 等占位组名（如 `fig1_whole.png`）。输出文件的 `{panels}` 必须为论文图中**真实存在的 panel 标签**（如 A、B、BC、DEFG），与图中标注一致。

## 流程

```
1. 确定分组方案（用户指定 或 LLM 自动判断）
2. LLM 看图 → 定位每个 panel 的位置 + 给出每组粗坐标
3. 像素分析 → 运行 analyze_image，解读间隙输出，与粗坐标对齐确定精确裁剪坐标
4. 裁剪并保存到 papers/{Author}{Year}/figures/panels/
5. 生成带彩色边框的验证图 → Read 工具逐张查看
6. 如有偏差 → 调整坐标，重新裁剪 + 验证
7. 全部确认后删除验证图
```

## 第一步：确定分组 + LLM 定位

### 模式 A：用户指定分组

用户输入示例：`从 fig1.png 中提取 A, DEFG, BCH`

解析规则：
- 每个逗号分隔项为一组，组名即 panel 标签拼接（如 `BCH` = 面板 B + C + H）
- LLM 看图后定位每个**单独 panel** 的矩形范围
- 每组的边界 = 组内所有 panel 的最小外接矩形（min x1, min y1, max x2, max y2）

输出定位表：

```
单 panel 定位：
| Panel | 位置 (x1, y1, x2, y2) |
|-------|----------------------|
| A     | (0, 0, 545, 570)    |
| B     | (0, 580, 540, 940)   |
| C     | (540, 580, 1928, 940)|
| D     | (545, 0, 1240, 310)  |
| E     | (1240, 0, 1928, 310) |
| F     | (545, 310, 1240, 570) |
| G     | (1240, 310, 1928, 570)|
| H     | (0, 950, 1928, 1310) |

用户分组聚合：
| 组名  | 包含 panel | 聚合坐标              |
|-------|-----------|----------------------|
| A     | A         | (0, 0, 545, 570)     |
| DEFG  | D,E,F,G   | (545, 0, 1928, 570)  |
| BCH   | B,C,H     | (0, 580, 1928, 1310) |
```

### 模式 B：LLM 自动分组（2–3 组硬上限）

自动模式的核心约束：**每张 Figure 最终输出 2 或 3 个子图组，绝不超过 3 个。**

LLM 看图后按以下步骤输出分组表：

1. 识别 Figure 中所有 panel（A, B, C, D, …）
2. 将全部 panel 合并为 **2–3 个组**，优先合并而非拆分
3. 若 panel 数 ≤ 3，每个 panel 可单独成组
4. 若 panel 数 > 3，必须聚合到 ≤ 3 组（按下方合并策略）

```
| 组名  | 包含 panel  | 分组理由                    |
|-------|-----------|---------------------------|
| ABC   | A, B, C   | 上层行：系统发育树 + 质量条形图 + 表格  |
| DEFG  | D, E, F, G | 中层行：4 张相关散点图           |
| H     | H         | 底层行：独立 3D 建模图          |
```

**合并策略**（按优先级）：
1. **空间邻近**：同一水平行或垂直列的 panel 优先合并
2. **内容相似**：类型相近的图（如多张散点图、多张热图）合并
3. **共享元素**：共用图例、共用坐标轴、箭头连接的 panel 合并
4. **面积均衡**：避免组间面积差异过大（如一组占 80%，另一组占 20%）

**禁止**：
- 输出 > 3 个组（违反硬上限）
- 输出 1 个组（无意义拆分，至少 2 个）
- 将不相邻、不相关的 panel 强行合并仅为凑数

两种模式共同输出每组的粗坐标估计 `(left, top, right, bottom)`。

## 第二步：像素精修

用 Python + PIL + numpy，对图像做白色间隙检测。

### 核心算法

```python
from PIL import Image
import numpy as np

def find_gaps(arr_1d, min_len=10, thresh=252):
    """在一维亮度数组中找连续白色区段。"""
    gaps = []
    i, n = 0, len(arr_1d)
    while i < n:
        if arr_1d[i] > thresh:
            start = i
            while i < n and arr_1d[i] > thresh:
                i += 1
            if i - start >= min_len:
                gaps.append((start, i, i - start))
        else:
            i += 1
    return gaps

def analyze_image(img_path):
    gray = np.array(Image.open(img_path).convert('L'))
    h, w = gray.shape
    row_mean = gray.mean(axis=1)
    h_gaps = find_gaps(row_mean, min_len=12)
    # 过滤边缘间隙
    h_interior = [(s, e, l) for s, e, l in h_gaps if s > 10 and e < h - 10]

    # 按间隙大小排序，取前 4-5 个主要间隙定义水平分带
    h_interior.sort(key=lambda x: x[2], reverse=True)
    major_h = sorted(h_interior[:5], key=lambda x: x[0])

    # 对每个水平带做垂直间隙分析
    edges = [0]
    for s, e, _ in major_h:
        edges.extend([s, e])
    edges.append(h)

    bands = []
    for i in range(0, len(edges) - 1, 2):
        y1, y2 = edges[i], edges[i + 1]
        if y2 - y1 < 40:
            continue
        band_col = gray[y1:y2, :].mean(axis=0)
        v_gaps = find_gaps(band_col, min_len=8)
        v_interior = [(s, e, l) for s, e, l in v_gaps if s > 10 and e < w - 10]
        v_interior.sort(key=lambda x: x[2], reverse=True)
        bands.append({'y': (y1, y2), 'v_gaps': v_interior[:3]})

    return w, h, major_h, bands
```

### 精修策略

1. **水平分界**：取主要 H gap 的 `(start, end)` → 上方 panel 底边 = `start`，下方 panel 顶边 = `end`
2. **垂直分界**：在每个水平带内独立计算 → 取最大 V gap 的 `(start, end)`
3. **对齐粗坐标**：LLM 粗坐标用于选择"哪个间隙是真正的 panel 分界"（而非 panel 内部空白）

### 关键注意事项

- **panel 内部也有空白**（如 fig1 的 Contig N50 和 Scaffold N50 之间的 "//" 断裂），不能盲目选最大间隙
- **数字表格会产生大量假水平间隙**（如 fig4 的 CTCF 数值表），需要 LLM 判断哪些间隙是真实分界
- **不同水平带的垂直分界位置往往不同**（如 fig1 上半部分 A|DEFG 分界在 x=845，中间部分 B|C 分界完全不同）

### 实操流程

对每张 Figure 运行 `analyze_image`，输出格式如下：

```
=== fig1.png (1928x2000) ===
Top H gaps (y): [(951, 1007, 56), (1494, 1543, 49)]
Top V gaps (x): []
  Band y=[0:951] V gaps: [(866, 891, 25), (1419, 1437, 18)]
  Band y=[1007:1494] V gaps: [(511, 566, 55)]
  Band y=[1543:2000] V gaps: [(1107, 1168, 61), (730, 769, 39)]
```

解读方法：

1. **H gaps → 水平分带**：`(951, 1007)` 表示 y=951~1007 是白色间隙，将图像分成上 `[0:951]`、中 `[1007:1494]`、下 `[1543:2000]` 三个水平带
2. **每个带内的 V gaps → 垂直分界**：上层带 V gap `(866, 891)` 表示 x=866~891 是垂直白色间隙
3. **与 LLM 粗坐标对齐**：LLM 判断上层带左侧为 A、右侧为 DEFG → 精确边界为 A 右边 = x=866，DEFG 左边 = x=891
4. **确定裁剪坐标**：`A = (0, 0, 866, 951)`，`DEFG = (891, 0, 1928, 951)`

关键原则：
- 间隙的 `start` 是上/左 panel 的底/右边界，`end` 是下/右 panel 的顶/左边界
- 选择哪个间隙作为分界由 LLM 粗坐标决定（选最接近粗坐标的间隙）
- 如果一个带内没有 V gap，说明该带只有一个 panel 跨全宽

## 第三步：裁剪并保存

确定所有坐标后，批量裁剪：

```python
from PIL import Image
import os

def crop_panels(fig_dir, panel_dir, fig_num, panels):
    """
    panels: dict of {group_name: (x1, y1, x2, y2)}
    例如: {"A": (0, 0, 866, 951), "DEFG": (891, 0, 1928, 951)}
    """
    os.makedirs(panel_dir, exist_ok=True)
    img = Image.open(f"{fig_dir}/fig{fig_num}.png")
    for name, box in panels.items():
        cropped = img.crop(box)
        out = f"{panel_dir}/fig{fig_num}_{name}.png"
        cropped.save(out)
        size = os.path.getsize(out)
        print(f"  fig{fig_num}_{name}.png: {cropped.size[0]}x{cropped.size[1]} ({size} bytes)")
```

调用示例（以 Li2022 Fig 1 为例）：

```python
fig_dir = "papers/Li2022/figures"
panel_dir = "papers/Li2022/figures/panels"

crop_panels(fig_dir, panel_dir, 1, {
    "A":    (0,   0,    866, 951),
    "DEFG": (891, 0,    1928, 951),
    "B":    (0,   1007, 511, 1494),
    "C":    (566, 1007, 1928, 1494),
    "H":    (0,   1543, 1928, 2000),
})
```

## 第四步：可视化验证

```python
from PIL import Image, ImageDraw, ImageFont

COLORS = ["#E74C3C", "#2980B9", "#27AE60", "#8E44AD", "#E67E22", "#1ABC9C"]
LINE_W = 5

def draw_boundaries(img_path, panels, out_path):
    """panels: dict of {name: (x1, y1, x2, y2)}"""
    img = Image.open(img_path).convert('RGB')
    draw = ImageDraw.Draw(img)
    try:
        font = ImageFont.truetype("/System/Library/Fonts/Helvetica.ttc", 28)
    except Exception:
        font = ImageFont.load_default()

    for i, (name, box) in enumerate(panels.items()):
        c = COLORS[i % len(COLORS)]
        x1, y1, x2, y2 = box
        for j in range(LINE_W):
            draw.rectangle([x1 + j, y1 + j, x2 - j, y2 - j], outline=c)
        bb = draw.textbbox((0, 0), name, font=font)
        tw, th = bb[2] - bb[0], bb[3] - bb[1]
        tx, ty = x1 + 8, y1 + 8
        draw.rectangle([tx - 2, ty - 2, tx + tw + 4, ty + th + 4],
                        fill='white', outline=c)
        draw.text((tx, ty), name, fill=c, font=font)
    img.save(out_path)
```

### 验证流程

1. 对每张 Figure 调用 `draw_boundaries` 生成 `fig{N}_verify.png`
2. 用 Read 工具逐张查看验证图，确认彩色边框准确框住对应 panel
3. 如有问题：
   - 框太大（侵入相邻 panel）→ 缩小对应边
   - 框太小（截断内容）→ 检查是否选错了间隙，或 panel 内部有大空白被误判为分界
   - 框位置完全错误 → 重新看图修正 LLM 分组
4. 修正后重新裁剪（第三步）并重新验证
5. 全部确认后删除验证图：`rm papers/{Author}{Year}/figures/panels/*_verify.png`

## 经验教训（来自实测）

| 问题 | 原因 | 解决方法 |
|------|------|---------|
| Scaffold N50 柱状图被截断 | 内部边框线被误判为 panel 分界 | 在粗坐标附近多检查几个间隙候选 |
| fig4 被切成 11 块 | 数字表格的行间空白被当成 panel 间隙 | LLM 先确定"应该有几个分区" |
| OCR 漏检 panel 标签 | Tesseract 对粗体单字母识别率仅 60-70% | 不依赖 OCR，以 LLM 视觉为主 |
| 纯像素分析过度切分 | 无语义理解 | 必须 LLM 先定分组，再用像素精修 |
