XSS（跨站脚本攻击）与 [[SQL注入]] 同为 OWASP Top 10 中的经典 Web 漏洞，常与 [[Burp Suite]]、[[Hydra]] 等工具配合进行渗透测试。本笔记聚焦 XSS 的攻击方式、绕过技巧与防御策略。

---

# 一、概念

XSS（Cross-Site Scripting，跨站脚本攻击）是一种代码注入攻击。攻击者将恶意脚本注入到可信网站的页面中，当其他用户浏览该页面时，**恶意脚本**会在用户浏览器中执行[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。由于这些代码来自“可信”站点，浏览器会默认授予其访问 Cookie、会话令牌等敏感信息的权限[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

**核心危害**：**JavaScript 在浏览器中能做到的事情，XSS 攻击几乎都能做到。** 包括窃取 Cookie 实现会话劫持、执行未授权操作、键盘记录、钓鱼攻击、传播蠕虫等[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

XSS 是 CVE 数据库中报告最频繁的 Web 漏洞之一，长期位于 OWASP Top 10 榜单[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

**攻击本质**：应用程序的“交互性”要求将用户输入回显到页面上，当该输入未被安全处理时，浏览器会将其解析为可执行代码而非纯文本，从而导致攻击[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

---

## 二、攻击方式

XSS 攻击主要分为三类：**反射型**、**存储型**、**DOM 型**[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。前两种需要经过服务器，最后一种仅涉及客户端，由前端 JS 逻辑缺陷导致-。

### 2.1 反射型 XSS（Reflected XSS）

**攻击流程**：攻击者将恶意脚本附加到 URL 参数中，通过钓鱼等方式诱骗用户点击链接，服务器将参数“反射”回页面，浏览器执行恶意脚本[-3](https://cloud.baidu.com/article/3361360)[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

**数据流向**：前端 → 后端 → 前端（一次性反射）[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

**典型场景**：搜索框、URL 参数直接回显的页面。

**示例**：用户访问 `https://example.com/search?q=<script>alert('XSS')</script>`，若服务器未过滤，页面弹出警告框[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

**实战案例**：NASA 某子域名曾因 User-Agent 输入未过滤导致反射型 XSS，攻击者可利用该漏洞窃取会话 Cookie[-24](https://bugcrowd.com/disclosures/da0e3345-4a66-428f-a4d4-55f8dbf9c589/reflected-xss-exploit-chain-with-session-cookie-theft-on-nasa-subdomain)。

### 2.2 存储型 XSS（Stored XSS）

**攻击流程**：恶意脚本被永久存储在服务器（数据库或文件系统），当任何用户访问受影响页面时，脚本自动从服务器加载并执行[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)[-3](https://cloud.baidu.com/article/3361360)。

**数据流向**：前端 → 后端 → 存储 → 前端[-11](https://developer.baidu.com/article/detail.html?id=3361362)。

**典型场景**：留言板、评论区、用户资料、帖子内容。

**示例**：攻击者在评论区提交 `<img src="x" onerror="alert('XSS')"/>`，其他用户访问该页面时弹出警告框[-3](https://cloud.baidu.com/article/3361360)。

**实战案例**：富士通披露的 CVE-2024-42834 漏洞中，攻击者通过 API 在用户姓氏字段注入恶意脚本，管理员查看客户记录时脚本自动执行，可窃取会话令牌、冒充用户[-25](https://corporate-blog.global.fujitsu.com/apac/2025-08-26/01/)。

### 2.3 DOM 型 XSS（DOM-based XSS）

**攻击流程**：不依赖服务器，通过修改客户端 DOM 环境执行恶意代码，页面本身无变化，但 DOM 被恶意修改导致代码执行[-1](https://cloud.tencent.cn/developer/article/2437214?from=15425)。

**典型场景**：使用 `location.hash`、`document.referrer`、`document.URL` 等客户端源的 JS 逻辑。

**示例**：页面 JS 执行 `document.body.innerHTML = location.hash`，攻击者构造 URL `https://example.com/#<img src=x onerror=alert(1)>`，恶意代码被执行[-5](https://semgrep.dev/docs/learn/vulnerabilities/cross-site-scripting)。

### 2.4 三种类型对比

|类型|存储位置|是否需要服务器|持久性|危害范围|
|---|---|---|---|---|
|反射型|无（URL 中）|是|非持久（一次触发）|单个用户|
|存储型|服务器数据库|是|持久|所有访问用户|
|DOM 型|浏览器 DOM|否|视具体场景|取决于漏洞位置|