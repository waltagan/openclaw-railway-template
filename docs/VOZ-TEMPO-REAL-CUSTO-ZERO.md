# Comunicação por Voz em Tempo Real com OpenClaw — Opções de Custo Zero

Pesquisa realizada em 25/03/2026 sobre as melhores formas de criar uma interface de voz em tempo real com o OpenClaw, priorizando custo zero.

---

## Opção 1: Web Speech API (Navegador Nativo)

**Dificuldade:** Fácil | **Custo:** R$ 0 | **Funciona no Railway:** Sim

Usa as APIs nativas do próprio navegador para reconhecimento e síntese de voz. Nenhum servidor extra ou API paga necessária.

| Componente | Tecnologia |
|---|---|
| STT (Voz → Texto) | `SpeechRecognition` API do navegador |
| TTS (Texto → Voz) | `SpeechSynthesis` API do navegador |
| LLM | Google Gemini (já configurado) |
| Transporte | HTTP (OpenClaw API) |

**Como funciona:**
1. Usuário fala no microfone
2. Navegador converte voz em texto (STT nativo)
3. Texto é enviado ao OpenClaw via API
4. Resposta é lida em voz alta pelo navegador (TTS nativo)

**Prós:**
- Zero custo absoluto
- Zero infraestrutura extra
- Funciona com o Railway atual sem alterações
- Implementação simples (uma página HTML/JS)

**Contras:**
- Qualidade das vozes depende do sistema operacional e navegador
- Não é conversacional fluido (turn-based: fala → espera → responde)
- Suporte de idiomas limitado em alguns navegadores
- Requer HTTPS para funcionar

---

## Opção 2: Groq Free Tier + Kokoro Web

**Dificuldade:** Média | **Custo:** R$ 0 | **Funciona no Railway:** Sim

Combina o STT ultra-rápido da Groq (Whisper) com o TTS de alta qualidade do Kokoro, ambos gratuitos.

| Componente | Tecnologia |
|---|---|
| STT (Voz → Texto) | Groq Whisper API (free tier) |
| TTS (Texto → Voz) | Kokoro Web (open source, 82M params) |
| LLM | Google Gemini (já configurado) |
| Transporte | HTTP |

**Groq Free Tier:**
- API Whisper rodando em chips LPU (ultra-rápida)
- ~2 horas de transcrição por hora no tier gratuito
- Qualidade idêntica ao Whisper da OpenAI
- Cadastro em [console.groq.com](https://console.groq.com)

**Kokoro TTS:**
- Modelo open-weight com 82 milhões de parâmetros
- Qualidade de voz quase humana
- Disponível via [Kokoro Web](https://github.com/eduardolat/kokoro-web) ou self-hosted
- 50+ vozes disponíveis

**Prós:**
- Qualidade de áudio muito superior à Opção 1
- Latência baixa (Groq é extremamente rápido)
- Funciona no Railway atual
- Vozes naturais e diversas

**Contras:**
- Groq tem rate limits (suficiente para uso pessoal)
- Kokoro Web público pode ter fila de processamento
- Requer chave API da Groq (gratuita)

---

## Opção 3: Pipecat + Speaches + Kokoro (Self-Hosted)

**Dificuldade:** Alta | **Custo API:** R$ 0 | **Funciona no Railway:** Não (precisa GPU)

Stack 100% open source para conversação fluida bidirecional via WebRTC. Baseada no artigo da [WebRTC.ventures (fev/2026)](https://webrtc.ventures/2026/02/building-an-open-source-voice-ai-agent-that-avoids-vendor-lock-in/).

| Componente | Tecnologia |
|---|---|
| Orquestrador | Pipecat (open source, by Daily) |
| STT | Speaches (faster-whisper, Docker) |
| TTS | Kokoro (82M params, open-weight) |
| LLM | Ollama + Llama 3.2 (local) ou Gemini |
| Transporte | WebRTC (via Pipecat) |

**Como funciona:**
1. Usuário conecta via WebRTC no navegador
2. Pipecat roteia o áudio para Speaches (transcrição em tempo real)
3. Texto vai para Ollama/Llama 3.2 ou Gemini
4. Resposta vai para Kokoro (síntese de voz)
5. Áudio retorna via WebRTC — conversa fluida e bidirecional

**Repositório de referência:** [nuxero/voice-ai-landscape-demo](https://github.com/nuxero/voice-ai-landscape-demo)

**Prós:**
- Zero custo de API
- Conversação genuinamente fluida e em tempo real
- Dados 100% sob seu controle
- Qualidade de voz excelente

**Contras:**
- Precisa de servidor com GPU (não roda no Railway)
- Opções: VM com GPU (Oracle Cloud Free Tier, etc.) ou PC local
- Configuração mais complexa (Docker, múltiplos serviços)

---

## Opção 4: LiveKit Self-Hosted

**Dificuldade:** Alta | **Custo API:** R$ 0 | **Funciona no Railway:** Não (precisa servidor)

Infraestrutura WebRTC robusta e open source para comunicação em tempo real com agentes de IA.

| Componente | Tecnologia |
|---|---|
| Infra WebRTC | LiveKit Server (open source) |
| STT | Whisper local ou Groq free tier |
| TTS | Kokoro (self-hosted) |
| LLM | Gemini ou Ollama |
| Agentes | LiveKit Agents framework |

**LiveKit Cloud (alternativa):**
- Free tier: **1.000 minutos grátis** por mês
- 1 agente de voz incluído
- Sem necessidade de self-host para testes

**Repositório:** [livekit/agents](https://github.com/livekit/agents)

**Prós:**
- Infraestrutura WebRTC de produção
- Latência ultra-baixa
- SDK para todas as plataformas (web, mobile, desktop)
- Comunidade ativa e documentação extensa

**Contras:**
- Self-host precisa de servidor dedicado
- Configuração mais complexa
- LiveKit Cloud gratuito tem limite de 1.000 min/mês

---

## Tabela Comparativa

| Critério | Web Speech API | Groq + Kokoro | Pipecat Stack | LiveKit |
|---|---|---|---|---|
| **Custo Total** | R$ 0 | R$ 0 | R$ 0 (APIs) | R$ 0 (APIs) |
| **Dificuldade** | Fácil | Média | Alta | Alta |
| **Qualidade Voz** | Média | Alta | Excelente | Excelente |
| **Railway?** | Sim | Sim | Não (GPU) | Não (servidor) |
| **Tempo Real Fluido?** | Não (turn-based) | Quase | Sim | Sim |
| **Infra Extra?** | Nenhuma | Nenhuma | Servidor com GPU | Servidor |

---

## Recomendação

Para o cenário atual (OpenClaw hospedado no Railway, sem servidor extra):

- **Para começar rápido:** Opção 1 (Web Speech API) — uma página HTML funcional em minutos
- **Melhor custo-benefício:** Opção 2 (Groq + Kokoro) — qualidade profissional sem custo
- **Para escalar no futuro:** Opção 3 ou 4 — quando precisar de conversação realmente fluida

---

## Links Úteis

- [Groq Console (cadastro grátis)](https://console.groq.com)
- [Kokoro Web (TTS grátis)](https://github.com/eduardolat/kokoro-web)
- [Pipecat Framework](https://github.com/pipecat-ai/pipecat)
- [LiveKit Agents](https://github.com/livekit/agents)
- [Speaches ASR](https://speaches.ai/)
- [Artigo WebRTC.ventures — Voice AI Open Source](https://webrtc.ventures/2026/02/building-an-open-source-voice-ai-agent-that-avoids-vendor-lock-in/)
