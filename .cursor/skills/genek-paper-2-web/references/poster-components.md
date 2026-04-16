# HTML 网页可复用组件库

## CSS 变量（配色方案）

基础色板，根据论文或机构风格调整：

```css
:root {
  --primary: #1B3A4B;
  --secondary: #065A82;
  --accent: #00A7E1;
  --warm: #E8927C;
  --gold: #D4A574;
  --bg-light: #F4F7F9;
  --bg-cream: #FAFBFC;
  --text-dark: #1A1A2E;
  --text-muted: #5A6B7D;
}
```

### Result 独立 accent 色

每个 Result 使用不同的 accent 色作为顶部边框和编号圆标背景，帮助读者区分不同发现。按论文主题选色，例如：

```css
:root {
  --r1-color: #1B3A4B;
  --r2-color: #065A82;
  --r3-color: #2E8B57;
  --r4-color: #00A7E1;
  --r5-color: #D4A574;
  --r6-color: #E8927C;
}
.result-1 { border-top-color: var(--r1-color); }
.result-2 { border-top-color: var(--r2-color); }
/* ... */
```

---

## 三列 Grid 布局

```css
.poster { max-width: 1800px; margin: 0 auto; background: var(--bg-cream); }

.content {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 0.3in;
  padding: 0.45in 0.55in;
}
```

### 标准布局（6 个 Result 示例）

第 1 列 = 叙事列，第 2–3 列 = Result + Conclusions。

```css
.background    { grid-column: 1; grid-row: 1; }
.study-design  { grid-column: 1; grid-row: 2; }
.methods       { grid-column: 1; grid-row: 3; }

.result-1      { grid-column: 2; grid-row: 1; }
.result-2      { grid-column: 3; grid-row: 1; }
.result-3      { grid-column: 2; grid-row: 2; }
.result-4      { grid-column: 3; grid-row: 2; }
.result-5      { grid-column: 2; grid-row: 3; }
.result-6      { grid-column: 3; grid-row: 3; }

.conclusions   { grid-column: 2 / 4; grid-row: 4; }
```

Result 数量不同时调整行数，保持从左到右、从上到下排列：

```css
/* 4 个 Result */
.result-1 { grid-column: 2; grid-row: 1; }
.result-2 { grid-column: 3; grid-row: 1; }
.result-3 { grid-column: 2; grid-row: 2; }
.result-4 { grid-column: 3; grid-row: 2; }
.conclusions { grid-column: 2 / 4; grid-row: 3; }

/* 3 个 Result（最后行只占 1 格） */
.result-1 { grid-column: 2; grid-row: 1; }
.result-2 { grid-column: 3; grid-row: 1; }
.result-3 { grid-column: 2; grid-row: 2; }
.conclusions { grid-column: 2 / 4; grid-row: 3; }
```

---

## Section 基础样式

```css
.section {
  background: white;
  border-radius: 0.08in;
  padding: 0.3in;
  position: relative;
  box-shadow: 0 2px 10px rgba(0,0,0,0.04);
  border: 1px solid rgba(0,0,0,0.05);
  border-top: 4px solid var(--accent);
}
.section-title {
  font-family: 'Playfair Display', serif;
  font-size: 0.28in; font-weight: 700;
  color: var(--primary);
  margin-bottom: 0.18in; padding-bottom: 0.08in;
  border-bottom: 3px solid var(--accent);
  display: inline-block;
}
.section p, .section li {
  font-size: 0.17in; line-height: 1.6; color: var(--text-dark);
}
```

---

## Result 编号圆标

每个 Result section 带有绝对定位的编号圆标，引导阅读顺序：

```css
.result-number {
  position: absolute;
  top: -0.18in; left: 0.2in;
  width: 0.4in; height: 0.4in;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  font-family: 'Playfair Display', serif;
  font-size: 0.22in; font-weight: 900;
  color: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
  z-index: 2;
}
```

HTML：

```html
<div class="section result-1" style="border-top-color: var(--r1-color)">
  <div class="result-number" style="background: var(--r1-color)">1</div>
  <h2 class="section-title">Result 标题</h2>
  ...
</div>
```

---

## Conclusions 深色样式

Conclusions 跨第 2–3 列，使用深色渐变背景作为视觉终点：

```css
.conclusions {
  grid-column: 2 / 4;
  background: linear-gradient(135deg, var(--primary) 0%, var(--secondary) 100%);
  color: white;
  border-top: none;
}
.conclusions .section-title {
  color: white;
  border-bottom-color: var(--gold);
}

.concl-item {
  display: flex; align-items: flex-start;
  gap: 0.12in; margin-bottom: 0.12in;
  padding: 0.1in;
  background: rgba(255,255,255,0.06);
  border-radius: 0.05in;
  border-left: 3px solid var(--gold);
}
.concl-num {
  font-family: 'Playfair Display', serif;
  font-size: 0.28in; font-weight: 900;
  color: var(--accent); flex-shrink: 0;
  width: 0.35in; text-align: center;
}
.concl-text {
  font-size: 0.16in; line-height: 1.55;
  color: rgba(255,255,255,0.92);
}
```

HTML：

```html
<div class="section conclusions">
  <h2 class="section-title">Conclusions</h2>
  <div class="concl-item">
    <div class="concl-num">1</div>
    <div class="concl-text">结论内容...</div>
  </div>
  <div class="concl-item">
    <div class="concl-num">2</div>
    <div class="concl-text">结论内容...</div>
  </div>
</div>
```

---

## Stat Box

用于 Study Design 区域展示关键数字：

```css
.stat-row { display: flex; gap: 0.15in; flex-wrap: wrap; }
.stat-box {
  flex: 1; min-width: 1in;
  background: var(--bg-light); border-radius: 0.06in;
  padding: 0.14in; text-align: center;
  border-left: 3px solid var(--accent);
}
.stat-box .number {
  font-family: 'Playfair Display', serif;
  font-size: 0.34in; font-weight: 900;
  color: var(--secondary); display: block;
}
.stat-box .label {
  font-size: 0.12in; color: var(--text-muted);
  text-transform: uppercase; letter-spacing: 1px;
}
```

---

## Finding Card

```css
.finding-card {
  background: var(--bg-light); border-radius: 0.06in;
  padding: 0.15in; margin: 0.1in 0;
  border-left: 4px solid var(--accent);
}
.finding-card.important {
  border-left-color: var(--warm);
  background: #FFF8F5;
}
```

---

## Cladogram（多物种系统发育树）

仅用于涉及 ≥3 物种跨物种比较的论文。纯 CSS 实现，展示拓扑关系和类群分组，不标注分化时间。

```css
.cladogram {
  padding: 0.15in;
  background: var(--bg-light);
  border-radius: 0.06in;
  margin: 0.1in 0;
  font-size: 0.12in;
}
.clade-group {
  border-left: 2px solid var(--secondary);
  margin-left: 0.15in;
  padding-left: 0.15in;
  padding-top: 0.04in;
  padding-bottom: 0.04in;
}
.clade-label {
  font-weight: 700;
  color: var(--secondary);
  font-size: 0.13in;
  margin-bottom: 0.04in;
}
.clade-species {
  display: flex; flex-wrap: wrap; gap: 0.06in;
  margin: 0.04in 0;
}
.species-tag {
  display: inline-flex; align-items: center; gap: 0.04in;
  background: white; border-radius: 0.03in;
  padding: 0.03in 0.08in;
  font-size: 0.11in; font-weight: 600;
  border: 1px solid #D8DEE4;
}
.species-tag .sp-dot {
  width: 0.08in; height: 0.08in;
  border-radius: 50%;
}
.outgroup-tag {
  background: #FFF8F0;
  border-color: var(--warm);
}
```

HTML 示例（12 物种比较基因组学）：

```html
<div class="cladogram">
  <!-- 外群 -->
  <div style="margin-bottom:0.06in">
    <span class="species-tag outgroup-tag">
      <span class="sp-dot" style="background:var(--green-a)"></span> Zebrafish
    </span>
    <span style="font-size:0.1in; color:var(--text-muted); margin-left:0.06in">distant outgroup</span>
  </div>
  <div style="margin-bottom:0.08in">
    <span class="species-tag outgroup-tag">
      <span class="sp-dot" style="background:var(--warm)"></span> Chicken
    </span>
    <span style="font-size:0.1in; color:var(--text-muted); margin-left:0.06in">close outgroup</span>
  </div>
  <!-- 哺乳类 -->
  <div class="clade-group">
    <div class="clade-label">Mammals</div>
    <div class="clade-group">
      <div class="clade-label">Euarchontoglires</div>
      <div class="clade-species">
        <span class="species-tag"><span class="sp-dot" style="background:var(--secondary)"></span> Human</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--secondary)"></span> Rhesus</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--secondary)"></span> Mouse</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--secondary)"></span> Rat</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--secondary)"></span> Rabbit</span>
      </div>
    </div>
    <div class="clade-group">
      <div class="clade-label">Laurasiatheria</div>
      <div class="clade-species">
        <span class="species-tag"><span class="sp-dot" style="background:var(--accent)"></span> Cow</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--accent)"></span> Sheep</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--accent)"></span> Pig</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--accent)"></span> Cat</span>
        <span class="species-tag"><span class="sp-dot" style="background:var(--accent)"></span> Dog</span>
      </div>
    </div>
  </div>
</div>
```

此组件的结构可适配各类物种分组：植物（单子叶/双子叶）、微生物（革兰氏阳性/阴性）等，只需调整 clade-label 和颜色。

---

## 分阶段流程图（Pipeline）

采用 3 阶段分层设计，避免反复的分支箭头。每个阶段有独立视觉风格：

- **Stage 1（实验）**：深色渐变背景，用 `→` 串联实验步骤
- **Stage 2（数据处理）**：浅灰背景，tool-tag 标签展示软件工具
- **Stage 3（多维分析）**：白底蓝边分组框，内含 2×N 网格的分析卡片

```
┌─────────────────────────────┐
│ Stage 1 — Experiment        │  ← 深色渐变 (stage-exp)
│ 实验步骤 → 用箭头串联       │
└──────────────┬──────────────┘
               ▼
┌─────────────────────────────┐
│ Stage 2 — Data Processing   │  ← 浅灰背景 (stage-proc)
│ 处理步骤 + tool-tag 标签     │
└──────────────┬──────────────┘
               ▼
┌─ Stage 3 — Analysis ────────┐  ← 白底蓝边 (stage-analysis)
│ ┌─────────┐ ┌─────────┐    │
│ │ Card 1  │ │ Card 2  │    │  ← analysis-card 网格
│ ├─────────┤ ├─────────┤    │     每张卡片不同色彩左边框
│ │ Card 3  │ │ Card 4  │    │
│ ├─────────┤ ├─────────┤    │
│ │ Card 5  │ │ Card 6  │    │
│ └─────────┘ └─────────┘    │
└──────────────────────────────┘
```

### CSS

```css
.pipeline {
  display: flex; flex-direction: column;
  gap: 0; margin: 0.1in 0;
}
.pipeline-stage {
  position: relative;
  padding: 0.14in;
  border-radius: 0.06in;
  margin-bottom: 0;
}
.pipeline-stage.stage-exp {
  background: linear-gradient(135deg, var(--secondary), var(--primary));
  color: white;
}
.pipeline-stage.stage-proc {
  background: var(--bg-light);
  border: 1.5px solid #D0D7DE;
}
.pipeline-stage.stage-analysis {
  background: white;
  border: 2px solid var(--secondary);
}
.stage-label {
  font-size: 0.1in; font-weight: 700;
  text-transform: uppercase; letter-spacing: 1.5px;
  margin-bottom: 0.08in; opacity: 0.7;
}
.stage-exp .stage-label { color: rgba(255,255,255,0.7); }
.stage-proc .stage-label { color: var(--text-muted); }
.stage-analysis .stage-label { color: var(--secondary); }
.stage-title {
  font-family: 'Playfair Display', serif;
  font-size: 0.2in; font-weight: 700;
  margin-bottom: 0.04in;
}
.stage-exp .stage-title { color: white; }
.stage-proc .stage-title { color: var(--primary); }
.stage-detail {
  font-size: 0.12in; opacity: 0.8; line-height: 1.5;
}
.stage-exp .stage-detail { color: rgba(255,255,255,0.8); }
.stage-proc .stage-detail { color: var(--text-muted); }
.stage-tools {
  display: inline-flex; flex-wrap: wrap; gap: 0.04in;
  margin-top: 0.05in;
}
.tool-tag {
  display: inline-block;
  font-size: 0.1in; font-weight: 600;
  padding: 0.02in 0.07in;
  border-radius: 3px;
  letter-spacing: 0.3px;
}
.stage-exp .tool-tag {
  background: rgba(255,255,255,0.18); color: white;
}
.stage-proc .tool-tag {
  background: white; color: var(--secondary);
  border: 1px solid #D0D7DE;
}

/* 阶段间连接线 */
.pipeline-connector {
  display: flex; justify-content: center;
  padding: 0.04in 0;
}
.pipeline-connector .conn-line {
  width: 2px; height: 0.14in;
  background: var(--secondary);
  position: relative;
}
.pipeline-connector .conn-line::after {
  content: ''; position: absolute;
  bottom: -4px; left: -3.5px;
  border-left: 4.5px solid transparent;
  border-right: 4.5px solid transparent;
  border-top: 5px solid var(--secondary);
}

/* 分析卡片网格 */
.analysis-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.08in;
}
.analysis-card {
  background: var(--bg-light);
  border-radius: 0.05in;
  padding: 0.1in;
  border-left: 3px solid var(--accent);
}
.analysis-card .ac-name {
  font-size: 0.13in; font-weight: 700;
  color: var(--primary); margin-bottom: 0.03in;
}
.analysis-card .ac-tools {
  font-size: 0.1in; color: var(--text-muted);
  line-height: 1.4;
}
```

### HTML 示例

```html
<div class="pipeline">
  <!-- Stage 1: Experiment -->
  <div class="pipeline-stage stage-exp">
    <div class="stage-label">Stage 1 — Experiment</div>
    <div class="stage-title">Dilution Hi-C + RNA-seq</div>
    <div class="stage-detail">Formaldehyde crosslinking &#x2192; HindIII digestion &#x2192; Biotin ligation &#x2192; Sequencing</div>
  </div>

  <div class="pipeline-connector"><div class="conn-line"></div></div>

  <!-- Stage 2: Data Processing -->
  <div class="pipeline-stage stage-proc">
    <div class="stage-label">Stage 2 — Data Processing</div>
    <div class="stage-title">Mapping &#x2192; Filtering &#x2192; Normalization</div>
    <div class="stage-tools">
      <span class="tool-tag">Bowtie2</span>
      <span class="tool-tag">Hiclib</span>
      <span class="tool-tag">ICE</span>
    </div>
  </div>

  <div class="pipeline-connector"><div class="conn-line"></div></div>

  <!-- Stage 3: Multi-dimensional Analysis -->
  <div class="pipeline-stage stage-analysis">
    <div class="stage-label">Stage 3 — Multi-dimensional Analysis</div>
    <div class="analysis-grid">
      <div class="analysis-card">
        <div class="ac-name">A/B Compartments</div>
        <div class="ac-tools">PCA, AB index, Phylo-HMGP</div>
      </div>
      <div class="analysis-card">
        <div class="ac-name">TAD Calling</div>
        <div class="ac-tools">DomainCaller (DI + HMM)</div>
      </div>
      <!-- 按需添加更多 analysis-card -->
    </div>
  </div>
</div>
```

分析卡片数量按论文实际分析维度调整（常见 4–6 张），每张卡片可通过 `border-left-color` 赋予独立色彩以区分不同分析类型。

---

## 折叠图表

网页使用 **原图**（`figures/fig{N}.png`）。关键 Figure 用 `<details open>` 默认展开，次要 Figure 折叠。

```css
details.fig-collapse {
  margin-top: 0.12in;
  border: 1px solid #D8DEE4;
  border-radius: 0.06in;
  overflow: hidden;
}
details.fig-collapse summary {
  background: var(--bg-light);
  padding: 0.1in 0.14in;
  font-weight: 700; font-size: 0.14in;
  color: var(--secondary);
  cursor: pointer;
  display: flex; align-items: center; gap: 0.08in;
  list-style: none;
}
details.fig-collapse summary::-webkit-details-marker { display: none; }
details.fig-collapse summary::before {
  content: '\25B8'; font-size: 0.16in;
  color: var(--accent);
  transition: transform 0.2s;
  display: inline-block;
}
details.fig-collapse[open] summary::before {
  transform: rotate(90deg);
}

.fig-image-container {
  text-align: center; margin: 0.08in 0;
  border: 1px solid #E0E4E8; border-radius: 0.04in;
  overflow: hidden;
}
.fig-image-container img {
  max-width: 100%; height: auto;
  display: block; margin: 0 auto;
  cursor: zoom-in;
  transition: opacity 0.2s;
}
.fig-image-container img:hover { opacity: 0.85; }
.fig-caption {
  font-size: 0.12in; color: var(--text-muted);
  padding: 0.06in; background: var(--bg-light);
  text-align: left; line-height: 1.5;
}
```

```html
<!-- 关键 Figure：默认展开 -->
<details class="fig-collapse" open>
  <summary>Figure 1 &#x2014; 标题</summary>
  <div class="fig-content">
    <div class="fig-image-container">
      <img src="figures/fig1.png" alt="Figure 1">
      <div class="fig-caption">图注</div>
    </div>
  </div>
</details>

<!-- 次要 Figure：折叠 -->
<details class="fig-collapse">
  <summary>Figure 2 &#x2014; 标题</summary>
  ...
</details>
```

---

## Lightbox 放大查看

```html
<div class="lightbox-overlay" id="lightbox">
  <img id="lightbox-img" src="" alt="">
</div>
```

```css
.lightbox-overlay {
  display: none; position: fixed;
  top: 0; left: 0; width: 100%; height: 100%;
  background: rgba(0,0,0,0.88);
  z-index: 9999;
  justify-content: center; align-items: center;
  cursor: zoom-out;
}
.lightbox-overlay.active { display: flex; }
.lightbox-overlay img {
  max-width: 92%; max-height: 92%;
  object-fit: contain; border-radius: 4px;
  box-shadow: 0 8px 40px rgba(0,0,0,0.5);
}
```

```javascript
(function(){
  var overlay = document.getElementById('lightbox');
  var lbImg = document.getElementById('lightbox-img');
  document.querySelectorAll('.fig-image-container img').forEach(function(img){
    img.addEventListener('click', function(){
      lbImg.src = this.src;
      lbImg.alt = this.alt;
      overlay.classList.add('active');
    });
  });
  overlay.addEventListener('click', function(){ overlay.classList.remove('active'); });
  document.addEventListener('keydown', function(e){
    if(e.key === 'Escape') overlay.classList.remove('active');
  });
})();
```

---

## Footer

```css
.footer {
  background: var(--primary);
  padding: 0.25in 0.55in;
  display: grid;
  grid-template-columns: 1.5fr 1fr 1fr;
  gap: 0.4in;
  color: rgba(255,255,255,0.8);
  font-size: 0.13in;
  line-height: 1.6;
}
.footer strong {
  color: var(--gold);
  font-size: 0.14in;
  letter-spacing: 1px;
  text-transform: uppercase;
}
.footer a { color: var(--accent); text-decoration: none; }
```

```html
<div class="footer">
  <div>
    <strong data-lang="en">Acknowledgments</strong>
    <p data-lang="en">资助信息...</p>
  </div>
  <div>
    <strong data-lang="en">Data Availability</strong>
    <p data-lang="en">数据/代码链接...</p>
  </div>
  <div>
    <strong data-lang="en">Contact</strong>
    <p data-lang="en">通讯作者 email...</p>
  </div>
</div>
```

---

## 中英文双语切换

### 按钮样式

```css
.lang-switch {
  position: fixed;
  top: 0.2in; right: 0.3in;
  z-index: 100;
  display: flex; gap: 0;
  background: rgba(255,255,255,0.97);
  border-radius: 6px;
  box-shadow: 0 2px 16px rgba(0,0,0,0.18);
  overflow: hidden;
  border: 1px solid #D0D7DE;
}
.lang-btn {
  border: none; background: transparent;
  padding: 8px 18px;
  font-size: 15px; font-weight: 700;
  color: var(--text-muted);
  cursor: pointer;
  transition: all 0.2s;
}
.lang-btn:hover { color: var(--secondary); background: var(--bg-light); }
.lang-btn.active {
  background: var(--secondary);
  color: white;
}
```

### HTML 标记

```html
<div class="lang-switch">
  <button class="lang-btn active" data-target="en">EN</button>
  <button class="lang-btn" data-target="zh">中文</button>
</div>

<!-- 需要翻译的元素加 data-lang -->
<h2 data-lang="en">Introduction</h2>
<h2 data-lang="zh">研究背景</h2>
```

无需双语标记的元素：数字/统计值、图片、DOI、作者姓名。

### JS 切换逻辑

```javascript
(function(){
  var btns = document.querySelectorAll('.lang-btn');
  btns.forEach(function(btn){
    btn.addEventListener('click', function(){
      var lang = this.dataset.target;
      document.querySelectorAll('[data-lang]').forEach(function(el){
        el.style.display = el.dataset.lang === lang ? '' : 'none';
      });
      btns.forEach(function(b){ b.classList.remove('active'); });
      this.classList.add('active');
    });
  });
  document.querySelectorAll('[data-lang="zh"]').forEach(function(el){
    el.style.display = 'none';
  });
})();
```

---

## 打印与预览

```css
@media print {
  body { background: white; padding: 0; }
  .poster { box-shadow: none; max-width: 100%; }
  details.fig-collapse[open] { break-inside: avoid; }
  .lang-switch { display: none; }
  -webkit-print-color-adjust: exact;
  print-color-adjust: exact;
}
```

```bash
python3 -m http.server 8765
# 访问 http://localhost:8765/papers/{Author}{Year}/page.html
```

---

## 特殊字符速查

| 字符 | HTML 实体 | CSS content | 用途 |
|------|----------|-------------|------|
| ▸ | `&#x25B8;` | `\25B8` | 折叠箭头 |
| → | `&#x2192;` | `\2192` | 右箭头 |
| — | `&#x2014;` | `\2014` | 破折号 |
| × | `&times;` | - | 乘号 |
| ≥ | `&ge;` | - | 大于等于 |
| ≤ | `&le;` | - | 小于等于 |
