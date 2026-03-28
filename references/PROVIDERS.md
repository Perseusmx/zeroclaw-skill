# ZeroClaw Providers Reference

This reference was refreshed against the official ZeroClaw provider docs on **March 28, 2026**.

## Quick Commands

```bash
zeroclaw providers
zeroclaw models refresh
zeroclaw models refresh --provider <provider-id>
zeroclaw providers-quota
```

## Credential Resolution Order

Runtime credential lookup follows this order:
1. Explicit credential from config or CLI
2. Provider-specific environment variable
3. `ZEROCLAW_API_KEY`
4. `API_KEY`

Fallback providers resolve credentials independently.

## Provider Catalog

| Canonical ID | Aliases | Local | Provider-specific env vars |
|---|---|---:|---|
| `openrouter` | — | No | `OPENROUTER_API_KEY` |
| `anthropic` | — | No | `ANTHROPIC_OAUTH_TOKEN`, `ANTHROPIC_API_KEY` |
| `openai` | — | No | `OPENAI_API_KEY` |
| `ollama` | — | Yes | `OLLAMA_API_KEY` |
| `gemini` | `google`, `google-gemini` | No | `GEMINI_API_KEY`, `GOOGLE_API_KEY` |
| `venice` | — | No | `VENICE_API_KEY` |
| `vercel` | `vercel-ai` | No | `VERCEL_API_KEY` |
| `cloudflare` | `cloudflare-ai` | No | `CLOUDFLARE_API_KEY` |
| `moonshot` | `kimi` | No | `MOONSHOT_API_KEY` |
| `stepfun` | `step`, `step-ai`, `step_ai` | No | `STEP_API_KEY`, `STEPFUN_API_KEY` |
| `kimi-code` | `kimi_coding`, `kimi_for_coding` | No | `KIMI_CODE_API_KEY`, `MOONSHOT_API_KEY` |
| `synthetic` | — | No | `SYNTHETIC_API_KEY` |
| `opencode` | `opencode-zen` | No | `OPENCODE_API_KEY` |
| `zai` | `z.ai` | No | `ZAI_API_KEY` |
| `glm` | `zhipu` | No | `GLM_API_KEY` |
| `minimax` | `minimax-intl`, `minimax-io`, `minimax-global`, `minimax-cn`, `minimax-oauth` | No | `MINIMAX_OAUTH_TOKEN`, `MINIMAX_API_KEY` |
| `bedrock` | `aws-bedrock` | No | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` |
| `qianfan` | `baidu` | No | `QIANFAN_API_KEY` |
| `doubao` | `volcengine`, `ark`, `doubao-cn` | No | `ARK_API_KEY`, `DOUBAO_API_KEY` |
| `siliconflow` | `silicon-cloud`, `siliconcloud` | No | `SILICONFLOW_API_KEY` |
| `hunyuan` | `tencent` | No | `HUNYUAN_API_KEY` |
| `qwen` | `dashscope`, `qwen-intl`, `dashscope-intl`, `qwen-us`, `qwen-code`, `qwen-oauth` | No | `QWEN_OAUTH_TOKEN`, `DASHSCOPE_API_KEY` |
| `groq` | — | No | `GROQ_API_KEY` |
| `mistral` | — | No | `MISTRAL_API_KEY` |
| `xai` | `grok` | No | `XAI_API_KEY` |
| `deepseek` | — | No | `DEEPSEEK_API_KEY` |
| `together` | `together-ai` | No | `TOGETHER_API_KEY` |
| `fireworks` | `fireworks-ai` | No | `FIREWORKS_API_KEY` |
| `novita` | — | No | `NOVITA_API_KEY` |
| `perplexity` | — | No | `PERPLEXITY_API_KEY` |
| `cohere` | — | No | `COHERE_API_KEY` |
| `copilot` | `github-copilot` | No | GitHub token via config or `API_KEY` fallback |
| `cursor` | — | Yes | none; Cursor manages auth |
| `lmstudio` | `lm-studio` | Yes | optional |
| `llamacpp` | `llama.cpp` | Yes | `LLAMACPP_API_KEY` |
| `sglang` | — | Yes | `SGLANG_API_KEY` |
| `vllm` | — | Yes | `VLLM_API_KEY` |
| `osaurus` | — | Yes | `OSAURUS_API_KEY` |
| `nvidia` | `nvidia-nim`, `build.nvidia.com` | No | `NVIDIA_API_KEY` |

## Common Setup

Environment variable:

```bash
export OPENROUTER_API_KEY="..."
zeroclaw onboard --provider openrouter
```

Config file:

```toml
default_provider = "openrouter"
default_model = "anthropic/claude-sonnet-4-6"
```

Custom endpoint:

```toml
default_provider = "custom:https://your-api.example.com"
default_model = "your-model-id"
```

## Provider Notes

### Gemini

- Accepts `GEMINI_API_KEY`, `GOOGLE_API_KEY`, or Gemini CLI OAuth cache
- Thinking models are supported
- Internal reasoning parts are filtered from final responses

### Qwen

- Supports OAuth and DashScope API-key flows
- `qwen-code` is the OAuth-oriented path
- Daily free-tier limits apply to some OAuth-backed flows

### Doubao / Volcengine

- Canonical runtime ID is `doubao`
- Common aliases: `volcengine`, `ark`
- Use `ARK_API_KEY` where possible

### Bedrock

- Uses AWS credentials, not a single API key
- Supports tool calling and prompt caching via Converse API

### Ollama

- Default local endpoint: `http://localhost:11434`
- Vision is driven by `[IMAGE:<source>]` markers
- `:cloud` model suffix requires a remote `api_url`
- Optional reasoning toggle:

```toml
[runtime]
reasoning_enabled = false
```

### Cursor

- Provider ID: `cursor`
- Invokes the headless Cursor CLI
- Best suited for batch usage rather than high-throughput multi-turn inference

### Local Servers

| Provider | Default Endpoint |
|---|---|
| `lmstudio` | `http://localhost:1234/v1` |
| `llamacpp` | `http://localhost:8080/v1` |
| `sglang` | `http://localhost:30000/v1` |
| `vllm` | `http://localhost:8000/v1` |
| `osaurus` | `http://localhost:1337/v1` |

## Validation

```bash
zeroclaw providers
zeroclaw models refresh --provider openrouter
zeroclaw agent -m "ping"
zeroclaw providers-quota --format json
```
