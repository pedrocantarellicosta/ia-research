# Novidades de IA — Semana de 14/07 a 23/07/2026

> **Janela de pesquisa:** 08/07 a 23/07/2026. Itens não repetem o relatório de 13/07 (Grok 4.5, Copilot CLI GPT-5.6, CrewAI Flows, X MCP server, EMA do MCP, família GPT-5.6, GPT-Live-1, Claude API controls, UiPath Coding Agents for Test, Codex Computer Use).

---

## 1. MCP publica a maior revisão desde o lançamento: spec 2026-07-28 (RC) torna o protocolo stateless

### Referências
- [AI's most important protocol is getting a little bit easier to use (TechCrunch)](https://techcrunch.com/2026/07/20/ais-most-important-protocol-is-getting-a-little-bit-easier-to-use/) — 20/07/2026
- [The 2026-07-28 MCP Specification Release Candidate (MCP Blog)](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) — RC anunciado na semana de 20/07/2026

### O que é
A especificação **MCP 2026-07-28** — cujo **release candidate** foi divulgado nesta janela (TechCrunch em 20/07) — é a maior revisão do protocolo desde o lançamento. Ela **remove o handshake `initialize` e a sessão de nível de protocolo**, tornando o core **stateless**: cada requisição é autocontida, carregando versão do protocolo, client info e capabilities em metadados. Adiciona novos padrões de request (multi-round-trip), **headers roteáveis** (`Mcp-Method`), endurecimento de autorização e uma política formal de deprecação. Clientes que falam 2026-07-28 fazem fallback para o handshake antigo ao encontrar servidores mais velhos.

### Por que isso importa
Sessões com estado eram o maior obstáculo para **escalar servidores MCP**: exigiam sticky sessions, store de sessão compartilhado e inspeção profunda no gateway. Stateless permite rodar MCP atrás de um **round-robin comum**, rotear por header e **cachear `tools/list`** conforme o `ttlMs`. Na prática, isso alinha MCP ao modo como sites convencionais escalam — reduzindo custo e complexidade operacional e destravando adoção corporativa.

### Benefícios práticos
- Servidores MCP sem estado → escala horizontal trivial (LB comum).
- Roteamento por `Mcp-Method`; cache de `tools/list`.
- Autorização mais forte e política de deprecação formal.
- Compatibilidade com fallback para clientes/servidores antigos.

### Possíveis problemas ou limitações
Migração não é trivial: servidores que dependiam de estado de sessão precisam repensar a arquitetura (mover estado para tokens/DB). Multi-round-trip e novos headers ampliam a **superfície de segurança** (SecurityWeek já alertou para novos desafios). Por ser RC, ainda pode mudar antes do final em 28/07.

### Exemplo prático
```
# Antes (stateful): initialize -> sessionId -> toda request precisa do sessionId
# Depois (stateless): cada request carrega tudo em metadata
POST /mcp
Mcp-Method: tools/call
{ "protocolVersion": "2026-07-28", "clientInfo": {...}, "params": {...} }
```

### Relação com o ecossistema moderno
Base para **MCP Server Portals** da Cloudflare (relatório de infra), para o **trust layer** do Copilot no VS e para todo o edge/serverless que hospeda MCP. Afeta AI SDK, Claude, Cursor, Copilot — todos consumidores de MCP.

### Vale a pena acompanhar?
**Sim, criticamente** — é infraestrutura de base para todo o ecossistema de agentes. Times com servidores MCP em produção devem planejar a migração agora.

---

## 2. SDKs oficiais do MCP ganham betas v2: TypeScript ESM-only e Python renomeia FastMCP para MCPServer

### Referências
- [The 2026-07-28 MCP Specification Release Candidate (MCP Blog)](https://blog.modelcontextprotocol.io/posts/2026-07-28-release-candidate/) — semana de 20/07/2026
- [2026-07-28 MCP: stateless, multi-round-trip, routable headers, authorization hardening (4sysops)](https://4sysops.com/archives/2026-07-28-model-context-protocol-mcp-stateless-multi-round-trip-routable-headers-authorization-hardening/) — referência complementar

### O que é
Acompanhando o RC da spec, saíram **betas dos SDKs** oficiais: **Python v2** renomeia `FastMCP` para `MCPServer`; **TypeScript v2** se divide em pacotes focados e passa a ser **ESM-only**; **Go** ganha suporte à 2026-07-28 no `v1.7.0-pre.1`; **C#** sai como `2.0.0-preview.1`.

### Por que isso importa
As bibliotecas que a maioria dos devs usa para **escrever** servidores/clientes MCP estão sendo reescritas para o modelo stateless. TS ESM-only e a divisão em pacotes focados são breaking changes que exigem atenção — mas também modernizam a base (tree-shaking, pacotes menores). É o momento de re-arquitetar servidores MCP.

### Benefícios práticos
- APIs alinhadas ao core stateless.
- TS: pacotes menores, ESM-only, melhor tree-shaking.
- Suporte multi-linguagem coordenado (Python/TS/Go/C#).

### Possíveis problemas ou limitações
Breaking changes em massa: `FastMCP`→`MCPServer`, ESM-only (quebra CommonJS), split de pacotes. Betas — não usar em produção ainda. Ecossistema de exemplos/tutoriais vai levar tempo para atualizar.

### Exemplo prático
```python
# Python v2 (beta)
from mcp import MCPServer   # antes: from fastmcp import FastMCP
server = MCPServer("meu-servidor")

@server.tool()
def buscar(q: str) -> list[str]: ...
```

### Relação com o ecossistema moderno
Núcleo de qualquer integração **MCP** (Claude, Cursor, Copilot, ChatGPT). ESM-only conecta-se ao movimento geral (React Router 8, TS 7) de abandonar CommonJS.

### Vale a pena acompanhar?
**Sim** para quem mantém servidores MCP. Ainda em beta — teste, mas não migre produção antes do estável (28/07).

---

## 3. OpenAI lança Presence: plataforma enterprise para agentes de IA de voz e chat com governança

### Referências
- [OpenAI unveils Presence... realtime voice agents and chatbots (VentureBeat)](https://venturebeat.com/orchestration/openai-unveils-presence-a-new-platform-that-lets-enterprises-launch-and-manage-realtime-voice-agents-and-chatbots) — 22/07/2026
- [OpenAI Presence connects AI agents to enterprise data with built-in guardrails (Help Net Security)](https://www.helpnetsecurity.com/2026/07/22/openai-presence-ai-agent-platform/) — 22/07/2026

### O que é
Em 22/07/2026 a OpenAI apresentou o **Presence**, plataforma gerenciada para **construir, governar e melhorar continuamente** agentes de produção em **voz e chat**. Reúne políticas/SOPs, **guardrails**, ações aprovadas, **simulações**, ferramentas de avaliação (evals) e um **loop de melhoria via Codex** (o Codex analisa interações de produção e sugere melhorias, revisadas/aprovadas por humanos antes de irem ao ar). A OpenAI diz que o Presence já opera sua própria linha de suporte telefônico em inglês, resolvendo 75% das chamadas sem humano.

### Por que isso importa
É a virada da OpenAI de "acesso a modelo" para **sistema gerenciado de agentes de produção** — competindo com Presence/AgentForce-like da concorrência. O diferencial é o **ciclo fechado**: produção → evals → Codex propõe melhoria → humano aprova → deploy. Isso ataca o maior problema de agentes em produção: **melhorar com segurança** sem regressões.

### Benefícios práticos
- Guardrails, políticas e ações aprovadas de fábrica.
- Simulações e evals antes de ir a produção.
- Loop de melhoria contínua dirigido por Codex, com human-in-the-loop.
- Voz + chat na mesma plataforma.

### Possíveis problemas ou limitações
GA limitada, via **Forward Deployed Engineers** e integradores — ou seja, não é self-service ainda (alto custo/toque). Forte lock-in OpenAI. "Codex sugere melhorias" exige governança rigorosa para não introduzir regressões sutis. Métrica de "75% resolvido" é autorreportada.

### Relação com o ecossistema moderno
Compete com plataformas de agentes (Google Gemini Managed Agents, item #8; AG-UI/CopilotKit no front-end). Conecta **Codex**, **evals** e **guardrails** — o "AgentOps" da OpenAI.

### Vale a pena acompanhar?
**Promissor para empresas** com suporte/atendimento em escala. Para times menores, ainda inacessível (GA restrita).

---

## 4. Google lança Gemini 3.6 Flash, 3.5 Flash-Lite e o especializado 3.5 Flash Cyber

### Referências
- [Introducing Gemini 3.6 Flash, 3.5 Flash-Lite, and 3.5 Flash Cyber (Google Blog)](https://blog.google/innovation-and-ai/models-and-research/gemini-models/gemini-3-6-flash-3-5-flash-lite-3-5-flash-cyber/) — 21/07/2026
- [Google releases three new Gemini models — but no 3.5 Pro (TechCrunch)](https://techcrunch.com/2026/07/21/google-releases-three-new-gemini-models-but-no-3-5-pro/) — 21/07/2026

### O que é
Em 21/07/2026 o Google DeepMind lançou três modelos: **Gemini 3.6 Flash** (workhorse, US$1,50/US$7,50 por 1M tokens, contexto 1M, cutoff março/2026, ~17% menos tokens), **Gemini 3.5 Flash-Lite** (mais barato da classe) e **Gemini 3.5 Flash Cyber** — um modelo **fine-tunado para encontrar e corrigir vulnerabilidades de cibersegurança**, disponível só para governos e parceiros de confiança em piloto de acesso limitado.

### Por que isso importa
Flash 3.6 melhora coding/multimodal reduzindo custo — direto no fluxo de dev (já entrou no Copilot day-one). Mas o item mais estratégico é o **Flash Cyber**: um modelo dedicado a **segurança ofensiva/defensiva** sob acesso restrito, sinalizando a especialização de modelos por domínio sensível e o cuidado (acesso gated) que isso exige.

### Benefícios práticos
- Flash 3.6: mais barato, 1M contexto, forte em coding.
- Flash-Lite: menor custo para tarefas simples/alto volume.
- Flash Cyber: achar/corrigir vulnerabilidades (para quem tem acesso).

### Possíveis problemas ou limitações
Ausência de um "3.5 Pro" deixa lacuna no topo de raciocínio. Modelos de cyber gated levantam debate de dual-use (mesma capacidade serve ataque e defesa). Proliferação de variantes complica escolha.

### Relação com o ecossistema moderno
Entra em **GitHub Copilot** (infra item #9), **Gemini API/AI Studio**, **Antigravity**. Flash Cyber conecta-se ao tema de **IA em AppSec** e à spec MCP (superfície de segurança).

### Vale a pena acompanhar?
**Sim** — Flash 3.6 é escolha prática de custo/latência; Flash Cyber é tendência a observar (modelos de domínio sensível com acesso controlado).

---

## 5. Google provoca com o "Gemini 4" e reforça a cadência acelerada de modelos

### Referências
- [Google Just Teased Its Huge Gemini 4 Release (Droid Life)](https://www.droid-life.com/2026/07/21/google-drops-gemini-flash-3-6-on-us-teases-gemini-4/) — 21/07/2026
- [Google launches Gemini 3.6 Flash and 3.5 Flash-Lite, teases Gemini 4 (9to5Google)](https://9to5google.com/2026/07/21/gemini-3-6-flash-launch/) — 21/07/2026

### O que é
Junto ao lançamento dos Flash (21/07), o Google **provocou publicamente o Gemini 4**, sinalizando um salto geracional próximo enquanto ainda entrega incrementos na linha 3.x.

### Por que isso importa
Como **tendência de mercado**, mostra a cadência de "teasing" para segurar mindshare contra GPT-5.6 e Claude 5. Para times, é sinal de planejamento: evitar acoplar arquitetura a peculiaridades de uma geração específica, porque a próxima vem rápido.

### Benefícios práticos
- Sinaliza roadmap (planejamento de adoção).
- Pressão competitiva tende a melhorar preço/qualidade.

### Possíveis problemas ou limitações
É **teaser**, não produto — sem specs, datas firmes ou preço. Hype puro se tomado como base de decisão.

### Relação com o ecossistema moderno
Guerra de modelos (OpenAI/Google/Anthropic/Meta); afeta seletores multi-modelo de todas as ferramentas de dev.

### Vale a pena acompanhar?
**Apenas como sinal.** Não decida arquitetura com base em teaser.

---

## 6. Anthropic evolui o Claude Developer Platform: memória de agente (beta) e overrides de Managed Agents

### Referências
- [Claude Developer Platform Updates by Anthropic — July 2026 (Releasebot)](https://releasebot.io/updates/anthropic/claude-developer-platform) — atualizações de 22/07/2026
- [Anthropic Release Notes — July 2026 (Releasebot)](https://releasebot.io/updates/anthropic) — referência complementar

### O que é
O **Claude Developer Platform** ganhou nesta janela: (1) o header beta **`agent-memory-2026-07-22`**, que muda o comportamento de **listagem de memórias** com ordem estável definida pelo servidor e controle mais fino de `depth`, `path_prefix` e `cursor`; (2) na criação de sessão de **Managed Agents**, a possibilidade de **sobrescrever a config do agente por sessão** (`type: "agent_with_overrides"`) — trocando modelo, system prompt, tools, servidores MCP ou skills; (3) tool calls de MCP acima de 2 minutos passam a rodar em **background** automaticamente (`CLAUDE_CODE_MCP_AUTO_BACKGROUND_MS`).

### Por que isso importa
Memória de agente com **ordem estável e paginação previsível** é requisito para construir agentes que acumulam contexto de forma auditável (não o "resumo diário" difuso). Overrides por sessão dão flexibilidade sem duplicar definições de agente. E auto-background de tool calls longas evita travar a sessão — DX prático para agentes que chamam ferramentas lentas.

### Benefícios práticos
- API de memória de agente com listagem estável/paginável.
- Reuso de um agente base com overrides por sessão (modelo, tools, MCP, skills).
- Tool calls longas não bloqueiam a sessão.

### Possíveis problemas ou limitações
`agent-memory` é **beta** (comportamento pode mudar). Overrides por sessão, se mal governados, viram fonte de inconsistência ("por que esta sessão usou outro modelo?"). Auto-background muda semântica de execução — precisa de tratamento de resultado assíncrono.

### Exemplo prático
```json
// Criar sessão de Managed Agent com override
{
  "agent": {
    "type": "agent_with_overrides",
    "model": "claude-opus-4-8",
    "mcp_servers": ["playwright", "github"],
    "skills": ["convencoes-api-do-time"]
  }
}
```

### Relação com o ecossistema moderno
Núcleo de **AgentOps** com Claude; conecta **MCP**, **skills** (mesmo conceito do TanStack Intent/WebStorm) e memória persistente de agentes.

### Vale a pena acompanhar?
**Sim** para quem constrói agentes sobre a API da Anthropic. Memória e overrides são exatamente as primitivas que faltavam para produção.

---

## 7. Google adiciona Managed Agents à Gemini API: o harness do Antigravity como serviço gerenciado

### Referências
- [All the news from the Google I/O 2026 Developer keynote / Managed Agents in the Gemini API (Google Developers Blog)](https://developers.googleblog.com/all-the-news-from-the-google-io-2026-developer-keynote/) — cobertura do lançamento junto ao Gemini 3.6 (21/07/2026)

### O que é
O Google passou a oferecer **Managed Agents na Gemini API**, removendo a fricção de montar infraestrutura de agente: entrega o **harness de agente do Antigravity** (o mesmo usado internamente) como **serviço gerenciado**. Junto, apresentou o **Gemini 3.1 Flash TTS** em preview para voz.

### Por que isso importa
É o espelho da OpenAI Presence e da Anthropic Managed Agents: os três grandes agora oferecem **runtime de agente gerenciado**, não só o modelo. Para times, significa poder subir agentes de produção (loop, tools, orquestração) sem operar a infra — ao custo de lock-in no harness do fornecedor.

### Benefícios práticos
- Runtime de agente pronto (sem montar orquestração própria).
- Mesma tecnologia interna do Google (Antigravity harness).
- TTS de nova geração para agentes de voz.

### Possíveis problemas ou limitações
Lock-in no harness Google. "Managed" esconde detalhes de execução/custo. Preview para partes (TTS). Concorre num espaço lotado (Presence, Anthropic, AG-UI).

### Relação com o ecossistema moderno
**Gemini API/AI Studio**, **Antigravity** (infra item #3), **MCP**. Parte da tríade de "agentes gerenciados" de 2026.

### Vale a pena acompanhar?
**Sim para quem já está no Gemini** — reduz muito o esforço de subir agentes. Avalie o lock-in antes de padronizar.

---

## 8. Tricentis leva Agentic Test Automation ao SAP — IA aplicada a testes em transformações enterprise

### Referências
- [A weekly round-up of product launches and company news (QA Financial)](https://qa-financial.com/a-weekly-round-up-of-product-launches-and-company-news-3/) — semana de 20/07/2026

### O que é
A **Tricentis** anunciou nova funcionalidade de **teste agêntico** para programas de transformação de negócio em **SAP**, trazendo o **Tricentis Agentic Test Automation** para o **SAP Enterprise Continuous Testing**. Na prática, agentes exploram, geram e mantêm testes de processos SAP end-to-end.

### Por que isso importa
Mostra que **IA aplicada a testes** saiu do front-end web (Playwright Test Agents, self-healing) e entrou nas **suites enterprise mais caras e frágeis** (SAP), onde manutenção de teste consome 40–60% do tempo de QA. Testes agênticos que se auto-reparam em ERPs complexos são um dos usos de IA com ROI mais direto.

### Benefícios práticos
- Geração e manutenção automática de testes SAP (menos toil de QA).
- Continuous testing agêntico integrado ao ecossistema SAP.
- Redução de manutenção (self-healing) em processos que mudam muito.

### Possíveis problemas ou limitações
Ambiente enterprise/SAP = alto acoplamento e custo de licença (lock-in Tricentis+SAP). Testes gerados por IA em processos críticos exigem validação humana rigorosa (falso "verde" é perigoso em ERP). Fora do escopo web/front-end puro.

### Relação com o ecossistema moderno
Conecta **IA de testes** (Playwright Test Agents, `@shadcn/helpers` no front) ao mundo **enterprise/ERP**. Parte da onda de "Agentic Software Quality" mapeada pelo Gartner.

### Vale a pena acompanhar?
**Sim para quem opera SAP/enterprise.** Para times web, é sinal de que testes agênticos viraram baseline em todo o espectro de QA.

---

## 9. AWS lança CloudWatch Coding Agent Insights: observabilidade de agentes de código na organização

### Referências
- [AI Agents News — Week of July 21, 2026 (AI Agent Store)](https://aiagentstore.ai/ai-agent-news/this-week) — 20/07/2026

### O que é
Em 20/07/2026 a **Amazon CloudWatch** lançou o **Coding Agent Insights**, que mostra a líderes de engenharia **como as ferramentas de IA de código estão performando** na organização. Coleta telemetria do **Claude Code** (via gateway das apps Claude) e suporta também **Codex** e **GitHub Copilot**.

### Por que isso importa
É **observabilidade para AgentOps**: à medida que times adotam múltiplos agentes de código, gestores precisam medir uso, custo e impacto real (não só "instalamos"). Consolidar telemetria de Claude Code/Codex/Copilot no CloudWatch dá visão cross-tool — a mesma lógica de APM aplicada a agentes.

### Benefícios práticos
- Visão unificada de desempenho de agentes de código.
- Telemetria de múltiplos fornecedores (Claude, Codex, Copilot).
- Base para decisões de adoção/custo por dados, não achismo.

### Possíveis problemas ou limitações
Telemetria de agentes de código levanta questões de **privacidade/consentimento** dos devs (o que é medido?). Métricas de "produtividade de IA" são notoriamente difíceis (linhas != valor). Depende de gateways/integrações específicas.

### Relação com o ecossistema moderno
Conecta **observabilidade** (CloudWatch) a **agentes de código** (Claude Code/Codex/Copilot). Complementa o "uso do Copilot" que o VS/GitHub expõe (infra item #4).

### Vale a pena acompanhar?
**Sim para líderes de engenharia** que já rodam múltiplos agentes e precisam de dados. Cuidado com métricas de vaidade e privacidade.

---

## 10. Cloudflare formaliza o gateway de MCP com os Server Portals — governança de agentes como infra de IA

### Referências
- [Cloudflare Launches MCP Server Portals — A Unified Gateway to All MCP Servers (Cybersecurity News)](https://cybersecuritynews.com/cloudflare-unveils-mcp-server-portals/) — 14/07/2026

### O que é
> Também listado no relatório de Infra (item #8) pelo ângulo de rede/Zero Trust; aqui pelo ângulo de **arquitetura de agentes/MCP**.

Os **MCP Server Portals** (beta aberto, 14/07/2026) dão um **plano de controle único** para todas as conexões MCP de uma organização: descoberta, política, autorização e auditoria centralizadas. É a peça de **governança** que faltava para agentes que consomem dezenas de servidores MCP.

### Por que isso importa
Com a spec MCP virando **stateless** (item #1) e o número de servidores explodindo, o gargalo é **governar quem/o quê** os agentes acessam. Um portal centralizado transforma MCP de "N conexões soltas" em "um endpoint governado" — pré-requisito para agentes corporativos auditáveis.

### Benefícios práticos
- Descoberta e política central de servidores MCP.
- Autorização e auditoria unificadas para tráfego de agentes.
- Menor superfície de ataque (um ponto de controle).

### Possíveis problemas ou limitações
Ponto único de dependência (Cloudflare One). Beta. Centralização pode virar gargalo se mal dimensionada.

### Relação com o ecossistema moderno
**MCP** (spec 2026-07-28), **Zero Trust/edge**, **EMA do MCP** (13/07), trust layer do Copilot (infra #4). Núcleo do "AgentOps de segurança".

### Vale a pena acompanhar?
**Sim** — governança de MCP é a fronteira de 2026 para agentes em empresas.
