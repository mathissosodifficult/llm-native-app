# llm-native-app
LLM 原生应用学习项目

Agent 先读 [AGENTS.md](AGENTS.md)。重建开发环境的完整清单和命令在 [docs/dev-environment.md](docs/dev-environment.md)。

## Environment

Python 3.13.5 (see `.python-version`).

```bash
python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
```

Copy `.env.example` to `.env` and fill `DEEPSEEK_API_KEY`.

Top-level dependencies: openai, pydantic, httpx, python-dotenv. `requirements.txt` is a full freeze, including transitive packages.
