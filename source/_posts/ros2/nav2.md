---
title: ros2-源码编译ros2二进制安装nav2
top_img: /img/background.png
cover: /img/unhappy.jpg
date: 2025-08-18 17:19:41
tags: ros2
category: ros2
---

# 源码编译安装ROS2 humble

环境： Ubuntu Linux - Jammy (22.04) 64-bit

## 系统设置

根据[官网手册](https://docs.ros.org/en/humble/Installation/Alternatives/Ubuntu-Development-Setup.html)，首先进行系统设置

```shell
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```

然后添加ROS2 apt源，按照官网手册添加的官方ROS2apt源可能会出现密钥不正确或者下载非常慢的情况，因此添加ROS2镜像源。别忘了先进行系统更新和安装依赖项。参考[鱼香ROS社区](https://fishros.org.cn/forum/topic/3050/ros2%E5%AE%89%E8%A3%85/2)。

```shell
# 添加ROS2密钥
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

#添加ROS2软件源列表
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null。
```

接着下载开发工具和ROS工具

```shell
sudo apt update && sudo apt install -y \
  python3-flake8-docstrings \
  python3-pip \
  python3-pytest-cov \
  ros-dev-tools
```

因为主包系统是ubuntu22.04，按照官网步骤，还需要安装以下包

```shell
sudo apt install -y \
   python3-flake8-blind-except \
   python3-flake8-builtins \
   python3-flake8-class-newline \
   python3-flake8-comprehensions \
   python3-flake8-deprecated \
   python3-flake8-import-order \
   python3-flake8-quotes \
   python3-pytest-repeat \
   python3-pytest-rerunfailures
```

## ROS2安装与编译

第一步就是创建一个工作空间并且克隆所有的ROS2 repos。

```shell
mkdir -p ~/ros2_humble/src
cd ~/ros2_humble
vcs import --input https://raw.githubusercontent.com/ros2/ros2/humble/ros2.repos src

#可能会出现的问题，raw.githubusercontent.com的DNS被污染，国内很难访问。
#主包是改了Hosts文件
sudo vim /etc/hosts
#添加以下内容，但主包同时也进行了科学上网
185.199.108.133 raw.githubusercontent.com
```

然后使用rosdep安装依赖，也是要先更新系统，然后进行rosdep初始化和更新，最后下载依赖

```shell
sudo apt upgrade

sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"
```

最后就是编译源码啦

```shell
cd ~/ros2_humble/
colcon build --symlink-install --packages-skip-build-finished
```

# 二进制安装nav2

虽然说好像源码编译安装ROS2后，再源码编译nav2比较好，但是主包对于nav2并没有源码查看修改等需求，并且源码编译时间比较久，主包目前要配置的硬件机器性能有点子差，所以编译就需要非常多的时间，所以就选择二进制安装了。

这里安装就非常简单了，只需要下载对应ros2版本的包就行。

如果是跟主包一样是源码编译安装的ros2，那这里安装包的过程中会出现很多“ros-humble-*”样式的包提示要下载，那是因为apt并不知道我们已经安装了ros2了，所以它还需要下载一些nav2运行需要的依赖，但并没有完全将ros2核心下载下来（主包是这么理解的，不知道对不对）。

```shell
sudo apt install ros-<ros2-distro>-navigation2
sudo apt install ros-<ros2-distro>-nav2-bringup
```

别忘了装Turtlebot 3软件包（Humble和更早版本）：

```shell
sudo apt install ros-<ros2-distro>-turtlebot3-gazebo
```

# 运行示例

根据[官网运行示例](https://fishros.org/doc/nav2/getting_started/index.html#installation)，我们需要先source以下ros环境

```shell
source /opt/ros/<ros2-distro>/setup.bash
```

但是，主包并不是直接二进制安装的ros2，所以使用这个命令去source ros2是不行的，source之后也会提示找不到ros2命令，还需要source一下我们之前源码编译下载的ros2

```shell
source ~/ros2_humble/install/setup.bash
export TURTLEBOT3_MODEL=waffle
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:/opt/ros/<ros2-distro>/share/turtlebot3_gazebo/models

ros2 launch nav2_bringup tb3_simulation_launch.py headless:=False
```
这样就可以看到和官网运行示例一样的效果啦。
