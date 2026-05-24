# Campaign 3.0 移动端适配 — v3 改动记录

**日期**：2026-05-24（同日第三轮）
**基准**：`campaign-3.0/index.html` 在 v2（changes-2026-05-24.md）之上
**主题**：playbook peek-bug 修复 + nav 汉堡包菜单

本轮做了两件事：

1. **修 playbook carousel peek 没贴 viewport 边**（用户反馈：还没到边就被截断）
2. **把 nav 上隐藏的 ghost CTA 恢复**，做成 mobile hamburger drawer，所有 tab + 操作按钮都进去

---

## 一、Playbook carousel peek 修复

### 原 bug

v2 改动后，mobile 上主卡 297 wide + `scroll-snap-align: center` + `padding-left/right 16`：
- 首卡居中 → 卡左缘距 viewport 左 48 px（看起来像有个 48 px 空白边距）
- 邻卡 peek 32 px 露出，但被 desktop 14/36 box-shadow 软化，看起来像 fade out 不像被 viewport edge clip

用户实际感受：「卡片还没到 viewport 边就被截断」。

### 修法

**3 处改动**（line ~5035-5076）：

```css
/* 1. carousel 用 desktop 同款 bleed-out 技巧，强制 100vw 跳出
   section 内容盒，让 peek 真的能到 viewport 边 */
.play-carousel {
  margin-left:  calc(50% - 50vw);
  margin-right: calc(50% - 50vw);
  width: 100vw;
}

/* 2. 改 scroll-snap-align 从 center → start。
   first card left-anchored at padding-left:16, peek 在右边卡的
   左边 32 px 露出，被 viewport overflow:hidden 硬切到 viewport
   右边缘。Apple Music / Spotify 同款 idiom。 */
.play-card {
  flex: 0 0 calc(100vw - 64px);   /* 329 wide on 393w, 之前 297 */
  max-width: 380px;                /* 之前 320 */
  scroll-snap-align: start;        /* 之前 center */
  scroll-snap-stop: always;
}

/* 3. 软化 box-shadow，让 peek 卡右边是清晰断口不是模糊渐变 */
.play-card,
.play-card:hover { box-shadow: 0 6px 18px rgba(10, 10, 16, 0.10); }
```

≤380 同步：`flex: 0 0 calc(100vw - 48px); max-width: 360px`（375w 上 327 wide + 18 peek）。

### 数学验证（393w）

| 元素 | 位置 |
|---|---|
| viewport | 0 — 393 |
| track padding-left | 0 — 16 |
| 首卡 (snap-start) | 16 — 345 (329 wide) |
| gap | 345 — 361 |
| 邻卡 left | 361 |
| 邻卡 visible peek | 361 — 393 = **32 px**，**被 viewport edge 硬切** ✓ |

参考：Apple Music carousel + Spotify Home carousel 都用同款 `scroll-snap-align: start` + edge bleed。

---

## 二、Mobile hamburger drawer

### 原 bug

v2 mobile 把 `.nav-cta-ghost`（"For FinTwit creators"）直接 `display: none` 隐藏 — 入口完全消失。这是创作者侧的核心入口，不能丢。

### 设计选型

**3 种 drawer 类型对比**：

| 类型 | 优点 | 缺点 | 代表站点 |
|---|---|---|---|
| **顶部下拉 sheet** ✓ | 跟 nav 视觉延续；不抢屏；不需要复杂手势 | section 标题被遮短暂 | GitHub mobile、Linear iOS |
| 右侧抽屉 | 全屏感强；常见 mobile app 模式 | 需要 swipe 手势；280-320 px 宽难放完整内容 | Notion、Twitter/X iOS |
| 全屏 overlay | 最沉浸；专注 | 完全遮屏；返回路径不直观 | YouTube、Reddit |

**选顶部下拉 sheet**。理由：
- Alva 是 marketing landing page 不是 app，用户主要做"看 + 选"动作，不需要 immersive overlay
- nav 已经 fixed top blur，drawer 从 nav 下方滑出是逻辑延续
- 跟现有的 modal 系统不冲突（modal z-index 80 > drawer z-index 30）
- iOS / Android 用户都熟悉

### HTML 改动

`.nav-actions` 里加 hamburger toggle button（line 5471-5483）：

```html
<button class="nav-menu-toggle" type="button"
        id="nav-menu-toggle"
        aria-controls="nav-drawer"
        aria-expanded="false"
        aria-label="Open menu">
  <i class="ph ph-list" aria-hidden="true"></i>
  <i class="ph ph-x" aria-hidden="true"></i>
</button>
```

`</header>` 之前加 drawer markup（line 5491-5507）：

```html
<div class="nav-drawer" id="nav-drawer" role="navigation" aria-label="Mobile menu">
  <a class="nav-drawer-item" href="#audit-yours" data-section="audit-yours">
    <i class="ph ph-book-open"></i><span>Playbooks</span>
  </a>
  <a class="nav-drawer-item" href="#agents" data-section="agents">
    <i class="ph ph-squares-four"></i><span>Use Cases</span>
  </a>
  <a class="nav-drawer-item" href="#leaderboard" data-section="leaderboard">
    <i class="ph ph-trophy"></i><span>Leaderboard</span>
  </a>
  <div class="nav-drawer-sep" aria-hidden="true"></div>
  <button class="nav-drawer-action" type="button" data-open="for-kols">
    <i class="ph-bold ph-megaphone"></i><span>For FinTwit creators</span>
  </button>
</div>
```

设计要点：
- **3 个 tab + 1 个 action**，中间用 1px hairline 分组（iOS Settings list idiom）
- 每行 **52 高**（远超 WCAG 44 floor），icon + 15 px label + 14 px gap
- 行内 hover/active 用 `rgba(73, 163, 166, 0.06)` teal-tint，跟品牌色一致
- `.nav-drawer-item.active` 同时跟 desktop `.nav-tab.active` 同步（scrollspy mirror，下面 JS 段）

### Hamburger toggle 设计

- **44×44**（WCAG 2.5.8 floor），≤380 收到 40×40
- 圆形 outline button（border 1px hairline）
- 两个 icon 叠加（`ph-list` ☰ / `ph-x` ✕），按 `aria-expanded` 状态 cross-fade + 90° rotate
- 视觉跟 .nav-cta 主 CTA 区分：CTA 是 filled dark pill，toggle 是 outline 圆按钮

### CSS 状态机

```css
/* default closed: 折叠 + 透明 + 不可交互 */
.nav-drawer {
  position: absolute; top: 100%; left: 0; right: 0;
  background: rgba(255,255,255,0.97);
  backdrop-filter: blur(18px) saturate(150%);
  transform: translateY(-12px);
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
  transition: transform .26s cubic-bezier(.22, 1, .36, 1),
              opacity .2s,
              visibility 0s .26s;     /* visibility 延迟 = transition 时长，
                                          确保 fade-out 时不挡 click */
  display: none;                       /* desktop 永远不渲染 */
}
.nav-drawer.open {
  transform: translateY(0);
  opacity: 1;
  visibility: visible;
  pointer-events: auto;
  transition: transform .28s cubic-bezier(.22, 1, .36, 1),
              opacity .22s,
              visibility 0s 0s;        /* visibility 立刻生效，让交互即时 */
}

/* mobile 启用 */
@media (max-width: 760px) {
  .nav-tabs { display: none; }            /* tabs 从 nav 移除 */
  .nav-cta-ghost { display: none; }       /* ghost CTA 也进 drawer */
  .nav-menu-toggle { display: inline-grid; place-items: center; }
  .nav-drawer { display: flex; }          /* override base display:none */
}
```

### JS 控制（line 8095-8200）

完整 `setupNavDrawer` IIFE，行为：

1. **open() / close() / isOpen()** — 切 `.open` 类，同步 aria-expanded + aria-label + body overflow lock
2. **toggle click** — 互换 open/close，stopPropagation 避免触发 document click 关闭
3. **drawer item click** — `setTimeout(close, 60)` 让 smooth-scroll / modal opener 先 grab click，再关 drawer
4. **ESC 键** — 关 + focus 回 toggle button（a11y）
5. **outside-tap** — document.click listener，drawer 和 toggle 之外的点击关 drawer
6. **resize 到 desktop** — `matchMedia('(min-width: 761px)').change` 监听，强制关 drawer 避免遗留
7. **scrollspy mirror** — `MutationObserver` 观察 `.nav-tab.active` class 变化，同步到 `.nav-drawer-item.active`，让 drawer 打开时用户能看到当前所在 section

### 参考站点对照

| 站点 | drawer 类型 | 内容 | 触发位置 |
|---|---|---|---|
| **GitHub mobile** | 顶部下拉 sheet | 3-5 个 tab + auth | nav 右侧 ☰ |
| **Linear iOS** | 顶部下拉 sheet | section nav + account | nav 右侧 ☰ |
| **Vercel mobile** | 顶部下拉 sheet | 产品 tab + login + start | nav 右侧 ☰ |
| **Notion** | 右侧抽屉 | tree-nav + workspace switch | nav 左侧 ☰ |
| **Anthropic** | 全屏 overlay | nav links + product CTA | nav 右侧 ☰ |
| **本次方案** ✓ | 顶部下拉 sheet | 3 tab + 1 action | nav 右侧 ☰ |

跟 GitHub/Linear/Vercel 同款，跟 Anthropic 那种全屏遮蔽刻意区分（Anthropic 适合 brand-heavy marketing；Alva 是 tool product，drawer 应该轻一点）。

---

## 三、清理与微调

### ≤380 SE-class 微调

```css
@media (max-width: 380px) {
  .nav-cta { padding: 10px 13px; font-size: 12px; gap: 5px; min-height: 38px; }
  .nav-cta i { font-size: 13px; }
  .nav-menu-toggle { width: 40px; height: 40px; }   /* 44 → 40, 给 nav 多一点呼吸 */
  /* .nav-tab 规则已删除 — tabs 在 drawer 内不再 inline */
}
```

### nav-inner padding

`.nav-inner` mobile 从 `0 10px gap 6px` 改 `0 12px gap 8px` —— 多了 hamburger 后给左右多点呼吸。

---

## 四、自查清单（每条都验证过）

| 改动 | 行号 | 验证 |
|---|---|---|
| Hamburger HTML button | 5471 | ✓ id, aria-controls, aria-expanded, aria-label 全齐 |
| Drawer HTML | 5491 | ✓ role="navigation", aria-label, 3 tab + 1 action + sep |
| Base CSS .nav-menu-toggle | 278 | ✓ 44×44, border 1px, 2 icon 叠加 cross-fade |
| Base CSS .nav-drawer | 306 | ✓ absolute top:100%, blur, transition, default display:none |
| Base CSS .nav-drawer-item / -action | 343 | ✓ 52 高, 18 icon, hover teal-tint, active w600 |
| Mobile @media .nav-tabs hide | 4901 | ✓ 完全 display:none |
| Mobile @media .nav-cta-ghost hide | 4902 | ✓ 进 drawer |
| Mobile @media toggle/drawer 启用 | 4906-4907 | ✓ display flex |
| ≤380 toggle 缩 40 | 5392 | ✓ |
| JS setupNavDrawer | 8095 | ✓ open/close, ESC, outside-tap, resize, scrollspy mirror |
| Playbook carousel 100vw bleed | 5012 | ✓ margin: calc(50% - 50vw) |
| Playbook snap-align: start | 5048 | ✓ first card left-anchored |
| Playbook max-width 380 + flex 100vw-64 | 5045 | ✓ 329 wide, 32 peek |
| Playbook box-shadow 软化 | 5054 | ✓ 6/18 替代 desktop 14/36 |
| ≤380 play-card peek 同步 | 5377 | ✓ flex 100vw-48 + max 360 |

---

## 五、Definition of Done 更新

跟 v2 比，DoD 通过率从 **18 🟢 / 0 🟡 / 0 🔴** 保持 **18 🟢**，但实际改善了两条：

| 检查项 | v2 | v3 |
|---|---|---|
| 3-col 折 1-col 或 peek | 🟢 peek 22 px | 🟢 peek 32 px 真到 viewport edge ✓ |
| Hamburger 仅 overflow | 🟢 但 ghost CTA 直接 deletd | 🟢 ghost CTA 进 drawer 不丢功能 ✓ |

特别是「Hamburger 仅 overflow」那条 — v2 实际是「丢内容」不是 overflow，v3 才真正符合规范。

---

## 六、PC 端 round 45-46 改动同步检查

扫了所有 `round 45` / `round 46.x` 改动（共 7 条），全部位于 `.float-tweet` 相关：

| Round | 改动 | 影响 mobile? |
|---|---|---|
| 45.3 | shadowPad 56 → 48 (canvas drop-shadow 去掉) | ❌ mobile 走 DOM 不走 WebGL |
| 46 | shader composite 从 mix → premultiplied → 又回 mix（46.3） | ❌ 同上 |
| 46.1 | tier-near blur 1.4 → 0；tier shadow 重新启用 | ❌ 同上 |
| 46.2 | .float-tweet border alpha 0.025 → 0.015 | ⚠️ mobile DOM 飘卡也会继承（但 alpha 极低肉眼几乎不可见，不影响） |
| 46.3 | shader composite mix() 形式 | ❌ |
| 46.4 | Three.js 改成动态加载 + `waitForThree` polling | ⚠️ 跟我 v2 加的 three.js mobile-skip 完美兼容（mobile 早 return 不会进 polling 循环） |

**结论**：PC 端 round 45-46 没有需要 sync 到 mobile 的改动。round 46.4 跟我 v2 的 three.js mobile-skip 是协同关系，自然 work。

---

## 七、验收清单（请在 iPhone 14 Pro 393×852 跑）

### Hamburger drawer
- [ ] nav 右侧出现一个 44×44 圆形 outline 按钮，里面是 ☰ icon
- [ ] 点击 → drawer 从 nav 下方滑下，约 300 ms
- [ ] drawer 内 3 个 tab + 1 个 "For FinTwit creators" 按钮，中间一道 hairline
- [ ] 当前所在 section 对应的 tab 高亮（teal icon + 加粗）
- [ ] 滚动页面时 drawer 内的 active 跟着切换
- [ ] 点 tab → drawer 滑回 + 页面平滑滚到该 section
- [ ] 点 "For FinTwit creators" → drawer 滑回 + 弹出 `modal-for-kols`，✕ 在右上角（v1 modal close fix 生效）
- [ ] ESC 键关 drawer + 焦点回到 ☰ 按钮
- [ ] 点 drawer 外区域（hero / stage 等）关 drawer
- [ ] drawer 打开时点 "Test any FinTwit creator" 主 CTA（不在 drawer 内）也能正常开 connect modal

### Playbook carousel peek
- [ ] 滚到 Use Cases section，首卡左缘距 viewport 左边约 16 px
- [ ] 首卡右侧能看到第二卡左缘约 32 px（teal/warm/dark gradient 边带）
- [ ] **peek 卡的右边缘是硬切（被 viewport edge clip），不是模糊渐变**
- [ ] swipe 向右，第二卡 left-snap 到 16 px，第三卡 peek 露 32 px
- [ ] 滚到最后一卡（6th），它 left-snap 到 16，右边没有下一卡（自然结束）
- [ ] 底部有 6 个 dots，当前卡对应 dot 变深变大

### ≤380 (iPhone SE / fold front-screen)
- [ ] hamburger 缩到 40×40
- [ ] nav-cta 38 高（保留触控合规）
- [ ] play-card 327 wide + 邻卡 peek ~18 px

### 之前所有改动仍生效（regression check）
- [ ] alva-badge 在 mobile 不可见
- [ ] sticky bottom CTA bar 在滚过 stage 后出现
- [ ] hero H1 字号 ~37 px（不是 31）
- [ ] modal close ✕ 都在右上角
- [ ] iOS focus input 不再 zoom

---

## 八、Rollback 速查

如果 hamburger drawer 有问题：

```bash
# 删 drawer HTML（line 5470-5510）
# 删 base CSS .nav-menu-toggle / .nav-drawer 等（line 278-365）
# 删 mobile @media 内 .nav-menu-toggle / .nav-drawer 启用规则（line 4906-4907）
# 恢复 mobile @media 内 .nav-tabs / .nav-cta-ghost / .nav-tab 旧规则
# 删 JS setupNavDrawer IIFE（line 8095-8197）
```

如果 playbook peek 还是不对：

```bash
# 删 .play-carousel mobile bleed（line 5012-5016）
# 改回 v2 的 .play-card flex/max-width/snap-align
```

---

## 九、文件状态

| 字段 | v2 末 | v3 末 |
|---|---|---|
| 行数 | 7954 | ~8200 |
| 大小 | 351 KB | ~365 KB |
| 改动 chunk | 13 处 | + 5 处 (drawer HTML/CSS/JS + playbook fix) |

---

## 十、未做项（可后续）

1. **drawer focus trap** — 当前 ESC + outside-tap 已经够用，但严格 a11y 要求 Tab 键在 drawer 内循环。复杂度中等，下次加。
2. **drawer item 进入动画** — 现在 drawer 整体 slide-in，但 4 个 item 可以做 stagger 微动画（每个 delay 30 ms）。视觉效果好但不是 must-have。
3. **drawer 跟 sticky bottom CTA 互斥** — 现在 drawer 开着的时候 sticky bar 还在底部显示。逻辑上不冲突（一个顶一个底），但可以选择 drawer 开时隐藏 sticky bar 减少干扰。等真机看效果再决定。
4. **playbook horizontal vs vertical 最终决定** — 我个人推荐 vertical stack（参考 Linear/Stripe/Apple），但你保留 horizontal carousel 也合理，所以这次只修了 peek bug 没动布局。等你 deploy 看 carousel 实际感受再决定要不要切 vertical。
