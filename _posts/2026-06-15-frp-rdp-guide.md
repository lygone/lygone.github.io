---
layout: post
title: 用 FRP + RDP 实现家里电脑远程公司电脑（香港 VPS 中转）
tags: [FRP, RDP, 远程桌面, VPS, 教程]
category: 教程
description: 家里电脑没有公网 IP，公司电脑也没有公网 IP，两边都在 NAT 后面——用一台香港 VPS + FRP 做中转隧道，直接 mstsc 远程桌面连接，画质无损、延迟低、免费不限速。
---

## 背景

家里电脑想远程公司电脑，但两边都在 NAT 后面，没有公网 IP，也没有做内网穿透的条件。用 TeamViewer/AnyDesk 等第三方软件画质差、延迟高、还有商用检测限制。

解决方案：**用一台香港 VPS 做中转，FRP 转发 RDP 流量**，家里电脑直接 mstsc 远程桌面连接，画质无损、延迟低、不限速、不限商用。

## 整体架构

```
家里电脑 ──mstsc──→ HK VPS:23389 ──frp转发──→ 公司电脑:3389
                     ↑ frps 服务端              ↑ frpc 客户端
```

## 准备工作

- 一台香港 VPS（已用于科学上网的即可，FRP 只占 5-15MB 内存）
- 公司电脑（Windows，开启远程桌面）
- 家里电脑（Windows，自带 mstsc）

## 第一步：HK VPS 部署 frps（服务端）

### ① 下载 frp

SSH 连上你的香港 VPS：

```bash
ssh root@你的香港VPS_IP
```

下载 frp（去 [GitHub Releases](https://github.com/fatedier/frp/releases) 查最新版本号）：

```bash
wget https://github.com/fatedier/frp/releases/download/v0.61.0/frp_0.61.0_linux_amd64.tar.gz
tar xzf frp_0.61.0_linux_amd64.tar.gz
mv frp_0.61.0_linux_amd64 /opt/frp
cd /opt/frp
rm frpc frpc.toml  # 服务端不需要客户端文件
```

### ② 配置 frps

```bash
cat > /opt/frp/frps.toml <<'EOF'
bindPort = 7000

auth.method = "token"
auth.token = "改成你自己的随机密码"
EOF
```

### ③ 防火墙放行端口

**先确认 VPS 用的什么防火墙：**

```bash
which ufw firewalld iptables 2>/dev/null
```

**根据结果三选一执行：**

```bash
# 选项 A：ufw（Ubuntu 默认）
ufw allow 7000/tcp comment 'frp控制端口'
ufw allow 23389/tcp comment 'frp-rdp转发端口'

# 选项 B：firewalld（CentOS/Rocky 默认）
firewall-cmd --permanent --add-port=7000/tcp
firewall-cmd --permanent --add-port=23389/tcp
firewall-cmd --reload

# 选项 C：iptables 裸管理
iptables -A INPUT -p tcp --dport 7000 -j ACCEPT
iptables -A INPUT -p tcp --dport 23389 -j ACCEPT
iptables-save > /etc/iptables/rules.v4
```

> ⚠️ **同时去云厂商控制台**（阿里云/腾讯云/华为云等）→ 安全组/防火墙 → 入方向放行 **7000** 和 **23389** 端口的 TCP。

### ④ 启动 frps

```bash
# 测试运行
/opt/frp/frps -c /opt/frp/frps.toml
# 看到 "frps started successfully" → Ctrl+C 停止
```

注册为 systemd 服务（开机自启）：

```bash
cat > /etc/systemd/system/frps.service <<'EOF'
[Unit]
Description=FRP Server
After=network.target

[Service]
Type=simple
ExecStart=/opt/frp/frps -c /opt/frp/frps.toml
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now frps
systemctl status frps  # 确认 running
```

## 第二步：公司电脑部署 frpc（客户端）

### ① 下载 frpc

去 [GitHub Releases](https://github.com/fatedier/frp/releases) 下载 Windows 版 `frp_0.61.0_windows_amd64.zip`。

解压到 `C:\frp`，只保留 `frpc.exe` 和 `frpc.toml`，其他删掉。

### ② 配置 frpc.toml

```toml
serverAddr = "你的香港VPS_IP"
serverPort = 7000
auth.token = "跟服务端一致的密码"

[[proxies]]
name = "rdp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 3389
remotePort = 23389
```

### ③ 启动 frpc

**测试运行：**

```cmd
cd C:\frp
frpc.exe -c frpc.toml
```

看到 `start proxy success` → 通了，**直接最小化窗口锁屏走人**，到家就能连。

**想让 frpc 开机自启不弹窗，用 nssm 注册为服务：**

下载 [nssm](https://nssm.cc/download) → 解压出 `nssm.exe` 丢到 `D:\frp\` 目录，管理员 CMD 执行：

```cmd
D:\frp\nssm.exe install frpc
```

弹出窗口填三项：
- **Path**: `D:\frp\frpc.exe`
- **Startup directory**: `D:\frp`
- **Arguments**: `-c D:\frp\frpc.toml`

点 Install service，然后：

```cmd
nssm start frpc
```

以后开机自动后台运行，无窗口无图标。

> **想更省事？** 在桌面新建一个 `启动frp.bat` 快捷方式，内容为：
> ```bat
> @echo off
> cd /d D:\frp
> frpc.exe -c frpc.toml
> pause
> ```
> 双击启动，窗口显示 `start proxy success` 即为通。
> ```vbs
> CreateObject("WScript.Shell").Run "C:\frp\frpc.exe -c C:\frp\frpc.toml", 0, False
> ```
> 丢进 `shell:startup`（Win+R 输入这个路径打开启动文件夹）。

### ④ 开启公司电脑远程桌面

```
设置 → 系统 → 远程桌面 → 启用远程桌面
```

防火墙默认已放行 3389，如果之前关过：

```cmd
netsh advfirewall firewall add rule name="RDP" dir=in action=allow protocol=TCP localport=3389
```

## 第三步：家里电脑连接

```
Win+R → mstsc
计算机: 你的香港VPS_IP:23389
用户名: 公司电脑的开机账号
密码:   公司电脑的开机密码
```

首次连接提示证书不匹配 → 点是继续。

## 端口汇总

| 端口 | 方向 | 在哪放行 | 用途 |
|------|------|----------|------|
| 7000 TCP | 公司电脑 → VPS | VPS 安全组 + 系统防火墙 | frp 控制连接 |
| 23389 TCP | 家里电脑 → VPS | VPS 安全组 + 系统防火墙 | RDP 转发入口 |
| 3389 TCP | 公司电脑本地 | 公司电脑防火墙 | 远程桌面服务 |

## 排查指南

| 现象 | 原因 |
|------|------|
| 家里连上去提示连接超时 | VPS 安全组或系统防火墙 23389 没放行 |
| 家里连上去提示连接被拒绝 | 公司电脑 frpc 没运行，或 remotePort 不对 |
| frpc 日志报 `login to server failed` | VPS 安全组 7000 没放行，或 token 两端不一致 |
| frpc 日志报 `start proxy error` | 公司电脑 3389 端口未开启或被占用 |

**公司电脑上检查 3389：**

```cmd
netstat -ano | findstr :3389
```

有 LISTENING → 正常。

**VPS 上检查 frps：**

```bash
ss -tlnp | grep -E "7000|23389"
```

都 LISTEN → 正常。

## 资源占用

frps 在 VPS 上只占约 **5-15MB 内存**，CPU 几乎为 0。跟 Clash / v2ray 等代理工具共存完全无压力。

## 为什么选这个方案

- ✅ **画质无损** — RDP 原生协议，非第三方远控软件的有损压缩
- ✅ **延迟低** — 香港 VPS 到国内通常 30-80ms，比 TeamViewer/AnyDesk 流畅得多
- ✅ **文件拖拽传输** — RDP 原生支持，无需额外传文件
- ✅ **剪贴板共享** — 复制粘贴无缝同步
- ✅ **静默运行** — frpc 注册为服务后无窗口、无图标，公司 IT 难以发现
- ✅ **免费不限速** — frp 开源免费，VPS 是本来的，不额外花钱
- ✅ **加密传输** — frp 控制通道带 token 鉴权，数据通道走 TCP

## 对比其他方案

| 方案 | 画质 | 延迟 | 费用 | 隐蔽性 |
|------|------|------|------|--------|
| TeamViewer | 有损 | 中 | 商用收费 | 低（显眼界面） |
| AnyDesk | 有损 | 中 | 商用收费 | 低 |
| RustDesk 自建 | 有损 | 低 | 需 VPS | 中 |
| **FRP + RDP** | **无损** | **最低** | **仅 VPS 费用** | **最高** |
| WireGuard + RDP | 无损 | 最低 | 仅 VPS 费用 | 中（UDP 特征） |
