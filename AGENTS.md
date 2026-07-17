# ComfyUI Guardian Rules

This project controls ComfyUI maintenance on remote server <YOUR_SERVER> (<YOUR_SERVER_IP>) via SSH.

## Architecture

- **Agent runs locally** on this Windows machine
- **All server operations** are performed via `ssh <YOUR_SERVER_IP> "command"`
- **Shell scripts** (cg) are already deployed on the server at `<GUARDIAN_ROOT>/bin/`
- **SSH config** is set up with key-based auth (no password needed)

## Absolute Rules

1. **Never modify production first.** All changes start in test.
2. **Always backup before changes.** Run `ssh <YOUR_SERVER_IP> "cg snapshot"` before any modification.
3. **After changes, collect state.** Run `ssh <YOUR_SERVER_IP> "cg state test"` or `"cg state prod"`.
4. **Never upgrade numpy to 2.x.** numpy must remain 1.26.4.
5. **Never install soxr 1.x.** soxr must remain 0.4.0.
6. **Never downgrade httpx.** httpx must remain 0.28.1.
7. **Python must remain 3.12.7** in new environments.
8. **Never randomly place models.** Use the shared model root: `<MODELS_ROOT>/`.
9. **Model type determines directory:**
   - VAE: `models/vae/`
   - UNet: `models/unet/`
   - checkpoint: `models/checkpoints/`
   - text encoder: `models/text_encoders/`
   - LoRA: `models/loras/{base_model}/` (see LoRA sub-directory rule below)
   - ControlNet: `models/controlnet/`
10. **LoRA sub-directory by base model.** All LoRA files MUST be placed in a sub-directory under `models/loras/` named after the base model they adapt. Known base model types:
    - `xl` — SDXL base models
    - `flux1` — Flux.1 models
    - `flux_krea` — Krea-2 models (Flux-based)
    - `qwen_image_2511` — Qwen Image Edit 2511
    - `zimage` — Z-Image / z_image_turbo models
    - Other names as needed (ask user if uncertain)
    - Agent should auto-detect base model from context (e.g., `Comfy-Org/z_image_turbo` repo → `zimage`), or ask user to confirm.
    - If the sub-directory does not exist, create it: `ssh <YOUR_SERVER_IP> "mkdir -p <MODELS_ROOT>/loras/{base_model}"`
11. **LoRA metadata recording.** After placing a LoRA file, ask the user if they have basic info about the model. If yes, record it in `<GUARDIAN_ROOT>/registry/lora_metadata.json` (JSON format, AI-readable). Each entry should include:
    - `name`: LoRA name
    - `filename`: actual file name
    - `path`: relative path from `models/loras/`
    - `base_model`: base model type (xl/flux1/flux_krea/qwen_image_2511/zimage/...)
    - `trigger_words`: trigger words for prompt (string or list)
    - `recommended_weight`: recommended weight range (e.g., "0.7-0.9")
    - `sampler`: recommended sampler (e.g., "res_multistep")
    - `scheduler`: recommended scheduler (e.g., "simple")
    - `resolution`: recommended resolution (e.g., "1024x1024, 1024x1536")
    - `prompt_format`: prompt convention description
    - `author`: model author
    - `license`: license info
    - `notes`: any additional notes
12. **Ask user to choose variant** when a model has multiple versions (fp8/bf16/gguf).
13. **Merge shards** after download if a model is split.
14. **Verify SHA256** when cloud provides one.
15. **Always create a change plan before modifying test.** Run `ssh <YOUR_SERVER_IP> "cg change"` to generate a CHG-xxx.yaml file. No modification without an active change plan.
16. **ARCHITECTURE.md is auto-generated.** Never manually edit it. Run `ssh <YOUR_SERVER_IP> "cg regenerate"` to regenerate from `current_*.json`.
17. **Check disk space before downloading models.** Run `ssh <YOUR_SERVER_IP> "df -h /data"`. If free space < 300GB, warn user before proceeding.
18. **Register every model after download.** The `cg model` command auto-registers to `registry/models_registry.yaml`. Check `ssh <YOUR_SERVER_IP> "cg model list"` before downloading duplicates.
19. **Never delete models directly.** Use `ssh <YOUR_SERVER_IP> "cg model archive <name>"` to archive, or `"cg model remove <name>"` (requires confirmation).

## Environment Paths (on server)

### Test
- Root: `<TEST_ROOT>`
- Conda: `<TEST_CONDA_ENV>`
- Service: `<TEST_SERVICE>`
- Port: `<TEST_PORT>`

### Production
- Root: `<PROD_ROOT>`
- Conda: `<PROD_CONDA_ENV>`
- Service: `<PROD_SERVICE>`
- Port: `<PROD_PORT>`

### Legacy Production (DO NOT MODIFY)
- Root: `<LEGACY_ROOT>`

## Required Workflow

1. Run: `ssh <YOUR_SERVER_IP> "cg preflight"`
2. **Create change plan:** `ssh <YOUR_SERVER_IP> "cg change"` (generates CHG-xxx.yaml with details)
3. Modify test only (via SSH)
4. Restart test: `ssh <YOUR_SERVER_IP> "systemctl restart <TEST_SERVICE>"`
5. Validate: `ssh <YOUR_SERVER_IP> "cg verify test"` and `ssh <YOUR_SERVER_IP> "cg state test"`
6. Optionally run workflow validation: `ssh <YOUR_SERVER_IP> "cg validate-workflow"`
7. **Auto-regenerate ARCHITECTURE.md:** `ssh <YOUR_SERVER_IP> "cg regenerate"`
8. Ask user for approval before production promotion
9. Promote: `ssh <YOUR_SERVER_IP> "cg promote"`
10. Validate production: `ssh <YOUR_SERVER_IP> "cg verify prod"` and `ssh <YOUR_SERVER_IP> "cg state prod"`

## State Management

- **Current state JSON:** `state/current_test.json` and `state/current_prod.json` — machine-readable, always up-to-date
- **State history:** `state/history/YYYYMMDD.json` — archived after each `cg state` run
- **Environment diff:** `ssh <YOUR_SERVER_IP> "cg diff"` — compares test vs prod (nodes, packages, commits)
- **ARCHITECTURE.md:** Auto-generated from `current_*.json` by `cg regenerate`. Human-readable summary. Never manually edit.

## Policy Engine

Forbidden actions are defined in `policy/forbidden_actions.yaml` on the server. The `cg` CLI checks this file before executing any command. Key prohibitions:
- `pip install numpy/soxr/httpx` — blocked (version lock)
- `pip install -r requirements.txt` in prod — blocked
- `git pull` in prod — requires confirmation
- `rm` — requires confirmation
- Model deletion — requires confirmation, use `cg model archive/remove` instead

## Model Management

- Shared model root: `<MODELS_ROOT>/`
- Registry: `registry/models_registry.yaml` — tracks all deployed models with path, size, SHA256, source
- Lifecycle: download -> verify -> activate -> deprecate -> archive -> remove
- Always check `cg model list` before downloading to avoid duplicates
- Disk space thresholds: warn at <300GB free, critical at <100GB free

## Custom Node Management

- **Node registry:** `registry/nodes_registry.yaml` — tracks all installed custom nodes with dependencies, source, node count
- **Always pre-check before install:** Run `cg node check` to analyze dependencies and detect conflicts
- **Dependency analysis includes:**
  - Parsing `requirements.txt` (explicit dependencies)
  - Scanning all `.py` files for `import` statements (hidden dependencies not declared in requirements.txt)
  - Checking for conflicts with locked packages (numpy 1.26.4, soxr 0.4.0, httpx 0.28.1)
  - Checking for version conflicts with currently installed packages
- **Install flow:** check → user confirm → install deps → copy to custom_nodes → restart → register
- **Locked packages are never auto-upgraded:** numpy, soxr, httpx, torch, transformers
- **GitHub access:** Server cannot reach github.com directly. Node manager uses `<GITHUB_MIRROR>` mirror automatically.
- **Node info storage:** Each installed node is registered with: path, source, repo URL, dependencies, requirements_txt, hidden_dependencies, node_count, timestamp
- **Always install to test first**, validate, then promote to production

## Troubleshooter Skill

When preflight finds errors, service logs show issues, or the user reports ComfyUI problems, load the `comfyui-troubleshooter` skill for diagnostic knowledge. Use `cg state` / `cg diff` output as the equivalent of `state/inventory.json` referenced by the skill.

## Audit Mode

For read-only inspections without any risk of modification:
```
ssh <YOUR_SERVER_IP> "cg audit"
```
This collects state, checks packages, verifies GPU, and outputs a health score. No files are modified.

## SSH Command Patterns

All server commands use this format:
```
ssh <YOUR_SERVER_IP> "command here"
```

For commands with quotes, use single quotes inside:
```
ssh <YOUR_SERVER_IP> "cd <TEST_ROOT>/custom_nodes && git clone https://github.com/xxx/ComfyUI-xxx-node"
```

## cg CLI Reference

| Command | Description |
|---------|-------------|
| `cg preflight` | Run test environment preflight check |
| `cg snapshot` | Backup ARCHITECTURE.md |
| `cg state test\|prod` | Collect environment state (outputs current_{env}.json) |
| `cg verify test\|prod` | Verify critical package versions |
| `cg diff` | Compare test vs prod state |
| `cg change` | Generate a change plan (CHG-xxx.yaml) |
| `cg audit` | Read-only health check with scoring |
| `cg regenerate` | Auto-generate ARCHITECTURE.md from current_*.json |
| `cg validate-workflow` | Run test workflow via ComfyUI API |
| `cg model --source ... --repo ... --pattern ... --model-type ... --final-name ...` | Download model |
| `cg model list` | List registered models |
| `cg model archive <name>` | Archive a model |
| `cg model remove <name>` | Remove a model (requires confirmation) |
| `cg node check --github <URL> [--env test\|prod]` | Pre-check node dependencies and conflicts |
| `cg node install --github <URL> [--env test\|prod]` | Install custom node from GitHub |
| `cg node install --local <PATH> [--env test\|prod]` | Install custom node from local directory |
| `cg node list [--env test\|prod]` | List registered custom nodes |
| `cg node info <name>` | Show node details (deps, source, node count) |
| `cg node remove <name> [--env test\|prod]` | Remove a custom node |
| `cg promote` | Promote test environment to production |
