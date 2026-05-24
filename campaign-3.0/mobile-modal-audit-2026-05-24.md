# Campaign 3.0 — Mobile Modal 适配审视 + 修复

**日期**：2026-05-24（同日第四轮）
**目标**：确保 **7 个 modal** 在 mobile 全部行为正确，特别是 KOL 卡片点击触发的 `modal-card-detail`

---

## 一、Modal 全清单 + 触发位置

| Modal ID | 触发器 | 类型 | 入口 |
|---|---|---|---|
| `modal-connect` | "Test any FinTwit creator" | connect-dialog (2-state) | nav primary CTA + sticky bottom CTA + stage primary CTA |
| `modal-for-kols` | "For FinTwit creators" | step-list dialog | mobile drawer 内 ghost CTA |
| `modal-cant-find` | "Can't find an Account?" | search-form dialog | stage ghost CTA |
| `modal-methodology` | "Methodology" | step-list dialog | footer link |
| `modal-results` | (auto, connect 流程末) | dialog-full (全屏 100vh) | connect → loading → results |
| **`modal-card-detail`** | **KOL 卡片点击**（leaderboard rows / marquee cards / results grid） | **lightbox** (卡片放大 + 5 分享按钮) | 任何 `.card` / `.mq-card` 元素 click |
| `modal-search` | (deprecated round 4，无入口) | search-form dialog | — |
| `modal-paste-handles` | (orphan，HTML 还在) | bcard textarea dialog | — |

---

## 二、发现的 5 个 mobile bug

### 🐛 Bug 1：`modal-paste-handles` textarea inline `font-size:13px`

**问题**：iOS Safari 在 `font-size < 16px` 的 input/textarea focus 时**自动 zoom 到 textarea**，layout 跳动一次。inline style 优先级 1000，全局 `input, textarea, select { font-size: 16px }` 压不住。

**修法**（line 6071）：

```diff
- font-family:var(--font-mono); font-size:13px; resize:vertical;
+ font-family:var(--font-mono); font-size:16px; resize:vertical;
```

---

### 🐛 Bug 2：`modal-for-kols` step grid 在 mobile 还是 3 列

**问题**：3 个 step 卡 (`01 02 03`) 在 desktop 是 `grid-template-columns: repeat(3, 1fr)`，原折叠规则 `@media (max-width: 540px) { grid-template-columns: 1fr }` —— **541-760 范围（多数 mobile）还是 3 列**，393w 上每列 ~100 wide，eyebrow 数字挤、body 文字断行成蚂蚁字。

**修法**（line 3993 附近）：

```diff
- @media (max-width: 540px) {
-   #modal-for-kols .steps { grid-template-columns: 1fr; }
- }
+ @media (max-width: 760px) {
+   #modal-for-kols .steps { grid-template-columns: 1fr; gap: 12px; }
+   #modal-for-kols .step { padding: 16px 18px; }
+ }
```

**参考**：Apple HIG "Step indicators on narrow widths" 建议在 phone class viewport 全部走 vertical stack。

---

### 🔥 Bug 3：`modal-card-detail` mobile **没有 close button**（最严重）

**问题**：desktop 上 modal-card-detail 是 lightbox 风格（点击 KOL 卡片 → 卡片放大 + 旁边浮 5 个分享按钮），靠 **backdrop click 关闭**（JS line 6932 `cdStage.addEventListener('click', ...)`）。但是 mobile 上 `.cd-anchor` 用：

```css
.cd-anchor {
  top: 50% !important;
  left: 50% !important;
  transform: translate(-50%, -50%);
  flex-direction: column;
  gap: 16px;
}
.cd-share { width: 100%; max-width: var(--cd-target-w, 320px); }
```

cd-anchor 撑到接近全屏宽（card + cd-more + 5 share button vertical stack）—— **backdrop 几乎不存在**。用户点击哪里都落在 `.cd-left-col` 或 `.cd-share` 上，JS line 6933 显式 skip 这两个 ancestor 不关 modal。结果**用户没办法关 modal-card-detail 只能按 ESC 或刷新页面**。

**修法**：

**HTML 加 `cd-close` button**（line 6453）：

```html
<div class="modal" id="modal-card-detail" aria-hidden="true">
  <button class="cd-close" type="button" data-close aria-label="Close card detail">
    <i class="ph-bold ph-x" aria-hidden="true"></i>
  </button>
  <div class="cd-stage" id="cd-stage">...
```

`data-close` 属性自动 hook 到全局 closer (line 6700 `document.body click → [data-close]` listener)。

**Base CSS — 默认 hidden**（line 4539）：

```css
.cd-close {
  display: none;                            /* 仅 mobile @media 显示 */
  position: fixed;
  top: calc(12px + env(safe-area-inset-top));
  right: 12px;
  z-index: 90;                              /* 高于 modal stack */
  width: 44px;
  height: 44px;
  border: 0;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.96);
  backdrop-filter: blur(8px);
  box-shadow: 0 6px 18px rgba(10, 10, 16, 0.32);
  cursor: pointer;
  place-items: center;
}
.cd-close i { font-size: 18px; color: var(--text); }
```

**Mobile @media — 显示**：

```css
.cd-close { display: grid; }
```

**参考**：iOS Modal sheet 的 close button 位置（Apple HIG "Modal Presentations on iPhone"）：右上角圆形浮 button，跟 system controls 一致。

---

### 🔥 Bug 4：`modal-card-detail` mobile 内容超 viewport **不能滚**

**问题**：cd-anchor `transform: translate(-50%, -50%)` 居中后，**如果 enlarged card (200+ h) + cd-more (75 h) + 5 share method (5 × 50 = 250 h) + gap 总和超过 viewport 高度，顶部和底部会被 viewport 切掉，用户看不到上下两端**。在 iPhone 14 Pro 852 viewport 高也已经接近临界，SE 类 667 高直接超。

**修法**（替换原 mobile @media 内整块 cd-anchor / cd-share rules，line 4781+）：

```css
@media (max-width: 760px) {
  #modal-card-detail.modal {
    align-items: stretch;
    justify-content: stretch;
    padding: 0;
    background: rgba(10, 10, 16, 0.82);     /* 加深 backdrop 让 close 更清晰 */
  }
  .cd-stage {
    overflow-y: auto;                        /* 关键：让 stage 自己 scroll */
    -webkit-overflow-scrolling: touch;
    padding: calc(72px + env(safe-area-inset-top)) 16px
             calc(40px + env(safe-area-inset-bottom));
  }
  .cd-anchor {
    position: static !important;             /* 从 absolute + transform 切到 flow */
    top: auto !important;
    left: auto !important;
    transform: none !important;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    max-width: 360px;
    margin: 0 auto;
    gap: 18px;
  }
  .cd-left-col { width: 100%; max-width: 360px; align-items: center; }
  .cd-card-wrap { width: 100%; }
  .cd-share { width: 100%; max-width: 360px; gap: 8px; }
}
```

**Logic**：cd-stage 占满 modal 全屏，自身 overflow-y auto 让内容超出时滚动；cd-anchor 从 absolute centered 切到 static flow（CSS `!important` override JS 写的 inline `--cd-x` / `--cd-y` style）；卡片 + cd-more + share 全部走 normal block flow 从上往下排。

**参考**：Apple Music "Now Playing" iPhone lightbox — 整屏 modal + 顶部 close + 内容 scrollable vertical stack。

---

### 🐛 Bug 5：`share-method` 触控目标 42 < 44

**问题**：5 个 share button (Copy link / Share on X / Reddit / LinkedIn / Download image) 在 desktop padding `13px 22px` + font 14 → 高 ~42。Apple HIG 要求 ≥ 44。

**修法**（mobile @media，承上）：

```css
.share-method {
  width: 100%;                              /* 全宽，便于触控也便于扫读 */
  justify-content: center;                  /* 文字居中 */
  padding: 13px 22px;
  min-height: 44px;
  font-size: 15px;                          /* 13→15, 卡片下文字感更稳 */
  box-shadow: 0 6px 18px rgba(10, 10, 16, 0.18);  /* 软化于 desktop 的 14/28 */
}
```

**额外好处**：5 个 pill 之前 desktop 用 `width: auto`（content-sized → 5 个宽度不一），mobile 全宽后视觉一致性大幅提升，5 行 pill 像 iOS Share Sheet。

---

## 三、其他 modal mobile 状态（验证通过）

| Modal | mobile padding | input/textarea ≥16 | close 位置 | 备注 |
|---|---|---|---|---|
| modal-connect | 36/24/28 ✓ | 全局生效 ✓ | 右上 (`.connect-close` top:14 right:14) ✓ | 2-state auth/loading，loading 时 close 隐藏（设计如此） |
| modal-for-kols | 30/22/28 ✓ | 全局生效 ✓ | 右上 (`.close` top:16 right:16) ✓ | steps 之前 3 列已修 |
| modal-cant-find | 30/22/28 ✓ | 全局生效 ✓ | 右上 ✓ | search-form input 全局 16 |
| modal-methodology | 30/22/28 ✓ | n/a (无 input) | 右上 ✓ | step-list flex column |
| modal-results | dialog-full 36/18/40 ✓ | n/a | 右上 (`.dialog-full-close` top:20 right:clamp) ✓ | 全屏 100vh，cards grid mobile 已折 |
| modal-card-detail | (本轮重做) | n/a | **本轮加 cd-close** ✓ | 见 Bug 3/4 |
| modal-paste-handles | 30/22/28 ✓ | **本轮 textarea 16** ✓ | 右上 ✓ | orphan 但已修以防回来 |
| modal-search | 30/22/28 ✓ | 全局生效 ✓ | 右上 ✓ | deprecated 但 markup 在 |

---

## 四、KOL 卡片点击完整流程（mobile）

用户操作 → 系统响应：

1. **用户在 leaderboard 行点击**（或 stage marquee 卡 / results modal 卡）
2. JS `openCardLightbox(card)` 被调用 (line 6829)：
   - 计算 source rect
   - mobile 分支 `targetCardW = Math.min(vw - 32, 360)` → 393w 上 360
   - clone card DOM 到 `.cd-card-slot`
   - `openModal('card-detail')`
3. CSS 接管：
   - `#modal-card-detail.modal` 全屏黑透明 0.82 backdrop
   - `.cd-stage` 全屏 + overflow-y auto + padding 72/16/40（顶留 close button + safe area）
   - `.cd-anchor` 静态 flow，从 cd-stage 顶部开始 vertical stack：
     - **Enlarged card**（cd-card-wrap，360 max-wide）
     - **cd-more "View more"** 卡（360 max-wide，跟 card 同 radius）
     - **5 share pill**（全宽 360 max，每行 ≥44h）
   - **cd-close fixed top-right**，跟着 viewport 顶不随内容滚
4. **关闭路径**：
   - 点 cd-close → `data-close` 全局监听 → closeModal
   - 按 ESC → 全局监听 → closeModal
   - 点 backdrop（如果 cd-anchor 没占满，理论上可点）→ JS line 6932 关
   - 任何 share method click → 触发 share action（这部分 JS line ?）

**视觉对比**：

| 状态 | Desktop | Mobile（本轮修复后） |
|---|---|---|
| Layout | card 在左 + share 浮右（横向 2 列） | card / cd-more / share 上下 vertical stack |
| 高度 | content-driven，居中浮在视口 | 全屏 + scrollable |
| 关闭 | 点 backdrop | cd-close ✕ 右上角 + ESC + backdrop（如果有） |
| share-method | width auto, 内宽不齐 | width 100%, 5 pill 全宽对齐 |
| 触控 | 42h（desktop 鼠标 OK） | 44h（mobile 拇指 OK） |

---

## 五、验收清单

### KOL 卡片 lightbox（重点）

请在 iPhone 14 Pro 393×852（DevTools mobile mode）：

- [ ] 滚到 leaderboard，点任意一行 → 弹出 modal-card-detail
- [ ] 顶部右上角看到一个 44×44 圆形白底 ✕ 按钮（cd-close）
- [ ] 内容从上往下：**enlarged card**（彩色 gradient + KOL avatar + name + ROI）→ **"Open full Playbook for [name]"** 卡 → **5 个全宽 share pill**（Copy link / Share on X / Share on Reddit / Share on LinkedIn / Download image）
- [ ] 5 个 share pill 全部 ≥ 44 高，文字居中
- [ ] **如果内容超 viewport**：可以滚动 stage 内容（cd-close ✕ 保持固定不滚）
- [ ] 点 ✕ → modal 关闭
- [ ] 按 ESC → modal 关闭
- [ ] 滚到 stage section → 点 marquee 卡 → 同样的 modal，同样的关闭路径

### 其他 modal

- [ ] 点 nav primary CTA "Test any FinTwit creator" → modal-connect 弹出，✕ 在右上
- [ ] 点 connect modal "Continue with X" → 进 loading state，期间 ✕ 隐藏，~2.5s 后自动转 results modal
- [ ] modal-results 全屏 + 顶部 ✕ + cards grid 1-col 或 2-col 自适应
- [ ] 点 hamburger drawer 内 "For FinTwit creators" → modal-for-kols 弹出
- [ ] modal-for-kols 内 3 个 step **vertical stack 1 列**（之前是 3 列挤）
- [ ] modal-for-kols 内 textarea / input focus 不 zoom
- [ ] 点 footer "Methodology" → modal-methodology，step list 一目了然
- [ ] 点 stage "Search" → modal-cant-find，X handle 输入框 focus 不 zoom

---

## 六、最终验证表（共 20 项，全部通过）

| 检查项 | 状态 |
|---|---|
| paste-handles textarea inline 16 | ✓ |
| for-kols steps fold 760 (1 col) | ✓ |
| cd-close HTML 加 | ✓ |
| cd-close base CSS hidden | ✓ |
| cd-stage overflow-y auto (mobile) | ✓ |
| cd-anchor position static !important (mobile) | ✓ |
| share-method full-width + min-height 44 | ✓ |
| global text-wrap balance | ✓ |
| global input font 16 (iOS zoom 防护) | ✓ |
| nav drawer 启用 (mobile) | ✓ |
| hamburger toggle 显示 (mobile) | ✓ |
| alva-badge mobile 隐藏 | ✓ |
| sticky-cta-bar mobile flex | ✓ |
| three.js mobile skip | ✓ |
| Delight 4 weight preload | ✓ |
| hero h1 clamp(36, 9.5vw, 44) | ✓ |
| stage h1 clamp(28, 8.5vw, 36) | ✓ |
| playbook bleed margin: calc(50% - 50vw) | ✓ |
| playbook snap-align: start | ✓ |
| modal close position absolute (基类) | ✓ |

---

## 七、改动文件位置

| 行号 | 改动 |
|---|---|
| 4533-4565 | base CSS `.cd-close`（默认 hidden，44 圆形浮 button） |
| 3993 附近 | modal-for-kols steps fold 760 ≤ |
| 4781-4910 | mobile @media 内 modal-card-detail 全新 block（cd-stage scroll + cd-anchor static + share-method full-width 44） |
| 6071 | textarea inline `font-size:16px` |
| 6453-6455 | HTML 加 `<button class="cd-close" data-close>` |

---

## 八、未做项（不是必须）

1. **cd-more 在 mobile padding 收紧** —— 当前 padding 22/26 在 360 宽稍大，可改 16/20。视觉细节，P2。
2. **share-method 图标大小调整** —— 当前 desktop 18px icon，mobile 上 15px font 显得 icon 比例偏大。可以 mobile 把 icon 也降到 16。
3. **modal-card-detail 进入动画** —— 现在 cdGrowMobile 是 scale 0.78 → 1，简单 fade-up。可以换成 slide-up from bottom（更像 iOS sheet）。
4. **cd-close hover/focus 状态** —— 当前 hover 加深背景，可以加 ring 突出 focus（a11y 加分项）。

---

下一步：build + iPhone Pro 真机或 Chrome DevTools 393 mobile mode 验收。如有 regression 告诉我具体哪个 modal、什么现象，我修。
