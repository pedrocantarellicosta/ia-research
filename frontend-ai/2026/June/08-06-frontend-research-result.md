# Front-End AI Weekly — Novidades Front-End | 08 de Junho de 2026

> **Data de geração:** 08 de Junho de 2026
> **Período coberto:** 01–08 de Junho de 2026
> **Arquivo:** 10 Novidades Front-End
> **Curadoria:** Skill `frontend-ia-weekly` via Claude Sonnet 4.6

---

## 1. Next.js 16 — Agent DevTools Experimental e Browser Log Forwarding

### O que é

O Next.js seguiu expandindo as capacidades de desenvolvimento orientado a agentes nesta semana com dois recursos que se complementam ao `AGENTS.md` lançado na 16.2 (coberto na semana passada). O foco agora está em fechar o loop de observabilidade durante o desenvolvimento.

**Browser Log Forwarding** (`logging.browserToTerminal`): todo output de console do browser — `console.log`, `console.warn`, erros não tratados, stack traces — é capturado e re-emitido no terminal durante o `next dev`. Para agentes de IA rodando um dev server, isso é transformador: o agente não precisa mais de acesso ao browser para ver o que está acontecendo no lado do cliente. Cada entrada é estruturada como:

```json
{
  "timestamp": "2026-06-07T14:22:03.412Z",
  "level": "error",
  "message": "TypeError: Cannot read properties of undefined (reading 'map')",
  "stack": "at ProductList (src/components/ProductList.tsx:34:18)"
}
```

**Experimental Agent DevTools**: um servidor de ferramentas expostas via terminal que permite a agentes acessar o estado interno do Next.js dev server — rotas registradas, cache status, erros de build, Server Components rendering tree. É basicamente um painel de debug sem UI, consumível por qualquer processo que leia stdout estruturado.

```ts
// next.config.ts
const nextConfig = {
  logging: {
    browserToTerminal: true,
    level: 'error',
  },
  experimental: {
    agentDevTools: true,
  },
}
```

### Por que isso importa

O fluxo de debug com agentes até aqui exigia intervenção humana entre cada iteração: agente escreve código → dev abre o browser → vê o erro → copia → cola no agente. O Browser Log Forwarding elimina esse bottleneck para erros de runtime no cliente. O agente pode rodar o dev server, aguardar logs estruturados no stdout e agir diretamente — sem intermediário.

### Benefícios práticos

- Agentes corrigem erros de runtime de cliente sem precisar de browser access
- Logs chegam source-mapped — o agente vê o arquivo e linha TypeScript, não o bundle minificado
- Agent DevTools expõe o estado do Next.js dev server como dados estruturados
- Compatível com Claude Code, Antigravity, Cursor Composer e qualquer agente que consuma stdout
- Funciona em conjunto com o `AGENTS.md` e docs empacotadas — compõe o ecossistema agent-ready da Vercel

### Possíveis problemas ou limitações

- `agentDevTools` ainda é experimental — API pode mudar entre versões canary
- Browser Log Forwarding tem overhead de rede mínimo mas perceptível em apps com logging muito verboso
- Não captura logs de Service Workers ou Web Workers — apenas o main thread do browser
- Requer `next@16.x` — projetos em versões antigas não têm suporte

### Exemplo prático

```ts
// Fluxo de um agente (Claude Code) usando Browser Log Forwarding

// 1. Agente inicia dev server com logs estruturados:
// $ next dev --turbopack 2>&1 | tee dev.log

// 2. Agente edita ProductList.tsx e aguarda logs no stdout

// 3. Agente recebe via stdout:
// [browser:error] TypeError: Cannot read properties of undefined (reading 'map')
// at ProductList (src/components/ProductList.tsx:34:18)

// 4. Agente lê src/components/ProductList.tsx:34 diretamente
// 5. Agente corrige o null check
// 6. Ciclo se repete automaticamente

// next.config.ts
export default {
  logging: {
    browserToTerminal: true,
  },
  experimental: {
    agentDevTools: true,
  },
}
```

### Relação com o ecossistema moderno

- **Next.js App Router**: Agent DevTools expõe a rendering tree dos Server Components — útil para debugar waterfall de dados
- **Turbopack**: logs de build chegam com mesmo formato estruturado
- **Claude Code / Cursor**: ambos consomem stdout nativamente — integração zero-config
- **Vercel Deploy**: logs de preview deployment também se beneficiam do mesmo formato
- **Monorepos com Turborepo**: cada app pode rodar seu dev server com log forwarding, agente centraliza

### Vale a pena acompanhar?

**Sim, vale acompanhar.** A combinação de `AGENTS.md`, docs empacotadas e agora Browser Log Forwarding + Agent DevTools forma uma stack coesa de "framework agent-ready". Equipes que investem em automação de desenvolvimento com agentes ganham um ciclo de feedback completo sem configuração adicional.

---

## 2. Svelte 5.56.0 — Template Declarations, TypeScript 6.0 e MCP stdio Improvements

### O que é

O blog oficial "What's new in Svelte: June 2026" traz três atualizações técnicas relevantes, lançadas ao longo da semana:

**Template Declarations (`svelte@5.56.0`)**: agora é possível declarar variáveis diretamente no markup do template, eliminando a necessidade de subir lógica derivada para o `<script>` apenas para mantê-la acessível no template. A feature resolve um ponto de fricção real: quando você precisa de um valor calculado apenas uma vez no template, você era forçado a declarar no `<script>` mesmo sem precisar de reatividade.

```svelte
<!-- Antes: era necessário ir ao <script> -->
<script lang="ts">
  const truncated = title.slice(0, 50) + '...'
</script>

<!-- Agora: declaration diretamente no markup -->
{@const truncated = title.slice(0, 50) + '...'}
<h2>{truncated}</h2>
```

**TypeScript 6.0** no `svelte-language-server`, `svelte2tsx` e `svelte-check`: a linguagem ganhou inferência genérica melhorada, narrowing mais preciso em condicionais e `using` declarations (Explicit Resource Management). O suporte completo no language server significa que IDEs passam a tipar corretamente patterns que antes exigiam casts manuais.

**Svelte MCP — stdio mode com leitura direta de arquivos**: o `@svelte/mcp` agora pode ler conteúdo de arquivo diretamente em modo stdio, sem precisar fazer round-trips via tool calls ao sistema de arquivos. Em workflows locais com Claude Code ou Antigravity, isso reduz de 3 para 1 o número de chamadas MCP necessárias para fornecer contexto de um componente ao agente.

**vite-plugin-svelte**: o optimizer agora está habilitado também para ambientes de servidor durante o desenvolvimento, o que reduz o tempo de warm-up em apps SvelteKit com muitas dependências.

### Por que isso importa

Template declarations resolvem um antipadrão muito comum em Svelte: código de apresentação no `<script>` que não precisa ser reativo. Com a feature, a coerência e legibilidade dos componentes melhora, e agentes de IA geram menos código desnecessário ao analisar o template.

A chegada do TypeScript 6.0 no language server não é trivial — `using` declarations, por exemplo, permitem gerenciar recursos como connections e streams com garantia de cleanup, pattern relevante para componentes que lidam com WebSocket ou media streams.

### Benefícios práticos

- Template declarations reduzem ruído no `<script>` e melhoram coerência entre lógica e markup
- TypeScript 6.0: narrowing mais preciso elimina casts desnecessários e reduz erros sutis de tipo
- `using` declarations: cleanup automático de recursos sem `onDestroy` manual
- MCP stdio: menos round trips em sessões de agente — ciclos mais rápidos
- vite-plugin-svelte optimizer em server: warm-up mais rápido em SvelteKit com dependências pesadas

### Possíveis problemas ou limitações

- `{@const}` no markup pode ser abusado — valores derivados complexos ainda devem ir ao `<script>` para testabilidade
- TS 6.0 pode quebrar tipos em projetos com `strict: false` que dependiam de comportamento de narrowing antigo
- MCP stdio ainda requer configuração local — não funciona em ambientes de cloud CI sem ajuste

### Exemplo prático

```svelte
<script lang="ts">
  // using declaration — TS 6.0, cleanup automático
  using ws = new WebSocket('wss://api.example.com/live')

  let { items } = $props<{ items: Product[] }>()
</script>

<!-- Template declarations — 5.56.0 -->
{#each items as item}
  {@const isOnSale = item.price < item.originalPrice}
  {@const discount = Math.round((1 - item.price / item.originalPrice) * 100)}

  <div class:sale={isOnSale}>
    <span>{item.name}</span>
    {#if isOnSale}
      <badge>{discount}% OFF</badge>
    {/if}
  </div>
{/each}
```

### Relação com o ecossistema moderno

- **SvelteKit**: vite-plugin-svelte server optimizer melhora DX em projetos SvelteKit de médio/grande porte
- **Vite 7**: integração oficial mantém compatibilidade com a versão mais recente do bundler
- **Claude Code / Antigravity**: MCP stdio mode reduz latência em loops de geração/verificação de componentes
- **Monorepos**: svelte-check com TS 6.0 roda mais preciso em workspaces com tipos compartilhados
- **Design Systems em Svelte**: template declarations facilitam componentes de apresentação sem estado reativo desnecessário

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O Svelte tem entregado features de DX consistentes que reduzem boilerplate sem adicionar mágica. As melhorias MCP apontam para um ecossistema que está se posicionando conscientemente para workflows com agentes.

---

## 3. Vercel AI SDK — xAI Image Search, Gemini Embedding 2 e Deep Research Preview

### O que é

O Vercel AI SDK lançou múltiplas atualizações de patch na semana de 01–08 de junho focadas em expandir as capacidades de ferramentas de agentes e adicionar novos modelos de embedding e pesquisa.

**xAI Web Search com Image Search**: `xai.tools.webSearch()` agora aceita `enableImageSearch: true`, que envia `enable_image_search: true` para a API da xAI. Isso significa que ferramentas de busca em agentes front-end podem agora retornar resultados visuais estruturados — relevante para agentes que precisam buscar referências de UI, capturas de tela de produtos ou imagens de design.

```ts
import { xai } from '@ai-sdk/xai'
import { generateText } from 'ai'

const { text } = await generateText({
  model: xai('grok-3'),
  tools: {
    search: xai.tools.webSearch({
      enableImageSearch: true,
    }),
  },
  prompt: 'Quais são os padrões de UI mais usados em dashboards SaaS em 2026?',
})
```

**Gemini Embedding 2**: suporte ao novo modelo `text-embedding-3` do Google, com melhor performance semântica em tarefas de similaridade de código e documentação técnica. Para use cases de RAG em frontend (busca em design systems, busca em componentes, busca em documentação), isso representa uma melhoria mensurável de recall.

**Deep Research Models Preview**: suporte inicial a modelos de Deep Research da Google e Vertex AI — modelos que fazem múltiplas buscas internas e sintetizam uma resposta fundamentada. Útil para agentes que precisam analisar tradeoffs arquiteturais ou fazer comparação de bibliotecas com profundidade.

**MCP ping handling fix**: o SDK agora retorna um resultado JSON-RPC vazio correto em vez de erro no ping MCP, corrigindo um bug que causava reconexões desnecessárias em alguns clientes MCP.

### Por que isso importa

O Vercel AI SDK é a camada de abstração mais usada para integrar IA em projetos React e Next.js. Cada expansão de capabilities — novos modelos, novas ferramentas, correções de protocolo — chega nativamente para todos os projetos que já usam o SDK, sem mudança de arquitetura.

### Benefícios práticos

- Image search habilita agentes a referenciar e comparar UI visual, não só texto
- Gemini Embedding 2 melhora RAG sobre código e documentação técnica
- Deep Research: agentes passam a poder fazer análises fundamentadas sem prompt-chaining manual
- MCP ping fix elimina reconexões desnecessárias em servidores MCP locais
- Tudo via `npm update ai` — zero mudança de arquitetura para projetos existentes

### Possíveis problemas ou limitações

- Image search retorna URLs de imagens — o processamento visual ainda requer um modelo multimodal
- Gemini Embedding 2 tem custo diferente do modelo anterior — revisar billing em produção
- Deep Research models são preview — latência alta e rate limits conservadores
- `deprecates searchParameters` em agent tools quebra código existente que usava o parâmetro antigo

### Exemplo prático

```ts
// RAG sobre design system com Gemini Embedding 2
import { google } from '@ai-sdk/google'
import { embed } from 'ai'

// Indexar componentes do design system
const { embedding } = await embed({
  model: google.textEmbeddingModel('text-embedding-003'),
  value: 'Componente Button primário com estado de loading e ícone à esquerda',
})

// Buscar componente mais similar
const results = await vectorStore.query(embedding, { topK: 5 })
// → retorna Button, IconButton, LoadingButton, etc.
```

### Relação com o ecossistema moderno

- **Next.js App Router**: Deep Research models funcionam bem em Server Actions para análises sob demanda
- **React**: hooks do AI SDK (`useChat`, `useCompletion`) recebem os benefícios automaticamente
- **Edge Runtime**: embedding calls são compatíveis com Edge Functions na Vercel
- **Design Systems**: Gemini Embedding 2 melhora busca semântica em documentações de componentes
- **MCP**: fix de ping estabiliza conexões com servidores locais como `@svelte/mcp` e `chrome-devtools-mcp`

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O SDK continua sendo o ponto de entrada mais consistente para IA em projetos Vercel/Next. As atualizações desta semana são incrementais mas somam capacidades relevantes para agentes em frontend.

---

## 4. Figma MCP Server + Code Connect — Design-to-Code com Tokens Reais

### O que é

O Figma MCP Server (agora em beta público) é um servidor MCP oficial da Figma que transforma como agentes de IA consomem contexto de design. Em vez de o desenvolvedor exportar specs, copiar valores e colar no prompt, o agente conecta ao Figma diretamente e extrai o que precisa.

O servidor resolve um problema central: o Figma expõe dados de design brutos via REST API — posições absolutas de pixel, hex codes, opacidades isoladas. Esses dados são inúteis para um LLM que precisa gerar código. O MCP server transforma isso em contexto semântico:

- Posições de pixel → relações de layout (`centered inside parent`, `gap: 16px`)
- Hex codes → referências de design token (`color.brand.primary`)
- Layers aninhados → hierarquia de componentes flat
- Variáveis Figma → tokens CSS/JS equivalentes

**Code Connect** é a feature que muda o jogo: você mapeia explicitamente cada componente Figma para o componente real do seu codebase. Quando o agente pede contexto de um frame Figma, o servidor já sabe que `Button/Primary` no Figma é `<Button variant="primary">` no seu projeto.

```ts
// figma.connect.ts
import { figma } from '@figma/code-connect'
import { Button } from '@/components/ui/Button'

figma.connect(Button, 'https://figma.com/file/...?node-id=...', {
  props: {
    variant: figma.enum('Variant', {
      Primary: 'primary',
      Secondary: 'secondary',
      Destructive: 'destructive',
    }),
    disabled: figma.boolean('Disabled'),
    label: figma.string('Label'),
  },
  example: ({ variant, disabled, label }) => (
    <Button variant={variant} disabled={disabled}>{label}</Button>
  ),
})
```

Com Code Connect ativo, o agente gera `<Button variant="primary">Salvar</Button>` em vez de `<button class="bg-blue-600 px-4 py-2 rounded text-white">Salvar</button>`.

### Por que isso importa

O problema de agentes que "sabem Tailwind mas não sabem seu design system" é um dos mais frustrantes em times que têm um DS consolidado. O Figma MCP resolve isso de forma estrutural — não com prompts melhores, mas com contexto correto na fonte.

O impacto vai além de geração de código: revisões de design viram verificáveis. Um agente pode comparar o componente renderizado com a spec do Figma programaticamente.

### Benefícios práticos

- Agentes geram código usando os componentes reais do seu DS, não reinventando a roda
- Code Connect elimina a camada de tradução manual entre design e código
- Tokens Figma chegam como referências semânticas — output CSS usa os mesmos tokens
- Compatível com Claude Code, Cursor, Copilot no VS Code e Windsurf/Devin Desktop
- Reduz divergência design/código estruturalmente, não via convenção

### Possíveis problemas ou limitações

- Ainda em beta — algumas features de layout complexas não mapeiam perfeitamente
- Code Connect requer setup manual por componente — custo inicial proporcional ao tamanho do DS
- Requer Dev Mode ativo no Figma — feature de planos pagos
- Dados de design expostos ao agente local — verificar políticas de segurança para designs sob NDA
- Qualidade do output depende diretamente da qualidade do DS no Figma (componentes mal organizados = contexto ruim)

### Exemplo prático

```bash
# 1. Instalar e configurar o servidor MCP
npx figma-mcp install

# 2. No cursor/claude/windsurf, referenciar o frame:
# "Implemente este screen: figma.com/file/ABC?node-id=123"

# 3. O agente recebe automaticamente:
# - Layout relacional (flex, gap, padding)
# - Tokens de cor, tipografia, espaçamento
# - Mapeamento Code Connect → componentes reais
# - Variantes e estados do componente

# 4. Output do agente (com Code Connect):
# <ProductCard
#   title={product.name}
#   price={product.price}
#   image={product.imageUrl}
#   badge={product.isOnSale ? 'Sale' : undefined}
# />

# Sem Code Connect, o agente geraria CSS inline ou Tailwind hardcoded
```

### Relação com o ecossistema moderno

- **React/Vue/Svelte**: Code Connect funciona com qualquer framework — a conexão é no nível do componente
- **Design Systems (Radix, Shadcn, daisyUI)**: componentes do DS podem ter seus próprios `figma.connect` entries
- **Storybook**: stories podem ser geradas a partir do mapeamento Code Connect
- **Turborepo/Monorepos**: `packages/ui` mapeia uma vez, todos os apps se beneficiam
- **CI/CD**: Figma Code Connect CLI pode rodar em CI para detectar divergências design/código

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Para times com design systems consolidados e Figma como ferramenta principal, isso é investimento estrutural. O custo de setup do Code Connect se paga rapidamente em qualidade de código gerado por agentes.

---

## 5. Vercel json-render — Framework de UI Generativa Estruturada

### O que é

O `json-render` da Vercel atingiu mais de 13.000 estrelas no GitHub em poucos meses desde o lançamento em early 2026. A premissa é simples e poderosa: em vez de pedir a um LLM que gere texto explicando um dado, você pede que ele retorne uma **estrutura de UI tipada em JSON** que um renderer React converte em interface real.

A diferença central em relação ao AI SDK com streaming: o `json-render` não gera texto. Ele instrui o LLM a retornar um JSON Schema de UI — `DataGrid`, `Form`, `Chart`, `Card`, `Timeline` — e o renderer converte isso em componentes React renderizáveis. O LLM age como um **gerador de especificação de UI**, não como um gerador de HTML ou JSX.

```ts
import { generateUI } from 'json-render'
import { Renderer } from 'json-render/react'

// Pedir UI ao modelo
const ui = await generateUI({
  prompt: 'Mostre as métricas de performance do último mês para o usuário',
  context: { metrics: analyticsData },
  components: ['DataGrid', 'LineChart', 'StatCard'],
})

// Renderizar no React
export default function Dashboard() {
  return <Renderer spec={ui} components={customComponents} />
}
```

O resultado é uma `LineChart` com os dados de analytics já preenchidos, gerada dinamicamente — não um texto descrevendo os dados.

### Por que isso importa

A maioria das integrações de IA em frontend gera texto (chat) ou código (geração de componentes em build time). O `json-render` abre uma terceira via: **UI dinâmica gerada em runtime** a partir do contexto do usuário, constrangida a um conjunto definido de componentes. Isso resolve um problema real em produtos B2B — dashboards e relatórios que precisam variar por usuário, sem reinventar a wheel de geração de UI toda vez.

### Benefícios práticos

- UI gerada em runtime a partir do contexto real do usuário
- Output constrangido a componentes predefinidos — sem risco de injeção de HTML arbitrário
- Componentes customizáveis — o renderer aceita seu próprio DS
- JSON Schema versionável — specs de UI podem ser armazenadas e reproduzidas
- Compatível com SSR e Server Components — spec gerada no servidor, renderização no cliente

### Possíveis problemas ou limitações

- Custo por request de LLM — cada UI gerada custa tokens; caching de specs é essencial
- Conjunto de componentes limitado ao que o schema suporta — UIs muito customizadas não funcionam
- SEO limitado — conteúdo gerado em runtime não é indexável por padrão
- Biblioteca ainda jovem — breaking changes prováveis antes de v1.0
- Complexidade de depuração: "o LLM gerou a spec errada" vs "o renderer tem um bug" requer tooling específico

### Exemplo prático

```ts
// Relatório de vendas dinâmico por usuário
const dashboardSpec = await generateUI({
  model: 'claude-sonnet-4-6',
  prompt: `
    Gere um dashboard de performance para ${user.name}.
    Destaque os produtos com maior margem e sinalize meses abaixo da meta.
  `,
  context: {
    sales: user.salesData,
    targets: user.targets,
  },
  components: ['DataGrid', 'BarChart', 'AlertBadge', 'StatCard'],
  schema: {
    maxComponents: 6,
    layout: 'grid-2col',
  },
})

// Spec resultante (exemplo):
// {
//   layout: 'grid-2col',
//   components: [
//     { type: 'StatCard', title: 'Receita Total', value: 'R$ 142k', trend: '+12%' },
//     { type: 'BarChart', title: 'Top Produtos', data: [...] },
//     { type: 'AlertBadge', level: 'warning', message: 'Março abaixo da meta' },
//   ]
// }
```

### Relação com o ecossistema moderno

- **React/Next.js**: renderer nativo, Server Actions para geração server-side
- **Shadcn/ui**: componentes do DS podem ser registrados no renderer
- **Storybook**: specs JSON podem ser usadas como stories
- **Edge Runtime**: geração leve de specs funciona em Edge Functions
- **SSR/Server Components**: spec gerada no servidor, Renderer hidratado no cliente

### Vale a pena acompanhar?

**Promissor para empresas.** O caso de uso B2B — dashboards e relatórios dinâmicos por contexto de usuário — é forte. Para produtos de consumo ou landing pages estáticas, o overhead não compensa.

---

## 6. Puck AI — Page Builder Generativo com React Components Reais

### O que é

O Puck AI transforma um problema recorrente de CMS headless: gerar páginas via IA que usam seus componentes React reais — não mockups, não HTML genérico, não Tailwind inventado. O AI usa os componentes que já existem no seu codebase como os building blocks da geração.

O fluxo é diferente de ferramentas como v0 ou Lovable: em vez de gerar JSX novo, o Puck AI recebe um schema dos seus componentes existentes (tipo, props, defaults) e instrui o LLM a compor páginas usando apenas esses componentes. O output é uma spec JSON que o Puck renderiza como JSX real.

```tsx
// Registrar seus componentes no Puck
const puckConfig = {
  components: {
    HeroSection: {
      fields: {
        title: { type: 'text' },
        subtitle: { type: 'text' },
        ctaLabel: { type: 'text' },
        ctaHref: { type: 'text' },
        backgroundImage: { type: 'external', fetchList: fetchImages },
      },
      render: ({ title, subtitle, ctaLabel, ctaHref, backgroundImage }) => (
        <HeroSection {...{ title, subtitle, ctaLabel, ctaHref, backgroundImage }} />
      ),
    },
    // ... outros componentes do seu DS
  },
}

// IA gera a página usando esses componentes
const page = await puck.ai.generate({
  prompt: 'Crie uma landing page para um produto de B2B SaaS de analytics',
  config: puckConfig,
})
```

### Por que isso importa

O problema de "o AI gerou uma página bonita mas não usa nenhum dos meus componentes" é real em times com design systems consolidados. O Puck AI inverte isso: a IA não tem autonomia de criar novos componentes — ela só pode compor os que você definiu. O resultado é sempre consistente com o DS da empresa.

### Benefícios práticos

- Output são componentes reais — sem divergência de DS
- Não-técnicos podem gerar landing pages dentro dos limites definidos pelo time de desenvolvimento
- Schema de componentes funciona como documentação viva — o que está no Puck está disponível para IA
- Editável pós-geração — o Puck tem drag-and-drop nativo
- Open source — sem lock-in de plataforma

### Possíveis problemas ou limitações

- Qualidade da geração depende da riqueza do schema de componentes — DS pobres em metadados geram páginas fracas
- Não é para uso geral de AI coding — é específico para page building com conjunto fixo de componentes
- Integração com CMS headless (Contentful, Sanity) requer setup adicional
- Curva de aprendizado para configurar o schema de componentes corretamente

### Exemplo prático

```bash
# Setup básico
npm install @measured/puck @measured/puck-ai

# Configurar componentes, rodar AI generation
# Output: spec JSON → renderiza no editor Puck → publica no CMS
```

```tsx
// Editor com AI generation ativado
import { Puck } from '@measured/puck'
import { PuckAI } from '@measured/puck-ai'

export default function Editor() {
  return (
    <Puck config={puckConfig} data={initialData}>
      <PuckAI
        apiKey={process.env.ANTHROPIC_API_KEY}
        model="claude-sonnet-4-6"
      />
    </Puck>
  )
}
```

### Relação com o ecossistema moderno

- **Next.js**: integração nativa como route handler para o editor
- **Design Systems**: qualquer componente React registrável — Radix, Shadcn, custom
- **CMS Headless**: Sanity, Contentful, Strapi — Puck salva como JSON portável
- **Turborepo**: `packages/ui` exporta componentes que `apps/editor` registra no Puck
- **Server Components**: pages geradas podem ser renderizadas via RSC

### Vale a pena acompanhar?

**Promissor para empresas.** Especialmente para times que precisam dar autonomia de criação de conteúdo para marketing/produto sem abrir mão da consistência do DS. Para apps complexos ou B2B, o caso é mais fraco.

---

## 7. AI Observability Gap — Research Report: 89% Usam IA, Só 8% em Observability

### O que é

O Embrace.io publicou um research report com dados de 300 times de engenharia front-end sobre o estado da observabilidade com IA em 2026. Os números revelam uma assimetria expressiva: a grande maioria dos times já integrou IA no fluxo de desenvolvimento, mas quase nenhum usa IA na observabilidade da própria aplicação em produção.

**Números centrais:**
- 89% dos times front-end usam IA no desenvolvimento (geração de código, debugging, revisão)
- Apenas 8% aplicam IA ativamente em observabilidade (RUM, error tracking, anomaly detection)
- 74% dos times estão nos níveis 2-3 de maturidade em observabilidade — instrumentação parcial, sem visão end-to-end
- 69% usam RUM (Real User Monitoring) e Network/Performance Tracing — os mais adotados
- 74% dos IT leaders consolidariam em uma única plataforma se ela atendesse suas necessidades

**O que a IA pode fazer em observability que não está sendo aproveitado:**
1. **Anomaly detection proativo**: em vez de alerts baseados em thresholds fixos, ML detecta desvios do padrão histórico
2. **Root cause analysis automático**: correlação entre erros, network requests e mudanças de deploy — sem análise manual
3. **Sugestões de fix contextualizadas**: o monitor vê o erro + o componente + o contexto de usuário e sugere a correção
4. **Alert correlation**: agrupa alerts relacionados em incidentes com provável causa raiz, reduzindo alert fatigue

### Por que isso importa

Times front-end investem muito em melhorar o ciclo de desenvolvimento com IA mas deixam produção sem o mesmo nível de intelligence. O gap entre "IA no dev" e "IA em prod" representa uma janela de oportunidade estratégica — especialmente porque os dados de RUM que já existem na maioria dos projetos são a matéria-prima ideal para modelos de anomaly detection.

### Benefícios práticos

- RUM + AI: detecção proativa de regressões de performance antes de atingir todos os usuários
- Correlação automática entre deploys e erros — "esse erro apareceu 3 minutos após o deploy de X"
- Dashboards de anomalias baseados em padrão histórico, não em thresholds manuais
- Redução de alert fatigue em projetos com muitos componentes e routes

### Possíveis problemas ou limitações

- Maioria das ferramentas de AI observability ainda é voltada para backend/infra, não frontend específico
- False positives em anomaly detection em apps com tráfego muito variável (e-commerce em datas especiais, por ex.)
- Custo de retenção de dados de RUM cresce com a granularidade necessária para ML
- Times pequenos têm dificuldade de justificar o custo de plataformas de observability avançadas

### Exemplo prático

```ts
// Grafana Frontend Observability + Faro SDK (com AI insights)
import { initializeFaro, getWebInstrumentations } from '@grafana/faro-web-sdk'

initializeFaro({
  url: 'https://faro-collector.grafana.net/collect/...',
  app: {
    name: 'web-fm-seo-br',
    version: '1.4.2',
    environment: 'production',
  },
  instrumentations: [
    ...getWebInstrumentations({
      captureConsole: true,
      enablePerformanceInstrumentation: true,
    }),
  ],
})

// Com AI insights ativo no Grafana Cloud:
// - Anomalias de LCP detectadas automaticamente
// - Correlação: spike de CLS no checkout → deploy da v1.4.1 → componente PaymentForm
// - Alert agrupado: "5 errors relacionados a CartProvider nos últimos 15 min"
```

### Relação com o ecossistema moderno

- **Core Web Vitals**: LCP, CLS, INP monitorados com baseline histórico e anomaly detection
- **Next.js**: instrumentação automática via `@vercel/speed-insights` com exportação para Grafana
- **Sentry**: evoluiu para AI-assisted error triage — agrupa errors similares e prioriza por impacto
- **Datadog RUM**: AI-powered anomaly detection disponível para frontend metrics
- **OpenTelemetry**: padrão emergente para correlacionar front com backend no mesmo trace

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O gap identificado é real e os dados do report justificam investimento. Para times com tráfego relevante, o ROI de detecção proativa de regressões é direto — cada bug de performance em produção tem custo mensurável em conversão.

---

## 8. Module Federation 3.0 + Native ESM Federation — Microfrontends sem Build Step Complexo

### O que é

Em 2026, Module Federation atingiu a versão 3.0 com o suporte a **Native ESM Federation** — federação de módulos usando Import Maps do browser e Top-Level Await nativamente, sem depender do Webpack runtime. O resultado prático: microfrontends podem ser carregados e atualizados sem um build step de federação complexo.

**Como funciona Native ESM Federation:**

```html
<!-- import map no HTML do shell app -->
<script type="importmap">
{
  "imports": {
    "mfe-checkout": "https://cdn.example.com/checkout/1.4.2/index.js",
    "mfe-catalog": "https://cdn.example.com/catalog/2.1.0/index.js",
    "shared/react": "https://esm.sh/react@19.2.0"
  }
}
</script>
```

```ts
// Shell app — carrega MFE dinamicamente sem Webpack runtime
const { CheckoutModule } = await import('mfe-checkout')
// Top-Level Await: aguarda o MFE estar pronto antes de continuar
```

**IA para geração de configurações de orchestration**: a parte mais complexa de microfrontends — as configurações de container app, shared dependencies, module exposures — pode agora ser gerada por agentes a partir de uma descrição da arquitetura. Claude Code, por exemplo, consegue analisar um repositório multi-package e gerar as configurações de Module Federation corretas para cada app.

**Redução mensurável de build time**: times enterprise reportam redução de build time de "minutos para milissegundos" para o shell app — porque a federação acontece em runtime, não em build time.

### Por que isso importa

Module Federation 2.0 dependia fortemente do Webpack runtime e de configurações complexas de `exposes`, `remotes` e `shared`. A migração para Native ESM elimina essa dependência e torna a federação mais próxima de como o browser já funciona nativamente.

Para times que usam Vite, Rspack ou esbuild, isso é especialmente relevante — eles não precisavam de um adapter Webpack para Module Federation, o que era um ponto de atrito.

### Benefícios práticos

- Build time do shell app: de minutos para milissegundos com Native ESM
- Sem Webpack runtime obrigatório — funciona com Vite, Rspack, esbuild
- Import Maps nativas do browser — padrão web, sem lock-in de bundler
- IA pode gerar orchestration configs a partir de descrições de arquitetura
- Independência de deploy real: cada MFE atualiza sua URL no import map sem rebuild do shell

### Possíveis problemas ou limitações

- Import Maps não são suportados em navegadores muito antigos — verificar baseline do projeto
- Shared dependencies via import map requerem coordenação de versões entre times
- Native ESM Federation ainda é nova — tooling de debugging é menos maduro que Module Federation clássico
- Em ambientes com CSP restrita, importar de CDN externo pode ser bloqueado

### Exemplo prático

```ts
// mfe-checkout — expõe o módulo via ESM puro
// checkout/src/index.ts
export { CheckoutFlow } from './CheckoutFlow'
export { useCart } from './hooks/useCart'

// Vite build gera ESM puro:
// dist/index.js → import-able direto pelo browser

// Shell app — consome via import map
const { CheckoutFlow } = await import('mfe-checkout')

// Com agente gerando a config:
// "Gere a configuração de Module Federation para um shell app que
//  consome mfe-checkout (React 19) e mfe-catalog (Vue 3, isolado)"
// → Claude Code gera import map + lazy loading strategy + error boundaries
```

### Relação com o ecossistema moderno

- **Vite 7**: suporte nativo a ES modules facilita output compatível com ESM Federation
- **Turborepo**: cada package no monorepo pode ser um MFE com build independente
- **React Server Components**: MFEs podem ser RSC — federação de Server Components é o próximo passo
- **Edge Runtime**: MFEs servidos por edge CDNs com import maps atualizando versões sem redeploy
- **Nx**: gerador de módulos federados com suporte a Vite e Rspack

### Vale a pena acompanhar?

**Promissor para empresas.** Para times com múltiplos squads e ciclos de deploy independentes, Native ESM Federation é uma melhoria estrutural real. Para projetos menores, a complexidade adicional não compensa.

---

## 9. Design Tokens como Linguagem Universal para Agentes de IA

### O que é

Em 2026, ficou mais clara uma virada de paradigma: **design systems não são mais construídos para designers, são construídos para IA**. A lógica é direta — agentes de IA têm um problema consistente: geram código visualmente plausível mas que não conhece suas cores, sua escala de espaçamento ou sua tipografia. Design tokens resolvem isso quando fornecidos como contexto.

**Duas ferramentas relevantes nessa semana:**

**UXPin Forge**: o AI assistant da UXPin gera e itera layouts usando componentes codados reais — não apenas mockups visuais. O output é JSX com os componentes do seu DS, constrangido aos tokens definidos. O fluxo é: designer define o layout no UXPin → Forge gera JSX a partir dos componentes React reais mapeados na biblioteca.

**Banani**: plataforma text-to-design-system que gera um sistema completo de design — tokens de cor, tipografia, espaçamento, componentes — a partir de um prompt. Exporta para Figma (como variáveis nativas) e para código (como `tokens.json` ou CSS custom properties). Útil para bootstrapping rápido de novos projetos.

**O padrão emergente — CLAUDE.md com tokens:**

```markdown
# CLAUDE.md

## Design Tokens

Todos os valores visuais vêm de `packages/tokens/tokens.json`.
Nunca hardcode cores, espaçamentos ou tipografia.

### Cores
- `color.brand.primary` → `#0066FF` (uso: CTAs principais)
- `color.neutral.900` → `#111111` (uso: texto principal)

### Espaçamento
- `space.4` → `16px`
- `space.6` → `24px`

### Tipografia
- `type.heading.xl` → `clamp(1.5rem, 3vw, 2.25rem)`

Ao gerar componentes, sempre referencie os tokens por nome semântico.
```

Com esse CLAUDE.md, o agente gera `color: var(--color-brand-primary)` em vez de `color: #0066FF`.

### Por que isso importa

A qualidade do código gerado por agentes melhora diretamente com a qualidade do contexto fornecido. Tokens semânticos bem definidos são a forma mais eficiente de ensinar um agente as regras visuais de um produto — e são reutilizáveis por qualquer ferramenta que leia `CLAUDE.md`, `AGENTS.md` ou um arquivo de tokens.

### Benefícios práticos

- Agentes geram CSS/JSX usando tokens do DS, não valores hardcoded
- Tokens como contrato entre design e código — IA reforça o contrato automaticamente
- `tokens.json` em `CLAUDE.md` funciona como documentação viva de intenção visual
- Banani reduz de semanas para minutos o bootstrapping de um DS novo
- UXPin Forge conecta o loop design ↔ código sem tradução manual

### Possíveis problemas ou limitações

- Tokens precisam ser bem nomeados semanticamente — `color.brand.primary` > `blue-500`
- Agentes ainda podem ignorar tokens em componentes muito específicos sem exemplos concretos no CLAUDE.md
- Banani gera tokens genéricos — custo de refinamento proporcional à especificidade da marca
- UXPin Forge requer que os componentes estejam mapeados na biblioteca — setup inicial proporcional ao tamanho do DS

### Exemplo prático

```json
// packages/tokens/tokens.json
{
  "color": {
    "brand": {
      "primary": { "value": "#0066FF", "type": "color" },
      "secondary": { "value": "#00CC88", "type": "color" }
    },
    "feedback": {
      "error": { "value": "#FF3B30", "type": "color" },
      "warning": { "value": "#FF9500", "type": "color" }
    }
  },
  "space": {
    "1": { "value": "4px", "type": "spacing" },
    "2": { "value": "8px", "type": "spacing" },
    "4": { "value": "16px", "type": "spacing" }
  }
}
```

```css
/* Output esperado de um agente que conhece os tokens */
.button-primary {
  background: var(--color-brand-primary);
  padding: var(--space-2) var(--space-4);
}
/* Em vez de: background: #0066FF; padding: 8px 16px; */
```

### Relação com o ecossistema moderno

- **Style Dictionary**: transforma `tokens.json` em CSS custom properties, JS constants, Tailwind config
- **Figma Variables**: tokens sincronizados com Figma via Figma MCP mantêm DS e código alinhados
- **Tailwind CSS**: tokens podem gerar o `theme` do Tailwind automaticamente
- **Shadcn/ui**: variáveis CSS do Shadcn são tokens — compatíveis com o mesmo padrão
- **Storybook**: tokens documentados no DS guiam tanto a documentação quanto o contexto do agente

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Para qualquer time com DS consolidado, essa é a forma mais eficiente de melhorar a qualidade de código gerado por agentes. O investimento em tokens semânticos tem ROI duplo: melhora consistência humana E qualidade da geração de IA.

---

## 10. Autonoma — Component Contract Testing Gerado por Comportamento Real

### O que é

O Autonoma é uma ferramenta de teste que muda a premissa fundamental de como componentes são testados em 2026. Em vez de escrever stories manualmente (Storybook) ou fixtures estáticas (Vitest), o Autonoma observa o comportamento real dos componentes em uso — em produção ou em sessões de QA — e **gera contratos de componente automaticamente** a partir desse comportamento.

O contrato gerado descreve: quais props o componente recebe em condições reais, quais estados ele assume, quais efeitos produz. Isso se torna a spec de teste.

A diferença prática em relação a Storybook CT:

| | Storybook CT | Autonoma |
|---|---|---|
| Origem dos testes | Stories escritas manualmente | Comportamento real observado |
| Manutenção | Alta — stories desatualizam | Baixa — contrato atualiza com o uso |
| Fidelidade | jsdom ou Playwright | Real-browser via Playwright |
| Quando quebra | Quando a story está errada OU o componente mudou | Quando o componente quebra em condições reais |

**Integração com Portable Stories (2026 standard):**
O Autonoma exporta os contratos gerados como Portable Stories compatíveis com Playwright CT — você ganha fidelidade real de browser sem manter fixtures manualmente.

### Por que isso importa

Times perdem tempo expressivo mantendo stories que ficam desatualizadas conforme os componentes evoluem. O Autonoma resolve o problema na raiz: os contratos derivam do uso real, não de uma especificação estática que alguém escreveu há 6 meses.

### Benefícios práticos

- Zero stories para escrever ou manter — contratos gerados automaticamente
- Real-browser fidelity via Playwright — sem discrepância jsdom vs CI
- Testes baseados em comportamento real — menos falsos positivos
- Contratos versionáveis — diff de contrato mostra o que mudou no comportamento do componente
- Integração com CI/CD — contratos rodam automaticamente em cada PR

### Possíveis problemas ou limitações

- Componentes novos não têm comportamento observado — bootstrap inicial requer sessão de uso
- Dados sensíveis de usuários em produção precisam ser anonimizados antes de gerar contratos
- Contratos podem incluir edge cases raros de produção que são difíceis de reproduzir em CI
- Ainda é uma abordagem nova — comunidade e documentação menores que Storybook

### Exemplo prático

```ts
// autonoma.config.ts
export default {
  observe: {
    source: 'production',
    sampleRate: 0.1, // 10% das sessões de usuário
    anonymize: ['email', 'name', 'cpf'],
  },
  output: {
    format: 'portable-stories',
    dir: './src/contracts',
  },
}

// Contrato gerado automaticamente:
// src/contracts/ProductCard.contract.ts
export const ProductCardContracts = [
  {
    name: 'com badge de desconto',
    props: {
      title: 'Tênis Running Pro',
      price: 299.9,
      originalPrice: 399.9,
      imageUrl: '...',
    },
    expects: {
      renders: true,
      hasBadge: true,
      badgeText: '25% OFF',
    },
  },
  // ... gerado a partir de uso real
]

// Roda via Playwright CT — real browser
```

### Relação com o ecossistema moderno

- **Storybook + Playwright**: contratos gerados pelo Autonoma são compatíveis com o "2026 standard" de portable stories em Playwright CT
- **Vitest**: contratos exportados como test specs rodam no Vitest para feedback mais rápido
- **Next.js/React**: agnóstico de framework — observa qualquer componente React
- **CI/CD (GitHub Actions, GitLab CI)**: contratos rodam em PR via Playwright
- **Sentry/DataDog**: integração para capturar comportamento de componentes a partir de sessões reais instrumentadas

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Para times que sofrem com a manutenção de stories desatualizadas, o Autonoma endereça a causa raiz. A abordagem de "contrato a partir do comportamento real" é mais robusta do que especificações estáticas — é o mesmo insight que levou a contract testing em microsserviços.
