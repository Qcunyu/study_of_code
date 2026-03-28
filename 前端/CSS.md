# 一、CSS 是什么

- **层叠样式表**（Cascading Style Sheets），用于控制 HTML 元素的呈现。
- 核心思想：**样式与内容分离**，通过选择器匹配元素，统一设置外观。

---

# 二、引入方式

|方式|示例|说明|
|---|---|---|
|**外部样式表**|`<link rel="stylesheet" href="style.css">`|推荐，分离结构与样式，可复用、可缓存|
|**内部样式表**|`<style> body { margin: 0; } </style>`|写在 HTML 的 `<head>` 中，适用于单页面|
|**内联样式**|`<div style="color: red;">`|优先级最高，但难以维护，渗透测试中可能发现临时调试代码|

---

# 三、选择器（核心）
选择器用于指定哪些元素应用样式，也是 [[JavaScript]] 操作 DOM 的重要依据。

|选择器|示例|说明|
|---|---|---|
|**元素选择器**|`p { }`|所有 `<p>` 标签|
|**类选择器**|`.box { }`|所有 `class="box"` 的元素|
|**ID 选择器**|`#header { }`|`id="header"` 的唯一元素|
|**属性选择器**|`[type="text"] { }`|匹配特定属性值的元素|
|**后代选择器**|`div p { }`|`<div>` 内部的 `<p>`（所有层级）|
|**子选择器**|`div > p { }`|直接子元素 `<p>`|
|**伪类**|`a:hover { }`|元素状态，如悬浮、焦点|
|**伪元素**|`p::before { }`|插入虚拟内容|

**安全视角**：
- 属性选择器可用于检测是否存在特定元素（如 `input[type="hidden"]` 可被 [[JavaScript]] 修改）
- 伪类 `:hover`、`:active` 可能被用来隐藏钓鱼按钮（平时不可见，鼠标移动才显示）

---

# 四、盒模型（布局基础）

每个 HTML 元素都是一个矩形盒子，包含以下部分：
```text
+----------------------------------+
|         margin (外边距)           |
|  +----------------------------+  |
|  |         border (边框)       |  |
|  |  +----------------------+  |  |
|  |  |      padding (内边距) |  |  |
|  |  |  +----------------+  |  |  |
|  |  |  |   content      |  |  |  |
|  |  |  +----------------+  |  |  |
|  |  +----------------------+  |  |
|  +----------------------------+  |
+----------------------------------+
```

|属性|说明|
|---|---|
|`width` / `height`|默认只设置内容区域宽高（可通过 `box-sizing` 调整）|
|`padding`|内边距，背景色会延伸到此处|
|`border`|边框，可设置颜色、粗细、样式|
|`margin`|外边距，元素之间的间距|
|`box-sizing: border-box`|使宽高包含 border 和 padding，布局更友好|

---

# 五、布局方式

## 1. 常规流（块级 / 行内）

- 块级元素独占一行，行内元素同行排列。
- 通过 `display` 改变行为：`display: block`、`inline`、`inline-block`、`none`（隐藏且不占位）、`visibility: hidden`（隐藏但仍占位）
## 2. Flexbox（一维布局）
```css
.container {
    display: flex;
    justify-content: center;  /* 主轴对齐 */
    align-items: center;      /* 交叉轴对齐 */
    flex-direction: row;      /* 主轴方向 */
}
```
- 适用于导航栏、卡片列表、垂直居中。

### 3. Grid（二维布局）
```css
.container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;  /* 三列等宽 */
    gap: 10px;
}
```
- 适用于整体页面布局、复杂网格。

### 4. 定位（`position`）

|值|说明|
|---|---|
|`static`|默认，按常规流排列|
|`relative`|相对自身原本位置偏移|
|`absolute`|相对最近的非 static 父元素定位，脱离文档流|
|`fixed`|相对视口定位，不随滚动|
|`sticky`|粘性定位，滚动到阈值时表现类似 fixed|

**安全视角**：
- `position: absolute` + `left: -9999px` 或 `visibility: hidden` 可隐藏恶意元素，用于钓鱼或水坑攻击。
- `position: fixed` 可制作覆盖全屏的钓鱼浮层。

---

# 六、常用样式属性

### 1. 颜色与背景
- `color`: 文字颜色
- `background-color` / `background-image`: 背景
- `opacity`: 透明度（0~1）
### 2. 文字
- `font-size`、`font-family`、`font-weight`（粗细）、`text-align`（对齐）、`line-height`
### 3. 圆角与阴影
- `border-radius`: 圆角，常用于按钮、卡片
- `box-shadow`: 阴影，可制造视觉层次
### 4. 过渡与动画
- `transition`: 平滑变化（如 `transition: all 0.3s`）
- `@keyframes` + `animation`: 复杂动画

---

# 七、响应式设计

- **媒体查询**：根据屏幕尺寸、设备特性应用不同样式
```css
@media (max-width: 768px) {
    body {
        font-size: 14px;
    }
}
```
- **视口 meta 标签**：`<meta name="viewport" content="width=device-width, initial-scale=1.0">`，移动端适配必备。

---

# 八、与 [[HTML]]、[[JavaScript]] 的协作

|角色|说明|
|---|---|
|**[[HTML]]**|提供 DOM 结构，CSS 通过选择器选中元素|
|**[[JavaScript]]**|可动态修改元素的 `class`、`style` 属性，甚至添加/删除样式表，实现动态交互|
|**安全联动**|攻击者可能利用 [[JavaScript]] 动态修改 CSS，隐藏恶意内容或伪造界面（如假登录框）。CSS 的 `url()` 属性可用于 CSRF 或 SSRF 探测。|

**渗透测试中关注 CSS 的点**：
- 检查 `background-image: url()` 是否引用外部资源，可能存在 SSRF。
- 检查 `@import` 是否加载远程样式表，可能引入恶意内容。
- 利用 CSS 属性选择器 `[value^="admin"]` 可配合 [[JavaScript]] 窃取输入内容（但现代浏览器有防护）。
- 隐藏元素（`display: none`）中可能存放敏感信息，如测试环境路径、隐藏的 API 接口。

---

# 九、常见问题与速查

|问题|解决方案|
|---|---|
|水平垂直居中|Flexbox: `display: flex; justify-content: center; align-items: center;`|
|清除浮动|父元素 `overflow: hidden` 或使用伪类 `clearfix`|
|优先级冲突|内联样式 > ID > 类 > 元素，可通过 `!important` 强制，但不推荐|
|移动端适配|使用媒体查询 + `viewport` 标签|

---

## 十、快速回顾清单

- 三种引入方式：外部、内部、内联
- 选择器：元素、类、ID、属性、后代、子、伪类
- 盒模型：content → padding → border → margin
- 布局：常规流、Flex、Grid、定位（relative/absolute/fixed/sticky）
- 常用属性：颜色、字体、圆角、阴影、过渡
- 响应式：媒体查询、视口设置
- 与 [[HTML]]、[[JavaScript]] 的协作关系
- 安全视角：隐藏元素、远程资源加载、伪类利用