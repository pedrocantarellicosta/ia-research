# Front-End AI Weekly — Novidades de IA | 08 de Junho de 2026

> **Data de geração:** 08 de Junho de 2026
> **Período coberto:** 01–08 de Junho de 2026
> **Arquivo:** 5 Novidades de IA
> **Curadoria:** Skill `frontend-ia-weekly` via Claude Sonnet 4.6

---

## 1. MCP Apps — UI Rica em Iframes Sandboxed Dentro de Contextos de Agente

### O que é

O **MCP Apps** é uma extensão oficial ao Model Context Protocol, construída sobre a biblioteca `mcp-ui`, que transforma o que é possível retornar de uma tool MCP. Até agora, o protocolo era estritamente texto e dados estruturados — uma tool retornava JSON, markdown ou texto simples. Com MCP Apps, tools podem retornar **HTML interativo renderizado em iframes sandboxed** diretamente dentro da interface do agente.

Essa extensão foi incluída no Release Candidate do MCP (travado em 21/05) e já tem implementações em Claude.ai e em clients compatíveis com a spec.

**O que muda tecnicamente:**

```
Antes:
Tool → retorna JSON/texto → LLM interpreta → LLM responde em texto

Depois:
Tool → retorna UiResult com HTML → Client renderiza em iframe sandboxed
→ Usuário interage com a UI → actions são enviadas de volta para a tool
```

O HTML renderizado no iframe é sandboxed — sem acesso ao DOM pai, sem cookies, sem storage cross-origin. O canal de comunicação com o agente é via `postMessage` estruturado definido pelo protocolo.

**Casos de uso reais para frontend:**

```ts
// Tool que retorna um seletor visual de componentes do DS
server.tool('browse_design_system', async ({ query }) => {
  const components = await searchComponents(query)

  return createUiResult({
    html: generateComponentGallery(components), // cards com preview + código
    actions: [
      { name: 'insert_component', label: 'Inserir no código' },
      { name: 'view_docs', label: 'Ver documentação' },
    ],
  })
})

// Tool que retorna form de configuração de feature flag
server.tool('configure_feature_flag', async ({ flagName }) => {
  const current = await getFlag(flagName)
  return createUiResult({
    html: generateFlagConfigForm(current),
    actions: [{ name: 'save_flag', label: 'Salvar configuração' }],
  })
})
```

### Por que isso importa

A natureza texto-first do MCP criava uma limitação real: para qualquer dado que se beneficia de visualização (gráficos, tabelas, previews de componentes, diffs de layout), o agente precisava descrever em texto o que poderia simplesmente mostrar. MCP Apps fecha esse gap sem sair do contexto de agente.

Para times de frontend especificamente, isso abre casos de uso como: browsing visual do design system, comparação de variantes de componentes, configuração de tokens com preview em tempo real, e revisão de diffs visuais — tudo dentro da interface do agente, sem alternar de contexto.

### Benefícios práticos

- Browsing visual de componentes de DS dentro do agente — sem sair para Storybook
- Dashboards de métricas de componentes inline na conversa
- Forms de configuração (feature flags, tokens, env vars) sem abrir outra ferramenta
- Diffs visuais de layout (before/after) renderizados diretamente no contexto
- Protocolo padronizado — implementação única funciona em Claude.ai, Devin Desktop e qualquer client MCP Apps compatível

### Possíveis problemas ou limitações

- Sandbox restritiva — HTML com CSS externo ou scripts de terceiros não funciona por padrão
- Nem todos os clients MCP implementam MCP Apps — Claude.ai e Devin Desktop suportam, outros ainda não
- Complexidade de debug: "bug na UI do iframe" vs "bug no payload retornado pela tool" requer ferramentas específicas
- Estado dentro do iframe é volátil — recarrega com cada nova invocação da tool
- Interatividade limitada pelo protocolo `postMessage` — não é uma SPA completa

### Exemplo prático

```ts
// Servidor MCP que serve um mini design system browser

import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js'
import { createUiResult } from 'mcp-ui'

const server = new McpServer({ name: 'ds-browser', version: '1.0.0' })

server.tool('browse_components', { query: z.string() }, async ({ query }) => {
  const matches = await vectorSearch(query, componentEmbeddings)

  const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: system-ui; padding: 16px; background: #fff; }
        .grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
        .card { border: 1px solid #e2e8f0; border-radius: 8px; padding: 12px; cursor: pointer; }
        .card:hover { border-color: #0066ff; }
        .preview { background: #f8fafc; padding: 8px; border-radius: 4px; margin-bottom: 8px; }
        .name { font-weight: 600; font-size: 14px; }
        .meta { font-size: 12px; color: #64748b; }
        pre { font-size: 11px; background: #1e293b; color: #e2e8f0; padding: 8px; border-radius: 4px; overflow: auto; }
      </style>
    </head>
    <body>
      <p style="font-size:13px;color:#64748b;margin-bottom:12px">${matches.length} componentes para "${query}"</p>
      <div class="grid">
        ${matches.map(c => `
          <div class="card" onclick="selectComponent('${c.name}')">
            <div class="preview">${c.svgPreview}</div>
            <div class="name">${c.name}</div>
            <div class="meta">${c.category} · ${c.variants} variantes</div>
            <pre>${c.exampleUsage}</pre>
          </div>
        `).join('')}
      </div>
      <script>
        function selectComponent(name) {
          window.parent.postMessage({ action: 'insert_component', componentName: name }, '*')
        }
      </script>
    </body>
    </html>
  `

  return createUiResult({ html, sandbox: 'allow-scripts' })
})
```

### Relação com o ecossistema moderno

- **Storybook**: MCP Apps não substitui o Storybook mas pode servir como browsing in-context de componentes dentro do agente
- **Figma MCP**: previews visuais de frames Figma poderiam ser renderizados como MCP Apps
- **Claude Code**: ainda não suporta MCP Apps diretamente — renderização inline requer client com UI
- **Devin Desktop**: suporta MCP Apps nativamente no Agent Command Center
- **Design Tokens**: configuradores visuais de tokens como MCP App mudam o fluxo de ajuste de DS

### Vale a pena acompanhar?

**Sim, vale acompanhar.** MCP Apps muda o patamar do que é possível construir como ferramenta de agente para frontend. Para times que constroem ou mantêm MCP servers, implementar `mcp-ui` deve entrar no backlog.

---

## 2. Google Antigravity 2.0 — Browser Subagent com Chrome Real para Verificação Visual

### O que é

O **Browser Subagent** é o componente mais tecnicamente diferenciado do Google Antigravity 2.0. Diferente de ferramentas como Playwright em modo headless ou simulações de DOM, o Browser Subagent sobe uma instância **real do Chrome** — com estado de sessão, DevTools, e toda a superfície de renderização do browser — e opera a aplicação enquanto o agente a constrói.

O loop técnico é:
1. Agente implementa uma feature (componente, rota, interação)
2. Browser Subagent abre a URL no Chrome real
3. Executa o fluxo de usuário (cliques, inputs, navegação)
4. Captura screenshots de cada estado
5. Coleta erros de console, network failures, layout shifts
6. Reporta de volta para o agente com contexto visual + técnico
7. Agente corrige automaticamente se encontrar problema
8. Ciclo se repete até o comportamento estar correto

O que o diferencia do Playwright comum:
- **Estado de sessão real**: pode autenticar, carregar cookies, testar flows que dependem de estado de usuário
- **Rendering fidelity**: CSS animations, GPU-accelerated transforms, fontes reais — não simulação
- **DevTools access**: acesso ao CDP (Chrome DevTools Protocol) — performance traces, heap snapshots, network waterfall
- **Screenshot diffing**: compara screenshots entre iterações e sinaliza regressões visuais

```ts
// SDK — ativar Browser Subagent
const agent = new Antigravity({
  model: 'gemini-3.5-flash',
  subagents: {
    browser: {
      enabled: true,
      viewport: { width: 1440, height: 900 },
      session: { persist: true }, // mantém cookies entre ciclos
      capture: {
        screenshots: true,
        consoleErrors: true,
        networkRequests: true,
        performanceTrace: false, // ativa para análise de CWV
      },
    },
  },
})
```

### Por que isso importa

O problema fundamental de agentes de IA em frontend era a cegueira visual: o agente gera código que parece correto no TypeScript, passa no type-check, mas quebra visualmente — um elemento fora do lugar, um estado de loading que não termina, um botão que não fica clickable em mobile. Sem fechar esse loop visualmente, o ciclo de debug recai sobre o desenvolvedor.

O Browser Subagent é a primeira implementação production-grade de verificação visual automática integrada ao loop de desenvolvimento. Para frontend, isso é equivalente ao que o CI automático fez para a qualidade de código — mas para o comportamento visual.

### Benefícios práticos

- Agente verifica o resultado visual da própria implementação sem intervenção humana
- Regressões visuais detectadas entre iterações antes de chegar em PR
- Screenshots de cada estado do fluxo documentam o comportamento esperado automaticamente
- Erros de console capturados em contexto real — sem source maps artificiais
- Flows autenticados testáveis — o agente consegue testar features protegidas por login

### Possíveis problemas ou limitações

- **Custo**: cada sessão com Browser Subagent consome substancialmente mais créditos — não é para uso em cada task trivial
- **Tempo de ciclo maior**: sobe Chrome, espera render, captura screenshots — cada ciclo adiciona ~5-15 segundos
- **App precisa rodar localmente**: o Subagent acessa `localhost` — funciona em dev, não em preview remoto sem túnel
- **Flaky em apps com animações**: screenshots são capturados em momento fixo — animações podem causar false positives em diffing
- **Sem suporte a self-hosted ainda**: Browser Subagent requer cloud da Google por agora

### Exemplo prático

```bash
# Tarefa com Browser Subagent ativado
antigravity run \
  --browser-subagent \
  --task "
    Implemente o modal de confirmação de exclusão de item no carrinho.
    - Deve abrir ao clicar no ícone de lixeira
    - Deve mostrar nome do produto
    - Botão 'Cancelar' fecha o modal sem ação
    - Botão 'Confirmar' remove o item e fecha o modal
    Verifique que o fluxo funciona em 1440px e em 375px (mobile)
  " \
  --base-url http://localhost:3000

# Log do Browser Subagent:
# ✓ Modal abre ao clicar na lixeira (1440px) — screenshot capturado
# ✓ Nome do produto exibido corretamente
# ✗ Botão 'Confirmar' não dispara remoção — console.error: "removeItem is not a function"
# → Agente diagnostica: hook useCart não exporta removeItem
# → Agente corrige e re-verifica
# ✓ Fluxo completo funcionando em 1440px
# ✗ Modal overflow em 375px — screenshot mostra botões cortados
# → Agente ajusta padding e max-width do modal
# ✓ Fluxo validado em ambos os viewports
```

### Relação com o ecossistema moderno

- **Playwright**: Browser Subagent e Playwright servem propósitos diferentes — Subagent é para ciclo de desenvolvimento, Playwright é para testes de regressão em CI
- **Storybook**: o Subagent verifica componentes em contexto real de app, não em isolamento — complementar, não substituto
- **Chrome DevTools MCP**: Browser Subagent usa o mesmo CDP internamente — mas integrado ao loop do agente em vez de exposto como MCP tools
- **Core Web Vitals**: com `performanceTrace: true`, o Subagent pode medir LCP, INP e CLS automaticamente
- **Next.js App Router**: verifica Server Components renderizados, hidratação, Suspense boundaries

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Para times que investem em qualidade visual e têm flows complexos de UI, o Browser Subagent é uma contribuição genuína. O custo adicional por sessão se justifica para tasks de implementação de features inteiras, não para edits pontuais.

---

## 3. Gemini 3.5 Flash — Frontier Model para Agentic Coding a 289 tok/s

### O que é

O **Gemini 3.5 Flash** é o modelo frontier atual do Google para tarefas de coding e agentes, com disponibilidade geral desde 19 de maio de 2026. Em termos puramente técnicos para developers, os números mais relevantes:

- **289 tokens por segundo** — 4x mais rápido que modelos comparáveis de outros providers
- **Terminal-Bench 2.1: 76.2%** — benchmark de tarefas de agente em terminal (coding, debugging, scaffolding)
- **MCP Atlas: 83.6%** — benchmark específico para uso de ferramentas MCP
- **GDPval-AA: 1656 Elo** — benchmark de coding assistido
- **1 milhão de tokens de contexto** — processa codebases inteiras
- **Supera Gemini 3.1 Pro** em benchmarks de coding e agentic, com custo menor

**Para uso em frontend specificamente:**

A velocidade de 289 tok/s é relevante quando o modelo é usado para **streaming de UI em produção** — uma feature de chat ou autocomplete que usa Gemini 3.5 Flash entrega texto ao usuário 4x mais rápido que com modelos equivalentes, o que impacta diretamente a percepção de responsividade.

Em **ciclos de agente** (onde o modelo é invocado múltiplas vezes em sequência), a velocidade composta reduz o tempo total de uma task de refatoração de, por exemplo, 8 minutos para ~2 minutos.

**Thinking modes:**
- `none`: resposta direta, menor latência, para tasks simples
- `low`: chain-of-thought mínimo
- `medium`: default — equilíbrio latência/qualidade
- `high`: raciocínio extenso, para arquitetura e análise complexa

### Por que isso importa

O Gemini 3.5 Flash representa uma mudança na curva de custo-performance para agentes de IA em frontend. Anteriormente, usar um modelo frontier para tarefas de agente era caro o suficiente para forçar trade-offs — usar modelos menores para tasks simples, modelos maiores apenas para arquitetura. Com o Flash, a latência cai o suficiente para que um modelo frontier seja viável para o loop completo de desenvolvimento.

O MCP Atlas benchmark é especialmente relevante: 83.6% em uso de ferramentas MCP significa que o modelo é tecnicamente sólido para orquestrar as tools que o ecossistema já construiu.

### Benefícios práticos

- 4x mais rápido: streaming de UI responsivo em produção
- Contexto 1M: analisa monorepos inteiros sem chunking
- MCP Atlas 83.6%: confiança alta em uso de ferramentas MCP
- Thinking modes: ajuste fino entre latência e qualidade por tipo de task
- Supera Gemini 3.1 Pro em coding — migração tem ganho de qualidade E velocidade

### Possíveis problemas ou limitações

- **Preço mais alto**: $1.50/M input tokens vs $0.35 do Gemini 2.5 Flash — 4x mais caro
- **Default thinking mudou**: `high` → `medium` pode reduzir qualidade em tarefas que não configurarem explicitamente
- **Ainda novo**: benchmarks independentes (além dos próprios do Google) precisam validar as claims
- **Rate limits em regiões não-US**: disponibilidade global do GA pode ter delays em Vertex AI fora dos EUA

### Exemplo prático

```ts
// Uso via Vercel AI SDK (integração já disponível)
import { google } from '@ai-sdk/google'
import { streamText } from 'ai'

// Feature de AI chat com streaming responsivo
export async function POST(req: Request) {
  const { messages } = await req.json()

  const result = await streamText({
    model: google('gemini-3.5-flash'),
    messages,
    // Para tasks de código: usar high para melhor qualidade
    providerOptions: {
      google: {
        thinkingConfig: { thinkingLevel: 'high' }, // explícito
      },
    },
  })

  return result.toDataStreamResponse()
}

// Para features de chat em produção: usar medium (default) para latência
// Para tarefas de agente complexas (arquitetura, refatoração): usar high
```

```ts
// Comparação de latência em streaming (estimado)
// Gemini 3.1 Pro: ~72 tok/s → primeira palavra em ~1.4s para resposta de 100 tokens
// Gemini 3.5 Flash: ~289 tok/s → primeira palavra em ~0.35s para resposta de 100 tokens
// Impacto percebido em UI: expressivo — usuário vê texto chegando 4x mais rápido
```

### Relação com o ecossistema moderno

- **Vercel AI SDK**: `@ai-sdk/google` já inclui suporte — `google('gemini-3.5-flash')` funciona hoje
- **Antigravity 2.0**: modelo default do CLI e Browser Subagent
- **LangChain**: `@langchain/google-genai` — requer atualização para novo SDK
- **Next.js Server Actions**: latência baixa viabiliza Server Actions com Gemini em fluxos síncronos
- **Edge Runtime**: 289 tok/s com 1M context — adequado para Edge Functions de IA

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Para times que já usam a API do Gemini, migrar para 3.5 Flash tem ganho duplo de velocidade e qualidade. Para times que usam Claude ou GPT exclusivamente, é um benchmark útil de referência de velocidade.

---

## 4. Svelte ai-tools — Ferramentas Oficiais de IA no Ecossistema Svelte

### O que é

O Svelte team lançou o package `@svelte/ai-tools` como parte dos releases de junho 2026. É o primeiro package oficial do ecossistema Svelte focado especificamente em integração com agentes de IA e ferramentas de desenvolvimento assistido.

O package cobre três superfícies:

**1. Svelte MCP Server melhorado:**
O servidor MCP do Svelte ganhou **stdio mode com leitura direta de arquivos**. Anteriormente, quando um agente precisava do conteúdo de um componente `.svelte` para contexto, eram necessários 3 round trips: (1) listar arquivos, (2) solicitar leitura de arquivo via tool call, (3) receber o conteúdo. Com o stdio mode, o servidor lê o arquivo diretamente e inclui o conteúdo no response em um único round trip.

Para sessões de Claude Code ou Antigravity trabalhando em projetos Svelte, isso reduz visualmente a latência de contexto — especialmente em projetos com muitos componentes.

**2. ai-tools para language server:**
Novas ferramentas expostas pelo `svelte-language-server` que permitem a agentes consultar:
- Props disponíveis em um componente sem abrir o arquivo
- Eventos emitidos por um componente
- Stores acessíveis no contexto atual
- Runes disponíveis e seus types

```ts
// Exemplo de tool disponível via @svelte/ai-tools
// "quais props o componente ProductCard aceita?"
const props = await svelteTools.getComponentProps('src/components/ProductCard.svelte')
// → [{ name: 'product', type: 'Product', required: true }, { name: 'showBadge', type: 'boolean', default: false }]
```

**3. svelte-check melhorado para saída de agente:**
O `svelte-check` agora tem um modo de output `--format agent-json` que estrutura erros e warnings de forma otimizada para consumo por agentes — com contexto de arquivo, linha, tipo de erro e sugestão de correção em um schema consistente.

```bash
npx svelte-check --format agent-json 2>&1
# Output estruturado:
# [{"file":"src/lib/api.ts","line":47,"type":"error","message":"Type 'string' is not assignable to 'number'","suggestion":"Change type annotation to string or coerce with parseInt()"}]
```

### Por que isso importa

O Svelte é o único framework principal com um package oficial dedicado à integração de agentes. Isso sinaliza que o team do Svelte está priorizando a experiência de desenvolvimento com IA como uma feature de primeira classe — não como um caso de uso secundário. Para times Svelte, isso significa menos fricção ao usar agentes de código no dia-a-dia.

A saída estruturada do `svelte-check` para agentes é uma ideia que outros frameworks deveriam copiar — erros de tipo em formato legível por humanos são frequentemente ambíguos para LLMs, enquanto JSON estruturado com `suggestion` é acionável diretamente.

### Benefícios práticos

- MCP stdio: 3x menos round trips para carregar contexto de componentes Svelte
- Language server tools: agentes consultam props/events/stores sem ler arquivos manualmente
- `svelte-check --format agent-json`: erros de tipo diretamente acionáveis por agentes
- Package oficial: mantido pelo Svelte team, não uma solução de terceiro com risco de abandono
- Compatível com Claude Code, Antigravity, Cursor e qualquer cliente MCP

### Possíveis problemas ou limitações

- Novo — pouca documentação e exemplos de integração ainda
- Svelte MCP stdio mode requer configuração local — não funciona out-of-the-box em CI cloud
- Language server tools dependem do LSP estar rodando — overhead em ambientes sem editor
- `--format agent-json` ainda não inclui context de código ao redor do erro — pode ser necessário para erros ambíguos

### Exemplo prático

```json
// .claude/mcp.json — configuração do Svelte MCP com stdio mode
{
  "mcpServers": {
    "svelte": {
      "command": "npx",
      "args": ["@svelte/mcp", "--stdio"],
      "env": {
        "SVELTE_ROOT": "${workspaceFolder}"
      }
    }
  }
}
```

```bash
# Ciclo de verificação de tipos em sessão de agente
# Antes: agente roda svelte-check, parse manual do output
# Depois:
npx svelte-check --format agent-json | jq '.[] | select(.type == "error")'

# Output direto para o agente:
# {
#   "file": "src/routes/+page.svelte",
#   "line": 23,
#   "type": "error",
#   "message": "Property 'id' does not exist on type 'Product'",
#   "suggestion": "Add 'id: string' to the Product interface in src/lib/types.ts"
# }
```

### Relação com o ecossistema moderno

- **SvelteKit**: tools do language server funcionam com rotas, load functions e server endpoints de SvelteKit
- **Vite**: integração com `vite-plugin-svelte` — erros de build também exportáveis em `agent-json`
- **Turborepo**: `svelte-check` em cada package do monorepo com output unificado para agente
- **Claude Code**: configuração MCP com stdio mode diretamente compatível
- **CI/CD**: `svelte-check --format agent-json` pode alimentar pipelines de fix automático

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Para times Svelte, é um package obrigatório na toolchain de agentes. Para times avaliando Svelte, é um diferencial — o framework investe em DX para workflows com IA de forma estrutural.

---

## 5. Autonoma — Component Contract Testing por IA a Partir de Comportamento Real

### O que é

O **Autonoma** resolve um problema que se tornou mais evidente com a maturação do ecossistema de testes frontend em 2026: **stories manuais ficam desatualizadas**. O padrão de manter Storybook stories como fixtures de teste funciona enquanto o time é disciplinado em atualizá-las, mas em codebases com dezenas de componentes em evolução contínua, a divergência entre stories e realidade é o estado padrão, não a exceção.

A abordagem do Autonoma é diferente na origem: em vez de especificar o comportamento esperado manualmente, ele **observa o comportamento real** dos componentes — em produção, em sessões de staging, ou em sessões guiadas de QA — e gera contratos de componente automaticamente.

**Como funciona tecnicamente:**

```
1. Instrumentação: Autonoma SDK instrumenta os componentes em produção
   → Captura props, estados, outputs em cada render (anonimizado)

2. Observação: coleta N sessões de usuário real
   → Identifica patterns de uso: quais combinações de props ocorrem
   → Identifica invariantes: o que sempre é verdadeiro quando X é verdadeiro

3. Geração de contratos: traduz patterns em test specs
   → "Quando price < originalPrice, badge com % de desconto deve existir"
   → "Quando isLoading é true, skeleton deve existir e botão não deve ser clickable"

4. Execução: contratos rodam via Playwright CT em real browser
   → Fidelidade completa, sem jsdom
   → CI/CD: falha se o comportamento observado diverge do contrato
```

**Portabilidade com o 2026 standard (Playwright + Portable Stories):**

Os contratos gerados pelo Autonoma são exportáveis como Portable Stories — compatíveis com Playwright CT diretamente. Isso significa que times podem usar Autonoma para gerar e Playwright CT para executar, com o mesmo runner de CI.

```ts
// autonoma.config.ts
export default {
  observe: {
    source: 'production',
    sampleRate: 0.05, // 5% das sessões
    anonymize: {
      fields: ['email', 'name', 'cpf', 'phone'],
      strategy: 'hash', // irreversível, preserva padrões
    },
    minSessions: 100, // gera contrato com pelo menos 100 observações
  },
  output: {
    format: 'portable-stories',
    runner: 'playwright',
    dir: './src/__contracts__',
    updateStrategy: 'append', // adiciona novos patterns, não sobrescreve
  },
}
```

### Por que isso importa

O shift de "escreva o teste descrevendo o comportamento esperado" para "observe o comportamento real e gere o teste" é conceitualmente parecido com o que o contract testing (Pact) fez para microsserviços. A diferença é que o Autonoma aplica isso para o nível de componente no browser.

O benefício mais direto: zero custo de manutenção de fixtures. Quando um componente evolui, o Autonoma atualiza os contratos automaticamente — sem PR de "atualizar stories desatualizadas".

O segundo benefício: **o que não é testado está visível**. Componentes sem observações suficientes têm contratos fracos — o Autonoma sinaliza quais partes do codebase carecem de cobertura real.

### Benefícios práticos

- Zero stories para escrever ou manter
- Contratos refletem uso real — não suposições do dev que escreveu a story
- `updateStrategy: 'append'` garante que novos patterns de uso viram novos testes automaticamente
- Cobertura baseada em uso real — prioriza o que os usuários realmente fazem
- Integra com CI/CD nativo via Playwright CT

### Possíveis problemas ou limitações

- Componentes novos têm zero observações — bootstrap requer sessão de QA guiada
- Privacidade: dados de produção precisam de anonimização robusta — configurar com atenção a LGPD/GDPR
- Padrões raros de produção podem criar contratos que são difíceis de reproduzir em CI (depende de estado específico)
- A plataforma é SaaS — dados de uso ficam no servidor do Autonoma (verificar política de dados antes de adotar)
- Contratos gerados podem ser verbosos para componentes com muitas variantes — ruído em diff de PR

### Exemplo prático

```ts
// Contrato gerado automaticamente para ProductCard
// src/__contracts__/ProductCard.contract.ts

import { test, expect } from '@playwright/test'
import { composeStory } from '@storybook/react'
import * as stories from '../components/ProductCard.stories'

// Contrato 1: produto com desconto (observado em 847 sessões)
test('ProductCard — com badge de desconto', async ({ mount }) => {
  const Story = composeStory(stories.WithDiscount, {
    product: {
      name: 'Tênis Running Pro',
      price: 299.9,
      originalPrice: 399.9,
    },
  })

  const component = await mount(<Story />)

  // Invariantes observados:
  await expect(component.getByTestId('discount-badge')).toBeVisible()
  await expect(component.getByTestId('discount-badge')).toContainText('%')
  await expect(component.getByText('299')).toBeVisible()
  await expect(component.getByText('399')).toHaveCSS('text-decoration', /line-through/)
})

// Contrato 2: produto out of stock (observado em 312 sessões)
test('ProductCard — out of stock', async ({ mount }) => {
  const component = await mount(
    <ProductCard product={{ ...baseProduct, stock: 0 }} />
  )

  await expect(component.getByRole('button', { name: /comprar/i })).toBeDisabled()
  await expect(component.getByText(/indisponível/i)).toBeVisible()
})

// Contrato gerado: não precisou de nenhuma story manual
```

### Relação com o ecossistema moderno

- **Playwright CT**: executa contratos gerados nativamente — 2026 standard
- **Storybook**: Portable Stories exportadas pelo Autonoma são compatíveis — pode coexistir para documentação
- **Sentry/Datadog RUM**: instrumentação pode ser alimentada pelo SDK de observability existente, reduzindo overhead
- **Vitest**: contratos também exportáveis como Vitest specs para feedback mais rápido em dev local
- **Next.js/React**: agnóstico de framework — instrumenta qualquer componente React/RSC
- **CI/CD (GitHub Actions)**: contratos rodam via `playwright test` padrão — zero configuração adicional

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Para times com dívida técnica em stories desatualizadas ou sem cobertura de componentes, o Autonoma endereça a causa raiz. A abordagem de contrato derivado de uso real é mais robusta a longo prazo do que specs estáticas. Avaliar a política de privacidade de dados antes de adotar em produção.
