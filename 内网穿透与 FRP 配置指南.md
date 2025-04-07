

## 1. 内网穿透方案

### **1.1 使用 FRP 进行内网穿透**
FRP（Fast Reverse Proxy）是一款用于内网穿透的工具，支持 TCP、UDP、HTTP、HTTPS 等协议。

#### **FRP 的两部分**
- **FRPS（服务端）**：部署在公网服务器上，负责接收流量并转发。
- **FRPC（客户端）**：部署在内网机器上，将本地服务映射到公网。

### **1.2 FRP 的基本安装和启动**
#### **FRPS（云服务器）**
```bash
# 进入 FRP 目录
cd /usr/frp/frp_0.61.1_linux_amd64/

# 启动 FRP 服务端
./frps -c frps.toml
```

#### **FRPC（内网机器）**
```bash
# 启动 FRP 客户端
./frpc -c frpc.toml
```

### **1.3 FRP 配置文件（TOML 格式）**

#### **FRPS 配置（frps.toml）**
```toml
bindPort = 7000  # FRP 监听端口

vhostHttpPort = 8080  # FRP HTTP 端口
vhostHttpsPort = 8443 # FRP HTTPS 端口
```

#### **FRPC 配置（frpc.toml）**
```toml
serverAddr = "你的云服务器公网 IP"
serverPort = 7000

[[proxies]]
name = "web"
type = "http"
localPort = 80
customDomains = ["yourdomain.com"]
```

### **1.4 多端口映射**
要映射多个服务，可以在 `frpc.toml` 里添加多个 `[[proxies]]` 配置，例如：
```toml
[[proxies]]
name = "ssh"
type = "tcp"
localPort = 22
remotePort = 2222

[[proxies]]
name = "web"
type = "http"
localPort = 80
customDomains = ["yourdomain.com"]
```

这样，你可以通过 `ssh -p 2222 user@云服务器IP` 远程连接内网机器。

---
## 2. 结合 Nginx 进行反向代理

### **2.1 Nginx 配置（云服务器）**
在 `/etc/nginx/sites-available/default` 或 `/etc/nginx/nginx.conf` 添加：
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

如果使用 HTTPS，添加 SSL 证书：
```nginx
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### **2.2 重启 Nginx**
```bash
sudo systemctl restart nginx
```

### **2.3 访问方式**
如果你正确配置了 DNS 解析（A 记录指向你的云服务器公网 IP），你可以直接在浏览器访问：
```
http://yourdomain.com
https://yourdomain.com
```

---


## 后台启动

如果你的服务器长期运行，可以创建 systemd 服务。

  

**1. 创建服务文件**

```bash
sudo nano /etc/systemd/system/frps.service
```

填入以下内容：

```ini
[Unit]
Description=FRP Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/frp/frp_0.61.1_linux_amd64/frps -c /usr/frp/frp_0.61.1_linux_amd64/frps.toml
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

**2. 让 frps 随开机启动**

```bash
sudo systemctl daemon-reload
sudo systemctl enable frps
sudo systemctl start frps

sudo systemctl status frps

sudo systemctl stop frps
```