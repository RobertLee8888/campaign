# Campaign 3.0 mobile follow-up — 2 个用户反馈修复

**日期**：2026-05-25（同日 round 3）
**触发**：用户反馈 2 个具体问题：

1. "第一个 section 下面一半屏幕都是白色，看不到后面飘的卡片"
2. "第三个 section 在移动端不要 tab，改成图+文+按钮，直接纵向往下排。而且这个 section 左右要撑满"

---

## 一、Hero 飘卡分布 — 7 张覆盖全屏

### 问题

Mobile 上之前 `display: none` 隐藏除前 4 张外的所有飘卡。但前 4 张 `--y` 值是：
- order=1 → y=32%
- order=2 → y=18%
- order=3 → y=4%
- order=4 → y=38%

**4 张全部集中在视口上半（4-38%）**，下半屏 50-100% 完全空白 → 用户感觉「看不到飘卡」。

### 修法

改成 hide-all + handpick 7 张覆盖完整 y 分布：

| nth-child | data-order | --y | 分布位置 |
|---|---|---|---|
| 2 | 2 | 18% | 上 1/5 |
| 3 | 3 | 4% | 顶 |
| 4 | 4 | 38% | H1 左侧 |
| 7 | 7 | 56% | sub 下方 |
| 8 | 8 | 92% | 底 |
| 11 | 11 | 78% | 下 1/4 |
| 15 | 15 | 68% | 中下 |

```css
.opening-bg .float-tweet { display: none !important; }
.opening-bg .float-tweet:nth-child(2),
.opening-bg .float-tweet:nth-child(3),
.opening-bg .float-tweet:nth-child(4),
.opening-bg .float-tweet:nth-child(7),
.opening-bg .float-tweet:nth-child(8),
.opening-bg .float-tweet:nth-child(11),
.opening-bg .float-tweet:nth-child(15) {
  display: block !important;
}
```

避免 H1 中心 30-55% 区域过载（halo 已经在挡），重点补**顶 0-15%** 和 **下半屏 55-95%**。

---

## 二、Agents Section 移动端重构 — 图+文+按钮纵向 + 撑满

### 问题

Round 19 把 use cases 改成 pin-scroll 模式，desktop 漂亮，mobile 上之前的适配是：
- 5 个 tab-item 全展开 detail
- 隐藏右侧 cards-stage（cover 不显示）

用户反馈：
- 不要 tab UI 感（active state 高亮、左侧竖排切换感）
- 要"图+文+按钮"每个 use case 一组（cover + title + body + CTA）
- Section 左右撑满（cards 视觉到 viewport 边）

### 修法（CSS + JS 配合）

#### CSS 改动（mobile @media 内）

```css
@media (max-width: 900px) {
  /* Section full-bleed: 取消 inner padding，covers 跑到 viewport 边 */
  .agents-pin-inner { padding: 0; gap: 0; }
  .agents-stage { grid-template-columns: 1fr; gap: 0; }
  .agents-head { padding: 0 18px; margin-bottom: 28px; }  /* head 单独留 LR */

  /* tab-item 变 card：cover + title + body + CTA 纵向 */
  .agents-tabs-v { flex-direction: column; gap: 36px; }
  .agents-tab-item {
    background: transparent;       /* 取消 pill bg */
    border-radius: 0;
    padding: 0;
    display: flex;
    flex-direction: column;
    gap: 0;
  }
  .agents-tab-item:hover,
  .agents-tab-item.active { background: transparent; }   /* 取消 tab affordance */

  /* Cover full-bleed 100vw + 不要圆角 */
  .agents-tab-item .agents-card-cover {
    width: 100vw;
    max-width: 100vw;
    margin: 0 0 18px;
    border-radius: 0;
    aspect-ratio: 3 / 2;
  }
  .agents-tab-item .agents-card-cover i { font-size: 96px; }

  /* Title 改 H3 风格、统一颜色（不再 active state tint） */
  .agents-tab-v {
    font-size: 22px;
    font-weight: 600;
    line-height: 1.25;
    color: var(--text);
    padding: 0 18px;
    margin: 0 0 10px;
    cursor: default;               /* 不再是 tab button */
  }
  .agents-tab-item.active .agents-tab-v { color: var(--text); }  /* override desktop teal */

  /* Body + CTA 加回 18 LR padding */
  .agents-tab-detail {
    max-height: none; opacity: 1; transform: none;
    padding: 0 18px;
    gap: 16px;
  }
  .agents-tab-detail p { font-size: 16px; line-height: 1.55; max-width: none; }
  .agents-tab-cta {
    padding: 14px 22px;
    min-height: 44px;
    font-size: 14px;
    align-self: flex-start;
  }

  /* 原 cards-stage 隐藏（covers 已 clone 到 tab-item） */
  .agents-cards-stage { display: none; }
}
```

#### JS 改动 — `setupAgentsMobile`

把 desktop 上 `.agents-cards-stage` 内的 5 张 `.agents-card-cover` **克隆**到对应 `.agents-tab-item` 头部。**用克隆不是移动**，desktop 原 cover 保留（pin-scroll 正常）。

```js
(function setupAgentsMobile() {
  const MQ = '(max-width: 900px)';
  function sync() {
    const isMobile = window.matchMedia(MQ).matches;
    const tabs = document.querySelectorAll('.agents-tab-item');
    const cards = document.querySelectorAll('.agents-cards-stage .agents-card');
    if (!tabs.length) return;
    tabs.forEach((tab, i) => {
      const existing = tab.querySelector('.agents-card-cover.mobile-clone');
      if (isMobile) {
        if (existing) return;                          // 已注入
        const srcCover = cards[i]?.querySelector('.agents-card-cover');
        if (!srcCover) return;
        const clone = srcCover.cloneNode(true);
        clone.classList.add('mobile-clone');
        clone.removeAttribute('id');
        tab.insertBefore(clone, tab.firstChild);
      } else if (existing) {
        existing.remove();                             // 恢复 desktop
      }
    });
  }
  sync();
  window.matchMedia(MQ).addEventListener?.('change', sync);
})();
```

特性：
- **Idempotent**：用 `.mobile-clone` 标记位防重复注入
- **Resize-safe**：matchMedia change listener，desktop ↔ mobile 切换都正确
- **Desktop 不受影响**：原 cards-stage 内 5 张 cover 仍在，pin-scroll JS 不变

---

## 三、Mobile 上每个 use case 的最终视觉

```
┌────────────────────────────────────────┐
│                                         │  ← 18px LR padding（仅 head）
│  What can you do with our Playbooks?    │
│  Powered by verified track records...   │
│                                         │
├═════════════════════════════════════════┤  ← 这里到 viewport 边
│                                         │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │  ← cover full-bleed 100vw, 3:2
│  ▓                                   ▓ │     teal gradient + icon 96px
│  ▓             📊                    ▓ │
│  ▓                                   ▓ │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│                                         │
│   Backtest win rates                    │  ← 22px / 600, 18px LR
│   Stop guessing who's good. See         │  ← 16px body
│   exactly which calls made money...     │
│   ┌───────────────────────────────┐    │
│   │  Open playbook ↗              │    │  ← 44h, 14px font
│   └───────────────────────────────┘    │
│                                         │
│   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ │  ← 36px gap
│                                         │
│  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │  ← 下一个 use case cover
│  ▓             📈                    ▓ │     (dark variant)
│  ...                                    │
└────────────────────────────────────────┘
```

每个 use case 一组（cover full-bleed + title + body + CTA），上下排 5 个，section 总长可控。

---

## 四、Desktop 完全不受影响（自检）

- `.agents-pin-section { height: 200vh }` 保留 ✓
- `.agents-pin { position: sticky }` 保留 ✓
- `.agents-cards-stage` 在 desktop 仍显示原 5 张 cover ✓
- Pin-scroll JS controller 不变 ✓
- mobile @media 仅在 ≤900 触发，desktop ≥901 走原 layout ✓

---

## 五、所有之前 mobile 修复 regression 检查

12 项核心修复全部 verified：

| 检查项 | 状态 |
|---|---|
| sticky-cta-bar | ✓ 4 |
| nav-menu-toggle (hamburger) | ✓ 12 |
| alva-badge mobile hidden | ✓ |
| Hero `100svh` | ✓ |
| `text-wrap: balance` | ✓ |
| `scroll-snap-align: start` (playbook orphan, 待清理) | ✓ |
| `cd-close` (KOL modal close) | ✓ 8 |
| `stage-title-sub` mobile rule | ✓ 3 |
| `stage-marquee .stage-actions` vertical stack | ✓ |
| `agents-pin-section` 三件套 (pin off / detail expand / 视觉锚点处理) | ✓ |
| **本轮新加：setupAgentsMobile JS** | ✓ |
| **本轮新加：hero 飘卡 7 张分布** | ✓ |

---

## 六、验收清单

### iPhone 14 Pro 393×852 mobile mode

#### Hero（第一个 section）
- [ ] 滚到顶，看到 7 张飘卡分布全屏（不仅上半）
- [ ] 顶部 0-15% 有 1-2 张飘卡（order=3 在 y=4%）
- [ ] **下半屏 55-95% 有 4 张飘卡**（y=56/68/78/92%）—— 解决「白色」问题
- [ ] H1 周围 30-55% 没有飘卡贴文字（halo 不需要硬挡）

#### Agents（第三个 section）
- [ ] 滚到 Use Cases section，看到 H2 + sub
- [ ] **不再有 tab UI** — 没有左侧竖排的 title 列表，没有 active 高亮
- [ ] 5 个 use case 完整纵向排开，每个含：
  - **图（cover full-bleed 100vw 跑到 viewport 边）** + brand color gradient + icon 96px
  - 标题 22px 黑色（不是 teal 高亮 active 模式）
  - 描述 16px
  - "Open playbook ↗" CTA pill (44h)
- [ ] **Section 左右撑满** — cover 边到边，没有左右白边
- [ ] H2/sub/title/body/CTA 文本 ledged 18px LR padding（不是边到边，便于阅读）
- [ ] 不再有底部独立的 5 个 cards stack（原 cards-stage 已 hidden）

#### 跨设备
- [ ] DevTools resize 在 mobile/desktop 间切换，agents section 视觉正确（desktop 回到 pin-scroll，mobile 回到 vertical stack）
- [ ] resize 不出现重复 cover 或孤儿 cover（idempotent check）

### Tablet 768-900
- [ ] 走 mobile fallback layout（breakpoint 900）

### 之前所有修复无 regression
- [ ] Hamburger drawer 正常
- [ ] Sticky bottom CTA bar 正常
- [ ] KOL modal-card-detail close 正常
- [ ] Stage 双 CTA vertical stack
- [ ] iOS focus input 不 zoom

---

## 七、产品规律 #12 升级

原规律 #12「Pin-scroll fallback 三件套」第 3 条原本是「**隐藏或合并视觉锚点**」二选一。本轮选择"合并"路线（JS clone cover 到 detail），实战验证了「合并」方案的工程成本（~30 行 JS + resize-aware）。

**升级版 #12 第 3 条**：

| 选 | 工作量 | 视觉感 | 使用场景 |
|---|---|---|---|
| 隐藏 (`display: none`) | 1 行 CSS | 纯文字 list 略单调 | 视觉锚点是纯装饰、文字已足够（如 Stripe pricing feature list） |
| 合并到 detail（JS clone） | ~30 行 JS + resize listener | 每个 detail 完整 (图+文+CTA) | 视觉锚点承载品牌色 / 关键信息（如 Alva use case 用 brand gradient cover） |

判断标准：cover/visual 是否携带「不可文字替代」的信息（color、视觉对比、动态元素）。是 → 合并；否 → 隐藏。

Alva agents 选合并，因为每个 cover 用 brand color gradient（teal / dark / warm 区分 use case 类型），是品牌色锚点，不可去。

---

下一步：deploy 后真机验证。如果飘卡位置还需要微调（比如某张特别贴 H1），告诉我具体看哪张位置不对，调对应 nth-child y 值即可。
