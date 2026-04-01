# Pretext 引力场演化项目 / Pretext Gravitational Field Evolution

本项目是一个基于 **Vanilla Canvas 2D API** 与 **`@chenglou/pretext`** 排版引擎的高性能互动文本粒子物理系统。它展示了如何将网页纯文本排版转化为一套具备引力、弹力、碰撞等全动态效果的微观物理生态。

## 🌟 核心特性

- **零重度框架约束**：脱离 React/Vue 等繁重的 DOM 框架，完全使用原生 Canvas 结合原生 ESM (Import Maps) 模块驱动。
- **高级物理与形态系统**：
  - **动态游荡系统**：黑洞与白洞跟随光标带有缓动与惯性（Lerp插值）。
  - **基于 PBD (位置约束动力学) 的粒子引擎**：每个排版字符都被转化为独立的物理刚体实体。字与字之间发生碰撞、重力堆叠，彻底告别海量刚体过度修正引起的异常抖动。
  - **状态机流转**：文字粒子可在「悬浮 (Idle)」、「吞噬 (Swallowed)」、「坠落堆叠 (Knocked Down)」等多种状态中丝滑切换。
- **与 Pretext 引擎深度结合**：利用 `@chenglou/pretext` 提供精准备考断行，接着通过 Canvas `measureText` 进行“二次解构”，将整段排版布局拆分为无数个携带原始坐标的独立小球。不论物理引力如何拉扯，弹性张力（`springStiffness`）最终都会拉扯文字回归它们独特的排版原点。
- **光影与交互体验**：
  - **昼夜形态蜕变**：底层背景、字体颜色随着引力场在白洞（发射/黑夜）、黑洞（吞噬/白天）形态之间的切换发生无缝 CSS 渐变阻尼反转。
  - **射击与创世复苏机制**：交互发射高速光弹（击碎处于 idle 稳态的文字），并能将地面的碎片文字重新击飞半空并由原位弹簧再次捕获复原。
  - **高级玻璃态控制面板**：基于深度魔改覆写的 `lil-gui` 属性调整台，带有 iOS 质感的阻尼开关及随昼夜高仿主题配色的 `backdrop-filter` 模糊设计。

## 📁 核心文件结构

- `index.html` / `pretext.html` / `traditional.html`：项目入口及主视图。
- `struc.md`：核心架构与引擎物理特性的深度总结文档。
- `pretext-use.md`：对 `@chenglou/pretext` 高性能排版引擎工作流的集成原理及物理“二次拆字解构”的深入解析。

## 🚀 运行方式

无需任何构建工具（如 Webpack、Vite 等）或环境配置。你只需：

1. 克隆或下载本项目。
2. 使用现代主流浏览器（支持 ESM 与 Import Map）直接打开 `index.html`。
3. （可选）为避免部分浏览器的跨域/本地 file:// 协议限制，推荐可使用 `Live Server`（VS Code 插件）或其他简易本地服务器环境运行本项目根目录。

## 🛠 技术栈

- **渲染引擎**: HTML5 `<canvas>` (2D Context)
- **排版分词预测**: `@chenglou/pretext` (CDN 跨域导入)
- **UI & Controls**: 原生 CSS + `lil-gui`
- **物理机制**: 定制化 PBD (Position Based Dynamics) 粒子堆叠法则 + 自制引力/弹簧动力系统
