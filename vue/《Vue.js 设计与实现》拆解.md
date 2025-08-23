# 《Vue.js 设计与实现》拆解

> 对 **Vue 官方团队成员：霍春阳** 编写的 **《Vue.js 设计与实现》**精读、拆解

---

## 📚 第一篇 | [框架设计概览]

### 1.1 **权衡的艺术**

#### 📖 核心问题

❓ **这一章要解决什么问题？**

- 为什么 Vue 要选择声明式而不是命令式？
- 为什么 Vue 要选择虚拟 DOM 而不是 innerHTML？
- 为什么 Vue 要采用"运行时+编译时"而不是纯编译时/纯运行时？

💭 **我的理解**

**声明式选择的原因：**

- 命令式性能更好，声明式的代码可维护性更高
- Vue `内部封装命令式`实现，`对外提供声明式`
- 开发者只需关心结果而不是如何实现

**虚拟 DOM 选择的原因：**

- 性能排序：原生 DOM 操作 > 虚拟 DOM > innerHTML
- 可维护性排序：虚拟 DOM > innerHTML > 原生 DOM 操作
- 虚拟 DOM 在性能与可维护性之间找到最佳平衡点

**运行时+编译时选择的原因：**

- 纯编译时（如 Svelte）：性能理论最优，但缺乏运行时灵活性
- 纯运行时：灵活性最高，但无法做编译时优化
- 结合方案：既能`编译时分析优化，又保持运行时动态能力`

❓ **为什么需要这样设计？**

💭 **我的理解**

- 性能不是唯一目标，开发效率和维护成本同样重要
- 需要在理论最优和实际可用之间找平衡

❓ **如果 Vue 采用其他方案会有什么问题？**

💭 **我的理解**

**如果选择命令式（如原生 DOM 操作、jQuery 等）：**

- 代码复杂度激增，学习成本极高
- 维护困难，团队协作效率低
- 心智负担重，开发者需要关注实现细节而非业务逻辑

**如果选择 innerHTML：**

- 无法精准更新，每次都需要重建整个 DOM
- 无法支持组件化
- 跨平台受限(虚拟 DOM 可以抽象成对象)

**如果选择纯编译时：**

- 动态内容支持受限（无法处理运行时数据）
- 调试困难（编译后代码难读）

**如果选择纯运行时：**

- 无法分析用户提供的内容，编译时错误检查缺失
- 缺乏编译时优化空间（静态提升、预字符串化、Block 树等）
- 模板能力受限，要么性能差（运行时解析），要么心智负担高（纯 render 函数）

#### 🎯 关键概念

- **命令式 vs 声明式** ：

  - 命令式：告诉计算机"怎么做"，关注执行过程，像写菜谱的详细步骤
  - 声明式：告诉计算机"想要什么"，关注最终结果，像点菜只说菜名
  - Vue 选择声明式是为了降低心智负担，提升开发效率

- **虚拟 DOM 的选择**
  - 不是最快的方案，但是最平衡的方案
  - 提供了抽象层，支持跨平台和组件化
  - **允许框架做多层次优化：**
    - **编译时优化**：静态节点标记、动态节点收集、Block 树
    - **运行时优化**：diff 算法、批量更新、异步更新队列
    - **跨层优化**：编译时分析 + 运行时精准更新
- **运行时+编译时**
  - 编译时：template -> render 函数，做静态分析和优化
  - 运行时：执行 render 函数，处理动态逻辑
  - 两者结合：既有编译时优化，又有运行时灵活性

#### 🤔 质疑与思考

**问题 1：React 也用虚拟 DOM，Vue 和 React 的虚拟 DOM 有什么不同？**

- React：主要依靠虚拟 DOM diff，使用 Fiber 架构做时间切片
- Vue：结合响应式系统，能精确知道哪些组件需要更新
- 差异：Vue 的更新更精准，React 的并发能力更强

**问题 2：Svelte 的纯编译时方案真的不如 Vue 吗？**

- Svelte 优势：编译时优化彻底，`运行时体积小、首屏快`，性能理论最优
- Svelte 劣势：动态性受限，生态较小，复杂应用支持不足
- 适用场景：静态内容为主的网站 、高频小更新场景

**问题 3：在什么场景下，命令式代码比声明式更好？**

- 性能极致要求的场景（游戏、动画）
- 底层库开发（需要精确控制）
- 简单的 DOM 操作（不值得引入框架）

**问题 4：虚拟 DOM 相比直接 DOM 操作，在什么场景下优势最明显？**

- 频繁更新的复杂界面
- 数据频繁变化的表格、列表：框架自动 diff，精准更新，批量处理
- 组件化、状态管理复杂应用：
- 大量嵌套组件的页面
- 组件间状态共享和传递
- 组件的动态创建和销毁
- 跨平台需求

### 1.2 框架设计的核心要素

#### 📖 核心问题

❓ **这一章要解决什么问题？**

- 如何控制打包产物的体积？
- 如何提高框架的可维护性和用户体验？
- 如何处理框架中的错误？

💭 **我的理解**

**打包体积控制的必要性：**

- 打包体积直接影响页面加载速度和用户体验
- Vue 通过`环境变量、特性开关和 Tree-Shaking` 实现精确控制

**错误处理设计的价值**

- 统一的错误处理
- 集中收集错误便于调试和问题定位
- 区分`不同错误类型`有助于分类处理

**TypeScript 类型支持的重要性：**

- 提供了强大的类型检查，降低错误率
- 提供了更好的开发体验和 IDE 支持
- 类型定义本身也是一种文档

❓ **为什么需要这样设计？**

💭 **我的理解**

**打包体积控制：**

- Web 应用性能的关键在于资源加载速度
- 用户不应下载不需要使用的代码（如开发环境代码）
- 按需加载是现代前端框架的核心竞争力

**错误处理：**

- 框架内部错误如不统一处理、不明确会导致开发者体验差
- 统一接口便于开发者扩展错误处理（如上报、记录）

**关于类型系统：**

- 大型项目没有类型支持会导致维护困难
- IDE 智能提示依赖类型系统
- 类型是一种隐性文档，减少沟通成本

❓ **如果不这样做会怎样？**
💭 **我的理解**

**不控制打包体积：**

- 应用体积膨胀，加载时间延长， 用户体验变差
- 包含不必要代码（如开发警告）增加运行时开销

**不提供错误处理机制**

- 错误定位困难，开发效率低下
- 开发者需要自行处理各类错误，代码冗余

**不重视类型支持**

- 大型项目维护困难，重构风险高（靠人为约定而不是系统设计。）
- API 使用门槛提高，需要查阅更多文档
- 编辑器智能提示失效，开发效率降低

#### 🎯 关键概念

- **环境变量控制：**

  - Vue 使用`__DEV__`由*构建工具预定义*的环境变量区分开发/生产环境
  - 生产环境下自动移除所有开发警告代码

- **Tree-Shaking 优化**

  - Tree-Shaking 是一种静态分析技术，移除未使用的代码（dead code）
  - Vue 通过三种方式支持 Tree-Shaking：
    1. 使用 ES 模块（import/export）
    2. 导出独立函数而非默认导出对象
    3. 使用`/*#__PURE__*/`注释标记纯函数

- **特性开关和构建产物**

  - 特性开关可以仅包含使用到的功能，减少冗余代码
  - 按需加载
  - 可构建出多种不同用途的产物（完整版/运行时版/服务端版等）

- **错误处理**

  - Vue 提供 `app.config.errorHandler `全局配置， 所有内部错误通过`callWithErrorHandling`函数处理
  - 错误分类处理，包括：
    - 组件渲染错误
    - 生命周期钩子错误
    - 事件处理器错误
    - 侦听器错误
  - 提供错误上下文信息（组件名、钩子名等）
  - 集中处理所有类型错误，并支持自定义处理逻辑

  ```js
  let handleError = null
  export default {
    foo(fn) {
      callWithErrorHandling(fn)
    }
    // 用户可以调用该函数注册统一的错误处理函数
    registerErrorHandler(fn) {
      handleError = fn
    }
  }

  function callWithErrorHandling (fn) {
    try {
      fn && fn()
    } catch (e) {
      // 将捕获的错误传递给用户的错误处理程序
      handlerError(e)
    }
  }
  ```

- **TypeScript 支持**

  - Vue 3 完全支持使用 TypeScript 开发
  - 提供完整的类型定义和类型推导

#### 🤔 质疑与思考

**问题 1：为什么 Vue 要构建不同的打包产物？**

- 不同用户有不同需求，单一产物无法满足所有场景，用户可根据自身需求选择最适合的版本
  - 完整版（包含编译器）：适用于不使用构建工具的场景，可直接编译模板
  - 运行时版（不含编译器）：体积更小，适用于使用构建工具的场景
  - ESM 格式：支持 Tree-Shaking，适用于现代打包工具
  - IIFE/UMD 格式：适用于直接通过`<script>`标签使用的场景

**问题 2：callWithErrorHandling 接口可以捕获到所有错误吗？**

- 可以通过`try/catch`可以捕获到`运行时`以下类型的错误：
  - **组件渲染错误**：模板渲染过程中发生的错误
  - **生命周期钩子错误**：如 mounted, updated 等生命周期函数执行时的错误
  - **事件处理器错误**：用户交互事件处理函数中的错误（如 @click 绑定的处理函数）
  - **侦听器错误**：watch 和 computed 中发生的错误
- 不能捕获以下类型的错误：

  - **异步操作中的错误**：Promise 中的 reject、 setTimout、async/await
  - **非 Vue 管理的代码错误**：直接添加的 DOM 事件监听器 、第三方库
  - **网络请求错误：** Ajax/Fetch API 错误
  - **编译时错误：** 模板语法错误、TypeScript 类型错误

**问题 3：如何完善错误处理**

- window.onerror 和 unhandledrejection 事件处理异步错误
- 在关键异步操作中手动使用 try-catch 并调用错误处理器
- 使用 `errorCaptured` 生命周期钩子捕获特定组件树中的错误
- 采用第三方监控工具(如 Sentry)补充 Vue 的错误处理机制

### 1.3 **Vue.js 3 的设计思路**

#### 📖 核心问题

❓ 这一章要解决什么问题？

- Vue 框架的整体架构是怎样的？
- Vue 框架的核心模块之间是如何协同工作的？
- Vue 是如何从声明式 UI 描述到最终渲染为 DOM 的？
- 组件化是如何在 Vue 中实现的？

💭 我的理解

- **Vue 框架的整体架构：**
  - Vue 框架的设计主要分为三大部分：**声明式 UI 描述、渲染器和组件化系统**
  - 采用"运行时+编译时"相结合的架构模式，兼顾灵活性和性能优化
- **渲染链路工作流程：**
  1. 开发者通过**模板或渲染函数**声明式描述 UI
  2. 模板通过**编译器**转换为渲染函数
  3. 渲染函数执行生成虚拟 DOM
  4. 渲染器将虚拟 DOM 渲染为实际平台上的 UI 元素
  5. 当数据变化时，重新生成虚拟 DOM 并更新
- **组件化系统实现**
  - **组件本质是一组 DOM 元素的封装**
  - 组件可以抽象成函数或者带有 render 方法的对象
  - 组件可以包含自己的状态、行为和渲染逻辑

❓ 为什么需要这样设计？
💭 我的理解

**Vue 架构分层、保持相对独立（声明式 UI 描述、渲染器和组件化）：**

- 关注点分离，每个模块负责独立的功能，降低系统耦合
- 允许针对不同部分进行优化

**渲染器与平台分离**

- 使 Vue 能够在不同环境中运行（浏览器、服务器、原生应用等
- 提供了抽象层，隔离平台差异
- 支持自定义渲染目标，扩展到更多场景

**组件化设计**

- 提高代码复用率
- 关注点分离，每个组件专注自己的功能
- 简化状态管理和更新机制
- 支持团队协作和大型应用开发

❓ 如果不这样做会怎样？
💭 我的理解
**如果没有清晰的架构划分：**

- 代码耦合严重，难以维护和扩展
- 无法进行针对性优化

**如果渲染器与平台强耦合：（like ReactNative）**

- 无法支持跨平台，只能在浏览器环境中使用
- 难以支持服务端渲染(SSR)和原生应用开发
- 测试困难，依赖 DOM 环境
- 扩展性受限

**如果没有组件化机制：**

- 代码复用困难
- 状态管理复杂
- 大型应用开发效率低下
- 维护成本高昂

#### 🎯 关键概念

- **声明式 UI 描述**

1. **模板 template**

- HTML 扩展语法，直观易读
- 需要**编译器**转换为渲染函数, 支持编译优化(patchFlag)
- 支持指令、插值、事件等特性
- 示例：`<div @click="onClickMsg">{{ message }}</div>`

2. **渲染函数 render**

- 更接近 JavaScript，灵活度高
- 可直接操作虚拟 DOM 结构
- 适合复杂逻辑和动态 UI 生成
- 示例：`h('div', null, this.message)`

两者本质上都是声明式的，只是抽象级别不同。模板更接近最终输出，渲染函数更灵活但需要手动构建结构。

- **渲染器设计**
  渲染器`renderer`是 Vue 核心模块，负责将虚拟 DOM 渲染为真实 DOM 元素。

  渲染器设计的精髓在于将 DOM 操作抽象为通用 API，使 Vue 能够在不同平台上运行。

  以**首次渲染**来说分三步：

  - **创建元素**：将 vnode.tag 作为标签名创建 DOM
  - **为元素添加属性和事件**： 遍历 vnode.props 对象，例如 on 开头说明是事件，通过 addEventListener 绑定事件
  - **处理 children**： 如果 children 是数组则递归调用 renderer 渲染。如果是字符串，则通过 createTextNode 创建文本节点。

- **组件化机制**

  - **组件的本质：**

    - 本质上是一个返回虚拟 DOM 的函数或对象
    - 可以拥有自己的状态、方法和生命周期
    - 可以嵌套组成组件树形成复杂 UI

  - **组件的渲染流程：**
    - 创建组件实例：初始化组件状态和属性
    - 数据响应式处理：使数据变化能触发更新
    - 执行 `render 函数`：生成虚拟 DOM 树
    - `渲染器 renderer `处理：将虚拟 DOM 渲染为真实 DOM
    - 挂载完成：触发相关生命周期钩子
  - **组件更新流程：**

    - 检测数据变化：通过响应式系统
    - 重新执行 render 函数：生成新的虚拟 DOM
    - diff 算法比较：找出需要更新的部分
    - 最小化 DOM 操作：只更新变化的部分
    - 更新完成：触发更新后的生命周期钩子

- **运行时+编译时**

  - **编译时优化**：静态提升、预字符串化、树结构扁平化
  - **运行时灵活性**：支持动态内容、条件渲染、循环等
  - **最佳平衡点**：在开发体验和运行时性能之间取得平衡

#### 🤔 质疑与思考

**问题 1：Vue 的架构设计与 React 相比有什么优势和劣势？**

- **Vue 优势：** 响应式+模板+组件化
  - 响应式系统更自动化，直接修改数据即可触发更新
  - 模板语法更接近 HTML，学习成本较低
  - 更精确的更新策略，依赖收集能知道哪些组件需要更新
- **React 优势：** 单向数据流+JSX+组件化
  - 纯 JavaScript 思维，函数式编程思想更纯粹
  - Fiber 架构支持**并发渲染**和时间切片
  - 大型应用状态管理思想更清晰（单向数据流）

**问题 2：为什么 Vue 同时支持模板和渲染函数两种 UI 描述方式？**

- 满足不同开发者的偏好和需求，增加框架灵活性，符合 Vue 渐进式设计原则
- 模板适合大多数场景，直观易读
- 渲染函数适合动态性强、逻辑复杂的场景

**问题 3：渲染器的设计如何影响 Vue 的跨平台能力？**

- 通过工厂函数 `createRenderer`创建特定平台渲染器，能够在多种环境中运行
- 注入平台特定 API
- 渲染核心逻辑与平台操作分离（服务端渲染/ Weex）

**问题 4：组件的本质到底是什么？它与原生 Web Components 的关系是什么？**

- **组件的本质：**
  - 一个返回虚拟 DOM 树的函数或对象
  - 具有自己的状态、属性和方法的独立单元
- **与 Web Components 的关系**
  - 概念相似：都是为了 UI 复用和封装
  - 实现不同：Vue 组件是框架层面的抽象，Web Components 是浏览器原生标准
  - 兼容性：Vue 组件在所有环境中都能工作，Web Components 依赖浏览器支持
  - 功能对比：Vue 组件提供更丰富的功能（响应式、生命周期等）
- **Vue 对 Web Components 的支持：**
  - 可以在 Vue 应用中使用 Web Components
  - 可以将 Vue 组件编译为 Web Components
  - 两者可以互补使用

**问题 5：渲染器如何将虚拟 DOM 转换为真实 DOM 及更新机制**

1. **创建渲染器实例**

```js
// 创建渲染器的工厂函数
function createRenderer(options) {
  // 从配置中获取平台特定API
  const {
    createElement, // 创建元素
    setElementText, // 设置元素文本
    insert, // 插入元素到DOM
    patchProp, // 处理元素属性
    // 其他平台API...
  } = options;

  // 实现渲染器内部功能...

  // 返回渲染器对象
  return {
    render,
    hydrate,
    createApp: createAppAPI(render, hydrate),
  };
}
```

2. **虚拟 DOM 到真实 DOM 的转换过程**

```js
// 核心render函数
function render(vnode, container) {
  // 如果存在旧vnode，执行更新，否则执行挂载
  if (container._vnode) {
    patch(container._vnode, vnode, container);
  } else {
    // 首次渲染
    patch(null, vnode, container);
  }

  // 保存当前vnode用于后续更新
  container._vnode = vnode;
}

// patch函数 - 核心转换与更新逻辑
function patch(n1, n2, container, anchor = null) {
  // n1是旧vnode，n2是新vnode

  // 1. 处理不同类型的vnode
  const { type } = n2;

  // 如果是组件
  if (typeof type === "object" || typeof type === "function") {
    processComponent(n1, n2, container, anchor);
  }
  // 如果是普通元素
  else if (typeof type === "string") {
    processElement(n1, n2, container, anchor);
  }
  // 处理其他类型（文本、注释等）
  else {
    processText(n1, n2, container, anchor);
  }
}
```

3. **挂载元素的具体实现**

```js
// 处理元素的挂载和更新
function processElement(n1, n2, container, anchor) {
  if (n1 === null) {
    // 挂载新元素
    mountElement(n2, container, anchor);
  } else {
    // 更新已有元素
    patchElement(n1, n2);
  }
}

// 挂载元素
function mountElement(vnode, container, anchor) {
  // 1. 创建DOM元素
  const el = (vnode.el = createElement(vnode.type));

  // 2. 处理子节点
  if (typeof vnode.children === "string") {
    // 如果子节点是文本
    setElementText(el, vnode.children);
  } else if (Array.isArray(vnode.children)) {
    // 如果子节点是数组，递归挂载每个子节点
    for (let i = 0; i < vnode.children.length; i++) {
      patch(null, vnode.children[i], el);
    }
  }

  // 3. 处理props（属性、事件等）
  if (vnode.props) {
    for (const key in vnode.props) {
      patchProp(el, key, null, vnode.props[key]);
    }
  }

  // 4. 将创建的元素插入容器
  insert(el, container, anchor);
}
```

**DOM 更新机制**

1. **更新元素的实现**

```js
// 更新已存在的元素
function patchElement(n1, n2, container) {
  // 获取真实DOM元素并保存到新vnode上
  const el = (n2.el = n1.el);

  // 1. 更新props
  const oldProps = n1.props || {};
  const newProps = n2.props || {};

  // 设置新属性
  for (const key in newProps) {
    if (newProps[key] !== oldProps[key]) {
      patchProp(el, key, oldProps[key], newProps[key]);
    }
  }

  // 删除不再存在的旧属性
  for (const key in oldProps) {
    if (!(key in newProps)) {
      patchProp(el, key, oldProps[key], null);
    }
  }

  // 2. 更新子节点
  patchChildren(n1, n2, el);
}
```

2. **子节点更新策略**

```js
// 更新子节点
function patchChildren(n1, n2, container) {
  // 获取新旧子节点
  const oldChildren = n1.children;
  const newChildren = n2.children;

  // 根据子节点类型采用不同策略

  // 情况1: 新子节点是文本
  if (typeof newChildren === "string") {
    // 如果旧子节点是数组，需要移除所有旧节点
    if (Array.isArray(oldChildren)) {
      oldChildren.forEach((child) => unmount(child));
    }
    // 设置新文本
    setElementText(container, newChildren);
  }
  // 情况2: 新子节点是数组
  else if (Array.isArray(newChildren)) {
    // 如果旧子节点也是数组，需要diff算法
    if (Array.isArray(oldChildren)) {
      // 这里是diff算法的核心
      diffChildren(oldChildren, newChildren, container);
    }
    // 如果旧子节点是文本或不存在，清空容器并挂载新节点
    else {
      setElementText(container, "");
      newChildren.forEach((child) => patch(null, child, container));
    }
  }
  // 情况3: 新子节点不存在
  else {
    // 清空旧子节点
    if (Array.isArray(oldChildren)) {
      oldChildren.forEach((child) => unmount(child));
    } else if (typeof oldChildren === "string") {
      setElementText(container, "");
    }
  }
}
```
