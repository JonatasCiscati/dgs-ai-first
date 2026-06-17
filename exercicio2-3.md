# Exercício 2.3

## SKILL.md — `create-integration-test`

---

```markdown
# Skill: create-integration-test

**Nível:** Artifact
**Caminho no repositório:** `skills/artifact/create-integration-test.md`
**Versão:** 1.0

---

## Quando usar esta skill

Use esta skill sempre que precisar criar um **teste de integração** para um módulo do projeto NovaTech Assistant. Frases que ativam esta skill:

- "crie um teste de integração para [módulo]"
- "escreva um integration test para o handler de [endpoint]"
- "adicione testes de integração cobrindo o VC-XX"
- "implemente os cenários TC-XX-YY do test-plan.md"

Testes de integração neste projeto testam a integração entre módulos internos (ex: `handler` + `prompt-builder` + `response-validator`) usando msw para simular respostas de APIs externas (Azure AI Search, Azure OpenAI). Não fazem chamadas reais a serviços Azure.

---

## Dependências — ler antes de usar esta skill

1. `skills/foundation/typescript-conventions.md` — imports, naming, strict mode *(arquivo criado, vazio nesta fase — a ser escrito; ver Anexo C)*
2. `skills/foundation/error-handling.md` — como erros de integração devem ser tratados e assertados *(arquivo criado, vazio nesta fase — a ser escrito)*
3. `skills/domain/testing-patterns.md` — configuração do servidor msw, factories, fixtures *(arquivo criado, vazio nesta fase — a ser escrito; é o pré-requisito direto desta skill)*

---

## Template

```typescript
import { describe, it, expect, beforeAll, afterEach, afterAll } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { {{HANDLER_FUNCTION}} } from '{{HANDLER_PATH}}';
import { chunks, queries, expectedResponses } from '@tests/fixtures';
import { build{{REQUEST_TYPE}} } from '@tests/factories';

const server = setupServer();

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('{{ModuleName}}', () => {
  describe('{{behavior group}}', () => {
    it('should {{expected behavior}} when {{condition}}', async () => {
      // Arrange
      const request = build{{REQUEST_TYPE}}({ {{INPUT_FIELDS}} });
      server.use(
        http.post('{{AZURE_SEARCH_URL}}', () => HttpResponse.json({{SEARCH_MOCK_RESPONSE}})),
        http.post('{{AZURE_OPENAI_URL}}', () => HttpResponse.json({{COMPLETION_MOCK_RESPONSE}}))
      );

      // Act
      const response = await {{HANDLER_FUNCTION}}(request);
      const body = JSON.parse(response.body);

      // Assert
      expect(response.statusCode).toBe({{EXPECTED_STATUS}});
      expect(body.{{FIELD}}).{{ASSERTION}}({{EXPECTED_VALUE}});
      expect(body.source_document).toBe('{{EXPECTED_SOURCE}}');
    });
  });
});
```

**Placeholders obrigatórios:**
| Placeholder | Descrição |
|-------------|-----------|
| `{{HANDLER_FUNCTION}}` | Nome da função handler (ex: `queryHandler`) |
| `{{HANDLER_PATH}}` | Caminho relativo ao handler (ex: `../../src/functions/query/handler`) |
| `{{ModuleName}}` | Nome do módulo no `describe` raiz (ex: `QueryHandler`) |
| `{{REQUEST_TYPE}}` | Tipo do request factory (ex: `QueryRequest`) |
| `{{INPUT_FIELDS}}` | Campos do request com dados de fixture (ex: `question: queries.prazoDeVolucao`) |
| `{{EXPECTED_STATUS}}` | Status HTTP esperado (ex: `200`, `400`) |
| `{{FIELD}}` | Campo do body a assertar (ex: `answer`, `source_document`) |
| `{{EXPECTED_VALUE}}` | Valor ou substring esperada |
| `{{EXPECTED_SOURCE}}` | Identificador do documento fonte (ex: `"POL-001"`, `"PROC-042-v2"`) |

---

## Exemplo — DO (teste correto)

**Cenário:** TC-03-HP — Devolução de carga perigosa deve retornar negativa explícita

> Este exemplo reproduz fielmente o cenário **TC-03-HP** do test-plan.md (Exercício 2.2): mesmos chunks (`chunks.pol001B`), mesma query de domínio (`queries.devolucaoClasseAntt3`) e mesma assertion de negativa explícita. A consistência entre a skill e o test plan é proposital — quem implementa o teste rastreia a origem do cenário.

```typescript
import { describe, it, expect, beforeAll, afterEach, afterAll } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { queryHandler } from '../../src/functions/query/handler';
import { chunks, queries } from '@tests/fixtures';
import { buildQueryRequest, buildSearchResponse, buildCompletionResponse } from '@tests/factories';

const server = setupServer();

beforeAll(() => server.listen({ onUnhandledRequest: 'error' }));
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('QueryHandler', () => {
  describe('dangerous cargo return policy', () => {
    it('should return explicit denial when ANTT class 3 cargo return is requested', async () => {
      // Arrange
      const request = buildQueryRequest({ question: queries.devolucaoClasseAntt3 });
      server.use(
        http.post('https://novatech-search.search.windows.net/indexes/*/docs/search', () =>
          HttpResponse.json(buildSearchResponse([chunks.pol001B]))
        ),
        http.post('https://novatech-openai.openai.azure.com/*/chat/completions', () =>
          HttpResponse.json(buildCompletionResponse(
            'Carga perigosa classe 3 ANTT não pode ser devolvida pelo processo padrão. Entre em contato com Gestão de Riscos pelo ramal 4500.'
          ))
        )
      );

      // Act
      const response = await queryHandler(request);
      const body = JSON.parse(response.body);

      // Assert
      expect(response.statusCode).toBe(200);
      expect(body.answer).toMatch(/não pode ser devolvida/i);
      expect(body.answer).toContain('Gestão de Riscos');
      expect(body.source_document).toBe('POL-001');
    });
  });
});
```

---

## Exemplo — DON'T (problemas comuns gerados por IA)

```typescript
// PROBLEMAS:
// 1. Sem describe — perde contexto no relatório de CI
// 2. Dado de teste genérico: "test" não é uma pergunta de domínio
// 3. Sem mock de HTTP — chama Azure real em ambiente de teste
// 4. toBeDefined() não valida comportamento de negócio
// 5. Sem verificação do source_document
// 6. Input como string crua em vez de factory tipada

test('query endpoint returns result', async () => {
  const result = await queryHandler({
    body: JSON.stringify({ question: 'test' })
  });
  expect(result).toBeDefined();
  expect(result.statusCode).toBe(200);
});
```

---

## Anti-padrões Específicos de LLMs em Testes

| Anti-padrão | Por que acontece | Como corrigir |
|-------------|-----------------|---------------|
| Dados genéricos: `question: "test"`, `answer: "response"` | LLM preenche com placeholder sem contexto de domínio | Usar fixtures de `/tests/fixtures/queries.ts` com perguntas reais de logística |
| `expect(result).toBeDefined()` sozinho | LLM prioriza cobertura sintática, não semântica | Substituir por assertions específicas: `.toBe()`, `.toContain()`, `.toMatch()` |
| Ausência de comentários AAA | LLM omite estrutura quando não instruído | Incluir `// Arrange`, `// Act`, `// Assert` explicitamente no template |
| Mock direto de módulos internos (`vi.mock('../services/search')`) sem necessidade | LLM isola tudo por padrão | Mockar apenas a fronteira HTTP com msw; módulos internos devem ser testados reais |
| `beforeAll` com dados mutáveis compartilhados | LLM não considera paralelismo de testes | Usar `server.use()` dentro de cada `it` ou `beforeEach`; nunca mutá-los em `beforeAll` |
| `console.log` para depuração | LLM adiciona para "facilitar debug" | Remover todo `console.log` do corpo de testes; usar `logger` do pino se necessário |
| `server.listen()` sem `{ onUnhandledRequest: 'error' }` | LLM usa o setup padrão do msw, que deixa requests não-mockados **passarem silenciosamente** | Sempre `server.listen({ onUnhandledRequest: 'error' })` — qualquer chamada que escape do mock falha o teste, evitando chamada real a Azure AI Search / Azure OpenAI |

---

## Critérios de Maturidade desta Skill

Esta skill está pronta para uso pelo time quando:

1. Testes gerados com ela passam no CI na primeira tentativa (sem ajuste manual de imports)
2. Os placeholders produzem assertions específicas ao domínio (não genéricas) quando substituídos com dados de fixture
3. A revisão QA do checklist abaixo é concluída em menos de 2 minutos por teste
```

---

## Checklist de Revisão de Testes

> Revisão deve ser concluída em menos de 2 minutos por teste.

| # | Item | Verificação | Resultado |
|---|------|-------------|-----------|
| 1 | **Nomenclatura** | O teste usa `describe('NomeDoModulo', () => { it('should [comportamento] when [condição]') })`? | [ ] Sim / [ ] Não |
| 2 | **AAA explícito** | O corpo do `it` contém os comentários `// Arrange`, `// Act` e `// Assert`? | [ ] Sim / [ ] Não |
| 3 | **Dados de domínio** | As strings de `question` e dados de teste vêm de fixtures ou são perguntas reais de logística (NÃO contêm `"test"`, `"foo"`, `"hello"`, `"sample"`)? | [ ] Sim / [ ] Não |
| 4 | **Mock de HTTP** | Todo acesso a Azure AI Search e Azure OpenAI passa por handler msw registrado no `it` ou no `beforeEach`? | [ ] Sim / [ ] Não |
| 5 | **Assertion de comportamento** | Há ao menos uma assertion específica sobre o conteúdo de `body.answer` (`.toContain()`, `.toMatch()`, `.toBe()`)? | [ ] Sim / [ ] Não |
| 6 | **source_document validado** | Para testes do query endpoint, o campo `body.source_document` é assertado com valor específico (não apenas `toBeDefined()`)? | [ ] Sim / [ ] Não |
| 7 | **Sem efeitos colaterais** | O teste não modifica arquivos, não deixa handlers msw ativos (usa `afterEach(() => server.resetHandlers())`), não depende de outro teste ter rodado antes? | [ ] Sim / [ ] Não |
| 8 | **Sem console.log** | O corpo do teste não contém `console.log`, `console.error` ou `console.warn`? | [ ] Sim / [ ] Não |

**Aprovado se:** todos os 8 itens marcados como "Sim".
**Reprovado se:** qualquer item marcado como "Não" — devolver com o número do item e a correção esperada.

---

## Registro de Uso da Ferramenta (Claude + Claude Cowork)

### Etapa 1 — Claude (autoria do SKILL.md)

**Prompt inicial:**
```
Contexto: projeto NovaTech Assistant. Crie o SKILL.md "create-integration-test"
(nível Artifact, caminho skills/artifact/). Deve conter: frase-ativação, template com
placeholders, exemplo DO (teste bem escrito) e DON'T (problemas comuns de IA),
anti-padrões específicos de testes gerados por LLM, e dependências (skills Foundation
e Domain a ler antes).

Mantenha consistência com os Testing Standards já definidos (Exercício 2.1): describe/it,
AAA, assertions específicas, msw para HTTP, factories, fixtures. O exemplo DO deve
reusar um cenário real do test-plan (TC-03-HP — devolução de carga classe 3 ANTT).
```

### Etapa 2 — Iteração v1 → v2 (Claude)

| # | v1 | v2 | Por que mudou |
|---|----|----|---------------|
| 1 | Anti-padrões genéricos de teste ("não escreva testes frágeis") | Anti-padrões **específicos de LLM** com coluna "Por que acontece" (`toBeDefined()` → "LLM prioriza cobertura sintática"; `vi.mock` interno → "LLM isola tudo por padrão") | v1 não diferenciava erro de LLM de erro humano; v2 ataca o que o agente realmente gera errado |
| 2 | Exemplo DO inventado, desconectado | DO reproduz TC-03-HP do test-plan (mesmos chunks/queries) | v1 quebrava a rastreabilidade entre skill e test plan; v2 amarra os artefatos |
| 3 | Template sem `onUnhandledRequest` | Template com `listen({ onUnhandledRequest: 'error' })` **e** anti-padrão correspondente explicando o risco de omitir | v1 deixava o agente reproduzir o setup padrão inseguro do msw |
| 4 | Critérios de "pronta" subjetivos | Critérios de Maturidade mensuráveis ("passa no CI na 1ª tentativa", "review < 2 min") | v1 não permitia saber quando adotar a skill |

**Prompt de refino usado:**
```
Os anti-padrões da v1 são genéricos. Reescreva focando no que LLMs realmente erram em
testes (toBeDefined sozinho, dados "test", mock de módulo interno, beforeAll mutável,
console.log) com a causa de cada um. Conecte o exemplo DO ao TC-03-HP do test-plan.
Inclua onUnhandledRequest:'error' no template e como anti-padrão se omitido.
```

### Etapa 3 — Claude Cowork (checklist de revisão)

O Cowork foi usado para derivar e calibrar o checklist de revisão:
- **8 itens**, cada um mapeado **1:1** a uma regra dos Testing Standards (Ex. 2.1): nomenclatura, AAA, dados de domínio, mock HTTP, assertion de comportamento, `source_document`, sem efeitos colaterais, sem `console.log`.
- Formato **binário Sim/Não** (calibrado para revisão em < 2 min por teste — sem itens subjetivos).
- Critério de aprovação/reprovação explícito, com **instrução de devolução** (número do item + correção esperada) para que a revisão seja acionável.
