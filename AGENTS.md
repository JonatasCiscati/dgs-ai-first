# AGENTS.md — NovaTech Assistant

> Instruções operacionais para agentes de IA (GitHub Copilot e similares) que geram ou modificam código neste repositório. As regras abaixo são **prescritivas**: ao escrever testes, siga-as à risca. Quando uma regra usa **DEVE** / **NÃO DEVE**, trate-a como obrigatória — código que a viola não passa em revisão.

---

## Testing Standards

### Stack (decisão do Tech Lead — não alterar sem ADR)

| Item | Ferramenta | Observação |
|------|-----------|------------|
| Test runner | **Vitest** | unit e integração |
| Mock de APIs externas | **msw** (Mock Service Worker) | Azure AI Search, Azure OpenAI |
| CI | **GitHub Actions** | roda em todo PR; bloqueia merge se falhar |
| Cobertura mínima | **80%** | linhas por módulo; abaixo disso o merge é bloqueado |

> Esta seção operacionaliza a decisão de stack registrada pelo Tech Lead. Mudanças de framework/ferramenta exigem um ADR aprovado, não uma decisão pontual em PR.

---

### Nomenclatura `describe` / `it`

- Arquivos: `*.test.ts` (unit, co-localizado ao módulo) e `*.integration.test.ts` em `tests/integration/`.
- `describe('UnidadeSobTeste', ...)` — nomeia o módulo, classe ou handler real (ex.: `QueryHandler`, `ResponseValidator`). **NÃO DEVE** ser genérico (`describe('tests', ...)`).
- `it('should <comportamento esperado> when <condição>', ...)` — sempre em formato de frase verificável, descrevendo **comportamento de negócio**, não mecânica.
- Use `it`, **NÃO** `test`, por consistência (apenas um dos dois no repositório).

**CORRETO**
```typescript
describe('QueryHandler', () => {
  it('should return an answer with source_document when a matching chunk is found', ...)
  it('should refuse to answer when retrieval returns no chunk above the score threshold', ...)
  it('should cite POL-001 when the question is about the return policy', ...)
})
```

**INCORRETO** (anti-padrões reais a evitar)
```typescript
test('query endpoint works', ...)   // "works" não é comportamento; "test" em vez de "it"
it('test 1', ...)                    // não descreve nada
describe('handler', () => { ... })   // genérico, não nomeia a unidade real
```

---

### Padrão AAA (Arrange · Act · Assert)

Todo teste **DEVE** ter as três fases visualmente separadas por uma linha em branco, nesta ordem e **uma única vez** por teste:

```typescript
it('should refuse to answer when no chunk is above the score threshold', async () => {
  // Arrange — monta entrada e estado dos mocks
  server.use(searchReturnsNoResults());
  const event = makeQueryEvent({ question: 'Qual o prazo de devolução?' });

  // Act — executa exatamente UMA ação sob teste
  const response = await handler(event);

  // Assert — verifica o resultado observável
  expect(response.statusCode).toBe(200);
  expect(response.body.answer).toMatch(/não encontrei|não há informação/i);
  expect(response.body.source_document).toBeNull();
});
```

- **Act DEVE conter uma única chamada** ao alvo. Se precisa de duas, provavelmente são dois testes.
- **NÃO DEVE** intercalar Act e Assert em loop ("act, assert, act, assert") — isso esconde qual ação causou a falha.

---

### O que todo teste DEVE ter

1. **Nome que descreve comportamento de negócio** (`should ... when ...`), não a implementação.
2. **Asserções específicas sobre o valor observável**: shape da resposta, campos, status, conteúdo. Ex.: `expect(response.body.source_document).toBe('POL-001')`.
3. **Asserção sobre `source_document`** em todo teste que exercita o pipeline RAG — é requisito de produto que toda resposta seja rastreável à fonte.
4. **Mocks isolados por teste**: `server.resetHandlers()` em `afterEach`; nenhum teste depende do estado deixado por outro.
5. **Entradas construídas por factories/fixtures** nomeadas (ver abaixo), não JSON solto e mágico.
6. **Determinismo**: mesmo input → mesmo resultado, sem rede real, sem relógio real (`vi.useFakeTimers()` quando houver tempo/SLA envolvido).

### O que nenhum teste DEVE ter

1. **`expect(result).toBeDefined()` / `toBeTruthy()` como asserção principal.** Não prova comportamento — só prova que algo retornou. Asserte o conteúdo.
2. **Chamadas reais a Azure AI Search ou Azure OpenAI.** Toda saída de rede passa por msw. Um teste que vaza para a rede é um bug de teste.
3. **`question: 'test'` ou strings vazias de placeholder.** Use perguntas de domínio reais (devolução, frete, SLA).
4. **Lógica condicional (`if`/`try-catch`) no corpo do teste** para "passar nos dois casos". Um teste afirma **um** comportamento.
5. **Sleep/`setTimeout` arbitrário** para esperar assíncrono — use `await` e fake timers.
6. **Snapshots gigantes** como substituto de asserção pensada.

> **Anti-padrão de referência — NÃO replicar:**
> ```typescript
> test('query endpoint works', async () => {
>   const result = await handler({ body: '{"question": "test"}' });
>   expect(result).toBeDefined();
> });
> ```
> Problemas: nome não descreve comportamento; `test` em vez de `it`; sem fases AAA; `body` montado à mão como string crua; pergunta `"test"` sem domínio; `toBeDefined()` não verifica `answer`, `source_document` nem `statusCode`; nenhum mock de msw — pode até vazar para a rede. **Este teste passaria mesmo com o handler completamente quebrado.**

---

### Mocking — msw + factories

Toda fronteira externa (Azure AI Search, Azure OpenAI) é simulada via msw. **NÃO** mocke o `fetch`/SDK diretamente nem stub manual por teste — registre handlers no servidor msw compartilhado.

**Setup obrigatório** (em `tests/setup.ts`, carregado via `setupFiles` no `vitest.config.ts`):
```typescript
import { afterAll, afterEach, beforeAll } from 'vitest';
import { server } from './mocks/server';

beforeAll(() => server.listen({ onUnhandledRequest: 'error' })); // request não mockada = falha
afterEach(() => server.resetHandlers());                          // isolamento entre testes
afterAll(() => server.close());
```

`onUnhandledRequest: 'error'` é obrigatório: garante que nenhuma chamada de rede escape sem mock.

**Factories de handler** (em `tests/mocks/handlers/`) — funções nomeadas que descrevem o cenário, retornando handlers msw. Cada teste compõe seu cenário com `server.use(...)`:
```typescript
// tests/mocks/handlers/azure-search.ts
export const searchReturnsChunk = (chunk: SearchChunk) =>
  http.post(`${SEARCH_URL}/docs/search`, () =>
    HttpResponse.json({ value: [chunk] }));

export const searchReturnsNoResults = () =>
  http.post(`${SEARCH_URL}/docs/search`, () =>
    HttpResponse.json({ value: [] }));

export const openAiReturnsCompletion = (content: string) =>
  http.post(`${OPENAI_URL}/chat/completions`, () =>
    HttpResponse.json({ choices: [{ message: { role: 'assistant', content } }] }));

export const openAiTimesOut = () =>
  http.post(`${OPENAI_URL}/chat/completions`, () => HttpResponse.error());
```

**Factories de entrada** — nunca monte o evento à mão; use uma factory com defaults sensatos e override por teste:
```typescript
// tests/factories/query-event.ts
export const makeQueryEvent = (overrides: Partial<QueryInput> = {}) => ({
  body: JSON.stringify({ question: 'Qual é o prazo de devolução?', ...overrides }),
});
```

---

### Fixtures de domínio

Fixtures vivem em `tests/fixtures/` e refletem a documentação de logística real. Use os identificadores canônicos — **NÃO** invente IDs nem use `'doc1'`/`'test'`.

| Documento | ID canônico | Domínio coberto |
|-----------|-------------|-----------------|
| Política de devolução | `POL-001` | prazos, condições, exceções de devolução |
| Procedimento de frete | `PROC-042-v2` | cálculo de frete, classes de carga |
| Acordo de nível de serviço | `SLA-2024` | prazos de resposta, janelas de atendimento |

```typescript
// tests/fixtures/chunks.ts
export const chunkPolDevolucao: SearchChunk = {
  id: 'POL-001#03',
  source_document: 'POL-001',
  content: 'O prazo para devolução é de 30 dias corridos a partir do recebimento.',
  score: 0.92,
};

export const chunkFreteCargaPerigosa: SearchChunk = {
  id: 'PROC-042-v2#11',
  source_document: 'PROC-042-v2',
  content: 'Cargas das classes ANTT 1 a 6 não são elegíveis para frete padrão.',
  score: 0.88,
};
```

Asserções de RAG devem casar com a fixture, garantindo rastreabilidade da resposta:
```typescript
server.use(searchReturnsChunk(chunkPolDevolucao), openAiReturnsCompletion('O prazo é de 30 dias.'));
// ...
expect(response.body.source_document).toBe('POL-001');
```

---

### Cobertura

- Configure `coverage` no `vitest.config.ts` com `provider: 'v8'` e `thresholds: { lines: 80, ... }`.
- Cobertura < 80% **DEVE** falhar o job de CI — não é métrica informativa, é gate de merge.
- **NÃO** persiga 80% com testes vazios (`toBeDefined()`); cobertura sem asserção de comportamento é dívida, não qualidade.
