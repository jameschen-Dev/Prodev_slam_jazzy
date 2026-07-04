# Prodev SLAM Jazzy

基于 ROS2 Jazzy 的 SLAM 仿真项目，使用 Ubuntu 24.04 和 Gazebo Sim (gz sim) 进行机器人仿真。

## 项目结构

```
Prodev_slam_jazzy/
├── Prodev_bringup/          # 顶层系统启动功能包
│   ├── config/              # 系统配置文件
│   ├── launch/              # 顶层 launch 文件
│   └── rviz/                # RViz 配置文件
├── Prodev_simulation/       # 仿真功能包
│   ├── config/              # 配置文件 (TF 参数等)
│   ├── launch/              # Launch 启动文件
│   ├── urdf/                # 机器人 URDF 模型
│   └── worlds/              # Gazebo 世界文件
├── Prodev_slam/             # SLAM 功能包 (Cartographer)
│   ├── config/              # Cartographer 配置文件
│   ├── launch/              # SLAM 启动文件
│   └── rviz/                # SLAM 可视化配置
├── .devcontainer/           # VS Code Dev Container 配置
├── docs/                    # 项目文档
├── scripts/                 # 脚本工具
├── tools/                   # 工具
└── demo/                    # 演示
```

## 环境要求

- **操作系统**: Ubuntu 24.04 (Noble)
- **ROS2 版本**: Jazzy Jalisco
- **仿真器**: Gazebo Sim (gz sim)
- **构建工具**: colcon
- **GPU 支持**（可选）：NVIDIA 驱动 + [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)（用于 Docker 内 GPU 加速渲染）

## 机器人模型

项目使用差速驱动的圆柱底盘机器人，配备以下传感器：

| 传感器 | 话题 | 规格 |
|--------|------|------|
| LiDAR | `/scan` | 360° 激光雷达，0.1-10m，10Hz |
| IMU | `/imu` | 100Hz |
| Odometry | `/odom` | 差速驱动里程计，100Hz |
| Camera | `/camera` | 640×480 RGB，30Hz |

**运动控制**：通过 `/cmd_vel`（`geometry_msgs/msg/Twist`）发送速度命令。

## Docker 部署

项目提供两种 Docker 使用方式：

1. **本地生产镜像**：仓库外层的 `Dockerfile`（以 `~/Prodev_jazzy` 为构建上下文），用于构建可独立运行的镜像。
2. **Dev Container**：`.devcontainer/Dockerfile`，用于 VS Code 开发容器，挂载本地源码进行开发。

### 构建本地镜像

在仓库外层目录（包含 `src/Prodev_slam_jazzy`）执行：

```bash
docker build -t prodev_jazzy .
```

> **国内用户推荐**：使用中科大镜像源加速构建
> ```bash
> bash src/Prodev_slam_jazzy/scripts/docker_run.sh --build --mirror ustc
> ```

### 运行容器

**基本运行：**

```bash
docker run -it --rm --name prodev_jazzy_container prodev_jazzy
```

**支持 GUI 显示（RViz2 / Gazebo）：**

```bash
# 允许 Docker 访问 X11 显示
xhost +local:docker

docker run -it --rm --name prodev_jazzy_container \
    --env="DISPLAY=$DISPLAY" \
    --env="QT_X11_NO_MITSHM=1" \
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    --gpus all \
    prodev_jazzy
```

**挂载本地代码（开发模式）：**

```bash
docker run -it --rm --name prodev_jazzy_container \
    --volume="$(pwd)/src/Prodev_slam_jazzy/Prodev_simulation:/ros2_ws/src/Prodev_simulation" \
    --volume="$(pwd)/src/Prodev_slam_jazzy/Prodev_bringup:/ros2_ws/src/Prodev_bringup" \
    --volume="$(pwd)/src/Prodev_slam_jazzy/Prodev_slam:/ros2_ws/src/Prodev_slam" \
    --env="DISPLAY=$DISPLAY" \
    --env="QT_X11_NO_MITSHM=1" \
    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    --gpus all \
    prodev_jazzy
```

### 便捷脚本

使用 `scripts/docker_run.sh` 快速启动：

```bash
# 基本运行
bash src/Prodev_slam_jazzy/scripts/docker_run.sh

# 启用 GUI（RViz2 / Gazebo），自动挂载 NVIDIA GPU
bash src/Prodev_slam_jazzy/scripts/docker_run.sh --gui

# 开发模式（挂载本地源码）
bash src/Prodev_slam_jazzy/scripts/docker_run.sh --gui --dev

# 强制重新构建镜像
bash src/Prodev_slam_jazzy/scripts/docker_run.sh --build --gui

# 无缓存构建（源码有变更时使用）
bash src/Prodev_slam_jazzy/scripts/docker_run.sh --no-cache --gui --mirror ustc

# 使用中科大镜像源构建（适合国内网络）
bash src/Prodev_slam_jazzy/scripts/docker_run.sh --build --mirror ustc

# WSL2 下启用 GUI
bash src/Prodev_slam_jazzy/scripts/docker_run.sh --gui --wsl
```

**参数说明：**

- `--build`：强制重新构建 Docker 镜像
- `--no-cache`：无缓存重新构建（源码有变更时使用，隐含 `--build`）
- `--gui`：启用 X11 转发，支持 RViz2 / Gazebo 显示
- `--dev`：挂载本地 `Prodev_simulation`、`Prodev_bringup` 和 `Prodev_slam` 源码
- `--mirror <official|ustc>`：选择 apt 镜像源，默认 `official`
- `--wsl`：针对 Windows WSL2 调整 X11 和 GPU 参数

> **WSL2 使用 GUI 注意**：Windows 宿主机需要运行 X Server（如 VcXsrv、Xming），并开启 "Disable access control"。

### 常用 Docker 命令

```bash
# 查看运行中的容器
docker ps

# 进入已运行的容器
docker exec -it prodev_jazzy_container bash

# 停止容器
docker stop prodev_jazzy_container

# 删除容器
docker rm prodev_jazzy_container

# 重新构建镜像（代码有变更时）
docker build --no-cache -t prodev_jazzy .
```

## 启动仿真

进入容器后：

```bash
# Source 工作空间
source /ros2_ws/install/setup.bash
```

### 方式一：完整仿真 + SLAM 一键启动

```bash
ros2 launch Prodev_slam slam_sim.launch.py
```

### 方式二：分别启动

```bash
# 终端1：启动 Gazebo 仿真
ros2 launch Prodev_simulation gazebo_sim.launch.py

# 终端2：启动 Cartographer SLAM（含 rviz2 可视化）
ros2 launch Prodev_slam cartographer.launch.py
```

### 键盘控制机器人

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

操作方式：`i` 前进 / `,` 后退 / `j` 左转 / `l` 右转 / `k` 停止

### 保存地图

建图完成后，通过 Cartographer 服务保存地图：

```bash
# 冻结当前轨迹
ros2 service call /finish_trajectory cartographer_ros_msgs/srv/FinishTrajectory "{trajectory_id: 0}"

# 保存为 pbstream 格式（容器内）
ros2 service call /write_state cartographer_ros_msgs/srv/WriteState "{filename: '/ros2_ws/map.pbstream'}"
```

将地图从容器复制到主机：

```bash
docker cp prodev_jazzy_container:/ros2_ws/map.pbstream ~/map.pbstream
```

> 已保存的地图示例见 `Prodev_slam/maps/map.pbstream`。

### 其他启动方式

```bash
# 仅启动仿真（不含 SLAM）
ros2 launch Prodev_simulation gazebo_sim.launch.py

# 启动整体系统 bringup（顶层入口）
ros2 launch Prodev_bringup prodev_bringup.launch.py
```

## Dev Container

使用 VS Code 打开本项目，选择 "Reopen in Container"，即可在预配置好的容器中进行开发。容器会自动挂载 `Prodev_slam_jazzy` 到 `/ros2_ws/src/Prodev_slam_jazzy` 并执行 `colcon build`。

## License

本项目采用 [GNU General Public License v3.0 (GPL-3.0)](./LICENSE) 开源协议。

This project is licensed under the [GNU General Public License v3.0 (GPL-3.0)](./LICENSE).
