---
description: 对比测试与生产 ComfyUI 环境
agent: comfyui-guardian
---

对比服务器 <YOUR_SERVER> 上的 ComfyUI 测试和生产环境。

步骤：
1. 执行 `ssh <YOUR_SERVER_IP> "cg state test"` 收集当前测试环境状态
2. 执行 `ssh <YOUR_SERVER_IP> "cg state prod"` 收集当前生产环境状态
3. 执行 `ssh <YOUR_SERVER_IP> "cg diff"` 进行对比
4. 以清晰表格汇报差异：
   - ComfyUI commit（测试 vs 生产）
   - 节点数差异
   - 仅在测试中的 custom_nodes（将会添加到生产）
   - 仅在生产中的 custom_nodes（将会从生产移除）
   - 包版本差异
   - 两个环境的磁盘空间
5. 如果存在显著差异，说明是否需要通过 `cg promote` 同步

所有输出使用中文。
