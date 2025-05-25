# 使用cephadm 搭建ceph学习环境
## Cephadm 安装依赖项指南
## 必要组件清单
| 组件             | 必要性       | 说明                                                                 |
|------------------|--------------|----------------------------------------------------------------------|
| Python 3         | 必须         | Cephadm 和核心组件依赖 Python 3.6+                                  |
| Systemd          | 必须         | 用于管理服务（适用于大多数现代 Linux 发行版）                       |
| Podman/Docker    | 必须（容器化部署时） | 容器运行时，用于部署 Ceph 组件（默认容器化部署）                   |
| 时间同步（Chrony/NTPD） | 必须       | 分布式系统严格依赖时间同步                                          |
| LVM2             | 推荐         | 用于存储设备管理（非强制，但建议安装）                              |

---

# 安装步骤 (系统Ubuntu-22.04)
提示：如果遇到权限不够，加上sudo
---
### 1. 安装预先配置
```cpp
1. sudo apt update//更新
2. sudo apt install net-tools -y //用于ifconfig等命令
3. sudo apt install openssh-server -y//安装ssh
```
***
### 允许ssh远程登陆
```cpp
sudo nano /etc/ssh/ssd_config
 PasswordAuthentication yes //把这个改了
 prohibit-password yes
```
```cpp
3.1 sudo systemctl enable --now ssh//开启ssh
3.2 sudo systemctl status ssh//查看status
```
### 用于检查下面安装及配置
```cpp
# 检查 Python 3
python3 --version

# 检查 systemd
systemctl --version

# 检查 Docker
docker info

# 检查 chrony
chronyc tracking

# 检查 lvm2
pvdisplay
```
### 2. 安装 Python 3
#### Ubuntu/Debian
```bash
sudo apt update && sudo apt install python3 -y
```
### 安装容器运行时（Podman/Docker）
- 选项 1：安装 Podman（推荐）
***
​Ubuntu/Debian:
```bash
sudo apt install podman -y
​CentOS/RHEL:
bash
sudo dnf install podman -y
```
- 选项 2：安装 Docker
***
- 推荐这种
Ubuntu/Debian:
```cpp
curl -fsSL https://get.docker.com | sudo sh 
或者
curl -fsSL https://get.docker.com | sh -s -- --mirror Aliyun
```
​Ubuntu/Debian:
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
```
***
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
***
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
***
```bash
sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io -y
```
***
```bash
sudo usermod -aG docker $USER && newgrp docker
```
***
​CentOS/RHEL:
```bash
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
```
```bash
sudo dnf install docker-ce docker-ce-cli containerd.io -y
```
```bash
sudo systemctl enable --now docker
```
```bash
sudo usermod -aG docker $USER && newgrp docker
```
验证：
```bash
podman --version || docker --version  # 输出容器运行时版本
```
***
4. 时间同步（Chrony）
- 安装与配置
​Ubuntu/Debian:
```bash
sudo apt install chrony -y
```
​CentOS/RHEL:
```bash
sudo dnf install chrony -y
```
### 温馨提示
- 如果chronyc tracking
显示时间不是世界时间，就需要加以改造
```cpp
sudo nano /etc/chrony/chrony.conf
把pool全部删除，换成下面这个
pool pool.ntp.org iburst
sudo systemctl restart chronyd
sudo chronyc makestep//就可以了
//必要时开放udp123
sudo ufw allow 123/udp
```
​配置 NTP 服务器​
- （编辑 /etc/chrony.conf）:
```bash
server ntp.aliyun.com iburst
server time1.cloud.tencent.com iburst
```
- 重启服务:
```cpp
sudo systemctl restart chronyd && sudo systemctl enable chronyd
```
验证：
```bash
chronyc tracking  # 检查时间同步状态
```
5. 安装 LVM2
- 安装命令
​Ubuntu/Debian:
```bash
sudo apt install lvm2 -y
```
​CentOS/RHEL:
```bash
sudo dnf install lvm2 -y
```
验证：
```bash
sudo pvdisplay  # 检查物理卷信息（若无设备则为空）
```
***
### 总结表格
组件	Ubuntu/Debian 命令	CentOS/RHEL 命令
​Python3	sudo apt install python3	sudo dnf install python3
​Systemd	通常预装	通常预装
​Podman	sudo apt install podman	sudo dnf install podman
​Docker	按 Docker 官方步骤安装	按 Docker 官方步骤安装
​Chrony	sudo apt install chrony	sudo dnf install chrony
​LVM2	sudo apt install lvm2	sudo dnf install lvm2

#### 注意事项
​权限要求：所有命令需以 root 用户或 sudo 权限执行。
​重启服务：安装完成后建议重启系统。
​防火墙配置：若使用 Docker，需开放容器网络流量（如 docker0 网桥）。
​时间同步：确保所有节点时间偏移 <1 秒（通过 chronyc tracking 验证）。
