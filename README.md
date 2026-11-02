# micromouse-ros2-course

ROS2 Humble 与 TurtleBot3 Burger 的地老鼠走迷宫课程工程，包含基础运动、雷达分区、左墙跟随、Pure Pursuit、里程计栅格建图和 A* 全局规划。

作者：向浩轩、杨晟。根据小组补充分工说明，实验总体规划、Ubuntu 实操、大部分代码、调试和报告由两位成员分工完成；工程开发阶段使用 AI 辅助部分相关代码。具体范围、修改与已有复验依据见 TEAM.md 和 NOTICE.md。

课程代码：`(2026-2027-1)-S0610350-04`；第三组；提交日期：2026年11月2日。真实仓库：<https://github.com/Bsnbj-nb/micromouse-ros2-course>。

已在 VMware 中的 Ubuntu 22.04.5、ROS2 Humble、Gazebo 11.10.2、gazebo_ros 3.9.0、turtlebot3_description 2.3.6、turtlebot3_gazebo 2.3.8 和 RViz 11.2.29 实际构建与运行；148 项自动检查通过。原始 ROS 记录包含配置、轨迹、状态、扫描地图、真值与 rosbag。单元检查与真实动力学运行分别保存，失败轮次没有删除。


## 当前源码与修订核验

本次归档已同步课程本地修订版的分工说明、退出修复、新增测试与 AI 辅助记录。源码包共 57 个文件，仅包含 `src/micromouse_course`，不含 build/install/log。

- [完整源码包](micromouse-source.zip)
- [逐文件 SHA-256 清单](source-manifest.json)
- [本次三项修订与 AI 辅助说明](revision_20261102.md)
- [Ubuntu 构建与154项检查日志](ubuntu-validation.log)
- [Ubuntu 新退出复验结果](shutdown-verification.json)
- [既有逐轮实测指标](measured-results.csv)

源码 ZIP SHA-256：`45f10cf03f2c20b214b69b6a81f38a8123fb8ed07a5cc9047ab1cdc05edf745d`。

下载后将 ZIP 中的 `src` 解压到自己的 ROS 工作空间，再按下文构建。R01 的 156.8 s 是退出修复后的独立回归记录，不替换下文2026年10月31日的性能对比数据。旧 C17 完成迷宫后的异常记录保留；当前修复及新的成功退出证据分别编号。

## 环境

- Ubuntu 22.04，系统 Python 3.10。
- ROS2 Humble。
- Gazebo Classic 11，使用 `gazebo_ros`，不是 `ros_gz`。
- ROBOTIS TurtleBot3 Burger 的 Humble 模型和仿真包。
- NumPy 仅用于几何回归工具；ROS 算法主体使用标准库。

Humble 仿真安装说明：<https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/>。
Classic 已于 2025 年 1 月结束生命周期；本工程使用它是为匹配课程环境。不要混用 Jazzy/Gazebo Sim 的模型插件。

## 安装依赖

在已经安装 ROS2 Humble 的 Ubuntu 22.04 中执行：

```bash
sudo apt update
sudo apt install ros-humble-gazebo-ros-pkgs ros-humble-turtlebot3-gazebo \
  ros-humble-turtlebot3-description ros-humble-xacro ros-humble-rviz2 \
  python3-colcon-common-extensions python3-pytest python3-numpy python3-yaml
mkdir -p ~/micromouse_course_ws
```

将源码压缩包中的 `src` 目录解压到 `~/micromouse_course_ws`。源码包不含第三方模型，不需要复制其他机器的 `build` 或 `install`。

## 编译与检查

```bash
source /opt/ros/humble/setup.bash
cd ~/micromouse_course_ws
bash src/micromouse_course/scripts/preflight.sh
colcon build --symlink-install --packages-select micromouse_course
source install/setup.bash
bash src/micromouse_course/scripts/check.sh
```

在新终端里重新 source `/opt/ros/humble/setup.bash` 和本工作空间的 `install/setup.bash`。不要在 Conda 环境中编译或运行本包。

## 统一启动入口

```bash
ros2 launch micromouse_course course.launch.py world:=course_maze mode:=idle
```

`world` 支持 `course_maze`、`control_arena`、`failure_loop`。

| mode | 功能 | 建议场景 |
|---|---|---|
| idle | 静止检查传感器与模型 | course_maze |
| open_loop | 按仿真时间直行与原地转向 | control_arena |
| point | 里程计点到点 P 控制 | control_arena |
| waypoints | 依次到达给定航点 | control_arena |
| avoid | 前方障碍反应式转向 | control_arena |
| wall_basic | 基础左墙跟随 | course_maze |
| wall_adaptive | 依据转向与前方净距调速 | course_maze |
| pure_pursuit | 基于折线投影和弧长前视的纯追踪 | control_arena |
| manual | 键盘请求经安全检查后输出 | 任一场景 |
| global | 接收 A* 路径并按折角航点跟踪 | 有观测地图的场景 |

一个控制节点独占 `/cmd_vel`。人工键盘使用 `ros2 run micromouse_course course_keyboard`，发送的是 `/teleop_cmd_vel`，不要再启动其他直接写 `/cmd_vel` 的节点。

其他参数：`mapping:=true` 启动建图；`planning:=true` 启动规划器；global 自动启动规划器；`rviz:=false` 或 `gui:=false` 减少显示负担；`params_file:=/绝对路径/参数.yaml` 覆盖算法参数；`loop_detection:=true` 开启重复访问检测。

## 建图与规划

```bash
ros2 launch micromouse_course course.launch.py mode:=wall_basic mapping:=true
```

完成观测后，在已 source 的另一个终端保存地图：

```bash
ros2 service call /save_map std_srvs/srv/Trigger "{}"
```

地图默认为 `~/micromouse_runs/map.json`、`map.pgm`、`map.yaml`。结束整套 launch，再从相同场景、出生位姿和 odom 原点重启：

```bash
ros2 launch micromouse_course course.launch.py mode:=global mapping:=false \
  map_file:=$HOME/micromouse_runs/map.json
```

发送任务场景坐标中的目标：

```bash
ros2 run micromouse_course send_goal --ros-args -p use_sim_time:=true -p x:=4.35 -p y:=3.5
```

也可在 Fixed Frame 为 `odom` 的 RViz 中使用 2D Goal Pose。不能未经转换把场景坐标直接当作 odom 坐标发送。

本实现是激光与里程计占据栅格，没有回环、漂移修正或重定位。规划时未知和证据不足区域默认不通行，地图不完整时先补观测。A* 不保证绕过动态障碍，前方安全距离触发时控制器停车，检查后调用 `/replan`。

## 记录与提交

每次启动在 `~/micromouse_runs/` 新建时间戳目录，保存 `trajectory.csv`、`run_config.json`、`summary.json`。跨算法对比应固定场景、出生位姿、路径、速度上限和地图参数，每组至少实际重复三次。

```bash
python3 src/micromouse_course/tools/analyse_runs.py --root ~/micromouse_runs \
  --out ~/micromouse_runs/comparison.csv
```

最小雷达距离不是车体净距，也不是碰撞计数。开环的 `task_completed` 只表示定时动作序列结束，不表示准确到达目标。检查报告和几何回归不能代替课程要求的 Gazebo 运行证据。

提交代码时仅打包 src：

```bash
python3 src/micromouse_course/tools/package_submission.py --group 第3组 --names 向浩轩_杨晟
```

报告和演示视频统一采用“第3组_向浩轩_杨晟”前缀。

## 已验证结果与限制

以下为代表轮次的真实仿真结果，时间采用 ROS 仿真时间：

| 轮次 | 模式 | 完成时间 / s | 轨迹长度 / m | 最小有效雷达距离 / m |
|---|---|---:|---:|---:|
| C02 | 点 P 控制 | 16.5 | 2.175 | 1.406 |
| C03 | 顺序航点 | 47.9 | 5.894 | 1.410 |
| C04 | Pure Pursuit | 54.3 | 5.777 | 1.414 |
| C10 | 固定速度左墙跟随 | 162.0 | 15.727 | 0.250 |
| C11 | 自适应速度左墙跟随 | 191.2 | 15.483 | 0.262 |
| C13 | 保存扫描地图上的 A* 跟踪 | 77.4 | 6.689 | 0.187 |

C06 的旧前墙阈值 0.30 m 导致掉头时触发近障停车并转弯超时。默认阈值已提前为 0.40 m，旋转安全门仍为 0.18 m。固定速度修正后的三轮 C10/C15/C17 均完成；完整重复数据在证据附件中。

自适应版本的近障距离更大，但总时间与采样角速度总变差也更大；不能声称所有指标都更优。Pure Pursuit 会切角，中间航点最小偏差约 0.084 m，不能代替基础航点模式的逐点 0.07 m 到达判据。全局规划使用既有扫描地图，其 77.4 s 不含建图成本。栅格建图没有回环校正，`failure_loop` 及重复访问检测没有完成独立脱困实测。

## 可复现实验脚本

在已 source 的终端中运行；同一时刻只启动一套 Gazebo：

```bash
python3 src/micromouse_course/tools/run_trial.py R01 control_arena point --record --seconds 90
python3 src/micromouse_course/tools/run_trial.py R02 course_maze wall_basic --record --mapping --seconds 500
python3 src/micromouse_course/tools/run_trial.py R03 course_maze global --record --seconds 300 \
  --map-file "$HOME/course_results/R02/map.json" --goal 4.35 3.5
```

结果默认保存至 `~/course_results/轮次名`，同名目录不会覆盖。可用 `--output-root` 改变保存位置。`--video --gui` 在 GNOME 桌面会先暂停物理、开始真实屏幕录制再恢复，需 `python3-dbus`、`wmctrl` 和已登录的图形会话。视频只用于展示，性能统计使用独立无界面轮次。

## 模块分工

向浩轩负责总体方案与系统集成、Ubuntu 环境、场景/模型和建图规划；杨晟负责运动控制、扫描处理、墙跟随及策略比较；两人共同完成实验规划、集成联调、证据核对与报告编写。具体修改及已有复验记录见 TEAM.md；运行日志不含操作者字段，人员归属按小组补充说明记录。

## 参考来源

1. Coulter R C. Implementation of the Pure Pursuit Path Tracking Algorithm[R]. Carnegie Mellon University, CMU-RI-TR-92-01, 1992. <https://publications.ri.cmu.edu/implementation-of-the-pure-pursuit-path-tracking-algorithm>。
2. Hart P E, Nilsson N J, Raphael B. A Formal Basis for the Heuristic Determination of Minimum Cost Paths[J]. IEEE Transactions on Systems Science and Cybernetics, 1968, 4(2): 100–107. DOI: <https://doi.org/10.1109/TSSC.1968.300136>。
3. Elfes A. Using occupancy grids for mobile robot perception and navigation[J]. Computer, 1989, 22(6): 46–57. DOI: <https://doi.org/10.1109/2.30720>。
4. ROBOTIS. TurtleBot3 Gazebo Simulation[EB/OL]. <https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/>。查阅日期：2026-10-31。
5. Open Robotics. ROS 2 Humble Quality of Service settings[EB/OL]. <https://github.com/ros2/ros2_documentation/blob/humble/source/Concepts/Intermediate/About-Quality-of-Service-Settings.rst>。查阅日期：2026-10-31。

## 开源许可与课程信息

仓库：<https://github.com/Bsnbj-nb/micromouse-ros2-course>。本项目原创代码按 MIT 许可证发布，已由项目提供者确认；第三方模型与依赖遵守其各自许可证。课程号为 (2026-2027-1)-S0610350-04，第三组，提交日期为 2026年11月2日。

## 本次核查补充

三组墙跟随对比中，C10/C11开启建图，C15/C16和C17/C18关闭建图；每组内部设置匹配，但六轮并非全部启动条件一致。C17完成运动后，truth_recorder在退出阶段仍报RuntimeError；clean_shutdown只表示轨迹记录器关闭，不保证所有进程无异常。本次仅修改说明文档，没有新增Gazebo实验，也没有更新公开仓库或虚拟机中的文件。

上一轮核查辅助说明（保留原记录）：本轮提交材料的要求核对、证据一致性检查和局部说明文字修订使用AI辅助；主体工作归属按小组补充说明记录。未新增Ubuntu/Gazebo实验，未改变原始数据。

## 2026年11月2日追加修订：退出修复及交付更新（AI 辅助）

此前说明及 C17 归档记录保留，以下为另一次修订。AI 编写了 `truth_recorder.py` 的退出时序修复和 6 项回归测试，在用户 Ubuntu 虚拟机中自动构建、运行并整理新证据；AI 还协助最终 Word 的原生 PDF 导出、字体核对，以及公开仓库说明、源码包和校验清单同步。新测试没有记作组员的人工操作。

Ubuntu 中 154 项检查通过。R01 新完整迷宫运行用时 156.8 s（仿真时间），记录节点正常退出；R02 在运动中由脚本发送相当于 Ctrl+C 的 SIGINT，正常退出且任务未完成；另外 12 次真实 ROS DDS 通信下的 SIGINT/SIGTERM 退出测试均返回 0，无 Traceback。后者使用专用消息发布器，不是 12 次迷宫仿真。详见 `revision_20261102.md`。
