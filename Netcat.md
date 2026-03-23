## 一、概念

Netcat（简称 `nc`）是一款命令行网络工具，被誉为“TCP/IP 的瑞士军刀”[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-4](https://developer.aliyun.com/article/1691317)。它可以在网络中读写数据，支持 TCP、UDP 和 UNIX 域套接字[详细文档](https://manpages.debian.org/testing/netcat-openbsd/nc_openbsd.1.en.html)

**为什么叫“瑞士军刀”？**

- 体积小巧（可执行文件仅约 200KB）
- 功能灵活多样：端口扫描、文件传输、聊天服务器、反向 Shell、隧道代理等[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)
- 几乎所有 Linux 发行版都默认安装，Windows 也有移植版本

**主要应用场景**：

- 网络调试与故障排查
- 渗透测试中的端口扫描、反弹 Shell、数据窃取
- 临时文件传输（当 FTP/SCP 不可用时）
- 简易代理和隧道搭建[-3](https://developer.aliyun.com/article/1684112)

## 二、工作原理（本质）

Netcat 的核心原理非常简单：**在两个网络端点之间建立连接，然后将标准输入的数据发送出去，并将收到的数据输出到标准输出**[-5](https://cyberpanel.net/blog/netcat-command-in-linux#:~:text=Let's%20dive%20in!-,What%20is%20Netcat%20Command%20in%20Linux?,get%20access%20to%20the%20remote.)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)。



**两种工作模式**[-2](https://manpages.debian.org/testing/netcat-openbsd/nc_openbsd.1.en.html)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)：

1. **客户端模式**：主动连接远程主机的指定端口
    
    bash
    
    复制
    
    下载
    
    nc target.com 80
    
2. **服务端模式**：在本地监听端口，等待连接
    
    bash
    
    复制
    
    下载
    
    nc -l -p 8888
    

**数据流向的本质**：你可以把它理解为 `cat` 命令的网络版——`cat` 在本地文件间复制数据，而 `nc` 在网络连接间复制数据[-1](https://cloud.tencent.com.cn/developer/article/2473236?from=15425&frompage=seopage)[-8](https://www.eccouncil.org/cybersecurity-exchange/ethical-hacking/mastering-netcat-in-penetration-testing-a-step-by-step-tutorial/)。当两端都连接后，任何一端输入的内容会实时出现在另一端，形成双向通信。