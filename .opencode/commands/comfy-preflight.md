---
description: 运行 ComfyUI 测试环境预检
agent: comfyui-guardian
---

对服务器 <YOUR_SERVER> 上的 ComfyUI 测试环境执行预检。

步骤：
1. 执行 `ssh <YOUR_SERVER_IP> "cg preflight"` 并捕获输出
2. 审查全部 7 项检查结果：
   - 架构备份状态
   - 未关闭的变更计划（列出 pending/in-progress 的 CHG 文件）
   - 测试状态收集（current_test.json）
   - 关键包版本（numpy, soxr, httpx, Python）
   - 服务状态（<TEST_SERVICE>）
   - 节点数
   - GPU 状态（显存、利用率、温度）
3. 如果任何检查失败，停止并清楚解释失败原因
4. 以汇总表格汇报所有检查结果
5. 如果有未关闭的变更计划，提醒用户在变更完成后关闭
6. **不修改生产环境**

所有输出使用中文。
