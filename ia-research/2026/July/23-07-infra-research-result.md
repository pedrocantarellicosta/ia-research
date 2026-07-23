# Novidades Infra / Geral — Semana de 14/07 a 23/07/2026

> **Janela de pesquisa:** 08/07 a 23/07/2026. Itens não repetem o relatório de 13/07 (Cursor 3.11, VS Code 1.128, JetBrains Codex, Codex no ChatGPT desktop, Zed llama.cpp, Kimi K2.7 no Copilot, Claude Cowork nuvem, Vercel Services, Netlify AI Gateway, Base44).

---

## 1. WebStorm 2026.2: "agent skills manager", Copilot nativo, JetBrains AI gratuito e TypeScript 7

### Referências
- [Download WebStorm 2026.2: TypeScript 7 Support, AI, and more (JetBrains Blog)](https://blog.jetbrains.com/webstorm/2026/07/webstorm-2026-2/) — 16/07/2026

### O que é
A JetBrains lançou o **WebStorm 2026.2** em 16/07/2026 com três eixos: (1) um **agent skills manager** que dá aos agentes de IA "conhecimento de stack" reutilizável a cada sessão; (2) **GitHub Copilot nativamente integrado** (sem plugin separado); (3) **JetBrains AI** reorganizado — AI Assistant e Junie sob uma assinatura única, com **recursos de IA gratuitos** nas IDEs (autocompletar ilimitado, suporte a modelos locais, e acesso por créditos a modelos na nuvem). No lado de linguagem, **TypeScript 7** sai da caixa para projetos que já o usam, com type-checking mais rápido.

### Por que isso importa
JetBrains torna IA **gratuita por padrão** na IDE — um movimento competitivo direto contra Cursor/Copilot que muda a economia para quem já vive no WebStorm/IntelliJ. E o "agent skills manager" é o mesmo conceito que aparece no TanStack Intent (front-end) e no ecossistema Claude/Cursor: **conhecimento curado e reutilizável** por sessão, atacando o problema de agentes que "esquecem" o padrão do projeto.

### Benefícios práticos
- IA gratuita (autocompletar ilimitado + modelos locais) sem custo adicional.
- Copilot nativo, sem plugin.
- Skills reutilizáveis dão contexto consistente aos agentes.
- TS 7 acelera type-check em bases grandes sem migração forçada.

### Possíveis problemas ou limitações
Recursos na nuvem seguem por **créditos** — "gratuito" tem teto. Integrar múltiplos provedores (Junie, AI Assistant, Copilot) numa IDE só pode gerar confusão de UX sobre "qual agente estou usando". Suporte a TS 7 depende do estado do projeto.

### Exemplo prático
Fluxo: abrir um monorepo → registrar um skill "convenções de API do time" no agent skills manager → toda sessão de Junie/AI Assistant passa a respeitar o padrão sem re-explicar no prompt.

### Relação com o ecossistema moderno
Alinha JetBrains ao movimento de **skills** (TanStack Intent, Claude, Cursor), a **modelos locais** (privacidade/on-prem) e ao **MCP**. TS 7 conecta-se ao esforço de type-checking em Go (tsgo) que aparece também no Svelte/Angular.

### Vale a pena acompanhar?
**Sim** para times JetBrains — IA gratuita na IDE é mudança material de custo. Para times já em Cursor, é sinal de que a pressão competitiva vai derrubar preços.

---

## 2. Cursor 3.12: planos antes de executar no Slack, ambientes multi-repo e contexto cross-channel

### Referências
- [Cursor Changelog — 3.12](https://releases.sh/cursor/releases) — 17/07/2026
- [Cursor Changelog (July 2026) — Gradually](https://www.gradually.ai/en/changelogs/cursor/) — referência complementar

### O que é
O **Cursor 3.12** (17/07/2026) foca na integração com **Slack**: o agente agora **compartilha um plano antes de começar** a executar, roda em **ambientes multi-repo**, e consegue trabalhar **entre canais e threads** para ganhar mais contexto. É a continuação da linha 3.11 (Side Chats, iOS beta) coberta em 13/07.

### Por que isso importa
"Plano antes de executar" no Slack é governança prática: em vez de um agente autônomo agir direto num repo, o time revisa o plano no canal onde já trabalha. Multi-repo + cross-channel reconhece que features reais atravessam vários serviços e conversas — aproximando o agente do fluxo real de engenharia distribuída.

### Benefícios práticos
- Revisão humana do plano antes da execução (menos surpresas).
- Suporte a mudanças que tocam múltiplos repositórios.
- Contexto puxado de várias threads/canais.
- Trabalho iniciado de onde o time já conversa (Slack).

### Possíveis problemas ou limitações
Agentes disparados do Slack ampliam a superfície de "quem pode mandar o agente mexer no código" — exige controles de permissão claros. Contexto cross-channel levanta questões de **privacidade/escopo** (o agente lê o que?). É incremental sobre o 3.11.

### Relação com o ecossistema moderno
Integra **Slack**, **multi-repo/monorepos** e **CI/CD** (agente que abre PRs). Compete diretamente com o GitHub Copilot coding agent e com o Claude Code no Slack.

### Vale a pena acompanhar?
**Sim**, incremental mas na direção certa (governança). Para quem já usa Cursor+Slack, adoção quase imediata.

---

## 3. Google Antigravity 2.3.0/2.3.1: mensagens em fila, execução configurável e estabilidade

### Referências
- [Google Antigravity — Changelog](https://antigravity.google/changelog) — 2.3.0 em 13/07/2026 e 2.3.1 em 16/07/2026
- [Antigravity Updates by Google — July 2026 (Releasebot)](https://releasebot.io/updates/google/antigravity) — referência complementar

### O que é
A plataforma agent-first do Google recebeu **2.3.0** (13/07/2026) com **mensagens em fila (queued messages)**, configurações para **controlar a execução de mensagens** (com opção "send now"), anexos de arquivos de texto simples e correções; e **2.3.1** (16/07/2026) corrigindo uma falha em que config vazia/malformada impedia o AGY de carregar na inicialização.

### Por que isso importa
Filas de mensagens e controle de execução são recursos de **orquestração multi-agente** — o Antigravity está amadurecendo de "clone de Cursor" para plataforma de orquestração (Antigravity 2.0 desktop + CLI + SDK + IDE). Para quem avalia o ecossistema Google (Gemini) como base de dev agêntico, é o produto a observar.

### Benefícios práticos
- Fila de mensagens para pipelines de agentes sem perder ordem.
- Execução configurável (agendar vs. "send now").
- Correção crítica de boot com config inválida.

### Possíveis problemas ou limitações
Lançamentos quase diários indicam produto ainda em amadurecimento — estabilidade variável. A migração da Gemini CLI para Antigravity CLI (anunciada meses antes) ainda gera atrito de ecossistema. Forte lock-in no stack Google/Gemini.

### Relação com o ecossistema moderno
Compete com **Cursor/Copilot/Zed**; conecta-se a **Gemini 3.6 Flash** (relatório de IA) e ao **MCP**. O SDK expõe o mesmo harness de agente que o Google usa internamente.

### Vale a pena acompanhar?
**Sim, com cautela** — promissor para quem aposta em Gemini; ainda instável para padronizar em produção.

---

## 4. GitHub Copilot no Visual Studio ganha "trust layer" para servidores MCP e agente de modernização C++ em GA

### Referências
- [GitHub Copilot in Visual Studio — June update (GitHub Changelog)](https://github.blog/changelog/2026-07-14-github-copilot-in-visual-studio-june-update/) — 14/07/2026

### O que é
A atualização do **Copilot no Visual Studio** (publicada em 14/07/2026) traz uma **visão mais clara do uso do Copilot**, uma nova **camada de confiança (trust layer) para servidores MCP** e os **primeiros cenários C++ do agente de modernização chegando a GA**.

### Por que isso importa
O "trust layer" para MCP é resposta direta a um problema real de 2026: com milhares de servidores MCP, **confiar cegamente** em ferramentas externas é risco de segurança (prompt injection, exfiltração). Formalizar confiança/aprovação de servidores MCP dentro da IDE é governança que faltava. E o agente de modernização em C++ mostra que a IA de migração sai do JS/Java e entra em bases legadas mais pesadas.

### Benefícios práticos
- Governança de MCP na IDE (aprovar/confiar em servidores).
- Visibilidade de uso do Copilot para gestores.
- Modernização assistida de C++ em GA (bases legadas).

### Possíveis problemas ou limitações
Trust layer ajuda, mas a **responsabilidade de curadoria** continua com o time — uma UI de confiança não elimina o risco de um servidor MCP malicioso aprovado por engano. Cenários C++ ainda são "os primeiros" (cobertura limitada).

### Relação com o ecossistema moderno
Conecta **MCP** (segurança/EMA), **Visual Studio** e o esforço de **modernização de legado** com IA. Casa com o tema de MCP Server Portals (item #8) e com a nova spec MCP (relatório de IA).

### Vale a pena acompanhar?
**Sim**, sobretudo o trust layer de MCP — segurança de MCP é a próxima grande dor corporativa.

---

## 5. Anthropic lança Claude for Teachers — agentes de IA (Claude Code e Cowork) chegam à sala de aula

### Referências
- [Introducing Claude for Teachers (Anthropic)](https://www.anthropic.com/news/claude-for-teachers) — 14/07/2026
- [Anthropic Launches AI For Teachers (Forbes)](https://www.forbes.com/sites/danfitzpatrick/2026/07/14/anthropic-launches-ai-for-teachers/) — 14/07/2026

### O que é
Em 14/07/2026 a Anthropic lançou o **Claude for Teachers**, gratuito para professores verificados de K-12 nos EUA (por pelo menos um ano). Inclui uma biblioteca de "teaching skills" ancoradas em ciência da aprendizagem, conexão a currículos mapeados a padrões dos 50 estados, integração com nove plataformas de ensino, e **acesso às ferramentas agênticas Claude Code e Cowork** (para, por exemplo, analisar dados de turma e automatizar tarefas repetitivas).

### Por que isso importa
Como **case de mercado e de distribuição**, mostra a estratégia de "ganhar o andar de baixo" (educação) e de empacotar **agentes** (Code/Cowork) como ferramentas de produtividade para não-desenvolvedores. É a mesma corrida de OpenAI (ChatGPT for Teachers), Microsoft e Google — sinal de que agentes de propósito geral estão sendo verticalizados agressivamente.

### Benefícios práticos
- Distribui ferramentas agênticas para um público não-técnico.
- Skills de ensino curadas (padrão de qualidade).
- Integração com ecossistema edtech existente.

### Possíveis problemas ou limitações
Críticos apontam riscos de privacidade (dados de alunos), viés e dependência precoce de IA na educação. "Gratuito por um ano" é aquisição de mercado — o custo real vem depois. Fora do escopo técnico de dev, mas relevante como tendência.

### Relação com o ecossistema moderno
Estende **Claude Code/Cowork** para novos verticais; parte da guerra de plataformas de IA em educação.

### Vale a pena acompanhar?
**Sim como tendência** (verticalização de agentes). Não é ferramenta de dev, mas indica para onde o mercado empurra os agentes.

---

## 6. OpenAI abre programa "ChatGPT for Small Business"

### Referências
- [Introducing the ChatGPT for small business program (OpenAI)](https://openai.com/index/introducing-chatgpt-small-business-program/) — 21/07/2026

### O que é
Em 21/07/2026 a OpenAI lançou o **ChatGPT for Small Business**, um programa para ajudar pequenas empresas a serem mais produtivas e escalarem com ChatGPT (onboarding, recursos e provavelmente pacotes/descontos direcionados a SMBs).

### Por que isso importa
É movimento de **go-to-market**: depois de enterprise e educação, a OpenAI ataca o segmento SMB, historicamente carente de suporte. Para o mercado de ferramentas dev, sinaliza commoditização — IA generalista chegando a todo tipo de empresa, o que pressiona nichos e muda expectativas de clientes.

### Benefícios práticos
- Onboarding e recursos específicos para SMB.
- Potencial de padronizar fluxos de IA em empresas pequenas.

### Possíveis problemas ou limitações
Pouco detalhe técnico; é sobretudo distribuição/marketing. Não muda capacidades de modelo. Risco de lock-in de SMBs no ecossistema OpenAI.

### Relação com o ecossistema moderno
Tendência de mercado (adoção massiva de IA). Concorre com pacotes SMB de Google/Microsoft.

### Vale a pena acompanhar?
**Apenas como sinal de mercado.** Baixa relevância técnica direta para times de engenharia.

---

## 7. Modelos GPT-5.6 chegam ao AWS Bedrock e ganham setup de Lambda em um clique

### Referências
- [AWS Weekly Roundup: One-click Lambda setup prompt, OpenAI GPT-5.6 models on Bedrock, and more (AWS Blog)](https://aws.amazon.com/blogs/aws/aws-weekly-roundup-one-click-lambda-setup-prompt-openai-gpt-5-6-models-on-bedrock-and-more-july-20-2026/) — 20/07/2026

### O que é
No roundup de 20/07/2026, a AWS anunciou a disponibilidade dos **modelos GPT-5.6 da OpenAI no Amazon Bedrock** e um **prompt de configuração de Lambda em um clique** (setup guiado por IA). Bedrock passa a servir a família GPT-5.6 ao lado de Claude, Gemini e modelos abertos.

### Por que isso importa
Ter GPT-5.6 no Bedrock significa **multi-cloud de modelos** de verdade: empresas AWS podem usar GPT-5.6 sob governança/rede da AWS, sem sair do Bedrock. Isso reduz lock-in de provider de modelo e simplifica compliance. O setup de Lambda em um clique é IA aplicada à própria plataforma (infra self-service).

### Benefícios práticos
- GPT-5.6 sob IAM/VPC/observabilidade da AWS.
- Menos atrito para padronizar governança de modelos.
- Provisionamento de Lambda assistido por IA.

### Possíveis problemas ou limitações
Modelos de terceiros no Bedrock costumam ter defasagem de features/preço vs. API nativa da OpenAI. Setup "um clique" pode esconder decisões de arquitetura (permissões amplas demais).

### Relação com o ecossistema moderno
**Edge/serverless (Lambda)**, **multi-cloud**, governança corporativa. Complementa AI Gateways (Vercel/Netlify) como forma de rotear modelos.

### Vale a pena acompanhar?
**Sim para times AWS** — disponibilidade de GPT-5.6 no Bedrock é decisão de arquitetura relevante.

---

## 8. Cloudflare lança MCP Server Portals em beta aberto — gateway Zero Trust para todo MCP da organização

### Referências
- [Cloudflare Launches MCP Server Portals — A Unified Gateway to All MCP Servers (Cybersecurity News)](https://cybersecuritynews.com/cloudflare-unveils-mcp-server-portals/) — 14/07/2026

### O que é
Em 14/07/2026 a Cloudflare abriu o beta dos **MCP Server Portals**: um **endpoint único** por onde passam **todas as conexões MCP** de uma organização. Roteando cada requisição MCP por um portal, clientes Cloudflare One aplicam **políticas Zero Trust**, ganham visibilidade completa e reduzem a superfície de ataque exposta por integrações dirigidas por IA.

### Por que isso importa
Conforme MCP explode (10k+ servidores, 200+ implementações oficiais), o problema deixa de ser "conectar" e passa a ser **governar e auditar** o que agentes acessam. Um gateway centralizado com Zero Trust é a peça de infra que faltava para MCP corporativo — a contraparte de rede do "trust layer" que o Copilot no VS (item #4) traz na IDE.

### Benefícios práticos
- Ponto único de política/observabilidade para MCP.
- Zero Trust aplicado a tráfego de agentes.
- Redução de superfície de ataque de integrações IA.

### Possíveis problemas ou limitações
Centralizar tudo cria **ponto único de dependência** (e de falha) na Cloudflare. Beta — maturidade e cobertura ainda em prova. Só faz sentido pleno para quem já é Cloudflare One (lock-in).

### Relação com o ecossistema moderno
Conecta **MCP**, **Zero Trust/edge** e a nova **spec MCP 2026-07-28** (relatório de IA). Casa com o EMA (Enterprise Managed Authorization) do MCP coberto em 13/07.

### Vale a pena acompanhar?
**Sim** — governança de MCP é a dor corporativa emergente de 2026, e portais/gateways são a resposta natural.

---

## 9. Gemini 3.6 Flash chega ao GitHub Copilot no mesmo dia do lançamento

### Referências
- [Gemini 3.6 Flash is now available in GitHub Copilot (GitHub Changelog)](https://github.blog/changelog/2026-07-21-gemini-3-6-flash-is-now-available-in-github-copilot/) — 21/07/2026

### O que é
Em 21/07/2026, dia do anúncio do modelo, o **Gemini 3.6 Flash** ficou disponível no **GitHub Copilot** (planos Pro, Pro+, Max, Business e Enterprise). É o "workhorse" do Google, com contexto de 1M, ~17% menos tokens que o antecessor e cutoff avançado para março/2026.

### Por que isso importa
Disponibilidade **day-one** num assistente de código mainstream mostra o quão rápido o seletor de modelos das ferramentas de dev virou commodity multi-provider. Times passam a escolher modelo por tarefa (custo/latência/qualidade) dentro da mesma IDE, sem trocar de ferramenta.

### Benefícios práticos
- Modelo mais barato e rápido disponível no fluxo Copilot.
- 1M de contexto para tarefas de repо grande.
- Escolha por tarefa dentro do mesmo seletor.

### Possíveis problemas ou limitações
Proliferação de modelos no seletor gera paralisia de escolha e resultados inconsistentes entre eles. Depender de vários providers complica billing e previsibilidade.

### Relação com o ecossistema moderno
Seletor multi-modelo (Copilot/Cursor/Zed), **Gemini API**, e a estratégia agent-first do Google (Antigravity, item #3).

### Vale a pena acompanhar?
**Sim, incremental** — relevante para quem otimiza custo/latência de assistentes de código.

---

## 10. Meta lança sua primeira API paga para desenvolvedores com o Muse Spark 1.1 (agêntico, 1M de contexto)

### Referências
- [Meta Builds AI Revenue Stack: Developer API Launches as Advertiser Automation Goes Global (TechTimes)](https://www.techtimes.com/articles/320609/20260715/meta-builds-ai-revenue-stack-developer-api-launches-advertiser-automation-goes-global.htm) — 15/07/2026

### O que é
Em 15/07/2026 a Meta anunciou o **Muse Spark 1.1**, modelo agêntico com **contexto de 1M**, que estreia junto com a **primeira API paga para desenvolvedores da Meta** (public preview, US$20 em créditos, US-only no lançamento). O modelo oferece **computer use** (desktop, browser e mobile) e **delegação a subagentes em paralelo**.

### Por que isso importa
Marca a entrada da Meta no mercado de **API paga de modelos** — até então focada em pesos abertos (Llama). É mais um fornecedor de peso disputando o stack agêntico (computer use + subagentes), pressionando OpenAI/Google/Anthropic em preço e capacidade. Para times, mais uma opção de modelo agêntico com computer use nativo.

### Benefícios práticos
- Computer use multiplataforma (desktop/browser/mobile) de fábrica.
- Subagentes em paralelo (orquestração nativa).
- 1M de contexto; créditos grátis para testar.

### Possíveis problemas ou limitações
Preview, US-only, ecossistema de tooling ainda imaturo vs. concorrentes estabelecidos. Histórico da Meta em produtos de API para dev é curto (risco de descontinuidade). Computer use amplia riscos de segurança/permite ações amplas.

### Relação com o ecossistema moderno
Concorre com **GPT-5.6/Gemini 3.6/Claude** no stack de agentes; computer use conecta-se ao tema de **Agentic Browsing** e **MCP**.

### Vale a pena acompanhar?
**Ainda cedo**, mas relevante estrategicamente — a Meta virando fornecedora de API paga muda a dinâmica competitiva. Bom para experimentação, não para produção crítica ainda.
