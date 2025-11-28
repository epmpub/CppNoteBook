
在香港 VPS 上搭一个高性能、纯命令行的代理服务器，国内用户只需要一行命令或几行配置就能直连 GitHub、Docker Hub，且速度极快。

推荐方案（2025 年最稳最快）使用 Hysteria2 + Reality（QUIC 协议）或者 VLESS + Reality + Vision
优点：  

- 抗封锁能力极强（伪装成正常 HTTPS 流量）  
- 延迟低、速度高（香港到国内基本 10–30ms）  
- 不需要任何国内客户端软件（支持 Clash、v2rayN、Shadowrocket 等，也支持一行命令直连）  
- 配置极其简单，5 分钟搞定

方案一：最简单 5 分钟搭建（推荐新手）—— Hysteria2

bash



```bash
# 在香港 VPS（Ubuntu 22.04/24.04 或 Debian 12）一键脚本
bash <(curl -fsSL https://get.hy2.sh/)

# 脚本运行完会让你输入域名（可以填一个你自己的域名，也可以直接回车用 IP）
# 默认监听 443 端口，协议 hy2，使用 QUIC

# 完成后会输出类似信息：
Server address: 你的香港IP或域名:443
Password: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

国内用户使用方法（零客户端）：

1. 任意支持 Hysteria2 的工具（一行命令就行）
   Windows（PowerShell）：

   powershell

   

   ```powershell
   iwr https://github.com/apernet/hysteria/releases/download/app/v2.6.0/hysteria-windows-amd64.exe -OutFile hy2.exe
   .\hy2.exe client -c "hysteria2://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx@你的香港IP:443/?sni=www.bing.com"
   ```

   macOS（Terminal）：

   bash

   

   ```bash
   brew install hysteria
   hysteria2 client -c "hysteria2://xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx@你的香港IP:443/?sni=www.bing.com"
   ```

   Linux（任意发行版）：

   bash

   

   ```bash
   bash <(curl -fsSL https://get.hy2.sh/) client
   # 然后输入上面那串 hysteria2:// 开头的链接
   ```

2. 或者用 Clash Meta / v2rayN / Shadowrocket 直接扫二维码或粘贴链接即可

方案二：更通用、支持所有客户端 —— VLESS + Reality（2025 年主流）

bash



```bash
# 同样一键脚本（3x-ui 面板作者维护的最稳版本）
bash <(curl -Ls https://raw.githubusercontent.com/mhsanaei/3x-ui/master/install.sh)

# 安装完后浏览器访问 https://你的IP:2053
# 默认账号密码都是 admin
# 新建入站 → 协议选 VLESS + Reality
# Vision + uTLS + Fingerprint 选 chrome
# ShortIds 和公钥脚本会自动生成
# 域名填一个常见的，比如 www.bing.com 或 www.microsoft.com
```

国内用户使用方法（零客户端）：Windows 一行命令（nekoray / v2rayN 自带核心）：

powershell



```powershell
# 下载最新 nekoray
Start-Process "https://github.com/MatsuriDayo/nekoray/releases/latest"
# 启动后直接粘贴下面这行链接即可：
vless://uuid@你的香港IP:443?security=reality&pbk=公钥&fp=chrome&spx=%2F&sni=www.bing.com&flow=xtls-rprx-vision#HK-GitHub
```

方案三：极简 SOCKS5/HTTP 代理（只为 git 和 docker 用，不想折腾协议）如果你只想让 git clone 和 docker pull 走代理，不在乎其他：

bash



```bash
# 安装 tinyproxy（纯 HTTP 代理）或 dante（SOCKS5）
apt update && apt install -y tinyproxy

# 编辑配置文件
cat > /etc/tinyproxy/tinyproxy.conf <<EOF
Port 8888
Listen 0.0.0.0
Allow 0.0.0.0/0
DisableViaHeader On
EOF

systemctl restart tinyproxy
```

国内用户只需要两行配置：

bash



```bash
# git
git config --global http.proxy http://香港IP:8888
git config --global https.proxy http://香港IP:8888

# docker
mkdir -p /etc/systemd/system/docker.service.d
cat > /etc/systemd/system/docker.service.d/proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://香港IP:8888"
Environment="HTTPS_PROXY=http://香港IP:8888"
Environment="NO_PROXY=localhost,127.0.0.1"
EOF
systemctl daemon-reload && systemctl restart docker
```

总结对比（2025 年推荐顺序）

| 方案          | 搭建难度 | 速度 | 抗封锁   | 国内是否需要客户端 | 推荐指数 |
| ------------- | -------- | ---- | -------- | ------------------ | -------- |
| Hysteria2     | ★☆☆☆☆    | 极快 | 极强     | 基本不需要         | ★★★★★    |
| VLESS+Reality | ★★☆☆☆    | 很快 | 极强     | 基本不需要         | ★★★★★    |
| TinyProxy     | ★☆☆☆☆    | 一般 | 容易被封 | 不需要             | ★★☆☆☆    |

强烈建议直接用 Hysteria2 一键脚本，5 分钟搞定，国内用户连客户端都不用装，一行命令或者扫个码就能满速拉 GitHub 和 Docker Hub。需要我直接给你生成完整的一键脚本 + 客户端配置模板吗？