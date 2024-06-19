# deploy-coder-gitlab

Deploying coder, gitlab, gitlabrunner on an Ubuntu 22.04 LTS environment

## 1、Download

> [Ubuntu 22.04.4 LTS (Jammy Jellyfish)](https://releases.ubuntu.com/jammy/)
>
> https://releases.ubuntu.com/jammy/ubuntu-22.04.4-desktop-amd64.iso

ubuntu-22.04.4-desktop-amd64.iso

## 2、Update

~~~bash
# Upgrade the kernel
sudo apt-get update
sudo apt-get upgrade
# Restart 
shutdown -r now
~~~

## 3、Firewall

~~~bash
# Check status
sudo ufw status
# Disable
sudo ufw disable
~~~

## 4、Xshell 

Convenient for copying commands

> [Xshell 连接 Ubuntu 教程（超详细）,并解决二个常见问题（一直连不上、root用户拒绝密码）_xshell连接ubuntu-CSDN博客](https://blog.csdn.net/qq_44222849/article/details/105043739)

~~~bash
# 1. Download SSH service
sudo apt-get install openssh-server
# 2. Verify if the service has started after downloading
ps -e | grep ssh
# If only ssh-agent is displayed, it means it hasn't started yet.
# 3. Start the service
/etc/init.d/ssh start
# 4. copy the ens:32 ip
ifconfig -a 
ip addr
~~~

## 5、Clash 

> [FBIMAY0/ClashForWindows_Archive: Clash for Windows Repository is deleted, here are its releases. (github.com)](https://github.com/FBIMAY0/ClashForWindows_Archive) 
>
> [Ubuntu 配置clash的四种方式 – 日拱一卒 (joeyne.cool)](https://www.joeyne.cool/http/proxy/ubuntu-安装clash并配置开机启动/#clash-for-windows)

clash can solve the problem of accessing github network in China, only for learning.

### 5.1、Recommendation

~~~bash
# download
https://github.com/FBIMAY0/ClashForWindows_Archive/releases/download/0.20.39/Clash.for.Windows-0.20.39-x64-linux.tar.gz
# Go to the download directory (by default, downloads are made to the ~/Downloads directory, if not, please go to the corresponding download directory)
cd ~/Downloads
tar -zxvf Clash.for.Windows-0.20.39-x64-linux.tar.gz
mv Clash.for.Windows-0.20.39-x64-linux clash
cd clash
# Execute the cfw command to open the clash interface
./cfw
# Purchase a subscription (don't want to be invited, can be removed?code=*) https://lovenao.pro/#/register?code=BP0YO0TZ
# Set the system network proxy mode in the upper right corner
# manual
http  127.0.0.1 7890
https 127.0.0.1 7890
socks 127.0.0.1 7891
# Open google.com to see if it's available

# Desktop
# Go to the user application directory
cd ~/.local/share/applications
touch clash.desktop
# Paste the following code into the clash.desktop file (Icon is the application icon, you can download it from the web and bring it in, for example, I put the downloaded icon clash.png under ~/Documents directory)
[Desktop Entry]
Name=clash for windows
Icon=~/Documents/clash.png
Exec=~/Downloads/clash/cfw
Type=Application

# Adding executable permissions
chmod +x clash.desktop
# If you can't see the application icon, try logging out or searching for clash.
~~~

### **5.2、The other way(it might not work.)**

> [Elegycloud/clash-for-linux-backup: 基于Clash Core 制作的Clash For Linux备份仓库 A Clash For Linux Backup Warehouse Based on Clash Core (github.com)](https://github.com/Elegycloud/clash-for-linux-backup/tree/main)

~~~bash
# download
git clone https://github.com/Elegycloud/clash-for-linux-backup.git
# Purchase a subscription (don't want to be invited, can be removed?code=*) https://lovenao.pro/#/register?code=BP0YO0TZ
# Note: The variable CLASH_SECRET in the .env file is a custom Clash Secret. When the value is null, the script will automatically generate a random string.
cd clash-for-linux
vim .env

# start
cd clash-for-linux
sudo bash start.sh

source /etc/profile.d/clash.sh
proxy_on
# check
netstat -tln | grep -E '9090|789.'
# output
#tcp6       0      0 :::9090                 :::*                    LISTEN     
#tcp6       0      0 :::7890                 :::*                    LISTEN     
#tcp6       0      0 :::7891                 :::*                    LISTEN     
#tcp6       0      0 :::7892                 :::*                    LISTEN 

env | grep -E 'http_proxy|https_proxy'
# output
#https_proxy=http://127.0.0.1:7890
#http_proxy=http://127.0.0.1:7890
~~~

## **6、Docker**

> [Install Docker Engine on Ubuntu | Docker Docs](https://docs.docker.com/engine/install/ubuntu/)
> [Linux post-installation steps for Docker Engine | Docker Docs](https://docs.docker.com/engine/install/linux-postinstall/)

~~~bash
# Remove all the original docker
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

# install
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

# Download the latest version
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# check
sudo docker run hello-world

# Add sudo management
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world
# Modify original file permissions (optional)
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
# Add boot-up
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
~~~

## 7、Sysbox

> [sysbox/docs/user-guide/install-package.md at master · nestybox/sysbox (github.com)](https://github.com/nestybox/sysbox/blob/master/docs/user-guide/install-package.md#sysbox-user-guide-installation-with-the-sysbox-package)

~~~bash
wget https://downloads.nestybox.com/sysbox/releases/v0.6.4/sysbox-ce_0.6.4-0.linux_amd64.deb
sudo wget https://downloads.nestybox.com/sysbox/releases/v0.6.4/sysbox-ce_0.6.4-0.linux_amd64.deb
# validate 
sha256sum sysbox-ce_0.6.4-0.linux_amd64.deb
# d034ddd364ee1f226b8b1ce7456ea8a12abc2eb661bdf42d3e603ed2dc741827  sysbox-ce_0.6.4-0.linux_amd64.deb
# Delete all docker containers
docker rm $(docker ps -a -q) -f
sudo apt-get install jq -y
sudo apt-get install ./sysbox-ce_0.6.4-0.linux_amd64.deb
sudo systemctl status sysbox -n20

# Default use --runtime=sysbox-runc
sudo vim /etc/docker/daemon.json
# add
{
#...
  "default-runtime": "sysbox-runc",
  "runtimes": {
     "sysbox-runc": {
        "path": "/usr/bin/sysbox-runc"
     }
  }
#...
}

sudo systemctl restart docker
~~~

## 8、Coder

> [Installation - Coder v2 Docs](https://coder.com/docs/v2/latest/install)

### 8.1、Build

database

~~~bash
# preliminary step：Update + Firewall + Clash + Xshell + Docker + Sysbox
docker pull postgres:14.2
sudo mkdir /data
docker run -id --name=postgresql --restart=always -v /data/postgresql/data:/var/lib/postgresql/data -p 5432:5432 -e POSTGRES_PASSWORD=123456 -e LANG=C.UTF-8 postgres:14.2
# Use navicat to create a database(= coder_db, user = postgres)
~~~

Coder

~~~bash
# one-click installation
curl -L https://coder.com/install.sh | sh
# start and add boot-up
sudo systemctl enable --now coder
systemctl status coder
# Access 0.0.0.0:3000

# Modify configuration (see 8.2Configuration)
sudo vim /etc/coder.d/coder.env

systemctl restart coder
systemctl status coder
~~~

### 8.2、Configuration

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

2 ways to create docker containers to use docker inside 8.3.1, 8.3.2

#### 8.3.1、Sysbox（Recommendation）

Can be modified based on starter docker template (recommended)

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

concrete operation

~~~
resource "coder_agent" "main" {
  arch           = data.coder_provisioner.me.arch
  os             = "linux"
  # update startup script
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

# remove
resource "docker_image" "main" {
  ...
}

# update
resource "docker_container" "workspace" {
  count = data.coder_workspace.me.start_count
  # update this
  image   = "codercom/enterprise-base:ubuntu"
  # Uses lower() to avoid Docker restriction on container names.
  name = "coder-${data.coder_workspace.me.owner}-${lower(data.coder_workspace.me.name)}"
  # Hostname makes the shell more user friendly: coder@my-workspace:~$
  hostname = data.coder_workspace.me.name
  # Use the docker gateway if the access URL is 127.0.0.1
  entrypoint = ["sh", "-c", replace(coder_agent.main.init_script, "/localhost|127\\.0\\.0\\.1/", "host.docker.internal")]
  env        = ["CODER_AGENT_TOKEN=${coder_agent.main.token}"]
  # add this
  runtime = "sysbox-runc"
  ...
}
~~~

#### 8.3.2、Host docker

starter docker template (insecurity)

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

The persistence file needs to be under /home/<coder-user-name>/, not /home/coder.

There may be permissions issues when using code-server

~~~bash
# Modify current directory permissions
sudo chown -R $USER /home/coder-1/
~~~

#### 8.4.2、Git

Self-managed GitLab needs to complete step 10.

##### 8.4.2.1、http (Recommendation)

need 10.3

~~~bash
# git clone
git clone http://192.168.79.134:30080/group1/project-1.git
#Cloning into 'project-1'...
#Open the following URL to authenticate with Git:
#http://127.0.0.1:7080/external-auth/primary-gitlab (ctrl+left mouse button)

# The browser will open the site to authenticate automatically, and then "You've authenticated with GitLab!" will appear.
#You are now authenticated. Feel free to close this window!
#......
~~~

##### 8.4.2.2、ssh

ssh to gitlab, not coder's official gitlab integration method.

~~~bash
# config git
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
cat ~/.ssh/id_rsa.pub

# Add ssh key to gitlab personal account ssh key
# Avatar -> Edit Profile -> SSH Key

git clone git@192.168.79.134:example-group/example-project.git
# input yes
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

# start
redis-server
# access cli
redis-cli
# 127.0.0.1:6379> ping
# PONG
systemctl status redis-server
~~~

configuration

vim /etc/redis/redis.conf

~~~
#bind 127.0.0.1 -::1 externally accessible
bind * -::*
# requirepass foobared set password
requirepass user0632
~~~

`systemctl restart redis-server` restart redis

## 10、Gitlab

> [Download and install GitLab | GitLab](https://about.gitlab.com/install/#ubuntu)
>
> [Configuring a Linux package installation | GitLab](https://docs.gitlab.com/omnibus/settings/)

### 10.1、Build

~~~bash
# Preparation：Update + redis
# Install and configure the necessary dependencies
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates tzdata perl

sudo apt-get install -y postfix
# Select 'Internet Site' and press enter

# install ce
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
# Method 1
sudo GITLAB_ROOT_PASSWORD="<strongpassword>" EXTERNAL_URL="http://gitlab.example.com" apt install gitlab-ce
# Method 2
sudo apt install gitlab-ce
sudo vim /etc/gitlab/gitlab.rb
# external_url 'http://192.168.79.134:30080'
sudo gitlab-ctl reconfigure
sudo cat /etc/gitlab/initial_root_password
# Password: C+FMW80tbIjNUFseZPh6f3yIyTDRiJSpQswUvpGv/YE=

# access http://192.168.79.134:30080
~~~

### 10.2、Configuration

~~~bash
sudo vim /etc/gitlab/gitlab.rb

# update
external_url 'http://192.168.79.134:30080'
registry_external_url 'http://192.168.79.134:30080'
gitlab_rails['time_zone'] = 'Asia/Shanghai'
~~~

### 10.3、integrated

**gitlab:**

Application

name: any-you-like
Redirect URI: <CODER_ACCESS_URL>/external-auth/<CODER_EXTERNAL_AUTH_0_ID>/callback

name: coder2
Redirect URI: http://127.0.0.1:7080/external-auth/primary-gitlab/callback

**coder:**

~~~bash
sudo vim /etc/coder.d/coder.env
# add
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

Then in UI -> Deployment -> External Authentication, you can see an additional line

**runner**

Administration -> CI/CD -> Runner -> New instance runner (tag is the flag of the task to be run later, descript ≈ name, it's ok if you don't fill it in)

## 11、Gitlab-runner

> [Install GitLab Runner manually on GNU/Linux | GitLab](https://docs.gitlab.com/runner/install/linux-manually.html)
>
> [Registering runners | GitLab](https://docs.gitlab.com/runner/register/index.html)

2 ways 11.1, 11.2

### 11.1、 Ubuntu deb

Deployed on the same server as gitlab, it worked once, but after rebooting the machine, sysbox and everything else went down, I suggest looking at the Docker way!

**build**

~~~bash
# Replace ${arch} with any of the supported architectures, e.g. amd64, arm, arm64
# A full list of architectures can be found here https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/index.html
#curl -LJO "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/deb/gitlab-runner_${arch}.deb"
sudo curl -LJO "https://s3.dualstack.us-east-1.amazonaws.com/gitlab-runner-downloads/latest/deb/gitlab-runner_amd64.deb"

# install
#dpkg -i gitlab-runner_<arch>.deb
sudo dpkg -i gitlab-runner_amd64.deb
~~~

**Register to gitlab**

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

# gitlab-runner run will give you an error
# ERROR: Failed to load config stat /home/user0632/.gitlab-runner/config.toml: no such file or directory  builds=0 max_builds=1

gitlab-runner list
#Runtime platform                                    arch=amd64 os=linux pid=11678 revision=91a27b2a version=16.11.0
#Listing configured runners                          ConfigFile=/home/user0632/.gitlab-runner/config.toml
~~~

Configuration

`sudo vim /etc/gitlab-runner/config.toml`

### 11.2、Docker（Recommendation）

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

## issues

