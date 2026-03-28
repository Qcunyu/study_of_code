# 一、HTML 是什么

- **超文本标记语言**，负责网页的 **结构**。
- 不是编程语言，而是通过标签来描述内容（标题、段落、链接、图片等）。
- 浏览器解析 HTML，将其渲染成可视化页面。

---
# 二、基本文档结构

```html
<!DOCTYPE html>          <!-- 声明文档类型，告诉浏览器用标准模式渲染 -->
<html lang="zh-CN">      <!-- 根元素，lang 属性声明主要语言 -->
<head>                   <!-- 头部：元数据，不显示在页面中 -->
    <meta charset="UTF-8">   <!-- 字符编码，必须写 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- 移动端适配 -->
    <title>页面标题</title>   <!-- 浏览器标签栏显示的标题 -->
    <link rel="stylesheet" href="style.css"> <!-- 外部 [[CSS]] -->
    <script src="script.js" defer></script>  <!-- 外部 [[JavaScript]] -->
</head>
<body>                   <!-- 主体：页面可见内容 -->
    <h1>一级标题</h1>
    <p>段落内容</p>
</body>
</html>
```

---
# 三、常用标签分类

## 1. 块级元素（占满整行，可设置宽高）

| 标签                    | 作用                       |
| --------------------- | ------------------------ |
| `<h1>`~`<h6>`         | 标题，`<h1>` 最重要，一个页面建议只用一次 |
| `<p>`                 | 段落                       |
| `<div>`               | 通用容器，无语义，布局主力            |
| `<header>`、`<footer>` | 页眉、页脚                    |
| `<nav>`               | 导航栏                      |
| `<main>`              | 主内容区                     |
| `<section>`           | 章节                       |
| `<article>`           | 独立文章                     |
| `<ul>`、`<ol>`、`<li>`  | 无序/有序列表                  |
| `<table>`             | 表格                       |
| `<form>`              | 表单区域                     |

## 2. 行内元素（不换行，宽高由内容撑开）

|标签|作用|
|---|---|
|`<span>`|通用行内容器，无语义|
|`<a href="...">`|超链接|
|`<img src="..." alt="...">`|图片|
|`<strong>` / `<b>`|加粗（strong 有强调语义）|
|`<em>` / `<i>`|斜体（em 有强调语义）|
|`<br>`|换行|
|`<input>`、`<label>`|表单控件（行内块）|

## 3. 语义化标签（让结构清晰，利于 SEO 和可访问性）

- `<header>`、`<nav>`、`<main>`、`<section>`、`<article>`、`<aside>`、`<footer>`
- 尽量少用 `<div>` 和 `<span>` 代替语义标签。

---

# 四、常用属性

|属性|适用标签|说明|
|---|---|---|
|`id`|几乎所有|唯一标识，用于 [[CSS]] 或 [[JavaScript]] 选中|
|`class`|几乎所有|类名，可多个，用于样式复用|
|`style`|几乎所有|内联样式（尽量避免）|
|`data-*`|几乎所有|自定义数据属性，如 `data-id="123"`，常与 [[JavaScript]] 配合|
|`src`|`<img>`, `<script>`, `<iframe>`|资源地址|
|`href`|`<a>`, `<link>`|链接地址|
|`alt`|`<img>`|图片无法显示时的替代文本（必须）|
|`target`|`<a>`|`_blank` 新窗口打开；`_self` 当前窗口|
|`rel`|`<a>`|如 `noopener noreferrer` 安全属性|

---

# 五、表单（渗透测试重点关注）
表单是用户输入数据的主要方式，也是安全测试的关键点。
## 1. 基本结构
```html
<form action="login.php" method="POST">
    <label for="username">用户名：</label>
    <input type="text" name="username" id="username" required>
    <br>
    <label for="password">密码：</label>
    <input type="password" name="password" id="password">
    <br>
    <input type="hidden" name="token" value="abc123">   <!-- 隐藏字段 -->
    <button type="submit">登录</button>
</form>
```
## 2. 常见 input 类型

|`type`|说明|
|---|---|
|`text`|单行文本|
|`password`|密码（输入内容掩盖）|
|`email`|邮箱（会校验格式）|
|`number`|数字|
|`radio`|单选按钮（同 name 一组）|
|`checkbox`|复选框|
|`file`|文件上传|
|`hidden`|隐藏字段，不在页面显示，但会随表单提交|
|`submit`|提交按钮|

## 3. 其他表单标签
- `<select>` + `<option>`：下拉选择框
- `<textarea>`：多行文本域
## 4. 渗透测试中关注点

- **`action` 属性**：表单提交的目标 URL，可能指向其他域或内网地址。
- **`method`**：`GET` 或 `POST`。GET 参数会在 URL 中显示，容易被浏览器记录；POST 相对隐蔽。
- **`hidden` 字段**：可能存储 token、用户 ID、权限标识，可尝试篡改。
- **`required` 属性**：前端验证，可被绕过（直接构造请求）。绕过方式常借助 [[JavaScript]] 修改或直接用 [[Burp Suite]] 重放。
- **文件名**：`name` 属性是后端接收的参数名，可用来推测后端逻辑。

---

## 六、HTML 注释与信息泄露
```html
<!-- 这里写注释，有时开发人员会留下敏感信息，如 TODO、测试账号、路径 -->
```
- 在渗透测试中，**查看网页源码**（Ctrl+U）或使用 [[Burp Suite]] 可发现**隐藏注释**、**测试代码**、**后台路径**等。

---

## 七、与 [[CSS]]、[[JavaScript]] 的联系

|角色|说明|
|---|---|
|**HTML**|定义页面内容和结构，提供 DOM 元素|
|**[[CSS]]**|通过选择器（`#id`、`.class`、标签名）控制 HTML 元素的视觉表现|
|**[[JavaScript]]**|通过 DOM 操作动态修改 HTML 内容和属性，处理用户交互，表单验证，发送异步请求等|

在安全测试中：
- HTML 是信息收集的起点（源码、注释、表单）
- [[JavaScript]] 常常隐藏 API 接口、前端验证逻辑、敏感信息（如 token）
- [[CSS]] 一般安全风险较小，但可能用于钓鱼（隐藏元素）或配合 XSS 绕过 WAF

---

## 八、快速回顾清单

- 文档结构：`<!DOCTYPE>`、`<html>`、`<head>`、`<body>`
- 常用块级元素：`<div>`、`<p>`、`<h1>`~`<h6>`、`<ul>/<ol>`、`<form>`
- 常用行内元素：`<span>`、`<a>`、`<img>`、`<input>`
- 表单字段：`<input type="text/password/radio/checkbox/file/hidden">`、`<select>`
- 属性：`id`、`class`、`href`、`src`、`alt`、`data-*`、`name`、`value`
- 注释：`<!-- ... -->`，可能泄露信息
- 渗透视角：`action`、`method`、`hidden` 字段、前端验证、源码注释
- 与 [[CSS]]、[[JavaScript]] 的协作关系