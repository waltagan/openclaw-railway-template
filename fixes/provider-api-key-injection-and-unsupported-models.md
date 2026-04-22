# Provider API Key Injection Failure + Unsupported GPT-5.4 Models

## Data: 2026-04-13

## Problema 1: API keys nao foram injetadas no config

### Sintoma
```
[WARN] failed to set models.providers.openai: exit=1
Error: Config validation failed: models.providers.openai.baseUrl:
Invalid input: expected string, received undefined
```

(Outros providers mostram o mesmo padrao: `baseUrl` ausente.)

### Causa raiz
O schema de validacao do Openclaw exige que `baseUrl` esteja presente junto
com `apiKey` ao configurar um provider. Usar `openclaw config set
models.providers.openai.apiKey <key>` falha porque cria um objeto
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

Exemplos de configuracao completa (OpenAI e Google neste template):
- **OpenAI:** `baseUrl` `https://api.openai.com/v1`, `models: [{ id: "gpt-5.4" }, { id: "gpt-4.1" }, ...]`
- **Google:** `baseUrl` `https://generativelanguage.googleapis.com`, `models: [{ id: "gemini-3-pro-preview" }, { id: "gemini-2.5-flash" }]`

### Solucao (iteracao 3 - v2026.4.12 schema refinado)
O campo `models` exige array de **objetos** com o identificador do modelo, nao strings:
```js
// Errado: models: ["gpt-5.4"]
// v2026.4.x antigo: models: [{ name: "gpt-5.4" }]
// Versao mais recente: models: [{ id: "gpt-5.4" }, ...]  (validacao pede `id`)
```

**Licao:** O schema de `models.providers.<name>` muda entre versoes do Openclaw.
Sempre validar os campos obrigatorios nos logs apos atualizar a versao.
Ver tambem: `fixes/openclaw-models-id-eacces-plugin-install.md` (id + EACCES em container).

## Problema 3: TUI token_mismatch

### Sintoma
```
[ws] unauthorized ... reason=token_mismatch
gateway connect failed: GatewayClientRequestError: unauthorized: gateway token mismatch
```

### Causa raiz
O Web TUI spawna `openclaw tui` via PTY sem passar o token do gateway.
O `openclaw tui` tenta se conectar ao gateway usando o token do config file,
que pode estar dessincronizado com o token real do gateway (especialmente
apos `doctor --fix` na nova versao reescrever o config).

### Solucao
Passar o token explicitamente no spawn do TUI:
1. `--token <TOKEN>` como argumento CLI
2. `OPENCLAW_GATEWAY_TOKEN` como variavel de ambiente no env do PTY

## Problema 2: Modelos GPT-5.4 nao reconhecidos

### Sintoma
```
FailoverError: Unknown model: openai/gpt-5.4-mini
Model "openai/gpt-5.4-mini" not found. Fell back to "google/gemini-2.0-flash-lite".
```

### Causa raiz
O Openclaw v2026.3.13 nao possui os modelos GPT-5.4 no seu catalogo interno.
Esses modelos foram lancados pela OpenAI em marco de 2026, mas o suporte
depende da versao do Openclaw (v2026.4.12+ provavelmente os inclui).

### Solucao
Removidos os modelos `openai/gpt-5.4`, `openai/gpt-5.4-mini` e
`openai/gpt-5.4-nano` da lista de modelos e fallbacks. Mantidos apenas
modelos compativeis com v2026.3.13 (gpt-4.1, gpt-4.1-mini).

Para adicionar GPT-5.4 no futuro, atualizar o Openclaw no Dockerfile:
```dockerfile
RUN npm install -g openclaw@latest
```

## Arquivos alterados
- `src/server.js` (bloco de autonomy + provider injection)
