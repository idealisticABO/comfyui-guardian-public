# ComfyUI Guardian Agent

## 简介
ComfyUI 生产环境守护 Agent，通过 SSH 控制 ComfyUI 服务器。
支持 test→prod 工作流、依赖保护、变更控制、模型注册、自定义节点管理、故障诊断。

**服务器端工具 (`cg` CLI) 与 AI 框架无关**，任何终端都能用。本地 Agent 定义支持多种工具适配。

## 架构

```
你的电脑 (任意 AI 编码工具)            服务器
┌──────────────────────┐         ┌─────────────────────────┐
│  AGENTS.md            │         │  cg CLI                │
│  (规则文档, 适配你的工具) │  SSH    │  ├ preflight / audit   │
│                       │ ──────> │  ├ state / verify      │
│  troubleshooter/      │         │  ├ model / node        │
│  (故障诊断知识库)       │         │  ├ promote / diff      │
└──────────────────────┘         │  └ config.yaml 驱动    │
                                 └─────────────────────────┘
```

## 快速开始

### 1. 部署服务器端工具 (通用, 与 AI 工具无关)
```bash
scp guardian-installer.sh <YOUR_SERVER>:/tmp/
ssh <YOUR_SERVER> "bash /tmp/guardian-installer.sh"
```
安装器会交互式询问所有路径，自动部署 `cg` CLI 和所有脚本。

部署后验证：
```bash
ssh <YOUR_SERVER> "cg config"
ssh <YOUR_SERVER> "cg audit"
```

### 2. 选择你的 AI 编码工具

#### 方式 A: opencode (原生支持)
1. 全局替换 `<YOUR_SERVER_IP>` 等占位符为你的实际值
2. 直接在本目录运行 `opencode`
3. 使用 `/comfy-preflight`、`/comfy-audit` 等命令

#### 方式 B: Claude Code
```bash
# 复制规则文件
cp AGENTS.md CLAUDE.md
# 替换占位符
sed -i 's/<YOUR_SERVER_IP>/你的IP/g' CLAUDE.md
sed -i 's/<TEST_ROOT>/你的test路径/g' CLAUDE.md
# ... 其他占位符

# 启动 Claude Code, 它会自动读取 CLAUDE.md
claude
```

#### 方式 C: Cursor
```bash
# Cursor 读取 .cursorrules
cp AGENTS.md .cursorrules
# 替换占位符后, Cursor 自动读取
```

#### 方式 D: Cline / Roo Code
```bash
# Cline 读取 .clinerules
cp AGENTS.md .clinerules
# 替换占位符
```

#### 方式 E: GitHub Copilot
```bash
mkdir -p .github
cp AGENTS.md .github/copilot-instructions.md
# 替换占位符
```

#### 方式 F: 任意 LLM (ChatGPT / Gemini / 本地模型)
直接把 `AGENTS.md` 内容粘贴到系统提示词中，替换占位符后使用。
通过终端手动执行 SSH 命令即可：
```bash
ssh <YOUR_SERVER> "cg preflight"
ssh <YOUR_SERVER> "cg audit"
ssh <YOUR_SERVER> "cg state test"
```

### 3. 日常使用 (所有工具通用)

无论你用什么 AI 工具，服务器端的 `cg` 命令都是一样的：

| 操作 | 命令 | 说明 |
|------|------|------|
| 预检 | `ssh <SERVER> "cg preflight"` | 7 步检查 test 环境 |
| 审计 | `ssh <SERVER> "cg audit"` | 只读健康检查 |
| 状态 | `ssh <SERVER> "cg state test"` | 采集环境状态 |
| 验证 | `ssh <SERVER> "cg verify test"` | 检查版本锁 |
| 对比 | `ssh <SERVER> "cg diff"` | test vs prod 差异 |
| 变更 | `ssh <SERVER> "cg change \"描述\""` | 生成变更计划 |
| 模型 | `ssh <SERVER> "cg model list"` | 查看模型注册表 |
| 节点 | `ssh <SERVER> "cg node list"` | 查看自定义节点 |
| 升级 | `ssh <SERVER> "cg promote"` | test → prod |

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
├── AGENTS.md                    ← Agent 规则 (所有工具的规则源)
├── opencode.json                ← opencode 专用配置 (仅 opencode 用户需要)
├── guardian-installer.sh        ← 服务器端自解压安装器 (通用)
├── .opencode/                   ← opencode 专用 (仅 opencode 用户需要)
│   ├── agent/
│   │   └── comfyui-guardian.md  ← Agent 定义
│   ├── commands/                ← 8 个 slash 命令
│   └── skills/
│       └── comfyui-troubleshooter/  ← 故障诊断技能
├── troubleshooting/            ← 故障诊断知识库 (通用, 任何 LLM 可读)
│   ├── SKILL.md
│   └── references/
│       ├── models.md
│       └── troubleshooting.md
└── README.md                   ← 本文件
```

> **非 opencode 用户**：只需 `AGENTS.md` + `guardian-installer.sh` + `troubleshooting/`，忽略 `.opencode/` 和 `opencode.json`。

## 已知兼容性问题 (重要)
以下包的版本经过大量验证，升级会导致真实崩溃，建议保持：
- numpy: 1.26.4 — 2.x 会破坏 insightface / numba
- soxr: 0.4.0 — 1.x 版本 nanobind 崩溃
- httpx: 0.28.1
- Python: 3.12.7

`cg verify` 会自动检查这些版本，违反时报警。

## 故障诊断
`troubleshooting/` 目录包含 ComfyUI 故障诊断知识库，任何 LLM 都可以作为上下文读取：
- `SKILL.md` — 诊断流程和决策树
- `references/models.md` — 已知模型问题和解决方案
- `references/troubleshooting.md` — 常见错误模式

使用时把这些文件内容作为上下文提供给 LLM，然后描述你的问题。

## License
Apache-2.0
