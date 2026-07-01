# IA Weekly Research — 15/06/2026
> 10 Novidades: Agentes, MCPs, APIs e IA Aplicada a Testes

---

## 1. Anthropic Advisor Tool (Beta) — Dual-Pipeline: Executor + Advisor em Paralelo

### O que é

A Anthropic lançou o **Advisor Tool** em public beta — um novo paradigma agentic que pareia um modelo executor mais rápido com um modelo advisor de maior inteligência. Durante tasks de longa duração, o advisor fornece orientação estratégica mid-generation sem bloquear o executor. Na prática: um Sonnet 4.6 executa tarefas de codificação em alta velocidade, enquanto um Fable 5 observa o progresso e injeta guidance estratégica em pontos críticos ("antes de continuar, considere o impacto desta mudança em X").

O advisor não substitui o executor — ele consulta. O executor decide se segue ou não o conselho, mantendo o fluxo de execução rápido.

### Por que isso importa

Tasks longas de agente hoje têm um problema: o modelo executor perde contexto estratégico à medida que o contexto cresce. O Advisor Tool resolve isso com um segundo modelo que mantém a visão estratégica sem carregar o contexto técnico crescente. É especialmente útil em refatorações complexas onde decisões arquiteturais no início afetam o resultado final.

### Benefícios práticos

- Executor rápido + advisor inteligente: melhor qualidade sem sacrificar velocidade
- Advisor injeta guidance em pontos críticos sem bloquear o fluxo
- Separa raciocínio estratégico (Fable 5) de execução técnica (Sonnet 4.6) naturalmente
- Custo-efetivo: advisor só consome tokens quando ativado em pontos críticos
- API: configurável via `advisor_model` e `advisor_trigger_conditions`

### Possíveis problemas ou limitações

- Ainda em public beta — API pode mudar antes do GA
- Custo duplo: dois modelos consumindo tokens simultaneamente em pontos críticos
- O executor pode ignorar o advisor e tomar decisões subótimas mesmo com guidance
- Latência aumenta quando o advisor é consultado — tasks real-time não são candidatas
- Definir `advisor_trigger_conditions` corretamente requer experimentação por caso de uso

### Exemplo prático

```ts
import Anthropic from '@anthropic-ai/sdk'

const client = new Anthropic()

// Configuração com Advisor Tool
const response = await client.messages.create({
  model: 'claude-sonnet-4-6',         // executor: rápido
  max_tokens: 8096,
  tools: [codeEditTool, testRunTool],
  system: 'You are refactoring a large codebase. Execute efficiently.',
  messages: [{ role: 'user', content: 'Refactor auth module to use JWT' }],
  // Beta: Advisor Tool
  betas: ['advisor-2026-06'],
  advisor: {
    model: 'claude-fable-5',           // advisor: inteligente
    trigger_on: ['architecture_decision', 'security_concern', 'breaking_change'],
    advisor_prompt: `You are a senior architect. When consulted, provide 
    strategic guidance in 2-3 sentences. Flag security and breaking changes.`
  }
})
```

O fluxo em runtime:
```
Executor (Sonnet 4.6): "I'll replace session tokens with JWT..."
Advisor trigger: architecture_decision detected
Advisor (Fable 5): "Ensure refresh token rotation is implemented to prevent 
                    token theft. The current session table can be repurposed 
                    for refresh tokens — avoid a new migration."
Executor: incorporates guidance → implements refresh token rotation
```

### Relação com o ecossistema moderno

- **Claude Code**: Advisor Tool disponível como modo experimental no Claude Code 
- **Monorepos**: advisor com contexto arquitetural do monorepo orienta decisões cross-package
- **CI/CD**: tasks longas de migração em CI se beneficiam do advisor para manter consistência

### Vale a pena acompanhar?

**Sim, vale acompanhar — e testar em tasks longas de refatoração.** O padrão "executor veloz + advisor estratégico" é arquiteturalmente correto para automação de alta qualidade. Ainda em beta, mas a direção é clara.

---

## 2. Claude Code Sub-Agents (5 Níveis) — Orquestração Hierárquica de Tasks

### O que é

Claude Code passou a suportar **sub-agents aninhados até 5 níveis de profundidade** — agentes que são spawnados por agentes, que podem spawnar mais agentes, recursivamente. Na prática: um orchestrador de alto nível delega para especialistas, que por sua vez delegam para workers ainda mais específicos. Cada nível pode ter suas próprias tools disponíveis, seu próprio contexto e seu próprio modelo. Sub-agents independentes rodam em paralelo. O Claude Code Agent SDK expõe a API de orquestração em Python e TypeScript.

### Por que isso importa

A composição de agentes especializados supera um único agente generalista para tasks complexas. Um agente de "security review" especializado em identificar vulnerabilidades fará melhor work que um agente generalista com uma instrução de "review para segurança também". Sub-agents com 5 níveis de profundidade permitem hierarquias organizacionais reais: orchestrador → time lead → especialista → worker → verificador.

### Benefícios práticos

- Especialização real: cada sub-agent otimizado para um tipo de task específica
- Paralelismo: sub-agents independentes rodam simultaneamente → tasks mais rápidas
- Escalabilidade: adicionar um novo tipo de task = adicionar um novo tipo de sub-agent
- Isolamento de contexto: cada sub-agent tem seu próprio context window limpo
- SDK Python/TypeScript: composição de agentes como código normal

### Possíveis problemas ou limitações

- 5 níveis com paralelismo = crescimento exponencial de consumo de tokens e custo
- Debug é complexo: trace de execução em 5 níveis de profundidade é difícil de seguir
- Sub-agents muito autônomos podem tomar decisões conflitantes entre si
- Sem mecanismo nativo de rollback se um sub-agent faz mudanças incorretas
- O SDK ainda tem limitações de timeout e rate limiting que afetam hierarquias profundas

### Exemplo prático

```ts
// Claude Code Agent SDK — TypeScript
import { Agent, spawn } from '@anthropic-ai/agent-sdk'

const codeReviewOrchestrator = new Agent({
  model: 'claude-opus-4-8',
  tools: [],
  instructions: 'You orchestrate a comprehensive code review',
  
  async run(input: { prUrl: string }) {
    // Spawn 4 sub-agents em paralelo (nível 1)
    const [security, performance, tests, style] = await Promise.all([
      spawn('security-reviewer', { prUrl: input.prUrl }),
      spawn('performance-analyzer', { prUrl: input.prUrl }),
      spawn('test-coverage-checker', { prUrl: input.prUrl }),
      spawn('code-style-enforcer', { prUrl: input.prUrl }),
    ])

    // Security reviewer spawna sub-agents por categoria (nível 2)
    // spawn('xss-checker', ...), spawn('injection-checker', ...), etc.
    
    return compileReport([security, performance, tests, style])
  }
})
```

Iniciando via Claude Code:
```bash
/start-review --pr=123 --orchestrator=code-review-orchestrator
```

Output do orchestrador:
```
[Level 1] Security Reviewer: 2 issues found
  [Level 2] XSS Checker: 0 issues
  [Level 2] SQL Injection Checker: 1 issue (api/users.ts:L78)
  [Level 2] Auth Checker: 1 issue (missing rate limiting on /login)
[Level 1] Performance Analyzer: 1 issue (N+1 in getUserOrders)
[Level 1] Test Coverage: 67% (below 80% threshold)
[Level 1] Style Enforcer: 12 minor issues (auto-fixable)
```

### Relação com o ecossistema moderno

- **Self-hosted Sandboxes**: cada sub-agent pode ter seu próprio sandbox isolado
- **CI/CD**: orchestrador como step de review automático completo em PRs
- **Monorepos**: orchestrador → sub-agents por package → workers por arquivo

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O padrão de composição de agentes especializados é o modelo correto para automação de software em escala. O SDK ainda está evoluindo, mas já é utilizável para pipelines de review e análise.

---

## 3. Nuxt MCP Server — Fonte de Verdade Estruturada para Agentes Locais

### O que é

O **Nuxt MCP Server** (`@nuxt/mcp`) é o servidor MCP oficial do ecossistema Nuxt, que expõe como tools estruturadas: documentação completa (páginas, guias, API reference), catálogo de módulos com meta-dados (compatibilidade, stars, maintained), blog posts e changelogs, e guias de deployment por plataforma. É o mesmo servidor que alimenta o Nuxt Agent (Nuxi) no site nuxt.com — garantindo que o agente web e assistentes locais (Cursor, Claude Desktop) compartilhem a mesma fonte de dados atualizada.

### Por que isso importa

O problema comum com agentes AI em projetos Nuxt é que os modelos têm conhecimento desatualizado do framework (corte de treinamento em ago/2025 para maioria dos modelos). O MCP server resolve isso: as docs são buscadas em runtime, não do treinamento. Um agente usando o Nuxt MCP server nunca vai sugerir APIs depreciadas ou padrões do Nuxt 3 quando o projeto é Nuxt 4.

### Benefícios práticos

- Docs sempre atualizadas: agente usa documentação em tempo real, não do treinamento
- Catálogo de módulos: agente conhece quais módulos existem, sua compatibilidade e maturidade
- Zero alucinações sobre APIs Nuxt — grounded em fonte oficial
- Configuração em 2 linhas no Cursor ou Claude Desktop
- Compartilhado entre nuxt.com e ferramentas locais — consistência garantida

### Possíveis problemas ou limitações

- Requer conexão com internet; ambientes air-gapped não funcionam
- Não cobre documentação de módulos de terceiros (só o catálogo de referência)
- O MCP server pode retornar docs de versão diferente da usada no projeto se não configurado explicitamente
- Latência de rede adiciona ~200-500ms por consulta de docs

### Exemplo prático

```json
// Cursor — .cursor/mcp.json
{
  "mcpServers": {
    "nuxt": {
      "command": "npx",
      "args": ["-y", "@nuxt/mcp@latest", "--version=4"]
    }
  }
}
```

```json
// Claude Desktop — claude_desktop_config.json
{
  "mcpServers": {
    "nuxt": {
      "command": "npx",
      "args": ["-y", "@nuxt/mcp"]
    }
  }
}
```

Ferramentas expostas pelo MCP server:
```
- nuxt_docs(query: string) → documentação relevante
- nuxt_modules(filter: { category, compatible_with }) → módulos filtrados
- nuxt_changelog(version?: string) → changelog do Nuxt
- nuxt_recipes(use_case: string) → receitas de implementação
```

Exemplo de uso pelo agente:
```
[CONTEXT from Nuxt MCP]
Tool: nuxt_docs("server-side rendering configuration nuxt 4")
Result: "Nuxt 4 SSR is configured via nuxt.config.ts ssr property...
         Universal rendering is default. Prerendering via nitro.prerender..."
```

### Relação com o ecossistema moderno

- **Storybook MCP**: combinado, o agente tem contexto de framework (Nuxt) + design system
- **Claude Code**: skill que usa Nuxt MCP para context-aware scaffolding
- **Figma Dev Mode MCP**: pipeline design→docs→code com context completo

### Vale a pena acompanhar?

**Sim, vale acompanhar — e configurar imediatamente para projetos Nuxt.** É um dos MCPs de maior custo-benefício disponíveis: instalação em 2 minutos, impacto imediato na qualidade das sugestões do agente.

---

## 4. Storybook MCP — Loop de Autocorreção: Agentes que Corrigem Seus Próprios Bugs

### O que é

O **Storybook MCP Server** oficial expõe os componentes do design system como contexto estruturado para agentes, mas o diferencial técnico é o **autonomous correction loop**: o agente gera UI usando os componentes do DS, executa automaticamente os testes de interação e acessibilidade do Storybook, vê o que falhou, e corrige seus próprios bugs antes de entregar o código ao desenvolvedor. O loop continua até todos os testes passarem ou o agente atingir um limite de tentativas configurável.

### Por que isso importa

Este é o primeiro exemplo prático de "agente com feedback de qualidade integrado" em desenvolvimento frontend: o agente não apenas gera código — ele valida e itera sobre esse código usando a mesma suíte de testes que o time humano usa. O resultado é código que passa nos testes desde o primeiro commit, não após uma rodada de revisão humana.

### Benefícios práticos

- Output de agente já passa nos testes de interação e acessibilidade do Storybook
- Menos ciclos de revisão humana para bugs básicos de acessibilidade ou comportamento
- Testes como gate de qualidade automático: agente não entrega código que falha
- Design System constraints enforced via testes — não apenas via prompt
- Story UI: não-devs geram layouts com linguagem natural com qualidade garantida por testes

### Possíveis problemas ou limitações

- Requer Storybook bem configurado com interaction tests e a11y tests por componente
- Loop de correção tem custo de tokens (múltiplas tentativas + execução de testes)
- Componentes sem stories testáveis não se beneficiam do loop de autocorreção
- Limite de tentativas pode ser atingido sem resolução para bugs complexos de acessibilidade
- Storybook deve estar rodando localmente ou em staging para o MCP funcionar

### Exemplo prático

Fluxo do loop de autocorreção:

```
Prompt: "Create a user profile card using our design system"

Iteration 1:
  → Agente consulta Storybook MCP para Card, Avatar, Badge components
  → Gera UserProfileCard.tsx com os componentes corretos
  → Executa: npx storybook test --story=UserProfileCard
  → Resultado: 2 failures
    - [a11y] Badge: insufficient color contrast (4.2:1, requires 4.5:1)
    - [interaction] Avatar fallback not rendered when src fails

Iteration 2:
  → Corrige Badge variant para "secondary" (maior contraste)
  → Adiciona onError handler no Avatar
  → Executa: npx storybook test --story=UserProfileCard
  → Resultado: All tests pass ✓

Entrega ao desenvolvedor: código com testes passando + story gerada
```

```ts
// Configuração do Storybook MCP com correction loop
{
  "mcpServers": {
    "storybook": {
      "command": "npx",
      "args": [
        "@storybook/mcp-server",
        "--storybook-url", "http://localhost:6006",
        "--correction-loop", "true",
        "--max-iterations", "3",
        "--test-on-generate", "true"
      ]
    }
  }
}
```

### Relação com o ecossistema moderno

- **Vue + React**: Storybook MCP é framework-agnostic
- **Nuxt MCP**: combinados — agente tem context de framework + design system
- **CI/CD**: o mesmo teste que o agente usa no loop também roda no pipeline de CI
- **axe-core**: testes de acessibilidade do Storybook usam axe-core internamente

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O loop de autocorreção é um padrão que vai se expandir para outros domínios (testes unitários, tipos TypeScript, lint). Storybook MCP é a implementação mais madura atualmente disponível.

---

## 5. Playwright Test Agents Agentic Loop — Pipeline Completo Planner→Generator→Healer

### O que é

O **Playwright Test Agents** (v1.56+) implementa um sistema de três agentes especializados que podem ser encadeados em um loop agentic completo: **Planner** (explora o app e gera plano de testes em Markdown), **Generator** (transforma o plano em código Playwright executável, validando seletores contra browser real) e **Healer** (executa a suite, identifica falhas e gera correções automaticamente). O loop pode ser executado em sequência, individualmente ou de forma cíclica (Planner→Generator→Healer→Healer→Healer até passar). Integra com Cursor, VS Code, Claude Code e OpenCode via `--loop=<tool>`.

### Por que isso importa

O maior custo dos testes E2E é a manutenção contínua — seletores quebram com mudanças de UI, testes falham por timing, novos fluxos ficam sem cobertura. O Healer fecha esse loop automaticamente: após um deploy, o CI identifica quais testes falharam, o Healer analisa e propõe fixes, e o desenvolvedor apenas revisa e aprova. Isso transforma testes E2E de "custo contínuo" em "investimento que se auto-mantém".

### Benefícios práticos

- Pipeline completo de test coverage autônomo para fluxos novos
- Healer reduz dramatically o custo de manutenção de suites E2E existentes
- Planner gera planos legíveis por humanos — revisáveis antes da execução do Generator
- Seletores do Generator validados contra browser real antes de gerar código
- Loop configurável: usa tool de IA preferida (Claude, Cursor, VS Code)

### Possíveis problemas ou limitações

- Healer tem taxa de sucesso variável: funciona bem para mudanças simples de seletor, mas falha em mudanças de fluxo complexas
- Planner pode gerar planos excessivamente abrangentes em apps grandes — revisão humana necessária
- Loop completo consome tokens de IA + tempo de execução de browser: custo por execução é relevante
- Testes gerados por Generator às vezes testam implementation details — problema comum em geração automática
- Requer app rodando para o Planner explorar; sem app = sem plano

### Exemplo prático

```bash
# Instalar + inicializar loop com Claude Code
npx playwright init-agents --loop=claude

# Executar apenas o Healer no suite existente
npx playwright agent healer --suite=tests/e2e/

# Executar pipeline completo para um novo fluxo
npx playwright agent --loop planner,generator,healer \
  --url http://localhost:3000/checkout \
  --goal "Test the complete purchase flow from cart to confirmation"
  
# Output do Planner (Markdown):
# ## Checkout Flow Test Plan
# 1. Navigate to /cart with items
# 2. Click "Proceed to Checkout"  
# 3. Fill shipping address (required fields)
# 4. Select shipping method
# 5. Enter payment details
# 6. Submit order
# 7. Verify confirmation page with order number
# 8. Verify email confirmation trigger (check network request)
```

Output do Healer após UI change:
```
Test failure: checkout_spec.ts:L45
  Selector '#proceed-to-checkout' not found
  
Healer analysis: Button was renamed to "Continue to Checkout" in recent deploy
Suggested fix: Replace '#proceed-to-checkout' with 
  getByRole('button', { name: 'Continue to Checkout' })
  
Confidence: 94% — Apply? [Y/n]
```

### Relação com o ecossistema moderno

- **Vitest**: testes unitários com Vitest + Playwright Agents E2E = suite completo com auto-healing
- **CI/CD**: Healer como step de "auto-fix" em pipeline de staging após deploy
- **Vue + React**: agnóstico de framework — funciona com qualquer app que rode no browser
- **axe-core**: Planner pode incluir accessibility checks no plano de testes

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O Healer é a feature de maior ROI — times que hoje gastam horas mantendo suites E2E quebradas vão transformar isso em minutos de revisão de sugestões automáticas.

---

## 6. Vercel AI SDK v5 — UIMessage vs ModelMessage + Agentic Loop Control

### O que é

Vercel AI SDK v5 (lançado em julho de 2025, amplamente adotado em 2026) introduziu uma separação arquitetural fundamental: **UIMessage** (mensagem como a UI vê — com streaming parcial, parts de reasoning, tool calls visuais) vs. **ModelMessage** (mensagem como o modelo precisa — sem metadata de UI, formato específico do provider). A conversão entre os tipos é feita automaticamente antes do envio ao modelo. Adicionalmente, **Agentic Loop Control** via `stopWhen` e `prepareStep` permite controle fino de quando um loop de tool calls termina e como preparar o próximo passo.

### Por que isso importa

A separação UIMessage/ModelMessage elimina uma classe de bugs em apps de chat: armazenar a mensagem como UIMessage no banco permite restaurar o estado visual exato (partes de reasoning, tool calls expandidos/colapsados), enquanto o ModelMessage garante que o modelo receba apenas o que precisa. Antes, times escolhiam um formato e perdiam algo — agora os dois são first-class.

### Benefícios práticos

- Persistência de chat com estado visual completo (reasoning, tool calls) via UIMessage
- Modelo recebe formato limpo via ModelMessage — sem metadata de UI desnecessária
- `stopWhen`: define condições para parar o loop agentic (ex: resultado encontrado, limite de steps)
- `prepareStep`: modifica tools disponíveis ou system prompt a cada step do loop
- Agent class: wrapper leve para loops agênticos sem overhead de frameworks pesados

### Possíveis problemas ou limitações

- Breaking change em relação a AI SDK v4: mensagens precisam ser convertidas; migração não é trivial
- UIMessage tem schema mais complexo para serializar/deserializar do banco de dados
- `stopWhen` com condições complexas pode ter comportamento inesperado com diferentes modelos
- Agent class ainda limitada — loops complexos precisam de `generateText`/`streamText` diretos
- Documentação de migração v4→v5 ainda incompleta em alguns casos de edge

### Exemplo prático

```ts
// AI SDK v5 — UIMessage vs ModelMessage
import { useChat } from '@ai-sdk/react'
import { convertToModelMessages } from 'ai'

// Na UI
const { messages, handleSubmit } = useChat()
// messages são UIMessage[] — com partes de reasoning, tool calls, etc.

// No server action/route
import { streamText } from 'ai'

export async function POST(req: Request) {
  const { messages } = await req.json() // UIMessage[]
  
  const result = await streamText({
    model: anthropic('claude-sonnet-4-6'),
    messages: convertToModelMessages(messages), // → ModelMessage[]
    tools: { ... }
  })
  
  return result.toDataStreamResponse()
}
```

```ts
// Agentic Loop Control com stopWhen e prepareStep
import { generateText } from 'ai'

const result = await generateText({
  model: anthropic('claude-sonnet-4-6'),
  tools: { searchWeb, readFile, writeCode },
  
  // Para quando encontrar código funcionando
  stopWhen: ({ steps }) => {
    const lastStep = steps[steps.length - 1]
    return lastStep?.toolResults?.some(r => r.result?.status === 'tests_pass')
  },
  
  // Modifica tools por step
  prepareStep: ({ stepNumber, steps }) => {
    if (stepNumber === 0) {
      // Primeiro step: só pode pesquisar
      return { tools: { searchWeb } }
    }
    if (stepNumber > 5) {
      // Após 5 steps: força finalização
      return { system: 'Finalize with what you have. Do not use more tools.' }
    }
    return {}
  },
  
  maxSteps: 10,
  prompt: 'Implement JWT authentication in auth/index.ts',
})
```

### Relação com o ecossistema moderno

- **Vue + Nuxt**: `@ai-sdk/vue` tem `useChat` com UIMessage nativo
- **React + Next.js**: `@ai-sdk/react` é a implementação de referência
- **Nuxt UI**: componentes UChatMessages consomem UIMessage nativamente
- **Vercel AI Gateway**: AI SDK v5 roteado via Gateway com fallback automático

### Vale a pena acompanhar?

**Sim, vale acompanhar — e migrar projetos em produção.** A separação UIMessage/ModelMessage resolve um problema real de persistência de estado de chat. `stopWhen`/`prepareStep` são os primitivos corretos para loops agênticos controlados.

---

## 7. Models API da Anthropic — Campos de Capability e 300k Batches Output

### O que é

Duas mudanças técnicas na Anthropic API em junho de 2026: (1) **Models API com capability fields** — cada modelo retorna agora um objeto `capabilities` com flags booleanas para vision, extended_thinking, tool_use, computer_use, document_intelligence; (2) **300k output tokens via Message Batches API** — o limite de output foi aumentado para 300.000 tokens em batch requests, acessível com o header beta `anthropic-beta: output-300k-2026-03-24`, disponível em Claude Opus 4.8 e Sonnet 4.6.

### Por que isso importa

Capability fields permitem que aplicações se adaptem programaticamente ao modelo disponível — sem hardcoding de "o modelo X suporta vision". 300k output tokens via Batches transforma o que é possível em processamento batch: gerar suites de teste completas, documentação extensiva, ou análises grandes em uma única request batch.

### Benefícios práticos

- Capability flags: código que adapta features ao modelo sem if/else hardcoded
- 300k output: gerar suites de teste de 10.000 linhas em uma batch request
- Batches API para processamento assíncrono em larga escala sem streaming
- Models API como single source of truth para decisões de roteamento

### Possíveis problemas ou limitações

- 300k output só disponível via Batches API (assíncrono) — não funciona com streaming
- Capability fields ainda em beta — podem mudar antes do GA
- 300k tokens de output tem custo proporcional — calcular ROI antes de usar
- Batches API tem timeout de 24h; requests muito longas podem expirar

### Exemplo prático

```ts
import Anthropic from '@anthropic-ai/sdk'

const client = new Anthropic()

// Verificar capabilities programaticamente
const models = await client.models.list()
const sonnet46 = models.data.find(m => m.id === 'claude-sonnet-4-6')

if (sonnet46.capabilities.vision) {
  // Habilitar features de análise de imagem
  enableImageAnalysis()
}

if (sonnet46.capabilities.extended_thinking) {
  // Habilitar modo de raciocínio profundo
  enableDeepReasoning()
}

// 300k output via Message Batches API
const batch = await client.beta.messages.batches.create({
  requests: repos.map(repo => ({
    custom_id: repo.name,
    params: {
      model: 'claude-sonnet-4-6',
      max_tokens: 300000,
      messages: [{
        role: 'user',
        content: `Generate a complete test suite for this codebase:\n\n${repo.code}`
      }]
    }
  }))
}, {
  headers: {
    'anthropic-beta': 'output-300k-2026-03-24'
  }
})

// Polling do resultado
const results = await client.beta.messages.batches.results(batch.id)
```

### Relação com o ecossistema moderno

- **Vercel AI SDK**: models list integrado; capability flags influenciam provider config
- **Claude Code**: usa Models API internamente para selecionar modelo por capability
- **CI/CD**: Batches API para geração de documentação ou testes em background

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Capability fields são feature de longa duração — todo cliente Anthropic deveria migrar para verificação programática de capabilities. 300k output em Batches é para casos específicos de processamento em larga escala.

---

## 8. shadcn/improve Pattern — "Think Expensive, Execute Cheap" como Padrão Emergente

### O que é

O lançamento do `shadcn/improve` (10 jun 2026) cristalizou um padrão arquitetural emergente em workflows agênticos que vai além do skill específico: **separar raciocínio estratégico (modelo caro, uma vez) de execução técnica (modelo barato, múltiplas vezes)**. O skill usa o modelo mais capaz disponível para auditar um codebase e produzir um plano de implementação tão preciso que modelos mais baratos podem executar as steps sem supervisão humana. A regra central é hard-coded no skill: **nunca editar código-fonte** — o output é sempre um documento de plano, não mudanças diretas.

### Por que isso importa

Este padrão resolve o maior custo de workflows agênticos: usar modelos frontier para tasks que modelos menores poderiam fazer. Ao separar as fases, o modelo frontier só é ativado para o que realmente requer sua capacidade — raciocínio sistêmico sobre o codebase inteiro — enquanto a execução step-by-step usa modelos mais baratos que já comprovadamente realizam essa task bem.

### Benefícios práticos

- Auditoria de qualidade frontier + execução de custo controlado
- Planos legíveis por humanos: equipe revisa estratégia antes da execução
- Read-only durante auditoria: zero risco de side effects acidentais
- Padrão reutilizável: não é específico ao shadcn/improve — qualquer workflow agêntico se beneficia
- Janelas de acesso a modelos frontier (ex: Fable 5 gratuito até 22/06) são ideais para a fase de auditoria

### Possíveis problemas ou limitações

- A qualidade do plano depende da qualidade da análise — codebases mal estruturados produzem planos genéricos
- Teams precisam criar um processo de revisão dos planos antes de autorizar execução
- A separação perde valor quando a task requer raciocínio contínuo durante a execução (não apenas no início)
- Planos para codebases grandes podem ser extensos demais para o modelo executor ter contexto completo

### Exemplo prático

O padrão como template reutilizável:

```markdown
# [AUDIT PHASE — Use most capable model]

Você está no modo de AUDITORIA. Regras:
1. NUNCA edite código-fonte
2. Analise profundamente o codebase
3. Identifique os problemas de maior impacto
4. Escreva planos executáveis por um modelo menor

## Estrutura do Plano
Para cada problema encontrado:
- **Localização**: arquivo:linha
- **Problema**: descrição técnica precisa
- **Impacto**: estimativa de severidade
- **Plano de execução**: steps que um modelo Sonnet pode executar
  - Step 1: [ação específica e verificável]
  - Step 2: [ação específica e verificável]
- **Verificação**: como confirmar que o fix funcionou

Output: APENAS o documento de plano, sem edições de código.
```

```markdown
# [EXECUTION PHASE — Use cheaper model]

Você está no modo de EXECUÇÃO. Regras:
1. Execute APENAS as steps do plano fornecido
2. Execute uma step de cada vez
3. Após cada step, rode os testes especificados
4. Se um teste falhar, pare e relate — não improvise
5. Não faça mudanças não previstas no plano

[Plano gerado pelo modelo de auditoria aqui]
```

### Relação com o ecossistema moderno

- **Claude Code**: padrão nativo com Agent SDK (orchestrador usa Fable 5, workers usam Sonnet)
- **Anthropic Advisor Tool**: evolução desse padrão onde o advisor é o "modelo caro" contínuo
- **CI/CD**: fase de auditoria semanalmente (Fable 5), execução de tasks diariamente (Sonnet)

### Vale a pena acompanhar?

**Sim, vale acompanhar — e aplicar imediatamente.** Este padrão é independente de ferramenta específica e aplica-se a qualquer workflow agêntico. É a diferença entre usar AI de forma intuitiva e de forma estratégica.

---

## 9. Hud SDK — Runtime Data Estruturado para AI Coding Agents

### O que é

**Hud** possui um SDK (`hud-sdk` no npm/PyPI) que instrumenta código em produção e transmite dados em nível de função — call counts, argumentos de entrada/saída, latência, taxa de erro — em formato estruturado especificamente projetado para ser consumido por AI coding agents. A diferença técnica em relação a APMs tradicionais: o formato de output é JSON com metadata contextual que agentes entendem (qual função, em qual arquivo, linha, com quais inputs, em qual condição o erro ocorreu), não métricas numéricas brutas. A extensão IDE (VS Code/JetBrains) renderiza essas informações inline no código.

### Por que isso importa

Debugging com AI hoje tem um gap crítico: o agente vê o código estático mas não sabe o que aconteceu em runtime. Com o Hud SDK, o agente recebe: "a função `getUserById` foi chamada 847 vezes na última hora com taxa de erro de 2.3%, falhando quando `userId` é `undefined`, chamada de `authMiddleware.ts:L78`". Isso permite diagnóstico imediato sem o dev precisar reproduzir o bug localmente.

### Benefícios práticos

- Contexto de produção inline na IDE: linha do bug, argumentos que causaram o erro, frequência
- AI agents com runtime context: diagnóstico mais preciso, menos tentativa e erro
- Redução de triagem de P1: de horas para minutos (documentado em case studies)
- Overhead negligível: deployado em escala sem impacto mensurável
- Estrutura do output otimizada para LLMs: não precisa de "tradução" de métricas brutas

### Possíveis problemas ou limitações

- Dados de produção na IDE levantam questões de segurança (PII em argumentos de funções)
- Depende de extensão instalada no IDE — adição de dependência ao workflow
- Não substitui APM completo — é complementar, foco no contexto de AI coding
- Categoria emergente: integração com Cursor, Claude Code ainda não é nativa (requer setup manual)
- Empresa relativamente nova — risco de continuidade de suporte a considerar

### Exemplo prático

```ts
// Instrumentação básica do Hud SDK
import { instrument } from '@hud-sdk/node'

instrument({
  apiKey: process.env.HUD_API_KEY,
  services: ['api', 'worker', 'scheduler'],
  sensitivity: 'function-level',
  ideSink: true,        // envia para extensão IDE
  agentContext: true,   // formata output para AI agents
  privacy: {
    redactFields: ['password', 'token', 'ssn'], // PII redaction
    maxArgLength: 200,
  }
})
```

Contexto injetado automaticamente no Cursor quando o agente está trabalhando em `auth.ts`:
```
[HUD Runtime Context — auth.ts]
Function: validateToken (auth.ts:L42)
Last 1h: 1,247 calls | 0.8% error rate | avg 12ms
Failing inputs: token="undefined" (47 times), token="" (12 times)
Callers: middleware/auth.ts:L89 (83%), api/auth/route.ts:L34 (17%)
Last error: 2026-06-15T14:23:18Z — "Cannot parse undefined as JWT"
```

O agente com esse contexto identifica imediatamente: "o middleware está passando `token` sem verificar se existe — adicionar guard clause em middleware/auth.ts:L89".

### Relação com o ecossistema moderno

- **Next.js + Nuxt**: SDK para Node.js compatível com ambos
- **Vercel + Cloudflare**: versão edge do SDK (< 5kb gzipped)
- **CI/CD**: alerta automático quando error rate sobe após deploy
- **Cursor + Claude Code**: contexto de runtime disponível inline durante coding

### Vale a pena acompanhar?

**Sim, vale acompanhar.** A categoria "runtime context para AI agents" é nova e o Hud é o líder atual. Para times já usando AI coding intensivamente, o ROI no debugging é claro. Avalie a política de privacy antes de adotar em produção com dados sensíveis.

---

## 10. AI Testing Landscape 2026 — Vitest + Playwright + Visual Regression + Agentes

### O que é

O landscape de testes frontend em 2026 consolidou em quatro camadas com integração crescente de AI: **Vitest** substituiu Jest como padrão para unit/integration (100x mais rápido, native ESM, compatível com Vite+); **Playwright** domina E2E com os Test Agents (Planner/Generator/Healer); **Visual Regression Testing** maturou com ferramentas como Chromatic e Percy integrando AI para distinguir mudanças intencionais de regressões; e **AI-Powered Test Orchestration** — agentes que decidem dinamicamente quais testes rodar baseados no diff do PR, priorizando cobertura dos caminhos de código alterados.

### Por que isso importa

A integração de AI em cada camada cria um pipeline de qualidade que é mais do que a soma das partes: Vitest rápido → Playwright Agents auto-healing → Visual AI sem false positives → Orchestração inteligente de quais testes priorizar. Times que antes escolhiam entre velocidade e coverage agora podem ter os dois.

### Benefícios práticos

- Vitest: cold start de 2ms vs. Jest 400ms; compatível com Vite+/Vite 8 nativamente
- Playwright Agents: E2E que se auto-mantém (Healer) e gera sua própria cobertura (Planner+Generator)
- Visual AI: apenas regressões reais são flagadas — sem false positives por anti-aliasing
- Test orchestration por AI: PRs pequenos rodam subset focado; PRs grandes rodam suite completo
- Cobertura de acessibilidade via axe-core integrado em todos os níveis

### Possíveis problemas ou limitações

- Migrar de Jest para Vitest ainda tem edge cases em projetos legados (principalmente CJS)
- Visual Regression AI tem custo de screenshot storage e processamento que escala com o DS
- AI Orchestration de testes ainda é experimental — maioria dos times usa heurísticas manuais
- Playwright Agents consomem tokens de AI — cost per pipeline run precisa ser calculado
- Coverage de acessibilidade automatizada cobre apenas 20-40% dos problemas reais (WCAG 2.2)

### Exemplo prático

Pipeline completo de testes em 2026:

```yaml
# .github/workflows/test.yml
name: Full Test Suite

jobs:
  unit-integration:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx vp test --coverage  # Vitest via Vite+
      - run: npx vp test --reporter=json > coverage.json
      
  e2e:
    needs: unit-integration
    runs-on: ubuntu-latest
    steps:
      - run: npm run dev &
      - run: |
          # AI Orchestration: descobre quais E2E rodar baseado no diff
          npx playwright test \
            --reporter=ai-orchestrator \
            --diff-based-selection \
            --url=http://localhost:3000
      - run: |
          # Healer: auto-corrige testes quebrados por mudanças de UI
          npx playwright agent healer \
            --suite=tests/e2e/
            
  visual-regression:
    needs: e2e
    runs-on: ubuntu-latest
    steps:
      - run: npx chromatic --only-changed  # AI distingue regressões de intencionais
      
  accessibility:
    needs: unit-integration
    steps:
      - run: |
          npx playwright test tests/a11y/ \
            --grep="wcag" \
            --reporter=axe-summary
```

Configuração do Vitest com cobertura e a11y:
```ts
// vite.config.ts (ou vitest.config.ts)
export default {
  test: {
    environment: 'jsdom',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov', 'json'],
      thresholds: { lines: 80, branches: 75, functions: 80 },
    },
    setupFiles: ['./tests/setup/a11y.ts'],
  },
}
```

### Relação com o ecossistema moderno

- **Vite+**: `vp test` unifica Vitest no CLI único
- **Vue + React**: Vitest suporta `@vue/test-utils` e `@testing-library/react` nativamente
- **CI/CD**: AI Orchestration reduz tempo de pipeline sem sacrificar cobertura
- **Design Systems**: testes de componentes + visual regression + a11y = cobertura completa do DS

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O landscape de 2026 é mais maduro e integrado do que nunca. O investimento mínimo recomendado: Vitest (imediato), Playwright Agents Healer (quando tiver suite E2E), Visual Regression AI (quando DS tiver escala).

---

*Pesquisa gerada em 15/06/2026 | Próxima edição: 22/06/2026*
