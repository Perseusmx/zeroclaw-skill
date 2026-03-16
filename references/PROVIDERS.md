# ZeroClaw Providers Reference

ZeroClaw supports 40+ AI providers through a unified interface. This document provides complete provider catalog, authentication methods, and configuration examples.

**Quick reference:**
```bash
# List all providers
zeroclaw providers

# Refresh model catalogs
zeroclaw models refresh

# Refresh specific provider
zeroclaw models refresh --provider <provider-id>
```

## Credential Resolution Order

ZeroClaw resolves credentials in this order:

1. **Explicit credential** from config.toml or CLI
2. **Provider-specific environment variable**
3. **Generic fallback**: `ZEROCLAW_API_KEY`, then `API_KEY`

For fallback chains (`reliability.fallback_providers`), each fallback provider resolves credentials independently.

## Complete Provider Catalog

| Provider ID | Aliases | Local | Environment Variable |
|---|---|---:|---|
| `openrouter` | — | No | `OPENROUTER_API_KEY` |
| `anthropic` | — | No | `ANTHROPIC_OAUTH_TOKEN`, `ANTHROPIC_API_KEY` |
| `openai` | — | No | `OPENAI_API_KEY` |
| `openai-codex` | — | No | (OAuth via Codex CLI) |
| `azure-openai` | `azure_openai` | No | `AZURE_OPENAI_API_KEY`, `AZURE_OPENAI_ENDPOINT` |
| `ollama` | — | Yes | `OLLAMA_API_KEY` (optional) |
| `gemini` | `google`, `google-gemini` | No | `GEMINI_API_KEY`, `GOOGLE_API_KEY` |
| `venice` | — | No | `VENICE_API_KEY` |
| `vercel` | `vercel-ai` | No | `VERCEL_API_KEY` |
| `cloudflare` | `cloudflare-ai` | No | `CLOUDFLARE_API_KEY` |
| `moonshot` | `kimi` | No | `MOONSHOT_API_KEY` |
| `kimi-code` | `kimi_coding`, `kimi_for_coding` | No | `KIMI_CODE_API_KEY`, `MOONSHOT_API_KEY` |
| `synthetic` | — | No | `SYNTHETIC_API_KEY` |
| `opencode` | `opencode-zen` | No | `OPENCODE_API_KEY` |
| `zai` | `z.ai` | No | `ZAI_API_KEY` |
| `glm` | `zhipu` | No | `GLM_API_KEY` |
| `minimax` | `minimax-intl`, `minimax-io`, `minimax-global`, `minimax-cn`, `minimaxi`, `minimax-oauth`, `minimax-oauth-cn`, `minimax-portal`, `minimax-portal-cn` | No | `MINIMAX_OAUTH_TOKEN`, `MINIMAX_API_KEY` |
| `bedrock` | `aws-bedrock` | No | `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` |
| `qianfan` | `baidu` | No | `QIANFAN_API_KEY` |
| `doubao` | `volcengine`, `ark`, `doubao-cn` | No | `ARK_API_KEY`, `DOUBAO_API_KEY` |
| `qwen` | `dashscope`, `qwen-intl`, `dashscope-intl`, `qwen-us`, `dashscope-us`, `qwen-code`, `qwen-oauth`, `qwen_oauth` | No | `QWEN_OAUTH_TOKEN`, `DASHSCOPE_API_KEY` |
| `groq` | — | No | `GROQ_API_KEY` |
| `mistral` | — | No | `MISTRAL_API_KEY` |
| `xai` | `grok` | No | `XAI_API_KEY` |
| `deepseek` | — | No | `DEEPSEEK_API_KEY` |
| `together` | `together-ai` | No | `TOGETHER_API_KEY` |
| `fireworks` | `fireworks-ai` | No | `FIREWORKS_API_KEY` |
| `novita` | — | No | `NOVITA_API_KEY` |
| `perplexity` | — | No | `PERPLEXITY_API_KEY` |
| `cohere` | — | No | `COHERE_API_KEY` |
| `copilot` | `github-copilot` | No | (use config/`API_KEY` fallback with GitHub token) |
| `cursor` | — | No | (Cursor IDE integration) |
| `lmstudio` | `lm-studio` | Yes | (optional; local by default) |
| `llamacpp` | `llama.cpp` | Yes | `LLAMACPP_API_KEY` (optional) |
| `sglang` | — | Yes | `SGLANG_API_KEY` (optional) |
| `vllm` | — | Yes | `VLLM_API_KEY` (optional) |
| `osaurus` | — | Yes | `OSAURUS_API_KEY` (optional) |
| `nvidia` | `nvidia-nim`, `build.nvidia.com` | No | `NVIDIA_API_KEY` |
| `telnyx` | — | No | `TELNYX_API_KEY` |
| `aihubmix` | — | No | `AIHUBMIX_API_KEY` |
| `siliconflow` | `silicon-flow` | No | `SILICONFLOW_API_KEY` |

## Quick Setup

### Environment Variable (Simple)

```bash
# Set provider key
export OPENROUTER_API_KEY="..."

# Set default provider
zeroclaw onboard --provider openrouter
```

### Configuration File (Recommended)

```toml
# ~/.zeroclaw/config.toml
default_provider = "openrouter"
# api_key resolved from $OPENROUTER_API_KEY or $ZEROCLAW_API_KEY env var (recommended)
default_model = "anthropic/claude-sonnet-4.5"
```

## Custom Provider Setup

### OpenAI-Compatible Endpoint

```toml
default_provider = "custom:https://your-api.example.com"
# api_key resolved from env var (see Credential Resolution Order above)
default_model = "your-model-id"
```

### Anthropic-Compatible Endpoint

```toml
default_provider = "anthropic-custom:https://your-api.example.com"
# api_key resolved from env var (see Credential Resolution Order above)
default_model = "your-model-id"
```

## Provider-Specific Notes

### Anthropic

- OAuth token or API key supported
- Uses official Anthropic Messages API
- Supports native tool calling and prompt caching

### OpenAI

- Uses official OpenAI Chat Completions API
- Supports GPT-4, GPT-4.1, and all OpenAI models
- Native function calling enabled

### Gemini

- Auth from `GEMINI_API_KEY`, `GOOGLE_API_KEY`, or Gemini CLI OAuth
- Thinking models (e.g. `gemini-3-pro-preview`) supported
- Internal reasoning parts filtered from responses

### Ollama

**Vision support:**
- Use `[IMAGE:<source>]` markers in messages
- Multimodal normalization to Ollama's `messages[].images` field

**Cloud routing:**
- Use `:cloud` suffix with remote endpoints
- Set `api_url` in config for remote Ollama
- Local model discovery excludes `:cloud` entries

**Reasoning toggle:**
```toml
[runtime]
reasoning_enabled = false
```

### AWS Bedrock

- Requires AWS credentials: `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`
- Optional: `AWS_SESSION_TOKEN`, `AWS_REGION` (default: `us-east-1`)
- Uses Converse API
- Supports native tool calling and prompt caching
- Cross-region inference profiles supported

### MiniMax

**OAuth setup:**
```toml
default_provider = "minimax-oauth"
# api_key resolved from $MINIMAX_OAUTH_TOKEN or $MINIMAX_API_KEY env var
```

Environment variables:
- `MINIMAX_OAUTH_TOKEN` (preferred)
- `MINIMAX_API_KEY` (legacy)
- `MINIMAX_OAUTH_REFRESH_TOKEN` (auto-refresh)

Optional:
- `MINIMAX_OAUTH_REGION` (global/cn)
- `MINIMAX_OAUTH_CLIENT_ID`

### Qwen Code

**OAuth setup:**
```toml
default_provider = "qwen-code"
# api_key resolved from $QWEN_OAUTH_TOKEN env var or ~/.qwen/oauth_creds.json
```

Credential resolution:
1. Explicit `api_key` value
2. `QWEN_OAUTH_TOKEN`
3. `~/.qwen/oauth_creds.json` (cached)
4. `QWEN_OAUTH_REFRESH_TOKEN`
5. `DASHSCOPE_API_KEY` (fallback)

Optional: `QWEN_OAUTH_RESOURCE_URL`

### Azure OpenAI

- Requires `AZURE_OPENAI_API_KEY` and `AZURE_OPENAI_ENDPOINT`
- Endpoint format: `https://<resource>.openai.azure.com`
- Supports all Azure-deployed models
- Uses Azure-specific API versioning

### OpenAI Codex

- OAuth-based authentication via Codex CLI
- Provider ID: `openai-codex`
- Credentials resolved from Codex CLI OAuth cache

### Kimi Code

- Endpoint: `https://api.kimi.com/coding/v1`
- Default model: `kimi-for-coding` (alternative: `kimi-k2.5`)
- Runtime adds `User-Agent: KimiCLI/0.77`

### Local Servers

**Ollama:**
- Default: `http://localhost:11434`
- Optional: `OLLAMA_API_KEY` if auth enabled

**LM Studio:**
- Local by default, no auth required
- Auto-detects running server

**llama.cpp:**
- Default: `http://localhost:8080/v1`
- Optional: `LLAMACPP_API_KEY` if `--api-key` set

**SGLang:**
- Default: `http://localhost:30000/v1`
- Optional: `SGLANG_API_KEY` if auth required
- Requires `--tool-call-parser` for tool calling

**vLLM:**
- Default: `http://localhost:8000/v1`
- Optional: `VLLM_API_KEY` if auth required

**Osaurus:**
- Default: `http://localhost:1337/v1`
- Default key: `"osaurus"` (optional)
- Supports multiple API formats: OpenAI, Anthropic, Ollama, Open Responses
- Built-in MCP support

### NVIDIA NIM

- Base URL: `https://integrate.api.nvidia.com/v1`
- Model discovery: `zeroclaw models refresh --provider nvidia`

Recommended models:
- `meta/llama-3.3-70b-instruct`
- `deepseek-ai/deepseek-v3.2`
- `nvidia/llama-3.3-nemotron-super-49b-v1.5`
- `nvidia/llama-3.1-nemotron-ultra-253b-v1`

### Telnyx

- Provider ID: `telnyx`
- Uses `TELNYX_API_KEY`
- OpenAI-compatible endpoint

### AiHubMix

- Provider ID: `aihubmix`
- Multi-provider aggregator
- Uses `AIHUBMIX_API_KEY`

### SiliconFlow

- Provider ID: `siliconflow` (alias: `silicon-flow`)
- Uses `SILICONFLOW_API_KEY`
- OpenAI-compatible endpoint

### Vercel AI Gateway

- Provider ID: `vercel` or `vercel-ai`
- Base URL: `https://ai-gateway.vercel.sh/v1`
- No project deployment required
- Use `VERCEL_API_KEY` for authentication
- Avoid `https://api.vercel.ai` (returns DEPLOYMENT_NOT_FOUND)

## Model Routing by Hint

Route models dynamically using hints:

```toml
[[model_routes]]
hint = "reasoning"
provider = "openrouter"
model = "anthropic/claude-opus-4-20250514"

[[model_routes]]
hint = "fast"
provider = "groq"
model = "llama-3.3-70b-versatile"
```

Use in tool calls:
```text
hint:reasoning
hint:fast
```

## Embedding Routing

Configure embedding providers with hints:

```toml
[memory]
embedding_model = "hint:semantic"

[[embedding_routes]]
hint = "semantic"
provider = "openai"
model = "text-embedding-3-small"
dimensions = 1536

[[embedding_routes]]
hint = "archive"
provider = "custom:https://embed.example.com/v1"
model = "your-embedding-model-id"
dimensions = 1024
```

Optional per-route key:
```toml
[[embedding_routes]]
hint = "semantic"
provider = "openai"
model = "text-embedding-3-small"
# api_key from env var or set here (encrypted at rest)
```

## Upgrading Models Safely

Use stable hints to minimize breakage:

1. Keep call sites stable (`hint:reasoning`, `hint:semantic`)
2. Change only target models under `[[model_routes]]`
3. Run `zeroclaw doctor` and `zeroclaw status`
4. Smoke test one representative flow before rollout

## Security Considerations

- Never commit API keys to version control
- Use environment variables for sensitive credentials
- Set file permissions: `chmod 600 ~/.zeroclaw/config.toml`
- Rotate keys regularly
- Use minimal permissions for service accounts
- Consider using secret managers for production deployments
