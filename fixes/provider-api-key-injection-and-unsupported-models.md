# Provider API Key Injection Failure + Unsupported GPT-5.4 Models

## Data: 2026-04-13

## Problema 1: API keys nao foram injetadas no config

### Sintoma
```
[WARN] failed to set models.providers.anthropic.apiKey: exit=1
Error: Config validation failed: models.providers.anthropic.baseUrl:
Invalid input: expected string, received undefined
```

Mesmo erro para `models.providers.openai.apiKey`.

### Causa raiz
O schema de validacao do OpenClaw exige que `baseUrl` esteja presente junto
com `apiKey` ao configurar um provider. Usar `openclaw config set
models.providers.anthropic.apiKey <key>` falha porque cria um objeto
provider incompleto (sem baseUrl).

### Solucao (iteracao 1 - v2026.3.13)
Trocar de `config set <path> <value>` (campo individual) para
`config set --json <path> <json>` (objeto completo com apiKey + baseUrl).

### Solucao (iteracao 2 - v2026.4.12)
A versao v2026.4.12 adicionou campo obrigatorio `models` (array) ao schema.
O objeto do provider agora precisa de apiKey + baseUrl + models:

```js
const providerJson = JSON.stringify({ apiKey: key, baseUrl, models });
await runCmd(OPENCLAW_NODE, clawArgs(["config", "set", "--json", cfgPath, providerJson]));
```

Configuracao completa por provider:
- Anthropic: baseUrl `https://api.anthropic.com`, models `["claude-sonnet-4-6", "claude-opus-4-6"]`
- OpenAI: baseUrl `https://api.openai.com/v1`, models `["gpt-5.4", "gpt-5.4-mini", "gpt-5.4-nano", "gpt-4.1", "gpt-4.1-mini"]`
- Google: baseUrl `https://generativelanguage.googleapis.com`, models `["gemini-3-pro-preview", "gemini-2.5-flash"]`

**Licao:** O schema de `models.providers.<name>` muda entre versoes do OpenClaw.
Sempre validar os campos obrigatorios nos logs apos atualizar a versao.

## Problema 2: Modelos GPT-5.4 nao reconhecidos

### Sintoma
```
FailoverError: Unknown model: openai/gpt-5.4-mini
Model "openai/gpt-5.4-mini" not found. Fell back to "google/gemini-2.0-flash-lite".
```

### Causa raiz
O OpenClaw v2026.3.13 nao possui os modelos GPT-5.4 no seu catalogo interno.
Esses modelos foram lancados pela OpenAI em marco de 2026, mas o suporte
depende da versao do OpenClaw (v2026.4.12+ provavelmente os inclui).

### Solucao
Removidos os modelos `openai/gpt-5.4`, `openai/gpt-5.4-mini` e
`openai/gpt-5.4-nano` da lista de modelos e fallbacks. Mantidos apenas
modelos compativeis com v2026.3.13 (gpt-4.1, gpt-4.1-mini).

Para adicionar GPT-5.4 no futuro, atualizar o OpenClaw no Dockerfile:
```dockerfile
RUN npm install -g openclaw@latest
```

## Arquivos alterados
- `src/server.js` (bloco de autonomy + provider injection)
