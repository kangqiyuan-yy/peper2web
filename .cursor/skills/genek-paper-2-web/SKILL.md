---
name: genek-paper-2-web
description: 从论文生成交互式 HTML 网页（page.html）。三列 Grid 响应式布局 + 交互组件，原图展示，中英双语切换。当用户提及"论文转网页"、"学术网页"、"paper to web"时使用此技能。
---

# Paper2Web — 论文 → 交互式网页

从单篇论文生成一份交互式 HTML 网页。使用 **原图**（`figures/fig{N}.png`），采用三列 Grid 响应式布局 + Lightbox + 中英双语切换，适合在电脑浏览器中查看。

## 适用边界

- `genek-paper-2-web` 定义网页的内容结构、布局规则、组件用法与验收门禁
- 不涉及 PPT 生成（PPT 由 `genek-paper-2-slide` 负责）
- 图片提取与 panel 拆分由上游 skill 完成

## 目录结构

每篇论文以 **第一作者姓氏 + 年份** 命名独立子目录：

```
papers/
└── {Author}{Year}/
    ├── paper.pdf
    ├── figures/
    │   ├── fig1.png ... figN.png            (原图，genek-extract-figures)
    │   └── panels/                          (panel 图，genek-split-figures)
    └── page.html                            (本 skill 产物)
```

## 执行流程

| 步骤 | 目标 | 输入 | 输出 | 成功判据 |
|------|------|------|------|----------|
| **S0** | 建立论文目录 | `paper.pdf` | `papers/{Author}{Year}/` | 目录结构正确 |
| **S1** | 抽取结构化内容 | `paper.pdf` | 标题/作者/Background/Design/Methods/Results/Conclusions | 字段齐全，无空段 |
| **S2** | 提取原图 | `paper.pdf` | `figures/fig1..figN.png` | 图片数与主图数一致 |
| **S3** | 生成网页 | 结构化内容 + 原图 | `page.html` | 页面可打开，图文完整 |
| **S4** | QA 门禁 | page.html | 通过/不通过 | 满足本文件 QA Gate |

失败策略：
- S2 失败：回退到 `genek-extract-figures` 的备选路径

## 图片使用规则

| 格式 | 使用图片 | 路径 | Result 对应 |
|------|---------|------|------------|
| page.html | **原图** | `figures/fig{N}.png` | **1 张原图 = 1 个 Result 卡片** |

每个 Result 卡片对应论文的一张完整 Figure 原图。Result 数量 = 论文主图数量（通常 4–7 张）。网页空间充裕，用原图展示完整 Figure 更直观。

## 项目设计（Study Design）

从论文 Methods/Results 中提取实验设计信息，以**表格 + 关键统计数字**呈现。

常见字段（按论文实际内容选取）：

| 字段 | 适用场景 | 示例 |
|------|---------|------|
| 物种 (Species) | 动物/植物/微生物研究 | Human, Mouse, Chicken 等 |
| 样本类型 (Sample) | 所有实验研究 | Fibroblast cell lines, Tumor tissue, Serum |
| 样本数量 (Replicates) | 所有实验研究 | 1–5 replicates per species |
| 患者/个体 (Cohort) | 临床/群体研究 | 500 patients, 1200 healthy controls |
| 实验技术 (Technology) | 所有组学研究 | Hi-C, RNA-seq, ATAC-seq, WGS |
| 测序平台 (Platform) | 测序类研究 | Illumina HiSeq X Ten |
| 测序量 (Sequencing depth) | 测序类研究 | ~230M contacts per library |
| 分辨率 (Resolution) | 组学数据分析 | 20 kb, single-cell |
| 参考基因组 (Reference) | 基因组学研究 | GRCh38, GRCm38 等 |
| 时间点 (Timepoints) | 纵向/发育研究 | E8.5, E10.5, E14.5, P0, Adult |
| 处理条件 (Treatment) | 干预/药物研究 | Control vs. Drug (10 μM, 48h) |

如论文涉及多物种/多样本，优先提取为一张汇总表。

### 多物种研究的进化关系要求

当论文涉及 **≥3 个物种的跨物种比较** 时，Study Design 中必须包含：

1. **简易系统发育树（cladogram）**：用 CSS 组件展示物种间拓扑关系
2. **类群分组着色**：用颜色区分主要分类群
3. **外群标识**：明确标出外群物种

组件实现见 [web-components.md → Cladogram](references/poster-components.md)。

不涉及跨物种比较的论文不需要 cladogram。

## 网页布局规则

### 三列 Grid 布局原则

网页采用响应式 3 列 grid（max-width 约 1800px）。各区域固定分配：

```
         Col 1                Col 2                    Col 3
Header   ──────────────────── 全宽 ─────────────────────
Row 1    Background           Study Design (跨 2–3 列)
Row 2    Methods              Result 1                 Result 2
Row 3    Conclusions          Result 3                 Result 4
Row 4                         Result 5                 Result 6
Footer   ──────────────────── 全宽 ─────────────────────
```

**第 1 行 = 概览行**：Background（Col 1）和 Study Design（Col 2–3）并排。Study Design 跨 2 列以容纳表格和 cladogram。

**第 1 列 Row 2 = Methods**：使用**分阶段流程图（Pipeline）**替代文字。分 3 个视觉阶段：Stage 1（实验，深色渐变）→ Stage 2（数据处理，浅灰 + tool-tag 标签）→ Stage 3（多维分析，白底蓝边分组框 + 2×N 分析卡片网格）。每步必须标注所使用的**软件工具名称**。详见 [web-components.md → 分阶段流程图](references/poster-components.md)。

**第 1 列 Row 3 = Conclusions**：深色渐变背景 + 编号列表，作为第 1 列的视觉终点。

**第 2–3 列 = Result 区域**：从 Row 2 开始，严格按 **从左到右、从上到下** 排列：
- 2 个 Result → 1 行 × 2 列
- 3–4 个 → 2 行 × 2 列（奇数时最后一行只占 1 格）
- 5–6 个 → 3 行 × 2 列
- ≥7 个 → 考虑合并相近的 Result

### 视觉风格

采用 **Editorial / 学术期刊编辑风格**（参考 `frontend-design` skill），具体规格如下：

**字体**
- 标题：`Cormorant Garamond`（Google Fonts，wght 400/600/700/900）
- 正文：`Lora`（Google Fonts，wght 400/500/600）

**配色**
```css
--bg:      #f7f3ed;   /* 羊皮纸暖米色主背景 */
--bg-warm: #f0e9df;   /* 卡片次背景 */
--card-bg: #faf8f4;   /* 卡片白底 */
--border:  #d4c9b8;   /* 边框 */
--crimson: #8b1a1a;   /* 主强调色（深绯红） */
--gold:    #c9a84c;   /* 副强调色（古金） */
--dark:    #1a1208;   /* 主文字色 */
--muted:   #6b5d4f;   /* 次文字色 */
```

**视觉规则**
- Header：羊皮纸色 + 深绯红大字标题（`Cormorant Garamond`），顶部三色渐变装饰横条，底部金色虚线
- Section 卡片：白底、1px `#d4c9b8` 边框、顶部 3px 绯红色线
- Section 标题：`Cormorant Garamond`，绯红色，金色下划线（`border-bottom: 1px solid var(--gold)`）
- 每个 Result 使用**独立 accent 色**作为顶部边框色线，通过颜色区分不同发现
- Result 卡片带**编号圆标**（绝对定位），引导阅读顺序
- 统计数字框（Stat box）：暖米色背景，左侧金色边线，数字用绯红色 `Cormorant Garamond` 大字
- Finding 卡片：羊皮纸背景 + 绯红左边线，加大号装饰引号
- 关键 Figure **默认展开**（`<details open>`），次要 Figure 折叠；summary 用绯红色文字
- Conclusions：深绯红渐变背景（`#8b1a1a → #5c2d0e`）+ 金色大数字编号
- Footer：深暖棕背景（`#2c1a0a`）+ 双线金色顶部边框
- 语言切换按钮：绯红色激活态

**禁止使用**：Inter / Roboto / Arial / Space Grotesk 等泛用无衬线字体；蓝紫渐变配色；纯白 `#ffffff` 背景。

## 双语要求

- `data-lang="en"` / `data-lang="zh"` 标记文本
- 右上角语言切换按钮，默认英文
- 无需双语标记的元素：数字/统计值、图片、DOI、作者姓名

## 依赖 skill

- **图片提取**：`genek-extract-figures` → 原图到 `figures/`

## 组件与参考

- 网页布局与交互组件 → [references/poster-components.md](references/poster-components.md)
- QA 门禁清单 → [references/qa-checklists.md](references/qa-checklists.md)

## QA 门禁（交付前必须通过）

1) 资产完整性
- `figures/fig{N}.png` 连续且可读
- 不存在临时/验证残留文件

2) 内容完整性
- `page.html` 含 Background / Study Design / Methods / Results / Conclusions / Footer
- 无占位文本（lorem、xxxx、template）

3) 版式质量底线
- 无明显文本溢出/重叠
- 无空白 Result 卡片、无破图
- 响应式布局，适合电脑浏览器查看

4) 双语
- EN/ZH 切换功能正常
- 默认英文显示

5) 交付清单
- 必交付：`page.html`
