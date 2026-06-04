# ainfera-microsoft-agent-framework

**Microsoft Agent Framework + Ainfera Routing.** Two env vars.

[Microsoft Agent Framework](https://github.com/microsoft/agent-framework) is the 2026 merged successor to AutoGen + Semantic Kernel. This package points MAF's chat client at [Ainfera Routing](https://ainfera.ai) — gateway-routed inference, signed audit, hash-chained receipts, `routing_outcomes` captured per call.

## Install

```bash
pip install ainfera-microsoft-agent-framework
```

## Quickstart (2 env vars)

```bash
export AINFERA_API_KEY=ainfera_...  # https://app.ainfera.ai/signup
# AINFERA_API_URL defaults to https://api.ainfera.ai/v1 (override for staging)
```

```python
import asyncio
from agent_framework import ChatAgent
from ainfera_maf import ainfera_chat_client

async def main() -> None:
    agent = ChatAgent(
        chat_client=ainfera_chat_client(model="ainfera-inference"),
        instructions="You are a helpful assistant. Reply briefly.",
    )
    result = await agent.run("Say hello in one sentence.")
    print(result)

asyncio.run(main())
```

That's the whole story. The agent's call routes through `api.ainfera.ai/v1` → Ainfera's routing engine picks the right backend → response comes back with a signed receipt + a `routing_outcomes` row in the audit chain.

## Model choice

- **`ainfera-inference`** (default) — exercises the routing engine. The engine picks the right backend per request based on the constrained-objective decision (quality / cost / latency). Recommended for production.
- **`claude-opus-4-7` / `gpt-5-5` / `gemini-3-1-pro` / `grok-4` / `mistral-large-3`** — pin a specific model. Bypasses the routing decision; useful for benchmarking. See `GET https://api.ainfera.ai/v1/models` for the live catalog.

## Tool-use + streaming

Both pass through. MAF's `ChatAgent` supports tool calling + streaming on `OpenAIChatClient`; the Ainfera gateway forwards both to the routed backend (`/v1/messages` carries SSE; tools are translated per the Anthropic / OpenAI dialect).

## Why route through Ainfera?

- **Signed audit per turn.** Every routed call lands a hash-chained `audit_event` + a §16 `routing_outcomes` row. Verifiable.
- **One billing surface, many providers.** Pre-paid wallet model; per-call cost-plus margin; provider routing is invisible to the agent.
- **Routing-decision moat data.** Each call adds to `q_empirical` — the proprietary signal that makes the routing decisions better over time.

## Configuration

| Env var | Default | Required |
|---|---|---|
| `AINFERA_API_KEY` | (none) | **Yes** — must start with `ainfera_` |
| `AINFERA_API_URL` | `https://api.ainfera.ai/v1` | No |

You can also pass explicit `api_key=` / `base_url=` / `model=` arguments to `ainfera_chat_client(...)`. Explicit args win over env vars.

## What this adapter is NOT

- Not a bypass — every call goes through the gateway (one-router invariant).
- Not a payment system — billing lives on the Ainfera side (the SDK doesn't see card/bank data).
- Not Azure-specific — this is for the public-OpenAI surface of MAF. Use MAF's native `AzureOpenAIChatClient` for Azure-direct.

## Family

This is one of the Ainfera framework adapters:

- [ainfera-openai-compatible](https://github.com/ainfera-ai/ainfera-openai-compatible) — bare OpenAI SDK
- [ainfera-langchain](https://github.com/ainfera-ai/ainfera-langchain) — LangChain
- [ainfera-langgraph](https://github.com/ainfera-ai/ainfera-langgraph) — LangGraph
- [ainfera-crewai](https://github.com/ainfera-ai/ainfera-crewai) — CrewAI
- [ainfera-llamaindex](https://github.com/ainfera-ai/ainfera-llamaindex) — LlamaIndex
- [ainfera-letta](https://github.com/ainfera-ai/ainfera-letta) — Letta (MemGPT)
- [ainfera-google-adk](https://github.com/ainfera-ai/ainfera-google-adk) — Google ADK
- [ainfera-hermes](https://github.com/ainfera-ai/ainfera-hermes) — Hermes
- [ainfera-mcp](https://github.com/ainfera-ai/ainfera-mcp) — MCP reference
- [ainfera-openclaw](https://github.com/ainfera-ai/ainfera-openclaw) — OpenClaw
- **ainfera-microsoft-agent-framework** (this repo) — Microsoft Agent Framework
- [ainfera-pydantic](https://github.com/ainfera-ai/ainfera-pydantic) — Pydantic AI

All share the same `AINFERA_API_KEY` + `AINFERA_API_URL` contract.

## License

Apache 2.0.
