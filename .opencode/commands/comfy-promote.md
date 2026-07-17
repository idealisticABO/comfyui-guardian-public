---
description: 将验证通过的测试变更推广到生产
agent: comfyui-guardian
---

将服务器 <YOUR_SERVER> 上的 ComfyUI 测试环境变更推广到生产。

$ARGUMENTS

执行前：
1. 确认测试已通过验证：执行 `ssh <YOUR_SERVER_IP> "cg verify test"`
2. 执行 `ssh <YOUR_SERVER_IP> "cg diff"` 展示测试与生产差异
3. **必须**请求用户明确批准生产推广
4. 汇总将要发生的变更：
   - ComfyUI commit 差异（测试 vs 生产）
   - 仅在测试中的 custom_nodes 目录（将添加到生产）
   - 仅在生产中的 custom_nodes 目录（将从生产移除）
   - 注意：pip 包不会自动同步

执行后：
1. 执行 `ssh <YOUR_SERVER_IP> "cg promote"`
2. 执行 `ssh <YOUR_SERVER_IP> "cg verify prod"`
3. 执行 `ssh <YOUR_SERVER_IP> "cg state prod"`
4. 执行 `ssh <YOUR_SERVER_IP> "cg regenerate"` 自动生成更新后的 ARCHITECTURE.md
5. 汇报生产节点数和服务状态
6. 如果 pip 包有差异，列出并提供手动安装命令建议
7. 关闭关联的变更计划（更新 status 为 completed）

**未经用户明确批准，不得执行推广。**

所有输出使用中文。
