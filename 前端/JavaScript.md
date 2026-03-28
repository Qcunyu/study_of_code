 > JavaScript 负责页面的**行为与交互**，与 [[HTML]] 定义结构、[[CSS]] 控制样式共同构成前端三件套。  
> 本笔记涵盖核心语法、DOM 操作、事件、异步编程，以及渗透测试中需关注的前端安全点。

---

# 一、JavaScript 是什么

- 轻量级解释型脚本语言，运行在浏览器或 Node.js 环境中。
- 在浏览器端，JS 可操作 DOM（文档对象模型）、处理用户事件、发送网络请求、存储数据等。
- 安全视角：JS 是 XSS（跨站脚本）攻击的核心载体，也常用于前端验证绕过、信息泄露。

---

# 二、引入方式

|方式|示例|说明|
|---|---|---|
|**外部脚本**|`<script src="script.js" defer></script>`|推荐，分离行为与结构，可缓存，`defer` 延迟执行直到 HTML 解析完成|
|**内部脚本**|`<script> console.log('hello'); </script>`|写在 HTML 中，可在 `<head>` 或 `<body>` 末尾|
|**内联事件**|`<button onclick="alert('hi')">`|可维护性差，但可能在渗透测试中发现测试代码|
**注意**：`<script>` 默认会阻塞 HTML 解析，建议放在 `</body>` 前或使用 `async/defer`。

---

# 三、变量与数据类型

## 1. 变量声明

| 关键字     | 作用域   | 特点                  |
| ------- | ----- | ------------------- |
| `var`   | 函数作用域 | 存在变量提升，可重复声明，不推荐使用  |
| `let`   | 块级作用域 | 推荐，不可重复声明           |
| `const` | 块级作用域 | 声明常量，引用不可变，对象属性仍可修改 |
## 2. 基本数据类型
- `string`, `number`, `boolean`, `null`, `undefined`, `symbol` (ES6), `bigint`
- 类型判断：`typeof` 运算符

### 3. 引用类型
- `object`：普通对象 `{}`、数组 `[]`、函数 `function`
- 判断数组：`Array.isArray(arr)`

### 4. 类型转换
- 隐式转换：`'5' - 3` → 2，`'5' + 3` → '53'（字符串拼接）
- 显式转换：`Number()`、`String()`、`Boolean()`、`parseInt()`、`parseFloat()`

---

# 四、运算符与流程控制

- **算术、比较、逻辑、三元**：`&&`、`||`、`!`、`??`（空值合并）、`?.`（可选链）
- **条件语句**：`if...else`、`switch`
- **循环**：`for`、`while`、`do...while`、`for...of`（遍历可迭代对象）、`for...in`（遍历对象属性）

---

# 五、函数

## 1. 函数声明
```javascript
function add(a, b) {
    return a + b;
}
```
## 2. 函数表达式
```javascript
const add = function(a, b) { return a + b; };
```
## 3. 箭头函数 (ES6)
```javascript
const add = (a, b) => a + b;
// 注意：箭头函数没有自己的 this，不绑定 arguments
```
## 4. 默认参数
```javascript
function greet(name = 'Guest') { ... }
```
## 5. 高阶函数
- 函数作为参数或返回值，常见于 `map`、`filter`、`reduce`、`setTimeout` 等。

---

## 六、对象与数组

### 1. 对象操作
```javascript
const obj = { name: 'Alice', age: 25 };
obj.name        // 点语法
obj['name']     // 方括号，可用于动态属性
delete obj.age  // 删除属性
'name' in obj   // 检查属性是否存在
```
### 2. 数组常用方法

|方法|说明|
|---|---|
|`push()` / `pop()`|末尾增删|
|`unshift()` / `shift()`|开头增删|
|`map()`|映射新数组|
|`filter()`|过滤|
|`reduce()`|累计计算|
|`forEach()`|遍历|
|`find()` / `findIndex()`|查找元素/索引|
|`includes()`|是否包含|
|`join()`|转为字符串|
|`slice()` / `splice()`|切片/修改原数组|

---

## 七、DOM 操作（与 [[HTML]] 交互）

DOM（文档对象模型）将 HTML 文档表示为树形结构，JS 可动态修改。

### 1. 选择元素

javascript

复制

下载

document.getElementById('id')
document.querySelector('.class')      // 返回第一个匹配
document.querySelectorAll('div')       // 返回 NodeList

### 2. 修改内容与属性

javascript

复制

下载

element.innerHTML = '<strong>hi</strong>';   // 解析 HTML，有 XSS 风险
element.textContent = 'hi';                  // 纯文本，安全
element.setAttribute('class', 'active');
element.getAttribute('data-id');
element.classList.add('hidden');

### 3. 创建与插入元素

javascript

复制

下载

const newDiv = document.createElement('div');
document.body.appendChild(newDiv);
parent.insertBefore(newDiv, referenceNode);

### 4. 删除元素

javascript

复制

下载

element.remove();

### 5. 事件监听

javascript

复制

下载

const btn = document.querySelector('button');
btn.addEventListener('click', (event) => {
    console.log(event.target);
});
// 常见事件：click, submit, input, keyup, load, DOMContentLoaded

**安全视角**：

- `innerHTML` 直接插入用户输入会导致 **XSS**，优先使用 `textContent`。
    
- 动态添加的 `<script>` 不会执行，但可通过 `eval` 或 `Function` 构造函数执行代码，这些都是危险操作。