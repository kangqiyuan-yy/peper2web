# 出版商图片 URL 模板

> 遇到新出版商时，访问论文网页版，查找 "Full size image" 链接，
> 用浏览器 DevTools 获取图片真实 URL，总结模板后补充到此文档。

## Springer Nature / BMC

适用：Nature, Nature Methods, BMC Biology, Genome Biology, Scientific Reports 等。

**URL 格式**：

```
https://media.springernature.com/full/springer-static/image/art%3A{DOI_ENCODED}/MediaObjects/{JOURNAL_ID}_{YEAR}_{ARTICLE_ID}_Fig{N}_HTML.png
```

**DOI 解析规则**：

```
DOI:  10.1186/s12915-022-01301-7
               ^^^^^  ^^^^  ^^^^
               |      |     |
               |      |     ARTICLE_ID = 1301（去掉 -7 修订号）
               |      YEAR = 2022
               JOURNAL_ID = 12915（/s 后到首个 - 之间）
```

| DOI 示例 | JOURNAL_ID | YEAR | ARTICLE_ID |
|----------|-----------|------|------------|
| `10.1186/s12915-022-01301-7` | 12915 | 2022 | 1301 |
| `10.1038/s41586-023-06004-9` | 41586 | 2023 | 6004 |
| `10.1186/s13059-021-02500-z` | 13059 | 2021 | 2500 |
| `10.1038/s41467-024-45678-3` | 41467 | 2024 | 45678 |

**注意**：部分旧文章或 review 的 URL 可能不符合此模板，需回退到浏览器方式。

---

## PLOS

适用：PLOS ONE, PLOS Biology, PLOS Genetics 等。

**URL 格式**：

```
https://journals.plos.org/plosone/article/figure/image?id=10.1371/journal.pone.{ARTICLE_ID}.g{NNN}
```

其中 `NNN` 为 3 位数 Figure 编号（`001`, `002`, ...）。

示例（DOI `10.1371/journal.pone.0250000`）：

```bash
for i in $(seq 1 5); do
  n=$(printf "%03d" $i)
  curl -sL -o "figures/fig${i}.png" \
    "https://journals.plos.org/plosone/article/figure/image?id=10.1371/journal.pone.0250000.g${n}"
done
```

**注意**：URL 中的期刊名需匹配（`plosone` / `plosbiology` / `plosgenetics` 等）。

---

## MDPI

适用：所有 MDPI 期刊（Cells, IJMS, Genes, Cancers 等）。

**URL 格式**：

```
https://www.mdpi.com/{JOURNAL_PATH}/{VOLUME}/{ISSUE}/{ARTICLE_ID}/{ARTICLE_SLUG}-g{NNN}.png
```

MDPI 文章网页版图片直接嵌入且可右键保存，URL 规律性强。建议直接访问网页版下载。

---

## Elsevier (Cell, Lancet, etc.)

**无固定 URL 模板**，图片通过 CDN 分发（`ars.els-cdn.com`），路径含随机 hash。

提取策略：
1. 访问 `https://doi.org/{DOI}` → 跳转到 ScienceDirect 文章页
2. 点击 Figure → "Download high-res image"
3. 从 DevTools Network 面板复制图片 URL
4. URL 通常形如：`https://ars.els-cdn.com/content/image/1-s2.0-S{ID}-gr{N}_lrg.jpg`

---

## Wiley

**无固定 URL 模板**。

提取策略：
1. 访问 `https://doi.org/{DOI}` → 跳转到 Wiley Online Library
2. 文章页内图片通常可直接右键保存
3. 高分辨率图片需点击 "Open in figure viewer" → "Download PowerPoint"

---

## Oxford University Press (OUP)

适用：Nucleic Acids Research, Bioinformatics 等。

**URL 格式**（部分期刊）：

```
https://oup.silverchair-cdn.com/oup/backfile/Content_public/Journal/{JOURNAL}/{VOLUME}/{ISSUE}/{DOI_SUFFIX}/{FIGURE_FILE}
```

路径不稳定，建议优先从文章网页版下载。

---

## 预印本（bioRxiv / medRxiv）

**URL 格式**：

```
https://www.biorxiv.org/content/biorxiv/early/{YEAR}/{MONTH}/{DAY}/{ID}/F{N}.large.jpg
```

预印本图片质量通常较低（JPEG 压缩），建议：
1. 若已正式发表，从正式出版商下载
2. 若仅预印本可用，从网页版下载 `.large.jpg` 版本

---

## 通用回退策略

当上述模板均不适用时：

1. **浏览器访问** `https://doi.org/{DOI}`
2. **搜索页面** 中的 "Full size image"、"Download figure"、"High resolution" 链接
3. **DevTools (F12)** → Network → 过滤 `image` → 刷新页面 → 找到图片请求
4. **记录 URL 规律** → 补充到本文档对应出版商章节
