# Novidades Front-end — Semana de 14/07 a 23/07/2026

> **Janela de pesquisa:** 08/07 a 23/07/2026. Todos os itens abaixo foram publicados/lançados dentro dessa janela e **não** repetem tópicos já cobertos no relatório de 13/07 (Cloudflare AI traffic, Chrome DevTools 150, shadcn Base UI default, shadcn/typeset, Figma Make GPT-5.6, shadcn-vue Rhea, Nuxt Content llms.txt, PrimeUI, WebMCP, LiteRT.js, etc.).
>
> **Nota de transparência:** priorizei itens com data verificável na janela em vez de completar cota com conteúdo antigo. A maior parte das novidades reais de front-end desta semana concentrou-se no ecossistema Vue/Nuxt e em bibliotecas de UI para IA — por isso o arquivo traz 8 itens densos em vez de 20 rasos.

---

## 1. Vue 3.6.0-rc.1 chega com Vapor Mode completo e reescrita da reatividade sobre alien-signals

### Referências
- [VUE 3.6 RC LANDS WITH VAPOR MODE COMPLETE AND REACTIVITY REWRITE (RepoJournal)](https://repojournal.com/showcase/vuejs/2026-07-18) — 18/07/2026
- [Vue 3.6 Pre-Release Notes: Major Changes You Should Know in 2026 (Stackademic)](https://blog.stackademic.com/vue-3-6-pre-release-notes-major-changes-you-should-know-in-2026-a584adcf1be7) — referência complementar

### O que é
O Vue lançou em 18/07/2026 o **3.6.0-rc.1**, primeiro release candidate que marca o **Vapor Mode como completo e pronto para produção**. Vapor é uma estratégia de compilação alternativa que elimina totalmente o Virtual DOM para componentes que optam por ele (`<script setup vapor>`), gerando código que atualiza o DOM diretamente. O RC também traz a **reescrita completa de `@vue/reactivity`** sobre o algoritmo **alien-signals**, com ganhos medidos de performance e de memória (queda de ~22% de pico de memória em alguns apps por não alocar objetos VNode).

### Por que isso importa
Vue passa a jogar no mesmo campo de rendering de Solid.js — sem exigir troca de framework. Para times que já têm base Vue/Nuxt, o Vapor destrava cenários de UI de altíssima frequência (dashboards em tempo real, editores, canvas, streaming de tokens de IA) sem reescrever a aplicação. E a reatividade sobre signals aproxima o modelo mental do Vue do restante do ecossistema (Solid, Angular Signals, Svelte runes), o que reduz atrito de contratação e de leitura de código.

### Benefícios práticos
- Rendering direto de DOM, sem diffing de VNode, para componentes opt-in.
- Menor pressão de memória e menos garbage collection — importante em telas de chat que renderizam streaming contínuo.
- Reatividade mais previsível e granular (só o nó ligado à propriedade re-renderiza).
- Migração incremental: componentes Vapor convivem com componentes clássicos no mesmo app.

### Possíveis problemas ou limitações
É um **RC**, não estável — APIs e edge cases ainda podem mudar. Vapor é opt-in e nem toda biblioteca do ecossistema (diretivas, plugins que dependem de VNode) funciona em modo Vapor ainda, então há risco de fragmentação temporária ("meu componente é Vapor, mas essa lib não suporta"). A reescrita da reatividade, por mais bem testada, é o tipo de mudança que só revela regressões sutis sob carga real.

### Exemplo prático
```vue
<script setup vapor>
import { ref } from 'vue'
// Reatividade sobre alien-signals; sem VNode alocado para este componente
const tokens = ref<string[]>([])
function onChunk(chunk: string) {
  tokens.value.push(chunk) // update direto no DOM, ideal para streaming de LLM
}
</script>

<template>
  <p>{{ tokens.join('') }}</p>
</template>
```

### Relação com o ecossistema moderno
Base para o **Nuxt 5** (item #2 usa Nuxt 4.5 como ponte). Alinha Vue à onda de signals de Angular/Solid/Svelte. Combina com **LiteRT.js** (inferência local no browser, coberta em 13/07) e com UIs de streaming de IA, onde o custo de VNode pesa. Pinia 3 já está sendo adaptado para operar em harmonia com os signals do 3.6.

### Vale a pena acompanhar?
**Sim, vale acompanhar de perto** — é a mudança estrutural mais importante do Vue desde o 3.0. Ainda **não** para produção crítica (é RC), mas times Vue devem começar a testar Vapor em telas de alta frequência agora.

---

## 2. Nuxt 4.5 muda a build layer para Vite 8 + Rspack 2 e ativa SSR streaming experimental

### Referências
- [Nuxt 4.5 (Nuxt Blog)](https://nuxt.com/blog/v4-5) — 18/07/2026
- [Nuxt 4.5 moves its build layer to Vite 8 and Rsbuild while staging the jump to Nuxt 5 (Stackmaven)](https://stackmaven.io/news/nuxt-4-5-release/) — 18/07/2026

### O que é
Nuxt **4.5** (18/07/2026), descrito pela própria equipe como "o maior release em bastante tempo", reconstrói a camada de build: adota **Vite 8** (Rolldown como bundler Rust default), adiciona o builder **Rspack 2 via Rsbuild**, introduz **SSR streaming experimental**, um **sistema estável de códigos de erro**, o composable `useLayout`, named views e bastante fundação para o **Nuxt 5**. Sai junto o patch de manutenção `v3.21.9` para a linha 3.x — que chega ao **fim de vida em 31/07/2026**.

### Por que isso importa
SSR streaming é o item de maior impacto para IA: permite enviar o shell da página e ir preenchendo blocos conforme o servidor (ou um LLM) produz conteúdo, reduzindo TTFB percebido em páginas geradas dinamicamente. E a troca para Vite 8/Rolldown corta drasticamente tempo de build — relevante para monorepos e para o loop de agentes de código que rebuildam com frequência. O EOL do Nuxt 3 força um cronograma de migração para milhares de projetos.

### Benefícios práticos
- Builds muito mais rápidos (Rolldown/Oxc no lugar de esbuild+Babel).
- SSR streaming para renderização progressiva de conteúdo pesado/gerado por IA.
- Escolha de builder (Vite ou Rspack) conforme o perfil do projeto.
- `server.forwardConsole` do Vite 8 encaminha logs/erros do cliente para o terminal — "especialmente útil ao trabalhar com coding agents".

### Possíveis problemas ou limitações
SSR streaming é **experimental** — hidratação parcial e ordem de flush ainda têm arestas. Migrar a build layer inteira (Vite 8 + Rspack 2) num único minor é agressivo: plugins de build customizados podem quebrar. E o EOL do Nuxt 3 em 31/07 pressiona times que não têm janela para atualizar.

### Exemplo prático
```ts
// nuxt.config.ts
export default defineNuxtConfig({
  experimental: {
    ssrStreaming: true, // envia o shell e faz flush por blocos
  },
  builder: 'rspack', // ou 'vite' (Vite 8 + Rolldown)
})
```

### Relação com o ecossistema moderno
Ponte direta para **Nuxt 5**; casa com **Vue 3.6 Vapor** (item #1), com **Edge/Nitro** (streaming no edge) e com **Turborepo/monorepos** (builds rápidos). SSR streaming conversa com Server Components/streaming do lado React e com AI Gateways que produzem conteúdo incremental.

### Vale a pena acompanhar?
**Sim** — é atualização obrigatória de planejamento para qualquer time Nuxt, sobretudo pelo EOL do 3.x. SSR streaming: **promissor, mas ainda experimental**.

---

## 3. Nuxt UI v4.10.0 adiciona fluxo de aprovação de ferramentas (tool-approval) aos componentes de chat de IA

### Referências
- [Nuxt UI Releases — v4.10.0](https://ui.nuxt.com/releases) — 16/07/2026

### O que é
O Nuxt UI **v4.10.0** (16/07/2026) evolui a suíte de componentes de chat de IA: o `ChatPrompt` ganha **slot de `body`** e realce de foco; o `ChatTool` ganha uma **prop `actions`** para **workflows de aprovação de ferramenta** (o usuário aprova/rejeita uma tool call antes de o agente executar); melhora indicadores de streaming e suporte a `plain text` no Editor. Fora do escopo de IA, embute os ícones no build (renderização imediata em SSR, funciona offline, sem chamada runtime à API do Iconify) e adiciona `unmountOnHide` para preservar estado de modais/slideovers.

### Por que isso importa
Aprovação de ferramenta ("human-in-the-loop") deixa de ser algo que cada time reimplementa e vira **componente de primeira classe** do design system Vue. Isso é exatamente a superfície de UI que faltava para construir agentes seguros no front-end: mostrar ao usuário *o que* o agente quer fazer e exigir consentimento antes de efetivar uma ação com efeitos colaterais.

### Benefícios práticos
- UI pronta de tool-approval, alinhada ao padrão human-in-the-loop do AI SDK.
- Ícones offline/SSR-ready — melhora LCP e elimina dependência de rede em runtime.
- `unmountOnHide` preserva formulário/scroll em fluxos de chat com modais.
- Indicadores de streaming mais consistentes (respeitando `prefers-reduced-motion`).

### Possíveis problemas ou limitações
Ainda é acoplado ao Nuxt UI (lock-in de design system). A prop `actions` resolve a UI, mas a **segurança real** depende de o backend nunca executar a tool sem o token de aprovação — a camada visual pode dar falsa sensação de proteção se o servidor não validar. Componentes de chat de IA em bibliotecas de UI amadurecem rápido e podem ter breaking changes frequentes.

### Exemplo prático
```vue
<UChatTool
  :tool="toolCall"
  :actions="[
    { label: 'Aprovar', color: 'primary', onClick: () => approve(toolCall.id) },
    { label: 'Rejeitar', color: 'error', onClick: () => reject(toolCall.id) },
  ]"
/>
```

### Relação com o ecossistema moderno
Casa com **AI SDK** (human-in-the-loop / `ToolLoopAgent`), com **Nuxt/Nitro** (Server Routes que validam a aprovação) e com o **MCP** (tool approval como contraparte de UI dos tool contracts). É o análogo Vue do que `@shadcn/helpers`/AI Elements fazem no lado React.

### Vale a pena acompanhar?
**Sim** para quem constrói produtos de chat/agente em Vue. Tool-approval nativo é um sinal claro de que design systems estão absorvendo padrões de UX de agentes.

---

## 4. shadcn/ui promove React Aria a base de componentes de primeira classe

### Referências
- [shadcn/ui — Changelog (React Aria)](https://ui.shadcn.com/docs/changelog) — julho de 2026 (entrada mais recente do changelog, posterior a 13/07)

### O que é
Depois de tornar **Base UI** o default (coberto em 13/07), o shadcn/ui adicionou o **React Aria** como **terceira base headless de primeira classe** — ao lado de Base UI e Radix. A entrada (a mais recente do changelog em julho/2026) inclui documentação completa, suporte nos oito temas de estilo e **output de registry com escopo** específico para componentes Aria. Na prática: `npx shadcn init` passa a permitir escolher React Aria como fundação de acessibilidade dos componentes.

### Por que isso importa
React Aria (Adobe) é referência em acessibilidade e interações complexas (foco, teclado, ARIA, i18n de data/número). Tê-lo como base oficial dá aos times a opção de priorizar **acessibilidade robusta** — que, além de inclusão, é exatamente o que **agentes de navegação de IA** leem (a árvore de acessibilidade é o "DOM" que os agentes entendem). Ou seja, escolher React Aria melhora simultaneamente humanos com deficiência e a legibilidade do app para agentes (Agentic Browsing / AEO).

### Benefícios práticos
- Base de acessibilidade de nível Adobe sem sair do fluxo shadcn (copy-paste + registry).
- Melhor semântica ARIA → melhor score de Agentic Browsing no Lighthouse/PageSpeed.
- Três fundações para escolher conforme o projeto (Base UI, Radix, React Aria).
- Registry com escopo evita puxar dependências desnecessárias.

### Possíveis problemas ou limitações
Três bases oficiais aumentam a **superfície de manutenção** e a chance de fragmentação (componentes de terceiros feitos para Radix podem não casar 1:1 com Aria). React Aria tem curva de aprendizado maior e bundle próprio. Misturar bases no mesmo projeto pode gerar inconsistências de comportamento de foco.

### Exemplo prático
```bash
npx shadcn@latest init   # escolha "React Aria" como base
npx shadcn@latest add combobox   # componente com a11y de teclado/foco do React Aria
```

### Relação com o ecossistema moderno
Conecta **design systems** a **AEO/Agentic Browsing** (item de SEO para agentes), a **React/Next**, e ao ecossistema de registries do shadcn. É a materialização de "acessibilidade = pré-requisito para IA ler seu site".

### Vale a pena acompanhar?
**Sim**, sobretudo para times que levam acessibilidade e prontidão para agentes a sério. Para quem já padronizou em Base UI/Radix, é opcional.

---

## 5. shadcn lança @shadcn/helpers: testar fluxos de chat de IA sem modelo, API ou rede

### Referências
- [shadcn/ui — Changelog (Introducing @shadcn/helpers)](https://ui.shadcn.com/docs/changelog) — julho de 2026 (entrada do changelog posterior a 13/07)

### O que é
O `@shadcn/helpers` é um pacote open-source novo com **helpers para desenvolvimento de IA**. O destaque são **adapters para AI SDK e TanStack AI** que permitem **testar fluxos de conversa de forma determinística — sem modelo, sem API route, sem requisição de rede e sem API key**. Você escreve interações de chat previsíveis para construir e documentar componentes.

### Por que isso importa
Testar UI de chat sempre foi frágil: depende de LLM não-determinístico, rede e custo. Ter um adapter que **simula o stream** de mensagens/tool calls de maneira determinística torna possível ter **testes unitários e stories estáveis** para componentes de IA — algo que faltava para levar chat/agent UIs a padrões sérios de CI.

### Benefícios práticos
- Testes de UI de chat determinísticos e offline (sem flakiness de LLM).
- Stories/documentação de componentes de IA reproduzíveis.
- Zero custo de API em CI.
- Cobre tanto o **AI SDK** (Vercel) quanto o **TanStack AI**.

### Possíveis problemas ou limitações
Simular o stream **não** cobre o comportamento real do modelo (alucinação, latência, ordem de tokens sob carga) — é bom para a **camada de UI**, não para avaliar o agente. Como todo helper acoplado a AI SDK/TanStack AI, segue as breaking changes desses SDKs. Ainda é novo; a superfície de API pode mudar.

### Exemplo prático
```ts
import { mockChat } from '@shadcn/helpers/ai-sdk'

const chat = mockChat({
  script: [
    { role: 'assistant', chunks: ['Olá', ', ', 'como posso ajudar?'] },
    { role: 'tool', name: 'searchDocs', result: { hits: 3 } },
  ],
})
// renderiza o componente com um stream determinístico — sem rede
```

### Relação com o ecossistema moderno
Complementa **AI SDK** e **TanStack AI**, integra com **Vitest/Playwright** em CI, e casa com o padrão de **AI Elements**/registries do shadcn. É a peça de **testes** do lado React que espelha o tool-approval do Nuxt UI (item #3).

### Vale a pena acompanhar?
**Sim** para quem constrói e testa UIs de chat/agente em React. Resolve uma dor concreta (flakiness) de forma pragmática.

---

## 6. TanStack Intent: publicar e consumir "Agent Skills" como artefatos de pacote

### Referências
- [Agent Skills (TanStack Intent) — TanStack AI Docs](https://tanstack.com/ai/latest/docs/getting-started/agent-skills) — atualizado 16/07/2026
- [Using TanStack Intent to ship and consume agent skills (Vercel Knowledge Base)](https://vercel.com/kb/guide/using-tanstack-intent-to-ship-and-consume-agent-skills) — referência complementar

### O que é
**TanStack Intent** é uma CLI para **empacotar e distribuir Agent Skills** — documentos Markdown que ensinam agentes de código (Claude Code, Cursor, Copilot etc.) a usar sua biblioteca **corretamente** — como artefatos versionados de pacote. Em vez de o agente "adivinhar" a API da sua lib a partir de tipos, ele recebe um skill oficial, mantido pelo autor da lib, dentro do `node_modules`/registry.

### Por que isso importa
É a resposta do ecossistema TanStack ao problema de **manutenibilidade com IA**: agentes erram menos quando têm instruções curadas da própria biblioteca. Transformar isso em artefato distribuível (com versionamento) alinha "documentação para humanos" e "documentação para agentes" no mesmo pipeline de release — reduzindo o clássico "o agente usou uma API deprecada".

### Benefícios práticos
- Menos alucinação de API: o agente lê o skill oficial da lib.
- Versionamento: o skill acompanha a versão do pacote.
- Interoperável entre agentes (Claude Code, Cursor, Copilot…).
- Autor da lib controla a narrativa de uso correto.

### Possíveis problemas ou limitações
Depende de adoção: só ajuda se as bibliotecas realmente publicarem skills e os agentes os consumirem. Ainda não há padrão único de formato de skill entre ferramentas (risco de fragmentação com o skill format de outros fornecedores). E skills desatualizados podem *piorar* o resultado se não forem mantidos junto com o código.

### Exemplo prático
```bash
# Autor da lib publica o skill junto com o pacote
npx tanstack-intent publish ./skills/use-query.md

# Consumidor instala e o agente passa a "conhecer" a API correta
npx tanstack-intent add @tanstack/query
```

### Relação com o ecossistema moderno
Conecta **bibliotecas** a **agentes de código**, a **monorepos** (skills por pacote) e ao movimento de "skills" que aparece também em WebStorm 2026.2 e no ecossistema Claude/Cursor. É complementar a **MCP** (MCP dá ferramentas; skills dão conhecimento de uso).

### Vale a pena acompanhar?
**Promissor**, especialmente para autores de bibliotecas. Ainda cedo para virar padrão, mas o problema que ataca (agentes usando APIs erradas) é real e caro.

---

## 7. assistant-ui + Mastra: UI de chat de IA em React sobre shadcn ganha tração (v0.14.27)

### Referências
- [@assistant-ui/react — npm (v0.14.27)](https://www.npmjs.com/package/@assistant-ui/react) — 17/07/2026 (publicado ~6 dias antes de 23/07)
- [Using Assistant UI — Mastra Docs](https://mastra.ai/guides/build-your-ui/assistant-ui) — referência complementar

### O que é
O **assistant-ui** é uma biblioteca open-source TypeScript/React para construir experiências de chat de IA de nível produção, **construída sobre shadcn/ui e Tailwind**. A versão **0.14.27** foi publicada em ~17/07/2026. Um destaque recente é a integração formalizada com o **Mastra** (`@mastra/core` para runtime de agente; `@mastra/ai-sdk` para converter o stream do Mastra no formato de UI message stream do AI SDK), permitindo full-stack no Next.js ou backend separado escalável.

### Por que isso importa
Consolida-se um **stack de referência** para chat/agente em React: assistant-ui (UI) + AI SDK (protocolo de stream) + Mastra (runtime de agente). Isso reduz o "monte seu próprio chat" que ainda dominava 2025 e dá aos times componentes de mensagem, streaming, tool calls e attachments prontos — sobre a base familiar do shadcn.

### Benefícios práticos
- Componentes de chat prontos (streaming, tool UI, attachments) sobre shadcn/Tailwind.
- Dois modos de integração com Mastra: embutido no Next.js ou backend standalone.
- Interop com AI SDK (message stream) — não prende a um provider.
- Base shadcn = fácil de customizar com o design system existente.

### Possíveis problemas ou limitações
Versão `0.x` — API ainda instável, breaking changes esperados. O stack (assistant-ui + Mastra + AI SDK) adiciona várias dependências que precisam evoluir em sincronia. Para casos simples, pode ser mais peso do que um componente próprio.

### Exemplo prático
```tsx
import { AssistantRuntimeProvider } from '@assistant-ui/react'
import { useChatRuntime } from '@assistant-ui/react-ai-sdk'

export function Chat() {
  const runtime = useChatRuntime({ api: '/api/chat' }) // rota Mastra + AI SDK
  return (
    <AssistantRuntimeProvider runtime={runtime}>
      <Thread />
    </AssistantRuntimeProvider>
  )
}
```

### Relação com o ecossistema moderno
**React/Next**, **shadcn/ui**, **Tailwind**, **AI SDK** e **Mastra**. É a contraparte React madura do que Nuxt UI (item #3) oferece em Vue, e consome o mesmo protocolo de stream do `@shadcn/helpers` (item #5).

### Vale a pena acompanhar?
**Sim** para quem constrói chat/agente em React e quer sair do zero. Trate como `0.x`: ótimo para produtos, mas pin de versão e testes (item #5) são obrigatórios.

---

## 8. CopilotKit e o protocolo AG-UI empurram a "Generative UI" — agentes que enviam a própria interface

### Referências
- [Generative UI Spectrum: How Agents Now Ship Their Own Interfaces (CopilotKit Blog)](https://www.copilotkit.ai/blog/generative-ui-explained-how-agents-now-ship-their-own-interfaces) — referência do tema
- [Auto-Tune: Stop Writing the Prompt, Train the Model (CopilotKit Blog, David McKay)](https://www.copilotkit.ai/blog) — 20/07/2026
- [AG-UI Protocol (CopilotKit)](https://www.copilotkit.ai/ag-ui) — referência do protocolo

### O que é
O **CopilotKit** — mantenedor do protocolo **AG-UI (Agent–User Interaction)** — publicou nesta janela conteúdo consolidando a ideia de **Generative UI**: agentes que, além de texto, **emitem componentes de interface** de volta para o app (formulários, gráficos, cartões acionáveis) via uma conexão bidirecional padronizada entre frontend e backend agêntico. Em 20/07/2026 saiu o artigo "Auto-Tune" (treinar modelos especialistas a partir do tráfego de produção em vez de só ajustar prompt) e houve evento ao vivo (23/07) sobre construir apps agênticos e Generative UI com Angular. AG-UI já é citado como adotado por Google, Microsoft, Amazon e Oracle, e por frameworks como LangChain, Mastra, PydanticAI e Agno.

### Por que isso importa
Muda o contrato front-end↔agente: em vez de o front-end só renderizar strings, ele passa a **receber e montar UI dirigida pelo agente**, com eventos bidirecionais (o agente pede input, o usuário responde, o agente continua). É a base arquitetural para copilotos "dentro do app" que fazem parte do produto — não um chat colado no canto.

### Benefícios práticos
- Padrão aberto (AG-UI) para conectar qualquer backend agêntico a qualquer frontend.
- Generative UI: agente devolve componentes ricos, não só texto.
- Suporte multi-framework (React, Angular, mobile) e multi-agente (LangChain, Mastra…).
- Interações bidirecionais (human-in-the-loop) padronizadas.

### Possíveis problemas ou limitações
Generative UI dirigida por agente abre **superfície de segurança/UX** séria: renderizar componentes decididos por um LLM exige allow-list rígida e validação (risco de UI injection). Ainda é um espaço em definição — múltiplos protocolos concorrentes (AG-UI vs. abordagens proprietárias) podem fragmentar. E "agente escolhe a UI" pode piorar consistência de design se não houver design system forte por trás.

### Exemplo prático
Fluxo típico AG-UI:
```
Usuário: "compare os planos e me deixe escolher"
Agente  -> emite componente <PlanComparison plans=[...] onSelect=... />
Frontend -> renderiza (a partir de allow-list) e devolve o evento onSelect
Agente  -> continua o fluxo com a escolha
```

### Relação com o ecossistema moderno
Conecta **React/Angular**, **microfrontends** (agente como origem de UI), **design systems** (componentes allow-listed), **Mastra/LangChain/PydanticAI** e o padrão **MCP** (ferramentas) — AG-UI é a camada de *interação* que falta ao MCP, que é a camada de *ferramentas*.

### Vale a pena acompanhar?
**Sim, promissor** — Generative UI é uma das direções mais concretas de "IA no front-end". Ainda **cedo** para padronizar em produção crítica; ótimo para prototipar copilotos in-app com governança de componentes.
