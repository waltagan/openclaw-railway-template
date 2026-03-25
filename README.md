# OpenClaw + Railway + Cursor - Guia Completo de Setup

## O que e o OpenClaw?

O **OpenClaw** e um assistente pessoal de IA open-source que roda nos seus proprios dispositivos/servidores. Ele responde nos canais que voce ja usa (WhatsApp, Telegram, Slack, Discord, Web UI) e pode executar tarefas usando plugins/skills (ferramentas).

## Stack deste projeto

| Componente | Tecnologia |
|-----------|------------|
| **Hospedagem** | Railway (PaaS) |
| **IA Provider** | Google (Gemini) |
| **Canais** | WhatsApp + Web UI |
| **IDE/Dev** | Cursor |
| **Controle de versao** | GitHub |
| **Template** | codetitlan/openclaw-railway-template (oficial do Railway, +2000 deploys) |

---

## IMPORTANTE: Onde vivem os arquivos e como edita-los

Tudo roda no Railway. O OpenClaw **nao roda localmente**. Existem dois tipos de arquivos:

### Arquivos do TEMPLATE (voce ve no Cursor via Git)

Codigo do "wrapper" que empacota o OpenClaw para Railway. Voce edita no Cursor e faz push, Railway faz redeploy automatico.

```
openclaw-railway/               (seu repositorio Git - visivel no Cursor)
├── src/              # Servidor wrapper (setup wizard, proxy, health)
├── docs/             # Documentacao do template
├── Dockerfile        # Build do container
├── entrypoint.sh     # Script de inicializacao
├── railway.toml      # Configuracao do Railway
├── package.json      # Dependencias Node.js
├── .env.example      # Exemplo de variaveis de ambiente
└── README.md         # Este arquivo
```

### Arquivos de RUNTIME do OpenClaw (vivem no container Railway)

Configuracoes, IDENTITY.md, credenciais, skills, workspace. Ficam no volume `/data` dentro do container e **nao sincronizam automaticamente** com o Cursor.

```
/data/                           (volume Railway - NAO visivel no Cursor)
├── .openclaw/
│   ├── openclaw.json            # Configuracao principal
│   ├── IDENTITY.md              # Personalidade/instrucoes do agente
│   ├── credentials/             # Credenciais (WhatsApp, API keys)
│   └── plugins/                 # Plugins/skills instalados
└── workspace/                   # Workspace do agente
```

### Como editar arquivos de runtime no Cursor (fluxo copiar-editar-enviar)

Quando precisar editar um arquivo do OpenClaw (ex: `IDENTITY.md`), use este fluxo:

**1. Puxar o arquivo do Railway para o Cursor:**

No terminal do Cursor, entre no container e exiba o conteudo:
```bash
railway shell
cat /data/.openclaw/IDENTITY.md
```
Copie o conteudo e cole em um arquivo local (ex: `configs/IDENTITY.md`).

**Ou use um comando unico para copiar direto (sem entrar no shell interativo):**
```bash
railway run cat /data/.openclaw/IDENTITY.md > configs/IDENTITY.md
```

**2. Edite no Cursor** com todo conforto (autocomplete, formatacao, etc.)

**3. Envie de volta para o Railway:**

No terminal do Cursor:
```bash
railway shell
```

Dentro do shell, use o editor ou cole o conteudo:
```bash
cat > /data/.openclaw/IDENTITY.md << 'EOF'
(cole o conteudo editado aqui)
EOF
```

**Ou via Web TUI:** acesse `https://seu-app.up.railway.app/tui` e edite com `nano` ou `vi`.

### Pasta `configs/` local (opcional, para referencia)

Recomendo criar uma pasta `configs/` no repositorio para manter copias de referencia dos arquivos de runtime. Isso permite:
- Ter historico Git das suas mudancas
- Editar com conforto no Cursor
- Compartilhar configuracoes entre ambientes

```bash
mkdir -p configs
```

> **ATENCAO:** Os arquivos em `configs/` sao apenas copias de referencia. A versao "real" que o OpenClaw usa esta sempre dentro do container em `/data/.openclaw/`. Apos editar, voce precisa enviar de volta para o container.

### Resumo dos metodos de acesso ao container:

| Metodo | Como usar | Melhor para |
|--------|-----------|-------------|
| **Railway CLI** | `railway shell` no terminal do Cursor | Copiar arquivos, comandos rapidos |
| **Railway run** | `railway run cat <arquivo>` no Cursor | Puxar arquivo sem shell interativo |
| **Web TUI** | `https://seu-app.up.railway.app/tui` | Editar com nano/vi no navegador |
| **Railway Dashboard** | Dashboard → servico → Shell | Acesso rapido pelo navegador |
| **Setup Wizard** | `https://seu-app.up.railway.app/setup` | Config editor visual |

---

## Indice

1. [Pre-requisitos](#1-pre-requisitos)
2. [Criar o repositorio no GitHub](#2-criar-o-repositorio-no-github)
3. [Deploy no Railway](#3-deploy-no-railway)
4. [Configurar variaveis de ambiente](#4-configurar-variaveis-de-ambiente)
5. [Configurar volume persistente](#5-configurar-volume-persistente)
6. [Habilitar rede publica](#6-habilitar-rede-publica)
7. [Acessar o Setup Wizard](#7-acessar-o-setup-wizard)
8. [Configurar o Google Gemini como provider](#8-configurar-o-google-gemini-como-provider)
9. [Conectar o WhatsApp](#9-conectar-o-whatsapp)
10. [Acessar a Web UI (Control UI)](#10-acessar-a-web-ui-control-ui)
11. [Conectar Cursor ao projeto](#11-conectar-cursor-ao-projeto)
12. [Customizar o OpenClaw](#12-customizar-o-openclaw)
13. [Comandos uteis](#13-comandos-uteis)
14. [Troubleshooting](#14-troubleshooting)
15. [Links de referencia](#15-links-de-referencia)

---

## 1. Pre-requisitos

Antes de comecar, voce vai precisar de:

- [ ] **Conta no Railway** (voce ja tem)
  - Acesse: https://railway.com
  - Certifique-se de ter um plano que suporte volumes (plano Trial ou superior)
- [ ] **Conta no GitHub** (voce ja tem)
  - Acesse: https://github.com
- [ ] **API Key do Google Gemini**
  - Acesse: https://aistudio.google.com/apikey
  - Clique em "Create API Key"
  - Copie e guarde a chave (voce vai precisar dela no passo 7)
- [ ] **WhatsApp** instalado no celular (para vincular)
- [ ] **Cursor IDE** instalado
  - Acesse: https://cursor.com (se ainda nao tem)
- [ ] **Git** instalado no seu computador
  - Verifique com: `git --version`
- [ ] **Node.js** instalado (apenas para instalar Railway CLI)
  - Verifique com: `node --version`
  - Se nao tiver: https://nodejs.org

---

## 2. Criar o repositorio no GitHub

> **Ja feito!** Fork criado em: https://github.com/waltagan/openclaw-railway-template

O fork foi criado a partir do template oficial: https://github.com/codetitlan/openclaw-railway-template

Para clonar no Cursor (ja feito):

```bash
cd /Users/waltagan/openclaw_railway
git clone https://github.com/waltagan/openclaw-railway-template.git .
```

> **Status:** Fork ja criado e clonado no Cursor.

---

## 3. Deploy no Railway

1. Acesse: https://railway.com/new
2. Clique em **"Deploy from GitHub repo"**
3. Conecte sua conta GitHub (se ainda nao conectou)
4. Selecione o repositorio **waltagan/openclaw-railway-template**
5. Railway vai detectar o `Dockerfile` automaticamente
6. Clique em **"Deploy Now"**

> **Vantagem do fork:** Voce edita os arquivos do template no Cursor, faz push, e o Railway faz redeploy automatico.

---

## 4. Configurar variaveis de ambiente

No painel do Railway, va em **Settings > Variables** do seu servico e adicione:

### Variavel obrigatoria:

| Variavel | Valor | Descricao |
|---------|-------|-----------|
| `SETUP_PASSWORD` | `sua_senha_segura_aqui` | Senha para acessar o wizard e autenticar no gateway |

### Variaveis recomendadas:

| Variavel | Valor | Descricao |
|---------|-------|-----------|
| `OPENCLAW_STATE_DIR` | `/data/.openclaw` | Diretorio de estado persistente |
| `OPENCLAW_WORKSPACE_DIR` | `/data/workspace` | Diretorio de trabalho persistente |
| `OPENCLAW_GATEWAY_TOKEN` | `token_secreto_aqui` | Token estavel entre redeploys |

### Variaveis opcionais:

| Variavel | Valor | Descricao |
|---------|-------|-----------|
| `PORT` | `8080` | Porta do servico (padrao do Railway) |
| `INTERNAL_GATEWAY_PORT` | `18789` | Porta interna do gateway OpenClaw |
| `ENABLE_WEB_TUI` | `true` | Habilita terminal no navegador em `/tui` |
| `TUI_IDLE_TIMEOUT_MS` | `3600000` | Timeout de inatividade do TUI (recomendado: 1h = 3600000) |
| `TUI_MAX_SESSION_MS` | `14400000` | Duracao maxima de sessao TUI (recomendado: 4h = 14400000) |

> **IMPORTANTE:** Nunca use senhas fracas. O `SETUP_PASSWORD` protege o acesso completo ao seu assistente. Recomendo habilitar `ENABLE_WEB_TUI=true` para ter acesso ao terminal via navegador.

---

## 5. Configurar volume persistente

O volume garante que configuracoes, credenciais e workspace sobrevivam a redeploys:

1. No painel do Railway, va ate o seu servico
2. Clique na aba **"Volumes"** (ou "Add Volume" no painel lateral)
3. Crie um volume com:
   - **Mount Path:** `/data`
   - **Size:** 5 GB (recomendado pela documentacao)
4. Clique em **"Create Volume"**

> **Por que isso e importante?** Sem o volume, toda vez que o Railway fizer redeploy, suas configuracoes, credenciais do WhatsApp e skills serao perdidas.

---

## 6. Habilitar rede publica

Para acessar a Web UI e o Setup Wizard, voce precisa de um dominio publico:

1. No painel do Railway, va ate **Settings > Networking**
2. Em **"Public Networking"**, habilite o **HTTP Proxy**
3. Porta: `8080`
4. Railway vai gerar um dominio automatico como: `https://openclawwaltagan-production.up.railway.app`
5. **Anote este dominio** - voce vai usa-lo nos proximos passos

> **Nota:** Railway fornece automaticamente a variavel `RAILWAY_PUBLIC_DOMAIN` que o template usa para configurar origens permitidas de WebSocket.

> **Seguranca:** O template expoe o gateway a internet publica. Revise a documentacao de seguranca em https://docs.openclaw.ai/gateway/security. Se voce so usa canais de chat (WhatsApp/Telegram), considere desabilitar a rede publica apos o setup inicial.

---

## 7. Acessar o Setup Wizard

Agora que o deploy esta rodando:

1. Abra o navegador e acesse:
   ```
   https://openclawwaltagan-production.up.railway.app/setup
   ```
2. Digite a senha que voce definiu em `SETUP_PASSWORD`
3. Voce vera o **Setup Wizard** com as seguintes opcoes:
   - Selecao de provider de IA
   - Configuracao de API key
   - Configuracao de canais de mensagem

### Endpoints disponiveis apos deploy:

| URL | Funcao |
|-----|--------|
| `/setup` | Wizard de configuracao (protegido por senha) |
| `/openclaw` | Control UI (interface do OpenClaw) |
| `/tui` | Terminal no navegador (se `ENABLE_WEB_TUI=true`) |
| `/healthz` | Health check publico |
| `/logs` | Logs em tempo real |

---

## 8. Configurar o Google Gemini como provider

No Setup Wizard:

1. Em **"AI Provider"**, selecione **"Google"**
2. Em **"API Key"**, cole a chave que voce gerou no Google AI Studio
3. O modelo padrao sera o **Gemini** (o mais recente disponivel)
4. Clique em **"Run Setup"**

### Obtendo a API Key do Google Gemini:

1. Acesse: https://aistudio.google.com/apikey
2. Faca login com sua conta Google
3. Clique em **"Create API Key"**
4. Selecione um projeto do Google Cloud (ou crie um novo)
5. Copie a API Key gerada
6. **Guarde em local seguro** (voce nao conseguira ve-la novamente)

> **Custo:** O Google Gemini oferece um tier gratuito generoso. Verifique os limites em https://ai.google.dev/pricing

---

## 9. Conectar o WhatsApp

Apos o setup inicial, conecte o WhatsApp:

### Via Web UI (recomendado):

1. Acesse a Control UI:
   ```
   https://openclawwaltagan-production.up.railway.app/openclaw
   ```
2. Va em **Channels** (Canais)
3. Selecione **WhatsApp**
4. Um **QR Code** sera exibido
5. No seu celular:
   - Abra o **WhatsApp**
   - Va em **Configuracoes > Dispositivos Vinculados**
   - Clique em **"Vincular um dispositivo"**
   - Escaneie o QR Code
6. Aguarde a vinculacao (pode levar alguns segundos)

### Via terminal (Web TUI ou Railway Shell):

```bash
openclaw channels login --channel whatsapp
```

### Configuracao de acesso (quem pode falar com o bot):

Apos vincular, configure quem pode enviar mensagens ao bot:

```bash
openclaw config set channels.whatsapp.dmPolicy pairing
```

- **`pairing`** (padrao): Novos usuarios precisam de aprovacao
- **`allowlist`**: Apenas numeros na lista podem interagir
- **`open`**: Qualquer pessoa pode interagir (nao recomendado)

Para aprovar um pedido de pareamento:

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODIGO>
```

Voce tambem pode aprovar pareamentos pela interface grafica do `/setup` (secao "Device Pairing Helper").

> **Recomendacao:** Use um **numero separado** para o WhatsApp do OpenClaw (um chip extra), pois o canal funciona como "WhatsApp Web" e pode conflitar com sessoes existentes.

---

## 10. Acessar a Web UI (Control UI)

A Web UI e sua interface principal de controle:

- **URL:** `https://openclawwaltagan-production.up.railway.app/openclaw`
- **Autenticacao:** Acesse primeiro pelo `/setup` e clique no link para o OpenClaw UI (o token e passado automaticamente)

Na Web UI voce pode:
- Conversar diretamente com o agente
- Gerenciar canais (WhatsApp, etc.)
- Instalar e configurar skills/plugins
- Ver logs e status do gateway
- Aprovar pareamentos de novos usuarios

---

## 11. Conectar Cursor ao projeto

### 11.1. Abrir o projeto no Cursor

Se voce ja clonou no passo 2:

1. Abra o Cursor
2. **File > Open Folder**
3. Selecione `/Users/waltagan/openclaw_railway`

Se ainda nao clonou:

```bash
cd /Users/waltagan/openclaw_railway
git clone https://github.com/waltagan/openclaw-railway-template.git .
```

### 11.2. Instalar Railway CLI no Cursor

A Railway CLI permite acessar o terminal do container direto do Cursor:

```bash
npm install -g @railway/cli
railway login
railway link
```

### 11.3. Acessar o terminal do container (3 formas)

#### Forma 1: Railway CLI no terminal do Cursor (recomendado)

```bash
railway shell
```

Isso abre um terminal remoto dentro do container Railway. Voce pode executar qualquer comando do OpenClaw:

```bash
openclaw status
openclaw doctor
cat /data/.openclaw/IDENTITY.md
```

#### Forma 2: Web TUI (terminal no navegador)

1. Certifique-se de que `ENABLE_WEB_TUI=true` esta nas variaveis do Railway
2. Acesse: `https://openclawwaltagan-production.up.railway.app/tui`
3. Autentique com o `SETUP_PASSWORD`

#### Forma 3: Railway Dashboard (navegador)

1. Acesse https://railway.com → seu projeto → seu servico → **Shell**

### 11.4. Fluxo copiar-editar-enviar (arquivos de runtime)

Quando precisar editar um arquivo do OpenClaw (ex: `IDENTITY.md`, `openclaw.json`):

**Passo 1 - Puxar o arquivo do container para o Cursor:**

```bash
railway run cat /data/.openclaw/IDENTITY.md > configs/IDENTITY.md
```

**Passo 2 - Editar no Cursor** com todo o conforto (syntax highlight, autocomplete, etc.)

**Passo 3 - Enviar de volta para o container:**

```bash
railway shell
```

Dentro do shell remoto:
```bash
cat > /data/.openclaw/IDENTITY.md << 'EOF'
(cole o conteudo editado aqui)
EOF
```

> **Dica:** Crie a pasta `configs/` no repositorio para manter copias locais dos arquivos de runtime. Assim voce tem historico Git das suas mudancas.

### 11.5. Dois fluxos de trabalho

| O que voce quer fazer | Fluxo |
|-----------------------|-------|
| **Editar o template** (Dockerfile, src/, setup wizard) | Editar no Cursor → `git push` → Railway redeploy automatico |
| **Editar configs do OpenClaw** (IDENTITY.md, openclaw.json, skills) | `railway run cat ...` → editar no Cursor → enviar via `railway shell` |

---

## 12. Customizar o OpenClaw

### Estrutura do projeto no Cursor (template):

```
openclaw-railway/
├── src/              # Servidor wrapper (setup wizard, proxy reverso, health)
├── docs/             # Documentacao do template
├── configs/          # Copias locais de arquivos de runtime (voce cria)
│   ├── IDENTITY.md   # Copia local da personalidade do agente
│   └── openclaw.json # Copia local da configuracao
├── Dockerfile        # Build do container (instala OpenClaw via npm)
├── entrypoint.sh     # Script de inicializacao do container
├── railway.toml      # Configuracao de build/deploy do Railway
├── package.json      # Dependencias Node.js
├── .env.example      # Exemplo de variaveis de ambiente
└── README.md         # Este arquivo
```

> A pasta `configs/` e opcional, criada por voce para manter copias locais dos arquivos de runtime para referencia e historico Git.

### Funcionalidades disponiveis via web:

| Funcionalidade | URL | Descricao |
|----------------|-----|-----------|
| **Setup Wizard** | `/setup` | Configurar provider, API key e canais |
| **Control UI** | `/openclaw` | Chat, canais e plugins |
| **Web TUI** | `/tui` | Terminal no navegador |
| **Health Check** | `/healthz` | Monitoramento de saude |
| **Live Logs** | `/logs` | Logs em tempo real |
| **Config Editor** | via `/setup` | Editor do `openclaw.json` com backup automatico |
| **Debug Console** | via `/setup` | 13+ comandos permitidos sem SSH |
| **Device Pairing** | via `/setup` | Aprovar dispositivos WebSocket |
| **Backup/Restore** | via `/setup` | Export/import de configuracao completa |

### Editar IDENTITY.md (personalidade do agente):

```bash
# Puxar do container
railway run cat /data/.openclaw/IDENTITY.md > configs/IDENTITY.md

# Editar no Cursor...

# Enviar de volta
railway shell
cat > /data/.openclaw/IDENTITY.md << 'EOF'
(conteudo editado)
EOF
```

### Instalar novas skills (via terminal remoto):

```bash
railway shell
openclaw skills install <nome-da-skill>
```

### Criar uma skill customizada:

```bash
railway shell
openclaw skills create minha-skill
```

### Pinear versao do OpenClaw:

Adicione a variavel de ambiente no Railway:
```
OPENCLAW_VERSION=v2026.2.16
```

Se nao definir, o template detecta automaticamente a ultima versao estavel.

---

## 13. Comandos uteis

### Comandos OpenClaw (executar no terminal remoto: TUI, railway shell ou dashboard shell):

```bash
# Status geral
openclaw status
openclaw doctor
openclaw gateway status

# Canais
openclaw channels status
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp

# Skills
openclaw skills list
openclaw skills install <nome>
openclaw skills remove <nome>

# Configuracao
openclaw config list
openclaw config set <chave> <valor>

# Backup
openclaw backup create

# Logs
openclaw logs --follow

# Atualizacao
openclaw update
```

### Comandos Railway CLI (executar no terminal do Cursor):

```bash
# Login
railway login

# Vincular projeto
railway link

# Ver logs do servico
railway logs

# Acessar shell remoto do container
railway shell

# Ver variaveis de ambiente
railway variables

# Status do deploy
railway status

# Abrir no navegador
railway open
```

### Comandos Git (executar no terminal do Cursor):

```bash
# Ver alteracoes
git status

# Adicionar alteracoes
git add .

# Commit
git commit -m "descricao da alteracao"

# Push (dispara redeploy automatico no Railway)
git push origin main

# Pull (atualizar local com alteracoes do remoto)
git pull origin main
```

---

## 14. Troubleshooting

### Problema: Deploy falha no Railway

**Solucao:**
1. Verifique os logs: `railway logs`
2. Certifique-se de que o `SETUP_PASSWORD` esta definido
3. Verifique se o volume esta montado em `/data`

### Problema: Control UI diz "disconnected" ou "pairing required"

**Solucao:**
1. Acesse `/setup` primeiro, depois clique no link para OpenClaw UI (passa o token automaticamente)
2. Aprove dispositivos pendentes na secao "Device Pairing" do `/setup`
3. Execute `openclaw doctor --repair` no terminal remoto

### Problema: WhatsApp desconecta frequentemente

**Solucao:**
1. No terminal remoto, execute: `openclaw doctor`
2. Se necessario, re-vincule: `openclaw channels login --channel whatsapp`
3. Certifique-se de que `OPENCLAW_STATE_DIR=/data/.openclaw` esta definido

### Problema: Web UI nao carrega

**Solucao:**
1. Verifique se o HTTP Proxy esta habilitado na porta `8080`
2. Verifique o health check: `https://openclawwaltagan-production.up.railway.app/healthz`
3. Tente acessar: `https://openclawwaltagan-production.up.railway.app/setup`

### Problema: Gemini retorna erro de API

**Solucao:**
1. Verifique se a API Key esta correta
2. Acesse https://aistudio.google.com/apikey e gere uma nova chave
3. Atualize via Setup Wizard ou terminal remoto

### Problema: Setup reseta apos redeploy

**Solucao:**
1. Verifique se `OPENCLAW_STATE_DIR=/data/.openclaw` esta definido
2. Verifique se `OPENCLAW_WORKSPACE_DIR=/data/workspace` esta definido
3. Verifique se o volume esta montado em `/data`

### Problema: TUI nao aparece

**Solucao:**
1. Defina `ENABLE_WEB_TUI=true` nas variaveis do Railway
2. Redeploy o servico
3. Acesse `https://openclawwaltagan-production.up.railway.app/tui`

### Problema: Railway CLI nao conecta

**Solucao:**
1. Verifique a versao: `railway --version`
2. Faca login novamente: `railway login`
3. Re-vincule o projeto: `railway link`

---

## 15. Links de referencia

### Documentacao oficial:
- OpenClaw Docs: https://docs.openclaw.ai
- OpenClaw Install: https://docs.openclaw.ai/install
- OpenClaw Railway: https://docs.openclaw.ai/install/railway
- OpenClaw WhatsApp: https://docs.openclaw.ai/channels/whatsapp
- OpenClaw Gateway: https://docs.openclaw.ai/gateway
- OpenClaw Remote Access: https://docs.openclaw.ai/gateway/remote
- OpenClaw Security: https://docs.openclaw.ai/gateway/security

### Repositorios GitHub:
- OpenClaw (projeto oficial): https://github.com/openclaw/openclaw
- Railway Template (oficial do Railway): https://github.com/codetitlan/openclaw-railway-template
- Railway Deploy Page: https://railway.com/deploy/openclaw-railway-template

### Plataformas:
- Railway: https://railway.com
- Google AI Studio (API Keys): https://aistudio.google.com/apikey
- Cursor IDE: https://cursor.com

### Tutoriais em video:
- Full OpenClaw Setup Tutorial: https://www.youtube.com/watch?v=fcZMmP5dsl4
- OpenClaw + Railway: https://www.youtube.com/watch?v=SplQZqjWoiA

---

## Patterns e Tecnologias

| Tecnologia | Versao/Detalhes |
|-----------|-----------------|
| **OpenClaw** | Latest (instalado via npm no Docker build) |
| **Node.js** | 22.16+ / 24 (recomendado) |
| **Runtime** | Docker (Dockerfile) |
| **Cloud** | Railway (PaaS) |
| **IA** | Google Gemini |
| **Canais** | WhatsApp (Baileys) + Web UI |
| **IDE** | Cursor |
| **VCS** | Git + GitHub |
| **Package Manager** | pnpm |
| **Template** | codetitlan/openclaw-railway-template v1.2.0 |
