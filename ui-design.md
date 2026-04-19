# 斗罗大陆·武魂觉醒 — UI 设计文档 (v8)

> 单文件 HTML/CSS/JS · 移动端优先 · max-width 430px

---

## 目录

1. [设计语言](#设计语言)
2. [CSS 变量系统](#css-变量系统)
3. [布局结构](#布局结构)
4. [顶部状态栏](#顶部状态栏)
5. [底部导航栏](#底部导航栏)
6. [浮动背包按钮](#浮动背包按钮)
7. [通知系统](#通知系统)
8. [页面系统](#页面系统)
9. [武魂页（Soul Page）](#武魂页-soul-page)
10. [试炼页（Hunt Page）](#试炼页-hunt-page)
11. [星运页（Lottery Page）](#星运页-lottery-page)
12. [任务页（Tasks Page）](#任务页-tasks-page)
13. [世界探索页（Abyss Page）](#世界探索页-abyss-page)
14. [战斗界面（Combat Overlay）](#战斗界面-combat-overlay)
15. [武魂图鉴侧边栏（Grimoire Sidebar）](#武魂图鉴侧边栏-grimoire-sidebar)
16. [模态框系统](#模态框系统)
17. [觉醒界面（Awaken Screen）](#觉醒界面-awaken-screen)
18. [结果展示（Result Overlay）](#结果展示-result-overlay)
19. [粒子与星空背景](#粒子与星空背景)
20. [动画系统](#动画系统)
21. [品质主题色系](#品质主题色系)
22. [响应式与交互规范](#响应式与交互规范)

---

## 设计语言

### 整体风格

**暗黑东方幻想 · 玻璃态光效**

- 深邃星空背底（`#050810` → `#08101a`）
- 金色为主调（`#c9a227`），贯穿所有核心交互元素
- 毛玻璃效果（`backdrop-filter: blur`）用于通知气泡、浮层
- 内发光与外发光配合品质色，形成层次感
- 大量使用 radial-gradient 营造光晕
- 字体：`Ma Shan Zheng`（标题/武魂名）+ `Noto Sans SC`（正文）

### 核心设计原则

1. **克制信息密度**：折叠展开设计（技能、近期收获、抽取记录、概率详情）
2. **品质驱动视觉**：每个品质对应独立色彩体系，从背景光晕到边框颜色全套统一
3. **动效服务反馈**：粒子爆发、浮动动画、轨道旋转均与游戏事件绑定
4. **移动端优先**：`user-select: none`，`-webkit-tap-highlight-color: transparent`，`overflow-x: hidden`

---

## CSS 变量系统

```css
:root {
  /* 主题金色 */
  --gold:   #c9a227;
  --gl:     #f0d060;   /* 金亮 */
  --gd:     #8a6e1a;   /* 金暗 */

  /* 背景层次 */
  --bg:     #050810;   /* 最深底色 */
  --bgc:    #0f1628;   /* 卡片底色 */
  --bgc2:   #141c35;   /* 卡片次层 */

  /* 文字 */
  --txt:    #e8dfc0;   /* 主文字（暖白） */
  --dim:    #626870;   /* 灰淡文字 */

  /* 边框 */
  --bdr:    rgba(201,162,39,.2);   /* 标准边框 */
  --bdrB:   rgba(201,162,39,.52);  /* 强调边框 */

  /* 品质颜色 */
  --common: #9ca3af;
  --rare:   #3b82f6;
  --epic:   #8b5cf6;
  --legend: #f59e0b;
  --apex:   #ef4444;
  --hc:     #10b981;   /* 普通隐藏 */
  --ha:     #ec4899;   /* 顶级隐藏 */
  --twin:   #f0abfc;   /* 双生 */
  --triple: #e2e8f0;   /* 三生 */
  --divine: #ffd700;   /* 神级 */

  /* 魂环阶色（快速参考） */
  --r100:   #a78bfa;   /* 百年 */
  --r1k:    #60a5fa;   /* 千年 */
  --r10k:   #34d399;   /* 万年 */
  --r100k:  #fbbf24;   /* 十万年 */
  --r1m:    #f87171;   /* 百万年 */
  --rUnk:   #e879f9;   /* 不可估量 */
  --rGod:   #fffbeb;   /* 神赐 */
  --rCosmic:#00ffff;   /* 宇宙之核 */

  /* 药草绿 */
  --herb:   #22c55e;
}
```

动态 CSS 变量（由 JS 注入，随武魂品质变化）：

```css
--soul-col       /* 武魂主色 */
--soul-glow      /* 武魂光晕色 */
--qc-border      /* 品质边框色 rgba */
--qc-bg          /* 品质背景色浅 */
--qc-bg2         /* 品质背景色最浅 */
--qcR / --qcG / --qcB  /* 品质 RGB 分量，用于 calc */
```

---

## 布局结构

```
#app  (position:relative, max-width:430px, margin:auto)
├── #topbar      (flex-shrink:0, z-index:50)
├── #notif       (fixed, top:70px, right:10px, z-index:300)
├── #content     (flex:1, overflow-y:auto, padding-bottom:66px)
│   ├── #page-soul
│   ├── #page-hunt
│   ├── #page-lottery
│   ├── #page-tasks
│   ├── #page-abyss
│   └── #PF  (魂环融合子页)
└── #bnav        (fixed, bottom:0, z-index:50, height:58px)

全局固定层（z-index 排序）:
  #SB            z-index:0   星空背景
  canvas#C       z-index:0   粒子画布
  #soul-geo-cvs  z-index:0   武魂几何背景
  #bag-fab       z-index:100 浮动背包按钮
  #grimoire-sidebar z-index:400
  #combat-ov     z-index:400 战斗界面（动态插入）
  #modal         z-index:280
  #RSM           z-index:350 魂环选择模态
  #OR            z-index:250 觉醒结果
  #SA            z-index:200 觉醒界面
  #gm-panel      z-index:600 GM面板
  #bag-overlay   z-index:90  背包浮层
```

---

## 顶部状态栏

```
#topbar
├── .tb-row
│   ├── .tb-title        "斗罗觉醒"（GM长按触发区）
│   └── .tb-pills
│       ├── .pill.pow    ⚔️ 战力（红色主题）
│       ├── .pill        ⚡ 等级
│       └── .pill        💫 魂力
└── .exp-row
    ├── .exp-wrap > .exp-fill    经验进度条（金色渐变，0.6s ease过渡）
    └── .exp-lbl                 "当前EXP / 目标EXP"
```

**样式特点**：
- `background: linear-gradient(180deg, rgba(5,8,16,.98), rgba(10,15,30,.95))`
- 底部边框：`1px solid var(--bdr)`
- Pill 组件：20px 圆角，半透明金色背景，10px 字体

---

## 底部导航栏

5个导航项，等宽分布：

| 图标 | 标签 | 页面 ID |
|------|------|---------|
| ⚡ | 武魂 | soul |
| 🔮 | 试炼 | hunt |
| 🌟 | 星运 | lottery |
| 📋 | 任务 | tasks |
| ⚔️/🔒 | 世界 | abyss |

**激活态**：
- 标签色 → `var(--gl)`
- 图标 `scale(1.1)`
- 底部装饰线：`height:2px; background: linear-gradient(90deg, var(--gd), var(--gl))`

**世界页**：Lv.30 前显示🔒并 `opacity:.4`

**任务红点**：当有可领取任务时，动态插入 7px 红色圆点（`#ef4444`）

---

## 浮动背包按钮

```css
#bag-fab {
  position: fixed;
  bottom: 70px;
  right: 14px;
  width: 46px; height: 46px;
  border-radius: 50%;
  background: linear-gradient(135deg, rgba(201,162,39,.25), rgba(201,162,39,.1));
  border: 1.5px solid var(--bdrB);
  z-index: 100;
}
```

- 默认：🎒 图标
- 展开态（`.open`）：改为 ✕，背景变红色调，`box-shadow: 0 0 16px rgba(239,68,68,.3)`
- 点击展开：在 `#app` 末尾插入 `#bag-overlay`（`z-index:90`，`animation: sUp .25s`）

---

## 通知系统

```
#notif  (fixed, top:70px, right:10px, width:222px, z-index:300)
```

每条通知 `.ntf`：

- 形态：22px圆角气泡（长文本时变12px圆角 `.long`）
- 左侧圆点：5px 实心彩色（`var(--ntf-col)`）带发光
- 进场：`ntfIn .28s cubic-bezier(.34,1.56,.64,1)` （弹入+scale）
- 出场：`ntfOut .35s ease 2.95s`（淡出+右移）
- 最多同时显示 5 条

**5个通知等级**：

| class | 主色 | 边框 | 背景 | 使用场景 |
|-------|------|------|------|----------|
| `normal` | `var(--gl)` 75% | 金色 | 标准暗 | 一般信息 |
| `epic` | `#a78bfa` | 紫色 | 深蓝紫 | 史诗品质、魂骨掉落 |
| `legend` | `#f59e0b` | 橙金 | 深暗橙 | 传说品质、重要奖励 |
| `divine` | `#ffd700` | 金色强 | 深暗金 | 神级事件 |
| `cosmic` | `#00ffff` | 青色 | 深暗青 | 宇宙之核、图鉴完成 |

---

## 页面系统

```css
.page { display: none; padding: 11px; }
.page.active { display: block; }
#page-soul { padding: 0; }      /* 武魂页无外边距 */
#page-lottery { padding: 0; }   /* 星运页无外边距 */
#page-tasks { padding: 0; }     /* 任务页无外边距 */
```

页面切换无动画，直接 display 切换（性能优先）。

### 通用子组件

**Section Header `.sh`**：
```css
display: flex; align-items: center; justify-content: space-between;
margin-bottom: 10px;
```

**Section Title `.st`**：
```css
font-family: 'Ma Shan Zheng'; font-size: 14px; color: var(--gl);
display: flex; align-items: center; gap: 5px;
/* 左侧3px金色竖条 */
```

**Card `.card`**：
```css
background: linear-gradient(135deg, var(--bgc), var(--bgc2));
border: 1px solid var(--bdr); border-radius: 12px; padding: 12px;
/* 左上角隐式金光 via ::before */
```

---

## 武魂页 Soul Page

### 结构层次

```
#page-soul
├── .soul-v2-hero           (flex-col, align-center)
│   ├── .soul-orbit         轨道系统 (190×190px)
│   │   ├── .sol-ring ×3   三层旋转椭圆环（品质色）
│   │   ├── .sol-dot ×2    主轨道光点（0°/180°）
│   │   ├── .sol-dot.s2 ×2 副轨道光点（90°/270°，外轨）
│   │   ├── .sol-glow       品质色径向模糊光晕 (128×128)
│   │   └── .sol-icon       武魂图标（82px，浮动动画）
│   ├── .sol-name           武魂名（品质色，文字阴影）
│   ├── .sol-meta           品质标签 + 段位 + 状态
│   ├── .sol-title          称号显示（可点击切换）
│   ├── .sol-attrs          属性标签组（浅色胶囊）
│   ├── .sol-actions        4个快捷操作按钮
│   └── 经验条
├── .sv2-god-banner         (神级武魂专用，有则显示)
├── .sv2-card.accent        魂环卡（弧形轨道排列+列表）
├── .sv2-gear (2列)
│   ├── .sv2-gcol           魂骨（3×2 网格）
│   └── .sv2-gcol           神器（单图 + 信息）
├── .sv2-resonance          (神骨共鸣提示，有则显示)
├── .sv2-card               魂技卡（前3个）
└── .sv2-bottom-row         图鉴按钮 + 再次觉醒按钮
```

### 轨道系统动画

```css
/* 三层旋转环 */
.sol-ring.r1 { animation: solSpin 14s linear infinite; }
.sol-ring.r2 { animation: solSpin 26s linear infinite reverse; }
.sol-ring.r3 { animation: solSpin 38s linear infinite; border-style: dashed; }

/* 轨道光点绕行 */
@keyframes solDotOrbit {
  0%   { transform: rotate(var(--s,0deg)) translateX(80px) translateY(-2.5px); }
  100% { transform: rotate(calc(var(--s,0deg) + 360deg)) translateX(80px) translateY(-2.5px); }
}

/* 武魂图标浮动 */
@keyframes solIconFloat {
  0%,100% { transform: translateY(0); }
  50%     { transform: translateY(-7px); }
}
```

### 魂环弧形展示

10个魂环按弧形排列，arcPositions 数组定义各环 left/bottom 偏移：
```
{l:0,b:0} {l:32,b:16} {l:64,b:30} {l:97,b:40} {l:130,b:44}
{l:163,b:40} {l:195,b:30} {l:226,b:16} {l:256,b:4} {l:285,b:0}
```

每个 `.sv2-r-orb`：28×28px 圆形，动态品质色边框+背景+阴影。

### 快捷操作按钮 `.sol-act`

4个等宽按钮，激活状态（`.hi`）使用动态品质边框色：
```css
.sol-act.hi {
  border-color: var(--qc-border);
  background: var(--qc-bg);
}
```

### 几何背景画布

`#soul-geo-canvas`（fixed, inset:0, z-index:0）绘制：
- 顶部径向光晕（品质色，opacity约9%）
- 左下副光晕（5%）
- 2~6层旋转几何环（依品质分级，`geo` 参数控制层数）

---

## 试炼页 Hunt Page

### 2×2 紧凑区域网格

```css
display: grid; grid-template-columns: 1fr 1fr; gap: 8px;
```

4种区域卡片（`.zone-card`）：
- `forest`：绿色边框 `rgba(16,185,129,.28)`
- `chaos`：紫色边框 `rgba(139,92,246,.32)`
- `primordial`：青色边框 `rgba(6,182,212,.28)`
- `random`：金色边框 `rgba(201,162,39,.28)`

每个卡片：22px图标 + 14px标题 + 标签组 + 费用显示。

### 成神之路卡片（全宽）

```css
.zone-card.godpath {
  background: linear-gradient(135deg, rgba(80,0,0,.95), rgba(50,0,0,.98));
  border: 1px solid rgba(239,68,68,.5);
  box-shadow: 0 0 20px rgba(239,68,68,.15);
}
.zone-card.godpath.locked { opacity: .5; filter: grayscale(.6); }
```

### 特殊道路 2×2 网格

```css
.path-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
```

`.pcard`：标准卡片，解锁时使用路径主题色边框，锁定时灰化。

### 近期收获（折叠）

`.ring-item`：横向排列，圆形彩色首字缩写 + 名称/技能/战力。

---

## 星运页 Lottery Page

### Banner 轮播（Swipe Banner）

```
.lot-banner-outer (overflow:hidden, height:calc(48vh - 56px), min-height:220px)
└── .lot-banner-track (display:flex, width:300%, transition:transform .38s)
    ├── .lot-slide.ls-0  普通池（灰调背景）
    ├── .lot-slide.ls-1  高级池（紫调背景）
    └── .lot-slide.ls-2  顶级池（橙调背景）
```

每个 Slide 包含：
- `canvas.lot-geo-cvs`：动态几何背景（轨道环系统，各池独立配置）
- `.lot-ltd`：限时标签（红色徽章，绝对定位右上角）
- `.lot-inner`：内容层（顶部信息 + 特色奖品卡片）

**滑动交互**：
- Touch：touchstart/move/end 控制轨道 translateX
- Mouse：mousedown/move/up（桌面端支持）
- 超过 40px 阈值切换池

**底部指示点**：
```css
.ldot { width:20px; height:3px; border-radius:2px; }
.ldot.active { width:28px; background: var(--lpc,#9ca3af); box-shadow: 0 0 6px; }
```

### 抽取面板（.lot-panel）

```
.lot-draw-grid (2列)
├── 单次抽取按钮（显示券数量徽章）
└── 五连抽取按钮（显示券数量徽章）

.lot-ten（全宽十连按钮）
├── 左：图标 + 名称 + 子标题
└── 右：费用 + 券注释
```

**十连按钮**：
```css
border: 1.5px solid rgba(239,68,68,.55);
background: linear-gradient(135deg, rgba(239,68,68,.1), rgba(245,158,11,.07), rgba(139,92,246,.05));
box-shadow: 0 0 16px rgba(239,68,68,.12);
```
Shimmer动画：`ltShine 2.8s ease infinite`（伪元素白光横扫）

### 券徽章 `.ldg-tix-badge`

```css
font-size: 8px; font-weight: 700; padding: 2px 6px; border-radius: 10px;
border: 1px solid; background: rgba(0,0,0,.3);
```
颜色动态绑定奖池主色。

### 概率详情（折叠）

`.lot-rates`：默认收起，点击 `.lr-head` 展开 `.lr-body`。

### 抽取记录

`.lh-item`：传说命中时使用金色边框/背景，宇宙之核使用青色边框/背景。

---

## 任务页 Tasks Page

### 魂力修炼 Widget（.cult-widget）

```css
background: linear-gradient(135deg, rgba(16,185,129,.07), rgba(16,185,129,.025));
border: 1px solid rgba(16,185,129,.18);
border-radius: 16px;
```

内含几何背景画布（`#cw-geo`），绿色轨道环系统。

两条进度条：
- 修炼（绿色渐变，cult-f）
- 探索（蓝色渐变，expl-f）

两个操作按钮（2×1网格）：
```css
.cw-btn.cult-b { border-color: rgba(16,185,129,.28); }
.cw-btn.expl-b { border-color: rgba(59,130,246,.25); }
```

### 踏春欧皇 Banner（.sb-banner）

```css
background: linear-gradient(135deg, rgba(255,192,203,.08), rgba(201,162,39,.05), rgba(255,192,203,.04));
border: 1px solid rgba(255,192,203,.25);
border-radius: 16px;
```

- 左侧：脉冲圆点 + 标签 + 名称 + 描述
- 右侧：🏆图标（浮动动画）+ 完成次数 + 展开箭头
- 展开后显示5个子任务详情（`.sb-detail`）
- 几何背景：粉色轨道环（`#sb-geo`，sbDotPulse 脉冲）

### 任务卡片（.tc）

```
.tc（默认灰色）
.tc.claimable（金色边框 + Shimmer动画）
.tc.claimed（opacity:.42）
.tc.hidden（紫色边框）

内部结构：
├── .tc-ico-w   图标容器（38×38，动态主色边框）
├── .tc-body    名称 + 描述 + 进度条 + 券标签
└── .tc-right   奖励魂力 + 领取按钮

进度条：
  .tc-pb { height: 3px; background: rgba(255,255,255,.05); }
  .tc-pf { height: 100%; transition: width .5s ease; }
  /* claimable时：金色渐变 */
  /* hidden任务：紫色渐变 */
```

---

## 世界探索页 Abyss Page

### Tab 栏

两个标签：`⚔️ 异界副本` / `🌐 更多世界`

激活态动态更新 border-color / background / color / font-weight。

### 大卡片（.world-card-big）

```css
border-radius: 16px; overflow: hidden;
min-height: 200px;
display: flex; flex-direction: column; justify-content: flex-end;
```

每个副本层：
- 深色多层渐变背景（各层独立色调）
- 径向光晕叠加（品质色）
- 右上角大图标（52px，低不透明度装饰）
- 底部内容区：标题 + 标签组 + 进度条 + 进入按钮

### 进度条样式

```css
height: 4px; background: rgba(255,255,255,.08);
overflow: hidden; border-radius: 2px;
/* fill 使用层级主题色 */
```

### 第二页（更多世界）

展示「敬请期待」占位卡片，保持视觉一致性，深灰调背景，降饱和度图标。

---

## 战斗界面 Combat Overlay

动态插入 `#combat-ov`（`fixed, inset:0, z-index:400`）。

### 布局（flex-column）

```
顶部状态栏（副本名 + 回合 + 速度切换 + 撤退）
↓
双方面板区（固定高度，flex-shrink:0）
  ├── 敌方（图标 + 名称 + HP条 + 阶段 + 状态）
  ├── ⚔️ 分隔
  └── 玩家（图标 + 名称 + HP条 + MP条 + 状态）
↓
战斗日志（flex:1, overflow-y:auto, 最近5条）
↓
操作区（flex-shrink:0）
  ├── 进行中：普攻 + 防御 + 4技能
  └── 已结束：离开按钮
```

**HP条**：
- 玩家：绿色渐变 `#16a34a → #22c55e`
- 敌方：红色渐变 `#dc2626 → #ef4444`
- 高度 8px，圆角 4px，过渡 0.4s ease

**MP条**：高度 4px，蓝紫渐变 `#6366f1 → #818cf8`

**技能按钮**：CD中灰色禁用，可用时紫色激活，MP不足同样禁用

---

## 武魂图鉴侧边栏 Grimoire Sidebar

```css
#grimoire-sidebar {
  position: fixed; inset: 0; z-index: 400;
  pointer-events: none;  /* 关闭时穿透 */
}
#grimoire-sidebar.open { pointer-events: all; }

.gs-panel {
  position: absolute; top:0; left:0; bottom:0;
  width: 88%; max-width: 360px;
  transform: translateX(-100%);
  transition: transform .32s cubic-bezier(.4,0,.2,1);
}
#grimoire-sidebar.open .gs-panel { transform: translateX(0); }
```

**遮罩**：`.gs-backdrop`，`background: rgba(0,0,0,0)`，打开后过渡到 `rgba(0,0,0,.75)`

### 头部（.gs-hd）

- 紫色渐变背景 + 几何画布（`#gs-hd-cvs`，3层紫色轨道环）
- 统计：已发现 / 全101种 / 完成度%
- 脉冲圆点动画：`gsDot 2.2s ease infinite`

### 筛选器（.gs-filters）

7个胶囊标签，每个绑定对应品质的 `--gfr/gfg/gfb/gfc` 变量。

### 武魂网格（.gs-grid）

```css
display: grid; grid-template-columns: repeat(3, 1fr); gap: 7px;
```

每格 `.gc`：
- 已发现（`.found`）：品质色边框 + 微妙背景 + Shimmer动画 + 角标✓/★
- 未发现（`.unfound`）：`filter: grayscale(1); opacity: .25`

### 详情底托（.gs-detail）

```css
position: absolute; bottom: 0; left:0; right:0;
transform: translateY(100%);
transition: transform .3s cubic-bezier(.4,0,.2,1);
```
上滑展开，下滑/托底关闭。

---

## 模态框系统

### 通用底部弹窗（#modal）

```css
position: fixed; inset: 0; z-index: 280;
background: rgba(0,0,0,.86);
display: none; align-items: flex-end; justify-content: center;
```

`.msheet`（内容区）：
```css
background: linear-gradient(180deg, #0e1525, #080d1a);
border-top: 1px solid var(--bdr);
border-radius: 18px 18px 0 0;
max-height: 82vh; overflow-y: auto;
animation: sUp .28s ease;
```

`@keyframes sUp`：`translateY(100%) → translateY(0)`

**点击背景关闭**，内部点击不冒泡。

### 魂环选择模态（#RSM）

与 #modal 同结构，`z-index:350`，max-height:72vh。

### GM 面板（#gm-panel）

```css
position: fixed; inset: 0; z-index: 600;
background: rgba(0,0,0,.97);
display: none; flex-direction: column;
```

按功能区块分组，使用统一按钮样式：
- `.gm-btn-g`：金色（金色操作）
- `.gm-btn-r`：红色（危险操作）
- `.gm-btn-b`：蓝色（数值设置）
- `.gm-btn-p`：紫色（物品操作）

---

## 觉醒界面 Awaken Screen

```css
#SA {
  position: fixed; inset: 0; z-index: 200;
  background: radial-gradient(ellipse at 50% 38%, rgba(18,26,50,.98), #050810);
  display: flex; flex-direction: column; align-items: center; justify-content: center;
}
```

主元素：
- `.aw-glow`：脉冲光晕球（`awG 3s ease infinite`，scale 1→1.18）
- `.aw-c`：195×195 圆形按钮，三层旋转环（ar1/ar2/ar3，速度9s/14s/21s）
- `.aw-ico`：70px emoji，可替换图标
- `.aw-hint`：闪烁提示文字（`blk 1.8s infinite`）
- `.aw-btn`：大按钮，hover时金色外发光

---

## 结果展示 Result Overlay

```css
#OR {
  position: fixed; inset: 0; z-index: 250;
  background: rgba(0,0,0,.91);
  display: none; flex-direction: column; align-items: center; justify-content: center;
}
```

内容（层叠于径向背景光晕之上）：
- `.or-ico`：86px武魂图标，`flt 2s ease infinite`（上下浮动-8px）
- `.or-q`：品质名称（品质色）
- `.or-nm`：武魂大名（34px，品质色）
- `.or-stats`：3个小卡片（初始魂力 / 属性 / 品质）
- `.or-tap`：点击提示（闪烁）

---

## 粒子与星空背景

### 星空（#SB）

50个 `.star` div，随机分布：
- 尺寸：0.3~2.3px
- 动画：`twk var(--d) ease infinite var(--dl)`（透明度周期变化）
- 变量：`--d`（时长2~5s），`--dl`（延迟-5~0s），`--mn/--mx`（亮度范围）

### 粒子（canvas#C）

`spawnBurst(col, n)` 函数：
- 从视窗中心上方（`width/2, height×0.42`）爆发
- 每粒子：随机方向速度、生命值1→0、重力+0.055/帧
- 衰减：0.018~0.042/帧
- 大小：0.8~4.8px

---

## 动画系统

| 关键帧名 | 用途 | 时长 |
|----------|------|------|
| `twk` | 星星闪烁 | 2~5s |
| `rot` | 觉醒界面旋转环 | 9/14/21s |
| `awG` | 觉醒光晕脉冲 | 3s |
| `blk` | 闪烁文字 | 1.5~1.8s |
| `flt` | 结果图标浮动 | 2s |
| `bP` | 结果背景脉冲 | 0.9s |
| `ntfIn` | 通知进场 | 0.28s |
| `ntfOut` | 通知退场 | 0.35s |
| `sUp` | 模态框上弹 | 0.28s |
| `solSpin` | 武魂轨道环旋转 | 14/26/38s |
| `solDotOrbit` | 武魂光点绕行 | 14s（内轨）/ 23s（外轨） |
| `solGlowPulse` | 武魂中心光晕 | 3.2s |
| `solIconFloat` | 武魂图标浮动 | 3.5s |
| `lotCardPop` | 十连卡片弹出 | 0.5s cubic-bezier(.34,1.56,.64,1) |
| `ltShine` | 十连按钮光扫 | 2.8s |
| `ltpShine` | 抽取按钮光扫 | 4~6s |
| `tcShimmer` | 可领取任务光扫 | 2.4s |
| `sbDotPulse` | 踏春活动脉冲 | 2s |
| `sbIcoFloat` | 踏春图标浮动 | 3s |
| `gcShimmer` | 图鉴格子光扫 | 5~11s |
| `gsDot` | 图鉴脉冲点 | 2.2s |
| `qtDotPulse` | 武魂品质标签圆点 | 2.4s |

---

## 品质主题色系

武魂页整体颜色随武魂品质动态切换，通过 JS 设置 CSS 变量实现：

| 品质 | 主色 | 光晕色 | 几何环层数 |
|------|------|--------|-----------|
| common | `#9ca3af` | `rgba(156,163,175,.3)` | 2 |
| rare | `#3b82f6` | `rgba(59,130,246,.35)` | 3 |
| epic | `#8b5cf6` | `rgba(139,92,246,.4)` | 3 |
| legend | `#f59e0b` | `rgba(245,158,11,.4)` | 4 |
| apex | `#ef4444` | `rgba(239,68,68,.45)` | 5 |
| hc | `#10b981` | `rgba(16,185,129,.4)` | 3 |
| ha | `#ec4899` | `rgba(236,72,153,.45)` | 4 |
| twin | `#f0abfc` | `rgba(240,171,252,.5)` | 5 |
| triple | `#e2e8f0` | `rgba(226,232,240,.5)` | 6 |

---

## 响应式与交互规范

### 视口约束

```css
html, body { width: 100%; height: 100%; overflow: hidden; }
#app { max-width: 430px; margin: 0 auto; }
```

所有固定定位元素同步 `max-width: 430px; left: 50%; transform: translateX(-50%)`。

### 触摸规范

- `user-select: none`（全局）
- `-webkit-tap-highlight-color: transparent`（全局）
- `-webkit-overflow-scrolling: touch`（所有滚动容器）
- 滚动条隐藏：`::-webkit-scrollbar { display: none }`
- 点击反馈：`:active { transform: scale(.97) }` 标准缩放

### 滚动区域

- `#content`：主内容滚动区（`overflow-y: auto`，底部 66px 留白）
- `.msheet`：模态框内容滚动
- `.gs-body`：侧边栏主体滚动
- `.gs-detail`：详情区滚动

### 字体缩放层级

| 用途 | 字体族 | 尺寸 |
|------|--------|------|
| 主标题/武魂名 | Ma Shan Zheng | 19~34px |
| 页面标题 | Ma Shan Zheng | 14~22px |
| 正文内容 | Noto Sans SC | 11~12px |
| 标签/说明 | Noto Sans SC | 9~10px |
| 超小标注 | Noto Sans SC | 7~8px |
