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

# 三、常用标签分类

## 1. 块级元素（占满整行，可设置宽高）

|标签|作用|
|---|---|
|`<h1>`~`<h6>`|标题，`<h1>` 最重要，一个页面建议只用一次|
|`<p>`|段落|
|`<div>`|通用容器，无语义，布局主力|
|`<header>`、`<footer>`|页眉、页脚|
|`<nav>`|导航栏|
|`<main>`|主内容区|
|`<section>`|章节|
|`<article>`|独立文章|
|`<ul>`、`<ol>`、`<li>`|无序/有序列表|
|`<table>`|表格|
|`<form>`|表单区域|

### 2. 行内元素（不换行，宽高由内容撑开）

|标签|作用|
|---|---|
|`<span>`|通用行内容器，无语义|
|`<a href="...">`|超链接|
|`<img src="..." alt="...">`|图片|
|`<strong>` / `<b>`|加粗（strong 有强调语义）|
|`<em>` / `<i>`|斜体（em 有强调语义）|
|`<br>`|换行|
|`<input>`、`<label>`|表单控件（行内块）|

### 3. 语义化标签（让结构清晰，利于 SEO 和可访问性）

- `<header>`、`<nav>`、`<main>`、`<section>`、`<article>`、`<aside>`、`<footer>`
    
- 尽量少用 `<div>` 和 `<span>` 代替语义标签。