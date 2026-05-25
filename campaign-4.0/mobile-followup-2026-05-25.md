# Campaign 3.0 mobile follow-up — 跟进 round 46.x 内容更新

**日期**：2026-05-25
**触发**：Landing 页面又更新了一轮（round 46.x 共 13 个小版本），需要确认这些改动是否影响 mobile 适配。

---

## 一、变更扫描总览

文件从 v3 末的 **8200 行 / 365 KB** → 现在 **8517 行 / 372 KB**（+285 行，+7 KB）。

新增的 13 个 round 46.x 子版本（46.6/46.7/46.9/46.10/46.11/46.15/46.16/46.17/46.18/46.19/46.21/46.22/46.24）**全部围绕 desktop hero 飘浮 tweet 卡片的 WebGL 渲染细节**：

| Round | 改动 |
|---|---|
| 46.6 | 色散 (chromatic aberration) 强度 0.013→0.020 |
| 46.7 | 卡片 left/right 边描边修复 |
| 46.9 | Texture color space SRGBColorSpace → NoColorSpace |
| 46.10 | premultiplied alpha 适配 |
| 46.11 | TIER_BLUR 全归 0（避免 alpha + RGB 混色产生灰边） |
| 46.15-46.17 | Border color / opacity / shadow offset |
| 46.18-46.19 | 整数像素 snap（消除 sub-pixel 抖动）|
| 46.21-46.24 | Border + radius 微调（最终 1px + r=5）|

**结论**：这 13 个 round **mobile 不受影响** —— WebGL 在 mobile 早 return (`if (window.matchMedia('(max-width: 760px)').matches) return;`)，移动端走 DOM 飘卡。

---

## 二、内容层面改动（mobile-relevant）

虽然 WebGL 部分跟 mobile 无关，但**内容层面**有 4 个改动需要 mobile 跟进：

### 1. Hero H1 新文案

```diff
- "On X, you follow @___ Do you know their track record?" (带 @yardeni roller)
+ "Which FinTwit account / actually makes money?" (2 行短句)
```

**Mobile 影响**：✅ 无问题 — 当前 `clamp(36px, 9.5vw, 44px)` ≈ 37 px @393w，2 行短文 OK。

### 2. Stage H1 新文案（4 行 + sub）

```html
<h1 class="stage-title">
  <span class="stage-title-line">We built <span class="alva">live playbooks</span></span>
  <span class="stage-title-line">for FinTwit's</span>
  <span class="stage-title-line">hottest accounts.</span>
  <span class="stage-title-line stage-title-sub">Updated every hour.</span>
</h1>
```

**Mobile 问题** 🐛：`.stage-title-sub` 在 CSS 中**完全没定义** —— 4 行 stage H1 全部同字号，sub-line 视觉上跟主 H1 等权重，失去 "tail" 角色。

**修法**：mobile @media 内新加：

```css
.stage-marquee .stage-title-sub {
  font-size: 0.7em;          /* 33 px → ~23 px */
  font-weight: 500;
  opacity: 0.7;
  display: block;
  margin-top: 0.4em;
}
```

只动 mobile，desktop 保留（desktop 4 行同字号视觉上影响较小）。

### 3. Stage ghost CTA 改名

```diff
- "Search" (1 词，~50 wide)
+ "Can't find an Account?" (5 词，~230 wide)
```

**Mobile 问题** 🔴：双 button 横排在 393 viewport **挤不下**：

```
"Check your following" (~21 char + icon)  ≈ 220 wide
"Can't find an Account?" (~23 char + icon) ≈ 230 wide
gap 10 + section padding ≈ 30
total ≈ 490 wide > 393 viewport
```

Desktop `.stage-actions` 已有 `flex-wrap: wrap`，太宽会 wrap 到第二行，但**两个 button 宽度不一**视觉混乱。

**修法**：mobile @media 内强制 vertical stack + full-width：

```css
.stage-marquee .stage-actions {
  flex-direction: column;
  align-items: stretch;
  width: 100%;
  max-width: 360px;
}
.stage-actions .btn-primary,
.stage-actions .btn-ghost {
  width: 100%;
  justify-content: center;
  padding: 14px 22px;
  font-size: 14px;
  min-height: 44px;
}
```

每个 button 独立一行 full-width，跟 sticky bottom CTA bar + modal-card-detail share methods 同款 idiom。

### 4. Playbook tabs 5 个新名（之前 6 个）

```diff
- Bull vs bear / Critique a pick / Build a mix / Replay past calls / Screen creators / Watchlist alerts (6)
+ Backtest win rates / Simulate ROI / Signals & alerts / Bull or Bear / Stress-test (5)
```

**Mobile 影响**：
- ✅ 数量 6→5，更少装得下
- ⚠️ 但 mobile `.play-tab` 之前是 `padding: 9px 12px font-size: 12.5px` ≈ 30 h tap target — **下于 36 触控 floor**（漏修）
- 新增更长 label "Backtest win rates"、"Signals & alerts" 单 tab 宽度增加，但 tabs 已经 `overflow-x: auto`，OK

**修法**（顺便修旧问题）：

```css
.play-tab { padding: 11px 14px; font-size: 13px; min-height: 38px; }
.play-tab i { font-size: 12px; }
.play-tab.active::after { left: 14px; right: 14px; }
```

触控 30 → 38（secondary touch floor 36+ 达标）。

---

## 三、Playbook cards 结构改了（HTML 检查）

之前 5 卡 / 6 卡都含 `<h3>` 标题，现在 cards **没有 h3**，只有 `[cover] + <p> + <a class="play-cta">`：

```html
<article class="play-card" id="play-bull-bear">
  <div class="play-cover" data-variant="teal"><i class="ph ph-chart-bar"></i></div>
  <p>Stop guessing who's good. See exactly which calls made money, and which didn't.</p>
  <a class="play-cta">Open playbook ↗</a>
</article>
```

注释里写："h3 removed in round 9; mobile p now sits right under the cover."

**Mobile 已经适配**：`mobile @media .play-card p { font-size: 16px; line-height: 1.5; margin-top: 20px; }` —— 已经写了 `margin-top: 20px` 给 cover→p 留呼吸。完全适配 ✓。

**JS dots 同步**：`setupPlayDots` 是按 `track.querySelectorAll('.play-card').length` 自动生成 dots —— 5 卡 → 5 dots，完全自适应 ✓。

---

## 四、所有之前 mobile 修复完整性验证

12 项 v3 末的核心修复，逐一验证仍生效：

| 检查项 | 状态 |
|---|---|
| sticky-cta-bar HTML + CSS + JS | ✓ 4 处 |
| nav-menu-toggle (hamburger) | ✓ 12 处 |
| nav-drawer (mobile drawer) | ✓ 30 处 |
| `.alva-badge { display: none }` mobile | ✓ |
| Hero `min-height: 100svh` | ✓ |
| `text-wrap: balance` | ✓ |
| `scroll-snap-align: start` (playbook peek) | ✓ |
| three.js mobile-skip | ✓ |
| Delight font 4-weight preload | ✓ 4 处 |
| `cd-close` (KOL card-detail modal close) | ✓ 8 处 |
| `min-height: 44px` (touch targets) | ✓ 3 处 |
| modal close `.dialog{position:relative} + .close{absolute}` | ✓ |

**没有 regression** —— 13 轮 round 46.x 没有破坏之前的 mobile 修复。

---

## 五、本轮新增的 mobile 修复

2 处 edit：

### Fix 1: `.stage-marquee .stage-actions` vertical stack + `.stage-title-sub` 字号

行号 ~5099-5135（mobile @media 内 stage 部分）

```css
/* stage 双 button vertical stack full-width，"Can't find an Account?" 不再挤 */
.stage-marquee .stage-actions {
  flex-direction: column;
  align-items: stretch;
  width: 100%;
  max-width: 360px;
}
.stage-actions .btn-primary,
.stage-actions .btn-ghost {
  width: 100%;
  justify-content: center;
  padding: 14px 22px;
  font-size: 14px;
  min-height: 44px;
}

/* stage-title-sub 字号小一档，视觉重建 sub-line 层级 */
.stage-marquee .stage-title-sub {
  font-size: 0.7em;       /* 33 px → ~23 px */
  font-weight: 500;
  opacity: 0.7;
  display: block;
  margin-top: 0.4em;
}
```

### Fix 2: `.play-tab` mobile 触控目标 30→38

行号 ~5277

```css
.play-tab { padding: 11px 14px; font-size: 13px; min-height: 38px; }
.play-tab i { font-size: 12px; }
.play-tab.active::after { left: 14px; right: 14px; }
```

---

## 六、验收清单

请在 iPhone 14 Pro 393×852 mobile mode 验：

### 新内容验收
- [ ] Hero H1 "Which FinTwit account / actually makes money?" 2 行 37 px 显示正常
- [ ] Stage H1 4 行 stack：3 行主 (33 px) + "Updated every hour." (23 px, 半透明) 视觉层级清晰
- [ ] Stage 双 button **vertical stack full-width**：「Check your following」+「Can't find an Account?」上下排，宽度一致
- [ ] Playbook tabs 5 个新名（Backtest win rates / Simulate ROI / Signals & alerts / Bull or Bear / Stress-test），横向 scroll 顺畅，每个 tab ≥ 38 高
- [ ] Playbook 5 张 card，cover + 描述 + Open playbook CTA（没 h3 也 OK）
- [ ] Playbook dots 5 个，跟随 swipe 切 active

### 之前修复无 regression
- [ ] Hamburger drawer 仍然能打开（含 3 tab + For FinTwit creators）
- [ ] Sticky bottom CTA bar 滚过 stage 后浮现
- [ ] modal-card-detail (点 KOL 卡) 右上有 ✕ 按钮，内容可滚
- [ ] 所有 modal close ✕ 在右上角
- [ ] iOS focus textarea/input 不 zoom
- [ ] alva-badge 在 mobile 不可见

### ≤380 SE-class
- [ ] H1 32 px，sub 字号合适
- [ ] play-card 295 wide + 双侧 peek ~40
- [ ] play-tab 38 高（不再退缩）

---

## 七、Round 46.x 系列总结（不需要 mobile 跟进的部分）

只为留档说明这 13 个 round mobile 为什么不用动。所有 round 都在 desktop hero `.float-tweet` 卡片的 WebGL 渲染管线内：

| Round | 改的 | 为什么 mobile 不受影响 |
|---|---|---|
| 46.6 | shader uCA 0.013→0.020 | mobile skip WebGL |
| 46.7 | 卡片描边方位修复 | 同上 |
| 46.9 | texture color space | 同上 |
| 46.10 | premultiplied alpha | 同上 |
| 46.11 | TIER_BLUR 0 | 同上 |
| 46.15-46.17 | border/shadow tuning | 同上 |
| 46.18-46.19 | pixel snap | 同上 |
| 46.21-46.24 | border + radius 1px/5r | 同上 |

Mobile 飘卡用 DOM (`.float-tweet { width: 184px !important; ... }`) + 4 张 cap（`:nth-child(n+5) { display: none }`），WebGL 这些 round 都跟它没关系。

---

## 八、产品规律沉淀（新发现）

把这次发现的 1 条加进 Notion 资料库的 "Alva 产品 mobile 适配 10 大规律"（实际应该升级为 11 条）：

**规律 11（新）：双 button CTA 在 mobile 一律 vertical stack + full-width**

Desktop 上 stage / hero / modal 的双 button（primary + ghost）横排展示，但 mobile 上：
- 单 button 加上 padding (14×22) + 14 font + iOS 系统字距，每 button 至少 200 wide
- 横排 2 个 + gap 至少 410 wide > 393 viewport
- 即使 flex-wrap 解决溢出，宽度不一视觉混乱

**统一规则**：
```css
.cta-pair, .stage-actions, .modal-actions {
  flex-direction: column;
  align-items: stretch;
  width: 100%;
  max-width: 360px;
  gap: 10px;
}
.cta-pair > .btn-primary,
.cta-pair > .btn-ghost {
  width: 100%;
  justify-content: center;
  min-height: 44px;
}
```

同款 idiom 已用在：
- Sticky bottom CTA bar
- modal-card-detail share methods (5 个 pill 全宽)
- 这次新加的 stage-actions

---

下一步：deploy 后在真机 iPhone 14 Pro 跑一遍验收清单，重点确认 stage 双 button 是否 vertical stack + stage-title-sub 是否真的小一档。如有 regression 告诉我，我修。
