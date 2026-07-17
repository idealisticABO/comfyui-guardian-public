---
description: 只读审计 ComfyUI 环境
agent: comfyui-guardian
---

对服务器 <YOUR_SERVER> 上的 ComfyUI 环境执行只读审计。**不修改任何文件，不重启任何服务。**

步骤：
1. 执行 `ssh <YOUR_SERVER_IP> "cg audit"` 并捕获输出
2. 审查所有检查结果：
   - 测试环境状态（节点数、服务状态）
   - 生产环境状态
   - 两个环境的关键包版本
   - GPU 状态（显存、利用率、温度）
   - /data 磁盘空间
3. 以表格形式汇报两个环境的健康摘要
4. 标记发现的问题（包版本不匹配、磁盘空间不足、GPU 温度过高、服务宕机）
5. **不修改任何文件，不重启任何服务**

所有输出使用中文。

$ARGUMENTS
