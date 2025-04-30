# 防御式 CSS 课程 - 第二章：视觉盒模型选择

## 什么是视觉盒模型？

视觉盒模型也被称为**视觉格式化模型**，它会根据 CSS 盒模型将 HTML 文档中的元素转换为一个个盒子。它会根据盒子的包含块（Container Block）的边界来渲染盒子。它也是我们常说的格式化上下文（Formatting Context），比如大家常听到的 BFC、IFC 等。

在 CSS 中，任何一个盒子有两种模型：

- **盒模型（Box Model）**：主要用来计算盒子大小，包括 CSS 的`width`、`height`、`border`、`padding`和`margin`
- **视觉盒模型（Visual Formatting Model）**：主要用来计算盒子位置，即用于布局。由盒子的尺寸、类型、定位方案、文档树中的其它元素、浏览器视窗尺寸与位置、所包含的图片尺寸等因素决定

## 为什么要选择合理的视觉盒模型？

选择合理的视觉盒模型对于防御式 CSS 至关重要。不合理的选择会导致：

1. CSS 代码冗余，增加维护成本
2. 布局在不同环境下容易崩溃
3. 界面难以适应不同内容长度和屏幕尺寸

以常见的按钮为例，同样是"Sign Up"按钮，在不同场景下可能需要不同的视觉表现：

- 内联式按钮（宽度由内容决定）
- 块级按钮（宽度填满容器）

## HTML 元素与视觉盒模型的关系

HTML 中的标签元素主要分为：

- **块元素**：如`<div>`、`<p>`和`<ul>`等
- **内联元素**：如`<span>`、`<a>`和`<strong>`等
- **替换元素**：如`<img>`

这些不同类型的 HTML 元素都有着自身默认的视觉盒模型格式：

- 块元素对应**块格式化上下文（Block Formatting Context）**
- 内联元素对应**行内格式化上下文（Inline Formatting Context）**

简单来说：

- 块元素的默认宽度会填满其包含块，即`width: 100%`
- 内联元素的默认宽度和其内容有关，相当于`max-content`

## 如何改变视觉盒模型的格式？

在 CSS 中，可以通过`display`属性来手动调整盒子类型（视觉盒模型）。`display`属性提供了多种值类型，最常用的包括：

- `none`：从盒子树中移除元素及其所有后代
- `block`：正常流内的块级盒子
- `inline`：正常流内的内联级盒子
- `inline-block`：内联块级盒子
- `flex`：块级弹性盒子容器
- `inline-flex`：内联级弹性盒子容器

值得注意的是，`display`属性未来会采用双值语法，例如：

```css
.container {
  display: flex;

  /* 等同于 */
  display: block flex;
}

.container {
  display: inline-flex;

  /* 等同于 */
  display: inline flex;
}
```

这种双值语法可以帮助我们更好地理解`display`属性：第一个值定义元素的外部显示类型（块级或内联），第二个值定义元素的内部布局（flow、flex 等）。

## 防御式选择视觉盒模型的实践

根据 UI 形式选择合适的视觉盒模型，可以遵循以下步骤：

1. 分析 UI 元素在文档流中的表现形式（是块级还是内联）
2. 选择合适的 HTML 元素（优先考虑语义化）
3. 根据需要使用`display`属性调整视觉盒模型

### 实例分析：按钮组件

以注册表单中的按钮为例：

- 内联式按钮（宽度由内容决定）：

  ```css
  .button--inline {
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }
  ```

- 块级按钮（宽度填满容器）：
  ```css
  .button--block {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  ```

使用 CSS 变量可以让代码更加灵活：

```css
.button {
  display: var(--button-display, inline-flex);
  align-items: center;
  justify-content: center;
}

.button--block {
  --button-display: flex;
}
```

# 防御式 CSS 课程 - 第三章：Flexbox 中的换行机制

## Flexbox 布局中的换行问题

CSS Flexbox 已然成为当前最受欢迎的布局技术之一。只需要在容器元素上显式设置`display`属性的值为`flex`或`inline-flex`，其子元素（包括伪元素`::before`和`::after`以及匿名元素）会自动沿着 Flex 容器的主轴方向排成一行（或列）。

然而，在实际使用中，经常会遇到这样的情况：**Flex 容器无法容纳所有的 Flex 项目**。这可能由以下原因造成：

- **内容增加**：原本设计稿模板提供的内容（Flex 项目）是三个，但服务端实际输出的内容可能比三个多
- **视窗变小**：Web 页面会在不同终端设备上呈现，当终端设备的浏览器视窗宽度变窄时，也会造成 Flex 容器没有足够空间

此时，它将会直接打破 Web 布局：**内容（Flex 项目）溢出容器或容器出现水平滚动条**。

![Flex项目溢出示例](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db80d9d0d58c4a8aa8e7ed5043de167b~tplv-k3u1fbpfcp-zoom-1.image)

## 防御性解决方案

在使用 CSS Flexbox 构建 Web 布局时，一旦将容器声明为 Flex 容器之后，其所有 Flex 项目就会排列成一行（或列），即使容器没有足够空间时，Flex 项目也不会自动换行。

这种行为是一种预知的行为，并不是渲染问题，只是与我们预期的效果不同。要解决这个问题，方法非常简单：

**只需要在 Flexbox 容器上显式设置`flex-wrap`的值为`wrap`或`wrap-reverse`**（其默认值为`nowrap`）

```css
.card__content {
  display: flex;
  flex-wrap: wrap;
}
```

![添加flex-wrap后的效果](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/279886278b8c462fb7f7764206ca1fb2~tplv-k3u1fbpfcp-zoom-1.image)

因此，**在使用 CSS Flexbox 构建布局时，应该尽量在 Flexbox 容器上设置`flex-wrap:wrap`来避免意外布局行为**。这样编写的 CSS 具有防御性：

```css
/* 不具防御性的CSS */
.flex-container {
  display: flex; /* 或inline-flex */
}

/* 具有防御性的CSS */
.flex-container {
  display: flex; /* 或inline-flex */
  flex-wrap: wrap;
}
```

## 重要提示

需要注意的是，**`flex-wrap: wrap`只有在 Flex 容器没有足够空间容纳 Flex 项目时才会生效**。即，同一 Flex 行所有 Flex 项目最小内容宽度总和大于 Flex 容器宽度时，才会让 Flex 项目换行。

虽然在 Flexbox 容器中显式设置`flex-wrap: wrap`可以预防布局溢出，但并不代表在所有 Flexbox 容器上都要设置，我们应该在具体使用过程中有一个心理预判。

## 完整卡片布局示例

以下是一个卡片布局的完整示例，展示了如何结合`flex-wrap`和其他 Flexbox 属性创建防御性强的布局：

```html
<div class="card">
  <img src="https://picsum.photos/400/300?random=1" alt="" />
  <div class="content">
    <h4>防御式CSS</h4>
    <p>如何编写防御式CSS，使你的代码变得更健壮！</p>
  </div>
  <button>阅读全文</button>
</div>
```

```css
.card {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.card img {
  flex-shrink: 0; /* 防止图片因容器空间不足被挤压 */
}

.card .content {
  flex: 1 1 0%; /* 自动匹配Flex容器的剩余空间 */
  min-width: 0; /* 防止内容超出容器 */
}

.card h4 {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.card p {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.card button {
  margin-block: auto; /* 垂直居中 */
  margin-inline-start: auto; /* 居右对齐 */
}
```

# 防御式 CSS 课程 - 第四章：Flexbox 中的最小内容尺寸

## 最小内容尺寸的概念

在 Flexbox 布局中，一个经常被开发者忽视但极其重要的概念是**最小内容尺寸**。这个概念直接影响了 Flex 布局的稳定性，理解它可以帮助我们避免许多常见的布局问题。

### 什么是最小内容尺寸？

每个元素盒子都有一个最小内容尺寸。一般情况下：

- 如果元素的内容仅是字符串，那么它的最小内容尺寸等于内容中最长字符串的宽度
- 如果元素的内容是图文混合，那么它的最小内容尺寸等于图片的原始尺寸

最小内容尺寸在 CSS 中可以用关键词`min-content`来表示。如果你想让元素的尺寸刚好等于其内容最小尺寸，可以设置：

```css
.element {
  width: min-content;
}
```

如果将元素的尺寸设置为小于`min-content`时，就会产生内容溢出或出现滚动条。

## 最小尺寸与最小内容尺寸的区别

- **最小尺寸**是通过`min-width`、`min-inline-size`等属性设置的，它是一个 CSS 属性
- **最小内容尺寸**是一个尺寸值，即`min-content`关键词的计算值

关键区别在于：**元素盒子的最小尺寸不会小于该盒最小内容的尺寸**。即使你设置了较小的`min-width`值，如果内容的最小尺寸更大，那么元素的实际宽度将由内容决定。

## Flexbox 中的最小内容尺寸问题

在 Flexbox 布局中，Flex 项目的`min-width`属性默认值是`auto`，而不是`0`。这与普通元素不同，普通元素的`min-width: auto`计算值是`0`。

这个微小的差异导致了 Flexbox 布局中的一个重要特性：**Flex 项目在收缩时，其宽度不会小于其最小内容尺寸**。

这一特性会在以下场景产生意外结果：

### 场景一：等宽布局失效

许多开发者认为只需设置`flex: 1`就能实现等宽布局，但实际上如果某一列的内容较长，这一列会比其他列宽：

```html
<footer>
  <div><svg class="icon--home"></svg>首页</div>
  <div><svg class="icon--hot"></svg>沸点</div>
  <div><svg class="icon--discover"></svg>发现</div>
  <div><svg class="icon--book"></svg>掘金小册</div>
  <div><svg class="icon--my"></svg>我</div>
</footer>
```

```css
footer {
  display: flex;
}

footer > div {
  flex: 1 1 0%;
}
```

### 场景二：长字符串无法断行

当 Flex 项目中出现长字符串时，即使设置了`overflow-wrap: break-word`，长字符串也可能不会断行，导致布局溢出：

```html
<div class="card card--flexbox">
  <img src="https://picsum.photos/400/300?random=2" alt="" />
  <div class="card__content">
    <h3>防御式CSS</h3>
    <p>内容中有一个长字符串 VeryVeryVeryVeryLooooooooooooooooooongWord</p>
  </div>
</div>
```

```css
.card--flexbox {
  display: flex;
  gap: 1rem;
  align-items: center;
}

.card__content {
  flex: 1 1 0%;
}

.card--flexbox p {
  margin-top: 10px;
  overflow-wrap: break-word;
}
```

### 场景三：撑破弹性布局

当在 Flex 项目中嵌套另一个 Flex 容器（如幻灯片组件）时，如果内部 Flex 容器的内容宽度总和大于外部容器，会导致整个布局被撑破：

```html
<div class="flexbox">
  <header>.header</header>
  <section>
    <main>
      <div class="carousel">
        <img src="https://picsum.photos/400/300?random=1" alt="" />
        <img src="https://picsum.photos/400/300?random=2" alt="" />
        <!-- 更多图片 -->
      </div>
    </main>
    <aside>.sidebar</aside>
  </section>
  <footer>.footer</footer>
</div>
```

```css
.flexbox {
  display: flex;
  flex-direction: column;
}

.flexbox section {
  display: flex;
  gap: 1rem;
  flex: 1 1 0%;
}

.flexbox aside {
  width: 280px;
  flex-shrink: 0;
}

.flexbox main {
  flex: 1 1 0%;
}

.carousel {
  display: flex;
  gap: 1rem;
  overflow-x: auto;
}
```

## 防御性解决方案

以上问题都是由同一个原因造成的：**Flex 项目的最小尺寸不会小于其最小内容尺寸**。解决方法也很简单，有以下几种方式：

### 方法一：重置 min-width 为 0

最简单直接的方法是在 Flex 项目上显式设置`min-width: 0`：

```css
.flex-item {
  flex: 1 1 0%;
  min-width: 0;
}
```

这样就允许 Flex 项目在必要时可以收缩到小于其内容的最小尺寸。

### 方法二：设置 overflow 属性

另一种方法是设置`overflow`属性为非`visible`值：

```css
.flex-item {
  flex: 1 1 0%;
  overflow: hidden; /* 或auto或scroll */
}
```

使用`hidden`可以避免出现滚动条，而`auto`或`scroll`在内容溢出时会显示滚动条。

### 方法三：结合文本处理技术

对于文本溢出问题，可以结合文本截断技术：

```css
/* 单行文本截取，末尾添加... */
.text-overflow {
  min-width: 0; /* 或overflow: hidden */
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* 多行文本截取，末尾添加... */
.line-clamp {
  min-width: 0; /* 或overflow: hidden */
  --line-clamp: 2;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: var(--line-clamp);
  -webkit-box-orient: vertical;
}
```

## 防御式 Flexbox 的最佳实践

基于以上分析，我们可以总结出以下防御式 Flexbox 的最佳实践：

1. **设置 flex-wrap: wrap**：在 Flex 容器上设置`flex-wrap: wrap`可以防止因空间不足导致的布局溢出

2. **重置 min-width: 0**：在使用`flex: 1`（或`flex: 1 1 0%`）的 Flex 项目上，总是显式设置`min-width: 0`

3. **处理长字符串**：对可能包含长字符串的元素，除了设置`overflow-wrap: break-word`外，还要确保其父级 Flex 项目设置了`min-width: 0`或`overflow: hidden`

4. **注意嵌套 Flex 容器**：当 Flex 项目内嵌套另一个 Flex 容器时，确保外层 Flex 项目设置了`min-width: 0`

## 小结

在 Flexbox 布局中，了解和处理最小内容尺寸问题是构建防御性 CSS 的关键。当我们使用`flex: 1`让元素自动填充空间时，务必记住：

**当你使用`flex: 1`（或`flex: 1 1 0%`）时，总是在对应的 Flex 项目上显式设置`min-width: 0`**。

这样简单的一行代码，可以避免许多复杂的 Flexbox 布局问题，使你的 CSS 更具防御性，代码也更健壮。

# 防御式 CSS 课程 - 第五章：布局中的滚动失效和默认拉伸

## 滚动失效问题

在使用 Flexbox 布局时，你可能会遇到一些看似奇怪的现象，比如在 Firefox、Safari 和 Edge 浏览器中容器的滚动失效，或者 Flex 项目因其他内容而被拉伸变形。这些并非浏览器的 bug，而是 Flexbox 布局的"特性"。

### 场景一：百分百无滚动布局的滚动失效

最常见的滚动失效问题出现在"百分百无滚动布局"中，即页面的页头和页脚分别固定在页面顶部和底部，只有中间的内容区域可滚动：

```html
<body>
  <header>Header</header>
  <main>Main Content</main>
  <footer>Footer Bar</footer>
</body>
```

```css
body {
  height: 100vh;
  overflow: hidden;

  display: flex;
  flex-direction: column;
  gap: 4px;
}

main {
  flex: 1 1 0%;
  min-height: 0;
  overflow-y: auto;
}
```

### 场景二：justify-content: end 致使容器水平滚动失效

当在 Flex 容器上设置`justify-content: end`时，可能会导致容器的水平滚动失效：

```css
.flex--container {
  display: flex;
  justify-content: end;
  overflow-x: auto;
}
```

这是因为水平滚动条的设计约定：**只有容器右侧有多余内容时，才需要滚动**。而`justify-content: end`会让内容向左溢出，违背了这一设计规则。

### 解决方案：使用 margin-left: auto

可以使用`margin-left: auto`（或`margin-inline-start: auto`）来替代`justify-content: end`的效果：

```css
.flex--container {
  display: flex;
  overflow-x: auto;
}

.flex--container .card:first-child {
  margin-left: auto;
  /* 或 */
  margin-inline-start: auto;
}
```

对于垂直方向的 Flex 容器，使用`margin-top: auto`：

```css
.flex--container--column {
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.flex--container--column .item:first-child {
  margin-top: auto;
  /* 或 */
  margin-block-start: auto;
}
```

### 场景三：居中对齐导致内容无法完全滚动查看

当 Flex 容器设置了`align-items: center`并且内容溢出时，可能会出现无法滚动查看全部内容的问题：

```css
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  overflow-x: auto;
}
```

当 Flex 项目宽度大于 Flex 容器宽度时，项目会在左右两边溢出。问题是，左侧的溢出区域超出了容器视口的起始边缘，用户无法滚动到该区域。

### 解决方案：使用 margin 自动对齐

最简单的解决方法是在 Flex 项目上使用`margin`自动对齐，而不是在容器上使用`align-items: center`：

```css
.container > span {
  margin-inline: auto;
}
```

未来，CSS 将支持安全对齐方式：`align-items: safe center`，但目前该特性兼容性有限。

## 默认拉伸问题

Flexbox 布局中另一个常见问题是元素的 UI 形状被拉伸或挤压，使 UI 变得难看。

### 场景一：项目被拉伸到相同高度

在 Flexbox 布局中，默认情况下，所有 Flex 项目在侧轴方向会被拉伸（`align-items: stretch`是默认值）：

```html
<div class="card">
  <img src="card-thumnail.jpg" alt="Card Thumnail" />
  <p>Card Description</p>
  <button>Button</button>
</div>
```

```css
.card {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}
```

这会导致所有 Flex 项目（图片、段落、按钮）被拉伸到相同的高度。

### 解决方案：修改对齐方式

要避免这个现象，需要将`align-items`设置为非`stretch`值：

```css
.card {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: flex-start; /* 或center、baseline等 */
}
```

或者在单个不需要拉伸的项目上设置`align-self`：

```css
.card > *:not(p) {
  align-self: flex-start;
}
```

### 场景二：项目被挤压变形

当 Flex 容器空间不足时，Flex 项目可能会被挤压变形：

```html
<div class="card">
  <img src="thumnail.pgn" alt="Card Thumnail" />
  <p>Card Description</p>
  <div class="card__action">
    <svg></svg>
  </div>
</div>
```

```css
.card {
  display: flex;
  align-items: center;
  gap: 10px;
}

.card p {
  flex: 1 1 0%;
  white-space: nowrap;
}
```

当段落内容过长时，由于 Flex 项目默认可以被收缩（`flex-shrink: 1`），其他项目（如图片）会被挤压变形。

### 解决方案：

有三种解决方法：

1. 在段落上设置`min-width: 0`：

```css
.card p {
  flex: 1 1 0%;
  min-width: 0;
}
```

2. 使用文本截断：

```css
.card p {
  flex: 1 1 0%;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
```

3. 在不需要被挤压的项目上设置`flex-shrink: 0`：

```css
.card > *:not(p) {
  flex-shrink: 0;
}
```

## 防御式 Flexbox 最佳实践

为了避免以上问题，请记住以下防御式 CSS 实践：

1. **滚动容器设计**

   - 在使用 Flexbox 创建"百分百无滚动布局"时，为主内容区域添加嵌套的滚动容器
   - 在滚动容器的父元素（Flex 项目）上设置`min-height: 0`
   - 避免在滚动容器上使用`justify-content: end`，改用`margin-left: auto`

2. **防止 UI 拉伸**

   - 在 Flex 容器上显式设置`align-items`为非`stretch`值（如`flex-start`、`center`）
   - 或在不需要拉伸的 Flex 项目上设置`align-self: flex-start`

3. **防止 UI 挤压**
   - 在伸缩项目上设置`min-width: 0`
   - 对于固定尺寸的项目，设置`flex-shrink: 0`防止其被挤压
   - 对于长文本，结合使用`overflow: hidden`和`text-overflow: ellipsis`

记住这些简单的技巧，将大大提高你的 Flexbox 布局的稳定性和可靠性。

# 防御式 CSS 课程 - 第七章：如何灵活设置元素之间的间距

## 间距的重要性

在 Web 设计中，元素之间的间距不仅影响美观度，还能传达信息的关联性。适当的间距有助于：

- 提高页面的视觉层次感
- 改善用户体验和可读性
- 帮助用户理解元素之间的关系

没有适当间距的页面会让用户感到困惑，难以分辨哪些元素是相关的，哪些是不相关的。

## CSS 中间距的类型

CSS 中的间距主要分为两种类型：

### 外间距

- 用于设置元素盒子外部之间的间距
- 主要通过`margin`属性设置
- 在 Flexbox 布局中，还可以通过`gap`属性设置

### 内间距

- 用于设置元素盒子边缘和其内容之间的间距
- 主要通过`padding`属性设置

外间距和内间距的主要差异：

- `padding`不可以设置负值，而`margin`可以
- `padding`没有叠加的概念，而`margin`会发生叠加
- `padding`会影响元素的实际大小(除非使用`box-sizing: border-box`)

## 不同间距方式的选择原则

### 何时使用 padding

- 设置容器内容（子元素）距容器边缘的间距时
- 需要增加元素可点击区域时
- 希望背景色扩展到间距区域时

### 何时使用 margin

- 设置元素与元素之间的间距时
- 需要创建负间距效果时
- 元素之间的间距不相等时

### 何时使用 gap

- 在 Flexbox 容器中设置所有 Flex 项目之间的等距间隔时
- 需要避免选择器复杂性时
- 项目间距都相等且不需要边缘间距时

## 间距设置示例

以卡片组件为例：

```html
<div class="card">
  <figure><img src="card-thumbnail.jpg" alt="Card Thumbnail" /></figure>
  <div class="card__body">
    <h3>Card Title</h3>
    <p>Card Description</p>
  </div>
</div>
```

最合适的 CSS 间距设置：

```css
figure {
  margin-bottom: 40px;
}

.card__body {
  padding: 0 40px 40px;
}

.card h3 {
  margin-bottom: 20px;
}
```

在这个例子中：

- 图片与卡片主体之间使用`margin`分隔
- 卡片主体内部使用`padding`控制内容与边缘的间距
- 标题与描述文本之间使用`margin`分隔

## 防患于未然的间距设置

### 使用相邻选择器(E + F)处理动态内容

当你不确定元素是否总是存在时，相邻选择器可以帮助你创建防御性更强的 CSS：

```css
.button + .button {
  margin-left: 1rem;
}
```

这段代码表示："如果一个按钮紧挨着另一个按钮，给第二个按钮加一个左边距"。这对处理动态数据非常有用，比如模态框中可能有一个或两个按钮的情况。

### 处理元素列表的间距

当设置列表项间距时，避免使用：

```css
/* 不推荐：会导致最后一个元素有多余的下边距 */
.card {
  margin-bottom: 20px;
}
```

改为使用：

```css
/* 推荐：只有相邻元素之间才有间距 */
.card + .card {
  margin-top: 20px;
}

/* 或者 */
.card:not(:last-child) {
  margin-bottom: 20px;
}
```

### 使用:empty 选择器处理空容器

当容器没有内容时，保留的 padding 会导致不必要的空白：

```css
.card {
  padding: 20px;
}

.card:empty {
  padding: 0;
  border: none;
}
```

这样当卡片没有内容时，不会显示一个空白区域。

### 滚动容器的间距问题

在水平滚动容器中，padding 可能会在视觉上"丢失"：

```css
.cards {
  display: flex;
  gap: 10px;
  padding: 18px;
  overflow-x: auto;
}
```

解决方案是使用嵌套结构：

```html
<div class="scroll--wrapper">
  <div class="cards">
    <!-- 卡片内容 -->
  </div>
</div>
```

```css
.scroll--wrapper {
  padding: 18px;
}

.cards {
  display: flex;
  gap: 10px;
  overflow-x: auto;
}
```

## 小结

设置元素间距时的防御式 CSS 原则：

1. **明确选择间距类型**：根据元素关系选择合适的 padding、margin 或 gap
2. **考虑动态内容**：使用相邻选择器或否定选择器处理元素数量变化
3. **处理边缘情况**：使用:empty 或特定选择器处理空容器
4. **注意滚动容器**：为滚动容器使用嵌套结构保留视觉间距
5. **灵活使用工具**：结合 CSS 选择器和间距属性创建更健壮的布局

记住："好的间距设计是用户体验的基础，防御性的间距设置是稳定布局的保障。"
