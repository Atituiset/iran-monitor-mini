**整体骨架**

这个页面的 HTML 布局是很典型的“**一个固定头图 Hero + 一列内容卡片 Section**”结构，主容器在 [index.html:826](/home/atituiset/Projects/iran-monitor-mini/index.html#L826)：

```html
<main class="page">
```

`main.page` 下面按顺序放了 6 个大区块：

1. 顶部 Hero 区：[index.html:827](/home/atituiset/Projects/iran-monitor-mini/index.html#L827)
2. 资产价格跟踪：[index.html:845](/home/atituiset/Projects/iran-monitor-mini/index.html#L845)
3. 预测市场跟踪：[index.html:854](/home/atituiset/Projects/iran-monitor-mini/index.html#L854)
4. BCA 内容流：[index.html:974](/home/atituiset/Projects/iran-monitor-mini/index.html#L974)
5. 航运观察 iframe：[index.html:993](/home/atituiset/Projects/iran-monitor-mini/index.html#L993)
6. 推荐链接列表：[index.html:1019](/home/atituiset/Projects/iran-monitor-mini/index.html#L1019)

也就是说它不是那种复杂的左右栏后台布局，而是更偏**专题页 / dashboard landing page** 的思路：  
**上面先给结论和状态，下面一块块展开不同数据模块。**

---

**布局主线**

这份页面最值得你学的，是它把“视觉统一”和“内容模块化”分开处理了。

先看总宽度控制，在 [index.html:82](/home/atituiset/Projects/iran-monitor-mini/index.html#L82)：

```css
.page {
  width: min(1180px, calc(100% - 32px));
  margin: 0 auto;
}
```

意思是：

- 页面最大宽度限制在 `1180px`
- 小屏时保留左右边距 `16px + 16px`
- `margin: 0 auto` 让它始终居中

这是很实用的专题页写法，解决了“大屏别太散，小屏别贴边”。

然后顶部 `hero` 不是普通流里的块，而是**固定定位**，在 [index.html:89](/home/atituiset/Projects/iran-monitor-mini/index.html#L89)：

```css
.hero {
  position: fixed;
  top: 16px;
  left: 50%;
  transform: translateX(-50%);
}
```

这说明设计目标是：**让顶部状态卡一直悬浮在页面上方，不随着内容滚走**。  
为了防止下面内容被固定头遮住，`.page` 又用了：

```css
padding: calc(var(--hero-offset, 0px) + 24px) 0 56px;
```

在 [index.html:85](/home/atituiset/Projects/iran-monitor-mini/index.html#L85)。  
也就是用 JS 动态测量 hero 高度，再把正文往下顶开。这是一个很常见、也很实战的做法。

---

**模块设计**

除了 `hero`，下面几乎所有内容区都复用了 `.panel` 这个“卡片基类”，定义在 [index.html:263](/home/atituiset/Projects/iran-monitor-mini/index.html#L263)：

```css
.panel {
  padding: 24px;
  border: 1px solid var(--line);
  border-radius: 24px;
  background: var(--panel);
}
```

这说明作者在布局上用了一个非常清晰的策略：

- 每个大块都是一个卡片
- 卡片之间通过 `margin-top` 拉开
- 卡片内部再根据内容类型决定是 grid、flex、list 还是 iframe

这比每个 section 都单独从头写一套样式更稳定，也更容易扩展。

你可以把它理解成：

- `.page` 负责“页面级宽度和整体节奏”
- `.panel` 负责“模块级容器外观”
- 子类如 `.embed-card`、`.macro-assets-grid`、`.feed-list` 负责“模块内部排版”

这个分层很值得学。

---

**几个关键区域怎么排的**

**1. Hero 顶部区**

Hero 的 HTML 很紧凑，在 [index.html:827](/home/atituiset/Projects/iran-monitor-mini/index.html#L827) 到 [index.html:843](/home/atituiset/Projects/iran-monitor-mini/index.html#L843)：

- 第一行 `hero-topline`
- 第二行 `h1`
- 第三行说明文字
- 第四行作者入口

其中第一行在 [index.html:138](/home/atituiset/Projects/iran-monitor-mini/index.html#L138) 用的是 `flex`：

```css
.hero-topline {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
```

左边是一组状态信息 `hero-meta`，右边是主题切换按钮。  
这类“左信息 + 右操作”的头部结构，`flex` 是最合适的。

`hero-meta` 自己又是一个 `inline-flex`，里面放：

- `eyebrow` 小标签
- `hero-status-line` 状态句

这说明作者在做的是**小组件拼装式布局**，不是全靠一个大容器硬排。

---

**2. 资产价格区**

资产区最核心的是 [index.html:467](/home/atituiset/Projects/iran-monitor-mini/index.html#L467)：

```css
.macro-assets-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 10px;
}
```

这是一种非常典型的**卡片宫格布局**：

- 一行 4 列
- 每列等宽
- `minmax(0, 1fr)` 防止内容把 grid 撑爆

里面每个资产卡 `.macro-asset-card` 都是统一的小卡片。  
这里选择 `grid` 而不是 `flex` 很对，因为它强调的是“二维整齐排列”。

---

**3. 预测市场区**

这个区也是 `panel`，但额外加了 `.embed-card`，在 [index.html:339](/home/atituiset/Projects/iran-monitor-mini/index.html#L339)：

```css
.embed-card {
  padding: 0;
  overflow: hidden;
}
```

意思是这个卡片内部要装 iframe/embed，所以不想让外层统一 `padding` 干扰内容边缘。

它内部又拆成两层：

- 顶部 `embed-head` 用 `flex` 做标题和外链
- 中间 `polymarket-wrap` 用 `grid` 两列放两个 iframe

对应 [index.html:383](/home/atituiset/Projects/iran-monitor-mini/index.html#L383)：

```css
.polymarket-wrap {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
}
```

这就是一个很标准的思路：

- 顶部操作栏用 `flex`
- 主体并列内容用 `grid`

下面的 `polymarket-grid` 也是同样套路，用于展示多个分组表格。

---

**4. BCA 内容流**

这块布局更偏“信息流”，核心不是多列，而是纵向阅读节奏。

头部 [index.html:307](/home/atituiset/Projects/iran-monitor-mini/index.html#L307) 的 `feed-head` 也是 `flex`：

```css
.feed-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
}
```

左边标题和说明，右边状态 badge。  
而正文 [index.html:573](/home/atituiset/Projects/iran-monitor-mini/index.html#L573) 的 `feed-list` 用的是：

```css
.feed-list {
  display: grid;
  gap: 12px;
}
```

注意这里只把 `grid` 当成“带统一间距的垂直堆叠容器”来用，不一定非要多列。这个用法很常见，也很好用。

每条 `feed-item` 自身不是 flex，而是通过：

- 左边一个编号 `.index`
- 右边一个 `inline-block` 文本区 `.text`

来排内容。  
这不是最现代的写法，但能看出作者想保留一种“序号 + 正文”的阅读结构。

---

**5. 航运观察和推荐区**

航运观察本质上是“标题栏 + iframe 内容 + 来源”，所以沿用了 `embed-card` 模式。  
推荐区则是最简单的列表，在 [index.html:523](/home/atituiset/Projects/iran-monitor-mini/index.html#L523)：

```css
.recommend-list {
  list-style: none;
  display: grid;
  gap: 10px;
}
```

这里也是把 `ul` 当成一个纵向卡片栈来处理。

---

**响应式设计**

这份页面响应式写得比较朴素，但思路是对的，集中在 [index.html:652](/home/atituiset/Projects/iran-monitor-mini/index.html#L652) 的 `@media (max-width: 900px)`。

主要做了几件事：

- Hero 顶部从横向分散改成更自然的换行
- `stat-wrap` 变 1 列
- `polymarket-wrap` 从 2 列变 1 列
- `polymarket-grid` 从 2 列变 1 列
- `macro-assets-grid` 从 4 列变 2 列
- `feed-head` 改成竖排

也就是说这套响应式不是彻底重构，而是遵循一个简单原则：

**桌面端多列，小屏端收成单列或双列。**

这正是 dashboard 页面最常见也最稳妥的做法。

---

**你可以重点学的设计思路**

这份代码里最值得你吸收的不是某一条 CSS，而是下面这几个习惯：

- **先搭页面骨架，再做模块基类。** `page` 和 `panel` 的分工很清楚。
- **不同内容类型，用不同布局工具。** 头部用 `flex`，卡片矩阵用 `grid`，长文列表用纵向堆叠。
- **统一视觉 token。** 颜色、阴影、边框几乎都走 CSS 变量，这让主题切换和整体风格统一变得很容易。
- **固定 Hero + 动态补偿高度。** 这比拍脑袋写死 `padding-top` 更稳。
- **把“外部嵌入内容”单独建一个卡片变体。** `.embed-card` 就是为 iframe 这种内容准备的。
- **响应式只改结构，不改语义。** HTML 不变，通过媒体查询收缩列数，这样维护成本低。

---

**如果你要模仿这类页面，可以照这个顺序写**

1. 先写 `main.page`
2. 做一个通用 `.panel`
3. 把每个功能区都当作独立 `section`
4. 区块头统一成“标题 + 描述/按钮”
5. 区块内部再决定用 `flex` 还是 `grid`
6. 最后补移动端列数变化

如果你愿意，我下一步可以继续做两种讲法里的任意一种：

1. 按“从上到下”逐段带你读这个 `index.html`
2. 单独讲这份页面里 `flex`、`grid`、`position: fixed` 分别是怎么配合的