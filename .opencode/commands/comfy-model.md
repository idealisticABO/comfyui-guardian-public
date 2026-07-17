---
description: 下载并部署 ComfyUI 模型
agent: comfyui-guardian
---

为服务器 <YOUR_SERVER> 上的 ComfyUI 准备模型部署。

$ARGUMENTS

规则：
1. **先查重：** 执行 `ssh <YOUR_SERVER_IP> "cg model list"` 查看模型是否已注册
2. **检查磁盘空间：** 执行 `ssh <YOUR_SERVER_IP> "df -h /data"`。可用空间 < 300GB 时警告用户，< 100GB 时阻止下载
3. 如果模型有多个变体（fp8/bf16/gguf），下载前询问用户选择
4. 与用户确认模型类型：
   - vae, unet, checkpoint, text_encoder, clip, lora, controlnet, upscale
5. 确定最佳下载源（优先 modelscope，其次 huggingface）
6. 执行：`ssh <YOUR_SERVER_IP> "cg model --source <source> --repo <repo> --pattern <pattern> --model-type <type> --final-name <name>"`
7. 模型存放在服务器 `<MODELS_ROOT>/{type}/` 下
8. 如果模型分片，脚本会自动合并
9. 如果云端提供 SHA256，则验证。如果未提供，则记录本地 SHA256
10. 下载后模型自动注册到 `registry/models_registry.yaml`
11. 汇报最终文件路径、大小、验证结果和注册状态

所有输出使用中文。
