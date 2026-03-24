[传送门](https://fofa.info)
# 一、概念

FOFA（Fingerprint of Assets）是华顺信安推出的一款**网络空间资产搜索引擎**，被誉为“网络空间的谷歌地图”[-9](https://www.huashunxinan.net/product-fofa)。它通过对全球公网对外开放的资产进行主动探测、抓取和存储，建立起庞大的网络资产指纹库，使用户能够快速检索全球范围内的互联网资产[-6](https://www.nosec.org/home/detail/5080.html)。

**通俗理解**：就像谷歌地图能看到全球建筑的位置和结构，FOFA能看到全球的IP、网站、摄像头、数据库等网络资产的分布和特征。它主要针对**公网资产**进行探测，内网资产涉及隐私和合规问题不在其范围内[-6](https://www.nosec.org/home/detail/5080.html)。

**官方地址**：[https://fofa.info](https://fofa.info/)

**核心价值**：
- 企业梳理公网暴露面，减少攻击风险[-9](https://www.huashunxinan.net/product-fofa)
- 漏洞爆发时快速评估影响范围[-1](https://nosec.org/home/detail/1858.html)
- 渗透测试前收集目标资产信息
- 安全研究中的数据统计分析[-6](https://www.nosec.org/home/detail/5080.html)

# 二、能做什么？

|场景|说明|典型案例|
|---|---|---|
|**资产梳理**|帮助企业发现暴露在公网的未知资产|用证书信息找到某电商1.8万个IP[-1](https://nosec.org/home/detail/1858.html)|
|**漏洞影响评估**|新漏洞爆发时快速定位受影响资产|用`banner*="5.?.?"`定位MySQL 5.x.x版本[-5](https://nosec.org/home/detail/5071.html)|
|**绕过CDN找真实IP**|通过证书等信息绕过CDN防护|搜索证书找到域名真实IP[-1](https://nosec.org/home/detail/1858.html)[-10](https://bbs.huaweicloud.com/blogs/399063)|
|**威胁情报拓线**|从少量IOC发现更多恶意资产|追踪COLDRIVER组织C2[-8](https://nosec.org/home/detail/5127.html)|
|**寻找有趣资产**|摄像头、矿机、Burp代理等|找到埃菲尔铁塔的公开摄像头[-1](https://nosec.org/home/detail/1858.html)
# 三、基本使用方法（语法格式）

FOFA的使用方式类似于谷歌或百度，用户可以输入关键词来匹配包含该关键词的数据。不同的是，这些数据不仅包括网页，还包括摄像头、打印机、数据库、操作系统等资产[-6](https://www.nosec.org/home/detail/5080.html)。

基本语法结构如下：
```text
[语法类型]="[关键词]"
```
**示例**：
```bash
title="后台管理"
```

**说明**：`title` 指定在网站标题中搜索，`"后台管理"` 是要匹配的关键词。

# 四、常用语法与选项

### 4.1 逻辑连接符

| 符号   | 含义   | 示例                                              | 说明                                                               |
| ---- | ---- | ----------------------------------------------- | ---------------------------------------------------------------- |
| `=`  | 匹配   | `title="后台"`                                    | 包含关键词即可                                                          |
| `==` | 完全匹配 | `title=="后台登录"`                                 | 必须完全相等                                                           |
| `&&` | 与    | `title="后台" && country="CN"`                    | 同时满足多个条件                                                         |
| `\|` | 或    | `port="80" \| port="443"`                       | 满足任一条件                                                           |
| `!=` | 不匹配  | `title!="404"`                                  | 排除包含该关键词的结果                                                      |
| `*=` | 模糊匹配 | `host*="*test*.baidu.com"`                      | 使用通配符（个人版及以上）[-5](https://nosec.org/home/detail/5071.html)       |
| `()` | 优先级  | `(title="admin" \| title="管理") && country="CN"` | 括号内优先运算[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0) |

### 4.2 基础类别语法

| 语法            | 作用                | 示例                   | 说明                                                                        |
| ------------- | ----------------- | -------------------- | ------------------------------------------------------------------------- |
| `title`       | 搜索网站标题            | `title="后台管理"`       | 查找标题包含该关键词的网站[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)    |
| `body`        | 搜索HTML正文          | `body="login"`       | 查找页面源码中包含该关键词的网站[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0) |
| `header`      | **==搜索HTTP响应头==** | `header="nginx"`     | 查找响应头包含nginx的资产[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)  |
| `host`        | 搜索主机名             | `host="login"`       | 查找主机名包含login的资产[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)  |
| `domain`      | 搜索根域名             | `domain="baidu.com"` | 查找该域名下的资产，包括子域名[-10](https://bbs.huaweicloud.com/blogs/399063)            |
| `ip`          | 搜索IP              | `ip="1.1.1.0/24"`    | 支持CIDR网段[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)         |
| `port`        | 搜索端口              | `port="443"`         | 查找开放指定端口的资产[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)      |
| `protocol`    | 搜索协议              | `protocol="https"`   | 查找使用该协议的服务[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)       |
| `server`      | 搜索服务器类型           | `server="Apache"`    | 查找使用该Web服务器的资产[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)   |
| `status_code` | 搜索HTTP状态码         | `status_code="403"`  | 查找返回特定状态码的页面[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)     |
| `icp`         | 搜索ICP备案号          | `icp="京ICP证030173号"` | 查找该备案号对应的网站[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)      |
| `os`          | 搜索操作系统            | `os="Windows"`       | 查找运行该OS的资产[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)       |
| `app`         | 搜索应用名称            | `app="weblogic"`     | 查找部署该应用的资产[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)       |

### 4.3 协议类语法（type=service）

|语法|作用|示例|
|---|---|---|
|`banner`|搜索服务banner|`banner="mysql version"`|
|`js_name`|搜索HTML中的JS文件|`js_name="jquery.js"`|

### 4.4 网站类语法（type=subdomain）

| 语法       | 作用    | 示例                     |
| -------- | ----- | ---------------------- |
| `host`   | 主机名搜索 | `host="www.baidu.com"` |
| `domain` | 域名搜索  | `domain="baidu.com"`   |

### 4.5 证书类语法

|语法|作用|示例|
|---|---|---|
|`cert`|搜索证书内容|`cert="Internet Widgits Pty Ltd"`|
|`cert.subject`|证书持有者|`cert.subject="Oracle"`|
|`cert.issuer`|证书颁发者|`cert.issuer="DigiCert"`|
|`cert.is_valid`|证书是否有效|`cert.is_valid=true`|
|`cert.is_expired`|证书是否过期|`cert.is_expired=false`|

### 4.6 地理位置语法（Location）

|语法|作用|示例|
|---|---|---|
|`country`|搜索国家|`country="CN"`（国家代码）[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)|
|`region`|搜索行政区|`region="Beijing"`[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)|
|`city`|搜索城市|`city="Beijing"`[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)|

### 4.7 时间类语法（Last update time）

|语法|作用|示例|
|---|---|---|
|`after`|指定时间之后|`after="2023-01-01"`|
|`before`|指定时间之前|`before="2024-01-01"`|

### 4.8 标记类语法（Special Label）

|语法|作用|说明|
|---|---|---|
|`is_honeypot`|蜜罐识别与过滤|`is_honeypot="false"`排除蜜罐（高级会员）[-2](https://www.nosec.org/home/detail/4763.html)|
|`is_domain`|是否有域名|`is_domain=true`筛选有域名的资产[-6](https://www.nosec.org/home/detail/5080.html)|
|`is_ipv6`|是否IPv6资产|`is_ipv6=true`[-10](https://bbs.huaweicloud.com/blogs/399063)|

### 4.9 独立IP语法
独立IP系列语法不可与上面其他语法共用[-3](https://rivers.chaitin.cn/blog/cqk3uk90lnebepb58kh0)：

| 语法         | 作用        | 示例             |
| ---------- | --------- | -------------- |
| `ip`       | 搜索指定IP    | `ip="1.1.1.1"` |
| `ip_ports` | 搜索IP开放的端口 | 配合ip使用         |
| `port`     | 搜索开放端口的IP | `port="80"`    |
# 五、进阶用法

## 5.1 模糊搜索（`*=`）

模糊搜索功能最初是为了解决分词不完整的问题，后来发现可用于更多特定搜索场景[-5](https://nosec.org/home/detail/5071.html)。**注意**：个人版及以上订阅会员或F点单功能消费可用[-5](https://nosec.org/home/detail/5071.html)。

**通配符说明**：
- `*`：匹配0个或多个任意字符
- `?`：匹配1个任意字符

**使用示例**：
```bash
# 查找子域名包含test的baidu.com资产
host*="test*.baidu.com"
# 查找MySQL 5.x.x版本
banner="mysql version" && banner*="5.?.?"
# 查找端口为4位数的资产
domain="fofa.info" && port*="????"
```
## 5.2 FID（Feature ID）- 资产特征ID
**什么是FID？**
FID是FOFA为每个网站生成的唯一特征标识，通过对网站的多个关键特征（页面结构、响应头、图标等）进行聚合计算得到[-6](https://www.nosec.org/home/detail/5080.html)。

**作用**：
- 即使网站换了域名、改了IP，只要网站模板相同，FID就相同
- 可以快速发现使用同一套建站模板的所有网站
- 常用于钓鱼网站拓线、恶意网站聚类
- 
**使用方法**：  
点击搜索结果页的FID标签即可自动查询，或直接用`fid="xxx"`语法[-6](https://www.nosec.org/home/detail/5080.html)。
## 5.3 规则集（预置指纹）

FOFA已经对大量常见应用建立了预置规则集（app指纹）。用户在搜索框中输入应用名称时，会自动推荐相关规则[-2](https://www.nosec.org/home/detail/4763.html)[-6](https://www.nosec.org/home/detail/5080.html)。

**查看方式**：
- 搜索框输入关键词，FOFA会列出已有的规则[-2](https://www.nosec.org/home/detail/4763.html)
- 规则列表和规则专题中查看[-2](https://www.nosec.org/home/detail/4763.html)
- 点击左侧筛选栏的规则条目自动添加语法[-10](https://bbs.huaweicloud.com/blogs/399063)

### 5.4 与 [[Netcat]] 配合使用

先用Netcat扫描目标端口，确定开放服务后，再用FOFA批量搜索同类资产进行漏洞影响评估。
### 5.5 与 [[Hydra]] 配合使用

通过FOFA搜索开放特定服务（如SSH、RDP）的资产，导出IP列表后用`[[Hydra]]`进行弱口令爆破。

# 六、实用场景举例

## 6.1 查找组件漏洞
```bash
app="Log4j2"                    # 可能存在log4j漏洞的组件
app="APACHE-ActiveMQ"           # ActiveMQ漏洞
app="Jenkins"                   # Jenkins漏洞
app="Struts2"                   # Struts2漏洞
app="APACHE-Shiro"              # Shiro漏洞
```
## 6.2 搜索文件上传接口
```bash
body="method=\"post\" enctype=\"multipart/form-data\"" && body="<input type=\"file\""
```
结合Form表单和文件上传标签特征，发现可进行文件上传的页面[-2](https://www.nosec.org/home/detail/4763.html)[-10](https://bbs.huaweicloud.com/blogs/399063)。

## 6.3 搜索密码框中的弱口令
```bash
body="value=\"123456\" && type=\"password\""
```
在前端代码中搜索密码框内预设的弱口令值[-2](https://www.nosec.org/home/detail/4763.html)。

## 6.4 绕过CDN找真实IP
```bash
cert="phpinfo.me"               # 通过证书搜索真实IP
```
网站有CDN时，可通过SSL证书信息找到源站真实IP[-1](https://nosec.org/home/detail/1858.html)[-10](https://bbs.huaweicloud.com/blogs/399063)。

## 6.5 资产拓线（APT追踪）
以COLDRIVER组织追踪为例，从已知C2地址出发[-8](https://nosec.org/home/detail/5127.html)：
```bash
# 初始IOC：45.133.216.15:3000
# 拓线一：端口+banner+证书
banner="\x15\x03\x03\x00\x02\x022" && port="3000" && cert="Internet Widgits Pty Ltd"
# 拓线二：证书有效期
cert="Internet Widgits Pty Ltd" && cert="2023-06-23 15:59 UTC"
# 最终语法
banner="\x15\x03\x03\x00\x02\x022" && port="3000" && cert="Internet Widgits Pty Ltd" && cert="2023-06-23 15:59 UTC"
```