# Campaign 3.0 mobile follow-up — round 4

**日期**：2026-05-25（同日 round 4）
**触发**：用户反馈 "移动端还是有问题，尤其是第三个 section"
**方法**：用 hosted iframe 393×852 mobile 模式实地审视，找出具体 bug

---

## 一、实测发现（hosted 现状）

用 iframe 模拟 mobile，量到的精确数据：

### Hero（第一个 section）✅ 已修复

```
visibleTweetCount: 7
visibleTweetOrders: ["2","3","4","7","8","11","15"]
对应 --y: 18%, 4%, 38%, 56%, 92%, 78%, 68%
```

7 张飘卡分布全屏，**用户「下半屏白色」问题已解决**。

### Agents（第三个 section）⚠️ 部分修复

实测：
```
tab.x = 18,  tab.w = 391
cover.x = 18, cover.w = 391    ← cover 跟 tab 起点一致！
```

**这就是 user 看到的问题**：

- Tab-item 在 `.agents-side` 内，从 x=18 起（左侧有 18 px 父容器 padding/margin）
- Cover 是 tab-item 的子，**继承 tab-item 的 x=18 起点**
- 即使我之前写了 `width: 100vw`（=391），cover 还是从 x=18 开始 → **左侧 18 px 空白、右侧溢出**
- 实地截图能清楚看到 cover 左边一条**深色 vertical strip**（dark variant cover 特别明显）

### 截图证据

Agents section 第一个 use case「Backtest win rates」teal cover + 第二个 dark variant「Simulate your ROI」cover：
- Cover 视觉上不到 viewport 左缘（约 18 px 留白）
- 右侧也类似（cover 溢出被 overflow 截断）
- **User 反馈「左右要撑满」就是指这个**

---

## 二、根因分析

`width: 100vw` 在嵌套容器内**不够**让元素真正贴 viewport edge。原因：

```
viewport (391 wide)
└─ .opening-bg (391 wide, x=0)
└─ .agents-pin-section (391 wide, x=0)
   └─ .agents-pin (391 wide, x=0)
      └─ .agents-pin-inner (391 wide, padding 0)
         └─ .agents-stage (391 wide)
            └─ .agents-side
               └─ .agents-tabs-v
                  └─ .agents-tab-item (391 wide, x=18 from container chain)
                     └─ .agents-card-cover (width: 100vw = 391, x= parent x = 18)
                        ← Cover 从 x=18 起算 391，溢出到 x=409
                        ← viewport overflow:hidden 在 x=391 切掉右边
                        ← 视觉结果：左侧 18 px 空白
```

**Width 100vw 只控制 width 数值，不控制元素位置**。位置由父容器 x-offset 决定。

**Desktop 的 `.play-carousel` 已经用过同款 bleed 技巧**（line 1893）：

```css
.play-carousel {
  margin-left:  calc(50% - 50vw);    /* 关键 */
  margin-right: calc(50% - 50vw);
  width: 100vw;
}
```

`calc(50% - 50vw)` 计算的是「父容器中心到 viewport 中心的负距离」，用作 margin 把元素拉到 viewport 0。

---

## 三、本轮修复（1 处 CSS edit）

`.agents-tab-item .agents-card-cover` mobile rule 强化：

```css
@media (max-width: 900px) {
  .agents-tab-item .agents-card-cover {
    position: relative;
    width: 100vw;
    max-width: 100vw;
    margin-left:  calc(50% - 50vw);   /* ← 新加 */
    margin-right: calc(50% - 50vw);   /* ← 新加 */
    margin-top: 0;
    margin-bottom: 18px;
    border-radius: 0;
    aspect-ratio: 3 / 2;
  }
}
```

跟 `.play-carousel` 同款 idiom，desktop 已验证 production-ready。

---

## 四、当前 hosted vs local 差异 — 用户必须 push 一次

我用 iframe 量到 hosted 实际状态：

| 改动 | Hosted 状态 | Local 状态 |
|---|---|---|
| 飘卡 7 张分布 | ✅ deployed | ✅ |
| setupAgentsMobile JS clone | ✅ deployed | ✅ |
| `.agents-cards-stage { display: none }` | ✅ deployed | ✅ |
| `.agents-tab-item { background: transparent }` | ✅ deployed | ✅ |
| `.agents-tab-v { color: var(--text) }` | ✅ deployed | ✅ |
| `.agents-tab-cta { min-height: 44px }` | ⚠️ unclear | ✅ |
| `.agents-tab-item .agents-card-cover { width: 100vw }` | ❌ **未 push** | ✅ |
| `.agents-tab-item .agents-card-cover { margin: calc(50% - 50vw) }` | ❌ **本轮新加** | ✅ |

**结论**：Hosted 是**部分 push** 状态。User 看到「还有问题」很大程度是这个原因。

**下一步：把 local 整个 `index.html` 完整 push 一次**，cover 就会真正撑满 viewport。

---

## 五、预期修复后的视觉

```
┌─────────────────────────────────────────┐  ← viewport 391 wide
│   What can you do                       │   ← 18 LR padding
│   with our Playbooks?                   │
│   Powered by verified track records...  │
│                                          │
█████████████████████████████████████████ │  ← cover 真到 viewport 0
█           📊  Backtest win rates       █ │   100vw, 3:2 aspect
█████████████████████████████████████████ │
│                                          │
│   Backtest win rates              (22px) │   ← 18 LR padding
│   Stop guessing who's good. See ... (16) │
│   ┌──────────────────┐                   │
│   │  Open playbook ↗ │                   │   ← 44h CTA
│   └──────────────────┘                   │
│                                          │
│                                          │   ← 36 gap
█████████████████████████████████████████ │  ← 下一个 cover
█           📈  Simulate your ROI        █ │   dark variant
█████████████████████████████████████████ │
│                                          │
│   ...                                    │
└─────────────────────────────────────────┘
```

**核心**：cover 真正铺到 viewport 左右两侧（x=0 到 x=391），不再有 18 px 留白。

---

## 六、验收清单（push 后）

### iPhone 14 Pro 393×852 mobile mode

#### Hero（已修，复测）
- [ ] 7 张飘卡分布全屏（顶 / 中 / 下 都有）
- [ ] 下半屏 55-95% 看到至少 3-4 张飘卡

#### Agents（本轮重点）
- [ ] Section H2 + sub 左对齐 18 LR padding
- [ ] **每个 use case 的 cover 真正铺满 viewport 0-100vw**（左右没有 white/dark vertical strip）
- [ ] Cover icon 居中（96px）
- [ ] Title (22px / 600 / 黑色，**不再 teal 高亮 active 模式**)
- [ ] Description 16px / line-height 1.55
- [ ] "Open playbook ↗" CTA pill 44h
- [ ] 5 个 use case 上下排（cover→title→body→CTA 一组），item 之间 36 gap
- [ ] Section 总高约 2700 px（5 个 use case × ~530 each）— 合理

#### 跨设备
- [ ] DevTools resize desktop ↔ mobile 切换，cover 位置正确
- [ ] Desktop 仍是 pin-scroll（cover 在右列、不在 tab-item）

#### Regression
- [ ] Hamburger drawer 正常
- [ ] Sticky bottom CTA bar 正常
- [ ] KOL 卡 modal close 右上角
- [ ] Stage 双 CTA vertical stack

---

## 七、Push 工作流提醒

下次 deploy mobile 改动建议：

1. **全文件 push** — 不要 partial copy paste（容易漏 CSS）
2. **Push 前先在 DevTools mobile mode 验** — 用 393 × 852 viewport 跑一遍验收清单
3. **Push 后用 hosted URL + iframe 393 复测** — 验证 hosted 跟 local 一致
4. **关键 CSS 标记**：所有 `calc(50% - 50vw)` 类 bleed 技巧的修改，push 时特别注意整段不能漏

---

## 八、产品规律 #12 进一步升级

之前规律 #12 第 3 条「视觉锚点 合并到 detail」选项已经在 Alva agents 验证过，本轮发现一个**配套坑**：

**升级版**：

合并视觉锚点到 detail 时，clone 的 cover 节点会落在 `.agents-tab-item` 容器内（继承父容器 x-offset），要做真正 full-bleed**必须用** `margin-left/right: calc(50% - 50vw)` bleed 技巧，**单独 `width: 100vw` 不够**。

`width: 100vw` 控制宽度数值，不控制元素 x 起始位置。在嵌套容器（agents-pin → pin-inner → stage → side → tabs-v → tab-item）内必须用 margin bleed 把元素拉回 viewport 0。

这也适用于：
- Mobile 上任何「需要 full-bleed」的图/视频/banner
- Bento card 的 cover image
- Modal 内部的 hero image

更新到 Notion 资料库 #5「Alva 产品 mobile 适配 12 大规律」的规律 #12 第 3 条小注。
