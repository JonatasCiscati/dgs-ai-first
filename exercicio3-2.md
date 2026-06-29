# Exercício 3.2

## Revisão Crítica dos Testes Gerados por IA

Três testes de integração gerados pelo Copilot foram submetidos à revisão de QA antes do merge. A revisão foi feita em três etapas: (1) análise própria de cada teste — o que testa, o que falha em testar, e o risco se o teste "passar" com o código errado — conduzida antes de qualquer apoio de ferramenta; (2) segunda revisão com o Claude e comparação honesta; (3) reescrita do Teste 1 numa versão que verifica o conteúdo da resposta, não apenas a sua existência.

A revisão usa como régua os padrões já definidos para o projeto: a seção **Testing Standards** do AGENTS.md, os **Testing Standards** detalhados em [exercicio2-1.md](exercicio2-1.md) e o **checklist de revisão de testes** (8 itens) da skill `create-integration-test` em [exercicio2-3.md](exercicio2-3.md). O contexto de framework é o do Anexo C: o projeto usa **Vitest**, não jest.

---

## Parte 1 — Análise própria (independente, antes do Claude)

### Teste 1 — Assertions vagas

```typescript
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolução' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});
```

- **O que testa:** que o endpoint `POST /api/query` responde com HTTP 200 e que o corpo da resposta não é `undefined`/`null`.
- **O que falha em testar:** se a resposta está **correta**. `expect(res.body).toBeDefined()` passa para qualquer objeto não-nulo — inclusive `{}`, um corpo vazio, uma mensagem de erro, ou uma resposta sobre o assunto errado. Não verifica:
  - se `body.answer` contém o prazo correto (7 dias úteis);
  - se `body.source_document` está presente e aponta para `POL-001` — requisito de produto de que **toda resposta seja rastreável à fonte** (VC-02 do plano de testes em [exercicio2-2.md](exercicio2-2.md));
  - a exceção de carga perigosa que deveria acompanhar o prazo.
  Além disso, não há **mock da fronteira HTTP** (Azure AI Search / Azure OpenAI). Sem msw, o teste ou chama serviços reais (custo de tokens, não-determinismo, falha no CI) ou depende de um `app` com integração não isolada.
- **Risco se "passar" com o código errado:** **alto e enganoso.** Um handler completamente quebrado — que retorne `{ status: 200, body: { erro: '...' } }`, ou a política errada, ou sem `source_document` — passaria neste teste. Ele dá **falsa segurança**: a suíte fica verde enquanto o comportamento de negócio pode estar quebrado. É um *smoke test* disfarçado de teste de comportamento.
- **Classificação:** Insuficiente. Reprovaria nos itens 3, 4, 5 e 6 do checklist de [exercicio2-3.md](exercicio2-3.md) (dados de domínio fracos, sem mock HTTP, sem assertion de comportamento, sem validação de `source_document`).

---

### Teste 2 — Dados irreais (não exercitam o domínio)

```typescript
describe('query endpoint edge cases', () => {
  it('should handle empty question', async () => {
    const res = await request(app).post('/api/query').send({ question: '' });
    expect(res.status).toBe(400);
  });
});
```

- **O que testa:** que uma pergunta vazia (`question: ''`) retorna HTTP 400. É um *edge case* de validação de input **legítimo e bem formado** — vale a pena ter.
- **O que falha em testar:** o problema não está neste teste isoladamente, e sim no que a suíte **não tem**. O bloco se chama "edge cases", mas o único caso é o de input vazio. **Nenhum teste exercita o domínio** — não há uma única pergunta real de logística (devolução, frete, SLA, carga perigosa) passando por retrieval + geração com assertion sobre a resposta. O happy path do domínio está ausente. A suíte cobre a borda trivial e deixa o miolo do produto sem cobertura.
- **Risco se "passar" com o código errado:** **médio-alto.** Toda a lógica de domínio — recuperação de chunks, citação de fonte, guardrail de carga perigosa, reconhecimento de tier inexistente — pode estar quebrada e esta suíte continua verde. Cria a **ilusão de "edge cases cobertos"** quando o caminho principal nunca foi exercitado. A cobertura de linhas pode até atingir o gate de 80% sem que uma única pergunta real tenha sido validada (o anti-padrão de "perseguir 80% com testes vazios" que o AGENTS.md proíbe explicitamente).
- **Classificação:** Incompleto. O caso de input vazio é válido; o que falta é o happy path de domínio com dados reais de fixture (`queries.prazoDeVolucao`, `queries.devolucaoClasseAntt3`, etc.).

---

### Teste 3 — Mock que mascara bug

```typescript
describe('feedback endpoint', () => {
  it('should save feedback', async () => {
    const mockCreate = jest.fn().mockResolvedValue({ id: '123' });
    const res = await request(app).post('/api/feedback').send({
      queryId: 'q1', rating: 5, comment: 'great'
    });
    expect(res.status).toBe(200);
    expect(mockCreate).toHaveBeenCalled();
  });
});
```

- **O que testa:** aparentemente, que `POST /api/feedback` retorna 200 e que a persistência foi chamada.
- **O que falha em testar (e por que o mock é perigoso):**
  1. **O mock não está conectado ao handler.** `mockCreate` é uma `jest.fn()` solta — não há `vi.mock`/injeção que substitua o `container.items.create` real do handler. Logo, `expect(mockCreate).toHaveBeenCalled()` ou falha (o handler nunca chama *este* objeto) ou só passa se houver um mock global não mostrado. Em qualquer cenário, a assertion **não prova** que o feedback foi persistido pelo código sob teste.
  2. **O mock sempre resolve sucesso** (`mockResolvedValue({ id: '123' })`) e o teste só envia um payload **válido**. Não há nenhum caso de input **inválido** (sem `queryId`, `rating` fora de faixa, `comment` ausente). Como descrito na armadilha do exercício: o mock é tão permissivo que **o teste passaria mesmo se a validação de input não existisse**.
  3. **Não verifica o que foi gravado.** Não assegura que o objeto persistido tem o shape correto, nem — criticamente — que **dados pessoais não são logados/gravados indevidamente**. O handler de feedback gerado por IA (revisado pelo Dev no exercício 3.2) loga `attendantEmail` em `console.log`; este teste não capturaria essa violação.
- **Risco se "passar" com o código errado:** **alto — é o teste mais perigoso dos três.** Ele dá selo verde a um handler que pode não ter **nenhuma** validação de input (exatamente o `feedback-handler.ts` com `as any` e sem Zod) e que pode estar logando PII. O teste afirma "should save feedback" e na prática não garante nem a persistência, nem a validação, nem a ausência de vazamento de dado pessoal.
- **Classificação:** Perigoso. Falsa segurança em um endpoint que escreve dados — o pior lugar para um teste complacente.

---

### Ponto de atenção — `jest` vs Vitest (inconsistência com o AGENTS.md)

O Teste 3 usa `jest.fn()`. O projeto, porém, padroniza **Vitest** como test runner — decisão de stack registrada no AGENTS.md e na estrutura do repositório (Anexo C, `vitest.config.ts`), e reafirmada nos Testing Standards de [exercicio2-1.md](exercicio2-1.md). Em Vitest, o equivalente é `vi.fn()` / `vi.mock()`. Usar a API do jest indica que o teste foi gerado **sem o contexto do projeto** — um sinal clássico de output de IA que "parece certo" isoladamente mas ignora as convenções do repositório. Dependendo da configuração, `jest` sequer está disponível como global, e o teste quebraria por `ReferenceError` antes de testar qualquer coisa.

Observação secundária de contexto: os três testes usam `request(app)` (supertest), enquanto o padrão do projeto (AGENTS.md, [exercicio2-3.md](exercicio2-3.md)) é **chamar o handler diretamente** e mockar a fronteira HTTP (Azure) com **msw**. A divergência de abordagem de teste, somada ao uso de jest, reforça que estes testes não seguiram a skill `create-integration-test`.

---

### Resumo da análise própria

| Teste | Diagnóstico | Risco | Principais itens reprovados no checklist (Ex. 2.3) |
|-------|-------------|-------|-----------------------------------------------------|
| 1 | Assertions vagas (`toBeDefined`) | Alto — verifica que existe algo, não que está certo | 3 (dados), 4 (mock HTTP), 5 (assertion de comportamento), 6 (`source_document`) |
| 2 | Só edge case trivial; sem happy path de domínio | Médio-alto — ilusão de cobertura | 3 (sem pergunta real de logística), 5 |
| 3 | Mock permissivo desconectado do handler | Alto — selo verde sem validação nem proteção de PII | 4, 5, 7 (efeitos colaterais/persistência), + `jest` em vez de Vitest |

---

## Parte 2 — Segunda revisão com o Claude e comparação

O Claude foi acionado **após** a análise própria estar fechada, recebendo os 3 testes, o AGENTS.md (resumo de Testing Standards) e o Anexo C, com instrução de revisar de forma independente.

### Concordâncias

- **Teste 1:** ambos apontaram `toBeDefined()` como assertion que não prova comportamento e a ausência de verificação de `source_document` e de mock HTTP.
- **Teste 3:** ambos identificaram que o mock permissivo deixa o teste passar sem validação de input e que ele é o mais arriscado dos três.
- **Teste 2:** ambos reconheceram que o caso de input vazio é válido, mas que falta o happy path de domínio.

### Divergências (e o que cada um acrescentou)

| Item | Análise própria | Acréscimo / divergência do Claude | Resolução |
|------|-----------------|-----------------------------------|-----------|
| `jest` vs Vitest | Identificado na análise própria como ponto de atenção principal | O Claude **não destacou** a inconsistência de framework na primeira passada — focou nas assertions. Só a mencionou quando questionado sobre o Anexo C. | Mantido como ponto de QA: é uma falha de **atenção ao contexto do projeto**, não só ao código isolado — exatamente o tipo de problema que a revisão humana pega e a IA, sem o contexto carregado, deixa passar. |
| Teste 3 — vazamento de PII | Levantei o risco de o teste não capturar o log de `attendantEmail` (cruzando com o handler de feedback do Dev 3.2) | O Claude inicialmente tratou o Teste 3 só como "falta cobertura de input inválido", sem conectar à violação de PII | Acréscimo da revisão humana incorporado: o teste deveria também asserir que dados pessoais não são logados. Convergência após a observação. |
| Teste 2 — gate de cobertura | Liguei a falta de happy path ao risco de atingir 80% de cobertura sem testar o domínio | O Claude acrescentou um ponto válido que eu não havia explicitado: sugerir **casos negativos adicionais** (pergunta sobre tier inexistente, frete sem destino) para cobrir os reconhecimentos de lacuna | Incorporado: a suíte de "edge cases" deveria incluir os reconhecimentos de lacuna (tier Platinum, destino faltante), não só input vazio. |

### Leitura da comparação

A análise humana e a do Claude **convergiram nos três diagnósticos centrais** (assertion vaga, ausência de domínio, mock perigoso). As contribuições foram complementares: a revisão humana trouxe a **atenção ao contexto do projeto** (jest/Vitest, abordagem de teste, vazamento de PII cruzando com o handler real); o Claude trouxe **ampliação de cobertura** (casos negativos adicionais). Não houve "concordamos em tudo" — o ponto da inconsistência de framework foi detectado primeiro pela revisão humana e só confirmado pelo Claude sob provocação, o que reforça por que a revisão de QA não pode ser delegada inteiramente à IA que gerou (ou revisa) o código sem o contexto do repositório carregado.

---

## Parte 3 — Teste 1 reescrito

A versão abaixo segue os Testing Standards do projeto (AGENTS.md, [exercicio2-1.md](exercicio2-1.md)) e a skill `create-integration-test` ([exercicio2-3.md](exercicio2-3.md)): Vitest, msw com `onUnhandledRequest: 'error'`, nomenclatura `describe`/`it` de comportamento, padrão AAA, dados de domínio via fixtures, e — o ponto central — **assertions que verificam o conteúdo da resposta e a citação de fonte**, não apenas que algo foi retornado.

```typescript
import { describe, it, expect, beforeAll, afterEach, afterAll } from 'vitest';
import { setupServer } from 'msw/node';
import { http, HttpResponse } from 'msw';
import { queryHandler } from '../../src/functions/query/handler';
import { chunks, queries } from '@tests/fixtures';
import { buildQueryRequest, buildSearchResponse, buildCompletionResponse } from '@tests/factories';

const server = setupServer();

beforeAll(() => server.listen({ onUnhandledRequest: 'error' })); // request não mockada = falha
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('QueryHandler', () => {
  describe('return policy query', () => {
    it('should answer with the correct deadline and cite POL-001 when asked about the return policy', async () => {
      // Arrange
      const request = buildQueryRequest({ question: queries.prazoDeVolucao });
      server.use(
        http.post('https://novatech-search.search.windows.net/indexes/*/docs/search', () =>
          HttpResponse.json(buildSearchResponse([chunks.pol001A]))
        ),
        http.post('https://novatech-openai.openai.azure.com/*/chat/completions', () =>
          HttpResponse.json(buildCompletionResponse(
            'O prazo de devolução é de 7 dias úteis após o recebimento, exceto para cargas perigosas classes 1 a 6 da ANTT.'
          ))
        )
      );

      // Act
      const response = await queryHandler(request);
      const body = JSON.parse(response.body);

      // Assert
      expect(response.statusCode).toBe(200);
      expect(body.answer).toContain('7 dias úteis');        // conteúdo CERTO, não só "existe"
      expect(body.answer).toMatch(/carga[s]? perigosa[s]?/i); // a exceção relevante está presente
      expect(body.source_document).toBe('POL-001');          // resposta rastreável à fonte
    });
  });
});
```

### O que mudou — antes × depois

| # | Problema no Teste 1 original | Correção aplicada | Motivo |
|---|------------------------------|-------------------|--------|
| 1 | `expect(res.body).toBeDefined()` | `expect(body.answer).toContain('7 dias úteis')` + `expect(body.source_document).toBe('POL-001')` | `toBeDefined()` passa para qualquer objeto não-nulo; as novas assertions provam que a resposta está **factualmente correta** e é **rastreável** |
| 2 | Verifica só o prazo geral (e nem isso) | Acrescenta `toMatch(/carga[s]? perigosa[s]?/i)` | A exceção de carga perigosa é parte do comportamento correto da política de devolução (POL-001 §3.2) — sem ela, a resposta estaria incompleta |
| 3 | Sem `source_document` | Assertion explícita em `body.source_document === 'POL-001'` | Requisito de produto: 100% das respostas citam fonte válida (VC-02 de [exercicio2-2.md](exercicio2-2.md)) |
| 4 | Sem mock de HTTP — chama Azure real / não-determinístico | `server.use(...)` com msw + `onUnhandledRequest: 'error'` | Isola a fronteira (Azure AI Search / OpenAI); request não mockada falha o teste em vez de vazar para a rede |
| 5 | `request(app).post(...)` (supertest) | Chamada direta a `queryHandler(request)` | Alinha à abordagem padrão do projeto (AGENTS.md) e remove dependência de subir o app |
| 6 | `question: 'prazo devolução'` (string solta) | `buildQueryRequest({ question: queries.prazoDeVolucao })` | Factory tipada + fixture de domínio: dados reais de logística, manuteníveis quando o schema mudar |
| 7 | `describe('query endpoint')` / `it('should return a response')` | `describe('QueryHandler')` / `it('should answer with the correct deadline and cite POL-001 ...')` | Nome de comportamento de negócio verificável, legível no relatório de CI |

> O teste reescrito **falha** se o handler retornar a política errada, omitir a exceção de carga perigosa, ou não preencher `source_document` — exatamente o que o original deixava passar.

---

## Registro de Uso da Ferramenta (Claude)

### Etapa 1 — Análise própria (sem ferramenta)

Os três testes foram revisados manualmente contra o AGENTS.md, os Testing Standards de [exercicio2-1.md](exercicio2-1.md) e o checklist de [exercicio2-3.md](exercicio2-3.md), fechando o diagnóstico da Parte 1 antes de qualquer uso de IA. A inconsistência `jest`/Vitest e o risco de vazamento de PII no Teste 3 foram identificados nesta etapa.

### Etapa 2 — Claude (segundo revisor)

**Prompt usado:**
```
Contexto: projeto NovaTech Assistant. Stack de testes (AGENTS.md / Anexo C): Vitest +
msw, chamada direta ao handler, assertion obrigatória de source_document no query
endpoint, dados de domínio de logística (nunca "test"/"foo").

Revise os 3 testes de integração abaixo de forma INDEPENDENTE (não vou te mostrar minha
revisão ainda). Para cada um: o que ele realmente testa, o que falha em testar, e o risco
se ele "passar" mas o código estiver errado. [3 testes colados]
```

**Prompt de refino (após comparar com a análise própria):**
```
Dois pontos que minha revisão pegou e a sua não destacou: (1) o Teste 3 usa jest.fn(),
mas o projeto usa Vitest (vi.fn) — é uma inconsistência com o AGENTS.md/Anexo C; (2) o
handler de feedback loga attendantEmail (PII), e o Teste 3 não capturaria isso. Reavalie
o Teste 3 considerando validação de input ausente E vazamento de dado pessoal.
```
O Claude confirmou ambos os pontos e acrescentou a sugestão de casos negativos de domínio para o Teste 2 (tier inexistente, frete sem destino). As divergências estão registradas na Parte 2.

### Etapa 3 — Reescrita do Teste 1

A reescrita seguiu o template e o checklist da skill `create-integration-test` ([exercicio2-3.md](exercicio2-3.md)) — Vitest, msw, AAA, fixtures de domínio e assertions de conteúdo + `source_document` — garantindo que o teste reescrito passe nos 8 itens do checklist de revisão de QA do projeto.
