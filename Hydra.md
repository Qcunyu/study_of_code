# 一、概念

**Hydra**（又名九头蛇）是一款开源的网络登录暴力破解工具，由著名的黑客组织THC（The Hacker's Choice）开发[-5](https://rivers.chaitin.cn/blog/cqjeskh0lnedo7thq0lg)[百度百科](https://baike.baidu.com/item/Hydra/2878158)。它被设计用来测试各种网络服务的安全性，通过自动化尝试大量的用户名和密码组合，验证是否存在弱口令漏洞。

**主要应用场景**：
- **渗透测试**：验证**SSH**、FTP、RDP等服务的弱口令风险
- **安全审计**：评估企业内部系统的密码强度
- **授权测试**：在获得授权后，模拟攻击者进行暴力破解
- **CTF比赛**：快速爆破题目中的服务登录凭证

**⚠️ 重要声明**：Hydra只能用于**合法授权的测试环境**。未经授权对他人系统进行暴力破解属于违法行为，可能面临法律制裁[-2](https://www.juhe.cn/news/index/id/11717)[-5](https://rivers.chaitin.cn/blog/cqjeskh0lnedo7thq0lg)。

# 二、工作原理（本质）

Hydra的核心是**并行化字典攻击**：利用预定义的字典（用户名列表、密码列表），以多线程方式同时对目标服务发起认证请求，根据响应判断凭证是否正确。

**三种攻击模式**：

|模式|说明|效率|适用场景|
|---|---|---|---|
|纯字典攻击|使用预设字典逐一尝试|中等|已知用户名或常用弱口令|
|混合攻击|在字典词条前后添加数字/符号|较低|覆盖密码变形|
|完全暴力|穷举所有字符组合（如aaaa）|极低|密码极短且字符集有限|

Hydra不利用漏洞，而是直接攻击==认证机制本身的弱点==——**弱口令**。

# 三、环境要求与安装

### 3.1 操作系统支持
- **Kali Linux**：预装，开箱即用
- **Ubuntu/Debian**：`sudo apt install hydra`
- **macOS**：`brew install hydra`
- **Windows**：通过WSL或使用Cygwin版本
### 3.2 验证安装
```bash
hydra -h
```

# 四、基本使用方法（语法格式）

Hydra 的使用方式非常灵活，用户可以根据目标服务类型选择不同的模块。其基本语法如下：

```bash
hydra [选项] [目标地址] [服务]
```
其中，`[选项]` 包括用户名、密码文件、线程数等；`[目标地址]` 是要攻击的目标 IP 或域名；`[服务]` 是要破解的协议类型（如 ssh、ftp、rdp 等）。

**示例**：

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 4 ssh://192.168.1.100
```
**说明**：`-l root` 指定单个用户名，`-P` 指定密码字典，`-t 4` 设置4个线程，目标是SSH服务。

# 五、常用命令与选项
## 5.1 核心参数

| 参数          | 说明                             | 示例                 |
| ----------- | ------------------------------ | ------------------ |
| `-l`        | 指定单个用户名                        | `-l admin`         |
| `-L`        | 指定用户名字典文件                      | `-L users.txt`     |
| `-p`        | 指定单个密码                         | `-p 123456`        |
| `-P`        | 指定密码字典文件                       | `-P passwords.txt` |
| `-C`        | 使用“用户名:密码”格式的文件                | `-C creds.txt`     |
| `-M`        | 指定目标主机列表文件                     | `-M targets.txt`   |
| `-t`        | 并发线程数（默认16）                    | `-t 8`             |
| `-o`        | 输出结果到文件                        | `-o result.txt`    |
| `-v` / `-V` | 显示详细过程                         | `-vV`              |
| `-f`        | 找到第一个有效凭证后停止                   | `-f`               |
| `-s`        | 指定非标准端口                        | `-s 2222`          |
| `-e nsr`    | 额外尝试：n(空密码)、s(用户名即密码)、r(反向用户名) | `-e ns`            |
| `-x`        | 自定义密码生成规则                      | `-x 4:4:a`         |
| `-w`        | 设置超时时间（秒）                      | `-w 10`            |
| `-R`        | 恢复中断的破解任务                      | `-R`               |

## 5.2 支持的服务协议（部分）

|协议|示例|
|---|---|
|ssh|`hydra -l root -P pass.txt ssh://192.168.1.1`|
|ftp|`hydra -L users.txt -P pass.txt ftp://192.168.1.1`|
|rdp|`hydra -l administrator -P pass.txt rdp://192.168.1.1`|
|mysql|`hydra -l root -P pass.txt mysql://192.168.1.1`|
|smb|`hydra -l admin -P pass.txt smb://192.168.1.1`|
|http-post-form|`hydra -L users.txt -P pass.txt 192.168.1.1 http-post-form`|
|telnet|`hydra -L users.txt -P pass.txt telnet://192.168.1.1`|
### 📌 为什么 Hydra 要区分这么多协议？
Hydra 支持超过 50 种协议，是因为**不同的网络服务使用各自独立的认证协议**（如 SSH 用密钥交换，HTTP 用表单提交，RDP 用远程桌面协议）。每种协议的握手方式、认证数据格式、成功/失败判断依据都截然不同，因此 Hydra 必须为每种协议单独实现“对话逻辑”。

**常见协议的使用场景举例**：

| 协议                 | 典型场景               | 什么时候用                               |
| ------------------ | ------------------ | ----------------------------------- |
| **ssh**            | Linux/Unix 服务器远程管理 | 目标开放 22 端口，需要爆破系统登录口令               |
| **ftp**            | 文件传输服务             | 目标开放 21 端口，用于获取网站或备份文件权限            |
| **rdp**            | Windows 远程桌面       | 目标开放 3389 端口，尝试获取 Windows 服务器或办公机权限 |
| **mysql**          | MySQL 数据库          | 目标开放 3306 端口，爆破数据库 root 或业务账号       |
| **smb**            | Windows 文件共享       | 内网中尝试获取 Windows 主机本地管理员口令           |
| **http-post-form** | Web 应用登录框          | 任何有登录表单的网站（OA、后台、邮箱等），是最灵活的爆破方式     |
| **redis**          | 内存数据库              | 目标开放 6379 端口，若未授权访问可进一步利用，有时需爆破密码   |
| **telnet**         | 老旧设备远程管理           | 路由器、交换机或工控设备，通常弱口令较多                |

**选择原则**：**先用端口扫描**（如 nmap）**确定目标开放了哪些服务**，再选择对应的协议进行爆破。==不同协议对应不同的攻击面==，选错协议则无法正确认证。

# 六、进阶用法

## 6.1 SSH服务爆破
```bash
# 已知单个用户名
hydra -l root -P /usr/share/wordlists/rockyou.txt -t 6 -vV -f ssh://192.168.1.100
# 用户名字典 + 密码字典
hydra -L users.txt -P passwords.txt -t 4 -vV ssh://192.168.1.100
# 指定非标准端口
hydra -l admin -P pass.txt -s 2222 ssh://192.168.1.100
```
## 6.2 FTP服务爆破
```bash
hydra -L users.txt -P pass.txt ftp://192.168.1.100
```
## 6.3 RDP（远程桌面）爆破

```bash
hydra -l administrator -P pass.txt rdp://192.168.1.100 -t 4 -vV
```
## 6.4 Web表单爆破（http-post-form）

需先分析目标表单结构，获取URL、POST数据和失败特征。
```bash
hydra -L users.txt -P pass.txt 192.168.1.100 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:Invalid login"
```
- `/login.php`：表单提交路径
- `username=^USER^&password=^PASS^`：POST数据，`^USER^`和`^PASS^`为占位符
- `Invalid login`：登录失败时页面包含的特征字符串

**如何获取表单信息**：  
浏览器F12 → 查看 `<form>` 标签的 `action` 和 `method` → 记录各 `<input>` 的 `name` 属性。
## 6.5 密码自动生成（`-x`）
```bash
# 生成所有4位纯小写字母密码（aaaa~zzzz）
hydra -l admin -x 4:4:a ssh://192.168.1.100
# 生成5-8位大小写字母+数字
hydra -l admin -x 5:8:aA1 ssh://192.168.1.100
```

语法：`MIN:MAX:CHARSET`
- `MIN`：最小长度
- `MAX`：最大长度
- `CHARSET`：a=小写字母，A=大写字母，1=数字
## 6.6 批量目标扫描（`-M`）
```bash
# targets.txt 每行一个IP或域名，可带端口
hydra -l admin -P pass.txt -M targets.txt ssh -t 4
```
## 6.7 恢复中断任务（`-R`）

如果破解过程中断，Hydra会生成恢复文件，下次用 `-R` 继续。