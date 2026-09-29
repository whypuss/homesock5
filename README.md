# homesock5

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

把 VPN Gate 的公共节点变成本地 **免密 SOCKS5 端口**：一个端口一个出口 IP，无需用户名和密码，直接 `IP:端口` 直连。
同时支持给每个出口挂载 Xray 节点链接（VLESS / VMess / Trojan），客户端连哪个端口就从哪个国家出去。

![主界面](https://images.joeyblog.net/2026/7/27/fanout-dashboard.png)

四条隧道跑在一台机器上，四个端口对应四个国家的出口，母机自己的 IP 不受影响：

![出口验证](https://images.joeyblog.net/2026/7/26/fanout-6-exit-ip.png)

---

## 特性

- ⚡ **SOCKS5 免密直连**：原生移除繁琐的 SOCKS5 用户名与口令校验，客户端直接填写 `IP` 与 `端口` 即可使用。
- 🌍 **多国出口扇出**：日本、美国、韩国、台湾等多国出口并行运行。
- 🔄 **自动故障转移**：每 10 秒自动健康检查，节点掉线自动无缝切换，本地监听端口保持不变。
- 🛡️ **Network Namespace 隔离**：每个节点独立 netns 与 OpenVPN 隧道，互不干扰，不影响宿主机网络。
- 🖥️ **精简 Web 管理面板**：一键新建出口、导出节点链接、无密直连复制。

---

## 原理

每个节点跑在独立的 network namespace 里，netns 内启动官方 openvpn 客户端。
SOCKS5 监听在母机，出站连接用 `setns` 切进对应 netns 建立。

这样做的好处：VPN 的路由劫持只影响自己的 netns，不会切断母机的网络；多个节点互不干扰，各自一个出口 IP。

```
客户端 ──> 母机 SOCKS5 :随机端口 (免密) ──> netns foN ──> openvpn ──> VPN Gate 节点
```

---

## 安装

需要 root，Linux（依赖 netns 与 TUN 模块）。

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/whypuss/homesock5/main/install.sh)
```

依赖（openvpn / curl / openssl / iproute / iptables）会按发行版自动装。

**Alpine** 默认不带 bash，先装一下：
```bash
apk add bash curl
bash <(curl -fsSL https://raw.githubusercontent.com/whypuss/homesock5/main/install.sh)
```

> **注意**：宿主环境必须放开 `/dev/net/tun`。

---

## 管理与使用

装完敲 `f` 打开管理菜单：

![管理菜单](https://images.joeyblog.net/2026/7/26/fanout-7-menu.png)

安装完成后会打印 Web 管理界面地址与访问口令：
```text
管理界面  http://<你的IP>:8899/<随机路径>/
访问口令  <随机口令>
```

### 快捷指令
```bash
f info       # 查看面板地址与连接信息
f list       # 查看当前隧道出口与 SOCKS5 端口
f restart    # 重启服务
f log        # 跟踪日志
f update     # 更新到最新版
f uninstall  # 卸载
```

---

## 界面预览

![新建出口](https://images.joeyblog.net/2026/7/27/fanout-wizard.png)

![节点详情](https://images.joeyblog.net/2026/7/27/fanout-detail.png)

---

## 许可

[MIT](LICENSE)。
节点来自 [VPN Gate](https://www.vpngate.net/)（筑波大学的学术实验项目）。
