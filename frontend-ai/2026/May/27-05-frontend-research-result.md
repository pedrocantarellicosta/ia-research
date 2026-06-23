# Front-End AI Weekly — Novidades Front-End | 27 de Maio de 2026

> **Data de geração:** 27 de Maio de 2026
> **Período coberto:** 20–27 de Maio de 2026
> **Arquivo:** 10 Novidades Front-End
> **Curadoria:** Skill `frontend-ia-weekly` via Claude Sonnet 4.6

---

## 1. Next.js 16.2 — AGENTS.md e Documentação Versionada Empacotada para Agentes de IA

### O que é

O Next.js lançou a versão 16.2 com uma mudança que afeta diretamente a qualidade do código gerado por qualquer agente de IA trabalhando em projetos Next.js. O problema central que a atualização resolve: agentes como Claude Code, Cursor e GitHub Copilot usam dados de treinamento para gerar código Next.js — e esses dados têm meses ou anos de atraso em relação à versão instalada em produção.

A partir da **v16.2.0-canary.37**, o Next.js empacota a documentação completa, versionada e correspondente ao build instalado, dentro do próprio pacote npm em `node_modules/next/dist/docs/`. Isso significa que ao rodar `npm install`, você instala não apenas o runtime do Next.js, mas também a documentação técnica exata para aquela versão — diretamente acessível ao agente que trabalha no projeto.

O segundo componente é o **AGENTS.md**: um arquivo de configuração na raiz do projeto que instrui qualquer agente a ignorar dados de treinamento desatualizados sobre Next.js e consultar a documentação empacotada. O `create-next-app` agora gera `AGENTS.md` e `CLAUDE.md` automaticamente. Para projetos existentes, basta estar na v16.2+.

Os benchmarks da Vercel são expressivos: um índice de 8KB comprimido embedado diretamente no `AGENTS.md` atingiu **100% de pass rate** em avaliações internas, enquanto agent skills atingiram no máximo 79%, mesmo com instruções explícitas para usá-las.

O arquivo tem marcadores gerenciados automaticamente:
```
<!-- BEGIN:nextjs-agent-rules -->
... seção gerenciada pelo Next.js, atualiza em cada versão ...
<!-- END:nextjs-agent-rules -->
```

Fora dos marcadores, o time pode adicionar instruções de projeto sem risco de sobrescrita em updates futuros.

### Por que isso importa

O problema de agentes gerando código desatualizado é um dos mais frustrantes no desenvolvimento com IA. Um agente treinado com dados de 2024 vai sugerir `getStaticProps` em vez de `generateStaticParams`, `pages/` em vez de `app/`, e padrões de `_app.tsx` em vez de Server Components. Com documentação empacotada e `AGENTS.md`, o agente tem acesso direto à fonte da verdade: a versão exata instalada no projeto.

### Benefícios práticos

- Agentes geram código alinhado com a versão exata do Next.js instalada
- 100% de pass rate em benchmarks vs 79% com skills externas
- `create-next-app` gera `AGENTS.md` e `CLAUDE.md` automaticamente — zero configuração para projetos novos
- Compatível com Claude Code, Cursor, GitHub Copilot, Gemini CLI e qualquer agente que leia `AGENTS.md`
- Seções gerenciadas e customizáveis coexistem sem conflito

### Possíveis problemas ou limitações

- Requer v16.2.0-canary.37 ou posterior — projetos em versões anteriores não têm o recurso
- Adiciona ~8KB (comprimido) ao pacote — trade-off pequeno mas existente
- Agentes que não leem `AGENTS.md` não se beneficiam automaticamente
- A documentação empacotada é somente leitura — não captura customizações locais do framework

### Exemplo prático

```markdown
# AGENTS.md (gerado pelo create-next-app)

<!-- BEGIN:nextjs-agent-rules -->
## Next.js Documentation Access

This project uses Next.js 16.2.x. Before any Next.js work, find and read
the relevant doc in `node_modules/next/dist/docs/`.

Your training data is outdated — the docs are the source of truth.

Available doc categories:
- app/building-your-application/routing/
- app/building-your-application/rendering/
- app/building-your-application/data-fetching/
- app/api-reference/components/
<!-- END:nextjs-agent-rules -->

## Instruções do projeto (não gerenciadas pelo Next.js)
- Monorepo Turborepo: apps/web, apps/admin, packages/ui
- Todos os componentes importam de packages/ui
- Server Components por padrão — Client Components apenas quando necessário
- Conventional Commits
```

```bash
# Para projetos existentes na v16.2+
npx next setup-agents
```

### Relação com o ecossistema moderno

- **Turborepo/Monorepos**: o `AGENTS.md` na raiz convive com `CLAUDE.md` e pode cobrir pacotes compartilhados
- **React Server Components**: a doc empacotada inclui a guia de RSC exata para a versão — sem ambiguidade de padrões
- **CI/CD**: como `AGENTS.md` é versionado em git, toda a equipe e pipelines de CI trabalham com as mesmas instruções
- **Design Systems**: o espaço fora dos marcadores é ideal para documentar convenções do design system que o agente deve respeitar

### Vale a pena acompanhar?

**Sim, vale acompanhar — e implementar imediatamente.** Para qualquer projeto Next.js ativo, configurar `AGENTS.md` é uma mudança de 10 minutos com impacto direto na qualidade de todas as interações com agentes. O delta de 100% vs 79% no pass rate é mensurável e qualquer time vai sentir na prática.

---

## 2. Svelte Agentation — Anotações de UI como Contexto Estruturado para Agentes

### O que é

Svelte Agentation (`sv-agentation.com`) é uma extensão de browser que resolve um problema cotidiano no fluxo com agentes: como dar ao agente contexto preciso sobre um elemento específico da UI sem ter que descrever em texto onde ele está, qual componente o gerou e o que deve mudar.

O fluxo é direto: você abre a aplicação com a extensão ativa, clica em qualquer elemento DOM, adiciona uma anotação textual, e copia o output estruturado gerado diretamente para Claude Code, Cursor ou qualquer ferramenta de IA.

A saída inclui:
- O **seletor source-aware**: referência ao componente Svelte e linha de origem (ex: `src/lib/components/Button.svelte:47`), não o seletor CSS frágil
- O **HTML e CSS computado** do elemento e contexto ao redor
- O **bounding box** exato na tela
- A **anotação textual** que você adicionou

O "source-aware DOM inspection" é o diferencial técnico: a extensão rastreia o elemento do DOM de volta ao arquivo de componente Svelte que o gerou. Anotações são isoladas por página — ao mudar de rota, as notas do contexto anterior não poluem o novo. Markers visuais na tela mostram onde notas foram salvas, com preview ao hover.

### Por que isso importa

Descrever em texto onde está um elemento e o que deve mudar é lento e impreciso. "O botão no header direito da navbar" é ambíguo. "O `Button.svelte:47` com classe `primary` que renderiza na `NavBar.svelte:23`" não é — e é exatamente isso que a extensão entrega como contexto para o agente. Para times que fazem revisão de design ou QA visual, isso transforma feedback em algo que um agente pode atuar diretamente.

### Benefícios práticos

- Elimina descrições textuais imprecisas de mudanças de UI
- Referência direta ao arquivo de componente Svelte, não ao seletor CSS frágil
- Notas por rota não se misturam ao navegar — fluxo limpo em SPAs
- Funciona com Claude Code, Cursor, Copilot CLI e qualquer ferramenta que aceite texto estruturado
- Callbacks customizáveis para integrar com tooling interno ou sistemas de métricas

### Possíveis problemas ou limitações

- **Específico para Svelte**: o source-aware inspection depende das particularidades do DOM do Svelte — não funciona out-of-the-box com React, Vue ou outros frameworks
- **Sem sync entre membros do time**: as notas são locais ao browser — não há mecanismo de compartilhamento
- **Early stage**: a API de callbacks pode mudar; a extensão tem superfície intencionalmente pequena
- **SSR com hidratação parcial**: pode criar casos onde o source mapping não funciona corretamente

### Exemplo prático

```
# Fluxo com Svelte Agentation:

1. Abre o app com a extensão ativa
2. Clica no elemento problemático
3. Adiciona anotação: "margem errada, deveria ser gap-4 (16px),
   igual ao padrão de spacing do design system"
4. Copia o output estruturado:

---
Element: src/lib/components/cards/ProductCard.svelte:89
Selector: .product-card > .footer > .price-block
Computed styles: margin-top: 8px (expected: 16px)
Bounding box: {x: 120, y: 480, width: 280, height: 32}
Annotation: "margem errada, deveria ser gap-4 (16px),
             igual ao padrão de spacing do design system"
---

5. Cola no Claude Code com o prompt: "Corrija o elemento anotado
   abaixo seguindo o design system:" + [paste output]
```

```javascript
// Callback para integrar com sistema de tracking interno
svagentation.configure({
  onAnnotationSave: (annotation) => {
    analytics.track('design-feedback', {
      component: annotation.sourceFile,
      issue: annotation.text,
      page: window.location.pathname,
    });
  },
  onAnnotationCopy: (payload) => {
    // Adiciona versão do design system ao contexto automaticamente
    return `${payload}\n\n[Design System]: ${designSystemVersion}`;
  }
});
```

### Relação com o ecossistema moderno

- **SvelteKit**: o source-aware tracking funciona especialmente bem com o roteamento do SvelteKit, onde rotas correspondem a componentes específicos
- **Storybook**: pode anotar stories com feedback de design antes de passar ao agente
- **Design Systems**: o output estruturado facilita comunicar qual token ou variante do design system deve ser aplicada
- **CI/CD**: com callbacks, anotações podem ser convertidas em issues do GitHub automaticamente antes de sprints de design review

### Vale a pena acompanhar?

**Sim, vale acompanhar para times Svelte.** O impacto no fluxo é real — transforma feedback visual em contexto preciso para agentes. Para times em outros frameworks, vale acompanhar como indicador de tendência: "source-aware DOM annotation" vai chegar para React/Vue em breve.

---

## 3. Chrome DevTools MCP — Agentes de IA que Debugam o Frontend Diretamente no Browser

### O que é

Chrome DevTools MCP (`chrome-devtools-mcp`, pacote npm oficial) é um servidor MCP do Google Chrome DevTools team que resolve o problema mais fundamental de agentes em frontend: eles geram código sem conseguir ver o que o código faz quando roda no browser. Está na **v0.21.0** após 43 releases em 7 meses — cadência de aproximadamente uma release por semana. É Apache-2.0 e mantido diretamente pela equipe do Chrome.

O servidor expõe as seguintes ferramentas para o agente:

**Performance:**
- `record_performance_trace()` — grava trace e extrai insights acionáveis (LCP, CLS, TBT)
- `run_lighthouse_audit()` — auditoria completa de performance, acessibilidade, SEO e retorna o relatório completo
- `get_crux_data(url)` — retorna dados reais de usuários do Chrome UX Report (CrUX)

**Debugging:**
- `get_console_logs()` — retorna logs com **source-mapped stack traces**: mapeia de volta ao TypeScript/JSX original, não ao bundle minificado
- `get_network_requests()` — requests com headers, payloads e status codes
- `take_screenshot()` — captura o estado visual atual

**Memory:**
- `take_heap_snapshot()` — snapshot do heap para identificar memory leaks (detached DOM nodes, closures acumuladas)

Em maio de 2026, o Chrome team adicionou conexão direta a **sessões ativas do browser** — sem exigir login extra. Você pode estar navegando em `localhost:3000` autenticado e o agente se conecta àquela sessão para debugar em tempo real.

### Por que isso importa

Antes do Chrome DevTools MCP, o ciclo de debug era cego: agente gera código → você roda → vê o erro → copia para o agente → agente corrige sem entender o estado completo. Com o MCP, o agente vê o console log com stack trace mapeado para o arquivo original, o trace de performance com o gargalo identificado, e o screenshot para confirmar o layout. O loop "código → rodar → informar agente" colapsa para "código → agente verifica diretamente".

### Benefícios práticos

- **Source-mapped stack traces**: erros apontam para o arquivo TypeScript/JSX original, não para o bundle minificado
- **Lighthouse integrado no loop de debug**: auditoria completa, identificação de issues e geração de fixes sem intervenção humana
- **CrUX real-user data**: verificação com dados de usuários reais, não apenas sintético
- **Heap snapshots para memory leaks**: diagnóstico de `detached DOM nodes` e closures acumuladas
- **Conexão a sessão ativa**: debug de features que requerem autenticação sem precisar re-logar

### Possíveis problemas ou limitações

- **Source maps devem estar disponíveis**: projetos sem source maps em staging/produção perdem o benefício principal
- **Latência por tool call**: em sessões de debug longas com muitas ferramentas, a latência se acumula
- **CrUX tem lag de 28 dias**: não reflete deploys recentes
- **Sessões ativas com estado complexo**: feature flags, dados mockados e autenticação multifator podem criar comportamento difícil de reproduzir

### Exemplo prático

```json
// .claude/mcp.json ou claude_desktop_config.json
{
  "mcpServers": {
    "chrome-devtools": {
      "command": "npx",
      "args": ["chrome-devtools-mcp@latest"]
    }
  }
}
```

```
# Prompt para o agente — debug de memory leak:
"O componente InfiniteScroll está causando memory leak em sessões longas.
 Use o Chrome DevTools MCP para:
 1. Tirar um heap snapshot inicial
 2. Simular scroll até o final da lista 3x
 3. Tirar outro heap snapshot
 4. Comparar os snapshots e identificar o que está crescendo
 5. Gerar a fix"

# O agente executa, identifica EventListeners não removidos no
# unmount e gera a fix com useEffect cleanup correto.
```

```
# Prompt para verificação de performance pré-deploy:
"Rode auditoria Lighthouse em http://localhost:3000.
 Se LCP > 2.0s ou score de acessibilidade < 90,
 identifique o problema e proponha a fix."
```

### Relação com o ecossistema moderno

- **React/Next.js**: source-mapped stack traces são críticos em Next.js App Router, onde erros traversam RSC → Client Component com stack traces longos
- **Vite**: source maps ativados por padrão em dev — compatibilidade imediata
- **Design Systems**: combinado com Lighthouse, o agente verifica se mudanças em componentes do design system introduziram regressões de acessibilidade ou performance
- **CI/CD**: pode ser integrado em pipelines headless via Chrome headless — audits automáticos em cada PR

### Vale a pena acompanhar?

**Sim, vale acompanhar — e já configurar.** A instalação leva 5 minutos e o impacto no debug é imediato. Uma release por semana demonstra comprometimento do Chrome team. Para times que usam Claude Code ou Cursor, este é um dos investimentos com maior ROI disponíveis hoje.

---

## 4. WebMCP — O Padrão Web Aberto para Expor Ferramentas a Agentes de Browser

### O que é

WebMCP (Web Model Context Protocol) é um padrão web proposto conjuntamente por **Google e Microsoft** no W3C Web Machine Learning Community Group, anunciado no Google I/O 2026 e disponível em origin trial no Chrome 149.

O problema: agentes de IA que operam em browsers hoje funcionam como usuários com visão mas sem inteligência sobre intenção — scrapeiam visualmente, clicam em elementos por aparência, preenchem forms campo a campo. Isso é lento, frágil e impreciso para tarefas complexas.

WebMCP permite que desenvolvedores **exponham explicitamente ferramentas estruturadas** a agentes de browser: funções JavaScript com schemas tipados, anotações em HTML forms, e intenções declaradas. O agente, em vez de "adivinhar" como navegar pelo DOM, recebe uma API declarativa do que a página é capaz de fazer.

**Duas formas de implementação:**

**1. Imperative API (JavaScript):**
```javascript
navigator.webmcp.registerTool({
  name: 'search_flights',
  description: 'Search available flights between two cities',
  schema: {
    type: 'object',
    properties: {
      origin: { type: 'string', description: 'IATA airport code' },
      destination: { type: 'string', description: 'IATA airport code' },
      date: { type: 'string', format: 'date' },
      passengers: { type: 'integer', minimum: 1, maximum: 9 }
    },
    required: ['origin', 'destination', 'date']
  },
  execute: async ({ origin, destination, date, passengers = 1 }) => {
    return await flightAPI.search({ origin, destination, date, passengers });
  }
});
```

**2. Declarative API (HTML form annotation):**
```html
<form webmcp-tool="subscribe_newsletter"
      webmcp-description="Subscribe user to newsletter with preferences">
  <input name="email" type="email"
         webmcp-description="User email address" />
  <select name="frequency"
          webmcp-description="Email frequency preference">
    <option value="weekly">Weekly</option>
    <option value="monthly">Monthly</option>
  </select>
</form>
```

**Enterprise adotando desde o início:** Booking.com, Expedia, Instacart, Intuit, Shopify e Redfin estão participando do origin trial.

### Por que isso importa

WebMCP representa uma mudança arquitetural na relação entre websites e IA. Hoje, um agente que visita seu e-commerce tenta entender a UI como humano. Com WebMCP, você define explicitamente: `search_products`, `add_to_cart`, `checkout`. O agente opera com precisão de API, não com incerteza visual. Para desenvolvedores, cria uma nova camada de abstração: assim como você expõe uma REST API para integrações de terceiros, você expõe uma WebMCP API para integrações de agentes.

### Benefícios práticos

- **Precisão de API para agentes no browser**: elimina erros de scraping visual
- **Sem servidor extra**: a lógica fica no JavaScript do frontend — sem backend adicional
- **Compatível com HTML existente**: a Declarative API aproveita forms existentes, sem reescrever lógica
- **Authorization built-in**: WebMCP trabalha com o modelo de permissão do browser
- **Padrão com 2 browsers alinhados**: Google e Microsoft na mesma spec desde o início — raro e forte

### Possíveis problemas ou limitações

- **Origin trial apenas**: Chrome 149 via opt-in — não é para produção ainda
- **Sem fallback automático**: browsers que não suportam WebMCP ignoram os atributos
- **Segurança é responsabilidade do dev**: ferramentas mal-escritas podem ser exploradas por agentes mal-configurados
- **W3C process é lento**: mesmo com Google e Microsoft alinhados, standardização completa pode levar 2-3 anos
- **Adoção bilateral necessária**: o padrão só tem valor se websites o implementarem proativamente

### Exemplo prático

```javascript
// E-commerce expondo toolset completo para agentes
if ('webmcp' in navigator) {
  navigator.webmcp.registerToolset({
    name: 'ShopTools',
    tools: [
      {
        name: 'search_products',
        description: 'Search catalog by query and filters',
        schema: {
          type: 'object',
          properties: {
            query: { type: 'string' },
            category: { type: 'string', enum: ['electronics', 'clothing', 'home'] },
            maxPrice: { type: 'number' },
            sortBy: { type: 'string', enum: ['relevance', 'price', 'rating'] }
          }
        },
        execute: async (params) => await catalogAPI.search(params)
      },
      {
        name: 'add_to_cart',
        description: 'Add product to current user cart',
        schema: {
          type: 'object',
          properties: {
            productId: { type: 'string' },
            quantity: { type: 'integer', minimum: 1 }
          }
        },
        permissions: ['cart:write'],
        execute: async (params) => await cartAPI.add(params)
      }
    ]
  });
}
```

### Relação com o ecossistema moderno

- **Next.js App Router**: ferramentas WebMCP podem ser registradas em Client Components com Server Actions como executores
- **React/Vue/Svelte**: a Imperative API é framework-agnostic
- **Design Systems**: components como forms e dialogs podem expor ferramentas WebMCP declarativamente via anotações HTML
- **Edge Runtime**: a Declarative API não requer JavaScript — funciona em páginas estáticas e SSR
- **Microfrontends**: cada microfrontend pode registrar seu próprio toolset WebMCP, compondo uma API de agentes distribuída

### Vale a pena acompanhar?

**Sim, vale acompanhar — ainda muito cedo para produção, mas é o futuro.** A co-autoria Google/Microsoft na spec é o sinal mais forte possível de adoção futura. Para equipes que constroem produtos onde agentes vão interagir (e-commerce, plataformas, SaaS), estudar a API agora é preparação estratégica para quando o padrão estabilizar.

---

## 5. TanStack AI — SDK de IA Provider-Agnóstico com Type Safety Total

### O que é

TanStack AI é um SDK TypeScript open-source do criador do TanStack Query, Form, Table e Router. A proposta é ser para IA o que o TanStack Query é para fetching de dados: uma abstração framework-agnostic com type safety de primeira classe e sem lock-in de provider.

O SDK se posiciona em frente ao Vercel AI SDK: enquanto o AI SDK está otimizado para Next.js/Vercel, o TanStack AI é genuinamente agnóstico — funciona em React, Solid, Vue, Svelte, Preact e runtimes headless.

**Providers suportados:** OpenRouter, OpenAI, Anthropic, Gemini, Ollama, Groq, Grok/xAI, ElevenLabs, fal.ai.

**Módulos tree-shakeable:**
```
@tanstack/ai            → core + chat
@tanstack/ai/image      → geração de imagem
@tanstack/ai/audio      → áudio e speech
@tanstack/ai/realtime   → voz em tempo real
@tanstack/ai/devtools   → devtools integradas
@tanstack/ai/react      → hooks React
@tanstack/ai/vue        → composables Vue
@tanstack/ai/svelte     → stores Svelte
```

**Type safety via Zod inference:**
```typescript
const tools = {
  searchProducts: {
    parameters: z.object({
      query: z.string().describe('Search query'),
      maxResults: z.number().optional().default(10),
    }),
    execute: async (params) => await db.search(params),
  },
};
// O tipo do resultado é inferido automaticamente — zero "any"
```

**Isomorphic tools** — mesma definição, implementação diferente por ambiente:
```typescript
const fileReader = createIsomorphicTool({
  server: async (path) => fs.readFileSync(path),
  browser: async (path) => await fetch(path).then(r => r.text()),
});
```

### Por que isso importa

O Vercel AI SDK tem lock-in de ecossistema: está otimizado para Next.js e o deploy stack da Vercel. Para times que usam Nuxt, SvelteKit, SolidStart ou stacks customizadas, o TanStack AI é o candidato natural — mesma DX, sem dependência de plataforma. A type safety via Zod inference também endereça um problema real: manter tipos sincronizados entre definições de ferramentas e componentes que usam os resultados é custoso em apps com muitas integrações de IA.

### Benefícios práticos

- Sem lock-in de framework ou provider — troque de OpenAI para Anthropic sem mudar código do componente
- Tree-shaking granular: importe apenas o que usa
- Zod inference elimina type drift entre definições de ferramentas e componentes
- Isomorphic tools: mesma definição com implementações específicas por ambiente
- Devtools integradas — visualiza estado de conversas, tool calls e streaming em desenvolvimento
- React Native support nos hooks React sem adaptadores especiais

### Possíveis problemas ou limitações

- **Alpha instável**: múltiplas breaking changes arquiteturais desde o lançamento — não adequado para produção no momento
- **Ecossistema menor**: menos exemplos, plugins e integrações prontas vs Vercel AI SDK
- **Curva de aprendizado**: isomorphic tools exigem clareza sobre o ambiente de execução
- **Streaming inconsistente com alguns providers locais**: Ollama e providers locais têm comportamento de streaming variável no browser

### Exemplo prático

```typescript
// React — pode trocar de provider sem mudar o componente
import { useChat } from '@tanstack/ai/react';
import { anthropic } from '@tanstack/ai/anthropic';
import { z } from 'zod';

const ProductSearchChat = () => {
  const { messages, send, isStreaming } = useChat({
    // Troque por: openai('gpt-5.5') ou gemini('gemini-3.5-flash')
    // sem mudar mais nada
    model: anthropic('claude-sonnet-4-6'),
    tools: {
      searchProducts: {
        parameters: z.object({
          query: z.string(),
          priceRange: z.object({
            min: z.number().optional(),
            max: z.number().optional(),
          }).optional(),
        }),
        execute: async (params) => await catalogAPI.search(params),
      },
    },
    system: 'You are a shopping assistant. Help users find products.',
  });

  return (
    <div>
      {messages.map((msg) => <MessageBubble key={msg.id} message={msg} />)}
      <ChatInput onSend={send} disabled={isStreaming} />
    </div>
  );
};
```

```vue
<!-- Vue — mesmas capabilities com composables -->
<script setup>
import { useChat } from '@tanstack/ai/vue';
import { gemini } from '@tanstack/ai/gemini';

const { messages, send, isStreaming } = useChat({
  model: gemini('gemini-3.5-flash'),
  tools: { /* mesmas definições */ },
});
</script>
```

### Relação com o ecossistema moderno

- **SvelteKit / Nuxt / SolidStart**: diferente do Vercel AI SDK, TanStack AI tem suporte de primeira classe para esses frameworks sem gambiarras
- **Turborepo**: um pacote `packages/ai-tools` com definições isomorphic pode ser compartilhado entre `apps/web` (React), `apps/mobile` (React Native) e `apps/admin` (Vue)
- **Vite**: tree-shaking granular maximamente eficiente com Vite
- **Design Systems**: devtools visualizam estado de conversas em desenvolvimento — útil para depurar como o agente usa ferramentas do design system

### Vale a pena acompanhar?

**Ainda está muito cedo para produção — mas é um projeto estratégico.** Para quem está no ecossistema Vercel/Next.js, o Vercel AI SDK v6 é a escolha certa agora. Para times em Nuxt, SvelteKit ou SolidStart, coloque o TanStack AI no radar e avalie em Q3 2026 quando espera-se uma versão beta mais estável.

---

## 6. Modern Web Guidance — 128 Features Web em 100+ Casos Verificados para Agentes de IA

### O que é

Modern Web Guidance é um conjunto de skills verificadas por especialistas do Google, lançado em early preview no Google I/O 2026. Resolve um problema sistêmico: agentes de IA têm dados de treinamento desatualizados sobre a plataforma web, resultando em código que usa APIs obsoletas, ignora recursos modernos do browser e frequentemente viola melhores práticas de acessibilidade, performance e segurança.

O projeto empacota **128 features da plataforma web** em **mais de 100 casos de uso verificados**, todas alinhadas com o **Web Platform Baseline** — a definição oficial do que é amplamente suportado por browsers modernos.

**Instalação em um comando:**
```bash
npx modern-web-guidance install
# Configura automaticamente: Claude Code, GitHub Copilot CLI, Gemini CLI, Antigravity
```

**Impacto medido:** benchmarks internos do Google mostram **+37 pontos percentuais** em aderência a melhores práticas web. Um agente com 50% de aderência passa para ~87%.

**Categorias de skills:**
- **Acessibilidade**: padrões ARIA, keyboard navigation, contrast ratio, focus management
- **Performance**: Core Web Vitals, lazy loading, critical rendering path
- **Modern CSS**: container queries, cascade layers, CSS nesting, `:has()` selector
- **Security**: CSP, Subresource Integrity, CORS patterns corretos
- **WebAssembly**: integração com JS, workers, performance patterns
- **PWA**: service workers, offline-first, push notifications

A integração com **Baseline** é importante: ao definir um `baseline-target` (ex: `widely-available-2024`), as skills selecionam automaticamente features e fallbacks corretos para aquele target — sem precisar declarar explicitamente o que é suportado.

### Por que isso importa

Qualquer desenvolvedor que trabalha com agentes regularmente reconhece o padrão: o agente usa `onKeyPress` em vez de `onKeyDown`, gera `div[onclick]` onde `button` seria correto, esquece `prefers-reduced-motion`, ou usa `fetch` sem `AbortController` em componentes que desmontam. Esses são bugs de qualidade que vêm da defasagem dos dados de treinamento. Modern Web Guidance não substitui o julgamento do agente — ela fornece regras verificadas por especialistas que o agente aplica ao gerar código web.

### Benefícios práticos

- **+37pp** em aderência a melhores práticas em benchmarks do Google
- **Instalação única**: um comando configura skills nos principais coding agents
- **Integração com Baseline**: sem precisar declarar compatibility — o Baseline target cuida disso
- **Gratuito e open-source**: sem lock-in

### Possíveis problemas ou limitações

- **Early preview**: alguns casos de uso podem estar incompletos
- **Crítica de acessibilidade**: Adrian Roselli publicou artigo crítico ("Maybe Don't Rely on Google's Modern Web Guidance") apontando que algumas skills de acessibilidade são incompletas ou imprecisas — merece leitura antes de adotar em critérios de auditoria
- **Não substitui revisão humana**: +87% ainda significa 13% de problemas não cobertos
- **Viés Google**: pode favorecer APIs Chrome-específicas

### Exemplo prático

```bash
npx modern-web-guidance install
# ✓ Installed 100+ use cases covering 128 web platform features
# ✓ Configured for: Claude Code, GitHub Copilot CLI, Gemini CLI
# ✓ Baseline target: widely-available-2025
```

```typescript
// Antes do Modern Web Guidance (output típico de agente sem skills):
function CustomButton({ onClick, children }) {
  return (
    <div className="btn" onClick={onClick}>  // ❌ div clicável sem acessibilidade
      {children}
    </div>
  );
}

// Com Modern Web Guidance:
function CustomButton({ onClick, children }) {
  return (
    <button
      type="button"
      className="btn"
      onClick={onClick}
      onKeyDown={(e) => {
        if (e.key === 'Enter' || e.key === ' ') onClick(e);
      }}
    >
      {children}
    </button>
  );
}
```

```tsx
// Imagem com todas as boas práticas de CWV aplicadas pelo agente:
<img
  src={hero.url}
  alt={hero.alt}
  width={hero.width}      // ← previne CLS
  height={hero.height}    // ← previne CLS
  loading="eager"         // ← hero image: carrega imediatamente
  fetchPriority="high"    // ← sinaliza prioridade ao browser
  decoding="async"        // ← não bloqueia o main thread
/>
```

### Relação com o ecossistema moderno

- **React/Next.js**: skills cobrem padrões de RSC, Client Components e Suspense
- **Vite**: skills de build calibradas para Vite/Rollup, não apenas Webpack
- **Design Systems**: um design system que instala Modern Web Guidance garante que o agente sempre gera componentes acessíveis e performáticos ao consumir a biblioteca
- **CI/CD**: combinado com Lighthouse CI, verifica que o código gerado pelo agente atinge os scores esperados

### Vale a pena acompanhar?

**Sim, vale acompanhar — mas leia a crítica do Roselli primeiro.** Adote as skills de performance e CSS moderno (menor risco), e valide manualmente as skills de acessibilidade antes de usá-las em critérios de auditoria.

---

## 7. Figma Dev Mode MCP — Design Tokens e Componentes como Contexto Direto para Agentes

### O que é

A Figma lançou em beta o **Dev Mode MCP Server** — um servidor MCP oficial que conecta diretamente os dados do Figma (design tokens, componentes, variáveis, Code Connect) aos agentes de coding como Claude Code e GitHub Copilot.

O fluxo habilitado:
1. Selecione um frame no Figma
2. Copie o link do frame
3. Cole no Claude Code com um prompt
4. O agente acessa o Figma via MCP, lê o contexto completo (layout, componentes, tokens, variantes, espaçamentos) e gera código que corresponde ao design

O servidor envia para o agente:
- **Style e variable usage**: quais tokens de design estão sendo usados no frame
- **Variable code syntax**: a sintaxe exata para referenciar variáveis CSS/tokens no código
- **Code Connect data**: quando configurado, mapeia componentes Figma para componentes de código existentes

Em fevereiro de 2026, a Figma expandiu para o sentido inverso: **"Send this to Figma"** — o estado renderizado de um componente no browser pode ser exportado automaticamente para layers editáveis no Figma.

O **Figma Console MCP** (projeto da comunidade: `github.com/southleft/figma-console-mcp`) vai além: permite criar, atualizar e depurar design systems inteiros via IA, expondo ferramentas para gerenciamento programático de estilos, componentes e variáveis.

### Por que isso importa

O gap mais custoso no desenvolvimento de UI é a tradução de "o que está no Figma" para "o que está no código". Esse processo envolve: abrir o Figma, inspecionar o frame, copiar tokens, verificar qual componente do design system mapeia para qual componente Figma. O Figma MCP torna isso um único passo. Para times com Code Connect configurado, o agente já sabe que `ButtonPrimary` no Figma é `<Button variant="primary">` no código — sem mapeamento manual.

### Benefícios práticos

- Geração de código que corresponde ao design: tokens e variantes exatos do frame
- Zero ambiguidade de tokens: o agente recebe `var(--color-brand-500)`, não uma cor hexadecimal
- Code Connect: o agente usa componentes existentes do design system quando disponíveis
- Bidirecional: código renderizado no browser → layers editáveis no Figma
- Figma Console MCP: gestão programática de design systems inteiros

### Possíveis problemas ou limitações

- **Dev Mode requerido**: disponível apenas em planos Professional e Enterprise do Figma
- **Code Connect é trabalhoso de manter**: o mapeamento Figma → código depende de Code Connect atualizado — overhead real em design systems grandes
- **Output precisa de revisão**: componentes com interações complexas, estados e responsividade ainda precisam de ajuste
- **Rate limits da Figma API**: sessões de geração intensiva em design systems grandes podem atingir rate limits

### Exemplo prático

```bash
# Configuração no .claude/mcp.json
npx figma-developer-mcp --figma-api-key=<sua-api-key>
```

```
# Prompt em Claude Code:
"Implemente o componente Card de produto do frame:
https://www.figma.com/design/[seu-projeto]/[frame-id]

Use os componentes de packages/ui e tokens de packages/tokens.
Não invente tokens — use apenas os que existem no arquivo de variáveis do Figma."

# O agente via MCP:
# 1. Lê o frame → Card com Image, Title, Price, CTA Button
# 2. Code Connect: Card → <Card>, Button → <Button variant="primary">
# 3. Tokens: spacing-4 = 16px, color-brand-600 = var(--color-brand-600)
# 4. Gera o componente TypeScript com imports corretos de packages/ui
```

```typescript
// Output do agente com Figma MCP ativo:
import { Card, CardImage, CardBody, Button } from '@company/ui';
import type { Product } from '@company/types';

export function ProductCard({ product }: { product: Product }) {
  return (
    <Card variant="elevated" className="gap-4 p-4">
      <CardImage src={product.imageUrl} alt={product.name} aspectRatio="4/3" />
      <CardBody className="gap-2">
        <h3 className="text-body-lg font-semibold">{product.name}</h3>
        <p className="text-brand-600 text-body-md font-bold">
          {formatPrice(product.price)}
        </p>
        <Button variant="primary" size="md" fullWidth>
          Adicionar ao Carrinho
        </Button>
      </CardBody>
    </Card>
  );
}
```

### Relação com o ecossistema moderno

- **React/Next.js**: Code Connect funciona com React — mapeamento usa syntax JSX
- **Storybook**: Figma Console MCP pode sincronizar stories com variantes do Figma
- **Turborepo**: o agente com Figma MCP referencia componentes de `packages/ui` em monorepos complexos
- **Design Tokens (W3C)**: Figma exporta tokens no formato W3C Design Tokens — compatível com Style Dictionary e outros pipelines de tokens

### Vale a pena acompanhar?

**Sim, vale acompanhar — e configurar Code Connect.** O Figma Dev Mode MCP com Code Connect é um dos fluxos de design-to-code mais promissores disponíveis. O investimento em configurar o mapeamento de componentes tem retorno imediato. Para times sem Dev Mode do Figma, ainda é muito cedo.

---

## 8. AI + Module Federation 3.0 — Migração Assistida de Microfrontends com Inteligência

### O que é

Module Federation 3.0, combinado com ferramentas de IA, está mudando como equipes migram monólitos SPA para arquiteturas de microfrontend e como mantêm composição em microfrontends existentes. A novidade não é o Module Federation em si — é o uso de IA para resolver os dois problemas mais difíceis da arquitetura: **identificar fronteiras** e **gerenciar dependências compartilhadas**.

**Novidades do Module Federation 3.0:**
- **Native ESM Federation**: federação baseada em ES Modules nativos sem runtime customizado do Webpack
- **Server-Side Module Federation**: agrega RSC (Remote Server Components) de diferentes origens em uma única resposta SSR streamed
- **Deduplicação automática**: resolve versões conflitantes de `react`, `react-dom` e outras shared libs sem configuração manual
- **Protocolo de manifesto padronizado**: `mf-manifest.json` acessível em runtime — LLMs conseguem parsear e entender a topologia da aplicação

**Como IA entra no fluxo:**

1. **Identificação de fronteiras**: ferramentas como `legacyleap.ai` usam IA para analisar monólitos SPA, identificar clusters de componentes com baixo acoplamento entre si e alta coesão interna, e propor fronteiras de microfrontend com justificativas e plano de migração priorizado
2. **Geração de relationship graphs**: visualizações de dependência que tornam explícito quais módulos são candidatos a serem expostos via Module Federation
3. **Detecção de shared dependencies**: IA identifica quais libs precisam de `singleton` no MF config e com qual estratégia
4. **Migração automática**: após definidas as fronteiras, IA gera wrappers, configs e adapters necessários para expor módulos

### Por que isso importa

Migrar de SPA monolítico para microfrontends normalmente leva 6-18 meses com alto risco de regressão. IA reduz o ciclo de análise (que sozinho pode levar semanas em codebases grandes) para horas, com decisões de fronteira auditáveis e baseadas em dados reais de acoplamento. O Server-Side MF com RSC também é significativo: pela primeira vez, microfrontends no lado do servidor permitem composição sem sacrificar SSR.

### Benefícios práticos

- Análise de fronteiras por IA: candidatos a microfrontend em horas, não semanas
- Relationship graphs auditáveis: decisões arquiteturais baseadas em dados de acoplamento real
- Server-Side MF com RSC: SSR em microfrontends sem sacrificar streaming
- Deduplicação automática no MF 3.0: fim dos conflitos de versão de React entre shells e remotes
- Native ESM: sem runtime customizado — os remotes são módulos ES nativos

### Possíveis problemas ou limitações

- **IA propõe fronteiras candidatas, não define arquitetura**: ainda requer julgamento de domínio para validar se as fronteiras propostas fazem sentido de negócio
- **Server-Side MF com RSC ainda é beta**: edge cases com suspense e streaming não resolvidos
- **Overhead de runtime**: Module Federation adiciona um layer de runtime que impacta TTI — em apps pequenas, não é justificável
- **Ferramentas de análise de IA são pagas**: LegacyLeap e similares são comerciais; não há equivalente open-source totalmente funcional

### Exemplo prático

```json
// mf-manifest.json — legível por LLMs para entender a topologia
{
  "id": "checkout-mfe",
  "name": "Checkout Microfrontend",
  "shared": {
    "react": { "singleton": true, "version": "19.1.0" },
    "@company/design-system": { "singleton": true, "version": "4.2.0" }
  },
  "exposes": {
    "./CheckoutFlow": "./src/components/CheckoutFlow",
    "./CartSummary": "./src/components/CartSummary",
    "./PaymentForm": "./src/components/PaymentForm"
  }
}
```

```typescript
// Server-Side MF com RSC — composição no servidor
// shell/app/page.tsx (Server Component)
import { lazy, Suspense } from 'react';

// Módulo carregado de origem remota no servidor
const CheckoutFlow = lazy(() => import('checkout_mfe/CheckoutFlow'));

export default function CheckoutPage() {
  return (
    <Suspense fallback={<CheckoutSkeleton />}>
      <CheckoutFlow /> {/* RSC do checkout MFE streamed junto com o shell */}
    </Suspense>
  );
}
```

```
# Prompt para análise de fronteiras:
"Analise src/components e src/features.
Identifique clusters com alta coesão interna e baixo acoplamento externo.
Para cada cluster candidato a microfrontend:
- Nomeie a fronteira de negócio
- Liste os componentes incluídos
- Identifique dependências compartilhadas necessárias
- Indique o risco de migração (baixo/médio/alto) com justificativa"
```

### Relação com o ecossistema moderno

- **Next.js**: Server-Side MF com RSC é nativamente compatível com o modelo de renderização do Next.js 15+
- **Turborepo**: Turborepo (build coordination) + MF 3.0 (runtime sharing) representa o estado da arte de monorepos com deploy independente
- **Rspack/Webpack 5**: MF 3.0 tem suporte nativo em ambos — Rspack tem melhor performance de build
- **Design Systems**: `@company/design-system` como `shared singleton` no MF config garante versão única em todo o ambiente

### Vale a pena acompanhar?

**Promissor para empresas com times de frontend distribuídos.** Para startups e times pequenos, a complexidade operacional ainda não compensa. Para enterprises com 4+ times independentes, MF 3.0 com análise de IA é a abordagem mais madura disponível. O Server-Side MF com RSC merece atenção especial — se estabilizar, resolve o maior trade-off da arquitetura de microfrontends.

---

## 9. Firebase AI Logic GA — Inferência Gemini Direto no Browser Sem Servidor

### O que é

Firebase AI Logic passou de beta para **Generally Available no Google I/O 2026**. O serviço permite que aplicações web, iOS, Android, Flutter e React Native chamem modelos Gemini **diretamente do código cliente** — sem backend intermediário, sem API keys expostas no bundle, e sem infraestrutura extra para gerenciar.

A segurança é garantida pelo Firebase Authentication e Firebase App Check: o SDK usa o token de autenticação do usuário para autorizar requisições ao Gemini, e o App Check verifica que a chamada vem de uma instância legítima do app.

**Principal novidade do I/O 2026:** **Local Web Inference também vai para GA** — modelos Gemini Nano rodam diretamente no browser usando WebGPU/WebNN, com fallback automático para cloud quando o device não tem capacidade. Zero latência de rede para o caso comum, degradação graciosa para devices limitados.

**Modelos disponíveis:**
- `gemini-3.5-flash` — rápido, custo baixo, ideal para produção
- `gemini-3.5-pro` — máxima qualidade para casos complexos
- `gemini-nano` — on-device (Chrome com WebGPU/WebNN)

**Novidade de maio 2026:** Grounding com Google Maps para reduzir alucinações em apps com localização.

### Por que isso importa

Adicionar features de IA a um frontend normalmente requer: criar endpoint de backend, gerenciar API keys com segurança, tratar rate limiting, implementar retry logic e manter mais infraestrutura. Firebase AI Logic elimina esse overhead para o caso de uso mais comum, usando a infraestrutura de autenticação que provavelmente já existe no projeto. O local inference é genuinamente novo: features de IA que rodam no device têm zero latência de rede, zero custo de API e funcionam offline.

### Benefícios práticos

- Zero infraestrutura de backend: nenhum servidor, nenhum endpoint a manter
- API keys protegidas nativamente via Firebase App Check + Auth
- Local inference GA: Gemini Nano no browser sem latência de rede
- Fallback automático: se WebGPU não está disponível, o SDK vai para cloud
- Grounding com Google Maps: reduz alucinações em apps com localização
- Integração com Firebase Auth, Storage e Firestore — stack coesa

### Possíveis problemas ou limitações

- **Local inference requer Chrome com WebGPU**: cobertura parcial em Firefox e Safari — fallback é cloud nesse caso
- **Gemini Nano tem capacidades limitadas**: não adequado para raciocínio complexo ou geração longa
- **Lock-in no ecossistema Google/Firebase**: migrar para outro provider requer substituir o stack inteiro
- **App Check tem curva de configuração**: especialmente para domínios customizados e staging
- **Custo da API cloud é responsabilidade do desenvolvedor**: Firebase facilita a integração mas não subsidia o custo de chamadas ao Gemini

### Exemplo prático

```typescript
// Web app com Firebase AI Logic — sem servidor necessário
import { initializeApp } from 'firebase/app';
import { getAI, getGenerativeModel, GoogleAIBackend } from 'firebase/ai';

const app = initializeApp(firebaseConfig);
const ai = getAI(app, { backend: new GoogleAIBackend() });

async function generateProductDescription(product: Product): Promise<string> {
  const model = getGenerativeModel(ai, {
    model: 'gemini-3.5-flash',
    generationConfig: { maxOutputTokens: 200, temperature: 0.7 },
  });

  const result = await model.generateContent(`
    Gere uma descrição de produto atraente (máximo 150 palavras) para:
    Nome: ${product.name}
    Categoria: ${product.category}
    Specs: ${product.specs.join(', ')}
  `);
  return result.response.text();
}
```

```typescript
// Local inference com fallback automático para cloud
import { getAI, getGenerativeModel, ChromeAIBackend } from 'firebase/ai';

async function classifyWithFallback(text: string): Promise<string> {
  try {
    const localAI = getAI(app, { backend: new ChromeAIBackend() });
    const model = getGenerativeModel(localAI, { model: 'gemini-nano' });
    return (await model.generateContent(text)).response.text();
  } catch {
    // Fallback transparente para cloud quando device não suporta
    const cloudAI = getAI(app, { backend: new GoogleAIBackend() });
    const model = getGenerativeModel(cloudAI, { model: 'gemini-3.5-flash' });
    return (await model.generateContent(text)).response.text();
  }
}
```

### Relação com o ecossistema moderno

- **React/Next.js**: funciona em Client Components — não compatível com Edge Runtime ou Server Components por design
- **Vue/Nuxt**: SDK JS é framework-agnostic — funciona com composables client-only
- **PWA**: local inference + PWA = features de IA completamente offline para casos simples
- **Vite**: `firebase init ai-logic` gera toda a configuração necessária para projetos Vite

### Vale a pena acompanhar?

**Sim, vale acompanhar — especialmente o local inference.** Para equipes que já usam Firebase, é o caminho mais rápido para adicionar features de IA ao frontend. Para equipes sem Firebase, avaliar se o custo de adoção compensa. O Gemini Nano on-device é genuinamente novo e vale atenção independente do stack — é o primeiro caso real de on-device frontier AI no browser em produção.

---

## 10. AI e Core Web Vitals — Performance Mínima Agora é Requisito para AI Overviews

### O que é

Uma mudança de comportamento confirmada em 2026: **sites com Core Web Vitals ruins raramente aparecem em AI Overviews** — os resumos gerados por IA no topo dos resultados do Google. A correlação foi documentada pela análise CD Studio (dezembro 2025) e confirmada por múltiplos profissionais de SEO ao longo de Q1 2026.

**Thresholds "Good" revisados em 2026:**
- **LCP (Largest Contentful Paint)**: < 2.0s (revisado de 2.5s)
- **INP (Interaction to Next Paint)**: < 200ms
- **CLS (Cumulative Layout Shift)**: < 0.1

A relação de IA com Core Web Vitals tem dois vetores críticos:

**1. IA gerando código que viola CWV:**
Agentes sem contexto de performance frequentemente geram código que prejudica LCP e INP: imagens sem `width`/`height` explícitos (causando CLS), sem `fetchPriority`, event handlers síncronos pesados, fontes sem `font-display: swap`. Modern Web Guidance (item #5) e Chrome DevTools MCP (item #3) são as principais ferramentas para mitigar isso.

**2. IA otimizando CWV:**
O Chrome DevTools MCP expõe métricas de LCP, INP e CLS diretamente para agentes. Lighthouse no DevTools agora interpreta o relatório e gera planos de ação priorizados: o agente identifica o recurso que bloqueia o LCP, propõe mudança de estratégia de carregamento, e verifica o resultado.

### Por que isso importa

Core Web Vitals deixaram de ser um fator de ranking secundário para se tornarem um **requisito de visibilidade na era dos AI Overviews**. Um site com bom conteúdo mas LCP de 3.5s pode simplesmente não aparecer no resumo de IA — perda de tráfego orgânico sem nenhuma mudança de algoritmo de ranking tradicional. Para times de frontend, CWV deixou de ser "métrica de performance" para ser "requisito de produto".

### Benefícios práticos

- Sites com "Good" CWV têm significativamente mais presença nos AI Overviews
- Chrome DevTools MCP + Lighthouse: agentes detectam e corrigem issues de CWV autonomamente
- Modern Web Guidance: previne geração de código que viola CWV (imagens sem dimensões, fontes sem `font-display`, etc.)
- INP substituiu FID: equipes que otimizaram para FID precisam revisar — INP é mais abrangente e mais difícil de otimizar

### Possíveis problemas ou limitações

- **Correlação ≠ causalidade**: a relação entre CWV e AI Overviews é observada empiricamente, não documentada oficialmente pelo Google
- **Threshold LCP 2.0s é mais agressivo**: sites previamente "passing" podem agora estar em "Needs Improvement"
- **INP é difícil em SPAs ricas**: dashboards e editors com alta interatividade naturalmente têm INP mais alto — otimizar sem sacrificar funcionalidade exige profundidade técnica
- **AI Overviews são geograficamente inconsistentes**: o comportamento varia por região e tipo de query

### Exemplo prático

```
# Prompt com Chrome DevTools MCP para otimização de LCP:
"Rode Lighthouse em http://localhost:3000 e:
1. Identifique o elemento LCP atual e o recurso que o bloqueia
2. Se LCP > 2.0s, identifique a causa raiz:
   - É uma imagem? Verifique src, loading, fetchpriority e srcset
   - É uma fonte? Verifique font-display e preloading
   - É um componente de terceiro? Identifique o script bloqueante
3. Gere as mudanças específicas para trazer LCP < 2.0s
4. Rode nova auditoria para confirmar o resultado"
```

```tsx
// Antes: código típico de agente sem contexto de CWV
<img src={hero.url} alt={hero.alt} className="hero-image" />

// Depois: com Modern Web Guidance aplicada
<img
  src={hero.url}
  alt={hero.alt}
  className="hero-image"
  width={hero.width}        // ← previne CLS
  height={hero.height}      // ← previne CLS
  loading="eager"           // ← hero: carrega imediatamente
  fetchPriority="high"      // ← sinaliza prioridade ao browser
  decoding="async"          // ← não bloqueia o main thread
/>
```

```html
<!-- Preload da fonte crítica para prevenir FOIT -->
<link
  rel="preload"
  href="/fonts/inter-var.woff2"
  as="font"
  type="font/woff2"
  crossorigin="anonymous"
/>
<style>
  @font-face {
    font-family: 'Inter';
    src: url('/fonts/inter-var.woff2') format('woff2');
    font-display: swap; /* ← evita flash of invisible text */
    font-weight: 100 900;
  }
</style>
```

### Relação com o ecossistema moderno

- **Next.js Image**: `<Image>` automatiza `width`/`height`, `loading` e `priority` — agentes com AGENTS.md usam o componente nativo
- **Nuxt/Vue**: `<NuxtImg>` faz para Nuxt o que `<Image>` faz para Next.js
- **Design Systems**: componentes de imagem que encapsulam melhores práticas de CWV garantem código otimizado automaticamente
- **Vite**: `vite-imagetools` integra otimização de imagem no fluxo de build

### Vale a pena acompanhar?

**Sim — e agir agora.** O threshold revisado de LCP < 2.0s merece uma auditoria imediata com PageSpeed Insights. Para times que dependem de tráfego orgânico, LCP < 2.0s, INP < 200ms e CLS < 0.1 deixaram de ser targets aspiracionais para serem requisitos de visibilidade em busca.

---

*Relatório gerado pela skill `frontend-ia-weekly` em 27/05/2026.*
*Fontes: Next.js Blog, Svelte Blog, Chrome Developers Blog, W3C WebMCP, TanStack GitHub, Google I/O 2026, Figma Blog, Firebase Blog, DEV Community, Vercel Blog.*
