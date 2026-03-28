## 一、HTML 是什么

- **超文本标记语言**，负责网页的 **结构**。
- 不是编程语言，而是通过标签来描述内容（标题、段落、链接、图片等）。
- 浏览器解析 HTML，将其渲染成可视化页面。

---
## 二、基本文档结构

```html
<!DOCTYPE html>          <!-- 声明文档类型，告诉浏览器用标准模式渲染 -->
<html lang="zh-CN">      <!-- 根元素，lang 属性声明主要语言 -->
<head>                   <!-- 头部：元数据，不显示在页面中 -->
    <meta charset="UTF-8">   <!-- 字符编码，必须写 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0"> <!-- 移动端适配 -->
    <title>页面标题</title>   <!-- 浏览器标签栏显示的标题 -->
    <link rel="stylesheet" href="style.css"> <!-- 外部 CSS -->
</head>
<body>                   <!-- 主体：页面可见内容 -->
    <h1>一级标题</h1>
    <p>段落内容</p>
</body>
</html>
```
