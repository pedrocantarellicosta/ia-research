# ⚙️ Infra & Geral AI Weekly Research — 01/07/2026

> 10 novidades de infraestrutura, ferramentas e updates de plataformas AI

---

## 1. OpenCode — O Agente de Coding Open-Source com 160K Stars e 7.5M Devs/Mês

### O que é

O OpenCode é um agente de coding open-source model-agnóstico que roda no terminal, desktop e como extensão de IDE. Com mais de 160.000 estrelas no GitHub e 7.5 milhões de desenvolvedores ativos por mês, ele se tornou a alternativa de código aberto para Cursor e Claude Code. Suporta 75+ providers de IA (Claude, GPT, Gemini, DeepSeek, modelos locais via Ollama), integração nativa com Language Server Protocol (LSP) e execução air-gapped completa.

### Por que isso importa

A principal diferença do OpenCode é o LSP integration: o agente não apenas lê texto, ele vê o código através do compilador — tipos reais, diagnósticos ao vivo, assinaturas de função, caminhos de import. Isso reduz alucinações em TypeScript significativamente. Para times em indústrias reguladas (fintech, saúde), o modo air-gapped permite uso de IA sem nenhum dado sair da máquina.

### Benefícios práticos

- LSP integration: type information, function signatures e live compiler diagnostics alimentam o modelo em tempo real.
- Multi-agent: subagentes paralelos — um planejando, outro implementando simultaneamente.
- MCP nativo: conecta com Figma, Storybook, bases de dados e qualquer servidor MCP.
- Air-gapped via Ollama/LM Studio: loop agentico completo sem chamadas externas de API.
- Suporte a 18+ linguagens via LSP.

### Possíveis problemas ou limitações

- A qualidade das sugestões depende do provider escolhido — com modelos locais pequenos, a qualidade cai significativamente.
- O TUI (Terminal UI) tem curva de aprendizado maior que interfaces visuais como Cursor.
- A extensão de IDE ainda está em maturação — menos features que Cursor.

### Exemplo prático

```bash
# Instalar OpenCode
npm i -g opencode-ai

# Iniciar com Claude como provider
opencode --model anthropic/claude-sonnet-4-6

# Iniciar com modelo local (air-gapped)
opencode --model ollama/llama3.3

# Atalho no VS Code/Cursor: Cmd+Esc (Mac) ou Ctrl+Esc (Windows/Linux)
```

### Relação com o ecossistema moderno

- **VS Code/Cursor**: extensão disponível no marketplace.
- **MCP**: suporte nativo a qualquer servidor MCP.
- **CI/CD**: pode rodar em pipelines para tarefas de análise automática.
- **Monorepos**: multi-session permite trabalhar em múltiplos packages simultaneamente.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — é a melhor alternativa open-source para Claude Code hoje, especialmente para times com necessidades de privacidade.

---

## 2. Kiro — IDE Spec-Driven da AWS com Planejamento Antes do Código

### O que é

O Kiro é a IDE agentica da Amazon Web Services, construída sobre Code OSS (a base open-source do VS Code), powered by Claude via Amazon Bedrock. Seu diferencial é o **spec-driven development**: antes de gerar qualquer código, o agente produz um documento estruturado com requisitos, plano de arquitetura e especificação de testes. O **hooks system** dispara quality gates automáticos (lint, test, docs) em cada save de arquivo.

### Por que isso importa

A maioria dos agentes de coding atua de forma reativa: você descreve o que quer e ele gera. O Kiro inverte: ele *planeja* antes de agir, reduzindo o ciclo de correção. Para times de produto, isso significa menos back-and-forth para chegar num estado de código que faz sentido arquiteturalmente.

### Benefícios práticos

- Spec-driven: o agente gera requisitos → arquitetura → testes spec antes de qualquer linha de código.
- Hooks automáticos: lint + test + doc generation em cada save — sem esquecer quality gates.
- AWS Transform integrado: modernização de codebases legados (refactoring, migrations) diretamente na IDE.
- Importa settings, themes e extensões Open VSX do VS Code — migração sem atrito.
- Gratuito durante período de preview.

### Possíveis problemas ou limitações

- Lock-in com AWS/Bedrock — não suporta providers externos com a mesma profundidade.
- O spec-driven approach adiciona tempo inicial — pode ser overhead para protótipos rápidos.
- A extensão Open VSX ainda não tem paridade completa com o marketplace oficial do VS Code.

### Exemplo prático

```
// Fluxo Kiro para uma nova feature:

1. Developer descreve: "Adicionar autenticação OAuth com GitHub"

2. Kiro gera spec:
   - Requirements: lista de critérios de aceite
   - Architecture: diagrama de componentes afetados
   - Test spec: casos de teste a serem implementados

3. Developer aprova/ajusta o spec

4. Kiro implementa com os specs como guardrail

5. Hooks automáticos: eslint → vitest → doc update
```

### Relação com o ecossistema moderno

- **VS Code**: mesma base — migração com importação de config.
- **AWS**: integração nativa com serviços AWS (S3, Lambda, RDS) via AWS Transform.
- **CI/CD**: hooks podem ser mapeados para actions similares no pipeline.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — o spec-driven approach é uma evolução real em como trabalhar com agentes de IA.

---

## 3. GitHub Copilot — Migração para Créditos por Uso e Claude Sonnet 5

### O que é

Em 1 de junho de 2026, o GitHub Copilot migrou para um modelo de **usage-based AI credits**: 1 crédito = $0.01, sem mais "premium requests" como categoria separada. Além disso, a versão CLI v1.0.67 (lançada em 30 de junho) adicionou **Claude Sonnet 5** como nova opção de modelo e corrigiu os sandbox controls para funcionarem instantaneamente sem interrupções mid-command.

### Por que isso importa

A mudança para créditos por uso é estrutural: times antes limitados pelos planos de "premium requests" agora podem usar Claude Sonnet 5 e outros modelos avançados sem upgrade de plano. O tradeoff é previsibilidade de custo — equipes com uso intensivo precisam monitorar consumo de créditos.

### Benefícios práticos

- Acesso a Claude Sonnet 5 sem plano especial — mais capacidade para raciocínio complexo.
- Sandbox controls instantâneos: sem mais interrupções ao executar comandos longos.
- Model switching no Copilot Chat: pode alternar entre GPT-4o, Claude Sonnet 5, Gemini 2.0 Flash por task.
- Copilot Code Review com sugestões inline em PRs — agora com modelos mais novos.

### Possíveis problemas ou limitações

- Modelo de créditos introduz variabilidade de custo — teams devem configurar limites de gasto.
- A transição foi em 1 de junho e novos sign-ups pagos estavam pausados durante o rollout.
- A qualidade de geração varia significativamente entre modelos — não há um default universal correto.

### Exemplo prático

```bash
# GitHub Copilot CLI v1.0.67 com Claude Sonnet 5
gh copilot suggest "Create a Vue 3 composable for infinite scroll" \
  --model claude-sonnet-5 \
  --target shell
```

### Relação com o ecossistema moderno

- **VS Code**: Copilot ainda é a extensão mais instalada no marketplace com 20M+ usuários.
- **GitHub**: Copilot Code Review integrado nativamente em PRs do GitHub.com.
- **CI/CD**: Copilot pode ser invocado como agent em GitHub Actions.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — a transição para créditos e a adição de Claude Sonnet 5 mudam a equação custo/benefício.

---

## 4. Antigravity CLI — Gemini CLI Absorvido, Nova Plataforma Unificada

### O que é

Em 18 de junho de 2026, o Google anunciou que o **Gemini CLI está sendo absorvido pelo Antigravity CLI**. Para usuários individuais, o Gemini CLI foi descontinuado nessa data — as funcionalidades migram para o Antigravity CLI. Endpoints enterprise do Vertex AI não são afetados. O Antigravity CLI foi posicionado como a plataforma unificada de Google AI para desenvolvedores, integrando modelos, ferramentas e pipelines.

### Por que isso importa

Times que usam Gemini CLI em scripts, CI/CD e automações precisam migrar antes que o suporte cesse completamente. Além disso, o Antigravity CLI expande além de simples completions — oferece execução de agentes, acesso a ferramentas Google Cloud e suporte ao protocolo A2A.

### Benefícios práticos

- Unificação: um CLI para todos os produtos Google AI — Gemini, ADK, A2A.
- Suporte ao protocolo A2A: agentes podem se comunicar com outros agentes via CLI.
- Integração com Google Cloud: acesso a BigQuery, Cloud Storage, Vertex AI diretamente.
- Retrocompatibilidade: os subcomandos principais do Gemini CLI são suportados em Antigravity.

### Possíveis problemas ou limitações

- Docs e install scripts em CI precisam ser atualizados — referências ao `gemini` CLI quebram após a migração.
- A documentação do Antigravity CLI ainda estava incompleta na data da transição.
- Usuários do Gemini CLI em ambientes air-gapped precisam avaliar as novas dependências.

### Exemplo prático

```bash
# Antes (Gemini CLI — descontinuado em 18/06/2026)
gemini --model gemini-2.0-flash "Explain this component"

# Depois (Antigravity CLI)
antigravity run --model gemini-2.0-flash "Explain this component"

# Atualizar CI/CD
# .github/workflows/ai-review.yml
- name: AI Code Review
  run: |
    npm install -g @google/antigravity-cli
    antigravity review --diff ${{ github.event.pull_request.diff_url }}
```

### Relação com o ecossistema moderno

- **CI/CD**: atualizar pipelines que usavam `gemini` CLI antes de setembro de 2026.
- **VS Code**: extensão Gemini Code Assist migra automaticamente para Antigravity.
- **Vertex AI**: enterprise endpoints seguem funcionando — a mudança é no DX surface.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — especialmente para times que usam Gemini CLI em automações. Migração é urgente.

---

## 5. Claude Code — Fable 5, Dynamic Workflows e 1M Token Context

### O que é

O Claude Code com Fable 5 (entrada no Terminal-Bench 2.1 em 17 de junho) atingiu **83.1% no Terminal-Bench 2.1** e **95.0% no SWE-bench Verified** — o melhor resultado em benchmarks de engenharia de software. O **Dynamic Workflows** permite que trabalhos grandes sejam divididos automaticamente em subagentes paralelos. A janela de contexto de **1M tokens** (em beta) permite analisar codebases inteiros sem chunking manual.

### Por que isso importa

Esses números não são só marketing: 95% no SWE-bench significa que o modelo resolve a maioria dos issues reais de GitHub sem intervenção humana. Para times de front-end, isso se traduz em: refatorações arquiteturais completas, migrações de framework e debugging de bugs difíceis de forma mais autônoma.

### Benefícios práticos

- 1M token context: analisa monorepos inteiros em uma única conversa.
- Dynamic Workflows: divide tarefas complexas em subagentes paralelos automaticamente.
- SWE-bench 95%: melhor resolução de issues reais de engenharia de software.
- Integração com MCP: acessa Figma, Storybook, banco de dados e APIs externas nativamente.
- Fable 5 lidera SWE-bench Pro (80.3%) — benchmark mais difícil com issues ambíguos.

### Possíveis problemas ou limitações

- Fable 5 e Mythos 5 estão com export suspenso desde 12 de junho — disponíveis apenas em produtos Anthropic.
- Contexto de 1M tokens ainda em beta — pode ter inconsistências em janelas muito longas.
- Dynamic Workflows tem custo de tokens proporcional ao número de subagentes.

### Exemplo prático

```bash
# Claude Code com contexto de 1M tokens para análise de monorepo
claude --context-window 1m \
  "Analise todos os componentes em packages/ e identifique:
   1. Componentes com props não utilizadas
   2. Imports circulares
   3. Componentes duplicados entre apps
   Gere um relatório com arquivo e linha para cada item."
```

### Relação com o ecossistema moderno

- **VS Code/Cursor**: Claude Code funciona como agente em ambos via extensão.
- **MCP**: conecta com qualquer servidor MCP para contexto adicional.
- **GitHub**: pode ser usado como CI agent em GitHub Actions.
- **Turborepo**: Dynamic Workflows se alinha com a filosofia de tarefas paralelas do Turborepo.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — Fable 5 representa um salto real de capability, não apenas um update incremental.

---

## 6. New Relic Now — June 2026 e Observabilidade com AI Ops

### O que é

O New Relic anunciou em junho de 2026 um conjunto de features de AI Ops que integram LLMs diretamente na plataforma de observabilidade. O principal é o **AI Assistant for Incident Response**: ao invés de um humano analisar logs e métricas durante um incidente, o assistente correlaciona automaticamente alertas, identifica a causa raiz provável e sugere runbooks de remediação.

### Por que isso importa

Observabilidade front-end está evoluindo de dashboards passivos para sistemas proativos. O New Relic com AI Ops consegue correlacionar uma degradação de INP em produção com um deploy específico, um componente específico e uma mudança de código específica — em segundos, não em minutos de análise manual.

### Benefícios práticos

- Correlação automática entre alertas de múltiplas fontes (APM, browser, infrastructure).
- Root cause analysis em linguagem natural — sem precisar escrever NRQL.
- Sugestão de runbooks baseada em incidentes históricos similares.
- Integração com BigQuery Agent Analytics do Google Cloud para análise de traces de agentes AI.

### Possíveis problemas ou limitações

- O AI Assistant usa os dados da conta New Relic — qualidade da análise depende da instrumentação existente.
- Custo adicional sobre o plano New Relic base.
- Root cause suggestions ainda erram em sistemas distribuídos complexos com muitas variáveis.

### Exemplo prático

```
// Fluxo de AI Ops no New Relic:

Alerta: "INP degradou 85% nos últimos 15 minutos em Chrome/Android"

AI Assistant analisa:
1. Correlaciona com deploy das 14:32 (commit abc123)
2. Identifica: ProductGallery.vue adicionou lazy loading com IntersectionObserver
3. Detecta: o observer está bloqueando a main thread em dispositivos com GPU limitada
4. Sugere: mover para Web Worker ou usar requestIdleCallback
5. Aponta runbook: "Performance Degradation - Interaction Metrics"
```

### Relação com o ecossistema moderno

- **Nuxt/Next.js**: agentes de browser da New Relic capturam métricas de frameworks SPA.
- **CI/CD**: alertas podem bloquear deploys via integração com pipeline.
- **Vercel**: complementar ao Speed Insights para análise mais profunda.

### Vale a pena acompanhar?

**Promissor para empresas** — o ROI aparece em times de médio/grande porte com SLAs de performance.

---

## 7. MCP em Produção — Enterprise Governance e SSO em 2026

### O que é

O Model Context Protocol (MCP) atingiu 97 milhões de downloads mensais de SDK e tem 10.000+ servidores MCP em produção. A principal evolução de meados de 2026 é a transição de deployments simples (API key + servidor MCP básico) para **enterprise gateways**: infraestrutura com SSO integrado, audit trails completos, governance de permissões e rate limiting por usuário/equipe.

### Por que isso importa

MCP se tornou a infraestrutura padrão de comunicação entre agentes de IA e ferramentas externas. Para times de engenharia, isso significa que qualquer agente (Claude Code, Cursor, OpenCode) pode agora acessar Figma, banco de dados, APIs internas, GitHub — de forma padronizada e auditável.

### Benefícios práticos

- SSO + RBAC: controle de qual agente acessa qual ferramenta por usuário/equipe.
- Audit trails: log completo de cada chamada de ferramenta feita por agentes de IA.
- Gateway centralizado: um ponto de controle para todos os MCPs da organização.
- MCP Apps: extensão do protocolo que suporta UI previews e elementos interativos (Figma designs, dashboards Slack) diretamente no agente.

### Possíveis problemas ou limitações

- Enterprise MCP gateways ainda são uma categoria nascente — poucas soluções maduras disponíveis.
- A segurança de MCPs é um vetor de ataque potencial — um MCP comprometido dá acesso ao agente.
- A complexidade de configurar SSO + audit em MCP é significativa para times pequenos.

### Exemplo prático

```ts
// Servidor MCP básico para expor dados internos
import { Server } from '@modelcontextprotocol/sdk/server/index.js'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js'

const server = new Server({
  name: 'internal-api-mcp',
  version: '1.0.0',
})

server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: 'get_component_usage',
    description: 'Get usage stats for a design system component',
    inputSchema: {
      type: 'object',
      properties: {
        componentName: { type: 'string' },
      },
    },
  }],
}))

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === 'get_component_usage') {
    const stats = await db.getComponentUsage(request.params.arguments.componentName)
    return { content: [{ type: 'text', text: JSON.stringify(stats) }] }
  }
})
```

### Relação com o ecossistema moderno

- **Figma**: MCP server nativo para agentes lerem design system.
- **Storybook**: MCP em desenvolvimento para expor componentes e stories.
- **GitHub**: MCP oficial para criar issues, PRs e code reviews via agentes.
- **Nuxt**: `@nuxtjs/mcp-toolkit` expõe documentação do projeto para agentes.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — MCP é infraestrutura, não tendência. Times de engenharia precisam de uma estratégia de MCP.

---

## 8. Google ADK 1.0 GA + A2A Protocol — Padrão Multi-Agent em TypeScript

### O que é

Em abril de 2026 no Google Cloud Next, o **Agent Development Kit (ADK)** atingiu o status 1.0 GA em Python, Go, Java e **TypeScript**. O protocolo **Agent-to-Agent (A2A)**, agora governado pela Linux Foundation, tem mais de 150 organizações em produção. Para devs front-end, o ADK TypeScript + o renderer **A2UI** (baseado em Lit Web Components) permitem construir interfaces para agentes de IA sem depender de React ou Vue.

### Por que isso importa

O A2A + ADK resolve a comunicação entre agentes de diferentes tecnologias: um agente Python pode delegar tarefas a um agente TypeScript, que por sua vez renderiza resultados em uma interface Lit. Para microfrontends e arquiteturas de agentes distribuídos, esse é o protocolo de orquestração que está emergindo como padrão.

### Benefícios práticos

- ADK TypeScript 1.0: estável para produção em Node.js e Edge.
- A2A Protocol: wire format padrão para delegação de tarefas entre agentes de diferentes providers.
- A2UI: renderer para agentes com componentes Lit — zero lock-in de framework.
- Graph-based workflow APIs: defina o fluxo de agentes visualmente com nós e arestas.
- Avaliação built-in: simulação de usuário e ambiente para testar agentes end-to-end.

### Possíveis problemas ou limitações

- A2UI com Lit Web Components tem DX menos polida que React/Vue para times acostumados com esses frameworks.
- A2A Protocol ainda não é suportado por todos os providers — Anthropic não tem suporte oficial ainda.
- ADK TypeScript 1.0 tem menos exemplos e documentação que a versão Python.

### Exemplo prático

```ts
// ADK TypeScript — agente simples com A2A
import { Agent, tool } from '@google/adk'

const searchAgent = new Agent({
  name: 'search-agent',
  model: 'gemini-2.0-flash',
  description: 'Searches the web for frontend development news',
  tools: [
    tool({
      name: 'web_search',
      description: 'Search the web',
      parameters: { query: { type: 'string' } },
      handler: async ({ query }) => {
        // implementação da busca
        return { results: [] }
      },
    }),
  ],
})

// Expor via A2A para outros agentes
export const agentCard = {
  name: searchAgent.name,
  capabilities: ['web_search'],
  endpoint: 'https://api.example.com/agents/search',
}
```

### Relação com o ecossistema moderno

- **Web Components**: A2UI usa Lit — compatível com qualquer framework via Custom Elements.
- **Microfrontends**: agentes A2A podem ser a camada de orquestração entre MFEs.
- **Google Cloud**: deploy nativo em Cloud Run com ADK.
- **Firebase**: integração direta para apps mobile + AI agents.

### Vale a pena acompanhar?

**Promissor para empresas** — A2A + ADK é a aposta do Google no futuro multi-agent. Vale acompanhar, principalmente quem usa GCP.

---

## 9. Cursor — Reestruturação de Planos e Modo Background Agent

### O que é

Em junho de 2026, o Cursor reestruturou seus planos para empresas: **Standard** ($32/seat/mês anual) e **Premium** ($96/seat/mês anual). Além da mudança de pricing, o Cursor expandiu seu **Background Agent** — que roda tarefas longas em segundo plano enquanto o dev trabalha em outra coisa — e melhorou o suporte a monorepos com workspaces isolados por projeto.

### Por que isso importa

O Background Agent muda fundamentalmente como trabalhar com IA em projetos grandes: ao invés de esperar o agente terminar uma tarefa longa, você pode pedir que ele refatore um módulo inteiro em background e ser notificado quando terminar — enquanto você trabalha em outra feature.

### Benefícios práticos

- Background Agent: tarefas longas (refatoração, análise de codebase) rodando em segundo plano.
- Workspaces isolados: cada projeto do monorepo tem contexto separado — menos ruído.
- Suporte a múltiplos MCPs por workspace.
- Model switching por tarefa: usa Claude para raciocínio, GPT-4o para completions rápidas.

### Possíveis problemas ou limitações

- O plano Premium ($96/seat) é caro para times grandes — TCO precisa ser comparado com alternativas.
- Background Agent ainda tem limitações em tarefas que requerem interação humana no meio do fluxo.
- A qualidade de workspaces em monorepos depende da configuração de limites de contexto.

### Exemplo prático

```
// Fluxo com Background Agent no Cursor:

1. Dev abre Cursor e inicia Background Agent:
   "Analise todos os componentes em src/components e crie stories Storybook
   para os que ainda não têm. Priorize componentes usados em mais de 3 lugares."

2. Background Agent começa a trabalhar

3. Dev trabalha em outra feature normalmente

4. 20 minutos depois: notificação "Background task complete: 23 stories created"

5. Dev revisa o PR gerado pelo agente
```

### Relação com o ecossistema moderno

- **VS Code**: Cursor é baseado em VS Code — extensões compatíveis.
- **MCP**: suporte nativo com configuração por workspace.
- **GitHub**: pode criar PRs automaticamente ao terminar tarefas em background.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — Background Agent é a feature mais transformadora de UX de IDE em 2026.

---

## 10. Antigravity / Windsurf — Consolidação do Mercado de AI IDEs

### O que é

O mercado de AI IDEs continua em consolidação acelerada em meados de 2026. O **Windsurf** foi adquirido pela OpenAI, o **Gemini CLI** foi absorvido pelo Antigravity, e a Amazon lançou o Kiro. O resultado é que o mercado está se dividindo em dois polos: ferramentas **agnósticas de modelo** (OpenCode, Cursor) e ferramentas **verticalmente integradas** com clouds específicas (Kiro/AWS, Antigravity/Google, GitHub Copilot/Microsoft).

### Por que isso importa

A escolha de IDE em 2026 é também uma escolha de cloud provider e modelo de governança. Times que querem flexibilidade de provider devem avaliar ferramentas agnósticas. Times com forte investimento em AWS, Google Cloud ou Azure têm ferramentas específicas com integração mais profunda.

### Benefícios práticos

- Antigravity (ex-Windsurf): integração com OpenAI GPT-5.5 e acesso a ferramentas Google.
- Kiro (AWS): integração nativa com serviços AWS e Claude via Bedrock.
- OpenCode: agnóstico, open-source, air-gappable — máxima flexibilidade.
- Claude Code: melhor raciocínio em benchmarks, profundidade com MCP ecosystem.

### Possíveis problemas ou limitações

- Consolidação cria lock-in: ferramentas de cloud vendors são poderosas mas atreladas ao ecossistema.
- Times que migram de ferramenta perdem contexto acumulado de conversas e preferências.
- A qualidade de ferramentas cloud-native depende da qualidade dos modelos daquela cloud — sem flexibilidade para mudar.

### Exemplo prático

```
// Matriz de decisão para escolha de AI IDE em 2026:

Cloud-native:
- AWS-heavy → Kiro
- Google Cloud → Antigravity CLI
- Microsoft/Azure → GitHub Copilot

Provider-agnóstico:
- Open-source + privacy → OpenCode
- Best DX + features → Cursor
- Best reasoning + MCP → Claude Code

Critérios adicionais:
- Air-gapped required → OpenCode (único com suporte completo)
- Enterprise governance → Kiro (spec-driven + audit hooks)
- Design System integration → Claude Code (Figma MCP + Storybook MCP)
```

### Relação com o ecossistema moderno

- **VS Code**: todas as ferramentas são compatíveis ou baseadas em VS Code.
- **MCP**: o diferencial de longo prazo será a qualidade e quantidade de MCPs suportados.
- **CI/CD**: todas as ferramentas oferecem algum nível de integração com GitHub Actions.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — a consolidação ainda está acontecendo. Avaliar ferramentas em H2 2026 fará sentido.

---

*Fontes principais: [OpenCode](https://opencode.ai/) · [Kiro](https://kiro.dev/) · [Havoptic AI Tools Tracker](https://www.havoptic.com/tools/github-copilot) · [Claude Code vs Cursor 2026](https://thenewstack.io/claude-code-vs-cursor-vs-codex-vs-antigravity-2026/) · [MCP Production Guide](https://thenewstack.io/model-context-protocol-roadmap-2026/) · [Google ADK Blog](https://developers.googleblog.com/agents-adk-agent-engine-a2a-enhancements-google-io/) · [New Relic June 2026](https://newrelic.com/blog/news/new-relic-now-june-2026-round-up) · [AI Coding Comparison](https://lushbinary.com/blog/ai-coding-agents-comparison-cursor-windsurf-claude-copilot-kiro-2026/)*
