# Campaign 3.0 mobile follow-up — Use Cases section 重构跟进

**日期**：2026-05-25（同日二轮跟进）
**触发**：PC 端把 Use Cases section 从 **playbook carousel** 完全重构成 **agents-pin-section（scrolly-telling pin-scroll 模式）**。

文件从 8517 行 → **9294 行**（+777 行，+32 KB）。

---

## 一、PC 端这一轮改了什么

### 大改动：Use Cases section 完全重构

**之前**（playbook carousel）：
```
[H2 标题]
[6→5 tabs row 横向 scroll]
[carousel: 5 card horizontal scroll with peek]
[dots indicator]
```

**现在**（agents-pin-section / scrolly-telling pin-scroll）：

```
┌────────────────────────────────────────────────────────────┐
│ pin-section: height 200vh, agents-pin position:sticky top  │
│ ┌──────────────────────┬───────────────────────────────┐  │
│ │ Side (480w)          │ Cards stage (right column)    │  │
│ │ H2 "What can you do…"│                                │  │
│ │ sub                  │  [agents-card 1 — cover only]  │  │
│ │                      │                                │  │
│ │ 5 vertical tabs:     │  [agents-card 2 cover]         │  │
│ │ ▼ Backtest win rates │                                │  │
│ │   description        │  [agents-card 3 cover]         │  │
│ │   [Open playbook ↗]  │                                │  │
│ │ ▶ Simulate ROI       │  [agents-card 4 cover]         │  │
│ │ ▶ Signals & alerts   │                                │  │
│ │ ▶ Bull or Bear       │  [agents-card 5 cover]         │  │
│ │ ▶ Stress-test        │                                │  │
│ │                      │  JS: scroll → translateY 切卡   │  │
│ └──────────────────────┴───────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

**核心交互**：用户进入 section → page sticky-pin → 滚动 → JS 计算 scroll progress → 切 active tab + cards-stage translateY 滑入对应 card。每 20vh 切一卡，5 卡占 100vh = section 总高 200vh。

### Round 46.25 / 46.27（hero 飘卡 WebGL）

跟之前的 round 46.x 一样，全部 **hero 飘卡 WebGL 渲染细节**（offscreen blur halo + canvas2D shadow API），**mobile 跳过 WebGL 不受影响**。

---

## 二、agents-pin-section mobile 现状（设计师已做的部分）

PC 设计师在 `@media (max-width: 900px)` 已经做了核心适配：

| 改动 | 效果 |
|---|---|
| `.agents-pin-section { height: auto; padding: 56px 0; }` | 取消 200vh 高 |
| `.agents-pin { position: static; height: auto; overflow: visible; }` | 取消 sticky pin |
| `.agents-stage { grid-template-columns: 1fr; }` | 折 1 列 |
| `.agents-tabs-v { flex-direction: column; gap: 14px; }` | tabs vertical |
| `.agents-tab-detail { max-height: none; opacity: 1; transform: none; }` | **强制展开所有 detail**（每个 tab 显示完整 description + CTA） |
| `.agents-track { transform: none !important; flex-direction: column; }` | cards stack vertical |

**好评**：设计师识别了 pin-scroll 在 mobile 是反模式，**主动 fallback 到 vertical list**。这跟我之前 landing-mobile-adaptation.md § 4 写的「Sticky-scroll cards 在 mobile collapse to vertical stack」一致。

**Breakpoint 选 900 而不是 760**：合理选择。Pin-scroll 在 tablet portrait（768-900）也空间不够，900 这个点 fallback 比 760 更稳。

---

## 三、本轮发现的 3 个 mobile 问题（设计师漏的）

### 🔴 P0 问题 1：5 个展开的 tab detail + 5 张 cards-stage = 信息重复 + section 极长

PC 上 cards-stage 是 pin-scroll 切换的"视觉锚点"（只显示 1 张当前 card cover）。Mobile 上：
- `agents-tab-detail` 已经**强制展开 5 块**，每块含 title + description + CTA（完整信息）
- `agents-cards-stage` 在下面又 stack **5 张 cover 卡片**

**结果**：
- 信息重复：同样 5 个 use case 被读了两遍
- Section 拉得极长：5 cards × 3:2 aspect ratio × 360 wide ≈ 1500 px 额外滚动
- Cover icon 没有 text 标签，跟上面 tab list 脱钩 → 视觉孤立

### 🟡 P1 问题 2：`.agents-tab-cta` 触控目标 38 < 44

Desktop CSS：`padding: 12px 24px; font-size: 14px;` → 高度约 38 px。
Mobile 没 override，**低于 Apple HIG / WCAG 44 触控 floor**。

这是 use cases section 每个 tab 的主 CTA「Open playbook ↗」，触控关键路径。

### 🟡 P1 问题 3：`.agents-tab-detail p` 字号 15 < 16 body floor

Desktop CSS：`font-size: 15px;`
Mobile 没 override，**低于 16 body floor**（虽然不会 iOS zoom，但读起来吃力）。

---

## 四、本轮修复（3 处 edit）

全部加在现有 `@media (max-width: 900px)` 内（line ~3468-3522）：

```css
@media (max-width: 900px) {
  /* ...existing pin-scroll fallback rules... */

  /* tab-detail p was 15 — below the 16 body floor. Bump on mobile;
     the heading + description now reads as a body paragraph the
     user can actually scan. */
  .agents-tab-detail p { font-size: 16px; line-height: 1.55; }

  /* CTA was 12/24 + 14 = 38 h on desktop. Mobile users tap-target
     floor is 44 (Apple HIG / WCAG). Push to 14/22 + 14, min-height
     44 — same idiom as .stage-actions buttons + .play-cta. */
  .agents-tab-cta {
    padding: 14px 22px;
    min-height: 44px;
    font-size: 14px;
  }

  /* HIDE the right-column cards-stage on mobile.
     Why: with the pin-scroll defeated and every tab-detail force-
     rendered, each .agents-tab-item already shows the title, body
     and CTA in full — a complete card. Showing the 5 cover-only
     cards below would (a) duplicate the same 5 use cases the user
     just read, (b) inflate section height by ~5 × cover-aspect =
     ~1500 px of extra scroll, and (c) leave the brand color icons
     orphaned without any text label nearby. */
  .agents-cards-stage { display: none; }
}
```

---

## 五、为什么选「隐藏 cards」而不是「合并 cover 到 tab-item」？

考虑了两个方案：

**方案 A（采用）：mobile 隐藏 cards-stage**
- 工作量：1 行 CSS
- 信息密度：tab-item 包含全部所需信息（title / description / CTA），cover 是装饰
- 风险：section 变成纯文字 list，视觉单调

**方案 B：JS 把每张 card-cover 移到对应 tab-item header**
- 工作量：~30 行 JS + 重新调试 tab-item layout
- 视觉：每个 tab-item 顶部有 mini cover 缩略图，brand color 锚点
- 风险：tab-item 高度变化、需要为 mobile 重做 cover aspect-ratio

**推荐 A** 理由：
1. 本次只是跟进，最小成本
2. Section 已经有 H2 + sub 提供视觉锚点
3. 5 个 tab 的 active state 用 teal color tint，仍有品牌色出现
4. 如果之后 design review 想要 mobile 视觉补强，再做 B（JS 改 ≤30 行）

---

## 六、跟之前规律的对照

这次修复套用了之前总结的 Alva 产品 mobile 适配 11 大规律：

| 规律 | 应用 |
|---|---|
| #5 复杂卡片分层折叠 | agents-pin-section 已经 fallback 到 vertical list（设计师做的） |
| #7 触控目标分级 | agents-tab-cta 38 → 44（primary action floor） |
| #6 表单字号 ≥16 | tab-detail p 15 → 16（body floor） |
| #11 双 button CTA vertical stack | 不适用（agents-tab-cta 单 button） |
| **新规律 #12（本轮新增）** | **Pin-scroll / scrolly-telling section 在 mobile 上一律 fallback 到 vertical list；同时务必 audit "PC 切换显示的视觉锚点（如 cards-stage、video frame、image gallery）"是否会跟展开的内容信息重复** |

---

## 七、新增产品规律 #12 — Pin-scroll fallback 三件套

Desktop pin-scroll / scrolly-telling 模式在 mobile 上 fallback 时，**至少做 3 件事**：

### 1. 解除 sticky pin

```css
.pin-section { height: auto; padding: 56px 0; }
.pin { position: static; height: auto; overflow: visible; }
```

### 2. 强制展开所有"切换内容"

PC 上一次只显示 1 个详情（active 切换），mobile 上**全部展开**，让用户 scroll 一次看完所有 details。

```css
.detail { max-height: none; opacity: 1; transform: none; }
```

### 3. **隐藏或合并"视觉锚点"（关键易漏点）**

PC 上 pin-scroll 通常配一个右侧/对侧的"视觉锚点"（card cover / video / image / chart），用来切换显示当前 active 内容的视觉。Mobile fallback 后，这个视觉锚点会变成：
- 一字排开的全部 N 个锚点 → 跟左侧展开的 detail **信息重复**
- 或者 stuck on 第一个 → 跟下面 4 个 detail 视觉脱钩

**修法二选一**：

A. **隐藏锚点**（最简）：`.cards-stage, .pin-visual { display: none; }`
B. **合并到 detail**（视觉富）：JS 把每个锚点移到对应 detail 块头部，每个 detail 自带 cover

Alva agents-pin-section 选了 A。

---

## 八、所有之前 mobile 修复完整性验证

12 项核心修复，跟上轮跟进对比：

| 检查项 | 状态 | 数量 |
|---|---|---|
| sticky-cta-bar | ✓ | 4 |
| nav-menu-toggle | ✓ | 12 |
| nav-drawer | ✓ | 30 |
| `.alva-badge { display: none }` mobile | ✓ | 1 |
| Hero `min-height: 100svh` | ✓ | 1 |
| `text-wrap: balance` | ✓ | 1 |
| `scroll-snap-align: start` | ✓ | 4 |
| `cd-close` (KOL modal close) | ✓ | 8 |
| `stage-title-sub` mobile rule | ✓ | 3 |
| `.stage-marquee .stage-actions` vertical stack | ✓ | 3 |
| `play-tab { padding: 11px 14px ... 38px }` | ⚠️ orphaned | 1 |
| **本轮新加：agents-cards-stage hidden** | ✓ | 1 |

⚠️ 注意：`play-tab` 规则还在文件里（line 5277），但是**`.play-tab` HTML 已经被 round 19 删除**了（playbook section 整体重构成 agents-pin-section）。这条 CSS 现在是 dead code，可以下一轮清理。

---

## 九、验收清单

请在 iPhone 14 Pro 393×852 mobile mode 验：

### 新 agents-pin-section
- [ ] 滚到 Use Cases section 进入正常 scroll（无 pin-lock，无 scrolljacking）
- [ ] H2 "What can you do with our Playbooks?" 28px 左右居中显示
- [ ] sub "Powered by verified track records..." 16px
- [ ] 5 个 tab-item 全部展开，每个含：
  - Title (17 px, 第一个是 teal 高亮)
  - Description (16 px)
  - "Open playbook ↗" CTA (≥44 高，dark pill)
- [ ] **底部没有 5 张 cards stack**（cards-stage 已 mobile hidden）
- [ ] Section 总高合理（不再 1500 px 额外滚动）

### Tablet 768-900 范围
- [ ] 同样走 mobile fallback layout（breakpoint 900 包含 tablet portrait）
- [ ] tab-cta 44 高
- [ ] tab-detail p 16

### 之前所有修复无 regression
- [ ] Hamburger drawer 打开（3 tab + For FinTwit creators）
- [ ] Sticky bottom CTA bar 出现 / 消失
- [ ] KOL 卡点击弹出 modal-card-detail + 右上 ✕ 可关
- [ ] Stage 双 CTA vertical stack full-width
- [ ] Hero H1 37 px 显示正常
- [ ] alva-badge 在 mobile 不可见

### 真机性能（推荐 Lighthouse）
- [ ] LCP ≤ 2.5 s（mid-tier 4G profile）
- [ ] CLS ≤ 0.1
- [ ] INP ≤ 200 ms

---

## 十、待清理的 dead code（建议下一轮）

整个 playbook section 被替换成 agents-pin-section 后，下面这些 CSS 块 / JS 函数全部 **orphan**（没有对应 HTML 渲染）：

- `.play-carousel`, `.play-track`, `.play-card`, `.play-tab`, `.play-cover`, `.play-cta` 全套 desktop + mobile CSS
- `.play-dots` + JS `setupPlayDots`（dots position sync）
- `.play-card scroll-snap-align`、bleed 等

文件里仍占 ~150+ 行 dead code。建议下一轮统一删除，能让文件回到 ~9100 行。

---

## 十一、Notion 资料库同步

本文档 + 新规律 #12 应该同步进 Notion **"Mobile Adaptation"** 资料库，建议作为 **资料库 #5 "Settings 页面适配 + Alva 产品 mobile 规律"** 的更新（把 "10 大规律" 升级到 "12 大规律"）。

下一步：要不要我帮你直接更新 Notion 那一页？
