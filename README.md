# rl_sar

> [!IMPORTANT]
> **声明：** 本仓库代码源自 [rl_sar](https://github.com/fan-ziqi/rl_sar)，仅用于个人修改、训练及学习目的。

[English document](README.md)

本仓库提供了机器人强化学习算法的仿真验证与实物部署框架，适配四足机器人、轮足机器人、人形机器人。"sar"代表"simulation and real"。

---

## 开发历程与操作日志 (Development Log)

### 步骤 1：仿真场景参数审计与物理模型对齐
* **时间戳**：2026-06-22
* **目的作用**：在进行 Sim2Sim 仿真演示前，定位 MuJoCo 与 Gazebo 场景文件，精确计算台阶高度，为步态一致性测试提供几何参考。
* **关键文件**：
  * MuJoCo 场景：`src/rl_sar_zoo/d1_description/mjcf/scene.xml`
  * Gazebo 场景：`src/rl_sar/worlds/stairs.world`
* **底层物理/数学原理**：
  * **MuJoCo (MJCF)**：长方体几何 `<geom type="box">` 的 `size` 属性代表 **半长、半宽、半高 (half-lengths)**。绝对上表面高度计算公式：$Z_{top} = pos.z + size.z$。第一级台阶实际起步高度为 $0.02m + 0.15m = 0.17m$，其余台阶相对高度增量为 $0.15m$。
  * **Gazebo (SDF)**：长方体几何 `<box><size>` 属性代表 **真实高 (full-lengths)**。绝对上表面高度计算公式：$Z_{top} = pos.z + \frac{size.z}{2}$。台阶相对高度增量为 $0.15m$。

### 步骤 2：Git 远程仓库所有权迁移与配置
* **时间戳**：2026-06-22
* **目的作用**：将代码库置于个人 GitHub 空间管理下，建立规范的 Git 开发工作流。
* **执行命令**：
  ```bash
  git remote rename origin upstream
  git remote add origin git@github.com:fireowl-sw/rl_sar.git
  git push -u origin --all
  git push -u origin --tags
  ```
* **产生的变化**：原开源项目重命名为 `upstream` 用于跟踪上游更新；新增个人 GitHub 命名空间 `fireowl-sw` 下的 `rl_sar` 仓库为 `origin` 主仓库，完成全部代码与分支推送。

### 步骤 3：Sim2Sim 本地仿真演示与闭环控制测试
* **时间戳**：2026-06-22
* **目的作用**：在 Windows 本地 WSL2 环境下启动双仿真引擎，调用宿主机 GPU 渲染，运行内置的智元 D1 强化学习控制权重，完成 Sim2Sim 步态控制闭环演示。
* **执行命令**：
  * **MuJoCo 演示**：
    ```bash
    ./cmake_build/bin/rl_sim_mujoco d1 scene
    ```
  * **Gazebo 演示**：
    ```bash
    ros2 launch rl_sar gazebo.launch.py rname:=d1
    ros2 run rl_sar rl_sim
    ```

---

## 准备工作 (Preparation)

克隆本仓库及子模块：

```bash
git clone --recursive --depth 1 https://github.com/fan-ziqi/rl_sar.git
```

更新仓库及子模块至最新版本：

```bash
git pull
git submodule update --init --recursive --recommend-shallow --progress
```

## 依赖项 (Dependency)

安装系统所需依赖环境（支持 Ubuntu 或 macOS）：

```bash
# Ubuntu
sudo apt install cmake g++ build-essential libyaml-cpp-dev libeigen3-dev libboost-all-dev libspdlog-dev libfmt-dev libtbb-dev liblcm-dev

# macOS
brew install boost lcm yaml-cpp tbb libomp pkg-config glfw
```

若需运行 ROS/ROS2 仿真环境，请安装以下适配的依赖包：

```bash
# ros-noetic (Ubuntu20.04)
sudo apt install ros-noetic-teleop-twist-keyboard ros-noetic-controller-interface ros-noetic-gazebo-ros-control ros-noetic-joint-state-controller ros-noetic-effort-controllers ros-noetic-joint-trajectory-controller ros-noetic-joy ros-noetic-ros-control ros-noetic-ros-controllers ros-noetic-controller-manager

# ros2-foxy (Ubuntu20.04) / ros2-humble (Ubuntu22.04)
sudo apt install ros-$ROS_DISTRO-teleop-twist-keyboard ros-$ROS_DISTRO-ros2-control ros-$ROS_DISTRO-ros2-controllers ros-$ROS_DISTRO-control-toolbox ros-$ROS_DISTRO-robot-state-publisher ros-$ROS_DISTRO-joint-state-publisher-gui ros-$ROS_DISTRO-gazebo-ros2-control ros-$ROS_DISTRO-gazebo-ros-pkgs ros-$ROS_DISTRO-xacro
```

## 编译项目 (Compilation)

在项目根目录下运行编译脚本：

```bash
./build.sh
```

如需单独编译特定功能包，可以在命令后附加包名：

```bash
./build.sh package1 package2
```

清理全部编译输出和生成的软链接：

```bash
./build.sh -c  # 或 ./build.sh --clean
```

如果不需要仿真环境而只进行硬件实物部署（仅使用 CMake 编译）：

```bash
./build.sh -m  # 或 ./build.sh --cmake
```

开启 MuJoCo 仿真器支持进行编译：

```bash
./build.sh -mj  # 或 ./build.sh --mujoco
```

查看编译脚本的详细参数帮助：

```bash
Usage: ./build.sh [OPTIONS] [PACKAGE_NAMES...]

Options:
  -c, --clean    Clean workspace (remove symlinks and build artifacts)
  -m, --cmake    Build using CMake (for hardware deployment only)
  -mj,--mujoco   Build with MuJoCo simulator support (CMake only)"
  -h, --help     Show this message
```

## 运行仿真与程序 (Running)

请将训练好的权重文件复制到指定目录 `policy/<ROBOT>/<CONFIG>`，并对应修改 `policy/<ROBOT>/base.yaml` 及 `config.yaml` 配置文件。

### 仿真运行 (Simulation)

#### Gazebo 仿真

启动 Gazebo 仿真物理场景：

```bash
# ROS1
source devel/setup.bash
roslaunch rl_sar gazebo.launch rname:=<ROBOT>

# ROS2
source install/setup.bash
ros2 launch rl_sar gazebo.launch.py rname:=<ROBOT>
```

打开新终端，启动强化学习控制节点：

```bash
# ROS1
source devel/setup.bash
rosrun rl_sar rl_sim

# ROS2
source install/setup.bash
ros2 run rl_sar rl_sim
```

首次运行 Gazebo 时若无法正常显示模型，需下载官方模型包：

```bash
git clone https://github.com/osrf/gazebo_models.git ~/.gazebo/models
```

#### Mujoco 仿真

运行基于 MuJoCo 的物理仿真程序，需传入机器人名称与场景名称：

```bash
./cmake_build/bin/rl_sim_mujoco <ROBOT> <SCENE>
# 例如: ./cmake_build/bin/rl_sim_mujoco g1 scene_29dof
```

#### Docker 容器运行

在 Docker 容器中运行仿真与测试：

```bash
# 启动容器并启用 X11 转发显示
xhost +local:docker
cd docker && docker compose up -d
docker compose exec rl_sar bash

# 在容器内启动 MuJoCo 仿真
./cmake_build/bin/rl_sim_mujoco g1 scene_29dof

# 在容器内启动 Gazebo 仿真
ros2 launch rl_sar gazebo.launch.py rname:=go2
# (新终端中运行) ros2 run rl_sar rl_sim

# 在容器内启动实物控制
./cmake_build/bin/rl_real_go2 <NETWORK_INTERFACE>
```

### 网页移动端控制 (Mobile Web Control)

安装移动端网页控制桥接所需的依赖包：

```bash
sudo apt install ros-${ROS_DISTRO}-rosbridge-suite
sudo apt install ros-${ROS_DISTRO}-web-video-server

# 若使用 Humble/Jazz/Rolling 以外的 ROS2 版本，需从源码编译 web_video_server
cd <your_ros2_workspace>/src
git clone https://github.com/RobotWebTools/web_video_server.git
cd <your_ros2_workspace>
colcon build --packages-select web_video_server
```

在机器人端运行控制桥接节点：

```bash
# ROS1
roslaunch rosbridge_server rosbridge_websocket.launch
rosrun web_video_server web_video_server

# ROS2
ros2 launch rosbridge_server rosbridge_websocket_launch.xml
ros2 run web_video_server web_video_server
```

在手机端浏览器访问 [http://robot.robotsfan.com/](http://robot.robotsfan.com/)，输入机器人 IP 与端口进行连接和方向手势控制。

### 键盘及手柄控制映射 (Gamepad & Keyboard Control)

仿真与实物控制过程中的键盘/手柄键位映射表如下：

| 手柄控制 (Gamepad) | 键盘控制 (Keyboard) | 功能描述 |
|---|---|---|
|**基础状态控制**|||
|A|Num0|插值驱动机器人从程序初始位姿站立到 `base.yaml` 中的默认关节位置 `default_dof_pos`|
|B|Num9|插值驱动机器人从当前位姿安全趴下并回到程序初始位姿|
|X|N|开启导航模式（屏蔽手柄摇杆输入，接受外部 `cmd_vel` 速度话题控制）|
|Y|N/A|N/A|
|**仿真操作**|||
|RB+Y|R|重置 Gazebo 仿真环境（当机器人摔倒时使其复位站立）|
|RB+X|Enter|开启/暂停 Gazebo 物理仿真循环（默认：开启状态）|
|**电机使能控制**|||
|LB+A|M|电机使能 (Motor enable)|
|LB+B|K|电机脱机/失能 (Motor disable)|
|LB+X|P|电机阻尼模式/被动状态 (Motor passive mode: `kp=0, kd=8`)|
|LB+RB|N/A|机器人紧急急停销控 (Emergency stop)|
|**步态与动作技能切换**|||
|RB+DPadUp|Num1|基础行走步态 (Basic Locomotion)|
|RB+DPadDown|Num2|技能动作 2 (Skill 2)|
|RB+DPadLeft|Num3|技能动作 3 (Skill 3)|
|RB+DPadRight|Num4|技能动作 4 (Skill 4)|
|LB+DPadUp|Num5|技能动作 5 (Skill 5)|
|LB+DPadDown|Num6|技能动作 6 (Skill 6)|
|LB+DPadLeft|Num7|技能动作 7 (Skill 7)|
|LB+DPadRight|Num8|技能动作 8 (Skill 8)|
|**运动速度输入**|||
|LY 摇杆|W/S|前后移动速度控制 (X 轴)|
|LX 摇杆|A/D|左右平移速度控制 (Y 轴)|
|RX 摇杆|Q/E|偏航角角速度控制 (Yaw 旋转)|
|摇杆回中|Space|清除当前全部速度控制命令为 0|

### 物理实物部署 (Real Robots)

<details>

<summary>Unitree A1 控制部署 (点击展开)</summary>

连接网线配置 PC 的 IP 处于 `192.168.123.X` 网段，并运行实物部署程序：

```bash
# ROS1
source devel/setup.bash
rosrun rl_sar rl_real_a1

# ROS2
source install/setup.bash
ros2 run rl_sar rl_real_a1

# CMake
./cmake_build/bin/rl_real_a1
```

</details>

<details>

<summary>Unitree Go2/Go2W/G1(29dofs) 控制部署 (点击展开)</summary>

#### 网线连接

连接网线并配置电脑 USB 以太网 IP 为 `192.168.123.222`（网关为 `255.255.255.0`）。查询网卡接口名称，并启动对应的控制程序：

```bash
# ROS1
source devel/setup.bash
rosrun rl_sar rl_real_go2 <YOUR_NETWORK_INTERFACE> [wheel]

# ROS2
source install/setup.bash
ros2 run rl_sar rl_real_go2 <YOUR_NETWORK_INTERFACE> [wheel]

# CMake
./cmake_build/bin/rl_real_go2 <YOUR_NETWORK_INTERFACE> [wheel]
```

G1 人形机器人 (29dofs)：

```bash
# ROS1
source devel/setup.bash
rosrun rl_sar rl_real_g1 <YOUR_NETWORK_INTERFACE>

# ROS2
source install/setup.bash
ros2 run rl_sar rl_real_g1 <YOUR_NETWORK_INTERFACE>

# CMake
./cmake_build/bin/rl_real_g1 <YOUR_NETWORK_INTERFACE>
```

#### 在板载电脑 (Jetson) 部署

通过 SSH 登录机器人板载电脑，在板载端构建并运行控制程序：

```bash
ssh unitree@192.168.123.18
```

```bash
# 编译并运行
./build.sh -m
./cmake_build/bin/rl_real_go2 <YOUR_NETWORK_INTERFACE> [wheel]
```

#### 开机自启动配置

编写 systemd 服务文件配置自启动：

```bash
sudo touch /etc/systemd/system/rl_sar.service
```

在自启动服务文件中填入如下配置：

```
[Unit]
Description=RL SAR Service
After=network.target

[Service]
Type=simple
User=unitree
WorkingDirectory=/home/unitree/rl_sar
ExecStart=/home/unitree/rl_sar/cmake_build/bin/rl_real_go2 eth0 wheel
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

通过 systemctl 进行开机启动的生命周期管理：

```bash
# 重载系统服务
sudo systemctl daemon-reload

# 开启开机自启动
sudo systemctl enable rl_sar.service

# 关闭开机自启动
sudo systemctl disable rl_sar.service

# 启动服务
sudo systemctl start rl_sar.service

# 停止服务
sudo systemctl stop rl_sar.service

# 重启服务
sudo systemctl restart rl_sar.service

# 查看日志
sudo journalctl -u rl_sar.service -f
```

</details>

<details>

<summary>Deeprobotics Lite3 控制部署 (点击展开)</summary>

连接 WiFi，设置 `rl_real_lite3.cpp` 及 `network.toml` 中配置对应的 IP，启动控制程序：

```bash
# ROS1
source devel/setup.bash
rosrun rl_sar rl_real_lite3

# ROS2
source install/setup.bash
ros2 run rl_sar rl_real_lite3

# CMake
./cmake_build/bin/rl_real_lite3
```

</details>

<details>

<summary>Agibot D1 人形机器人控制部署 (点击展开)</summary>

配置网络处于同网段下，修改机器人侧 `/opt/export/config/sdk_config.yaml` 文件的 `target_ip` 指向您的 PC 地址，并运行程序：

```bash
ssh firefly@192.168.234.1  # 密码: firefly
```

```bash
vim /opt/export/config/sdk_config.yaml
```

执行实物控制节点：

```bash
# ROS1
source devel/setup.bash
rosrun rl_sar rl_real_d1 [local_ip] [robot_ip]

# ROS2
source install/setup.bash
ros2 run rl_sar rl_real_d1 [local_ip] [robot_ip]

# CMake
./cmake_build/bin/rl_real_d1 [local_ip] [robot_ip]

# 例如（使用默认 IP 执行）
./cmake_build/bin/rl_real_d1 192.168.234.2 192.168.234.1
```

</details>

### 执行机构/关节动力学网络训练 (Actuator Network Training)

收集机器人关节运动数据记录为 `motor.csv` 并进行网络训练和离线重放：

```bash
# 训练动力学网络
rosrun rl_sar actuator_net.py --mode train --data a1/motor.csv --output a1/motor.pt

# 验证与回放动力学仿真
rosrun rl_sar actuator_net.py --mode play --data a1/motor.csv --output a1/motor.pt
```

## 新建机器人适配 (Add Your Robot)

为了接入新的自定义机器人，您需要创建并配置如下的机器人描述文件、策略网络、状态机和运行节点：

```yaml
# 机器人几何描述配置文件
rl_sar/src/rl_sar_zoo/<ROBOT>_description/CMakeLists.txt
rl_sar/src/rl_sar_zoo/<ROBOT>_description/package.ros1.xml
rl_sar/src/rl_sar_zoo/<ROBOT>_description/package.ros2.xml
rl_sar/src/rl_sar_zoo/<ROBOT>_description/xacro/robot.xacro
rl_sar/src/rl_sar_zoo/<ROBOT>_description/xacro/gazebo.xacro
rl_sar/src/rl_sar_zoo/<ROBOT>_description/config/robot_control.yaml
rl_sar/src/rl_sar_zoo/<ROBOT>_description/config/robot_control_ros2.yaml

# 策略控制与网络配置文件
policy/<ROBOT>/base.yaml
policy/<ROBOT>/<CONFIG>/config.yaml
policy/<ROBOT>/<CONFIG>/<POLICY>.pt
policy/<ROBOT>/<CONFIG>/<POLICY>.onnx

# 机器人有限状态机代码
src/rl_sar/fsm_robot/fsm_<ROBOT>.hpp
src/rl_sar/fsm_robot/fsm_all.hpp

# 实物控制源文件
rl_sar/src/rl_sar/src/rl_real_<ROBOT>.cpp
```

## 贡献 (Contributing)

本框架欢迎并感谢社区的任何贡献，如 Bug 反馈、新增机器人适配或代码 Pull Request。

## 引用 (Citation)

如果您在您的研究中使用本仓库代码，请按以下格式引用：

```
@software{fan-ziqi2024rl_sar,
  author = {fan-ziqi},
  title = {rl_sar: Simulation Verification and Physical Deployment of Robot Reinforcement Learning Algorithm.},
  url = {https://github.com/fan-ziqi/rl_sar},
  year = {2024}
}
```

## 致谢 (Acknowledgements)

本项目对以下优秀的机器人开源项目及其作者表示诚挚的感谢：

- [unitreerobotics/unitree_sdk2-2.0.0](https://github.com/unitreerobotics/unitree_sdk2/tree/2.0.0)
- [unitreerobotics/unitree_legged_sdk-v3.2](https://github.com/unitreerobotics/unitree_legged_sdk/tree/v3.2)
- [unitreerobotics/unitree_guide](https://github.com/unitreerobotics/unitree_guide)
- [unitreerobotics/unitree_mujoco](https://github.com/unitreerobotics/unitree_mujoco)
- [google-deepmind/mujoco-3.2.7](https://github.com/google-deepmind/mujoco)
- [mertgungor/unitree_model_control](https://github.com/mertgungor/unitree_model_control)
- [Improbable-AI/walk-these-ways](https://github.com/Improbable-AI/walk-these-ways)
- [ccrpRepo/RoboMimic_Deploy](https://github.com/ccrpRepo/RoboMimic_Deploy)
- [Deeprobotics/Lite3_Motion_SDK](https://github.com/DeepRoboticsLab/Lite3_MotionSDK)
- [chengyangkj/ROS_Flutter_Gui_App](https://github.com/chengyangkj/ROS_Flutter_Gui_App)
