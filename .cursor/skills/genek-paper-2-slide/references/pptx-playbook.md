# PPTX 生成操作手册（Paper2All）

本手册用于 `genek-paper-2-slide` 调用 `pptx` skill 时的最小执行标准。  
注意：本手册只定义论文场景规则，不覆盖 `pptx` skill 的通用排版细节。

## 1. 参数约定

建议统一使用以下参数：

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `AUTHOR_YEAR` | 是 | - | 例如 `Li2022` |
| `LANG_MODE` | 否 | `en_notes_zh` | `en_notes_zh` 或 `dual_slides` |
| `THEME` | 否 | `scientific_clean` | 与论文主题一致的配色 |
| `PANEL_SORT` | 否 | `fig_then_group` | `figN` 升序，再 `GROUP` 升序 |
| `NOTES_MODE` | 否 | `required_zh` | `required_zh` / `optional_zh` |

## 2. 生成前检查（Preflight）

必须全部通过再进入 PPT 生成：

1. 目录存在：`papers/{AUTHOR_YEAR}/`
2. panel 目录存在：`papers/{AUTHOR_YEAR}/figures/panels/`
3. panel 命名合法：`fig{N}_{GROUP}.png`
4. 结构化内容齐全：`title/authors/background/study_design/methods/results/conclusions`
5. `results` 至少 1 条，且每条有 `headline` 和 `key_points`

## 3. 生成规则（最小）

固定页序建议：

1. Title
2. Background
3. Study Design
4. Methods
5. Result slides（每张 panel 一页）
6. Conclusion
7. End / Acknowledgment

Result 页要求：

- 每页绑定唯一 panel 文件
- 页标题必须可追溯：`Result {k}` + `Figure {N}/Panel {GROUP}`
- 每页 2-3 条要点（避免整段长文本）

## 4. 失败重试策略

### A. panel 缺失或不合法

- 行为：立即停止（fail-fast）
- 处理：回到 panel 拆分流程，修复后重试

### B. 页数与 panel 数不一致

- 行为：判定失败，不交付
- 处理：检查排序和过滤逻辑，确保 1:1 映射

### C. 中文 notes 缺失（默认模式）

- 行为：判定失败
- 处理：补齐对应页 notes 后重试导出

## 5. 常见问题速查

| 问题 | 根因 | 修复动作 |
|------|------|----------|
| panel 顺序错乱 | 文件排序规则未统一 | 强制 `fig_then_group` 排序 |
| 图片拉伸 | 未保持纵横比 | 统一等比缩放，保留白边 |
| 文本溢出 | 单页信息过多 | 压缩要点到 2-3 条，长句拆分 |
| notes 缺中文 | 仅填英文正文 | 增加中文 notes 校验步骤 |
| 空白结果页 | panel 路径引用失败 | 检查路径和文件存在性 |

## 6. 最小调用示例（伪流程）

1) 从 `papers/{AUTHOR_YEAR}/figures/panels/` 收集并排序 panel 列表  
2) 组装幻灯片数据：`meta + sections + result_panels`  
3) 调用 `pptx` skill 生成 `presentation.pptx`  
4) 执行 QA：页数、映射、可读性、notes 完整性  
5) 通过后与 `poster.html` 一并交付

## 7. macOS 兼容导出（默认）

当运行环境为 macOS 时，默认执行兼容导出，减少 Keynote 打开失败风险。

### 标准流程

1. 先生成标准文件：`presentation.pptx`  
2. 使用 LibreOffice/soffice 进行兼容中转，输出：`presentation_keynote_compatible.pptx`  
3. 若仍不兼容，再导出：`presentation_keynote_compatible.ppt`

### 建议命令（示例）

```bash
soffice --headless --convert-to ppt --outdir "papers/${AUTHOR_YEAR}" "papers/${AUTHOR_YEAR}/presentation.pptx"
soffice --headless --convert-to pptx --outdir "papers/${AUTHOR_YEAR}" "papers/${AUTHOR_YEAR}/presentation_keynote_compatible.ppt"
```

说明：
- 兼容中转后务必保留原始 `presentation.pptx`，避免信息丢失
- 交付时优先提供 `presentation_keynote_compatible.pptx`
