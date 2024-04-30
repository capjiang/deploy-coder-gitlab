# deploy-coder-gitlab

在 Ubuntu 22.04 LTS 环境下，部署 coder，gitlab，gitlabrunner

## 1、下载

> [Ubuntu 22.04.4 LTS (Jammy Jellyfish)](https://releases.ubuntu.com/jammy/)
>
> https://releases.ubuntu.com/jammy/ubuntu-22.04.4-desktop-amd64.iso
>
> [VMware虚拟机安装Ubuntu 22.04：从入门到精通 (baidu.com)](https://cloud.baidu.com/article/3280919)

桌面版：ubuntu-22.04.4-desktop-amd64.iso

## 2、升级

~~~bash
# 升级内核
sudo apt-get update
sudo apt-get upgrade
# 重启适配
shutdown -r now
~~~

## 3、防火墙

~~~bash
# 查看状态
sudo ufw status
# 关闭
sudo ufw disable
~~~

## 4、Xshell 

方便 copy 命令

> [Xshell 连接 Ubuntu 教程（超详细）,并解决二个常见问题（一直连不上、root用户拒绝密码）_xshell连接ubuntu-CSDN博客](https://blog.csdn.net/qq_44222849/article/details/105043739)

~~~bash
# 1、下载SSH服务
sudo apt-get install openssh-server
# 2、验证下载后是否已经开启了服务：
ps -e | grep ssh
# 如果只有 ssh-agent 表示还没启动。
# 执行下句，开启服务
/etc/init.d/ssh start
# 如果显示 sshd 则说明已启动成功。
ipconfig -a 
ip addr
~~~

## 5、Clash

> [FBIMAY0/ClashForWindows_Archive: Clash for Windows Repository is deleted, here are its releases. (github.com)](https://github.com/FBIMAY0/ClashForWindows_Archive) 
>
> [Ubuntu 配置clash的四种方式 – 日拱一卒 (joeyne.cool)](https://www.joeyne.cool/http/proxy/ubuntu-安装clash并配置开机启动/#clash-for-windows)

clash 可解决国内访问 github 网络问题，仅学习使用

### 5.1、推荐方式

~~~bash
# download
https://github.com/FBIMAY0/ClashForWindows_Archive/releases/download/0.20.39/Clash.for.Windows-0.20.39-x64-linux.tar.gz
# 进入下载目录（默认情况是下载到 ~/Downloads 目录，如果不是请进入到对应的下载目录）
cd ~/Downloads
# 解压
tar -zxvf Clash.for.Windows-0.20.39-x64-linux.tar.gz
# 重命名
mv Clash.for.Windows-0.20.39-x64-linux clash
# 进入clash目录
cd clash
# 执行cfw命令，即可打开clash界面
./cfw
# 购买订阅(不想被邀请，可删除?code=*) https://lovenao.pro/#/register?code=BP0YO0TZ
# 右上角设置系统网络代理模式
# 手动
http  127.0.0.1 7890
https 127.0.0.1 7890
socks 127.0.0.1 7891
# 打开 google.com 查看是否可用

# 桌面图标
# 进入用户应用程序目录
cd ~/.local/share/applications
# 创建clash应用程序
touch clash.desktop
# 将以下代码粘贴到 clash.desktop 文件(Icon是应用程序图标，可以自行在网络下载，然后引入即可，比如我将下载的图标 clash.png 放到 ~/Documents 目录下面)
[Desktop Entry]
Name=clash for windows
Icon=~/Documents/clash.png
Exec=~/Downloads/clash/cfw
Type=Application
# 添加可执行权限
chmod +x clash.desktop
# 上面步骤操作完成，如果看不到应用程序图标，可以尝试注销用户或者直接搜索 clash
~~~

### **5.2、另外的方法(可能没用)**

> [Elegycloud/clash-for-linux-backup: 基于Clash Core 制作的Clash For Linux备份仓库 A Clash For Linux Backup Warehouse Based on Clash Core (github.com)](https://github.com/Elegycloud/clash-for-linux-backup/tree/main)

~~~bash
# 下载
git clone https://github.com/Elegycloud/clash-for-linux-backup.git
# 购买订阅地址(不想被邀请，可删除?code=*) https://lovenao.pro/#/register?code=BP0YO0TZ
# 注意： .env 文件中的变量 CLASH_SECRET 为自定义 Clash Secret，值为空时，脚本将自动生成随机字符串。
cd clash-for-linux
vim .env

# 启动
cd clash-for-linux
sudo bash start.sh

source /etc/profile.d/clash.sh
proxy_on
# 检查
netstat -tln | grep -E '9090|789.'
# 输出
tcp6       0      0 :::9090                 :::*                    LISTEN     
tcp6       0      0 :::7890                 :::*                    LISTEN     
tcp6       0      0 :::7891                 :::*                    LISTEN     
tcp6       0      0 :::7892                 :::*                    LISTEN 

env | grep -E 'http_proxy|https_proxy'
# 输出
https_proxy=http://127.0.0.1:7890
http_proxy=http://127.0.0.1:7890
~~~

## **6、Docker**

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
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

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

## 7、Sysbox

> [sysbox/docs/user-guide/install-package.md at master · nestybox/sysbox (github.com)](https://github.com/nestybox/sysbox/blob/master/docs/user-guide/install-package.md#sysbox-user-guide-installation-with-the-sysbox-package)

~~~bash
wget https://downloads.nestybox.com/sysbox/releases/v0.6.4/sysbox-ce_0.6.4-0.linux_amd64.deb
sudo wget https://downloads.nestybox.com/sysbox/releases/v0.6.4/sysbox-ce_0.6.4-0.linux_amd64.deb
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

## 8、Coder

> [Installation - Coder v2 Docs](https://coder.com/docs/v2/latest/install)

### 8.1、搭建

数据库

~~~bash
# 准备：升级 + 防火墙 + Clash + Xshell + Docker + Sysbox
docker pull postgres:14.2
sudo mkdir /data
docker run -id --name=postgresql --restart=always -v /data/postgresql/data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD=123456 -e LANG=C.UTF-8 postgres:14.2
# 使用 navicat 创建一个 database(= coder_db, user = postgres)
~~~

Coder

~~~bash
# 一键安装
curl -L https://coder.com/install.sh | sh
# 启动并开机自启
sudo systemctl enable --now coder
systemctl status coder
# 访问 0.0.0.0:3000

# 修改配置(见配置)
sudo vim /etc/coder.d/coder.env

systemctl restart coder
systemctl status coder
~~~

### 8.2、配置

sudo vim /etc/coder.d/coder.env

~~~
# Coder must be reachable from an external URL for users and workspaces to connect.
# e.g. https://coder.example.com
CODER_ACCESS_URL="http://192.168.79.132:7080"

CODER_HTTP_ADDRESS="0.0.0.0:7080"
CODER_PG_CONNECTION_URL="postgresql://postgres:123456@127.0.0.1:5432/coder_db?sslmode=disable"
CODER_TLS_CERT_FILE=
CODER_TLS_ENABLE=
CODER_TLS_KEY_FILE=

# Run "coder server --help" for flag information.
~~~

### 8.3、Template

2种创建docker容器内使用docker的方式 8.3.1、8.3.2

#### 8.3.1、Sysbox（推荐）

可基于 starter docker template 修改（推荐）

~~~
resource "docker_container" "workspace" {
  # ...
  name    = "coder-${data.coder_workspace.me.owner}-${lower(data.coder_workspace.me.name)}"
  image   = "codercom/enterprise-base:ubuntu"
  env     = ["CODER_AGENT_TOKEN=${coder_agent.main.token}"]
  command = ["sh", "-c", coder_agent.main.init_script]
  # Use the Sysbox container runtime (required)
  runtime = "sysbox-runc"
}

resource "coder_agent" "main" {
  arch           = data.coder_provisioner.me.arch
  os             = "linux"
  startup_script = <<EOF
    #!/bin/sh

    # Start Docker
    sudo dockerd &

    # ...
    EOF
}
~~~

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

#### 8.3.2、宿主机 docker

starter docker template（不安全）

Dockerfile

~~~dockerfile
FROM ubuntu

RUN apt-get update \
	&& apt-get install -y \
	curl \
	git \
	golang \
	sudo \
	vim \
	wget \
	# add docker cli
	docker.io \
	&& rm -rf /var/lib/apt/lists/*
...
~~~

main.tf

~~~
resource "docker_container" "workspace" {
  ...
  # add
  command = ["/bin/bash", "-c", "tail -f /dev/null"]
  ...
  # add
  volumes {
    host_path = "/var/run/docker.sock"
    container_path = "/var/run/docker.sock"
  }
  ...
}
~~~

### 8.4、Workspace

#### 8.4.1、code-server

需要持久化文件放在 /home/<coder-user-name>/ 下，而不是 /home/coder 下

使用 code-server 时可能会有权限问题

~~~bash
# 修改当前目录权限
sudo chown -R $USER /home/coder-1/
~~~

#### 8.4.2、Git

self-managed GitLab 需完成步骤10

##### 8.4.2.1、http（推荐）

需完成到10.3

~~~bash
# git clone
git clone http://192.168.79.134:30080/group1/project-1.git
#Cloning into 'project-1'...
#Open the following URL to authenticate with Git:
#http://127.0.0.1:7080/external-auth/primary-gitlab (ctrl+鼠标左键)

# 浏览器会打开网站自动进行认证，然后出现 “You've authenticated with GitLab!”
#You are now authenticated. Feel free to close this window!
#......
~~~

##### 8.4.2.2、ssh

ssh 连接 gitlab，不是 coder 官方集成 gitlab 方法

~~~bash
# 登录 git
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
cat ~/.ssh/id_rsa.pub

# 将 ssh 密钥添加到 gitlab个人账号的 ssh 密钥
# 头像 -> 编辑个人资料 -> SSH密钥

git clone git@192.168.79.134:example-group/example-project.git
# 输入 yes
~~~

## 9、Redis

> [Install Redis on Linux | Docs](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-linux/)

~~~bash
# If you're running a very minimal distribution (such as a Docker container) you may need to install lsb-release, curl and gpg first
sudo apt install lsb-release curl gpg

# install
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis

# 启动
redis-server
# 进入客户端
redis-cli
# 127.0.0.1:6379> ping
# PONG
systemctl status redis-server
~~~

配置

vim /etc/redis/redis.conf

~~~
#bind 127.0.0.1 -::1 外部可访问
bind * -::*
# requirepass foobared 设置密码
requirepass user0632
~~~

`systemctl restart redis-server` 重启 redis

## 10、Gitlab

> [Download and install GitLab | GitLab](https://about.gitlab.com/install/#ubuntu)
>
> [Configuring a Linux package installation | GitLab](https://docs.gitlab.com/omnibus/settings/)

### 10.1、搭建

~~~bash
# 准备：升级 + redis
# Install and configure the necessary dependencies
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl

sudo apt-get install -y postfix
# Select 'Internet Site' and press enter

# install ce
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
# 第一种方法
sudo GITLAB_ROOT_PASSWORD="<strongpassword>" EXTERNAL_URL="http://gitlab.example.com" apt install gitlab-ce
# 第二种方法
sudo apt install gitlab-ce
sudo vim /etc/gitlab/gitlab.rb
# external_url 'http://192.168.79.134:30080'
sudo gitlab-ctl reconfigure
sudo cat /etc/gitlab/initial_root_password
# Password: C+FMW80tbIjNUFseZPh6f3yIyTDRiJSpQswUvpGv/YE=

# 访问 http://192.168.79.134:30080
~~~

### 10.2、配置

~~~bash
sudo vim /etc/gitlab/gitlab.rb

# 如下
external_url 'http://192.168.79.134:30080'
registry_external_url 'http://192.168.79.134:30080'
gitlab_rails['time_zone'] = 'Asia/Shanghai'
~~~

### 10.3、集成

**gitlab:**

应用程序

name: any-you-like
Redirect URI: <CODER_ACCESS_URL>/external-auth/<CODER_EXTERNAL_AUTH_0_ID>/callback

name: coder2
Redirect URI: http://127.0.0.1:7080/external-auth/primary-gitlab/callback

**coder:**

~~~bash
sudo vim /etc/coder.d/coder.env
# 添加
CODER_EXTERNAL_AUTH_0_ID="primary-gitlab"
CODER_EXTERNAL_AUTH_0_TYPE=gitlab
# This value is the "Application ID"
CODER_EXTERNAL_AUTH_0_CLIENT_ID=1be4b7b1884ed2557e919ec57fb0ed54fcebc5b16c99ac0fc7fe3734867666a7
CODER_EXTERNAL_AUTH_0_CLIENT_SECRET=gloas-aef10a18df42186844c5307d36eefc2a1e071437995db3a0d22ff187fc7c625c
CODER_EXTERNAL_AUTH_0_VALIDATE_URL="http://192.168.79.134:30080/oauth/token/info"
CODER_EXTERNAL_AUTH_0_AUTH_URL="http://192.168.79.134:30080/oauth/authorize"
CODER_EXTERNAL_AUTH_0_TOKEN_URL="http://192.168.79.134:30080/oauth/token"
CODER_EXTERNAL_AUTH_0_REGEX=192\.168\.79\.134:30080
~~~

然后在 UI -> Deployment -> External Authentication 可以看到多了一行

**runner**

管理中心 -> CI/CD -> Runner -> 新建实例 runner（tag 是以后需要运行任务的标志，descript ≈ 名称，不填也没事）

## 11、Gitlab-runner

> [Install GitLab Runner manually on GNU/Linux | GitLab](https://docs.gitlab.com/runner/install/linux-manually.html)
>
> [Registering runners | GitLab](https://docs.gitlab.com/runner/register/index.html)

2种方式11.1，11.2

### 11.1、官方 Ubuntu

部署在和gitlab同一个服务器，正常运行过一次，但是机器重启后 sysbox 什么的都瘫痪了，建议看 Docker 方式

**搭建**

~~~bash
# Replace ${arch} with any of the supported architectures, e.g. amd64, arm, arm64
# A full list of architectures can be found here https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/index.html
#curl -LJO "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/deb/gitlab-runner_${arch}.deb"
sudo curl -LJO "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/deb/gitlab-runner_amd64.deb"

# install
#dpkg -i gitlab-runner_<arch>.deb
sudo dpkg -i gitlab-runner_amd64.deb
~~~

**注册到 gitlab**

~~~bash
# registry
sudo gitlab-runner register  --url http://192.168.79.134:30080  --token glrt-E_V3DVNu9zyiAh8XojMV
# Runtime platform arch=amd64 os=linux pid=5550 revision=91a27b2a version=16.11.0
# Running in system-mode.
#Enter the GitLab instance URL (for example, https://gitlab.com/):
[http://192.168.79.134:30080]: http://192.168.79.134:30080
#Verifying runner... is valid                        runner=E_V3DVNu9
#Enter a name for the runner. This is stored only in the local config.toml file:
[ubu-0426]: runner-1
#Enter an executor: ssh, docker, docker-autoscaler, instance, custom, shell, parallels, virtualbox, docker-windows, docker+machine, kubernetes:
docker
#Enter the default Docker image (for example, ruby:2.7):
node:alpine
#Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
#Configuration (with the authentication token) was saved in "/etc/gitlab-runner/config.toml" 

# gitlab-runner run 会报错，应该可以不运行这个
# ERROR: Failed to load config stat /home/user0632/.gitlab-runner/config.toml: no such file or directory  builds=0 max_builds=1

# 查看
gitlab-runner list
#Runtime platform                                    arch=amd64 os=linux pid=11678 revision=91a27b2a version=16.11.0
#Listing configured runners                          ConfigFile=/home/user0632/.gitlab-runner/config.toml
~~~

配置

`sudo vim /etc/gitlab-runner/config.toml`

### 11.2、Docker（推荐）

> [Run GitLab Runner in a container | GitLab](https://docs.gitlab.com/runner/install/docker.html)
>
> [Registering runners | GitLab](https://docs.gitlab.com/runner/register/index.html?tab=Docker)

~~~bash
docker run -d --name gitlab-runner --restart always \
  -p 8093:8093 \
  -v /srv/gitlab-runner/config:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner:v16.10.0

# register
docker exec -it gitlab-runner gitlab-runner register
#Runtime platform                                    arch=amd64 os=linux pid=77 revision=81ab07f6 version=16.10.0
#Running in system-mode.                            
                                                   
#Enter the GitLab instance URL (for example, https://gitlab.com/):
http://192.168.79.134:30080
#Enter the registration token:
glrt-KkfgzVsgy4a8e_VN1qCA
#Verifying runner... is valid                        runner=KkfgzVsgy
#Enter a name for the runner. This is stored only in the local config.toml file:
[7fc2cfce1ed2]: runner-1
#Enter an executor: docker-windows, kubernetes, docker-autoscaler, instance, custom, ssh, parallels, docker, shell, virtualbox, docker+machine:
docker
#Enter the default Docker image (for example, ruby:2.7):
docker:stable
#Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
 
#Configuration (with the authentication token) was saved in "/etc/gitlab-runner/config.toml"
~~~

## 问题

