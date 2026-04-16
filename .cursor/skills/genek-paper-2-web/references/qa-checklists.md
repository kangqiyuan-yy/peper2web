# Paper2Web QA 门禁清单（网页）

交付前，按以下清单逐项核对。任一硬门禁失败即不允许交付。

## A. 资产门禁（Hard Gate）

- [ ] `paper.pdf` 存在于 `papers/{AuthorYear}/`
- [ ] `figures/fig{N}.png` 连续、无缺号、可打开
- [ ] 无临时残留：`tmp-*`、`page-*`、`*_verify.png`

## B. 内容门禁（Hard Gate）

- [ ] `page.html` 包含：Background / Study Design / Methods / Results / Conclusions / Footer
- [ ] 无占位文本（例如 lorem、xxxx、template）
- [ ] 每个 Result 卡片包含：编号圆标 + 标题 + Figure 图片 + 解读文字

## C. 双语门禁（Hard Gate）

- [ ] EN/ZH 切换按钮存在且功能正常
- [ ] 默认显示英文
- [ ] 中文内容非空

## D. 视觉门禁（Soft Gate，建议全通过）

- [ ] 无明显文字溢出/重叠
- [ ] 图片未被拉伸变形
- [ ] Lightbox 点击放大功能正常
- [ ] 折叠/展开功能正常
- [ ] 打印预览布局合理

## E. 交付清单（Hard Gate）

- [ ] `page.html`
