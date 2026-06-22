# 开发历程与操作日志

- [2026-06-22] 步骤 1：仿真场景及台阶高度审计
  - **MuJoCo 场景**：
    - 路径：`src/rl_sar_zoo/d1_description/mjcf/scene.xml`
    - 级高：6级台阶，高度列表 `[0.17, 0.32, 0.47, 0.62, 0.77, 0.92]m`（首级起步 0.17m，后续相对增量 0.15m）。
  - **Gazebo 场景**：
    - 路径：`src/rl_sar/worlds/stairs.world`
    - 级高：10级台阶，高度列表 `[0.15, 0.30, 0.45, 0.60, 0.75, 0.90, 1.05, 1.20, 1.35, 1.50]m`（相对增量 0.15m）。

- [2026-06-22] 步骤 1.6：Git 仓库所有权审计与个人 Remote 配置
  - **审计结果**：原始 `origin` 为 `https://github.com/fan-ziqi/rl_sar.git`，不属于用户个人命名空间。
  - **操作记录**：已将原有 `origin` 重命名为 `upstream`，并新增个人 `origin` 指向 `git@github.com:fireowl-sw/rl_sar.git`。暂未推送，等待用户创建远程仓库。
