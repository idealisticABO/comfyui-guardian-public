---
description: 安装 ComfyUI 自定义节点（GitHub 或本地）
agent: comfyui-guardian
---

为服务器 <YOUR_SERVER> 上的 ComfyUI 测试环境安装自定义节点。

$ARGUMENTS

## 工作流程

### 第一步：获取来源
向用户确认安装来源（如果参数中未提供）：
- **GitHub 链接**：如 `https://github.com/T8mars/Comfyui-zhenzhen.git`
- **本地文件夹路径**：服务器上的节点目录路径

### 第二步：预检与依赖分析
执行预检查命令（只读，不安装）：
```
ssh <YOUR_SERVER_IP> "cg node check --github <URL> --env test"
```
或本地路径：
```
ssh <YOUR_SERVER_IP> "cg node check --local <PATH> --env test"
```

预检会执行以下 5 项分析：
1. 识别节点名称和目录结构
2. 解析 `requirements.txt` 中的显式依赖
3. **扫描所有 .py 文件的 import 语句**，发现未在 requirements.txt 中声明的隐藏依赖
4. 检查与锁定包的冲突（numpy 1.26.4, soxr 0.4.0, httpx 0.28.1, torch, transformers）
5. 检查版本冲突和缺失包

### 第三步：根据预检结果决定操作

**情况 A — 无冲突，无缺失包**：
- 直接向用户确认后安装

**情况 B — 无冲突，有缺失包**：
- 列出需要安装的包，向用户确认后安装

**情况 C — 有冲突或锁定包违规**：
- 详细说明每个冲突
- 给出建议（如：该节点要求 numpy>=2.0，与锁定版本 1.26.4 冲突，建议跳过该依赖或联系节点作者）
- **不自动安装**，等待用户明确指示
- 如果用户确认用 `--force` 强制安装，警告风险后执行

### 第四步：安装
用户确认后执行：
```
ssh <YOUR_SERVER_IP> "cg node install --github <URL> --env test"
```

安装流程：
1. 下载/复制节点到 `custom_nodes/` 目录
2. 安装缺失的 pip 包（跳过锁定包）
3. 重启测试服务 `<TEST_SERVICE>`
4. 等待启动，统计节点数
5. 自动注册到 `registry/nodes_registry.yaml`

### 第五步：验证与汇报
- 执行 `ssh <YOUR_SERVER_IP> "cg verify test"` 确认关键包版本未被破坏
- 执行 `ssh <YOUR_SERVER_IP> "cg state test"` 收集环境状态
- 汇报：节点名称、新增节点数、安装的依赖包、是否有警告
- 执行 `ssh <YOUR_SERVER_IP> "cg regenerate"` 更新 ARCHITECTURE.md

### 第六步：询问是否推广到生产
询问用户是否需要将节点推广到生产环境。如果确认：
1. 执行 `ssh <YOUR_SERVER_IP> "cg node install --github <URL> --env prod"`
2. 或通过 `cg promote` 统一推广

## 其他操作

- **查看已注册节点**：`ssh <YOUR_SERVER_IP> "cg node list"`
- **查看节点详情**：`ssh <YOUR_SERVER_IP> "cg node info <name>"`
- **移除节点**：`ssh <YOUR_SERVER_IP> "cg node remove <name> --env test"`

## 注意事项
- 服务器无法直接访问 github.com，脚本会自动通过 `<GITHUB_MIRROR>` 镜像下载
- 所有安装先在 test 环境进行，验证通过后才推广到 prod
- 锁定包（numpy, soxr, httpx, torch, transformers）不会被自动升级/降级
- 节点信息自动存储到 `registry/nodes_registry.yaml`，包括依赖列表、节点数、来源等

所有输出使用中文。
