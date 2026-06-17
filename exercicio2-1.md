# Exercício 2.1

## Testing Standards — Seção do AGENTS.md

---

```markdown
## Testing Standards

### Framework & Ferramentas

- **Test runner:** Vitest
- **Mocking de HTTP externo:** msw (Mock Service Worker) — obrigatório para todas as chamadas a Azure AI Search, Azure OpenAI e qualquer API externa
- **CI:** GitHub Actions (`.github/workflows/ci.yml`) — todos os testes executam em todo PR antes do merge
- **Cobertura mínima:** 80% de linhas por módulo; cobertura abaixo disso bloqueia o merge

> **Procedência da decisão:** stack de testes (Vitest + msw + cobertura mínima de 80%) conforme decisão do Tech Lead registrada em `docs/adr/NNNN-stack-de-testes.md` (numerar conforme a sequência real de `docs/adr/`, ex.: `0007-stack-de-testes.md`). Esta seção do AGENTS.md operacionaliza essa decisão para agentes de IA.

### Nomenclatura de Testes

- Arquivos de teste: `*.test.ts`, co-localizados ao módulo ou em `tests/unit/` e `tests/integration/`
- Bloco de contexto: `describe('NomeDoModulo', () => { ... })`
- Caso individual: `it('should [comportamento esperado] when [condição]', ...)`

**CORRETO:**
```typescript
describe('QueryHandler', () => {
  it('should return source_document in every response when chunk is found', ...)
  it('should deny return for dangerous cargo when class 1-6 ANTT is detected', ...)
})
```

**INCORRETO:**
```typescript
test('works', ...)
test('query endpoint', ...)
it('test 1', ...)
```

### Estrutura de Teste — DEVE TER

Todo teste DEVE seguir o padrão Arrange / Act / Assert (AAA) com comentários explícitos:

```typescript
it('should deny return for dangerous cargo when ANTT class 3 is specified', async () => {
  // Arrange
  const request = buildQueryRequest({ question: queries.devolucaoClasseAntt3 });
  server.use(
    mockAzureSearchResponse([chunks.pol001B]),
    mockAzureOpenAICompletion('Carga perigosa classe 3 ANTT não pode ser devolvida pelo processo padrão.')
  );

  // Act
  const response = await handler(request);
  const body = JSON.parse(response.body);

  // Assert
  expect(response.statusCode).toBe(200);
  expect(body.answer).toMatch(/não pode ser devolvida/i);
  expect(body.source_document).toBe('POL-001');
});
```

Todo teste DEVE ter:
- Ao menos uma assertion sobre o **comportamento de negócio** (não apenas sobre o formato da resposta)
- Validação do campo `source_document` em testes do query endpoint
- Setup de dados explícito no próprio teste — nunca depender de estado global mutável

### Estrutura de Teste — NÃO DEVE TER

- Chamadas diretas a Azure AI Search, Azure OpenAI ou qualquer HTTP externo — usar msw
- Testes que dependem de ordem de execução (estado compartilhado via `beforeAll` que muta entre testes)
- Assertions sozinhas sem validação de valor: `toBeDefined()`, `toBeTruthy()`, `toBeNull()` sem `expect(value.field).toBe(...)` em seguida
- Valores hardcoded de ambiente: connection strings, API keys, URLs de serviço
- `console.log` dentro do corpo de testes

### Mocking

**HTTP externo (Azure OpenAI, Azure AI Search):**
```typescript
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';

const server = setupServer();

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Por teste, sobrescrever com handler específico:
server.use(
  http.post('https://*/openai/deployments/*/chat/completions', () =>
    HttpResponse.json(buildOpenAIResponse('O prazo de devolução é de 7 dias úteis.'))
  )
);
```

**Dados de domínio:** usar factories — nunca inline object literals para entidades do domínio:
```typescript
// CORRETO
const request = buildQueryRequest({ question: queries.prazoDeVolucao });

// INCORRETO
const request = { body: JSON.stringify({ question: 'test' }) };
```

### Fixtures

Dados compartilhados de teste residem em `/tests/fixtures/`:

| Arquivo | Conteúdo |
|---------|----------|
| `chunks.ts` | Chunks RAG pré-construídos (`pol001A`, `pol001B`, `proc042v2B`, `sla2024A`, etc.) |
| `queries.ts` | Perguntas realistas do domínio de logística (nunca `"test"`, `"hello"`, `"foo"`) |
| `expected-responses.ts` | Shapes de resposta esperada por cenário |

Importar com named exports:
```typescript
import { chunks, queries, expectedResponses } from '@tests/fixtures';
```

### Cobertura por Camada

| Camada | Cobertura mínima | Escopo |
|--------|-----------------|--------|
| `tests/unit/` | 80% de linhas | Sem chamadas externas — mocks para tudo |
| `tests/integration/` | Todos os VCs do requirements.md | msw para APIs externas; integração entre módulos internos real |
| `tests/e2e/` | Apenas o caminho crítico | Fluxo completo query → resposta com source; **máximo 1 cenário e2e por módulo** (somente o caminho crítico). Novos e2e exigem aprovação do Tech Lead — consomem tokens reais |
```

---

## Reescrita do Teste — Antes e Depois

### Antes (teste gerado por IA sem guidance)

```typescript
test('query endpoint works', async () => {
  const result = await handler({ body: '{"question": "test"}' });
  expect(result).toBeDefined();
});
```

### Depois (teste reescrito seguindo os padrões)

```typescript
describe('QueryHandler', () => {
  it('should return an answer with source_document when a valid question is asked', async () => {
    // Arrange
    const request = buildQueryRequest({ question: queries.prazoDeVolucao });
    server.use(
      mockAzureSearchResponse([chunks.pol001A]),
      mockAzureOpenAICompletion('O prazo de devolução é de 7 dias úteis após o recebimento.')
    );

    // Act
    const response = await handler(request);
    const body = JSON.parse(response.body);

    // Assert
    expect(response.statusCode).toBe(200);
    expect(body.answer).toContain('7 dias úteis');
    expect(body.source_document).toBe('POL-001');
  });
});
```

### Explicação por Melhoria

| # | Problema no original | Correção aplicada | Motivo |
|---|---------------------|-------------------|--------|
| 1 | `test()` sem `describe` | `describe('QueryHandler', () => { it(...) })` | Identifica o módulo e o comportamento; relatórios de CI ficam legíveis |
| 2 | Sem Arrange / Act / Assert | Comentários explícitos separando as três fases | Torna a intenção do teste auditável; facilita diagnóstico de falha |
| 3 | `question: "test"` | `queries.prazoDeVolucao` (fixture de domínio) | Dados genéricos não testam o comportamento real do assistente; uma pergunta de logística real revela falhas de parsing e RAG |
| 4 | Sem mock de HTTP | `server.use(mockAzureSearchResponse(...), mockAzureOpenAICompletion(...))` | Sem mock, o teste chama Azure real — falha no CI, custo de tokens, não-determinístico |
| 5 | Input como string crua | `buildQueryRequest({ question: ... })` | Factory garante tipagem correta e facilita manutenção quando o schema mudar |
| 6 | `expect(result).toBeDefined()` | `expect(response.statusCode).toBe(200)` + `expect(body.answer).toContain(...)` + `expect(body.source_document).toBe(...)` | `toBeDefined()` passa para qualquer objeto não-nulo; as três assertions validam comportamento de negócio concreto |

---

## Critérios de Code Review para Testes Gerados por IA

Todo teste gerado por IA deve atender os três critérios abaixo para ser aprovado no code review de QA:

| # | Critério | Como verificar | Resultado esperado |
|---|----------|---------------|-------------------|
| 1 | **Dados de teste são do domínio de logística da NovaTech** | Ler as strings de `question` e respostas esperadas — se contiverem `"test"`, `"foo"`, `"hello"`, `"sample"` ou qualquer valor genérico, reprovar | Perguntas referenciam termos do domínio: "carga perigosa", "PROC-042", "multiplicador regional", "SLA Gold", "prazo de devolução" |
| 2 | **Todas as assertions validam comportamento, não existência** | Buscar no diff por `toBeDefined()`, `toBeTruthy()`, `toBeNull()` — se aparecerem sozinhos (sem assertion de valor em seguida), reprovar | Assertions especificam valores concretos: `toBe('POL-001')`, `toContain('7 dias úteis')`, `toBe(200)` |
| 3 | **HTTP externo está mockado com msw** | Verificar se o teste importa `server` de `msw/node` ou usa helper de mock; buscar por `fetch`, `axios.get`, `openai.chat.completions.create` sem mock — se presentes, reprovar | Qualquer chamada a Azure AI Search, Azure OpenAI ou outro serviço HTTP usa handler msw registrado antes do `act` |

---

## Registro de Uso da Ferramenta (Claude)

### Prompt inicial

```
Contexto: projeto NovaTech Assistant (RAG sobre documentação de logística — política
de devolução POL-001, frete PROC-042-v2, SLA-2024). Decisão do Tech Lead: Vitest para
unit/integração, msw para mock de APIs externas (Azure AI Search, Azure OpenAI),
CI no GitHub Actions, cobertura mínima de 80%.

Escreva a seção "Testing Standards" do AGENTS.md, prescritiva para agentes de IA
(Copilot). Deve cobrir: nomenclatura describe/it, padrão AAA, o que todo teste DEVE
e NÃO DEVE ter, mocking (msw + factories) e fixtures de domínio.

Teste ruim a corrigir como referência de anti-padrão:
  test('query endpoint works', async () => {
    const result = await handler({ body: '{"question": "test"}' });
    expect(result).toBeDefined();
  });
```

### Iteração v1 → v2

| # | v1 (primeira resposta do Claude) | v2 (refinada após crítica) | Por que mudou |
|---|----------------------------------|----------------------------|---------------|
| 1 | Regras genéricas de teste ("escreva bons nomes", "evite testes frágeis") | DEVE/NÃO DEVE com exemplos de código copiáveis e validação **obrigatória** de `body.source_document` no query endpoint | v1 era conselho que o agente ignora; v2 é regra executável e específica ao contrato do endpoint |
| 2 | "Evite mockar demais" (instrução vaga) | Separou explicitamente: **msw mocka apenas a fronteira HTTP** (Azure); módulos internos são testados reais | v1 levaria o agente a mockar módulos internos por padrão (anti-padrão de LLM); v2 fixa onde o isolamento pertence |
| 3 | Dados de exemplo genéricos (`question: 'foo'`) | Tabela de **fixtures** `/tests/fixtures/` (`chunks.ts`, `queries.ts`, `expected-responses.ts`) + regra "nunca inline object literals para entidades do domínio" | v1 reproduzia o anti-padrão que a seção deveria combater; v2 dá ao agente a fonte de dados de domínio reais |
| 4 | Cobertura mencionada como "ter boa cobertura" | Tabela de cobertura por camada (unit 80% / integration = todos os VCs / e2e = 1 caminho crítico) | v1 não era acionável; v2 dá limiares verificáveis por camada |

**Prompt de refino usado entre v1 e v2:**
```
Esta v1 está narrativa demais — um agente de IA não consegue executar "evite mocks
demais" nem "escreva bons nomes". Reescreva como regras prescritivas: cada DEVE/NÃO
DEVE precisa ser verificável por grep ou inspeção. Force validação de source_document
no query endpoint. Separe mock de fronteira HTTP (msw) de mock de módulo interno.
Substitua dados genéricos por fixtures de domínio reais (POL-001, PROC-042, SLA Gold).
```
