---
layout: post
title: OpenP2P — 免费、开源、低延迟的远程桌面方案（比 ToDesk 会员更快）
tags: [远程桌面, OpenP2P, P2P打洞, RDP, 效率工具]
category: 教程
description: 放弃 ToDesk 会员，放弃 RustDesk 自建，放弃 ZeroTier 和 Tailscale——一套完全免费且比付费版更流畅的 P2P 远程桌面方案，实测全画质零延迟。
---

## 为什么选 OpenP2P？

市面上的远程桌面方案大致分三类：

| 方案 | 费用 | 速度 | 画质 | 痛点 |
|------|------|------|------|------|
| ToDesk / TeamViewer | 免费版限时 | 快（走机房中转） | 画质压缩 | 免费版阉割严重 |
| RustDesk 公共中继 | 免费 | 卡 | 中等 | 中继服务器在海外 |
| RustDesk 自建 | ¥68+/年 | 快 | 高 | 需要公网服务器 |
| ZeroTier / Tailscale + RDP | 免费 | 看打洞成功率 | 依赖 RDP | Windows 兼容性差，配置折腾 |
| **OpenP2P + RDP** | **免费** | **快** | **最高** | 5 分钟配置 |

[OpenP2P](https://openp2p.cn) 是国产开源项目，专为中国复杂网络环境优化，打洞成功率远超 Tailscale 和 ZeroTier。核心优势就一条：**流量直连不经过第三方服务器**，延迟极低，带宽拉满。

> 实测环境：双方电信 100Mbps 上行 / 32 位全画质 / UDP 打洞直连，延迟和局域网无异。

---

## 一、注册 & 安装

前往 [console.openp2p.cn](https://console.openp2p.cn) 注册账号（邮箱即可），进入设备管理页面，默认是空的：

![安装页面](/assets/img/openp2p-install.png)

点击**安装**按钮，选择 `Windows 64 位`，分别在**主控端和被控端**下载运行。不要改文件名，双击执行即可。

安装完成后刷新网页，两台设备会出现在控制台。编辑名称方便区分主控和被控：

![两台设备](/assets/img/openp2p-devices.png)

**共享带宽**统一设置为 **30 Mbps**（双方上行带宽 ≥ 100 Mbps 的情况下），别忘了点保存。

---

## 二、配置端口转发

最关键的一步：在主控端创建端口转发。

> 如果需要双向互控，两台设备各自添加一条转发规则即可。

点击**新建端口转发**：

![新建端口转发](/assets/img/openp2p-forward.png)

配置要点：

| 参数 | 设置 | 说明 |
|------|------|------|
| 目标设备 | 被控端 | 哪台能提供远程桌面服务 |
| 远程端口 | `3389` | Windows RDP 默认端口 |
| 本地端口 | `23389` | 可自定义，记着就行 |
| 协议 | **TCP** | RDP 基于 TCP |
| 打洞协议优先级 | **UDP 优先** | 打洞快、延迟低 |

为什么选 TCP 协议但 UDP 优先打洞？RDP 本身跑的是 TCP，但建立 P2P 隧道时用 UDP 打洞——UDP 打洞成功率高、延迟低、握手开销小。打洞成功后底层自动走最优协议。

---

## 三、连接

配置完重启服务（杀掉客户端重新运行），在主控端：

```
Win + R → mstsc → 127.0.0.1:23389
```

输入被控端 Windows 账号密码即可登录。

一次不行就重试，打洞偶尔需要第二次握手。连上之后就稳了，不会断。

---

## 四、Windows 家庭版

如果被控端是 Win10/Win11 **家庭版**，不自带远程桌面服务端，打补丁：

1. 下载 [RDPWrap](https://github.com/stascorp/rdpwrap/releases)
2. 解压后右键 `install.bat` → **以管理员身份运行**
3. 运行 `RDPConf.exe`，三个状态全绿色即搞定

如果显示红色，将 `C:\Program Files\RDP Wrapper\rdpwrap.ini` 替换为[最新版配置文件](https://raw.githubusercontent.com/sebaxakerhtc/rdpwrap.ini/master/rdpwrap.ini)。

---

## 五、推荐 RDP 设置

打开 `mstsc`，点左下角**显示选项**：

```
显示 → 颜色: 32 位
体验 → 选择: 局域网（10Mbps 或更高）
勾选: 桌面背景、字体平滑、窗口动画
```

全画质毫无压力。如果上行不足，调整体验为"低速宽带"即可。

---

## 六、常见问题

**Q: 连接不上？**
重试一两次。始终失败就把打洞协议优先级改回"TCP 优先"以牺牲一点延迟换稳定性。

**Q: 略有延迟？**
测上行带宽（[中科大测速站](https://test.ustc.edu.cn)），上行低于 5Mbps 时 RDP 降画质。跨运营商（如家里联通 + 公司电信）本身会有 30-60ms 延迟，正常现象。

**Q: 被控端必须设密码？**
是的，Windows RDP 不允许空密码远程登录。或者去 `secpol.msc` → 安全选项 → 禁用"空密码账户只允许控制台登录"。

**Q: 会收费吗？**
不会。OpenP2P 完全开源免费，流量直连不经过服务器，没有收费空间。担心官方服务不可用？服务端代码开源，自己搭一个 Docker 一条命令：

```bash
docker run -d --name openp2p-server openp2pcn/openp2p-server
```

然后把客户端配置文件里的 `ServerHost` 改成你自己的 IP 就行——只换了牵线人，数据依然是直连。

---

## 总结

| 组件 | 效果 |
|------|------|
| OpenP2P UDP 打洞 | P2P 直连，不经过第三方 |
| Windows RDP 原生 | 分辨率最高，不压缩画质 |
| 免费 | ¥0 |

一套配好，终身免费。比 ToDesk 会员更快，比 RustDesk 公共中继更稳，比 Tailscale 更好配。
