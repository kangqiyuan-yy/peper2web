# Paper2Slide QA 门禁清单

交付前，按以下清单逐项核对。任一硬门禁失败即不允许交付。

## A. 资产门禁（Hard Gate）

- [ ] `paper.pdf` 存在于 `papers/{AuthorYear}/`
- [ ] `figures/fig{N}.png` 连续、无缺号、可打开
- [ ] `figures/panels/*.png` 存在且文件非空
- [ ] 无临时残留：`tmp-*`、`page-*`、`*_verify.png`

## B. 内容门禁（Hard Gate）

- [ ] `presentation.pptx` 包含：Title / Background / Study Design / Methods / Result / Conclusion
- [ ] 每个 Result 幻灯片包含：标题 + 图 + 2-3 条要点
- [ ] 无占位文本（例如 lorem、xxxx、template）

## C. 一致性门禁（Hard Gate）

- [ ] PPT Result 页数 = panel 文件数
- [ ] Result 页编号连续且与 panel 顺序一致
- [ ] Figure/Panel 溯源信息可追踪（至少在标题或备注中体现）

## D. 双语门禁（Conditional Gate）

默认模式 `en_notes_zh`：
- [ ] 所有页面英文正文可读
- [ ] 所有页面中文 notes 非空

双套模式 `dual_slides`：
- [ ] EN 套与 ZH 套页数一致
- [ ] 同页内容语义一致，无遗漏

## E. 视觉门禁（Soft Gate，建议全通过）

- [ ] 无明显文字溢出/重叠
- [ ] 图片未被拉伸变形
- [ ] 字体大小在可读范围（正文建议 >= 14pt）
- [ ] 关键信息未被页脚或图形遮挡

## F. 交付清单（Hard Gate）

- [ ] `presentation.pptx`
- [ ] macOS 环境下额外交付 `presentation_keynote_compatible.pptx`
