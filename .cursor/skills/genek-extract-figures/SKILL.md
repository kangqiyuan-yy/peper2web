---
name: genek-extract-figures
description: 从学术论文 PDF 中提取高质量 Figure 图片。优先从出版商网站下载原始图，备选 pdftoppm 整页转图+裁剪。支持 Springer Nature、Elsevier、Wiley、PLOS 等主流出版商。当用户提及"提取论文图片"、"下载论文 figure"、"论文插图"、"extract figures"时使用此技能。
---

# 学术论文图片提取

## 工作流程

```
1. 从论文 PDF 中识别 DOI 和 Figure 数量
   │
2. 判断出版商 → 选择提取方式
   ├── 已知出版商 URL 模板 → 方法一：直接下载（推荐）
   ├── 未知出版商 → 方法二：浏览器访问论文网页版，手动定位图片 URL
   └── 无法访问网页版 → 方法三：pdftoppm 整页转图 + 裁剪
   │
3. 输出到 papers/{Author}{Year}/figures/ 目录，命名 fig1.png ~ figN.png
4. 删除中间文件，验证文件大小和完整性
```

## 方法优先级

| 优先级 | 方法 | 质量 | 适用场景 |
|--------|------|------|----------|
| **1** | 出版商网站下载 | 最佳，纯图表 | 已发表论文，出版商 URL 可推断 |
| **2** | 浏览器访问网页版 | 最佳 | 未知出版商，但论文有网页版 |
| **3** | `pdftoppm` + 裁剪 | 需后处理 | 无法访问网页版（预印本、内部文档等） |
| **4** | `pdfimages` | 碎片化，不推荐 | 仅作最后手段 |

## 方法一：出版商网站下载（推荐）

### 第一步：识别出版商

从论文 PDF 中提取 DOI（通常在首页页脚或页眉），根据 DOI 前缀判断出版商：

| DOI 前缀 | 出版商 | URL 模板可用 |
|----------|--------|-------------|
| `10.1038/` | Springer Nature | ✅ |
| `10.1186/` | BMC (Springer) | ✅ |
| `10.1007/` | Springer | ✅ |
| `10.1016/` | Elsevier | ⚠️ 需从网页解析 |
| `10.1002/` | Wiley | ⚠️ 需从网页解析 |
| `10.1371/` | PLOS | ✅ |
| `10.3390/` | MDPI | ✅ |
| `10.1093/` | Oxford Univ Press | ⚠️ 需从网页解析 |

### 第二步：构造 URL 并下载

**Springer Nature / BMC 系列**（Nature, BMC Biology, Genome Biology 等）：

```
https://media.springernature.com/full/springer-static/image/art%3A{DOI_ENCODED}/MediaObjects/{JOURNAL_ID}_{YEAR}_{ARTICLE_ID}_Fig{N}_HTML.png
```

DOI 解析规则（`10.1186/s12915-022-01301-7` 为例）：
- `JOURNAL_ID` = `12915`（`/s` 后到首个 `-` 之间的数字）
- `YEAR` = `2022`
- `ARTICLE_ID` = `1301`（去掉末尾修订号 `-7`）

下载脚本（**替换前5个变量为目标论文参数**）：

```bash
#!/bin/bash
AUTHOR_YEAR="Li2022"                # ← 替换：第一作者姓氏+年份
DOI="10.1186/s12915-022-01301-7"    # ← 替换
JOURNAL_ID="12915"                  # ← 替换
YEAR="2022"                         # ← 替换
ARTICLE_ID="1301"                   # ← 替换
FIG_COUNT=6                         # ← 替换为实际 Figure 数量

OUT="papers/${AUTHOR_YEAR}/figures"
DOI_ENCODED=$(echo "$DOI" | sed 's|/|%2F|g; s|:|%3A|g')
mkdir -p "$OUT"
for i in $(seq 1 $FIG_COUNT); do
  url="https://media.springernature.com/full/springer-static/image/art%3A${DOI_ENCODED}/MediaObjects/${JOURNAL_ID}_${YEAR}_${ARTICLE_ID}_Fig${i}_HTML.png"
  curl -sL -o "${OUT}/fig${i}.png" "$url"
  size=$(wc -c < "${OUT}/fig${i}.png" 2>/dev/null || echo 0)
  if [ "$size" -gt 10000 ]; then
    echo "fig${i}.png: OK (${size} bytes)"
  else
    echo "fig${i}.png: FAILED (${size} bytes) - URL 可能不正确"
    rm -f "${OUT}/fig${i}.png"
  fi
done
```

其他出版商 URL 模板详见 → [references/publishers.md](references/publishers.md)

## 方法二：浏览器访问网页版

URL 模板未知或下载失败时：

1. 用浏览器打开 `https://doi.org/{DOI}` 跳转到论文网页版
2. 在文章中找到 Figure，点击 "Full size image" / "Download" / "High resolution"
3. 用开发者工具（F12 → Network）获取图片真实 URL
4. 用 `curl -sL -o figures/figN.png "URL"` 逐张下载
5. **发现新的 URL 规律时**，补充到 [references/publishers.md](references/publishers.md)

## 方法三：PDF 整页转图（备选）

前置条件：`brew install poppler`（macOS）或 `apt install poppler-utils`（Linux）

```bash
pdftoppm -png -r 300 paper.pdf figures/page
```

转换后人工确认 Figure 对应的页码，然后重命名：

```bash
mv figures/page-03.png figures/fig1.png
mv figures/page-05.png figures/fig2.png
# ...按实际页码调整
rm figures/page-*.png
```

### 裁剪（精度差，尽量避免）

```python
from PIL import Image
img = Image.open("figures/fig1.png")
# (left, upper, right, lower) — 需逐张手动确定坐标
cropped = img.crop((35, 210, 2445, 2815))
cropped.save("figures/fig1.png")
```

已知问题：页眉位置因期刊不同而变化；图注与正文分界不清晰；耗时且效果不稳定。

## 方法四：pdfimages（不推荐）

```bash
pdfimages -png paper.pdf figures/img
```

学术 Figure 通常由多个子面板（A, B, C...）+ 向量图形混合存储，`pdfimages` 只能提取嵌入位图，结果碎片化。

## 输出目录规范

每篇论文以 `papers/{Author}{Year}/` 为根目录，PDF 和提取图片均放在该目录下：

```
papers/
├── Li2022/
│   ├── paper.pdf
│   └── figures/
│       ├── fig1.png
│       └── fig2.png
├── Zhang2023/
│   ├── paper.pdf
│   └── figures/
│       ├── fig1.png
│       └── fig2.png
└── Wang2024a/              ← 同姓同年加 a/b 后缀
    ├── paper.pdf
    └── figures/
        └── fig1.png
```

命名规则：
- 目录名取第一作者 **姓氏**（首字母大写）+ 发表年份，如 `Li2022`
- 同姓同年冲突时追加小写字母：`Wang2024a`、`Wang2024b`
- 从论文 PDF 首页提取作者和年份信息

### 文件命名

- 主图：`fig1.png` ~ `fig{N}.png`
- 补充图：`sfig1.png` ~ `sfig{N}.png`
- 删除所有中间文件（`page-*.png`、`tmp-*.png`、`img-*.png`）

## 验证清单

- [ ] 每张图片 > 10KB（空文件说明 URL 错误）
- [ ] 图片数量与论文 Figure 数一致
- [ ] 图片内容是纯图表（无页眉页脚、无正文）
- [ ] 命名连续且与论文编号对应
- [ ] 多论文时各子目录命名正确（作者姓氏+年份）
