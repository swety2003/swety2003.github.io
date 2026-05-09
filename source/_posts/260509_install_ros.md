---
title: 安装 ros2 和 moveit
date: 2026-05-06
categories:
- ros2
tags:
- ros2
- moveit2
---

> 参考自实习的同事的工作日志

## 系统准备
使用 distrobox 创建容器，搞坏了直接删了完事

```bash
distrobox create --name ubuntu --image docker.xuanyuan.me/ubuntu:24.04 --home /home/sub/ubuntu --hostname ubuntu
```
## 软件包安装
参考链接 [官方文档](https://docs.ros.org/en/humble/Installation.html)

### 设置 locale
```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```
### 设置软件源
```bash
sudo apt install software-properties-common
sudo add-apt-repository universe

sudo apt update && sudo apt install curl -y
export ROS_APT_SOURCE_VERSION=$(curl -s https://api.github.com/repos/ros-infrastructure/ros-apt-source/releases/latest | grep -F "tag_name" | awk -F'"' '{print $4}')
curl -L -o /tmp/ros2-apt-source.deb "https://github.com/ros-infrastructure/ros-apt-source/releases/download/${ROS_APT_SOURCE_VERSION}/ros2-apt-source_${ROS_APT_SOURCE_VERSION}.$(. /etc/os-release && echo ${UBUNTU_CODENAME:-${VERSION_CODENAME}})_all.deb"
sudo dpkg -i /tmp/ros2-apt-source.deb
```
### 安装 ROS 2 和 moveit
```bash
sudo apt update
sudo apt install ros-jazzy-moveit  ros-jazzy-moveit 
```

### 激活环境
```bash
source /opt/ros/jazzy/setup.bash
```

## 安装 moveit2_tutorials
apt源没有 moveit2_tutorials，需要自己从源码编译

### 前置软件包
需要 rosdep 来解决依赖
rosdepc 为 rosdep 的包装，下载加速用

```bash
pip install rosdepc
```
我直接安装会被系统禁止，经过ai提醒可以用 pipx 自动创建虚拟环境

```bash
pipx install rosdepc rosdep --pip-args="-i https://pypi.tuna.tsinghua.edu.cn/simple"

#  指定用户 pipx 路径（可选，若 sudo 找不到文件）
sudo env PATH="$PATH:/home/$USER/.local/bin" rosdepc
sudo rosdep init rosdep update

sudo apt update
sudo apt dist-upgrade
```

### 配置 Colcon 和 安装 vcstool
```bash
sudo apt install python3-colcon-common-extensions
sudo apt install python3-colcon-mixin
colcon mixin add default https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml
colcon mixin update default

sudo apt install python3-vcstool
```

### 拉取源码

参考 [moveit2_tutorials](https://github.com/moveit/moveit2_tutorials) README **注意版本是 jazzy**

```bash
mkdir -p ~/ws_moveit2/src
cd ~/ws_moveit2/src
git clone https://github.com/ros-planning/moveit2_tutorials.git 
vcs import < moveit2_tutorials/moveit2_tutorials.repos
sudo apt update && rosdep install -r --from-paths . --ignore-src --rosdistro jazzy -y
```
### 编译

```bash
# 不要在 src 目录编译
cd ../
colcon build --mixin release
```

### 完成
如果一切正常，测试一下功能
```bash
source /opt/ros/jazzy/setup.bash
source ~/ws_moveit2/install/setup.bash

ros2 launch moveit_servo demo_ros_api.launch.py

# 另一个窗口
ros2 run moveit_servo servo_keyboard_input
```

非常的完美

![img](/images/ros2_install_image.png)

