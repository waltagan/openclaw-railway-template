# exec denied: allowlist miss

## Problema

O agente OpenClaw recebia `exec denied: allowlist miss` ao tentar executar qualquer comando (kill, psql, python, curl, etc.), mesmo com as seguintes configs aplicadas no `openclaw.json`:

- `tools.exec.ask: "off"`
- `tools.sandbox.tools.allow: ["*"]`
- `tools.sandbox.tools.deny: []`
- `approvals allowlist add --agent "*" "*"`

## Causa raiz

O OpenClaw tem **duas camadas separadas** que controlam a execução de comandos:

| Camada | Arquivo | Controla |
|--------|---------|----------|
| Layer 1 | `openclaw.json` (`tools.exec.*`) | Política de ferramentas do agente |
| Layer 2 | `~/.openclaw/exec-approvals.json` (`defaults.*`) | Gating real de execução no host |

A permissão efetiva é a **mais restritiva** entre as duas. Configurar apenas a Layer 1 não é suficiente — a Layer 2 (`exec-approvals.json`) mantém sua política default de `security: "allowlist"` + `ask: "on-miss"`, que bloqueia tudo que não está no allowlist.

## Solução

Configurar **ambas** as camadas no startup do container (`server.js`):

### Layer 1 — openclaw.json

```bash
openclaw config set tools.exec.security full
openclaw config set tools.exec.ask off
openclaw config set --json approvals.exec '{"enabled":false}'
```

### Layer 2 — exec-approvals.json (escrita direta)

```json
{
  "version": 1,
  "defaults": {
    "security": "full",
    "ask": "off",
    "askFallback": "full"
  },
  "agents": {
    "*": { "allowlist": [{ "pattern": "*" }] },
    "main": { "allowlist": [{ "pattern": "*" }] }
  }
}
```

O arquivo deve ser escrito em `$STATE_DIR/exec-approvals.json` (no Railway: `/data/.openclaw/exec-approvals.json`).

## Referências

- https://github.com/openclaw/openclaw/issues/15047
- https://github.com/openclaw/openclaw/issues/41156
- https://coclaw.com/guides/openclaw-exec-approvals-and-safe-bins
