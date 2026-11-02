# 来源、修改与许可

作者：向浩轩、杨晟。根据小组补充分工，实验总体规划、Ubuntu 中的实际操作、大部分代码编写、调试、实验分析及报告撰写由两人分工完成。原工程采用算法与ROS适配分层、单一速度出口、里程计转角闭环、开口去抖、扫描分区、占据栅格和A*。

工程开发阶段使用AI辅助部分相关代码的实现建议与局部片段，范围包括Xacro展开、launch参数与记录节点接口、异常处理及进程退出等；采用的代码由对应模块负责人修改整合和验证。修改及归档复验依据见TEAM.md。运行数据保留原记录，不以本文档修订代替新增仿真。人员归属是小组提供的说明，日志未独立登记操作者。

本项目原创代码按 MIT 许可证发布，见 LICENSE。机器人模型在运行时引用已安装的 ROBOTIS TurtleBot3 Burger Humble 资源，第三方网格和模型未复制到本包；其版权与 Apache-2.0 许可保持不变：

- https://github.com/ROBOTIS-GIT/turtlebot3/tree/humble
- https://github.com/ROBOTIS-GIT/turtlebot3_simulations/tree/humble
- https://emanual.robotis.com/docs/en/platform/turtlebot3/simulation/

Pure Pursuit、占据栅格和 A* 的原理引用见 README。维护者邮箱使用 GitHub 的 noreply 地址以避免公开个人联系邮箱。

上一轮核查辅助说明（保留原记录）：本轮提交材料的要求核对、证据一致性检查和局部说明文字修订使用AI辅助；主体工作归属按小组补充说明记录。未新增Ubuntu/Gazebo实验，未改变原始数据。

## 2026年11月2日追加修订：退出修复及交付更新（AI 辅助）

此前说明及 C17 归档记录保留，以下为另一次修订。AI 编写了 `truth_recorder.py` 的退出时序修复和 6 项回归测试，在用户 Ubuntu 虚拟机中自动构建、运行并整理新证据；AI 还协助最终 Word 的原生 PDF 导出、字体核对，以及公开仓库说明、源码包和校验清单同步。新测试没有记作组员的人工操作。

Ubuntu 中 154 项检查通过。R01 新完整迷宫运行用时 156.8 s（仿真时间），记录节点正常退出；R02 在运动中由脚本发送相当于 Ctrl+C 的 SIGINT，正常退出且任务未完成；另外 12 次真实 ROS DDS 通信下的 SIGINT/SIGTERM 退出测试均返回 0，无 Traceback。后者使用专用消息发布器，不是 12 次迷宫仿真。详见 `revision_20261102.md`。
