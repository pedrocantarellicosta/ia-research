# 🖥️ Frontend AI Weekly Research — 01/07/2026

> 20 novidades de IA aplicadas ao ecossistema front-end moderno

---

## 1. Nuxt UI — Componentes Nativos para Interfaces de Chat com IA

### O que é

O Nuxt UI (atualmente na versão v3+) lançou uma suite completa de componentes voltados para interfaces de chat com IA. Os novos componentes incluem `ChatMessage`, `ChatMessages`, `ChatPrompt`, `ChatPalette`, `ChatReasoning`, `ChatTool` e `ChatShimmer`. Eles foram desenhados para integração nativa com a Vercel AI SDK, suportando streaming de respostas, raciocínio visível (thinking blocks), tool calling e multi-model via Vercel AI Gateway.

### Por que isso importa

Times que trabalham com Vue/Nuxt agora têm um design system pronto e opinado para construir produtos de IA — sem precisar montar pipes de streaming manualmente ou criar componentes do zero. A integração é first-class: os componentes entendem os estados de streaming, loading e erro intrinsecamente.

### Benefícios práticos

- `ChatReasoning`: bloco colapsável que exibe o processo de raciocínio do modelo, com timer de streaming automático.
- `ChatTool`: mostra invocações de ferramentas com estados de loading e streaming, ideal para agentic UIs.
- `ChatShimmer`: animação de texto durante geração — elimina a necessidade de customização de skeleton.
- Templates prontos com autenticação, histórico de chat, sidebar, atalhos de teclado, light/dark mode.

### Possíveis problemas ou limitações

- Lock-in com Vercel AI SDK e Nuxt UI. Times com design system proprietário precisarão adaptar estilos significativamente.
- O `ChatTool` ainda não suporta estados de erro granular por tool — falhas são tratadas no nível do fluxo.
- Ainda não há suporte a Web Components nativos — os componentes são Vue-only.

### Exemplo prático

```vue
<template>
  <UChatMessages :messages="messages">
    <template #message="{ message }">
      <UChatMessage :message="message" />
      <UChatReasoning v-if="message.reasoning" :reasoning="message.reasoning" />
    </template>
  </UChatMessages>
  <UChatPrompt @submit="handleSubmit" :loading="isLoading" />
</template>

<script setup lang="ts">
import { useChat } from '@ai-sdk/vue'
const { messages, handleSubmit, isLoading } = useChat({ api: '/api/chat' })
</script>
```

### Relação com o ecossistema moderno

- **Nuxt/Vue**: componentes nativos sem adaptação.
- **Vercel AI SDK**: integração direta com `useChat` e `streamText`.
- **Design Systems**: extende o Nuxt UI, que usa Tailwind v4 e Reka UI internamente.
- **SSR/Server Components**: o `/api/chat` roda em Nitro (edge-ready).

### Vale a pena acompanhar?

**Sim, vale acompanhar** — especialmente para times Vue/Nuxt que querem construir produtos de IA. É a solução mais coesa do ecossistema Vue hoje.

---

## 2. Figma for Agents — Design System + MCP + Code Connect

### O que é

A Figma lançou o **Figma for Agents**, uma extensão do MCP (Model Context Protocol) que permite agentes de IA lerem diretamente o design system via servidor MCP da Figma. A chave é o **Code Connect** — um mapeamento explícito entre componentes Figma e seus equivalentes no código. Sem Code Connect, o agente não sabe qual componente Figma corresponde ao `<Button variant="primary">` do seu codebase.

### Por que isso importa

O fluxo Figma → IA → código finalmente tem uma ponte confiável. Antes, agentes geravam componentes do zero sem respeitar o design system existente. Com Code Connect + Figma MCP, o agente encontra os tokens corretos, as props certas e compõe componentes existentes ao invés de criar variantes arbitrárias.

### Benefícios práticos

- Agentes como Claude Code e Cursor conseguem consultar componentes, tokens e variantes diretamente do Figma.
- Reduz drasticamente a deriva entre design e código.
- Brad Frost (autor de Atomic Design) demonstrou: o agente leu `Star`, `Typography` e `Avatar`, entendeu props e estados, e compôs um `CustomerReview` component com testes.
- Push de variantes Figma direto para Storybook via pipeline automatizado.

### Possíveis problemas ou limitações

- Code Connect requer manutenção manual: cada componente precisa de mapeamento explícito. Em projetos grandes, isso vira trabalho de curadoria contínua.
- O MCP da Figma ainda não expõe tokens de animação e shadows com fidelidade total.
- A qualidade da saída depende 100% da qualidade do Code Connect — garbage in, garbage out.

### Exemplo prático

```ts
// code-connect/Button.figma.ts
import figma from '@figma/code-connect'
import { Button } from '../src/components/Button'

figma.connect(Button, 'https://figma.com/file/...', {
  props: {
    variant: figma.enum('Variant', {
      Primary: 'primary',
      Secondary: 'secondary',
    }),
    disabled: figma.boolean('Disabled'),
    label: figma.string('Label'),
  },
  example: ({ variant, disabled, label }) => (
    <Button variant={variant} disabled={disabled}>{label}</Button>
  ),
})
```

### Relação com o ecossistema moderno

- **Design Systems**: é a ponte entre Figma e o código — funciona com qualquer framework.
- **Storybook 9**: pipeline Figma → Storybook via Code Connect está documentado oficialmente.
- **Monorepos**: o Code Connect vive no repo junto com os componentes.
- **CI/CD**: pode ser validado em PR via Figma CLI.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — é infraestrutura estratégica para design systems maduros.

---

## 3. Storybook 9 — Suite de Testes Unificada com IA

### O que é

O Storybook 9 reformulou completamente sua abordagem de testes: cada story agora é automaticamente testável em três dimensões — **Interaction** (Vitest), **Accessibility** (axe-core, captura até 57% dos problemas WCAG) e **Visual** (Chromatic). Um novo widget no painel lateral exibe o status consolidado de todos os testes, e um botão executa a suite completa com um clique.

### Por que isso importa

O Storybook deixou de ser apenas uma ferramenta de documentação e se tornou a plataforma de testes de componentes. A integração com IA permite geração de stories para componentes sem cobertura, remediação automática de issues de acessibilidade e visual regression assistida.

### Benefícios práticos

- Stories como fonte de verdade para testes: nada de duplicar setup em arquivos de spec separados.
- IA gera stories iniciais para componentes sem cobertura — base para refinamento humano.
- AI-powered accessibility remediation: sugere correções para violações WCAG diretamente no contexto do componente.
- Integração nativa com Vitest e Chromatic sem configuração extra.

### Possíveis problemas ou limitações

- A remediação de a11y sugerida por IA ainda produz falsos positivos em componentes com interatividade complexa.
- Visual testing via Chromatic tem custo em planos pagos — não é completamente gratuito.
- A cobertura de 57% do axe-core ainda deixa 43% de problemas WCAG para revisão manual.

### Exemplo prático

```ts
// Button.stories.ts
import type { Meta, StoryObj } from '@storybook/react'
import { Button } from './Button'

const meta: Meta<typeof Button> = {
  component: Button,
  tags: ['autodocs'],
}
export default meta

export const Primary: StoryObj<typeof Button> = {
  args: { variant: 'primary', children: 'Click me' },
  // Accessibility test roda automaticamente — sem configuração adicional
}
```

### Relação com o ecossistema moderno

- **React/Vue/Svelte**: suporta todos os frameworks populares.
- **Design Systems**: é a plataforma de referência para componentes documentados e testados.
- **CI/CD**: Storybook Test pode rodar em pipeline via `storybook test --ci`.
- **Figma**: Code Connect conecta stories com componentes Figma.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — especialmente se o time já usa Storybook. A migração para v9 traz valor imediato.

---

## 4. Core Web Vitals 2026 — INP substitui FID e IA entra nos dois lados

### O que é

Em 2026, o INP (Interaction to Next Paint) é a métrica oficial de responsividade, substituindo completamente o FID. Os novos thresholds "Good" são: LCP < 2.0s, INP < 200ms, CLS < 0.1. O que mudou estruturalmente é que o Google agora usa IA (via Chrome) para medir a experiência do usuário, enquanto plataformas como Vercel e Cloudflare usam IA para otimizar — criando um battlefield de "AI vs AI".

### Por que isso importa

Sites com boas métricas de Core Web Vitals aparecem em AI Overviews do Google. Sites lentos raramente aparecem em respostas geradas por IA, segundo análise de dezembro de 2025. Isso conecta performance técnica diretamente com visibilidade em buscas baseadas em IA — um novo vetor de SEO.

### Benefícios práticos

- INP mede o pior percentil de interações — foca em casos extremos, não na mediana.
- Ferramentas como Vercel Speed Insights usam IA para identificar padrões de degradação de INP antes que afetem usuários reais.
- Cloudflare Zaraz e similares usam AI para decidir quais scripts de terceiros carregar e quando.
- Sinais de acessibilidade (consistência de ARIA, responsividade com leitores de tela) começam a influenciar rankings de qualidade.

### Possíveis problemas ou limitações

- O INP em aplicações React com muitos `useState` aninhados pode ser difícil de otimizar sem refatoração arquitetural.
- Ferramentas de IA para otimização automática (ex: Vercel) têm acesso limitado a código proprietário — a otimização real ainda é manual.
- AI Overviews ainda não têm uma correlação 1:1 comprovada com Core Web Vitals — é uma tendência observada, não uma regra documentada.

### Exemplo prático

```ts
// Medindo INP manualmente com web-vitals
import { onINP } from 'web-vitals'

onINP(({ value, rating, entries }) => {
  // Enviar para observabilidade
  analytics.track('inp', {
    value,
    rating, // 'good' | 'needs-improvement' | 'poor'
    longestEntry: entries[entries.length - 1],
  })
})
```

```ts
// Deferring non-critical interactions para melhorar INP
import { startTransition } from 'react'

function handleFilterChange(value: string) {
  startTransition(() => {
    setFilter(value) // React marca como não-urgente
  })
}
```

### Relação com o ecossistema moderno

- **Next.js/Nuxt**: SSR + Streaming melhora LCP e INP nativamente.
- **React**: `startTransition` e `useDeferredValue` são as APIs corretas para INP.
- **Edge Runtime**: edge functions reduzem TTFB, impactando diretamente LCP.
- **Vite**: o code splitting automático do Vite reduz JS blocking, melhorando INP.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — INP é critério de rankeamento e impacta visibilidade em AI Overviews.

---

## 5. Vercel AI SDK v4+ — streamUI, Server Actions e useObject

### O que é

O Vercel AI SDK na versão 4+ consolida streaming, Server Actions e geração de UI estruturada em uma API coesa. As principais adições são: `streamUI` (para streaming de componentes React via RSC), `useObject` (para streaming de objetos JSON tipados com Zod), e `generateObject` no servidor. A biblioteca é edge-runtime-first com apenas 67.5 kB gzipped.

### Por que isso importa

O SDK abstrai a complexidade de streaming de LLMs e oferece uma API React-nativa. Times não precisam mais lidar com SSE, parsing de deltas ou gerenciamento de estado de streaming manualmente. A tipagem end-to-end via Zod elimina uma classe inteira de bugs de runtime.

### Benefícios práticos

- `useChat` e `useCompletion`: hooks para interfaces de chat com estado gerenciado automaticamente.
- `useObject`: stream de objetos tipados — ideal para geração progressiva de dados estruturados.
- Provider agnostic: OpenAI, Anthropic Claude, Google Gemini, Mistral em uma linha de mudança.
- `ai-sdk-helpers`: adiciona rate limiting, logging estruturado e cache de resposta para produção.

### Possíveis problemas ou limitações

- `streamUI` requer React Server Components — só funciona em Next.js App Router (ou frameworks com RSC).
- O modelo mental de streaming de objetos com `useObject` pode ser confuso para times sem experiência com streaming.
- Provider agnosticism tem limites: features como Extended Thinking do Claude não têm equivalente em outros providers.

### Exemplo prático

```ts
// app/api/generate-report/route.ts
import { streamObject } from 'ai'
import { anthropic } from '@ai-sdk/anthropic'
import { z } from 'zod'

const reportSchema = z.object({
  title: z.string(),
  summary: z.string(),
  sections: z.array(z.object({
    heading: z.string(),
    content: z.string(),
  })),
})

export async function POST(req: Request) {
  const { topic } = await req.json()
  const result = streamObject({
    model: anthropic('claude-sonnet-4-6'),
    schema: reportSchema,
    prompt: `Generate a technical report about: ${topic}`,
  })
  return result.toTextStreamResponse()
}
```

```tsx
// app/report/page.tsx
'use client'
import { experimental_useObject as useObject } from 'ai/react'

export default function ReportPage() {
  const { object, submit, isLoading } = useObject({
    api: '/api/generate-report',
    schema: reportSchema,
  })

  return (
    <div>
      <button onClick={() => submit({ topic: 'React Server Components' })}>
        Gerar relatório
      </button>
      {object?.sections?.map((section, i) => (
        <section key={i}>
          <h2>{section?.heading}</h2>
          <p>{section?.content}</p>
        </section>
      ))}
    </div>
  )
}
```

### Relação com o ecossistema moderno

- **Next.js**: integração first-class com App Router e Server Actions.
- **Nuxt**: suporte via `@ai-sdk/vue` com composables nativos.
- **SvelteKit**: suporte oficial via `@ai-sdk/svelte`.
- **Edge Runtime**: construído para funcionar em Vercel Edge, Cloudflare Workers e Deno Deploy.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — é o SDK de referência para AI no ecossistema JavaScript/TypeScript.

---

## 6. Vue vs React em 2026 — A escolha de times AI-first

### O que é

Em 2026, a escolha entre Vue e React em times AI-first mudou de critério. Não é mais sobre DX pura ou tamanho de ecossistema: é sobre qual framework é melhor "compreendido" pelos LLMs no contexto de geração de código. Análises mostram que React/Next.js domina os datasets de treinamento, mas Vue 3 com Composition API tem menor taxa de alucinação em componentes gerados.

### Por que isso importa

Times que usam IA extensivamente no desenvolvimento precisam avaliar qual framework produz código correto mais consistentemente. Vue 3 com `<script setup>` tem sintaxe mais compacta e determinística, o que reduz ambiguidade na geração. React tem mais exemplos nos modelos, mas também mais padrões conflitantes (Class Components, HOCs, hooks patterns antigos).

### Benefícios práticos

- Vue 3 `<script setup>` + TypeScript: sintaxe concisa que LLMs reproduzem com menos variância.
- `@nuxtjs/mcp-toolkit`: expõe a documentação do Nuxt diretamente para agentes de IA via MCP — Claude Code e Cursor acessam exemplos atualizados em tempo real.
- React Server Components + TypeScript: melhor cobertura em modelos modernos, especialmente Claude Opus 4.8 e Fable 5.

### Possíveis problemas ou limitações

- A vantagem de "menos alucinação" em Vue é observacional, não benchmarkada formalmente.
- Times pequenos com um único desenvolvedor Vue podem ter dificuldade com o menor ecossistema de bibliotecas AI-specific para Vue.
- O `@nuxtjs/mcp-toolkit` requer configuração manual no agente — não é plug-and-play em todos os editores.

### Exemplo prático

```bash
# Instalar o MCP toolkit no projeto Nuxt
npx nuxi module add @nuxtjs/mcp-toolkit
```

```ts
// nuxt.config.ts — expõe docs para agentes IA
export default defineNuxtConfig({
  modules: ['@nuxtjs/mcp-toolkit'],
  mcp: {
    exposeComponents: true,
    exposeComposables: true,
    exposeRoutes: true,
  }
})
```

### Relação com o ecossistema moderno

- **Vue/Nuxt**: o `mcp-toolkit` conecta o projeto diretamente com Claude Code, Cursor e qualquer cliente MCP.
- **Design Systems**: componentes documentados via MCP ficam disponíveis para composição por agentes.
- **Monorepos**: o toolkit funciona por package — cada app do monorepo pode expor seu contexto.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — o `@nuxtjs/mcp-toolkit` é uma mudança de paradigma para times Vue.

---

## 7. Tailwind v4 + Design Tokens + IA — O novo padrão de consistência

### O que é

O Tailwind v4 adotou uma abordagem CSS-first onde todos os design tokens são expostos como variáveis CSS nativas via diretiva `@theme`. Isso resolve um problema crítico com geração de código por IA: LLMs que geram Tailwind sem contexto do design system criam código com cores e espaçamentos arbitrários. Com `@theme` + CSS custom properties, os tokens ficam legíveis para humanos e IA.

### Por que isso importa

51% dos devs CSS usam Tailwind em 2026. O problema é que IA gera Tailwind inconsistente quando não tem os tokens do projeto. A solução emergente é: antes de pedir a geração de componentes, injetar o `@theme` do projeto no contexto do modelo. Claude Sonnet 4.6 gera Tailwind mais semântico e consistente que GPT-4o nesse cenário.

### Benefícios práticos

- Tokens como variáveis CSS: `--color-brand-500`, `--spacing-layout`, etc. — legíveis por IA e por humanos.
- Geração automática de tokens a partir de paleta de cores via ferramentas como `style-dictionary` com plugin Tailwind v4.
- AI tools (Cursor, Claude Code) conseguem ler o `tailwind.config.ts` ou o CSS de tema e usar tokens corretos.
- Sincronização Figma → Design Tokens → Tailwind v4 via pipelines automatizados.

### Possíveis problemas ou limitações

- A migração de Tailwind v3 para v4 tem breaking changes na configuração — não é trivial em projetos grandes.
- Injetar o `@theme` inteiro no contexto do LLM consome tokens (literalmente e monetariamente).
- Ferramentas como `tailwindgenai.com` geram a partir do zero, ignorando o design system — criar guardrails é responsabilidade do time.

### Exemplo prático

```css
/* design-tokens.css */
@theme {
  --color-brand-50: oklch(97% 0.01 250);
  --color-brand-500: oklch(55% 0.18 250);
  --color-brand-900: oklch(25% 0.12 250);
  
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-layout: 1.5rem;
  
  --font-display: 'Inter', sans-serif;
  --radius-card: 0.75rem;
}
```

```
// Prompt para Claude Code com contexto de tokens:
"Crie um componente Card usando APENAS os tokens do @theme abaixo.
Não use valores arbitrários como text-blue-500 ou p-4.
[colar conteúdo do design-tokens.css]"
```

### Relação com o ecossistema moderno

- **Design Systems**: tokens como source of truth compartilhada entre Figma, Tailwind e componentes.
- **Monorepos**: `packages/design-tokens` publica tokens para todos os apps.
- **Vite**: Tailwind v4 usa um plugin Vite nativo, sem PostCSS obrigatório.
- **SSR**: CSS variables funcionam em SSR sem hydration issues.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — a migração para v4 + tokens é o caminho certo para design systems escaláveis.

---

## 8. AI e Refatoração Arquitetural de Componentes React/Vue

### O que é

Em 2026, agentes de IA como Claude Code e Cursor estão sendo usados para refatorações arquiteturais de escala média: reestruturação de árvores de componentes, identificação de gargalos de rendering, conversão de Class Components para hooks, extração de lógica para composables (Vue) ou custom hooks (React). Claude Opus 4.8 com janela de contexto de 1M tokens lidera em refatorações que envolvem múltiplos arquivos.

### Por que isso importa

Refatoração é a tarefa front-end onde IA mais agrega valor além de geração de código novo. O modelo consegue analisar padrões de re-render, prop drilling excessivo, e sugerir estratégias de composição sem introduzir regressões — especialmente quando o codebase tem testes.

### Benefícios práticos

- Detecção de prop drilling profundo e sugestão de Context API ou Pinia/Zustand como alternativas.
- Identificação de componentes com responsabilidades múltiplas para split horizontal.
- Conversão automática de `Options API` para `Composition API` com `<script setup>` em Vue.
- Claude Code com Dynamic Workflows divide jobs grandes de refatoração em subagentes paralelos.

### Possíveis problemas ou limitações

- IA pode sugerir abstrações prematuras — especialmente Context em componentes que não precisam de estado global.
- Refatorações que envolvem estado compartilhado entre microfrontends requerem contexto que o modelo não tem por padrão.
- A janela de contexto de 1M tokens ainda não é suficiente para codebases muito grandes — é necessário estratégia de chunking.

### Exemplo prático

```
// Prompt para Claude Code:
"Analise o componente ProductList.vue (e seus filhos até 3 níveis de profundidade).
Identifique:
1. Props sendo passadas mais de 2 níveis sem transformação
2. Lógica de negócio misturada com lógica de apresentação
3. Composables que poderiam ser extraídos

Não altere código ainda — apenas liste os problemas encontrados com linha e arquivo."
```

### Relação com o ecossistema moderno

- **Vue/Nuxt**: `<script setup>` + composables é o destino da maioria das refatorações Vue.
- **React/Next**: custom hooks + Server Components é o destino React.
- **Microfrontends**: a refatoração frequentemente revela limites de bounded context incorretos.
- **Turborepo**: facilita extrair lógica para packages compartilhados durante refatoração.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — refatoração assistida por IA é onde o ROI é mais imediato.

---

## 9. Mastra — Framework TypeScript para Agentes de IA em Apps Front-end

### O que é

O Mastra é um framework TypeScript moderno para construir aplicações e agentes de IA. Ele oferece primitivas de alto nível para agentes, workflows, memória, workspaces e observabilidade — tudo com tipos TypeScript first-class. É diferente do LangChain.js por ser mais opinionado e focado em DX para times de produto, não de pesquisa.

### Por que isso importa

Times front-end que querem adicionar capacidades agentísticas em suas apps Next.js ou Nuxt não precisam mais de um backend Python separado. O Mastra roda em Node.js/Edge e pode ser importado diretamente em Server Actions ou API Routes.

### Benefícios práticos

- Agents com tool calling, memória de longo prazo e recuperação de contexto out-of-the-box.
- Workflows visuais com grafo de passos — fácil de debugar e monitorar.
- Integração com qualquer modelo via adaptadores (OpenAI, Anthropic, Google).
- Observabilidade built-in: cada execução é traceable.

### Possíveis problemas ou limitações

- Framework relativamente novo — ecosystem ainda menor que LangChain.js.
- Workflows complexos com estado persistente requerem infraestrutura de banco de dados.
- A abstração de memória do Mastra tem custo de tokens em contexto — não é gratuita.

### Exemplo prático

```ts
// lib/agents/research-agent.ts
import { createAgent } from '@mastra/core'
import { anthropic } from '@mastra/anthropic'

export const researchAgent = createAgent({
  name: 'Research Agent',
  model: anthropic('claude-sonnet-4-6'),
  tools: [webSearchTool, scrapePageTool],
  memory: { type: 'postgres', connectionString: process.env.DATABASE_URL },
  systemPrompt: `Você é um pesquisador de front-end.
  Encontre informações técnicas relevantes e retorne em formato estruturado.`,
})

// app/api/research/route.ts
export async function POST(req: Request) {
  const { query } = await req.json()
  const result = await researchAgent.run(query)
  return Response.json(result)
}
```

### Relação com o ecossistema moderno

- **Next.js/Nuxt**: roda em API Routes e Server Actions.
- **Edge Runtime**: suporte parcial — workflows com banco de dados requerem Node.js runtime.
- **Observabilidade**: integra com Langfuse e Braintrust para tracing.

### Vale a pena acompanhar?

**Promissor para empresas** — mais maduro que construir do zero, menos complexo que LangChain.

---

## 10. AI-Powered SEO Técnico — AEO e GEO além do Core Web Vitals

### O que é

Em 2026, SEO técnico evoluiu para três dimensões: SEO tradicional, AEO (Answer Engine Optimization) para featured snippets e AI answers, e GEO (Generative Engine Optimization) para aparecer em respostas de ChatGPT, Gemini AI Overviews e Perplexity. Ferramentas de IA agora analisam se o conteúdo HTML está estruturado para ser "compreendido" por LLMs, não apenas por crawlers tradicionais.

### Por que isso importa

Sites com boa estrutura semântica (schema.org, ARIA, heading hierarchy) têm maior probabilidade de serem citados em AI Overviews. Isso significa que decisões de markup front-end — que componente usar, qual nível de heading, quais atributos ARIA — agora têm impacto direto em tráfego orgânico via IA.

### Benefícios práticos

- Ferramentas de IA analisam o DOM e sugerem melhorias de schema.org automaticamente.
- Geração automática de `application/ld+json` para componentes Next.js/Nuxt via LLMs.
- Análise de hierarquia de headings e semântica HTML com feedback instantâneo.
- Monitoramento de aparições em AI Overviews via ferramentas como Semrush AI Toolkit.

### Possíveis problemas ou limitações

- GEO ainda não tem métricas confiáveis e auditáveis — é difícil provar ROI.
- Schema.org gerado por IA pode ter erros de tipo que invalidam o rich snippet.
- Otimização para AI Overviews pode entrar em conflito com CTR direto — se o Google responde a pergunta, o usuário não clica.

### Exemplo prático

```tsx
// app/product/[slug]/page.tsx
import { generateProductSchema } from '@/lib/schema'

export default async function ProductPage({ params }) {
  const product = await getProduct(params.slug)
  
  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{
          __html: JSON.stringify(generateProductSchema(product)),
        }}
      />
      <main>
        <h1>{product.name}</h1>
        {/* ... */}
      </main>
    </>
  )
}
```

```ts
// Prompt para gerar schema com Claude:
"Dado este componente React de produto, gere um schema.org Product válido
em JSON-LD. Inclua: name, description, price, availability, brand, image.
Use os dados reais dos props. Não use valores placeholder."
```

### Relação com o ecossistema moderno

- **Next.js/Nuxt**: Metadata API nativa para Open Graph e structured data.
- **SSR**: schema.org precisa estar no HTML inicial — SSR é obrigatório.
- **Design Systems**: componentes semânticos corretos (usar `<article>` vs `<div>`) impactam GEO.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — GEO/AEO é o novo SEO técnico e ignora-lo é risco estratégico.

---

## 11. Micro-frontends + IA — Limites de Contexto e Automação de Contratos

### O que é

A maior dor de times que adotam microfrontends é a manutenção de contratos entre apps (tipos compartilhados, eventos de comunicação, APIs de composição). Em 2026, agentes de IA estão sendo usados para: detectar quebras de contrato entre MFEs, gerar documentação de interfaces automaticamente, e propor estratégias de bounded context baseadas em análise de dependências.

### Por que isso importa

Microfrontends introduzem complexidade de governança que cresce quadraticamente com o número de apps. IA consegue analisar o grafo de dependências e identificar acoplamentos ocultos que deveriam ser contratos explícitos — antes que virem bugs em produção.

### Benefícios práticos

- Análise automática de `import` entre pacotes para detectar violações de bounded context.
- Geração de tipos TypeScript para contratos de comunicação entre MFEs (custom events, props de Web Components).
- Detecção de duplicação de componentes entre apps que deveriam ser extraídos para um design system shared.
- Documentação automática de interfaces públicas de cada MFE.

### Possíveis problemas ou limitações

- Agentes de IA não conseguem executar código para testar contratos dinamicamente — a análise é estática.
- A qualidade da detecção de bounded context depende de convenções de nomenclatura consistentes no código.
- Module Federation com IA ainda requer configuração humana dos shared modules — automação parcial.

### Exemplo prático

```
// Prompt para Claude Code no contexto de monorepo:
"Analise as dependências entre apps/checkout e apps/catalog.
Liste todos os imports diretos entre eles (que não passam por packages/shared).
Para cada import, avalie se deveria ser: (a) movido para packages/shared,
(b) comunicado via custom event, ou (c) aceito como acoplamento legítimo.
Baseie-se no princípio de que MFEs não devem importar código uns dos outros diretamente."
```

### Relação com o ecossistema moderno

- **Turborepo**: o grafo de dependências do Turborepo é o mapa que o agente analisa.
- **Module Federation**: o agente pode sugerir o `shared` config correto.
- **Web Components**: interfaces de Web Components são contratos naturais para MFEs.
- **Monorepos**: o contexto de todos os packages precisa estar disponível para o agente.

### Vale a pena acompanhar?

**Promissor para empresas** — o valor real aparece em projetos com 5+ microfrontends.

---

## 12. React Server Components + IA — Padrões de Composição em 2026

### O que é

Em 2026, os React Server Components (RSC) estão estabilizados no Next.js App Router e os padrões de composição entre Server e Client Components estão documentados e sendo gerados corretamente por LLMs modernos. A principal evolução é a integração de streaming de dados de LLMs diretamente em RSC via `streamUI` do Vercel AI SDK.

### Por que isso importa

RSC permite que componentes que consomem APIs de IA sejam renderizados no servidor, sem custo de JavaScript no cliente. Isso é especialmente relevante para features como "AI Summary" ou "Smart Recommendations" — o modelo roda server-side e entrega HTML puro para o cliente.

### Benefícios práticos

- Zero JavaScript no cliente para componentes que apenas consomem respostas de IA.
- `streamUI`: entrega componentes React progressivamente conforme o modelo gera tokens.
- Server Actions com AI: forms que chamam LLMs sem exposição de API key no cliente.
- Streaming + Suspense: UI parcial aparece imediatamente enquanto o modelo processa.

### Possíveis problemas ou limitações

- `streamUI` ainda é experimental — não usar em produção sem testes extensivos.
- RSC + streaming cria complexidade de debug — ferramentas de DevTools ainda estão evoluindo.
- O modelo mental de "o que roda onde" em RSC ainda confunde devs vindos de CSR.

### Exemplo prático

```tsx
// app/product/[id]/ai-summary/page.tsx (Server Component)
import { streamUI } from 'ai/rsc'
import { anthropic } from '@ai-sdk/anthropic'

export default async function AISummary({ params }) {
  const product = await getProduct(params.id)
  
  const { value: summaryUI } = await streamUI({
    model: anthropic('claude-sonnet-4-6'),
    prompt: `Summarize this product for a customer: ${JSON.stringify(product)}`,
    text: ({ content }) => <p className="text-gray-600">{content}</p>,
  })
  
  return (
    <section aria-label="AI-generated summary">
      <Suspense fallback={<SummarySkeleton />}>
        {summaryUI}
      </Suspense>
    </section>
  )
}
```

### Relação com o ecossistema moderno

- **Next.js**: único framework com suporte completo a RSC + streaming em produção.
- **Vite**: RSC está em desenvolvimento no Vite — não disponível ainda fora de Next.js.
- **Edge Runtime**: RSC roda no Edge da Vercel com latência otimizada.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — RSC + AI é o padrão emergente para features de IA em Next.js.

---

## 13. AI-Driven Accessibility — Do Axe-core para Remediação Automática

### O que é

A acessibilidade web em 2026 evoluiu de auditoria passiva (axe-core rodando e listando erros) para um ciclo de remediação assistida por IA. Ferramentas como o assistente AI do Storybook 9, o AccessibilityAI da Deque e plugins de IDE conseguem não apenas detectar violações WCAG, mas sugerir e aplicar correções com contexto do componente.

### Por que isso importa

Times front-end gastam tempo significativo em correções de acessibilidade que são mecânicas e repetitivas (falta de `aria-label`, `alt` text ausente, contraste insuficiente). IA consegue resolver a maioria dessas automaticamente, liberando o time para problemas de acessibilidade cognitiva e de fluxo — que requerem julgamento humano.

### Benefícios práticos

- Geração automática de `alt` text descritivo para imagens via LLMs com visão.
- Correção automática de estrutura de headings, roles ARIA e landmarks.
- Sugestão de tokens de cor com contraste WCAG AA/AAA via ferramentas como Polychrome.
- Testes de a11y em Storybook 9 com sugestão de fix inline na violação.

### Possíveis problemas ou limitações

- `alt` text gerado por IA pode ser descritivo mas não contextual — "imagem de homem sorrindo" vs "CEO João Silva na conferência de 2025".
- Correções automáticas de ARIA podem introduzir semântica incorreta em componentes complexos.
- A acessibilidade cognitiva (clareza de linguagem, fluxo de navegação) não pode ser automatizada.

### Exemplo prático

```tsx
// Antes (violação de acessibilidade)
<img src="/hero.jpg" />
<button onClick={handleClose}>X</button>

// Depois (com remediação por IA aplicada)
<img src="/hero.jpg" alt="Equipe de desenvolvimento em sessão de pair programming" />
<button 
  onClick={handleClose} 
  aria-label="Fechar modal"
  type="button"
>
  <XIcon aria-hidden="true" />
</button>
```

### Relação com o ecossistema moderno

- **Design Systems**: componentes acessíveis como primitivos — evita corrigir nos consumers.
- **Storybook 9**: testes de a11y por story com remediação sugerida.
- **CI/CD**: `axe-core` + AI remediation pode rodar como quality gate no PR.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — automatizar a11y mecânica libera tempo para o que realmente importa.

---

## 14. Google Genkit 1.0 — Framework AI Multilingue com TypeScript e Go

### O que é

O Google Genkit atingiu a versão 1.0 estável com suporte a TypeScript, Python, Go e Dart. Para times front-end, a relevância está na versão TypeScript: é um framework opinionado para construir fluxos de IA (flows) com avaliação built-in, traces automáticos e uma Web UI visual para debug de agentes.

### Por que isso importa

Diferente do Vercel AI SDK (focado em UI/UX de chat e streaming), o Genkit foca em fluxos complexos de processamento: RAG pipelines, evaluations, multi-step workflows. Para times que precisam de mais do que chat — como geração de conteúdo, classificação, extração de dados — o Genkit oferece estrutura arquitetural.

### Benefícios práticos

- Web UI de debug: visualiza o trace de cada flow com inputs, outputs e latência por step.
- Evaluations built-in: testa se o flow produz outputs corretos sem ter que montar framework de teste do zero.
- Firebase integration: deploy de flows como Firebase Functions com um comando.
- Suporte a multiple providers em um flow: pode chamar Gemini para um step e Claude para outro.

### Possíveis problemas ou limitações

- A Web UI de debug só funciona em desenvolvimento — não há equivalente para produção sem exportar para uma plataforma de observabilidade.
- A abstração de "flows" pode ser overkill para casos de uso simples de chat.
- A integração com Next.js requer configuração adicional — não é tão smooth quanto o Vercel AI SDK.

### Exemplo prático

```ts
import { genkit } from 'genkit'
import { googleAI } from '@genkit-ai/googleai'
import { z } from 'zod'

const ai = genkit({ plugins: [googleAI()] })

export const extractMetadataFlow = ai.defineFlow(
  {
    name: 'extractMetadata',
    inputSchema: z.object({ url: z.string(), html: z.string() }),
    outputSchema: z.object({
      title: z.string(),
      description: z.string(),
      keywords: z.array(z.string()),
    }),
  },
  async ({ url, html }) => {
    const result = await ai.generate({
      model: 'googleai/gemini-2.0-flash',
      prompt: `Extract SEO metadata from this page: ${html.slice(0, 5000)}`,
      output: { schema: MetadataSchema },
    })
    return result.output
  }
)
```

### Relação com o ecossistema moderno

- **Next.js**: flows podem ser chamados em Server Actions.
- **Firebase**: deploy nativo como Cloud Functions.
- **Observabilidade**: exporta para OpenTelemetry — compatível com Langfuse e Jaeger.

### Vale a pena acompanhar?

**Promissor para empresas** — especialmente times que já usam Google Cloud.

---

## 15. Meticulous AI — Testes Frontend Sem Escrever Testes

### O que é

O Meticulous.ai é uma plataforma que gera e mantém testes de regressão front-end automaticamente, sem que o desenvolvedor escreva uma única linha de código de teste. O sistema grava sessões de usuário em staging/produção e usa IA para identificar mudanças visuais e comportamentais em PRs. Os testes se auto-reparam quando a UI muda intencionalmente.

### Por que isso importa

A maior barreira para coverage de testes front-end é o custo de manutenção — testes quebram a cada mudança de UI e consomem tempo do desenvolvedor para corrigir. O Meticulous resolve isso eliminando a autoria e a manutenção manuais.

### Benefícios práticos

- Zero código de teste para escrever: gravação passiva de sessões gera os casos.
- Self-healing: quando um componente muda intencionalmente, os testes se atualizam.
- Detecta regressões visuais e funcionais em PRs antes do merge.
- Integração com CI/CD via GitHub Actions.

### Possíveis problemas ou limitações

- Requer tráfego em staging para gravar sessões — não funciona para features novas sem teste inicial.
- Cobertura é limitada pelos fluxos que usuários realmente percorrem — edge cases raramente visitados ficam descobertos.
- Dependência de uma plataforma SaaS — lock-in e custo por sessão gravada.

### Exemplo prático

```yaml
# .github/workflows/meticulous.yml
name: Meticulous
on: [push]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: alwaysmeticulous/report-diffs-action@v1
        with:
          api-token: ${{ secrets.METICULOUS_API_TOKEN }}
```

### Relação com o ecossistema moderno

- **Next.js/Nuxt**: funciona com qualquer framework que roda no browser.
- **CI/CD**: primeiro-class com GitHub Actions.
- **Design Systems**: detecta regressões visuais em componentes compartilhados.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — especialmente para times com baixa coverage de testes front-end.

---

## 16. AI e Observabilidade Front-end — Embrace.io e RUM com IA

### O que é

O Embrace.io e plataformas similares de Real User Monitoring (RUM) estão integrando IA para análise de padrões de comportamento de usuário que revelam problemas de performance antes que se tornem incidentes. Em vez de alertas reativos de INP ou CLS, a IA detecta correlações entre componentes específicos, condições de rede e degradações de UX.

### Por que isso importa

RUM tradicional gera dados brutos que requerem análise manual. IA transforma esses dados em insights acionáveis: "o componente ProductGallery degrada INP em 40% em dispositivos Android com conexão 3G" — um insight que seria impossível derivar manualmente de métricas agregadas.

### Benefícios práticos

- Análise de correlação automática entre versões de deploy e degradação de métricas.
- Detecção de regressões em Core Web Vitals por segmento de usuário (device, browser, região).
- Alertas preditivos baseados em padrões históricos — avisa antes do usuário reportar.
- 66% dos times de frontend pesquisados pelo Embrace.io planejam integrar IA em observabilidade em 2026.

### Possíveis problemas ou limitações

- Custo de RUM + IA pode ser proibitivo para startups — ferramentas gratuitas como Vercel Speed Insights têm menos granularidade.
- Dados de usuário real requerem compliance (LGPD/GDPR) — coleta cuidadosa de metadados.
- IA ainda não consegue distinguir entre regressão intencional (nova feature pesada) e não intencional.

### Exemplo prático

```ts
// Instrumentação básica com web-vitals + endpoint de coleta
import { onCLS, onINP, onLCP } from 'web-vitals'

const vitalsEndpoint = '/api/vitals'

function sendToAnalytics(metric) {
  fetch(vitalsEndpoint, {
    method: 'POST',
    body: JSON.stringify({
      ...metric,
      url: window.location.href,
      userAgent: navigator.userAgent,
      connection: (navigator as any).connection?.effectiveType,
    }),
  })
}

onCLS(sendToAnalytics)
onINP(sendToAnalytics)
onLCP(sendToAnalytics)
```

### Relação com o ecossistema moderno

- **Next.js**: Vercel Speed Insights é o caminho de menor fricção.
- **Nuxt**: `@nuxtjs/web-vitals` para coleta.
- **Edge Runtime**: métricas de edge functions enriquecem o contexto de RUM.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — observabilidade front-end com IA é diferencial competitivo.

---

## 17. AI e SSR/Edge — Partial Pre-rendering e Streaming em Paralelo

### O que é

O Partial Pre-rendering (PPR) do Next.js e técnicas equivalentes em Nuxt (hybrid rendering) permitem que partes estáticas de uma página sejam servidas instantaneamente do edge, enquanto partes dinâmicas (incluindo chamadas a LLMs) são streamadas progressivamente. Isso combina o melhor do SSG e SSR sem comprometer a personalização.

### Por que isso importa

Features de IA (AI summaries, recommendations, chat) são inerentemente dinâmicas e lentas — o modelo precisa processar antes de responder. PPR + streaming resolve o dilema: o shell da página carrega em <100ms do edge, e o conteúdo de IA streama progressivamente conforme é gerado.

### Benefícios práticos

- LCP excelente mesmo com features de IA — o skeleton carrega instantaneamente.
- Streaming progressivo de conteúdo de IA sem bloquear a renderização da página.
- Edge + Node.js runtime na mesma página — estático no edge, dinâmico no Node.
- Compatível com Suspense boundaries para UX de loading granular.

### Possíveis problemas ou limitações

- PPR ainda está em experimental no Next.js — não recomendado para produção sem feature flag consciente.
- A configuração correta de Suspense boundaries requer planejamento arquitetural cuidadoso.
- Streaming de AI em Nuxt via Nitro tem suporte mais limitado que Next.js App Router.

### Exemplo prático

```tsx
// app/dashboard/page.tsx — PPR com AI streaming
import { Suspense } from 'react'
import { StaticHeader } from './StaticHeader' // Pre-rendered no edge
import { AIInsights } from './AIInsights'    // Streamado dinamicamente

export default function Dashboard() {
  return (
    <>
      <StaticHeader />  {/* Carrega em <100ms do edge */}
      <Suspense fallback={<InsightsSkeleton />}>
        <AIInsights />  {/* Streama conforme o modelo processa */}
      </Suspense>
    </>
  )
}
```

### Relação com o ecossistema moderno

- **Next.js**: PPR é exclusivo do App Router — requer `experimental.ppr: true`.
- **Edge Runtime**: shell estático serve do Vercel Edge, Cloudflare Pages, etc.
- **React Suspense**: boundaries são o mecanismo de controle do streaming.

### Vale a pena acompanhar?

**Ainda está muito cedo para PPR em produção** — mas o padrão arquitetural já deve ser aprendido.

---

## 18. AI para Debugging Front-end — Stack Traces Contextuais

### O que é

IDEs como Cursor, Kiro e Claude Code em 2026 integram debugging contextual assistido por IA: ao invés de copiar um stack trace manualmente para um chat, o agente captura automaticamente o erro, o componente pai, o estado de props no momento do crash e o último commit relevante, e gera uma hipótese de causa raiz com sugestão de fix.

### Por que isso importa

O debugging front-end perdeu horas de produtividade em loops de "copia stack trace → vai no chat → pega sugestão → aplica → novo erro". Com debugging contextual, o agente tem acesso ao código, ao histórico de git e ao runtime state simultaneamente — reduzindo o ciclo de investigação.

### Benefícios práticos

- Correlação automática entre erros de runtime e mudanças recentes no git (`git blame` automático).
- Sugestão de breakpoints relevantes baseada na análise do stack trace.
- Detecção de padrões de erro recorrentes e propostas de correção sistêmica, não pontual.
- Kiro: hook system dispara análise de erros automaticamente em cada save que quebra um test.

### Possíveis problemas ou limitações

- O contexto capturado automaticamente pode incluir dados sensíveis — é necessário configurar o que é enviado para o modelo.
- Hipóteses de causa raiz de IA ainda erram em bugs de timing e race conditions — requer verificação humana.
- Kiro hooks de debug funcionam melhor em projetos com boa cobertura de testes — sem testes, o sinal é fraco.

### Exemplo prático

```
// Fluxo de debugging com Claude Code:
1. `npm test` falha com: "TypeError: Cannot read property 'map' of undefined at ProductList.vue:45"
2. Claude Code analisa automaticamente:
   - ProductList.vue linha 45
   - Props passadas para o componente
   - Último commit que tocou ProductList.vue
   - Type definition de `products`
3. Hipótese gerada: "O componente assume que `products` é sempre um array, 
   mas a prop pode ser undefined durante o loading state. 
   Adicione optional chaining: `products?.map(...)` ou um default prop."
```

### Relação com o ecossistema moderno

- **Vue DevTools**: integração com Claude Code para inspecionar estado de componentes.
- **React DevTools**: Cursor consegue ler o component tree para contexto de debugging.
- **Vitest/Jest**: logs de falha de teste são input para análise de IA.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — debugging contextual é um dos maiores ganhos de produtividade práticos.

---

## 19. Svelte 5 + Runes — Reatividade Explícita Melhor Compreendida por IA

### O que é

O Svelte 5, lançado com o sistema de Runes (`$state`, `$derived`, `$effect`, `$props`), mudou a reatividade de implícita (mágica para humanos e IA) para explícita (declarativa e previsível). Isso tem um impacto direto na qualidade do código gerado por LLMs: o Svelte 4 com reatividade implícita gerava código incorreto com frequência. O Svelte 5 com Runes é significativamente melhor compreendido pelos modelos atuais.

### Por que isso importa

Frameworks com "magia" implícita (two-way binding implícito, reatividade baseada em assignment) são difíceis para LLMs gerarem corretamente. O sistema de Runes torna o código Svelte mais parecido com TypeScript/React — um território que os modelos dominam. Isso aumenta a confiabilidade do código gerado por IA para projetos Svelte.

### Benefícios práticos

- `$state()` e `$derived()` são primitivos explícitos — o modelo sabe exatamente o que está fazendo.
- Menos alucinações em componentes Svelte gerados por Claude, GPT e Gemini.
- `$props()` declara explicitamente a interface do componente — melhora type inference.
- SvelteKit com `load` functions bem tipadas é facilmente gerado por IA.

### Possíveis problemas ou limitações

- A migração de Svelte 4 para 5 requer refatoração da sintaxe de reatividade — não é automática.
- O ecossistema de componentes UI para Svelte ainda é menor que React e Vue.
- LLMs ainda confundem `$state` com `useState` em projetos mistos.

### Exemplo prático

```svelte
<script lang="ts">
  // Svelte 5 com Runes — explícito e compreensível por IA
  const { initialCount = 0 } = $props<{ initialCount?: number }>()
  
  let count = $state(initialCount)
  let doubled = $derived(count * 2)
  
  $effect(() => {
    document.title = `Count: ${count}`
  })
</script>

<button onclick={() => count++}>
  Count: {count} (doubled: {doubled})
</button>
```

### Relação com o ecossistema moderno

- **SvelteKit**: a pair natural para Svelte 5 em apps full-stack.
- **Vite**: Svelte usa Vite como bundler nativo.
- **Design Systems**: bibliotecas como Melt UI e shadcn-svelte adotaram Runes.

### Vale a pena acompanhar?

**Sim, vale acompanhar** — especialmente se o time usa Svelte ou considera adoção.

---

## 20. Frontend Modernization com IA — Legado para Stack Moderna

### O que é

Uma categoria emergente de ferramentas em 2026 é a de modernização automatizada de frontends legados: migração de jQuery para Vue/React, de Class Components para hooks, de Webpack para Vite, de JavaScript para TypeScript. O AWS Transform (integrado ao Kiro e VS Code) é o exemplo mais expressivo: usa IA para analisar, planejar e executar migrações de codebase em escala.

### Por que isso importa

Grandes empresas têm frontends legados que são risco de segurança, performance e produtividade. A modernização manual é cara e lenta. IA consegue realizar o trabalho mecânico da migração (converter sintaxe, atualizar APIs, adicionar tipos) liberando o time para decisões arquiteturais.

### Benefícios práticos

- AWS Transform: analisa codebase legado, gera plano de migração, executa em batches.
- Claude Code com Dynamic Workflows: divide migrações grandes em subagentes paralelos.
- Conversão automática de `require()` para ESM `import/export` sem perda de funcionalidade.
- Adição de tipos TypeScript com inferência baseada no comportamento de runtime.

### Possíveis problemas ou limitações

- Migrações automáticas produzem código que compila mas não necessariamente que funciona — testes são obrigatórios.
- Dependências com APIs quebradas entre versões requerem intervenção manual.
- O custo de tokens para migrar um codebase grande pode ser significativo.

### Exemplo prático

```
// Fluxo de modernização com Claude Code:

1. "Analise o diretório src/legacy e liste todos os padrões jQuery que
   precisam ser migrados para Vue 3. Categorize por complexidade."

2. "Para cada padrão de baixa complexidade listado, gere o equivalente
   Vue 3 com Composition API. Não altere o comportamento — apenas a sintaxe."

3. "Execute a migração dos 10 componentes de menor complexidade.
   Crie testes para cada um antes de migrar."
```

### Relação com o ecossistema moderno

- **Vite**: o destino natural de migrações de Webpack.
- **TypeScript**: adição gradual de tipos é o padrão recomendado.
- **CI/CD**: pipeline de testes é pré-requisito para qualquer modernização automatizada.
- **AWS Kiro**: oferece a UI mais integrada para modernização com IA hoje.

### Vale a pena acompanhar?

**Promissor para empresas** — o ROI é alto em projetos legados grandes, mas requer investimento inicial em testes.

---

*Fontes principais: [Nuxt UI Docs](https://ui.nuxt.com/docs/components/chat) · [Figma Code Connect](https://www.figma.com/solutions/ai-design-systems-generator/) · [Storybook 9 Blog](https://storybook.js.org/blog/storybook-9-beta/) · [Core Web Vitals 2026](https://maxtdesign.com/insights/marketing/seo-optimization/core-web-vitals-2026-the-technical-seo-blueprint/) · [Vercel AI SDK](https://ai-sdk.dev/) · [Mastra Framework](https://mastra.ai/) · [Vue School AI Course](https://vueschool.io/courses/ai-interfaces-with-vue-nuxt-and-the-ai-sdk) · [Builder.io React AI Stack](https://www.builder.io/blog/react-ai-stack-2026)*
