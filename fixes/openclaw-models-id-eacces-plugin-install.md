# OpenClaw: `models.providers.*.models[].id` e EACCES em `doctor` / plugins (Railway)

**Data:** 2026-04-22

## 1) Erro: `Config validation failed: models.providers.openai.models.0.id: expected string, received undefined`

### Sintoma
Ao subir o wrapper com `OPENAI_API_KEY` no Railway, o `config set --json models.providers.openai` falhava. Versões recentes do OpenClaw passaram a exigir cada item de `models` com campo **`id`** (string do id do modelo), não `name`.

### Correcao
Injetar:
`models: [{ id: "gpt-5.4" }, ...]` e o mesmo para Google: `{ id: "gemini-2.5-flash" }`, etc.

## 2) Erro: `EACCES` / `npm` ao instalar deps de plugins (`openclaw doctor --fix`, gateway)

### Sintoma
Mensagens como:
`permission denied, mkdir '/usr/local/lib/node_modules/openclaw/dist/extensions/.../node_modules'`
O processo roda como usuario `openclaw`, mas a instalacao global do pacote estava com dono `root`, entao o npm nao podia instalar dependencias on-demand usadas por plugins (acpx, browser, etc.).

### Correcao
No `Dockerfile`, apos `useradd`, dar:
`chown -R openclaw:openclaw /usr/local/lib/node_modules/openclaw`
para o usuario de runtime poder escrever dentro da instalacao do OpenClaw (install de bundled runtime deps do doctor/plugins).

**Nota de seguranca:** o diretorio continua so no image da app; o volume do Railway continua em `/data` para estado.

## 3) Erros de bonjour / systemd (informativo)

Avisos como `systemd user services are unavailable` ou `Gateway not running` no doctor sao esperados em container sem systemd; nao requerem correcao no template além do gateway ser iniciado pelo wrapper com `openclaw gateway run`.
