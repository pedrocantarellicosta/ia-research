# Front-End AI Weekly — Novidades Infra / Geral | 08 de Junho de 2026

> **Data de geração:** 08 de Junho de 2026
> **Período coberto:** 01–08 de Junho de 2026
> **Arquivo:** 5 Novidades Infra / Geral
> **Curadoria:** Skill `frontend-ia-weekly` via Claude Sonnet 4.6

---

## 1. Windsurf → Devin Desktop — Rebrand Completo e Agent Command Center

### O que é

Em 2 de junho de 2026, a Cognition retirou definitivamente o nome Windsurf e relançou o IDE como **Devin Desktop**. O rebrand não é cosmético — ele sinaliza uma mudança na filosofia do produto: de "IDE com assistente de IA" para "plataforma de agente" com o IDE como superfície secundária.

**Agent Command Center como surface padrão:** quando o Devin Desktop abre, a tela inicial não é mais o editor de código — é o Agent Command Center, uma interface para orquestrar agentes, visualizar tarefas em andamento, revisar outputs e aprovar ações. O editor de código fica acessível mas não é mais a metáfora central.

**Open Agent Client Protocol (ACP):** a Cognition abriu o protocolo de comunicação entre o Devin Desktop e seus agentes. O ACP define como clientes (IDEs, CLIs, dashboards externos) se comunicam com o runtime de agentes. Isso significa que times podem construir interfaces customizadas para o Devin sem depender do IDE oficial — ou integrar o protocolo com outros sistemas.

**Zero Data Retention por padrão:** todo código processado pelo Devin Desktop é descartado após a sessão — nenhum dado de código é usado para treinamento sem opt-in explícito. Feature originalmente do Windsurf que foi preservada e mantida como padrão no rebrand.

**Suporte a self-hosted deployment:** o Devin Desktop pode ser apontado para um runtime de agentes hospedado internamente — relevante para times com restrições de compliance que não podem enviar código para APIs externas.

### Por que isso importa

O rebrand de Windsurf para Devin Desktop é o primeiro IDE mainstream a fazer a inversão explícita: o agente é o protagonista, o editor é a ferramenta. Isso muda o mental model da ferramenta — em vez de "IDE que tem IA", é "plataforma de agente que tem um editor quando você precisa editar".

A abertura do ACP é estrategicamente relevante: cria um ecossistema em torno do Devin que não depende do app oficial, similar ao que o protocolo MCP fez pela camada de ferramentas de agentes.

### Benefícios práticos

- **Agent Command Center**: visibilidade central de todas as tarefas de agentes em andamento
- **ACP aberto**: integração com sistemas internos de ticketing, CI/CD e dashboards customizados
- **Zero Data Retention**: dados de código processados localmente, descartados pós-sessão
- **Self-hosted**: viabiliza uso em ambientes de compliance estrito (fintech, healthcare, enterprise)
- **Migração de Windsurf**: todas as configurações e plugins migram automaticamente

### Possíveis problemas ou limitações

- Repositório de plugins menor comparado ao VS Code — extensões específicas de linguagem podem faltar
- Agent Command Center como surface padrão tem curva de aprendizado — devs acostumados com IDE tradicional precisam adaptar mental model
- ACP ainda em early release — especificação pode mudar antes de estabilizar
- Plano enterprise ainda requer contrato separado com a Cognition para features avançadas de agentes
- Market share menor que Cursor e VS Code — comunidade de suporte e exemplos mais escassa

### Exemplo prático

```yaml
# ACP — configuração de agente customizado via protocolo aberto
# devin-agent.yaml

agent:
  name: "frontend-review-agent"
  protocol: acp/v1
  runtime:
    endpoint: "https://agents.company-internal.com/devin"
    auth: bearer
  tasks:
    - name: "Review PR"
      trigger: github.pull_request.opened
      actions:
        - analyze_diff
        - check_design_tokens
        - run_accessibility_audit
      output: github.pr_comment

# O ACP permite que esse agente seja disparado pelo Devin Desktop,
# por uma GitHub Action ou por um dashboard interno — mesmo runtime
```

```bash
# Migração do Windsurf para Devin Desktop
# As configurações são preservadas automaticamente
# MCP servers configurados continuam funcionando
# Plugins Windsurf compatíveis migram via Compatibility Layer
```

### Relação com o ecossistema moderno

- **MCP**: ACP e MCP são complementares — ACP é para comunicação de orquestração de agentes, MCP é para ferramentas que agentes usam
- **GitHub Actions**: ACP permite disparar o Devin Desktop como executor de CI diretamente
- **Claude Code**: ACP é compatível com outros runtimes de agente — Devin Desktop pode orquestrar sessões Claude Code via protocolo
- **VS Code Extensions**: Layer de compatibilidade suporta subset de extensões VS Code
- **Monorepos**: Agent Command Center visualiza tarefas paralelas em diferentes packages do monorepo

### Vale a pena acompanhar?

**Sim, vale acompanhar.** A abertura do ACP e a inversão agent-first posicionam o Devin Desktop como aposta de longo prazo. Para times que já usavam o Windsurf, a migração é automática. Para novos usuários, a curva de adaptação é real mas o posicionamento estratégico é único.

---

## 2. Google Antigravity 2.0 — Desktop App, CLI, Browser Subagent e Plano AI Ultra

### O que é

Lançado no Google I/O em 19 de maio e disponível amplamente a partir da semana de 1 de junho, o **Google Antigravity 2.0** é a maior atualização da plataforma desde o lançamento original. Enquanto a v1.0 era essencialmente um IDE com integração ao Gemini, a v2.0 é uma plataforma de agentes completa com 5 componentes:

1. **Desktop App (novo)**: aplicação standalone com suporte a orquestração de múltiplos agentes simultâneos, workflows de subagentes customizáveis e tasks em background. Integração nativa com Google AI Studio, Android Studio e Firebase.

2. **CLI (novo)**: `antigravity` CLI para rodar agentes diretamente do terminal — similar ao Claude Code, mas com acesso ao ecossistema Google. Ideal para CI/CD e automação de tarefas.

3. **SDK (novo)**: SDK de agentes para construir pipelines customizados com Gemini e ferramentas externas. Compatível com MCP para extensibilidade.

4. **Managed Agents via Gemini API**: tier de agentes gerenciados — o Google executa e escala os agentes, sem necessidade de infraestrutura própria.

5. **Enterprise path**: SLAs, compliance, suporte a VPC para dados sensíveis.

**Browser Subagent — o diferencial técnico:**
O Browser Subagent sobe uma instância real do Chrome (não headless simulado) e opera a aplicação enquanto constrói. Ele clica em botões, preenche formulários, tira screenshots e reporta de volta. Se um botão não funciona, o agente vê, diagnostica e corrige — sem intervenção humana. Isso fecha o loop de desenvolvimento visual de forma mais integrada do que qualquer ferramenta anterior.

**Voice Commands nativos**: suporte a comandos de voz para controlar o agente — "reverta essa mudança", "rode os testes", "abra o componente ProductCard".

**AI Ultra Plan ($100/mo)**: 5x os limites de IA do plano Pro, acesso antecipado a features de preview e suporte prioritário.

### Por que isso importa

O Antigravity 2.0 é a aposta do Google para competir com Claude Code e Cursor no espaço de agentes de desenvolvimento. A diferença estratégica é o Browser Subagent — nenhum outro IDE agentic tem uma instância real de Chrome como parte do loop de desenvolvimento. Para frontend especificamente, isso é relevante: o agente não apenas gera código, ele verifica o resultado visual automaticamente.

A abertura do SDK e a compatibilidade com MCP significa que o ecossistema de ferramentas construído para outros agentes funciona no Antigravity — sem retrabalho de configuração.

### Benefícios práticos

- **Browser Subagent**: verificação visual automática sem abrir o browser manualmente
- **CLI**: integração em CI/CD sem UI — rodar o Antigravity como executor de tarefas
- **SDK**: construir pipelines customizados com Gemini e ferramentas MCP existentes
- **Firebase integration**: scaffold de projetos Firebase, deploy e testes de security rules via agente
- **Voice commands**: hands-free para tasks de inspeção e navegação
- **Compatibilidade MCP**: todos os servidores MCP configurados migram nativamente

### Possíveis problemas ou limitações

- Forte lock-in no ecossistema Google — Firebase, Android, GCP nativo; outras clouds como afterthought
- Browser Subagent tem custo de compute adicional — cada sessão de verificação visual consome mais créditos
- CLI ainda em early access — comportamentos de edge case precisam de validação
- Plano AI Ultra ($100/mo) é caro para uso individual sem retorno claro de ROI mapeado
- Sem suporte a self-hosted por enquanto — dados processados na infraestrutura do Google

### Exemplo prático

```bash
# Instalação do CLI
npm install -g @google/antigravity

# Rodar agente com Browser Subagent ativo
antigravity run \
  --task "Implemente o checkout em 3 etapas e verifique que o fluxo funciona no browser" \
  --browser-subagent \
  --model gemini-3.5-flash \
  --project ./my-nextjs-app

# O agente:
# 1. Implementa os componentes
# 2. Sobe o dev server
# 3. Abre Chrome real
# 4. Navega pelo fluxo de checkout
# 5. Tira screenshots de cada etapa
# 6. Reporta: "Etapa 2 falhou — botão 'Continuar' não fica enabled após preencher CEP"
# 7. Diagnostica o bug
# 8. Corrige automaticamente
# 9. Verifica novamente
```

```ts
// SDK — pipeline customizado
import { Antigravity } from '@google/antigravity-sdk'

const agent = new Antigravity({
  model: 'gemini-3.5-flash',
  tools: ['browser', 'filesystem', 'terminal'],
  mcpServers: ['@svelte/mcp', 'chrome-devtools-mcp'],
})

const result = await agent.run(
  'Verifique se o design do ProductCard está conforme o Figma. Figma node: abc123'
)
```

### Relação com o ecossistema moderno

- **Next.js/React**: Browser Subagent verifica Server Components renderizados, não só código estático
- **Firebase**: integração nativa para scaffold, deploy, emulators via CLI
- **MCP**: SDK compatível com todos servidores MCP do ecossistema (Figma, chrome-devtools, Svelte MCP)
- **Android Studio**: agente pode tocar em projetos Android/React Native simultaneamente
- **GitHub Actions**: CLI pode ser usado como executor de agente em workflows CI/CD

### Vale a pena acompanhar?

**Sim, vale acompanhar.** O Browser Subagent é uma contribuição técnica genuína — é o único IDE agentic com verificação visual em Chrome real como parte do loop. Para times no ecossistema Google (Firebase, GCP, Android), o valor é ainda maior. Aguardar o SDK estabilizar antes de usar em produção.

---

## 3. Cursor — Repricing Teams: Standard vs Premium Seats

### O que é

Na primeira semana de junho, o Cursor reestruturou os planos de Teams para resolver o problema de previsibilidade de custos — o ponto de maior atrito em adoção corporativa. Os novos planos dividem os usuários em dois perfis de uso com preços e limites explícitos:

**Standard Seat:**
- $32/seat/mês (anual) / $40/seat/mês (mensal)
- Equivalente ao plano anterior de uso normal
- Inclui acesso completo ao Composer Mode, Subagents, indexação de codebase

**Premium Seat:**
- $96/seat/mês (anual) / $120/seat/mês (mensal)
- 5x o limite de uso do Standard
- Para power users e leads que usam o Cursor extensivamente para tasks de arquitetura, refatoração e code review
- Sem rate limiting em horários de pico

**Mudança estrutural:** anteriormente, o plano Teams tinha um pool de créditos compartilhado que tornava o custo mensal imprevisível. Com a nova estrutura, o custo é fixo por seat — finance pode prever o gasto trimestral com exatidão.

**Por que o repricing aconteceu:** o Cursor reportou que ~15% dos usuários Teams consumiam ~60% dos créditos do pool compartilhado, criando situações onde outros membros do time ficavam sem créditos no final do mês. A divisão Standard/Premium resolve isso isolando os padrões de uso.

### Por que isso importa

O modelo de crédito compartilhado era o principal obstáculo para times enterprise adotarem o Cursor em escala. Um time de 50 pessoas com 5 power users esgotava o pool antes do fim do mês — o que levava ao pattern de "esconder o uso do Cursor para não queimar créditos dos colegas". O repricing elimina essa fricção.

### Benefícios práticos

- **Custo previsível**: finance aprova budget trimestral de IA com segurança
- **Sem tragédia dos comuns**: power users têm seus próprios limites, não afetam o time
- **Composição de planos**: times podem ter 40 Standard + 10 Premium — custo otimizado por perfil
- **Subagents sem throttle em Premium**: tasks longas de agente não são interrompidas por rate limit

### Possíveis problemas ou limitações

- Premium $96/mo não tem tabela de ROI clara publicada — é difícil justificar para gestores sem dados de uso
- Times que usavam o pool compartilhado e tinham uso bem distribuído podem pagar mais com a nova estrutura
- Não há plano intermediário — o salto de $32 para $96 é expressivo
- Migração automática para o plano Standard pode reduzir créditos disponíveis para quem era power user no plano anterior

### Exemplo prático

```
# Cenário: time de 12 devs
# Antes: $19/seat × 12 = $228/mês (créditos compartilhados, imprevisível)
# Depois:
# - 9 devs Standard: 9 × $32 = $288/mês
# - 3 leads Premium: 3 × $96 = $288/mês
# Total: $576/mês — mais caro, mas previsível e sem conflito de créditos

# Para times menores (<5 pessoas), avaliar se o custo por seat do Premium
# é justificado em relação a usar Claude Code individual
```

### Relação com o ecossistema moderno

- **GitHub Copilot**: repricing posiciona Cursor Standard no mesmo range de Copilot Business ($19/mo) mas com features de agentes mais avançadas
- **Devin Desktop/Windsurf**: o segmento de preço é o mesmo — a guerra de pricing entre IDEs de agente está acelerada
- **Enterprise IT Procurement**: previsibilidade de custos era o bloqueador principal; plano fixo facilita aprovação de procurement
- **Claude Code**: para power users que preferem Claude Code + VS Code como alternativa, o Premium é a comparação direta

### Vale a pena acompanhar?

**Sim, para times que já usam o Cursor.** O repricing resolve um problema real. Para times avaliando a adoção, é o momento de fazer um piloto com plano Standard e identificar quem seria Premium.

---

## 4. Gemini 3.5 Flash GA — Breaking Changes na API e Migração Obrigatória

### O que é

O **Gemini 3.5 Flash** tornou-se GA (Generally Available) no Google I/O em 19 de maio. Com isso, a Google removeu a versão legada da Interactions API **em 8 de junho de 2026** — hoje — criando urgência de migração para todos os projetos que usam a API do Gemini.

**O modelo em si:**
- `gemini-3.5-flash` via Gemini API, AI Studio, Vertex AI
- 289 tokens por segundo — 4x mais rápido que modelos frontier comparáveis
- Contexto: 1M tokens
- Pricing: $1.50/M input tokens, $9.00/M output tokens
- Benchmark: 76.2% no Terminal-Bench 2.1, 83.6% no MCP Atlas

**Breaking changes obrigatórias:**

**1. thinking_budget → thinking_level:**
```ts
// ANTES (deprecated, removido hoje)
const response = await gemini.generate({
  model: 'gemini-3.5-flash',
  thinkingConfig: {
    thinkingBudget: 8192, // inteiro em tokens
  },
})

// DEPOIS (obrigatório a partir de 08/06/2026)
const response = await gemini.generate({
  model: 'gemini-3.5-flash',
  thinkingConfig: {
    thinkingLevel: 'medium', // 'none' | 'low' | 'medium' | 'high'
  },
})
```

**2. Default de thinking mudou: `high` → `medium`:**
Código que usava o default implícito pode ter mudança de comportamento — respostas mais rápidas mas com menos raciocínio chain-of-thought. Para tarefas de código e análise, verificar se `medium` entrega qualidade equivalente ao `high` anterior.

**3. Remoção da Interactions API legacy (08/06/2026):**
A API anterior de multi-turn conversations via `interactions.create()` foi removida. O padrão agora é `chats.create()` com o mesmo esquema de `messages`.

### Por que isso importa

A remoção da Interactions API é uma mudança breaking real com data definida — **hoje, 8 de junho**. Qualquer projeto que usa o SDK `@google/generative-ai` em versões antigas ou chama a API diretamente pode estar quebrando em produção agora.

A mudança de `thinking_budget` (inteiro) para `thinking_level` (enum) é mais elegante do ponto de vista de API — inteiros de tokens eram opacos para desenvolvedores, enums semânticos (`low`, `medium`, `high`) são mais fáceis de raciocinar.

### Benefícios práticos

- API mais semântica — `thinking_level: 'high'` é mais expressivo que `thinkingBudget: 16384`
- Default `medium` é mais rápido — tasks comuns têm menor latência sem configuração explícita
- Modelo 4x mais rápido que predecessores com qualidade frontier — excelente para streaming em UI
- 1M context window — processa codebases inteiras em uma chamada

### Possíveis problemas ou limitações

- **Breaking today**: projetos não migrados quebram em produção 8 de junho
- Default `high` → `medium` pode reduzir qualidade em tasks que dependem de raciocínio profundo sem configuração explícita
- Pricing mais alto que Gemini 3.1 ($1.50 vs $0.35/M tokens) — revisar estimativas de custo
- Suporte a Gemini 3.5 Flash no Vertex AI pode ter delay de 1-2 semanas em regiões não-US

### Exemplo prático

```ts
// Migração completa para Gemini 3.5 Flash GA

// ANTES
import { GoogleGenerativeAI } from '@google/generative-ai'
const genAI = new GoogleGenerativeAI(apiKey)
const model = genAI.getGenerativeModel({ model: 'gemini-2.5-flash' })
const chat = await model.startChat() // Interactions API

// DEPOIS
import { GoogleGenAI } from '@google/genai' // novo SDK
const ai = new GoogleGenAI({ apiKey })

const response = await ai.models.generateContent({
  model: 'gemini-3.5-flash',
  contents: [{ role: 'user', parts: [{ text: prompt }] }],
  config: {
    thinkingConfig: {
      thinkingLevel: 'medium', // 'none' | 'low' | 'medium' | 'high'
    },
    temperature: 0.2,
  },
})

// Multi-turn (substitui Interactions API)
const chat = ai.chats.create({
  model: 'gemini-3.5-flash',
  history: previousMessages,
})
const message = await chat.sendMessage({ message: 'Continue...' })
```

### Relação com o ecossistema moderno

- **Vercel AI SDK**: suporte ao `gemini-3.5-flash` via `@ai-sdk/google` — migração automática ao atualizar o SDK
- **LangChain**: `@langchain/google-genai` requer atualização para suportar o novo SDK
- **Antigravity 2.0**: modelo nativo — Browser Subagent e CLI usam Gemini 3.5 Flash por padrão
- **Google AI Studio**: atualizado — playground já usa o novo modelo
- **Edge Runtime**: Gemini 3.5 Flash funciona bem em Edge Functions pela baixa latência

### Vale a pena acompanhar?

**Sim, vale acompanhar — e migrar agora.** A remoção da Interactions API é urgente. O modelo em si é genuinamente rápido — 289 tok/s é relevante para streaming de UI em produção.

---

## 5. MCP Release Candidate — Stateless Operation, Server Cards e MCP Apps

### O que é

O Model Context Protocol travou seu **Release Candidate** em 21 de maio de 2026. A especificação final está programada para **28 de julho de 2026** — a janela atual é para que SDK maintainers e implementadores de clientes validem as mudanças. O que está no RC e muda a forma como servidores MCP são construídos e descobertos:

**1. Stateless Operation (a mudança mais impactante):**
Servidores MCP atuais precisam manter estado de sessão — o que os impede de escalar horizontalmente. Cada reconexão precisa cair no mesmo processo. O RC padroniza **session creation, resumption e migration**: o estado de sessão pode ser serializado e migrado para outro processo, tornando restart de servidor e scale-out transparentes para o cliente.

```
Antes: cliente → servidor (stateful, sessão fixada ao processo)
       restart do servidor = reconexão forçada, perda de estado

Depois: cliente → servidor A ou B ou C (stateless, estado no cliente)
        restart do servidor = sessão retomada no próximo processo disponível
```

**2. MCP Server Cards (`.well-known`):**
Padrão para discovery de servidores MCP via URL pública. Um servidor em `https://mcp.example.com` expõe suas capacidades em `https://mcp.example.com/.well-known/mcp-server-card.json`. Browsers, crawlers e registries podem descobrir automaticamente o que o servidor oferece sem se conectar.

```json
// .well-known/mcp-server-card.json
{
  "name": "figma-mcp",
  "version": "2.1.0",
  "capabilities": ["tools", "resources"],
  "tools": [
    { "name": "get_design_context", "description": "..." },
    { "name": "code_connect_lookup", "description": "..." }
  ],
  "auth": ["oauth2", "api-key"],
  "rateLimit": { "requestsPerMinute": 60 }
}
```

**3. MCP Apps — HTML Rico em Iframes Sandboxed:**
Extensão oficial ao protocolo (built on `mcp-ui`) que permite tools retornarem HTML interativo renderizado em iframe sandboxed dentro do contexto do chat/agente. Em vez de uma tool retornar texto JSON com dados de analytics, ela pode retornar um mini-dashboard HTML com gráficos, formulários e tabelas — renderizado diretamente na conversa.

Isso muda o que é possível construir com MCP: até a v0.21 tudo era texto. MCP Apps permite experiências ricas sem sair do contexto de agente.

### Por que isso importa

Stateless operation é a mudança técnica mais importante do MCP desde o lançamento. Ela desbloqueia uso em produção com múltiplas instâncias de servidor — o que atualmente não é viável para servidores com estado. Para qualquer empresa que queira hospedar MCP servers de forma confiável, isso é pré-requisito.

Server Cards democratiza o discovery — em vez de cada ferramenta ter sua própria página de documentação de configuração, o discovery pode ser automatizado.

### Benefícios práticos

- Stateless: MCP servers finalmente escaláveis horizontalmente com load balancer
- Server Cards: IDEs e orquestradores podem descobrir e configurar servidores automaticamente
- MCP Apps: UIs ricas sem sair do contexto de agente — dashboards, formulários, visualizações
- RC estável: SDK maintainers podem começar implementação sem risco de especificação mudar
- Janela de 10 semanas (21/05 → 28/07): tempo para validar antes da versão final

### Possíveis problemas ou limitações

- Stateless requer mudança de arquitetura nos servidores existentes — não é backward compatible sem camada de adaptação
- MCP Apps renderiza HTML em sandbox — interatividade real é limitada por política de sandbox do implementador
- Server Cards requer que servidores sirvam `.well-known` — servidores locais/stdio não se beneficiam do discovery
- Muitos servidores MCP populares ainda não implementaram o RC — gap entre spec e implementações reais

### Exemplo prático

```ts
// MCP Tool retornando MCP App (HTML interativo)
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js'
import { createUiResult } from 'mcp-ui'

server.tool('get_component_usage_dashboard', async ({ componentName }) => {
  const data = await fetchUsageMetrics(componentName)

  // Em vez de retornar JSON plano, retorna HTML interativo
  return createUiResult({
    html: `
      <div style="font-family: system-ui; padding: 16px;">
        <h2>${componentName} — Usage Dashboard</h2>
        <table>
          <tr><th>Route</th><th>Renders/day</th><th>Avg Load</th></tr>
          ${data.map(row => `
            <tr>
              <td>${row.route}</td>
              <td>${row.renders}</td>
              <td>${row.loadTime}ms</td>
            </tr>
          `).join('')}
        </table>
      </div>
    `,
    sandbox: 'allow-scripts',
  })
})
```

```ts
// Stateless server — implementação com o RC
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js'

const server = new McpServer({
  name: 'my-server',
  version: '1.0.0',
  // RC: stateless mode — sessões gerenciadas pelo cliente
  session: {
    mode: 'stateless',
    // Estado da sessão serializado e enviado em cada request
  },
})
```

### Relação com o ecossistema moderno

- **Claude Code**: suporte ao RC planejado para as próximas semanas — stateless mode melhora estabilidade de sessões longas
- **Figma MCP**: Server Cards permitirão que IDEs descubram o servidor Figma automaticamente
- **Antigravity 2.0 SDK**: SDK compatível com MCP e beneficia de stateless para Managed Agents
- **Devin Desktop (ACP)**: ACP e MCP são camadas complementares — ACP para orquestração, MCP para ferramentas
- **Svelte MCP**: stdio mode se beneficia do stateless para sessões de desenvolvimento local mais estáveis

### Vale a pena acompanhar?

**Sim, vale acompanhar.** Se você mantém servidores MCP ou usa MCP em produção, o stateless operation é um pré-requisito técnico para escala. A especificação final em julho é uma âncora para planejar implementações.
