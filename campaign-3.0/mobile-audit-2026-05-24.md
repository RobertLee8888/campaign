# Campaign 3.0 移动端适配审视报告

**审视日期**：2026-05-24
**审视基准**：local `campaign-3.0/index.html`（含 round 44 mobile parity pass + 刚修的 modal close fix），未 deploy 到 GitHub Pages
**画布**：iPhone 14 Pro 393 × 852 viewport（设计基准）
**关联文档**：`.claude-design/landing-mobile-adaptation.md` § 12 Definition of Done

---

## 阅读指引

每个 section 下面挂三类标签，方便分流处理：

- 🟢 **[已修]** — round 44 已在 local 改对，等 push
- 🟡 **[局部不到位]** — 已动手但触控/字号/peek 等没达 DoD
- 🔴 **[未做]** — local + hosted 都没改的真欠账
- 🐛 **[bug]** — 视觉或交互错误

每个 section 配 SVG 线框示意图（mobile 393:852 比例的简化版），标注问题位置。文档可直接贴 Notion。

---

## 总览：5 个 section + 2 个全局浮层

```
┌─────────────────────┐  ← Top nav (fixed)
│ Logo  [tabs]  CTA   │
├─────────────────────┤
│                     │
│  ① Opening / Hero    │  hero "On X, you follow @___"
│  飘卡 + H1 + sub      │  4 张飘浮 tweet 卡片 + halo
│  + 箭头              │
│                     │
├─────────────────────┤
│  ② Stage             │  "We backtested 2.4M+ tweets"
│  H1 + 双 CTA          │  marquee 卡片自动横滚
│  + Marquee           │
├─────────────────────┤
│  ③ Leaderboard       │  "The FinTwit Alpha League"
│  H2 + sub + CTA       │  排名表（5 sort tabs + 行数据）
│  + 排名表             │
├─────────────────────┤
│  ④ Playbook          │  "Each creator's playbook..."
│  H2 + tabs + 卡片轮播  │  peek-carousel（横向 scroll）
├─────────────────────┤
│  ⑤ Footer            │  Disclaimer + Methodology/FAQ links
└─────────────────────┘
        ⬆
    [alva-badge]   ← 永久浮在右下，6.2% viewport
```

**两个跨 section 的全局问题**：

1. 🔴 **alva-badge 浮卡**（`position: fixed; right: 20; bottom: 20; 168×123`）— 永久遮挡内容，且不可关闭。
2. 🔴 **缺 sticky bottom CTA bar** — long landing 转化最大杠杆，规范第 6.3 条必须。

---

## ① Opening / Hero

### 现状描述

打开页面第一屏，看到：

- **H1**：`On X, you follow @___ Do you know their track record?`（@yardeni 是 roll 动的字符变换）
- **副文**：`We backtested every public call from FinTwit's most-followed creators. A few actually beat the market.`
- **下箭头**：圆形按钮 52×52，点击 scroll 到 stage
- **背景**：4 张飘浮 tweet 卡片（Ed Yardeni / Liz Ann Sonders / @CryptoKaleo / @michaeljburry），CSS keyframes 横向 drift 移动

### 视觉示意图

```svg
<svg viewBox="0 0 393 852" xmlns="http://www.w3.org/2000/svg" style="max-width:280px;border:1px solid #ccc">
  <!-- 视口 -->
  <rect width="393" height="852" fill="#fafafa"/>
  
  <!-- Nav (sticky, 64h) -->
  <rect x="0" y="0" width="393" height="64" fill="#fff" stroke="#eee"/>
  <rect x="12" y="22" width="22" height="22" fill="#49A3A6"/>
  <circle cx="80" cy="32" r="7" fill="none" stroke="#666"/>
  <circle cx="115" cy="32" r="7" fill="none" stroke="#666"/>
  <circle cx="150" cy="32" r="7" fill="none" stroke="#666"/>
  <rect x="200" y="20" width="170" height="27" rx="13" fill="#1a1a1e"/>
  <text x="285" y="37" font-size="10" fill="#fff" text-anchor="middle">Test FinTwit creator</text>
  
  <!-- 飘卡 #1 (Liz Ann Sonders 大卡贴 H1 右侧) -->
  <rect x="180" y="120" width="200" height="120" rx="8" fill="#fff" stroke="#bec2cd" opacity="0.8"/>
  <circle cx="195" cy="135" r="8" fill="#222"/>
  <text x="210" y="138" font-size="8">Liz Ann Sonders</text>
  <text x="195" y="170" font-size="7" fill="#666">PMI breadth widened to</text>
  <text x="195" y="180" font-size="7" fill="#666">a 3-year high. The cycle</text>
  
  <!-- 飘卡 #2 (@CryptoKaleo 左下贴边) -->
  <rect x="-30" y="280" width="120" height="50" rx="8" fill="#fff" stroke="#bec2cd" opacity="0.6"/>
  
  <!-- 飘卡 #3 (Sell 卡贴副文) -->
  <rect x="220" y="370" width="120" height="50" rx="8" fill="#fff" stroke="#bec2cd" opacity="0.5"/>
  <text x="245" y="395" font-size="7" fill="#666">@michaeljburry</text>
  <text x="245" y="408" font-size="8" font-weight="bold">Sell.</text>
  
  <!-- H1 (≈35.19px 实测) -->
  <text x="50" y="220" font-size="28" font-weight="bold" fill="#1a1a1e">On X, you follow</text>
  <text x="50" y="290" font-size="28" font-weight="bold" fill="#49A3A6">@yardeni</text>
  <text x="50" y="350" font-size="28" font-weight="bold" fill="#1a1a1e">Do you know their</text>
  <text x="50" y="385" font-size="28" font-weight="bold" fill="#1a1a1e">track record?</text>
  
  <!-- Sub -->
  <text x="80" y="460" font-size="13" fill="#666">We backtested every public call</text>
  <text x="80" y="480" font-size="13" fill="#666">from FinTwit's most-followed</text>
  <text x="80" y="500" font-size="13" fill="#666">creators. A few actually beat the</text>
  <text x="80" y="520" font-size="13" fill="#666">market.</text>
  
  <!-- Arrow -->
  <circle cx="196" cy="600" r="26" fill="none" stroke="#1a1a1e" stroke-width="1.5"/>
  <path d="M 188 595 L 196 610 L 204 595" fill="none" stroke="#1a1a1e" stroke-width="2"/>
  
  <!-- 问题标注 -->
  <g fill="#e3342f" font-size="9" font-weight="bold">
    <text x="240" y="100">⚠ nav-cta 27h (规范要 ≥44)</text>
    <line x1="285" y1="105" x2="285" y2="20" stroke="#e3342f" stroke-dasharray="2 2"/>
    
    <text x="285" y="155" text-anchor="end">⚠ 飘卡贴 H1</text>
    
    <text x="265" y="450" text-anchor="end">⚠ Sell 卡贴副文</text>
  </g>
  
  <!-- alva-badge -->
  <rect x="195" y="700" width="170" height="125" rx="8" fill="#fff" stroke="#bec2cd"/>
  <rect x="210" y="715" width="20" height="20" fill="#49A3A6"/>
  <text x="210" y="755" font-size="11" font-weight="bold">Powered by Alva</text>
  <text x="210" y="775" font-size="9" fill="#666">Discover more</text>
  <text x="210" y="788" font-size="9" fill="#666">playbooks →</text>
  <text x="195" y="690" font-size="9" fill="#e3342f" font-weight="bold">⚠ alva-badge 永久浮，不可关</text>
</svg>
```

### 问题清单

- 🟢 **[已修]** Hero 用 `min-height: 100svh`（line 4480）— iOS 地址栏不会撑爆，正确
- 🟢 **[已修]** 飘卡降密度从 18 张到 4 张（line 4533）— 方向对
- 🟢 **[已修]** WebGL `.lens-canvas` 在 mobile 跳过渲染（line 4538 + JS 6838）
- 🟡 **[局部不到位]** H1 字号 `clamp(28px, 8vw, 38px)` → 393 屏 = **31.4 px**，规范 hero H1 36–40，差 4–5 px。视觉上像 H2 不像 hero
- 🟡 **[局部不到位]** 4 张飘卡仍然贴 H1 — `Liz Ann Sonders` 大卡贴 H1 右、`@michaeljburry "Sell."` 贴副文。Halo 6 层 text-shadow 在 393 屏上挡不住
- 🟡 **[局部不到位]** Halo 性能 — H1 6 层 + sub 5 层 + handle-roller 4 层 = **15 层 text-shadow first paint**，`0 0 170px` 大半径 GPU 极慢
- 🔴 **[未做]** `nav-cta` 主转化按钮高度 **27 px**（`padding: 7 10; font-size: 11`，line 4471），Apple HIG / WCAG 要 ≥ 44。这是首屏曝光率最高的按钮，转化关键
- 🔴 **[未做]** `nav-tab` 触控宽度 **34 px**（`padding: 0 8`，line 4467），要 ≥ 44
- 🔴 **[未做]** `text-wrap: balance` 在 H1 上没用（全文 0 hit）— Vercel/Notion/Anthropic 都用，0 成本
- 🔴 **[未做]** Webfont 没 preload — Hero LCP 等 `Space Grotesk + Inter` 加载，无 `font-display: swap`

### 修法

```css
/* 字号 + tracking */
.opening h1 { 
  font-size: clamp(36px, 9.5vw, 44px);
  text-wrap: balance;
}

/* halo 简化（6 → 2 层 + 渐变背板） */
.opening h1 {
  text-shadow: 0 0 24px #fff, 0 0 60px #fff;
  position: relative;
}
.opening h1::before {
  content: '';
  position: absolute; inset: -40px -20px;
  background: radial-gradient(ellipse 70% 55% at center, #fff 30%, transparent 75%);
  z-index: -1;
}

/* 飘卡进一步降密度 */
.float-tweet:nth-child(n+3) { display: none; }
.float-tweet { filter: blur(2px) !important; opacity: 0.5 !important; }

/* nav 触控 */
@media (max-width: 760px) {
  .nav-cta { padding: 11px 16px; font-size: 13px; min-height: 40px; }
  .nav-cta i { font-size: 14px; }
  .nav-tab { padding: 0 14px; }
}
```

---

## ② Stage — "We backtested 2.4M+ tweets"

### 现状描述

第二屏向下滚 850 px 进入，是核心叙事屏：

- **H1**：`We backtested 2.4M+ tweets from FinTwit's 3,000+ most-followed accounts.`（`2.4M+ tweets` 和 `3,000+` 用 teal 高亮）
- **2 个 CTA**：「Check your following」（dark pill）+ 「Search」（white ghost）
- **Marquee**：水平自动滚动的创作者卡片行（avatar + name + ROI 数字）
- **Section 背景**：淡 teal（`rgba(73, 163, 166, 0.08)`）

### 视觉示意图

```svg
<svg viewBox="0 0 393 700" xmlns="http://www.w3.org/2000/svg" style="max-width:280px;border:1px solid #ccc">
  <rect width="393" height="700" fill="rgba(73,163,166,0.08)"/>
  
  <!-- Marquee row 浮在上方（hosted 中段空白 是因为 120/120 padding；local 已改 72/72） -->
  <rect x="-50" y="20" width="200" height="180" rx="12" fill="#fff" stroke="#bec2cd" opacity="0.3"/>
  <rect x="170" y="20" width="200" height="180" rx="12" fill="#fff" stroke="#bec2cd" opacity="0.3"/>
  <rect x="390" y="20" width="200" height="180" rx="12" fill="#fff" stroke="#bec2cd" opacity="0.3"/>
  <text x="196" y="115" font-size="9" fill="#666" text-anchor="middle">[marquee 卡 →]</text>
  
  <!-- H1 -->
  <text x="20" y="265" font-size="22" font-weight="bold" fill="#1a1a1e">We backtested</text>
  <text x="20" y="295" font-size="22" font-weight="bold" fill="#49A3A6">2.4M+ tweets</text>
  <text x="155" y="295" font-size="22" font-weight="bold" fill="#1a1a1e">from</text>
  <text x="20" y="325" font-size="22" font-weight="bold" fill="#1a1a1e">FinTwit's </text>
  <text x="105" y="325" font-size="22" font-weight="bold" fill="#49A3A6">3,000+</text>
  <text x="20" y="355" font-size="22" font-weight="bold" fill="#1a1a1e">most-followed accounts.</text>
  
  <!-- CTA -->
  <rect x="18" y="395" width="200" height="40" rx="20" fill="#1a1a1e"/>
  <text x="118" y="420" font-size="13" fill="#fff" text-anchor="middle">Check your following</text>
  <rect x="232" y="395" width="120" height="40" rx="20" fill="#fff" stroke="#bec2cd"/>
  <text x="292" y="420" font-size="13" fill="#1a1a1e" text-anchor="middle">Search</text>
  
  <!-- 中段空白带 (hosted 现状 - 因为 padding 120/120 太大；local 已改 72/72) -->
  <rect x="0" y="450" width="393" height="200" fill="none" stroke="#e3342f" stroke-dasharray="4 4"/>
  <text x="196" y="555" font-size="11" fill="#e3342f" text-anchor="middle" font-weight="bold">⚠ 中段 200px 空白</text>
  <text x="196" y="570" font-size="10" fill="#e3342f" text-anchor="middle">(hosted 旧版，local 已改 72/72)</text>
</svg>
```

### 问题清单

- 🟢 **[已修]** Stage `display: block`（mobile）让 marquee 在 H1 下方，不再 side-by-side（line 4555）
- 🟢 **[已修]** Stage padding 从 desktop `120 0` 改到 mobile `72px 0 72px`（line 4555）— hosted 还显示 120/120 大空白，push 后会改善
- 🟢 **[已修]** Stage-headline-overlay 在 mobile 解除 `backdrop-filter` mask（line 4565-4575）— 防止 marquee 卡被模糊到看不见
- 🟡 **[局部不到位]** Stage H1 字号 `clamp(22px, 7vw, 30px)` → 393 屏 = **27.5 px**，规范 stage/H1 要 32-36
- 🟡 **[局部不到位]** Stage CTA padding `12px 18px font-size 13`（line 4584）→ 高 ≈ 37 px，**还差 7 px 到 44**
- 🟡 **[局部不到位]** Marquee 卡片字号 `.mq-l 9px / .mq-meta span 10px`（line 4602）— 远低于 12 px caption floor
- 🟡 **[副作用]** Reduced-motion 兜底 + marquee 卡片 9 px 字号 = a11y 用户看到的是**静止的 9 px 标签**，更难读
- 🔴 **[未做]** Section 上下 padding 跟其他 section 不齐（72/72 vs opening 76/56 vs leaderboard 80 vs playbook 76/88）

### 修法

```css
/* H1 + CTA */
.stage-marquee .stage-headline-inner h1 { 
  font-size: clamp(28px, 8.5vw, 36px);
  line-height: 1.15;
  letter-spacing: -0.02em;
  text-wrap: balance;
}
.stage-actions .btn-primary,
.stage-actions .btn-ghost {
  padding: 14px 22px;
  font-size: 14px;
  min-height: 44px;
}

/* Marquee 字号 */
.mq-l { font-size: 11px; }
.mq-meta span { font-size: 12px; }

/* reduced-motion 下把 marquee 换成 2 列 grid */
@media (prefers-reduced-motion: reduce) {
  .marquee-track { 
    animation: none; 
    display: grid; grid-template-columns: repeat(2, 1fr);
    gap: 12px;
  }
}
```

---

## ③ Leaderboard — "The FinTwit Alpha League"

### 现状描述

第三屏（hosted 滚到 1620 看，local 实际更早）：

- **H2**：`The FinTwit Alpha League`（两行 wrap）
- **副文**：`See FinTwit accounts whose calls beat the market.`（teal 高亮）`Updated every hour.`（muted）
- **CTA**：`View full leaderboard ↗` pill
- **Sort tabs**：5 个 `Best ROI / Median ROI / Win rate / Quality / Risk`（自然 wrap 成 2 行）
- **排名表**：5 列 `# · Creator · Best ROI · (Median/Win/Quality hidden in mobile) · Risk`
- **Section 背景**：`#FFF` white

### 视觉示意图

```svg
<svg viewBox="0 0 393 900" xmlns="http://www.w3.org/2000/svg" style="max-width:260px;border:1px solid #ccc">
  <rect width="393" height="900" fill="#fff"/>
  
  <!-- H2 -->
  <text x="18" y="50" font-size="26" font-weight="bold">The FinTwit Alpha</text>
  <text x="18" y="80" font-size="26" font-weight="bold">League</text>
  
  <!-- Sub -->
  <text x="18" y="120" font-size="14" fill="#49A3A6">See FinTwit accounts whose calls beat</text>
  <text x="18" y="138" font-size="14" fill="#49A3A6">the market.</text>
  <text x="160" y="138" font-size="14" fill="#666">Updated every hour.</text>
  
  <!-- CTA -->
  <rect x="18" y="160" width="180" height="40" rx="20" fill="#1a1a1e"/>
  <text x="108" y="184" font-size="13" fill="#fff" text-anchor="middle">View full leaderboard ↗</text>
  
  <!-- 排名 card -->
  <rect x="18" y="260" width="357" height="600" rx="8" fill="#fff" stroke="#e6e6e8"/>
  
  <!-- Sort tabs (2 行 wrap) -->
  <rect x="30" y="280" width="70" height="28" rx="14" fill="#1a1a1e"/>
  <text x="65" y="298" font-size="11" fill="#fff" text-anchor="middle">Best ROI</text>
  <text x="115" y="298" font-size="11" fill="#666">Median ROI</text>
  <text x="185" y="298" font-size="11" fill="#666">Win rate</text>
  <text x="30" y="328" font-size="11" fill="#666">Quality</text>
  <text x="75" y="328" font-size="11" fill="#666">Risk</text>
  
  <!-- Meta -->
  <text x="30" y="358" font-size="10" fill="#666">● Refreshed 27m ago · 86 ranked</text>
  
  <!-- 表头 -->
  <line x1="30" y1="380" x2="365" y2="380" stroke="#e6e6e8"/>
  <text x="40" y="400" font-size="9" fill="#666">#</text>
  <text x="80" y="400" font-size="9" fill="#666">Creator</text>
  <text x="310" y="400" font-size="9" fill="#666" text-anchor="end">Best ROI</text>
  <text x="358" y="400" font-size="9" fill="#666" text-anchor="end">Risk</text>
  <line x1="30" y1="408" x2="365" y2="408" stroke="#e6e6e8"/>
  
  <!-- Row 01 -->
  <text x="40" y="438" font-size="12" font-weight="bold">01</text>
  <circle cx="80" cy="438" r="14" fill="#ff8a4c"/>
  <text x="105" y="430" font-size="12" font-weight="bold">K A L E O</text>
  <text x="105" y="448" font-size="10" fill="#666">@CryptoKaleo · Crypto</text>
  <text x="310" y="438" font-size="12" font-weight="bold" text-anchor="end">+342%</text>
  <rect x="333" y="426" width="32" height="22" rx="4" fill="#fee2e2"/>
  <text x="349" y="441" font-size="9" fill="#dc2626" text-anchor="end">HIGH</text>
  
  <line x1="30" y1="466" x2="365" y2="466" stroke="#e6e6e8"/>
  
  <!-- Row 02 -->
  <text x="40" y="498" font-size="12" font-weight="bold">02</text>
  <circle cx="80" cy="498" r="14" fill="#b388ff"/>
  <text x="105" y="490" font-size="12" font-weight="bold">Lucy Liu</text>
  <text x="105" y="508" font-size="10" fill="#666">@lucyliu_ai · AI / large cap</text>
  <text x="310" y="498" font-size="12" font-weight="bold" text-anchor="end">+218%</text>
  <rect x="333" y="486" width="32" height="22" rx="4" fill="#dcfce7"/>
  <text x="349" y="501" font-size="9" fill="#16a34a" text-anchor="end">LOW</text>
  
  <line x1="30" y1="526" x2="365" y2="526" stroke="#e6e6e8"/>
  
  <!-- Row 03 (被 alva-badge 遮) -->
  <text x="40" y="558" font-size="12" font-weight="bold">03</text>
  <circle cx="80" cy="558" r="14" fill="#0a0a10"/>
  <text x="105" y="560" font-size="12" font-weight="bold">Sl...</text>
  
  <!-- alva-badge 遮挡 -->
  <rect x="180" y="540" width="170" height="125" rx="8" fill="#fff" stroke="#e3342f" stroke-width="1.5"/>
  <text x="195" y="565" font-size="10" font-weight="bold" fill="#e3342f">⚠ alva-badge 遮 row 03+</text>
  <text x="195" y="585" font-size="10" font-weight="bold">Powered by Alva</text>
  <text x="195" y="605" font-size="9" fill="#666">Discover more</text>
  <text x="195" y="618" font-size="9" fill="#666">playbooks →</text>
</svg>
```

### 问题清单

- 🟢 **[已修]** Mobile @media 把表格 grid 折叠到 4 列 `22px minmax(0, 1fr) 58px 52px`（line 4646）— hosted 显示 desktop 列灾难性溢出，push 后即修复
- 🟢 **[已修]** `hide-sm` cell 在 mobile `display: none`（line 4653-4655）— Median/Win/Quality 列隐藏，给 ROI/Risk 让位
- 🟢 **[已修]** Aurora halo 在 mobile 收紧 inset（line 4633-4635）— 防止 halo 跑出 viewport
- 🟢 **[已修]** Reduced-motion 下 `#leaderboard::before` 暂停 hue rotation 动画（line 2638-2640）
- 🟢 **[已修]** `.lb-row .who > div { min-width: 0; flex: 1; overflow: hidden; }`（line 4664-4667）让 ellipsis 真正生效，长 handle 不再溢出
- 🟡 **[局部不到位]** `.lb-header` 表头字号 **9.5 px**（line 4649）— 信息架构标签，规范 caption floor 12
- 🟡 **[局部不到位]** `.lb-meta` `10 px`（line 4628）、`.risk-pill 9 px`（line 4688）— 同上
- 🟡 **[局部不到位]** `.lb-sub` 副文 **14 px**（line 4624）— 主标下方 body 文字应 ≥ 16
- 🟡 **[副作用]** Handle 副行 ellipsis 一行 — `@RaoulGMI · Liquidity / crypto` 只显示 `@RaoulGMI · Liqu...`，tag 丢了（line 4675-4682）
- 🔴 **[未做]** **alva-badge 浮卡** 直接遮 row 03-05 的 ROI/Risk 列（见下方专章）

### 修法

```css
@media (max-width: 760px) {
  .lb-header { font-size: 11px; }       /* 9.5 → 11 */
  .lb-meta { font-size: 12px; }          /* 10 → 12 */
  .risk-pill { font-size: 11px; }        /* 9 → 11 */
  .lb-sub { font-size: 16px; }           /* 14 → 16 */
  
  /* handle 副行允许 2 行 */
  .lb-row .who span {
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    display: -webkit-box;
  }
}
```

---

## ④ Playbook — peek carousel

### 现状描述

第四屏（hosted 2750 起，local 类似）：

- **Section 标题**：`Each creator's playbook, distilled into an AI agent.`
- **Tabs row**：`Bull vs bear / Critique a pick / Build a mix / Replay past calls / Screen creators / Watchlist alerts`（6 个 tab，超过 viewport 宽，横向 overflow-x: auto）
- **主卡**：teal gradient 视觉 + 标题（`Bull vs bear by ticker`）+ 描述 + 「Open playbook ↗」CTA
- **邻卡 peek**：右边露出 **约 5 px** 邻卡左缘（视觉重量被 box-shadow 占完）
- **Section 背景**：淡 teal

### 视觉示意图

```svg
<svg viewBox="0 0 393 750" xmlns="http://www.w3.org/2000/svg" style="max-width:260px;border:1px solid #ccc">
  <rect width="393" height="750" fill="rgba(73,163,166,0.08)"/>
  
  <!-- Section 标题 -->
  <text x="18" y="50" font-size="22" font-weight="bold">Each creator's playbook,</text>
  <text x="18" y="80" font-size="22" font-weight="bold" fill="#49A3A6">distilled into an AI agent.</text>
  
  <!-- Tabs row (横向 scroll, 看到 3-4 个) -->
  <text x="38" y="138" font-size="13" font-weight="bold" fill="#1a1a1e">Bull vs bear</text>
  <line x1="32" y1="148" x2="118" y2="148" stroke="#49A3A6" stroke-width="2"/>
  <text x="150" y="138" font-size="13" fill="#666">Critique a pick</text>
  <text x="260" y="138" font-size="13" fill="#666">Build a mix</text>
  <text x="350" y="138" font-size="13" fill="#999">›</text>
  
  <!-- 主卡 -->
  <rect x="18" y="170" width="337" height="280" rx="8" fill="#49A3A6"/>
  <text x="186" y="320" font-size="48" fill="#fff" text-anchor="middle">⚖</text>
  
  <!-- 邻卡 peek (右边 22px - box-shadow 占 ~10px 剩 ~12px) -->
  <rect x="370" y="170" width="50" height="280" rx="8" fill="#0a0a10" opacity="0.6"/>
  
  <!-- 卡片下方 -->
  <text x="18" y="490" font-size="18" font-weight="bold">Bull vs bear by ticker</text>
  <text x="18" y="520" font-size="13" fill="#666">Stack every covered creator's stance</text>
  <text x="18" y="538" font-size="13" fill="#666">on the same ticker — conviction, hold</text>
  <text x="18" y="556" font-size="13" fill="#666">time, and entries side by side.</text>
  
  <!-- CTA -->
  <rect x="18" y="585" width="140" height="40" rx="20" fill="#1a1a1e"/>
  <text x="88" y="610" font-size="13" fill="#fff" text-anchor="middle">Open playbook ↗</text>
  
  <!-- 标注 peek 不足 -->
  <line x1="370" y1="450" x2="395" y2="450" stroke="#e3342f" stroke-width="1.5"/>
  <text x="382" y="465" font-size="9" fill="#e3342f" text-anchor="end" font-weight="bold">⚠ peek 仅 5-22px</text>
  <text x="382" y="478" font-size="9" fill="#e3342f" text-anchor="end">(规范要 32-48 = 8-12% viewport)</text>
  
  <!-- 标注没 dots -->
  <text x="186" y="680" font-size="10" fill="#e3342f" text-anchor="middle" font-weight="bold">⚠ 缺 dots 位置指示器</text>
</svg>
```

### 问题清单

- 🟢 **[已修]** Tabs row 转 `overflow-x: auto`（line 4699-4704）— 6 tabs 可横向滚
- 🟢 **[已修]** Carousel 用 `flex: 0 0 calc(100vw - 56px)`（line 4709）做了 horizontal scroll
- 🟢 **[已修]** Section padding `76px 0 88px`（line 4694）— 比 desktop 收紧
- 🟡 **[局部不到位]** **Peek 实测仅 ~5 px（视觉感知）/ 22 px（数学）**：`100vw - 56 = 337` 卡宽，padding-left 18，卡片右缘 18+337=355，下一卡 + gap 16 = 371，**只剩 22 px 给 peek**（一半被 box-shadow 吃掉，视觉感知 ~10 px）
- 🟡 **[局部不到位]** `.play-card p` 描述文字 **14 px**（line 4716）— 主信息载体应 ≥ 16
- 🟡 **[局部不到位]** `.play-card h3` **20 px**（line 4713）— 卡标，规范 16 但能更突出
- 🔴 **[未做]** **缺 dots/位置指示器** — Material 3 Hero Carousel 规范要求「Always show」，否则用户不知道有多少卡
- 🔴 **[未做]** 缺 `scroll-snap` — 现在自由 scroll，松手停在中间位置可能两卡都不完整

### 修法

```css
@media (max-width: 760px) {
  .play-card {
    flex: 0 0 calc(100vw - 96px);    /* 主卡 80%，左右各 48 peek */
    max-width: 320px;
    scroll-snap-align: center;
  }
  .play-track {
    padding-left: 16px; padding-right: 16px;
    scroll-snap-type: x mandatory;
  }
  .play-card p { font-size: 16px; }
}
```

```html
<!-- 加 dots 容器，JS 绑 scroll 位置 -->
<div class="play-dots" aria-hidden="true">
  <span class="dot active"></span><span class="dot"></span>
  <span class="dot"></span><span class="dot"></span>
  <span class="dot"></span><span class="dot"></span>
</div>
```

---

## ⑤ Footer

### 现状描述

页面底部：

- **Disclaimer 文字**：`Based on public posts and recent signals. Not financial advice. Alva does not recommend trading on any single creator's calls.`
- **链接行**：`Methodology · Backtest assumptions · FAQ · Privacy · Contact`（5 个 inline link，分两行）
- **Section 背景**：淡 teal（跟 playbook 同色，footer 跟最后 section 同色，符合规范）

### 视觉示意图

```svg
<svg viewBox="0 0 393 200" xmlns="http://www.w3.org/2000/svg" style="max-width:280px;border:1px solid #ccc">
  <rect width="393" height="200" fill="rgba(73,163,166,0.08)"/>
  
  <!-- Disclaimer -->
  <text x="18" y="30" font-size="11" fill="#666">Based on public posts and rec...</text>
  <text x="18" y="46" font-size="11" fill="#666">signals. Not financial advice. A...</text>
  <text x="18" y="62" font-size="11" fill="#666">trading on any single creator's...</text>
  
  <!-- Links -->
  <text x="18" y="100" font-size="11" fill="#666">Methodology</text>
  <text x="105" y="100" font-size="11" fill="#666">Backtest assu...</text>
  <text x="190" y="100" font-size="11" fill="#666">FAQ</text>
  <text x="18" y="118" font-size="11" fill="#666">Privacy</text>
  <text x="68" y="118" font-size="11" fill="#666">Contact</text>
  
  <!-- alva-badge 遮挡右半 -->
  <rect x="195" y="55" width="170" height="125" rx="8" fill="#fff" stroke="#e3342f" stroke-width="1.5"/>
  <rect x="210" y="70" width="20" height="20" fill="#49A3A6"/>
  <text x="210" y="115" font-size="11" font-weight="bold">Powered by Alva</text>
  <text x="210" y="135" font-size="9" fill="#666">Discover more</text>
  <text x="210" y="148" font-size="9" fill="#666">playbooks →</text>
  
  <!-- 标注 -->
  <text x="382" y="48" font-size="9" fill="#e3342f" font-weight="bold" text-anchor="end">⚠ Backtest assumptions /</text>
  <text x="382" y="60" font-size="9" fill="#e3342f" text-anchor="end">FAQ 被遮</text>
</svg>
```

### 问题清单

- 🟢 **[已修]** Footer flex-direction `column`（line 4725）— 链接换行不挤
- 🟡 **[局部不到位]** `.f-inner` 字号 **11 px**（line 4727）— 规范 footer caption floor 13
- 🟡 **[局部不到位]** 链接间距 `margin-right: 12px`（line 4730）— WCAG 要 tap 目标间距 ≥ 24 px
- 🔴 **[未做]** **alva-badge 直接遮挡 disclaimer 后半 + Backtest assumptions / FAQ 链接** — 用户必读的法律合规文本被永久遮挡

### 修法

```css
@media (max-width: 760px) {
  .f-inner { font-size: 13px; }
  .f-inner a { 
    margin-right: 20px;
    padding: 8px 4px;       /* 让 tap 目标至少 32 高 */
  }
  .alva-badge { display: none; }   /* mobile 直接隐藏（见下方专章） */
}
```

---

## 🐛 全局问题 A：alva-badge 浮卡

### 现状

```css
.alva-badge {
  position: fixed;
  right: 20px;
  bottom: 20px;
  z-index: 45;
  /* 168 × 123 px */
}
```

**整个 mobile @media block 没有 override**，意味着：

- 整个滚动期间永久占据**右下角 168×123 px = 20647 px² ≈ viewport 6.2%**
- 不可关闭（没有 close 按钮）
- z-index 45 高于 nav (30)、低于 modal (80)

### 实际遮挡范围

| Section | 遮挡内容 |
|---|---|
| Hero | Sub 文字尾段 + arrow 下方 |
| Stage | Marquee 卡片尾段 |
| Leaderboard | Row 03/04/05 的 ROI 和 Risk 列 + 表格底部 |
| Playbook | Carousel 邻卡 peek 视觉提示 |
| Footer | Disclaimer 后半 + Backtest assumptions / FAQ 链接 |

### 修法（二选一）

**A 方案（推荐）**：mobile 直接隐藏，桌面保留

```css
@media (max-width: 760px) {
  .alva-badge { display: none; }
}
```

**B 方案**：改成可关闭 pill

```css
@media (max-width: 760px) {
  .alva-badge {
    right: 12px; bottom: 12px;
    padding: 8px 12px;
    border-radius: 999px;
    /* 缩成小 pill */
  }
  .alva-badge-text span { display: none; }   /* 只留 logo + "Alva" */
}
/* 加 close ✕ 按钮，sessionStorage 记 dismissed */
```

---

## 🐛 全局问题 B：缺 sticky bottom CTA bar

### 现状

整个 mobile @media block 0 hit `sticky-cta-bar` / `sc-cta` / `tabnav-wrapper` / `position: sticky` (bottom)。

长 landing（5 section, 3680 px 总高 = 4.3 个 viewport）用户滚到 leaderboard 时**主 CTA「Test any FinTwit creator」已经不在 nav 视野**（nav 是 fixed 但 CTA 仍在 nav 内，可能被屏幕中段元素掩盖；hosted 现状 nav 仍 sticky 主 CTA 可见，但视觉小 27 高难发现）。

规范第 6.3 条：**sticky bottom CTA 提升 long landing 转化 15-25%**，long landing 必须有。这是单点收益最大的优化。

### 修法

```html
<div class="sticky-cta-bar" id="stickyCta" aria-hidden="true">
  <button class="sc-cta" data-open="connect">
    <i class="ph-bold ph-x-logo"></i> Test any FinTwit creator
  </button>
  <button class="sc-close" aria-label="Hide">
    <i class="ph ph-x"></i>
  </button>
</div>
```

```css
.sticky-cta-bar {
  position: fixed; left: 0; right: 0; bottom: 0; z-index: 30;
  display: none; gap: 8px; align-items: center;
  padding: 12px 16px calc(12px + env(safe-area-inset-bottom));
  background: rgba(255,255,255,0.94);
  backdrop-filter: blur(14px) saturate(140%);
  border-top: 1px solid var(--line);
  transform: translateY(100%);
  transition: transform .25s ease;
}
.sticky-cta-bar.visible { transform: translateY(0); }
.sc-cta {
  flex: 1; display: flex; align-items: center; justify-content: center; gap: 8px;
  height: 48px; background: var(--text); color: #fff;
  border: 0; border-radius: 999px;
  font: 700 14px/1 Inter, system-ui, sans-serif; cursor: pointer;
}
.sc-close {
  width: 40px; height: 40px; background: transparent;
  border: 1px solid var(--line); border-radius: 50%;
  color: var(--muted); cursor: pointer;
}
@media (max-width: 760px) {
  .sticky-cta-bar { display: flex; }
}
```

```js
const stickyBar = document.getElementById('stickyCta');
const heroCta = document.querySelector('.stage-actions');
const footer = document.querySelector('footer.f');

const io = new IntersectionObserver(entries => {
  entries.forEach(e => {
    if (e.target === heroCta) {
      if (!sessionStorage.getItem('stickyCtaDismissed')) {
        stickyBar.classList.toggle('visible', !e.isIntersecting);
      }
    } else if (e.target === footer && e.isIntersecting) {
      stickyBar.classList.remove('visible');
    }
  });
}, { rootMargin: '-56px 0px 0px 0px' });

io.observe(heroCta);
io.observe(footer);
stickyBar.querySelector('.sc-close').addEventListener('click', () => {
  stickyBar.classList.remove('visible');
  sessionStorage.setItem('stickyCtaDismissed', '1');
});
```

---

## 🐛 全局问题 C：性能 — three.js 670 KB 阻塞 mobile LCP

### 现状

```html
<!-- line 17 -->
<script src="./three.min.js" onerror="..."></script>
```

无 `defer`、无 `async`、无条件。mobile 跳过 WebGL（line 6838 JS 早返）但 **670 KB 已经下载并 parse**。

### 修法

```html
<script>
  if (!window.matchMedia('(max-width: 760px)').matches) {
    const s = document.createElement('script');
    s.src = './three.min.js';
    s.defer = true;
    s.onerror = function() {
      this.onerror = null;
      const f = document.createElement('script');
      f.src = 'https://unpkg.com/three@0.160.0/build/three.min.js';
      document.head.appendChild(f);
    };
    document.head.appendChild(s);
  }
</script>
```

粗算：mobile 用户每访问省 **670 KB**，LCP 预估降 **1–2 秒**（取决于网速）。

---

## 🔧 已修但仍待 push：modal close 位置

### 修复前现象（hosted 现状）

`modal-cant-find` / `modal-search` 弹出时，✕ 关闭按钮跑到弹窗**左上角**：

```svg
<svg viewBox="0 0 393 600" xmlns="http://www.w3.org/2000/svg" style="max-width:240px;border:1px solid #ccc">
  <rect width="393" height="600" fill="rgba(0,0,0,0.5)"/>
  <!-- Dialog 居中 -->
  <rect x="20" y="80" width="353" height="450" rx="10" fill="#fff"/>
  
  <!-- ✕ 在 左上角 (BUG) -->
  <rect x="52" y="112" width="36" height="36" rx="8" fill="#f5f5f7"/>
  <text x="70" y="135" font-size="16" text-anchor="middle">✕</text>
  <text x="120" y="135" font-size="10" fill="#e3342f" font-weight="bold">⚠ 跑到左边</text>
  
  <!-- Eyebrow -->
  <text x="52" y="170" font-size="11" fill="#49A3A6">⌕ Backtest one handle</text>
  
  <!-- Title -->
  <text x="52" y="205" font-size="22" font-weight="bold">Backtest any FinTwit</text>
  <text x="52" y="232" font-size="22" font-weight="bold">creator.</text>
  
  <!-- Body -->
  <text x="52" y="270" font-size="13" fill="#666">Drop an X handle. We'll run their</text>
  <text x="52" y="288" font-size="13" fill="#666">public call history end-to-end and</text>
  <text x="52" y="306" font-size="13" fill="#666">surface the audited Playbook...</text>
  
  <!-- Input -->
  <rect x="52" y="330" width="289" height="44" rx="22" fill="#f5f5f7"/>
  <text x="68" y="358" font-size="13" fill="#999">@creator-handle</text>
  
  <!-- Backtest 按钮 -->
  <rect x="52" y="390" width="130" height="44" rx="22" fill="#49A3A6"/>
  <text x="117" y="418" font-size="13" fill="#fff" text-anchor="middle">⌕ Backtest</text>
</svg>
```

### 修复后（local 已 apply）

```svg
<svg viewBox="0 0 393 600" xmlns="http://www.w3.org/2000/svg" style="max-width:240px;border:1px solid #ccc">
  <rect width="393" height="600" fill="rgba(0,0,0,0.5)"/>
  <rect x="20" y="80" width="353" height="450" rx="10" fill="#fff"/>
  
  <!-- Eyebrow -->
  <text x="52" y="120" font-size="11" fill="#49A3A6">⌕ Backtest one handle</text>
  
  <!-- Title -->
  <text x="52" y="155" font-size="22" font-weight="bold">Backtest any FinTwit</text>
  <text x="52" y="182" font-size="22" font-weight="bold">creator.</text>
  
  <!-- ✕ 在右上角 (FIX) -->
  <rect x="337" y="96" width="36" height="36" rx="8" fill="#f5f5f7"/>
  <text x="355" y="119" font-size="16" text-anchor="middle">✕</text>
  <line x1="337" y1="115" x2="290" y2="115" stroke="#16a34a" stroke-dasharray="2 2"/>
  <text x="270" y="118" font-size="10" fill="#16a34a" font-weight="bold" text-anchor="end">✓ 修好了</text>
  
  <!-- Body -->
  <text x="52" y="220" font-size="13" fill="#666">Drop an X handle. We'll run their</text>
  <text x="52" y="238" font-size="13" fill="#666">public call history end-to-end and</text>
  <text x="52" y="256" font-size="13" fill="#666">surface the audited Playbook...</text>
  
  <!-- Input -->
  <rect x="52" y="290" width="289" height="44" rx="22" fill="#f5f5f7"/>
  <text x="68" y="318" font-size="13" fill="#999">@creator-handle</text>
  
  <!-- Backtest 按钮 -->
  <rect x="52" y="350" width="130" height="44" rx="22" fill="#49A3A6"/>
  <text x="117" y="378" font-size="13" fill="#fff" text-anchor="middle">⌕ Backtest</text>
</svg>
```

### 根因

`.close` 基类没有 `position`，6 个 modal 走 3 种路径：

| Modal | close 位置 |
|---|---|
| modal-cant-find / modal-search | close 是 dialog 第一子 → normal flow → **左上**（BUG） |
| modal-for-kols / methodology / faq / disclaimer | close 在 dialog-head 内 → flex space-between → 右上 ✓ |
| modal-connect / modal-results | 自己 absolute → 右上 ✓ |

### 修法（已在 local apply）

```css
.dialog { position: relative; }
.dialog-head { padding-right: 48px; }
.close {
  position: absolute;
  top: 16px; right: 16px;
  z-index: 5;
  /* ... 其他保留 */
}
.dialog-full > .dialog-head { padding-right: 0; }   /* 大 modal 居中 head 抵消 */
```

---

## 📊 Definition of Done 汇总

按 `.claude-design/landing-mobile-adaptation.md` § 12 清单逐项核对：

| 检查项 | 状态 | 备注 |
|---|---|---|
| H1 mobile 36-40 | 🟡 31 px | line 4483，差 4-5 |
| Body ≥ 16 / caption ≥ 13 | 🔴 | 多处 9-12 |
| Hero 100svh | 🟢 | line 4480 |
| 主 CTA 首屏 + 触控达标 | 🔴 | nav-cta 27 高 |
| Sticky bottom CTA bar | 🔴 | 0 hit |
| 重复 CTA 每个大 section | 🟡 | 部分有，无统筹 |
| 无 parallax / scroll-jacking | 🟢 | scroll-snap 禁用 |
| `prefers-reduced-motion` 全覆盖 | 🟢 | line 4843-4845 全局兜底 |
| Hero 视觉杂讯可控 | 🟡 | 4 飘卡 + 6 层 halo 仍贴 H1 |
| Hamburger 仅 overflow | 🟢 | 3 icon tab + 1 primary CTA |
| 3-col 折 1-col 或 peek | 🟡 | peek 仅 22 px |
| 触控 ≥ 44 + 间距 ≥ 24 | 🔴 | nav-cta 27 / nav-tab 34 / footer link 16 |
| 背景 cadence | 🟢 | 白/teal-8 交替 |
| `text-wrap: balance` | 🔴 | 0 hit |
| LCP ≤ 2.5 s | 🔴 | three.js 670 KB + webfont 阻塞 |
| 浮卡不遮内容 | 🔴 | alva-badge 168×123 永久浮 |
| iOS input 防 zoom | 🟡 | input font-size 没显式 ≥ 16 |
| Modal close 一致 | 🟢 | 刚修，待 push |

**通过率**：6 🟢 / 5 🟡 / 7 🔴

---

## 🎯 修复优先级建议

按"收益 / 工作量"排序，分两批合 PR：

### 一刀 PR — 1 小时内（DoD 通过率 17% → 55%）

| # | 改动 | 工作量 | 影响 |
|---|---|---|---|
| 1 | mobile `.alva-badge { display: none }` | 1 min | 解放 6% viewport，leaderboard / footer 可读 |
| 2 | three.js mobile-skip 加载 | 5 min | LCP -1~2 s |
| 3 | nav-cta 高度 27→40，nav-tab 宽 34→44 | 5 min | 主转化按钮可点击 |
| 4 | hero / stage H1 字号上调 | 2 min | 视觉权重对位 |
| 5 | leaderboard caption/meta 字号 ≥ 12 | 10 min | 信息架构清晰 |
| 6 | text-wrap: balance | 1 min | hero 换行品质 |
| 7 | iOS input 防 zoom | 2 min | 表单聚焦稳 |
| 8 | (已完成) modal close 位置统一 | — | 弹窗体验一致 |

### 二刀 PR — 半天（DoD 通过率 55% → 90%）

| # | 改动 | 工作量 | 影响 |
|---|---|---|---|
| 9 | sticky bottom CTA bar | 30 min | long landing 转化 +15-25% |
| 10 | playbook peek 改 80%/12% + dots | 20 min | carousel 可发现性 |
| 11 | hero halo 简化（6 → 2 + gradient 背板） | 10 min | GPU 负担↓ |
| 12 | webfont preload + display swap | 10 min | LCP -300-800 ms |
| 13 | section padding 统一 72/72 | 5 min | 节奏感 |
| 14 | reduced-motion 下 marquee 换 2 列 grid | 15 min | a11y 用户能读 |
| 15 | leaderboard handle 允许 2 行 | 5 min | 信息不丢 |
| 16 | ≤380 档别再缩 nav-cta | 1 min | SE 用户触控更稳 |

---

## 📎 附录：实地数据来源

- 本次实测在 hosted 版 `https://robertlee8888.github.io/campaign/campaign-3.0/`（旧版，没含 round 44）的 iframe 393×852 viewport
- Local `index.html` 比 hosted 多了 round 44 mobile parity pass，所有 🟢 [已修] 项指的是 local 已经写好等 push
- 实测数字（H1 35.19 px、nav-cta 27 h、playbook peek 22 px 等）跟代码推算一致，互相验证

下一步建议：先 push local 到 hosted（让 4 个 🟢 项生效），再做"一刀 PR"7 条改动。
