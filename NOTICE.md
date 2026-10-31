# 来源、修改与许可

作者：向浩轩、杨晟。初始方案与部分代码由两位组员提供，经 AI 整理为 `Micromouse_Full_Task_20261031.zip`。本次继承其分层结构、单一速度出口、里程计转角闭环、开口去抖、扫描处理、占据栅格及 A* 思路，未把初始材料据为 AI 原创。

AI 辅助集成改动包括：官方 Xacro 展开及封装、Gazebo 真值只读记录、launch 参数适配、前墙转弯阈值由 0.30 调整为 0.40 m、退出清理、真实实验脚本与 GNOME 录屏。测试、日志、图表来自实际 Ubuntu 虚拟机。人工后续修改记录见 `docs/team.md`。

本项目原创代码按 MIT 许可证发布，见 LICENSE。机器人模型在运行时引用已安装的 ROBOTIS TurtleBot3 Burger Humble 资源，第三方网格和模型未复制到本包；其版权与 Apache-2.0 许可保持不变：

- https://github.com/ROBOTIS-GIT/turtlebot3/tree/humble
- https://github.com/ROBOTIS-GIT/turtlebot3_simulations/tree/humble
- https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/

Pure Pursuit、占据栅格和 A* 的原理引用见 README。维护者邮箱使用 GitHub 的 noreply 地址以避免公开个人联系邮箱。
