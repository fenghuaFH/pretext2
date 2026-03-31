# 深度解析：如何在原生物理系统中集成 Pretext 排版引擎

本项目引入了刚开源不久的高性能单次文本排版库 `@chenglou/pretext`。它彻底剥离了浏览器 DOM 和 Canvas 的重度依赖关系，允许开发者仅仅提供尺寸参数，就能计算出长文本在屏幕上具体该怎么换行。

在这本引擎里，我们不仅依赖了它的输出，还通过“二次解构”将**整行文字打散成了无数个带有原始坐标的独立小物理球**。

---

## 1. 为什么不用 DOM 也需要 Pretext？

正常在 Canvas 中使用 `fillText("Hello\nWorld")` 并不存在自动针对容器宽度的**折行和自动多行排版**。如果在 Canvas 中自己去实现它，由于汉字、标点符号的间距差异极大，代码会又长又难维护。

而 `@chenglou/pretext` 就专注于纯逻辑引擎做一件事：“根据一个指定的字体和最大宽度，告诉我这些字该分为几行，每一行包含什么字符。”

---

## 2. 引入与使用（原生 ESM）

在无需借助 Webpack / Vite 的单文件 HTML 项目中，我们直接在顶部 ImportMap 声明模块节点：

```html
<script type="importmap">
  {
    "imports": {
      "pretext": "https://esm.sh/@chenglou/pretext"
    }
  }
</script>
```

然后在 `<script type="module">` 内按需解构引入核心双方法：

```javascript
import { prepareWithSegments, layoutWithLines } from 'pretext';
```

---

## 3. 标准计算工作流解析

一旦当用户按下了“确认修改排版”按钮，或者当屏幕窗口大小拉伸重置（Resize 发生了 150ms 停顿时），我们会依次发下如下指令引擎。

### Step ① 预估文案：`prepareWithSegments`
Pretext 要求输入完整长明文字符串和将要使用的 Canvas 字体规格。它首先将其切切切块准备进运算池。

```javascript
// 前置要求：告诉排版引擎我用的字号粗细是什么（这关系到它在底层计算字符胖瘦）
const fontStr = `500 ${params.fontSize}px -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto...`;
const textString = params.textString; // 如 500 个“差评”

// 第一步：预处理
const preparedData = prepareWithSegments(textString, fontStr);
```

### Step ②：下放布局：`layoutWithLines`
这一步决定了“回车与截断”。我们告诉引擎，现在有这么多字、限制在一行只能放多少宽度 (`maxW`)、上下换行该隔开多少留白（`lineHeight`）。

```javascript
const maxW = width * 0.55; // 这意味着最多只能占用屏幕 55% 宽度的空间
const lineHeight = params.fontSize * 1.6;

// 第二步：正式布局
// 返回的结果内部会包含 `.lines` 数组
const layoutResult = layoutWithLines(preparedData, maxW, lineHeight);
```

---

## 4. 从行数据 (Line) 到粒子数据 (Particle) 的转换

由于 `pretext` 为了兼顾性能，通常默认的粒度是按“单词/行段”划分的。`layoutResult.lines` 里是一行一行的字符串（`line.text`），但这**并不满足我们做粒子特效的要求**，因为我们想要屏幕上的每一个单独的汉字（哪怕是一个句号）都能在受重力猛烈撞击时脱离队伍。

在这个项目中，我们运用了 Canvas 的 `ctx.measureText` 进行最后一步**拆字映射**：

```javascript
const startX = Math.max(40, width * 0.2 - maxW * 0.5); // 定位左侧
let currentY = startY;

// 第三步：历遍遍历，将虚拟排版赋予物理载体
let layoutItems = [];

// 进入排版引擎推演好的每一行
for (let i = 0; i < layoutResult.lines.length; i++) {
  const line = layoutResult.lines[i];
  let currentX = startX;
  
  // 强行把每一行里的字再逐个拆开
  for (const char of line.text) {
    // 仅仅用原生 Canvas 测算这个单汉字的精准宽度
    const charWidth = ctx.measureText(char).width;
    
    // 如果不是空壳，就直接实例化它为一个带有确切目标归属坐标 (currentX, currentY) 的物理球！
    if (char.trim() !== '') {
      layoutItems.push({ char, charWidth, originX: currentX, originY: currentY });
    }
    // 无论如何游标都要推进一段距离
    currentX += charWidth;
  }
  // 游标推行下一行
  currentY += lineHeight;
}
```

这也就是为何在引擎拉扯计算时——这无数个游离汉字不管在哪儿互相撞击挤压飞翔，它们脑袋里都深深记挂着各自独一无二的 `originX` 和 `originY`，并受着 `springStiffness`（排版弹簧张力）的不懈扯动试图回家。从而成就了完美的有机排版物理奇观。
