# ComfyUI Guardian Agent

## 简介
ComfyUI 生产环境守护 Agent，在本地运行，通过 SSH 控制 ComfyUI 服务器。
支持 test→prod 工作流、依赖保护、变更控制、模型注册、自定义节点管理、故障诊断。

## 快速开始

### 1. 配置 Agent
1. 修改 `opencode.json` 中的 `<YOUR_SERVER_IP>` 为你的服务器 IP
2. 修改 `AGENTS.md` 中所有 `<...>` 占位符为你的实际值
3. 修改 `.opencode/agent/comfyui-guardian.md` 中的占位符
4. 修改 `.opencode/commands/*.md` 中的占位符

### 2. 部署工具到服务器
```bash
scp guardian-installer.sh <YOUR_SERVER>:/tmp/
ssh <YOUR_SERVER> "bash /tmp/guardian-installer.sh"
```
安装器会交互式询问所有路径，自动部署 cg CLI 和所有脚本。

### 3. 运行
```bash
opencode
```
然后使用 `/comfy-preflight`、`/comfy-audit` 等命令。

## 占位符对照表

| 占位符 | 含义 | 示例 |
|--------|------|------|
| `<YOUR_SERVER_IP>` | 服务器 IP | 192.168.1.100 |
| `<YOUR_SERVER>` | 服务器主机名 | myserver |
| `<TEST_ROOT>` | ComfyUI test 根目录 | /opt/ComfyUI_test |
| `<PROD_ROOT>` | ComfyUI prod 根目录 | /opt/ComfyUI_prod |
| `<LEGACY_ROOT>` | Legacy 环境根目录 | /opt/ComfyUI |
| `<TEST_CONDA_ENV>` | test conda 环境路径 | /opt/conda/envs/comfyui_test |
| `<PROD_CONDA_ENV>` | prod conda 环境路径 | /opt/conda/envs/comfyui_prod |
| `<TEST_SERVICE>` | test systemd 服务名 | comfyui-test |
| `<PROD_SERVICE>` | prod systemd 服务名 | comfyui-prod |
| `<TEST_PORT>` | test 端口 | 8188 |
| `<PROD_PORT>` | prod 端口 | 8190 |
| `<MODELS_ROOT>` | 模型共享根目录 | /opt/ComfyUI/models |
| `<GUARDIAN_ROOT>` | Guardian 工具根目录 | /opt/comfyui-tools/guardian |
| `<GITHUB_MIRROR>` | GitHub 镜像 | mirror.example.com |

## 目录结构
```
.
├── AGENTS.md                    ← Agent 规则 (替换占位符后使用)
├── opencode.json                ← opencode 配置 (替换占位符后使用)
├── guardian-installer.sh        ← 服务器端自解压安装器
├── .opencode/
│   ├── agent/
│   │   └── comfyui-guardian.md  ← Agent 定义
│   ├── commands/                ← 8 个中文命令
│   │   ├── comfy-audit.md
│   │   ├── comfy-change.md
│   │   ├── comfy-diff.md
│   │   ├── comfy-model.md
│   │   ├── comfy-node.md
│   │   ├── comfy-preflight.md
│   │   ├── comfy-promote.md
│   │   └── comfy-snapshot.md
│   └── skills/
│       └── comfyui-troubleshooter/  ← 故障诊断技能
│           ├── SKILL.md
│           └── references/
│               ├── models.md
│               └── troubleshooting.md
└── README.md                   ← 本文件
```

## 已知兼容性问题 (重要)
以下包的版本经过大量验证，升级会导致真实崩溃，建议保持：
- numpy: 1.26.4 — 2.x 会破坏 insightface / numba
- soxr: 0.4.0 — 1.x 版本 nanobind 崩溃
- httpx: 0.28.1
- Python: 3.12.7

`cg verify` 会自动检查这些版本，违反时报警。

## License
Apache-2.0
