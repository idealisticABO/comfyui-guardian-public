---
description: 修改测试环境前生成变更计划
agent: comfyui-guardian
---

为即将进行的 ComfyUI 测试环境修改生成变更计划。

$ARGUMENTS

步骤：
1. 执行 `ssh <YOUR_SERVER_IP> "cg change \"$ARGUMENTS\""` 创建 CHG-xxx.yaml 文件
2. 读取生成的文件：`ssh <YOUR_SERVER_IP> "cat <GUARDIAN_ROOT>/changes/CHG-*.yaml | tail -20"`
3. 填写详细信息：
   - type: custom_node_install | package_install | model_deploy | comfyui_update | config_change
   - reason: 本次变更的原因
   - risk: low | medium | high
   - files_changed: 将修改哪些文件
   - rollback: 如何回滚
   - validation: 变更后需运行哪些验证
4. 将变更计划提交给用户审批
5. 在用户确认前**不得**执行任何修改

变更计划是安全审计记录。对测试环境的每次修改都必须有对应的 CHG 文件。

所有输出使用中文。
