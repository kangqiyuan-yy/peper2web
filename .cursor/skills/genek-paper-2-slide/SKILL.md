---
name: genek-paper-2-slide
description: 从论文生成中英文两版 PPTX 演示文稿（presentation_en.pptx + presentation_cn.pptx）。每张 panel 子图对应一页幻灯片，含详细解读。当用户提及"论文转PPT"、"论文转幻灯片"、"学术演示"、"presentation"时使用此技能。
---

# Paper2Slide — 论文 → PPTX 演示文稿（中英文双版本）

从单篇论文生成**中英文两版**结构化 PPTX 演示文稿。PPT 使用**拆分后的 panel 图**（`figures/panels/`），每张子图对应一页幻灯片，逐张讲解。两版 PPT 页数、页序、图片完全一致，仅文字语言不同。

## 适用边界（职责分层）

- `genek-paper-2-slide` 定义 **What**（内容结构、映射规则、命名规范、验收门禁）
- `pptx` skill 负责 **How**（具体排版渲染与 PPT 文件生成）
- `genek-paper-2-slide` 不重复实现 `pptx` 内部逻辑，只通过契约调用
- 不涉及 HTML 海报生成（海报由 `genek-paper-2-web` 负责）

## 目录结构

每篇论文以 **第一作者姓氏 + 年份** 命名独立子目录：

```
papers/
└── {Author}{Year}/
    ├── paper.pdf
    ├── figures/
    │   ├── fig1.png ... figN.png            (原图，genek-extract-figures)
    │   └── panels/
    │       ├── fig1_A.png                   (拆分后的 panel)
    │       ├── fig1_BC.png
    │       └── ...
    ├── create_pptx.js
    ├── presentation_en.pptx                 (英文版，本 skill 产物)
    └── presentation_cn.pptx                 (中文版，本 skill 产物)
```

## 内容源

- 从 `paper.pdf` 直接抽取结构化内容，不依赖 `poster.html` 或其他中间产物

## 执行流程

| 步骤 | 目标 | 输入 | 输出 | 成功判据 |
|------|------|------|------|----------|
| **S0** | 建立论文目录 | `paper.pdf` | `papers/{Author}{Year}/` | 目录结构正确 |
| **S1** | 抽取结构化内容 | `paper.pdf` | 标题/作者/Background/Design/Methods/Results/Conclusions | 字段齐全，无空段 |
| **S2** | 提取原图 | `paper.pdf` | `figures/fig1..figN.png` | 图片数与主图数一致 |
| **S3** | 拆分 panel | `figures/figN.png` | `figures/panels/fig{N}_{group}.png` | **必须**执行 `genek-split-figures`，产出真实子图；禁止整图占位（如 `_whole`） |
| **S4** | 生成 PPT（双版本） | 结构化内容 + panel 图 | `presentation_en.pptx` + `presentation_cn.pptx` | 两版均可打开，页序与映射一致 |
| **S5** | QA 门禁 | 两版 pptx | 通过/不通过 | 两版均满足本文件 QA Gate |

跳过策略：
- S2：若 `figures/fig{N}.png` 已存在且连续完整，跳过提取
- S3：**禁止跳过**。必须按 `genek-split-figures` 技能**严格执行**：对每张 `figures/fig{N}.png` 进行 LLM 看图 → 分组与粗坐标 → 像素精修 → 按 panel 裁剪，输出 `fig{N}_{group}.png`（group 为论文图中真实 panel 标签，如 A、BC、DEFG）。**禁止**将整图复制为 `fig{N}_whole.png`、`fig{N}_full.png` 等占位文件以代替拆分；凡仅以整图充当单一「panel」的，S3 视为未执行，不得进入 S4。

失败策略：
- S2 失败：回退到 `genek-extract-figures` 的备选路径
- S3 失败：允许人工校正 panel 分组后重跑
- S4 失败：返回到 PPT 契约检查，定位为"输入缺失"或"渲染失败"。若其中一版生成成功、另一版失败，仍判定为 S4 失败，需两版都通过

## 图片使用规则

| 格式 | 使用图片 | 路径 | Result 对应 |
|------|---------|------|------------|
| presentation.pptx | **panel 图** | `figures/panels/fig{N}_{panels}.png` | **1 张子图 = 1 页 PPT** |

每页 PPT 对应一张拆分后的 panel 子图，逐张讲解。一张 Figure 可能拆分为多个 panel，因此 PPT 页数通常多于论文主图数。

### Panel 命名规范

- 单 panel：`fig1_A.png`
- 多 panel 合并为一组：`fig1_BC.png`、`fig1_DEFG.png`

## PPT 调用契约（核心）

### 1) 固定输入契约

- 论文目录：`papers/{Author}{Year}/`
- panel 输入：`papers/{Author}{Year}/figures/panels/*.png`
- 命名格式：`fig{N}_{GROUP}.png`
- 结构化内容必须包含：
  - `title`, `authors`, `affiliation`
  - `background`, `study_design`, `methods`, `results[]`, `conclusions[]`
  - `results[]` 中每项需含 `result_id`, `headline`, `key_points`, `source_figure`

### 2) 固定输出契约

- 产物（必须同时交付两版）：
  - `papers/{Author}{Year}/presentation_en.pptx`（英文版）
  - `papers/{Author}{Year}/presentation_cn.pptx`（中文版）
- 平台兼容产物（macOS 默认启用，两版各一份）：
  - `papers/{Author}{Year}/presentation_en_keynote_compatible.pptx`
  - `papers/{Author}{Year}/presentation_cn_keynote_compatible.pptx`
- 两版共享：相同页数、相同页序、相同图片、相同配色/版式
- 页序：
  1. Title
  2. Background
  3. Study Design
  4. Methods
  5. Result Panels（1 panel = 1 页）
  6. Conclusion
  7. End / Acknowledgment（可与 Conclusion 合并）

### 3) 各页内容规格

- **Title**：论文标题、作者列表、单位、期刊/年份
- **Background**（1–2 页）：领域背景 2–3 句；高亮知识缺口/未解决问题（加粗框）；本研究切入点
- **Study Design**（1 页）：Stat box 关键数字 + 汇总表；多物种时含 cladogram
- **Methods**（1–2 页）：按 3 阶段组织（实验 → 数据处理 → 分析）；每步标注软件工具名；推荐流程图/SmartArt
- **Conclusions**（1 页）：编号列表，每条 1–2 句，与 Result 发现对应；可选 Future Directions

### 4) panel → slide 映射规则

- 规则：**1 张 panel 图 = 1 张 Result 幻灯片**
- 排序：先按 `figN` 数字升序，再按 `GROUP` 字典序（A < B < BC < DEFG）
- 推荐标题：`Result {k}: {headline}`，副标题显示来源 `Figure {N} / Panel {GROUP}`
- 详细解读要求：
  - **必选 3 条**：What is shown（该子图展示了什么数据/现象）、Evidence（关键数值、统计关系或对照结果）、Take-home message（一句话结论）
  - **可选**（按论文类型选取）：Biological interpretation、Cross-species implication、Clinical significance、Computational novelty 等
- 若 `results[]` 条目数与 panel 数不一致：
  1. 若 panel 更多：允许复用同一 `headline`，但必须在 notes 标注"same finding, different panel"
  2. 若 panel 更少：阻断交付，返回"不通过（输入不完整）"

### 5) 双版本语言契约

输出两个独立 PPT 文件，而非在同一文件内混合双语：

| 版本 | 文件名 | 幻灯片正文 | Speaker Notes |
|------|--------|-----------|---------------|
| 英文版 | `presentation_en.pptx` | English | English（可选附中文摘要） |
| 中文版 | `presentation_cn.pptx` | 中文 | 中文 |

- **英文版**：所有标题、要点、表格、结论均为英文。专业术语保留原文。Speaker notes 用英文书写。
- **中文版**：所有标题、要点、表格、结论翻译为中文。学术术语可保留英文原文并括注中文（如 "TADs (拓扑关联结构域)"）。Speaker notes 用中文书写，适合教学演讲。
- **图片标注不翻译**：panel 图本身为论文原图，内部标注保持英文原样，两版共用相同图片文件。
- **实现建议**：在 `create_pptx.js` 中准备中英文两套文本数据结构，通过循环或函数参数化生成两版 PPT，避免手动维护两份独立代码。

### 6) 错误与降级契约

- 缺失 panel 文件：禁止"空白页占位"，直接 fail-fast
- panel 文件损坏：跳过该文件并计入错误，最终判定不通过
- 渲染后页数不一致：按 Gate 判定不通过，不可直接交付
- **两版一致性**：英文版和中文版的页数必须相等，panel 图顺序必须相同。若一版渲染失败，两版均判定不通过
- macOS 导出策略（两版各执行一次）：
  - 先生成标准 `presentation_en.pptx` / `presentation_cn.pptx`
  - 默认执行一次兼容中转导出，生成 `*_keynote_compatible.pptx`

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

当论文涉及 **≥3 个物种的跨物种比较** 时，Study Design 幻灯片中必须包含：

1. **简易系统发育树（cladogram）**：用图形/SmartArt 展示物种间拓扑关系
2. **类群分组着色**：用颜色区分主要分类群（如哺乳类/鸟类/鱼类）
3. **外群标识**：明确标出外群物种

不涉及跨物种比较的论文不需要 cladogram。

## 依赖 skill

- **图片提取**：`genek-extract-figures` → 原图到 `figures/`
- **Panel 拆分**：`genek-split-figures` → 子图到 `figures/panels/`
- **PPT 生成**：`pptx` skill（pptxgenjs）→ 生成 .pptx

## 组件与参考

- PPT 调用手册（参数、重试、问题排查）→ [references/pptx-playbook.md](references/pptx-playbook.md)
- QA 门禁清单 → [references/qa-checklists.md](references/qa-checklists.md)

## QA 门禁（交付前必须通过）

1) 资产完整性
- `figures/panels/*.png` 存在且非空
- **Panel 必须来自真实拆分**：文件名须为 `fig{N}_{group}.png`，其中 `group` 为论文图中实际存在的 panel 标签（单字母或字母组合，如 A、BC、DEFG）。**禁止**使用 `_whole`、`_full`、`_all` 等占位组名；若存在此类文件，QA 不通过。
- 不存在临时/验证残留文件

2) 内容完整性（两版均需满足）
- 每版 PPT 含 Title / Background / Study Design / Methods / Result / Conclusion
- 每个 Result 页至少包含：标题、图、详细解读（建议 4-5 条）
- 无占位文本

3) 一致性校验
- 两版 PPT Result 页数 = panel 图数量
- 编号顺序一致，不跳号
- 每页可追溯到对应 Figure/Panel（标题或备注中标记来源）
- **英文版与中文版页数相同、panel 图顺序相同**

4) 双版本语言
- 英文版：所有幻灯片正文为英文
- 中文版：所有幻灯片正文为中文（学术术语可保留英文括注）
- 两版 Speaker Notes 分别使用对应语言

5) 版式质量底线（两版均需满足）
- 无明显文本溢出/重叠
- 图片未被拉伸变形
- 无空白结果页、无破图

6) 交付清单
- 必交付：`presentation_en.pptx` + `presentation_cn.pptx`