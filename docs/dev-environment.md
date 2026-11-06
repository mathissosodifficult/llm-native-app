# 开发环境清单

本文是另一名编码 agent 在新机器上重建本仓库开发环境的事实来源。受众是 agent，不是人类教程。命令按 Debian 系 Linux x86_64 书写。不要编造产品方向，不要添加应用源码。

入口索引是仓库根目录的 [AGENTS.md](../AGENTS.md)。环境（工具、锁定依赖、提示词）有变化时，两份文件在同一次变更里一起改。

## 不要做的事

- 不要打印、记录或提交 API 密钥。禁止 `set -x` 之后再碰 `DEEPSEEK_API_KEY`，禁止 `cat ~/.codex/config.toml` 或 `cat .env`。
- 不要把真实密钥写进仓库、PR、文档或命令参数。命令参数会进入 shell history。
- 不要修改 `requirements.txt` 里的 pin，不要把锁定文件重写成另一套版本。
- 不要安装 JDK、Maven、Gradle 或 Docker。参考机没有这些，它们不在当前环境内。
- 不要另装 DeepSeek SDK，不要把 `wire_api` 改成 `chat`。
- 不要为了消除 doctor 警告而删掉 `preferred_auth_method`。
- 不要编造应用功能、示例程序或产品方向。

## 仓库

- 地址：https://github.com/mathissosodifficult/llm-native-app （私有）。
- 参考机上的克隆路径是 `/home/box/llm-native-app`。新机器用 `gh repo clone` 即可，不必照搬该路径。
- 克隆和推送需要已登录的 GitHub CLI。

## Python

- 版本：`3.13.5`。仓库里的 `.python-version` 内容就是这一行。
- 顶层依赖（其余是传递依赖，全部锁定在 `requirements.txt`）：
  - `openai==3.46.0`
  - `pydantic==2.14.3`
  - `httpx==0.28.1`
  - `python-dotenv==1.2.4`
- 重建：

```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
```

`python3 --version` 必须是 `Python 3.13.5`。不要换成别的 3.13 补丁版本。

## 密钥与应用运行时

- 复制 `.env.example` 为 `.env`，只填写 `DEEPSEEK_API_KEY`。`.env.example` 里该键没有值。
- `.gitignore` 已忽略 `.env` 和 `.venv/`。不要忽略 `requirements.txt` 或 `.python-version`。
- DeepSeek 应用运行时使用 OpenAI 兼容 API：
  - 包：`openai`（不要使用单独的 DeepSeek SDK）
  - `base_url`：`https://api.deepseek.com`
  - 默认模型：`deepseek-flash`
  - 也可使用：`deepseek-v4-pro`
- LLM-native 原则：应用掌握流程和规则，模型是被调用的组件。不要把本仓库做成聊天应用脚手架。

## 参考机上的编码工具

### Codex CLI

- 包：`@openai/codex`
- 版本：`0.167.0`
- 位置：`~/.local/bin/codex`（npm 全局前缀 `~/.local`）

### CC Switch

- 项目：`farion1231/cc-switch`
- 版本：`v4.0.26`
- 文件：`CC-Switch-v4.0.26-Linux-x86_64.AppImage`
- 下载：https://github.com/farion1231/cc-switch/releases/download/v4.0.26/CC-Switch-v4.0.26-Linux-x86_64.AppImage
- 用途：管理 Codex 的供应商切换。
- 自 CC Switch `3.19.1` / `3.20.4` 起，新的 DeepSeek 预设直接通过 Responses API 访问 `https://api.deepseek.com/`，不需要本地反向代理。

Codex 到 DeepSeek 的配置以下面的官方脚本为准。本文不写 CC Switch 的点击步骤。

### DeepSeek 官方 Codex 配置脚本

```bash
bash <(curl -fsSL https://cdn.deepseek.com/api-docs/codex-deepseek-setup-en.sh)
```

- 菜单 `1` = `deepseek-flash`，菜单 `2` = `deepseek-v4-pro`。默认用 `1`。
- 脚本不接受命令行参数。非交互时从 stdin 读菜单选项（`printf '1\n' | bash <(curl ...)`）。
- 密钥来自环境变量 `DEEPSEEK_API_KEY`。脚本不回显密钥。值必须以 `sk-` 开头且不含双引号，否则脚本拒绝写入。
- 脚本要求 `codex` 已在 `PATH` 上，且 `~/.codex` 目录已存在（未运行过 Codex 时目录不存在，脚本会退出）。
- 写入 `~/.codex/config.toml` 和 `~/.codex/models.json`。`models.json` 由脚本生成，不要手写。
- 若已有不是该脚本创建的 `~/.codex/models.json`，且没有 `~/.codex/backup-deepseek`，脚本会拒绝覆盖。停下来，不要擅自删除用户已有的 Codex 配置。

文档撰写时抓取到的脚本自报 `SCRIPT_VERSION=1.4.0`。它写入的必填形态如下。尖括号只是占位，不是密钥：

```toml
model = "deepseek-flash"
model_provider = "deepseek"
preferred_auth_method = "apikey"
forced_login_method = "api"
model_reasoning_effort = "high"
web_search = "disabled"
show_raw_agent_reasoning = true
model_catalog_json = "~/.codex/models.json"

[model_providers.deepseek]
name = "deepseek"
base_url = "https://api.deepseek.com/"
wire_api = "responses"
experimental_bearer_token = "<DEEPSEEK_API_KEY>"
```

同一版脚本还会追加：

```toml
[desktop]
enabled-reasoning-efforts = ["low", "medium", "high", "xhigh", "ultra", "max"]
```

约束：

- `experimental_bearer_token` 取自 `DEEPSEEK_API_KEY`。含该键的 `~/.codex/config.toml` 设为 `chmod 600`。官方脚本本身不改权限，引导步骤里要补上。
- Codex `0.167.0` 会忽略 `preferred_auth_method`，`codex doctor` 因此给出警告。这是预期现象。其余键仍然要保留。
- `wire_api` 必须是 `responses`。`chat` 会被拒绝，Codex 起不来。

### GitHub CLI

- 主机：`github.com`
- 账号：`mathissosodifficult`
- Git 协议：`https`
- scopes：`gist`、`read:org`、`repo`
- 用途：克隆这个私有仓库并推送。

### Node.js

参考机已有 Node.js 24 和 npm，用来安装 Codex。没有记录更细的补丁号。新机器装 Node.js 24 即可。

### 明确不在环境内

JDK、Maven、Gradle、Docker 都没有安装。不要把它们的安装步骤加进引导。

## 常驻指令

与 `AGENTS.md` 保持同一段原文：

```text
本仓库的引导事实来源是 docs/dev-environment.md，入口索引是 AGENTS.md。读完后先安装文档中的工具和依赖，再写应用代码。
不要打印或提交 API 密钥。密钥只通过环境变量 DEEPSEEK_API_KEY 和被 gitignore 的 .env 使用。
优先企业里常见的技术栈。不要引入 awesome-llm-apps 风格的冷门框架。
不要编造产品或项目方向候选。应用方向由用户决定。
一般工作的默认模型是 deepseek-flash。更重的编码或复杂任务，用户也在 www.coze.cn 上用过 Kimi K3；那是 Coze 智能体，不是 code.coze.cn 扣子编程。Coze 只是可选的外部试用工具，不是本仓库依赖。
应用掌握流程和规则，模型是被调用的组件。不要把仓库做成聊天应用脚手架。
环境一旦变化（新工具、新的锁定依赖、新的提示词），在同一次变更中更新 AGENTS.md 和 docs/dev-environment.md。
```

## 一次性引导

在一台新的 Debian 系 Linux x86_64 上按编号执行。每一步的预期都写在命令后面。密钥只允许出现在环境变量和 `chmod 600` 的文件里。

### 1. 确认架构

```bash
uname -s
uname -m
```

预期：`Linux` 与 `x86_64`。不是 `x86_64` 就停止；不要改用 arm64 的 CC Switch 包顶上。

### 2. 安装 gh 并登录

已安装则跳过安装。`gh auth status` 已显示账号 `mathissosodifficult`、协议 `https`、且 scopes 含 `gist`、`read:org`、`repo` 时，跳过登录。

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt-get update
sudo apt-get install -y gh
gh auth login --hostname github.com --git-protocol https --scopes gist,read:org,repo
gh auth status
```

`gh auth login` 是交互式的。不要把 token 放进命令行。预期账号是 `mathissosodifficult`。

### 3. 克隆私有仓库

```bash
gh repo clone mathissosodifficult/llm-native-app
cd llm-native-app
```

参考机路径是 `/home/box/llm-native-app`。克隆到其他目录也可以，之后的命令都在仓库根目录执行。

### 4. 准备 Python 3.13.5 并安装依赖

```bash
python3 --version
```

预期输出：`Python 3.13.5`。

若不是这个版本，先安装这一精确版本，再继续。pyenv 不是参考机记录过的项目依赖，只作为新机器缺少 3.13.5 时的安装手段：

```bash
sudo apt-get update
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev \
  libbz2-dev libreadline-dev libsqlite3-dev curl git \
  libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
curl -fsSL https://pyenv.run | bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
pyenv install 3.13.5
pyenv global 3.13.5
hash -r
python3 --version
```

把上面的 `PYENV_ROOT` 和 `PATH` 写进 `~/.bashrc`，否则新 shell 会回到系统 Python。版本确认是 `Python 3.13.5` 之后：

```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
python -c 'import sys; assert sys.version.startswith("3.13.5"), sys.version'
python -c 'import openai, pydantic, httpx, dotenv; print(openai.__version__); print(pydantic.__version__); print(httpx.__version__)'
```

预期三行：`3.46.0`、`2.14.3`、`0.28.1`。

### 5. 写入 .env，不回显密钥

密钥由用户在环境之外提供。先在当前 shell 里导出 `DEEPSEEK_API_KEY`，不要把它写进仓库命令或回显。

```bash
cp .env.example .env
umask 077
python3 - << 'PY'
import os, pathlib, sys
key = os.environ.get("DEEPSEEK_API_KEY", "")
if not key or '"' in key:
    sys.exit("DEEPSEEK_API_KEY unset or contains a double quote; value not printed")
path = pathlib.Path(".env")
path.write_text(
    "# DeepSeek uses the OpenAI-compatible API at https://api.deepseek.com\n"
    "DEEPSEEK_API_KEY=" + key + "\n"
)
path.chmod(0o600)
print("wrote .env key_set=true mode=%o" % (path.stat().st_mode & 0o777))
PY
git check-ignore -v .env
```

预期：`git check-ignore` 命中 `.gitignore` 的 `.env` 规则，且输出里没有密钥。

随后让当前 shell 从文件加载，仍然不打印：

```bash
set -a
. ./.env
set +a
```

### 6. 安装 Node.js 24

`node -v` 已是 `v24` 开头则跳过。

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v
```

预期：`node -v` 以 `v24.` 开头。

### 7. 安装 Codex CLI 0.167.0

```bash
mkdir -p "$HOME/.local"
npm install -g --prefix "$HOME/.local" @openai/codex@0.167.0
export PATH="$HOME/.local/bin:$PATH"
command -v codex
codex --version
```

预期：`command -v` 打印 `$HOME/.local/bin/codex`，版本输出含 `0.167.0`。把 `$HOME/.local/bin` 放进 `~/.bashrc` 的 `PATH`。

### 8. 安装 CC Switch v4.0.26

```bash
install -d "$HOME/.local/bin"
curl -fL --retry 3 -o "$HOME/.local/bin/CC-Switch-v4.0.26-Linux-x86_64.AppImage" \
  "https://github.com/farion1231/cc-switch/releases/download/v4.0.26/CC-Switch-v4.0.26-Linux-x86_64.AppImage"
chmod +x "$HOME/.local/bin/CC-Switch-v4.0.26-Linux-x86_64.AppImage"
```

预期：该 AppImage 路径存在且可执行。直接运行若报 FUSE 错误，再安装 `libfuse2`。`libfuse2` 不是项目依赖。

### 9. 配置 Codex 使用 DeepSeek

在仓库根目录、密钥已导出、`codex` 在 `PATH` 上时执行。菜单选择 `1`（`deepseek-flash`）。

```bash
mkdir -p "$HOME/.codex"
python3 - << 'PY'
import os, sys
key = os.environ.get("DEEPSEEK_API_KEY", "")
if (not key.startswith("sk-")) or ('"' in key):
    sys.exit("DEEPSEEK_API_KEY rejected by the official setup script; value not printed")
print("prefix_ok=true")
PY
printf '1\n' | bash <(curl -fsSL https://cdn.deepseek.com/api-docs/codex-deepseek-setup-en.sh)
chmod 600 "$HOME/.codex/config.toml"
```

预期：脚本报告使用了 `DEEPSEEK_API_KEY`，并写入 `model = "deepseek-flash"`。终端输出中不应出现密钥本身。

### 10. 脱敏检查与 codex doctor

不要打印 token。只确认键存在。

```bash
python3 - << 'PY'
import stat, tomllib
from pathlib import Path
cfg_path = Path.home() / ".codex" / "config.toml"
models_path = Path.home() / ".codex" / "models.json"
mode = stat.S_IMODE(cfg_path.stat().st_mode)
with cfg_path.open("rb") as fh:
    cfg = tomllib.load(fh)
prov = cfg["model_providers"]["deepseek"]
token = prov.get("experimental_bearer_token") or ""
checks = {
    "model": cfg.get("model") == "deepseek-flash",
    "model_provider": cfg.get("model_provider") == "deepseek",
    "preferred_auth_method": cfg.get("preferred_auth_method") == "apikey",
    "forced_login_method": cfg.get("forced_login_method") == "api",
    "model_reasoning_effort": cfg.get("model_reasoning_effort") == "high",
    "web_search": cfg.get("web_search") == "disabled",
    "show_raw_agent_reasoning": cfg.get("show_raw_agent_reasoning") is True,
    "model_catalog_json": cfg.get("model_catalog_json") == "~/.codex/models.json",
    "provider_name": prov.get("name") == "deepseek",
    "base_url": prov.get("base_url") == "https://api.deepseek.com/",
    "wire_api": prov.get("wire_api") == "responses",
    "token_set": bool(token),
    "config_mode_600": mode == 0o600,
    "models_json": models_path.is_file(),
}
for name, ok in checks.items():
    print(f"{name}={'ok' if ok else 'MISSING'}")
if not all(checks.values()):
    raise SystemExit(1)
PY
codex doctor
```

预期：

- 上面每一项都是 `ok`，且没有任何一项的取值被打印成密钥。
- `codex doctor` 识别到当前 git 仓库，模型为 `deepseek-flash`。
- 关于忽略 `preferred_auth_method` 的警告可以出现，不算失败。
- `wire_api` 不是 `responses`，或模型不是 `deepseek-flash`，则引导失败。不要改成 `chat`。

## 变更纪律

新增工具、修改锁定依赖或修改常驻指令时，在同一次提交中更新本文和 `AGENTS.md`。不要只改其中一份。不要把密钥样例写进更新。
