# Docekr

## 一、简介

Docker是一个用于**构建**、**运行**、**传送**应用程序的平台。
Docker vs 虚拟机

Docker vs 容器
Docker是容器的子集

## 二、基本概念
- 镜像（image）：
- 容器（container）：
- 仓库（Registry）：

## 三、安装
* Windows
- 下载地址: [https://www.docker.com/](https://www.docker.com/)
- windows系统启用**Hper-V**功能：控制面板 - 搜索"启用或关闭 Windows 功能"

* Linux
```
sudo apt update
sudo apt install docker.io
sudo systemctl start docker
```

## 四、容器化和Dockerfile
1. 创建一个Dockefile（类似requirement.txt）
```
# 在项目根目录下创建一个名为"Dockerfile"无后缀的文件
# 例子
FROM node:14-alpine
COPY index.js /app/index.js
CMD ["node", "/app/index.js"]
```
2. 使用Dockerfile创建镜像
```
docker build -t {docker_name} {Dockerfile文件所在目录}
```
3. 使用镜像创建和运行容器
```
docker run {docker_name} 
```

## 五、其他指令
```
# ====== 镜像 =========
# 搜索镜像
docker search 软件名

# 拉取镜像
docker pull 镜像名

# 查看本地镜像列表
docker images   
docker image ls

# 删除镜像
docker rmi 镜像名/ID

# ====== 容器 =========
# 运行容器
docker run -d -p 主机端口:容器端口 --name 自定义名字 镜像名

# 查看正在运行的容器
docker ps

# 停止容器
docker stop 容器名/ID

# 删除容器
docker rm 容器名/ID
```

## 六、docker进阶
......