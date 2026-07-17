---
description: 备份 ARCHITECTURE.md 并收集环境状态
agent: comfyui-guardian
---

为服务器 <YOUR_SERVER> 上的两个环境创建架构备份并收集当前状态。

步骤：
1. 执行 `ssh <YOUR_SERVER_IP> "cg snapshot"` 备份 ARCHITECTURE.md
2. 执行 `ssh <YOUR_SERVER_IP> "cg state test"` 收集测试环境状态（生成 current_test.json + 归档到 history/）
3. 执行 `ssh <YOUR_SERVER_IP> "cg state prod"` 收集生产环境状态（生成 current_prod.json + 归档到 history/）
4. 执行 `ssh <YOUR_SERVER_IP> "cg regenerate"` 从 JSON 状态文件自动生成 ARCHITECTURE.md
5. 汇总：
   - 备份文件路径
   - 测试和生产的最新状态（节点数、服务状态、commit）
   - /data 磁盘空间
6. 汇报测试与生产之间的差异（custom_nodes 差异、commit 差异、包差异）

所有输出使用中文。
