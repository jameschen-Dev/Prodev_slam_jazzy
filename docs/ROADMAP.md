# Prodev SLAM Jazzy 开发路线图

本文档记录项目的开发计划和功能路线图。

## 已完成

### v0.0.1（2026-06-25）
- [x] 初始项目结构
- [x] 基础 `Prodev_simulation` 功能包（URDF、Gazebo 仿真、空世界）

### v0.0.2（2026-06-30）
- [x] Docker 生产镜像和 `docker_run.sh` 便捷脚本
- [x] VS Code Dev Container 开发容器
- [x] `Prodev_bringup` 顶层启动包
- [x] `slam_maze.world` 迷宫仿真世界
- [x] GitHub Issue/PR 模板、贡献指南、GPL-3.0 许可证

### v0.0.3（2026-07-05）
- [x] Cartographer 2D SLAM 建图功能包 (`Prodev_slam`)
- [x] 一键启动仿真 + SLAM (`slam_sim.launch.py`)
- [x] SLAM 地图保存工作流（pbstream 格式）
- [x] DiffDrive TF 桥接修复、IMU 帧对齐
- [x] rviz2 SLAM 可视化配置

---

## 下一版本 v0.0.4 (2026-08-01) 
- [] Nav2 实现多点导航

---

## 进行中

### Nav2 自主导航
- **分支**: `feature/nav2-navigation`
- **目标**: 集成 Nav2 导航栈，实现自主路径规划和避障
- **进展**:
  - [x] 创建 `Prodev_nav2` 功能包框架
  - [x] Nav2 参数配置 (`nav2_params.yaml`)
  - [x] 导航启动文件 (`navigation.launch.py`)
  - [x] Dockerfile 集成 Nav2 源码构建
  - [ ] 验证 AMCL 定位精度
  - [ ] 调优路径规划和避障参数
  - [ ] 完善 Nav2 与 Cartographer 的协作流程
  - [ ] 合并到 main 分支

---

## 计划中

### 雷达标定
- **分支**: `lidar_calibration`
- **目标**: 对 LiDAR 传感器进行精确标定，修正安装偏移
- **计划**:
  - [ ] 设计标定流程和工具
  - [ ] 更新 `sensor_tf.launch.py` 中的标定值（当前为 TODO）
  - [ ] 文档化标定操作步骤

### 顶层系统集成 (`Prodev_bringup`)
- **目标**: 完善顶层启动包，统一管理系统各子模块
- **计划**:
  - [ ] 集成 SLAM 启动到 `prodev_bringup.launch.py`
  - [ ] 集成 Nav2 导航启动
  - [ ] 集成传感器标定 TF
  - [ ] 支持运行时参数化配置（SLAM/导航模式切换）

### 多传感器融合
- **目标**: 融合 LiDAR、IMU、相机、里程计数据，提升定位精度
- **计划**:
  - [ ] 评估 EKF/UKF 融合方案（robot_localization）
  - [ ] 相机视觉里程计集成
  - [ ] 多传感器融合参数调优

### SLAM 算法扩展
- **目标**: 支持更多 SLAM 算法，便于对比评估
- **计划**:
  - [ ] 集成 SLAM Toolbox（ROS2 原生）
  - [ ] 集成 RTAB-Map（视觉 SLAM）
  - [ ] SLAM 算法性能对比基准测试

### 仿真增强
- **目标**: 丰富仿真环境和测试场景
- **计划**:
  - [ ] 更多 Gazebo 世界文件（室内、室外、动态障碍物）
  - [ ] 多机器人仿真支持
  - [ ] 传感器噪声模型配置
  - [ ] 仿真与真实机器人部署的桥接

### 文档与测试
- **目标**: 完善项目文档和自动化测试
- **计划**:
  - [ ] 自动化 CI/CD 流水线（GitHub Actions）
  - [ ] 单元测试和集成测试
  - [ ] API 文档自动生成
  - [ ] 用户教程和最佳实践文档