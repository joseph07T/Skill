---
name: freellmapi
description: Use when the user asks about FreeLLMAPI, free LLM providers or endpoints, or wants to point an OpenAI-compatible client at a self-hosted free-tier router.
---

# FreeLLMAPI

FreeLLMAPI (github.com/tashfeenahmed/freellmapi, MIT) is a self-hosted router that combines the free tiers of many LLM providers behind one OpenAI-compatible `/v1` endpoint. It stores provider keys encrypted, picks the best available model per request, falls over to the next provider when one is rate-limited, and tracks per-key usage against free-tier caps.

## When to use
- The user asks about free LLM APIs, free model endpoints, or alternative providers.
- The user wants one endpoint that rotates across free tiers.
- The user wants to connect a coding agent or OpenAI SDK to FreeLLMAPI.

## Install
- Docker one-liner: `curl -fsSL https://freellmapi.co/install.sh | bash` (creates `~/freellmapi`, generates an encryption key, starts the container). Tell the user they can read the script first and that re-running is safe.
- Windows: desktop app from the Microsoft Store. Other options: Docker Compose, local dev, Android via Termux (see docs/en/install/ in the repo).
- Dashboard: http://localhost:3001. Add provider keys on the **Keys** page, reorder the **Fallback Chain**, and copy the **unified API key** from the Keys page header.

## Using it
- Base URL: `http://localhost:3001/v1`; API key: the unified key (`freellmapi-...`).
- Model `auto` lets the router choose. Also `auto:fast`, `auto:smart`, a profile name, or a specific model id.
- Every response has an `X-Routed-Via: <platform>/<model>` header showing which provider served it.

```python
from openai import OpenAI
client = OpenAI(base_url="http://localhost:3001/v1", api_key="<unified-key>")
resp = client.chat.completions.create(
    model="auto",
    messages=[{"role": "user", "content": "Hello"}],
)
print(resp.choices[0].message.content)
```

It also supports streaming, tool calling, vision input, embeddings, and an Anthropic Messages surface; details are in docs/en/api/01-rest-api.md in the repo.

## Configuration notes
- `ENCRYPTION_KEY` (required in production) encrypts stored keys; `PORT` defaults to 3001.
- Docker publishes only on 127.0.0.1 by default. Opening it to the LAN (`HOST_BIND`) should only be done on a trusted network, because the proxy is single-user and protected only by the unified key.

## Rules
- Never ask for, print, or commit real provider keys or the unified key; use placeholders.
- Free tiers have rate limits and may change; don't promise uptime or quotas.
- For current provider and model lists, point to the repo README or freellmapi.co rather than quoting numbers from memory.
