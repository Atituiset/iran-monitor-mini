# CSS 变量 + JS 动态布局技法：让内容自动避开 Fixed 元素

## 问题背景

页面顶部有一个 `position: fixed` 的 hero 区域，它的高度**会随着内容换行而变化**（比如标题文本用 `clamp()` 响应式字号）：

```html
<header class="hero" style="position: fixed; top: 16px;">
  <h1 style="font-size: clamp(34px, 5vw, 62px);">美伊冲突主题跟踪页</h1>
  <!-- 标题在窄屏会变成两行，高度增加 -->
</header>

<main class="page">
  <!-- 这里的内容需要躲开 hero，不能被遮挡 -->
</main>
```

问题是：**hero 高度不固定，内容区怎么知道该留多少空白？**

---

## 解决方案：CSS 变量桥接法

### 核心思路

> **CSS 声明"我需要偏移 X 像素"，JS 计算"X 是多少"，然后把数字注进去。**

这样 CSS 负责布局规则，JS 负责测量数值，职责分离。

### Step 1 — CSS 声明变量和计算规则

```css
.page {
  /* 使用 CSS 变量，默认 0px，再加上固定的基础值 24px */
  padding: calc(var(--hero-offset, 0px) + 24px) 0 56px;
}
```

解释：
- `var(--hero-offset, 0px)` — 读取名为 `--hero-offset` 的 CSS 变量，变量不存在时默认 `0px`
- `calc(... + 24px)` — 变量值会由 JS 注入，CSS 只需要声明"我要在它的基础上加 24px"
- 左右 padding 为 0，底部 56px（固定值）

### Step 2 — JS 测量并注入变量值

```js
function syncHeroOffset() {
  const hero = document.querySelector(".hero");
  const topOffset = 16; // hero 距视口顶部的固定距离

  // 将计算结果注入到根元素的 CSS 变量
  document.documentElement.style.setProperty(
    "--hero-offset",
    (hero.offsetHeight + topOffset) + "px"
  );
}
```

`hero.offsetHeight` 返回 hero 元素的**实际渲染高度**（包括内容换行后撑开的高度）。

### Step 3 — 触发时机

```js
// 1. 页面加载时立即计算一次
syncHeroOffset();

// 2. 窗口大小变化时重新计算（响应式适配）
window.addEventListener("resize", syncHeroOffset);
```

---

## 完整流程图

```
页面加载
    │
    ├─► JS: 读取 .hero 的实际高度 (假设 100px)
    │
    ├─► JS: 计算 100 + 16 = 116
    │
    ├─► JS: document.documentElement.style.setProperty("--hero-offset", "116px")
    │          ↓
    │       <html style="--hero-offset: 116px;">
    │
    ├─► CSS: padding-top = calc(116px + 24px) = 140px
    │
    └─► 渲染: .page 的内容自然垂在 hero 下方 ✓
```

---

## 为什么不用其他方案？

| 方案 | 问题 |
|------|------|
| 写死 `padding-top: 140px` | hero 换行时高度变了，内容被遮挡或空白太多 |
| JS 直接改 `.page.style.paddingTop = "140px"` | 样式逻辑全堆在 JS 里，CSS 失去声明能力 |
| `margin-top` on `.page` | fixed 元素不影响文档流，margin 无法推开它 |

**CSS 变量 + JS 注入**是最优解：CSS 保持声明式，JS 只提供运行时才知道的数值。

---

## 变体：多场景复用

同一个 CSS 变量可以用于多处，只需要 JS 算一次：

```css
.hero {
  /* hero 自身也引用这个变量，保持和 .page 的间距一致 */
  margin-bottom: var(--hero-offset, 0px);
}

.page {
  /* 依赖同一个变量 */
  padding-top: calc(var(--hero-offset, 0px) + 24px);
}
```

---

## 关键语法回顾

### CSS 变量
```css
/* 声明 */
:root {
  --my-color: #ffc402;
}

/* 使用，默认值写法 */
var(--my-color, #000000)
```

### JS 注入
```js
// 读取
const value = getComputedStyle(document.documentElement).getPropertyValue('--my-color');

// 写入
document.documentElement.style.setProperty('--my-color', 'red');
```

### calc() 配合变量
```css
/* 变量和固定值一起参与计算 */
padding: calc(var(--offset, 0px) + 24px);
```
