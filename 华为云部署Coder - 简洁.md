# Deploy Coder

与测试的本地环境不同：无需代理可访问外网，https

## 1、开放端口

~~~bash
# 关闭防火墙
sudo ufw disable
# 开放端口
sudo ufw allow 22/tcp comment 'SSH'
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'
sudo ufw allow from 127.0.0.1 to any port 5432 proto tcp comment 'Local PostgreSQL'
sudo ufw allow from 127.0.0.1 to any port 7080 proto tcp comment 'Local Coder port'
# 配置 UFW 允许所有来自 Docker 子网的流量(见2.1)
sudo ufw allow from 172.20.0.0/16 comment 'Docker'
# 开启防火墙
sudo ufw enable

# 验证
sudo ufw status
# 结果
sudo ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere                   # SSH
[ 2] 80/tcp                     ALLOW IN    Anywhere                   # HTTP
[ 3] 443/tcp                    ALLOW IN    Anywhere                   # HTTPS
[ 4] 5432/tcp                   ALLOW IN    127.0.0.1                  # Local PostgreSQL
[ 5] 7080/tcp                   ALLOW IN    127.0.0.1                  # Local Coder port
[ 6] Anywhere                   ALLOW IN    172.20.0.0/16              # Docker
[ 7] 22/tcp (v6)                ALLOW IN    Anywhere (v6)              # SSH
[ 8] 80/tcp (v6)                ALLOW IN    Anywhere (v6)              # HTTP
[ 9] 443/tcp (v6)               ALLOW IN    Anywhere (v6)              # HTTPS
~~~

## **2、Docker**

> [Install Docker Engine on Ubuntu | Docker Docs](https://docs.docker.com/engine/install/ubuntu/)
> [Linux post-installation steps for Docker Engine | Docker Docs](https://docs.docker.com/engine/install/linux-postinstall/)

~~~bash
# 删除所有原先 docker
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

# 安装
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# 下载最新版本
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# 验证
sudo docker run hello-world

# 添加 sudo 管理
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
# 修改原文件权限（optional）
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
# 添加开机自启
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
~~~

### 2.1、Docker ufw

配置 UFW 允许所有来自 Docker 子网的流量

~~~bash
# 查看docker网络
docker network inspect bridge
# 允许来着这个子网的网络
sudo ufw allow from 172.20.0.0/16 comment 'Docker'
# 重载
sudo ufw reload

# （后面操作未经验证）
# 修改 Docker Daemon 配置
echo '{"iptables": false}' | sudo tee /etc/docker/daemon.json
# 重启 Docker 服务
sudo systemctl restart docker
# 允许所有内部流量
sudo ufw default allow routed
# 允许所有流量通过 docker0 网络接口（可选）
sudo ufw allow in on docker0
# 检查 UFW 状态
sudo ufw status verbose
~~~

## 3、Sysbox

> [sysbox/docs/user-guide/install-package.md at master · nestybox/sysbox (github.com)](https://github.com/nestybox/sysbox/blob/master/docs/user-guide/install-package.md#sysbox-user-guide-installation-with-the-sysbox-package)

~~~bash
wget https://downloads.nestybox.com/sysbox/releases/v0.6.4/sysbox-ce_0.6.4-0.linux_amd64.deb
# 验证包
sha256sum sysbox-ce_0.6.4-0.linux_amd64.deb
# d034ddd364ee1f226b8b1ce7456ea8a12abc2eb661bdf42d3e603ed2dc741827  sysbox-ce_0.6.4-0.linux_amd64.deb
# 删除所有 docker 容器
docker rm $(docker ps -a -q) -f
sudo apt-get install jq -y
sudo apt-get install ./sysbox-ce_0.6.4-0.linux_amd64.deb
sudo systemctl status sysbox -n20

# 默认使用 --runtime=sysbox-runc
sudo vim /etc/docker/daemon.json
# 添加下面
{
  "default-runtime": "sysbox-runc",
  "runtimes": {
     "sysbox-runc": {
        "path": "/usr/bin/sysbox-runc"
     }
  }
}
sudo systemctl restart docker
~~~

## 4、Coder

> [Installation - Coder v2 Docs](https://coder.com/docs/v2/latest/install)

### 4.1、搭建

数据库（使用docker的postgreSQL）

~~~bash
# 准备：升级 + 防火墙 + Clash + Xshell + Docker + Sysbox
docker pull postgres:14.2
sudo mkdir /data
docker run -id --name=postgresql --restart=always -v /data/postgresql/data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD='CHANGEME' -e LANG=C.UTF-8 postgres:14.2
# 使用 psql 创建
# 进入容器
docker exec -it postgresql bash

# 使用 psql 创建数据库
psql -U postgres
CREATE DATABASE coder_db;
\l
\q
exit

# 从主机检查连接到数据库
sudo apt install postgresql-client-common postgresql-client
psql -h localhost -p 5432 -U postgres -d coder_db
Password for user postgres: CHANGEME
~~~

Coder

~~~bash
# 一键安装
curl -L https://coder.com/install.sh | sh
# 启动并开机自启
sudo systemctl enable --now coder
systemctl status coder
# 访问 0.0.0.0:3000

# 修改配置(见4.2)
sudo vim /etc/coder.d/coder.env

systemctl restart coder
systemctl status coder
~~~

### 4.2、配置

2种方式，推荐4.2.1 （可能问题Coder restart fail）

**https(TLS)证书见5**

sudo vim /etc/coder.d/coder.env

#### 4.2.1 https (ok)

关闭nginx

`sudo systemctl stop nginx`

~~~bash
# Coder must be reachable from an external URL for users and workspaces to connect.
# e.g. https://coder.example.com
CODER_ACCESS_URL="https://your.domain.name"

CODER_HTTP_ADDRESS=
CODER_PG_CONNECTION_URL="postgresql://postgres:CHANGEME@127.0.0.1:5432/coder_db?sslmode=disable"
CODER_TLS_CERT_FILE="/etc/nginx/certs/your.domain.name/certificate.crt"
CODER_TLS_ENABLE=true
CODER_TLS_KEY_FILE="/etc/nginx/certs/your.domain.name/private.key"
CODER_TLS_ADDRESS="0.0.0.0:443"
# Run "coder server --help" for flag information.
~~~

### 4.3、Template

2种创建docker容器内使用docker的方式 8.3.1、8.3.2

#### 4.3.1、Sysbox (ok)

基于 starter docker template 修改（推荐）

具体：

~~~
resource "coder_agent" "main" {
  arch           = data.coder_provisioner.me.arch
  os             = "linux"
  # 修改启动脚本
  startup_script = <<-EOT
    set -e

    # Start Docker
    sudo dockerd &
    # install and start code-server
    curl -fsSL https://code-server.dev/install.sh | sh -s -- --method=standalone --prefix=/tmp/code-server --version 4.19.1
    nohup /tmp/code-server/bin/code-server --auth none --port 13337 >/tmp/code-server.log 2>&1 &

    # Close output pipes
    wait # Wait for all background processes to finish
    exec >/dev/null 2>&1 # Close stdout and stderr
  EOT
  ...
}

# 删掉
resource "docker_image" "main" {
  ...
}

# 修改
resource "docker_container" "workspace" {
  count = data.coder_workspace.me.start_count
  # 修改这个
  image   = "codercom/enterprise-base:ubuntu"
  # Uses lower() to avoid Docker restriction on container names.
  name = "coder-${data.coder_workspace.me.owner}-${lower(data.coder_workspace.me.name)}"
  # Hostname makes the shell more user friendly: coder@my-workspace:~$
  hostname = data.coder_workspace.me.name
  # Use the docker gateway if the access URL is 127.0.0.1
  entrypoint = ["sh", "-c", replace(coder_agent.main.init_script, "/localhost|127\\.0\\.0\\.1/", "host.docker.internal")]
  env        = ["CODER_AGENT_TOKEN=${coder_agent.main.token}"]
  # 添加这个
  runtime = "sysbox-runc"
  ...
}
~~~

### 4.4、Workspace

#### 4.4.1、code-server

需要持久化文件放在 /home/<coder-user-name>/ 下，而不是 /home/coder 下

使用 code-server 时可能会有权限问题

~~~bash
# 修改当前目录权限
sudo chown -R $USER /home/coder-1/
~~~

## 5、使用Https服务

### 5.1、Nginx

**下载**

~~~bash
# 下载nginx
sudo apt install nginx
# 启动并开机自启
sudo systemctl start nginx
sudo systemctl enable nginx
~~~



### 5.2、TLS证书

3种方式推荐5.2.1

#### 5.2.1、ZeroSSL (ok)

> [Free SSL Certificates and SSL Tools - ZeroSSL](https://zerossl.com/)

**New Certifaicate**

~~~
Domains: your.domain.name
Validity: 90-Day Certificate
Add-Ons: none
Finalize Your Order: none
~~~

**Verify Domain**

~~~bash
# HTTP File Upload
# Download Auth File
# 将验证文件上传到服务器任意目录
sudo mkdir /etc/nginx/well-known
sudo mv ~/YOUR_TXT.txt /etc/nginx/well-known/
# 配置nginx服务器验证
cd /etc/nginx/
sudo vim sites-available/coder-wellknown
sudo ln /etc/nginx/sites-available/coder-wellknown /etc/nginx/sites-enable/
# Verify Domain
~~~

nginx 配置 coder-wellknown

~~~
server {
    listen 80;
    server_name your.domain.name;

    location /.well-known/pki-validation/ {
        alias /etc/nginx/well-known/;
    }
}
~~~

**Install Certificate**

> [Installing SSL Certificate on NGINX – ZeroSSL](https://help.zerossl.com/hc/en-us/articles/360058295894-Installing-SSL-Certificate-on-NGINX)

~~~bash
# Download Certificate
# 将证书解压后的目录文件上传到服务器任意目录
sudo mkdir /etc/nginx/certs
sudo mv ~/your.domain.name /etc/nginx/certs/your.domain.name
ls
# ca_bundle.crt  certificate.crt  private.key
# 合并.cet文件
cat certificate.crt ca_bundle.crt >> certificate.crt
# 配置nginx服务器证书
sudo vim sites-available/coder-wellknown
# 重启nginx服务器
sudo systemctl restart nginx
# Installation Complete
~~~

nginx配置https证书

~~~
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name your.domain.name;

    ssl_certificate /etc/nginx/certs/your.domain.name/certificate.crt;
    ssl_certificate_key /etc/nginx/certs/your.domain.name/private.key;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    location = /404.html {
        internal;
    }
}
~~~



## 问题

### Coder restart fail

~~~bash
# 4.2 配置出错，可能为 CODER_PG_CONNECTION_URL 的用户密码存在特殊符号(#)，无法直接输入，使用(%23)替代
~~~



### **JetBrains Gateway connect fail**

> [JetBrains Gateway - Coder Docs](https://coder.com/docs/ides/gateway#configuring-the-gateway-plugin-to-use-internal-certificates)

~~~
2024-07-03 16:01:14,555 [3082876]   WARN - #c.j.g.s.RemoteCredentialsEx - Timeout of PT10M expired while waiting for '/bin/bash -lc echo\ REMOTE_EXEC_OUTPUT_MARKER_\ \&\&\ curl\ -fSL\ --output\ /home/jzj-admin/.cache/JetBrains/RemoteDev/dist/ideaIU-2024.1.4.tar.gz\ https://download.jetbrains.com/idea/ideaIU-2024.1.4.tar.gz' to terminate, forcefully terminating...
2024-07-03 16:01:14,773 [3083094]   WARN - #c.j.g.s.GoHighLevelHostAccessor - An error occurred while execution command: '/bin/bash -lc echo\ REMOTE_EXEC_OUTPUT_MARKER_\ \&\&\ curl\ -fSL\ --output\ /home/jzj-admin/.cache/JetBrains/RemoteDev/dist/ideaIU-2024.1.4.tar.gz\ https://download.jetbrains.com/idea/ideaIU-2024.1.4.tar.gz'
Exit code: 255
Stderr: ''
Stdout: '  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   138  100   138    0     0    444      0 --:--:-- --:--:-- --:--:--   443
  0 1266M    0 6077k    0     0  3573k      0  0:06:02  0:00:01  0:06:01 3573k
  0 1266M    0 6108k    0     0  2719k      0  0:07:57  0:00:02  0:07:55 59839
......
 81 1266M   81 1037M    0     0  1779k      0  0:12:09  0:09:57  0:02:12 1659k
 82 1266M   82 1039M    0     0  1780k      0  0:12:08  0:09:58  0:02:10 1750k
 82 1266M   82 1041M    0     0  1780k      0  0:12:08  0:09:59  0:02:09 1868k'
2024-07-03 16:01:14,780 [3083101] SEVERE - CoderRemoteConnectionHandle - Failed to connect (will not retry)
com.jetbrains.gateway.ssh.RemoteCommandException: Command "/bin/bash -lc echo\ REMOTE_EXEC_OUTPUT_MARKER_\ \&\&\ curl\ -fSL\ --output\ /home/jzj-admin/.cache/JetBrains/RemoteDev/dist/ideaIU-2024.1.4.tar.gz\ https://download.jetbrains.com/idea/ideaIU-2024.1.4.tar.gz" failed with exit code 255}
~~~

服务器10分钟未能下载完IDE应用，程序直接退出了

解决方案：手动下载ide后端，启动

> [下载 IntelliJ IDEA – 领先的 Java 和 Kotlin IDE (jetbrains.com)](https://www.jetbrains.com/zh-cn/idea/download/?section=linux)

~~~bash
mkdir ~/idea
cd ~/idea
wget -O ideaIU-2024.1.4.tar.gz https://download.jetbrains.com/idea/ideaIU-2024.1.4.tar.gz?_gl=1*wl8x82*_gcl_au*MTI4OTY3Mzc5Mi4xNzE1ODYzMjIw*_ga*MTMyMTMyNTcyLjE3MTU2MTQ4NTQ.*_ga_9J976DJZ68*MTcyMDc2NTE0OC4yNy4xLjE3MjA3NjUxODQuMjQuMC4w
tar -zxvf ideaIU-2024.1.4.tar.gz
cd bin/
ideaIU-2024.1.4.tar.gz
./remote-dev-server.sh run /path/to/project
~~~

Jetbrains Gateway / Idea 使用 coder 插件添加到 Coder 后选好workspace连接

