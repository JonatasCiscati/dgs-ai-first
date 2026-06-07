# Exercício 1.2

### Parte 1 — Avaliação manual das 5 respostas

As respostas foram avaliadas individualmente antes da elaboração da rubrica, usando como fonte de verdade a documentação da NovaTech (Anexo A): POL-001, PROC-042-v2, SLA-2024.

---

**Resposta 1**
- **Pergunta:** "Qual o prazo de devolução?"
- **Resposta do assistente:** "O prazo é de 7 dias úteis, exceto para cargas perigosas classes 1 a 6 da ANTT."
- **Fonte citada:** POL-001, seção 3.2
- **Avaliação: CORRETA**
- **Justificativa:** O prazo de 7 dias úteis está correto conforme POL-001 seção 3.1 (Chunk POL-001-A). A exceção para cargas perigosas classes 1 a 6 da ANTT está correta conforme POL-001 seção 3.2 (Chunk POL-001-B). A fonte citada é apropriada. A resposta é factualmente precisa e cobre os dois elementos essenciais da política de devolução.

---

**Resposta 2**
- **Pergunta:** "Quanto custa frete para 600kg para Manaus?"
- **Resposta do assistente:** "O frete especial para cargas acima de 500kg para a região Norte tem multiplicador de 1.8 sobre o valor base."
- **Fonte citada:** PROC-042-v2, seção 2
- **Avaliação: PARCIALMENTE CORRETA**
- **Justificativa:** O multiplicador 1.8 para a região Norte está correto conforme PROC-042-v2, seção 2.1 (Chunk PROC-042v2-B), e a fonte citada está correta. No entanto, a resposta omite dois elementos relevantes: (1) a existência de uma versão anterior (PROC-042-v1) com multiplicador 1.6 para a mesma região, que pode ainda ser aplicável a chamados abertos antes de 01/12/2023 conforme a cláusula transitória (Chunk PROC-042v2-E); (2) o fator de peso, que para 600kg (faixa 500-1000kg) é 1.0 — a resposta cita apenas o multiplicador regional, sem compor o cálculo completo. A informação fornecida não é incorreta, mas está incompleta para um atendente que precisa calcular o valor final.

---

**Resposta 3**
- **Pergunta:** "Qual o SLA do cliente Platinum?"
- **Resposta do assistente:** "O cliente Platinum tem resposta em até 1h e resolução em até 12h."
- **Fonte citada:** SLA-2024
- **Avaliação: INCORRETA — Alucinação**
- **Justificativa:** O tier "Platinum" não existe na NovaTech. O documento SLA-2024, seção 1 (Chunk SLA-2024-A), afirma explicitamente: "A NovaTech classifica seus clientes em 3 (três) tiers: Gold, Silver e Standard. Não existem outros tiers além dos três listados." Os valores de SLA apresentados (resposta 1h, resolução 12h) foram gerados sem qualquer base documental — os SLAs reais documentados são: Silver: resposta 4h / resolução 48h (Chunk SLA-2024-B) e Gold: resposta 2h / resolução 24h (Chunk SLA-2024-B). A resposta cita a SLA-2024 como fonte, mas o conteúdo contradiz diretamente o que o documento registra. Trata-se de alucinação com citação de fonte incorreta — o tipo de falha mais perigoso porque aparenta credibilidade.

---

**Resposta 4**
- **Pergunta:** "Posso devolver carga perigosa?"
- **Resposta do assistente:** "Sim, cargas perigosas podem ser devolvidas em até 7 dias úteis."
- **Fonte citada:** POL-001, seção 3.2
- **Avaliação: INCORRETA — Inversão de regra**
- **Justificativa:** O POL-001, seção 3.2 (Chunk POL-001-B), estabelece exatamente o oposto do que foi respondido: cargas perigosas das classes 1 a 6 da ANTT **não são elegíveis** para devolução pelo processo padrão. A fonte citada (POL-001, seção 3.2) é justamente a seção que contém a exceção proibitiva. O assistente inverteu o sentido da regra — confundiu a seção de exceções com uma permissão, e aplicou o prazo geral de 7 dias (da seção 3.1) a uma categoria explicitamente excluída. Esta é uma falha de interpretação de alta severidade: um atendente que confirme devolução de carga perigosa com base nesta resposta pode gerar passivo regulatório para a NovaTech.

---

**Resposta 5**
- **Pergunta:** "Qual o multiplicador de frete para o Sudeste?"
- **Resposta do assistente:** "O multiplicador regional para o Sudeste é 1.1."
- **Fonte citada:** PROC-042-v2, seção 2
- **Avaliação: PARCIALMENTE CORRETA**
- **Justificativa:** O valor 1.1 está correto para a versão vigente da PROC-042 (v2, novembro/2023), conforme o Chunk PROC-042v2-B. A fonte é correta. Contudo, assim como na Resposta 2, a resposta omite a existência da versão anterior que registrava o multiplicador do Sudeste como 1.0 (Chunk PROC-042-B). Para chamados abertos antes de 01/12/2023, o valor aplicável pode ser 1.0. A ausência desta contextualização pode induzir o atendente a aplicar o multiplicador incorreto em situações de transição. A resposta está correta para chamados novos, mas incompleta para o universo de chamados em andamento.

---

### Parte 2 — Rubrica de avaliação

A rubrica abaixo foi elaborada para avaliar respostas do assistente de forma objetiva e reproduzível. O objetivo é que dois avaliadores independentes cheguem a pontuações iguais ou com variação máxima de 1 ponto por dimensão.

#### Dimensão 1 — Precisão Factual

Avalia se a informação fornecida está correta em relação à documentação oficial da NovaTech.

| Nível | Descrição |
|-------|-----------|
| 1 | A informação está incorreta, invertida ou foi inventada sem respaldo documental. Inclui alucinações (ex: tier inexistente, valores fabricados) e inversões de regra (ex: afirmar que é permitido o que o documento proíbe). |
| 2 | A informação central está correta, mas há omissões relevantes que podem induzir decisões erradas em casos específicos (ex: omitir a existência de versão anterior com valores diferentes, omitir fator de peso ao citar apenas multiplicador regional). |
| 3 | Todas as informações são corretas, completas para o contexto da pergunta e rastreáveis na documentação. Não há omissões que possam gerar interpretações incorretas. |

#### Dimensão 2 — Citação de Fonte

Avalia se a fonte citada existe, é correta e é específica o suficiente para rastreabilidade.

| Nível | Descrição |
|-------|-----------|
| 1 | Sem citação de fonte, ou fonte citada não contém a informação fornecida (ex: citar POL-001 seção 3.2 para confirmar devolução de carga perigosa, quando essa seção proíbe a devolução). |
| 2 | Fonte citada existe e contém informação relacionada, mas a referência é imprecisa (ex: cita o documento sem indicar seção, ou cita seção que contém informação parcialmente correta). |
| 3 | Fonte citada existe, contém exatamente a informação fornecida, e a referência é específica (documento + seção identificados). |

#### Dimensão 3 — Aderência aos Guardrails

Avalia se a resposta respeita os quatro guardrails definidos: (1) sempre citar fonte; (2) nunca inventar prazos ou valores; (3) quando não encontrar resposta, dizer explicitamente; (4) responder em português formal.

| Nível | Descrição |
|-------|-----------|
| 1 | Viola um ou mais guardrails de forma clara (ex: inventa valores, não cita fonte, responde em idioma diferente do português, ou afirma com confiança sobre algo que deveria ser declarado como desconhecido). |
| 2 | Guardrails de forma e idioma respeitados, mas há violação parcial do guardrail de não inventar (ex: omite informação crítica ao invés de declarar que não a encontrou, sem inventar diretamente). |
| 3 | Todos os quatro guardrails são respeitados na resposta. |

#### Dimensão 4 — Completude

Avalia se a resposta cobre os elementos necessários para que o atendente tome uma decisão correta e completa.

| Nível | Descrição |
|-------|-----------|
| 1 | Informação crítica está ausente e a lacuna pode levar a uma decisão errada (ex: informar prazo de devolução sem mencionar a exceção para cargas perigosas). |
| 2 | A resposta é adequada para a maioria dos casos, mas está incompleta em aspectos que podem impactar situações específicas (ex: citar multiplicador regional correto sem mencionar fator de peso ou versão anterior). |
| 3 | A resposta cobre todos os elementos necessários para o contexto da pergunta, incluindo ressalvas e exceções relevantes. |

---

### Parte 3 — Template de avaliação reutilizável

O template abaixo pode ser aplicado a qualquer lote de respostas do assistente. Cada resposta recebe uma linha na tabela.

**Instruções de uso:**
1. Preencher ID, Pergunta e Resposta avaliada.
2. Atribuir nota 1, 2 ou 3 para cada dimensão, consultando a rubrica.
3. Somar as quatro notas para obter a Pontuação Total (mínimo 4, máximo 12).
4. Aplicar a classificação conforme escala: 10-12 = Aprovada | 7-9 = Aprovada com ressalva | 4-6 = Reprovada.
5. Registrar na coluna Evidência o trecho do documento que justifica a avaliação.

| ID | Pergunta | Resposta (resumo) | D1 Precisão Factual (1-3) | D2 Citação de Fonte (1-3) | D3 Aderência Guardrails (1-3) | D4 Completude (1-3) | Pontuação Total (4-12) | Classificação | Evidência / Justificativa |
|----|----------|-------------------|--------------------------|--------------------------|-------------------------------|---------------------|------------------------|---------------|--------------------------|
|    |          |                   |                          |                          |                               |                     |                        |               |                          |
|    |          |                   |                          |                          |                               |                     |                        |               |                          |

**Escala de classificação:**

| Pontuação | Classificação | Ação recomendada |
|-----------|---------------|------------------|
| 10 a 12   | Aprovada | Nenhuma ação imediata necessária |
| 7 a 9     | Aprovada com ressalva | Revisar com atendente; monitorar padrão |
| 4 a 6     | Reprovada | Bloquear resposta; investigar causa raiz no pipeline |

---

### Parte 4 — Pontuações aplicadas às 5 respostas

| ID | Pergunta (resumo) | D1 Precisão | D2 Citação | D3 Guardrails | D4 Completude | Total | Classificação |
|----|-------------------|-------------|------------|---------------|---------------|-------|---------------|
| 1  | Prazo de devolução | 3 | 3 | 3 | 3 | **12** | Aprovada |
| 2  | Frete 600kg para Manaus | 3 | 3 | 3 | 2 | **11** | Aprovada com ressalva |
| 3  | SLA cliente Platinum | 1 | 1 | 1 | 1 | **4** | Reprovada |
| 4  | Devolução de carga perigosa | 1 | 1 | 1 | 1 | **4** | Reprovada |
| 5  | Multiplicador Sudeste | 3 | 3 | 3 | 2 | **11** | Aprovada com ressalva |

**Detalhamento das notas:**

**Resposta 1** — D1: 3 (prazo e exceção corretos); D2: 3 (POL-001 seção 3.2 é a seção de exceções, que complementa a 3.1 — citação adequada); D3: 3 (cita fonte, não inventa, em português formal); D4: 3 (responde completamente a pergunta sobre prazo incluindo a exceção principal).

**Resposta 2** — D1: 3 (multiplicador 1.8 correto por v2); D2: 3 (PROC-042-v2 seção 2 existe e contém o multiplicador); D3: 3 (cita fonte, não inventa, em português); D4: 2 (omite fator de peso e ambiguidade com v1 — informação incompleta para cálculo final).

**Resposta 3** — D1: 1 (Platinum não existe; valores de SLA inventados); D2: 1 (SLA-2024 citado contradiz diretamente o conteúdo da resposta); D3: 1 (inventa prazos e valores, violando guardrail 2); D4: 1 (não há completude possível quando a premissa é falsa).

**Resposta 4** — D1: 1 (inverte a regra do POL-001-B; carga perigosa NÃO pode ser devolvida); D2: 1 (POL-001 seção 3.2 é a seção de exceções proibitivas, citada incorretamente para confirmar o oposto); D3: 1 (afirma com confiança o contrário do que o documento estabelece — viola guardrail 2); D4: 1 (ausência de informação crítica sobre procedimento correto: ramal 4500).

**Resposta 5** — D1: 3 (1.1 correto para v2); D2: 3 (PROC-042-v2 seção 2 existe e contém o multiplicador Sudeste); D3: 3 (cita fonte, não inventa, em português); D4: 2 (omite que v1 registrava 1.0 para Sudeste — relevante para chamados em transição).
