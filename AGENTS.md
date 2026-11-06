# Agent 入口

重建本仓库开发环境时，以 [docs/dev-environment.md](docs/dev-environment.md) 为事实来源。先读完该清单并执行其中的一次性引导，安装工具和依赖之后，再写应用代码。不要向用户追问文档里已经写明的版本号。

本文是短索引。版本、安装命令、Codex 配置键和引导步骤以 `docs/dev-environment.md` 为准。两份文件在同一次环境变更里一起改。

## 已成立的事实

- 私有仓库：https://github.com/mathissosodifficult/llm-native-app 。参考机克隆路径是 `/home/box/llm-native-app`。
- Python `3.13.5`（`.python-version`）。顶层依赖：`openai==3.46.0`、`pydantic==2.14.3`、`httpx==0.28.1`、`python-dotenv==1.2.4`。完整锁定文件是 `requirements.txt`，不要改这些 pin。
- 重建虚拟环境：`python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`
- 密钥只放 `.env` 的 `DEEPSEEK_API_KEY`。由 `.env.example` 复制而来。不要提交 `.env` 或密钥。
- 应用运行时用 `openai` Python 包调用 DeepSeek 的 OpenAI 兼容接口，`base_url` 为 `https://api.deepseek.com`。默认模型 `deepseek-flash`，也可使用 `deepseek-v4-pro`。不要另装 DeepSeek SDK。
- 本仓库的 LLM-native 原则：应用掌握流程和规则，模型是被调用的组件。不要把它做成聊天应用脚手架，也不要编造产品方向。

## 参考机工具（详情与安装命令见长文）

| 工具 | 版本 / 状态 |
| --- | --- |
| Codex CLI `@openai/codex` | `0.167.0`，安装到 `~/.local/bin/codex` |
| CC Switch `farion1231/cc-switch` | `v4.0.26`，`CC-Switch-v4.0.26-Linux-x86_64.AppImage` |
| GitHub CLI | 已登录 `github.com`，账号 `mathissosodifficult`，协议 `https`，scopes `gist` / `read:org` / `repo` |
| Node.js / npm | 参考机为 Node.js 24，用于安装 Codex |
| JDK、Maven、Gradle、Docker | 未安装，超出范围，不要添加 |

## 常驻指令

把下面这段当作在本仓库工作时的常驻指令：

```text
本仓库的引导事实来源是 docs/dev-environment.md，入口索引是 AGENTS.md。读完后先安装文档中的工具和依赖，再写应用代码。
不要打印或提交 API 密钥。密钥只通过环境变量 DEEPSEEK_API_KEY 和被 gitignore 的 .env 使用。
优先企业里常见的技术栈。不要引入 awesome-llm-apps 风格的冷门框架。
不要编造产品或项目方向候选。应用方向由用户决定。
一般工作的默认模型是 deepseek-flash。更重的编码或复杂任务，用户也在 www.coze.cn 上用过 Kimi K3；那是 Coze 智能体，不是 code.coze.cn 扣子编程。Coze 只是可选的外部试用工具，不是本仓库依赖。
应用掌握流程和规则，模型是被调用的组件。不要把仓库做成聊天应用脚手架。
环境一旦变化（新工具、新的锁定依赖、新的提示词），在同一次变更中更新 AGENTS.md 和 docs/dev-environment.md。
```
