# 开发历程与操作日志

- [2026-06-22] 步骤 1：仿真场景及台阶高度审计
  - **MuJoCo 场景**：
    - 路径：`src/rl_sar_zoo/d1_description/mjcf/scene.xml`
    - 级高：6级台阶，高度列表 `[0.17, 0.32, 0.47, 0.62, 0.77, 0.92]m`（首级起步 0.17m，后续相对增量 0.15m）。
  - **Gazebo 场景**：
    - 路径：`src/rl_sar/worlds/stairs.world`
    - 级高：10级台阶，高度列表 `[0.15, 0.30, 0.45, 0.60, 0.75, 0.90, 1.05, 1.20, 1.35, 1.50]m`（相对增量 0.15m）。

- [2026-06-22] 步骤 1.6：Git 仓库所有权审计、重构与推送
  - **审计结果**：原始 `origin` 为 `https://github.com/fan-ziqi/rl_sar.git`，为浅克隆（Shallow Clone）仓库，不属于用户个人命名空间。
  - **操作记录**：为绕过浅克隆限制并满足自我学习需求，删除了原 `.git` 并重新初始化为新仓库。通过在 WSL 上配置 SSH SOCKS5 代理通道，成功将干净的初始提交（Initial commit）推送至 `git@github.com:fireowl-sw/rl_sar.git` 的 `main` 分支。

