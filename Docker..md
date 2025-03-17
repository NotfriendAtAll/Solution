# 如何在wsl上安装Docker
- ：直接把复制到wsl的终端**bash**就ok了(使用前开梯子),同时确保wsl版本是2.0，不是的话管理员打开powershell
```cpp
wsl --update
```
---
### 卸载 Docker CE、CLI 和依赖
```cpp
sudo apt-get purge docker-ce docker-ce-cli containerd.io runc
```
***
### 删除 Docker 配置文件和数据目录
- sudo rm -rf /etc/docker
---
- sudo rm -rf /var/lib/docker
---
- sudo rm -rf /var/run/docker
---
### 重置 Systemd 服务缓存
```cpp
sudo systemctl daemon-reload
```
***
## 安装官方依赖
- sudo apt-get update
- sudo apt-get install -y \ apt-transport-https \
    ca-certificates \ curl \ gnupg-agent \
    software-properties-common
---
## 添加 Docker GPG 密钥
```cpp
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
---
## 安装 Docker CE
```cpp
sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
```
---
## 启动 Docker 服务并验证
-  重载 Systemd 配置

      sudo systemctl daemon-reload
***
- 启动 Docker 服务

      sudo systemctl start docker
***
- 设置开机自启

      sudo systemctl enable docker
***
- 验证服务状态

      sudo systemctl status docker
***
## 运行测试容器
```cpp
      sudo docker run hello-world
```
---
# 如果是下面输出，安装成功
      Hello from Docker!
***
希望对你有帮助!