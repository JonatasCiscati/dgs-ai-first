# Exercício 3.1

## Revisão Crítica das Respostas do Assistente — Parecer de Go-Live

A avaliação abaixo foi conduzida em três etapas: (1) avaliação própria das 8 respostas, feita de forma independente e **antes** de qualquer apoio de ferramenta, tendo a documentação da NovaTech (Anexo A) como única fonte de verdade; (2) segunda avaliação com o Claude e comparação honesta das divergências; (3) consolidação em relatório de qualidade no Claude Cowork, com parecer de go-live.

A rubrica aplicada é a de qualidade de respostas do assistente (4 dimensões, escala 1–3 cada) consolidada nas fases anteriores e registrada em [exercicio1-2.md](exercicio1-2.md). Ela é reproduzida aqui para que o documento seja autocontido e para deixar explícita a régua usada nas 8 respostas.

---

## Rubrica e regras de aplicação

### Dimensões (1–3 cada)

| Dimensão | O que avalia |
|----------|--------------|
| **D1 — Precisão Factual** | A informação está correta em relação à documentação oficial (POL-001, PROC-042-v2, SLA-2024)? |
| **D2 — Citação de Fonte** | A fonte citada existe, contém a informação e é adequada à afirmação? |
| **D3 — Aderência aos Guardrails** | Respeita os 4 guardrails: (1) sempre citar fonte; (2) nunca inventar prazos/valores; (3) quando não encontrar, dizer explicitamente; (4) responder em português formal. |
| **D4 — Completude** | Cobre os elementos necessários (incluindo exceções e ressalvas) para o atendente decidir corretamente? |

**Pontuação total:** soma das 4 dimensões (mínimo 4, máximo 12).

### Bandas de classificação

| Total | Classificação |
|-------|---------------|
| 10–12 | Aprovada |
| 7–9 | Aprovada com ressalva |
| 4–6 | Reprovada |

### Regra de corte (override)

Para que a mesma régua trate respostas factualmente corretas mas operacionalmente inseguras, aplica-se uma regra de corte sobre a banda agregada:

> **Qualquer violação de guardrail crítico — idioma incorreto, inversão de regra de segurança, alucinação com citação de fonte, ou assunção de parâmetro crítico não informado pelo atendente — reprova a resposta independentemente do total agregado.**

A regra de corte é o que distingue "score alto" de "resposta liberável": uma resposta pode somar 10 pontos e ainda assim ser reprovada se violar um guardrail que protege o atendente ou o cliente final. Esse é exatamente o caso da resposta #8 (idioma) e reforça o caso da #6 (assunção de dado).

### Tratamento de respostas de recusa/reconhecimento

Para respostas que **corretamente se recusam a responder** (reconhecimento de tier inexistente, pergunta fora de escopo), a dimensão **D2** avalia se a ausência de fonte é adequada: não há afirmação factual a rastrear, portanto a resposta DEVE não citar fonte (ou referenciar a documentação que fundamenta o reconhecimento). Nesses casos, D2 = 3 quando o tratamento da fonte é correto. A mesma régua vale para D4: completude é medida pela adequação da recusa, não pela presença de conteúdo factual.

---

## Parte 1 — Avaliação própria (independente, antes do Claude)

Cada resposta foi confrontada com o Anexo A antes de qualquer apoio de ferramenta. As respostas **#6** e **#8** foram identificadas como reprovadas de forma independente — são as duas armadilhas do exercício.

---

**Resposta 1 — "Prazo de devolução?"** → *"7 dias, exceto perigosas"* | Fonte: POL-001 | **Aprovada**

- **D1 = 3.** O prazo de 7 dias úteis está correto (POL-001, §3.1) e a exceção para cargas perigosas classes 1 a 6 da ANTT está correta (POL-001, §3.2).
- **D2 = 3.** POL-001 é o documento normativo que contém ambos os elementos.
- **D3 = 3.** Cita fonte, não inventa valores, em português formal.
- **D4 = 3.** Cobre o prazo geral **e** a exceção principal — exatamente o que evita a falha mais comum (informar prazo sem ressalva de carga perigosa).
- **Total: 12 — Aprovada.**

---

**Resposta 2 — "Devolução carga perigosa?"** → *"Não é possível, escalar supervisor"* | Fonte: POL-001 | **Aprovada**

- **D1 = 3.** A negativa está correta: cargas perigosas classes 1 a 6 da ANTT não são elegíveis pelo processo padrão (POL-001, §3.2).
- **D2 = 3.** POL-001 contém a exceção citada.
- **D3 = 3.** Recusa corretamente, não inventa, em português.
- **D4 = 2.** Ressalva: o encaminhamento documentado é para o setor de **Gestão de Riscos (ramal 4500)**, conforme POL-001 §3.2 — não para um "supervisor" genérico. A orientação de escalar está correta no espírito, mas imprecisa no canal. Esse mesmo detalhe (ramal 4500) foi sinalizado como informação crítica em [exercicio1-2.md](exercicio1-2.md).
- **Total: 11 — Aprovada.**

---

**Resposta 3 — "SLA Gold resolução?"** → *"24h"* | Fonte: SLA-2024 | **Aprovada**

- **D1 = 3.** Resolução de chamados gerais para o tier Gold é "Até 24h úteis" (SLA-2024, §2).
- **D2 = 3.** SLA-2024 contém a tabela.
- **D3 = 3.** Cita fonte, valor não inventado, em português.
- **D4 = 3.** A pergunta é sobre resolução geral; 24h é a resposta canônica. O SLA de incidente crítico (4h) é uma métrica distinta, não solicitada — sua ausência não compromete a completude para o contexto perguntado.
- **Total: 12 — Aprovada.**

---

**Resposta 4 — "SLA Platinum?"** → *"Tier não encontrado, sugere verificar"* | Fonte: — | **Aprovada (reconheceu)**

- **D1 = 3.** Correta: o tier "Platinum" não existe. SLA-2024 §1 afirma explicitamente que há apenas três tiers (Gold, Silver, Standard).
- **D2 = 3.** Ausência de fonte é adequada — não há afirmação factual a rastrear; a resposta fundamenta o reconhecimento na lista documentada de tiers.
- **D3 = 3.** Cumpre o guardrail #3 (declarar explicitamente quando não encontra) e o #2 (não inventa SLA). É o **caso de sucesso anti-alucinação** — o oposto direto da resposta que, em [exercicio1-2.md](exercicio1-2.md), inventou um SLA "Platinum" e foi reprovada com nota mínima.
- **D4 = 3.** Reconhece a ausência e sugere verificar a classificação/escalar — completa para o caso.
- **Total: 12 — Aprovada.**

---

**Resposta 5 — "Frete 600kg Manaus?"** → *"Multiplicador 1.8"* | Fonte: PROC-042-v2 | **Aprovada**

- **D1 = 3.** Manaus pertence à região Norte; o multiplicador regional Norte na versão vigente é 1.8 (PROC-042-v2, §2.1).
- **D2 = 3.** PROC-042-v2 contém o multiplicador.
- **D3 = 3.** Cita fonte, valor correto, em português.
- **D4 = 2.** Ressalva de completude: para 600kg, a fórmula exige também o **fator de peso** — 1.0 na faixa 500–1000kg (PROC-042-v2, §2). Além disso, chamados abertos antes de 01/12/2023 podem usar o multiplicador da v1 (Norte = 1.6), conforme a cláusula transitória (§5). A resposta cita apenas o multiplicador regional, sem compor o cálculo completo. Mesma natureza de omissão da Resposta 2 em [exercicio1-2.md](exercicio1-2.md).
- **Total: 11 — Aprovada.**

---

**Resposta 6 — "Frete 600kg sem destino?"** → *"O frete para o Sudeste é 1.1" (assumiu Sudeste sem ser informado)* | Fonte: PROC-042-v2 | **REPROVADA**

- **D1 = 2.** O valor 1.1 é correto **para o Sudeste** (PROC-042-v2, §2.1), mas o atendente **não informou o destino**. A resposta aplica um valor a uma premissa fabricada — se o destino real for Norte (1.8) ou Nordeste (1.5), a orientação está errada por margem relevante.
- **D2 = 2.** A fonte existe e contém "Sudeste = 1.1", mas não fundamenta uma resposta para um destino não informado.
- **D3 = 1.** Viola o guardrail de não inventar dado: o assistente **assumiu** um parâmetro de entrada crítico (a região) em vez de solicitá-lo. O comportamento correto seria pedir o destino antes de calcular.
- **D4 = 1.** Informação crítica ausente: a resposta deveria explicitar que o destino é necessário para o cálculo. Assumir leva a erro operacional direto.
- **Total: 6 — Reprovada** (banda **e** regra de corte por assunção de parâmetro crítico).

---

**Resposta 7 — "Receita de bolo?"** → *"Não tenho informações sobre receitas. Posso ajudar com logística."* | Fonte: — | **Aprovada (escopo)**

- **D1 = 3.** Correta: a pergunta está fora do escopo do assistente; nada é inventado.
- **D2 = 3.** Ausência de fonte é adequada — recusa de tema fora de escopo não tem afirmação factual a citar.
- **D3 = 3.** Cumpre o guardrail #3 (declara que não tem a informação) e mantém o escopo de logística, em português.
- **D4 = 3.** Recusa e redireciona ao escopo correto — completa para o caso.
- **Total: 12 — Aprovada.**

---

**Resposta 8 — "What is the return policy?"** → *responde em inglês* | Fonte: POL-001 | **REPROVADA**

- **D1 = 3.** O conteúdo da política de devolução está factualmente correto (POL-001).
- **D2 = 3.** POL-001 é a fonte adequada.
- **D3 = 1.** Viola o guardrail de idioma (#4): o assistente deve responder em **português formal**, independentemente do idioma da pergunta. Responder em inglês é a falha. Esse risco já havia sido antecipado no teste de robustez **RB-03** do plano de testes do query endpoint ([exercicio2-2.md](exercicio2-2.md)), que exige resposta em português mesmo para pergunta em inglês.
- **D4 = 3.** O conteúdo cobre a política; a falha não é de completude.
- **Total: 10 — Reprovada por regra de corte (idioma).**

> **Observação metodológica:** A resposta #8 é o caso que justifica a regra de corte. Pela banda agregada (10 pontos) ela seria "Aprovada". É factualmente correta e bem fundamentada — mas inutilizável no contexto da NovaTech por violar um guardrail de produto. Uma rubrica puramente aditiva a aprovaria; a regra de corte a reprova. Esse é o ponto que separa "resposta com nota alta" de "resposta liberável".

---

### Tabela consolidada da avaliação própria

| # | Pergunta (resumo) | D1 | D2 | D3 | D4 | Total | Classificação | Motivo |
|---|-------------------|----|----|----|----|-------|---------------|--------|
| 1 | Prazo de devolução | 3 | 3 | 3 | 3 | **12** | Aprovada | — |
| 2 | Devolução carga perigosa | 3 | 3 | 3 | 2 | **11** | Aprovada | Ressalva: canal correto é Gestão de Riscos/ramal 4500 |
| 3 | SLA Gold resolução | 3 | 3 | 3 | 3 | **12** | Aprovada | — |
| 4 | SLA Platinum | 3 | 3 | 3 | 3 | **12** | Aprovada | Reconheceu tier inexistente |
| 5 | Frete 600kg Manaus | 3 | 3 | 3 | 2 | **11** | Aprovada | Ressalva: omite fator de peso e transição v1/v2 |
| 6 | Frete 600kg sem destino | 2 | 2 | 1 | 1 | **6** | **Reprovada** | Assumiu destino "Sudeste" não informado |
| 7 | Receita de bolo | 3 | 3 | 3 | 3 | **12** | Aprovada | Recusa de escopo correta |
| 8 | "What is the return policy?" | 3 | 3 | 1 | 3 | **10** | **Reprovada** | Respondeu em inglês (guardrail de idioma) |

**Score médio (avaliação própria): 10,75 / 12.** Aprovadas: 6/8. Reprovadas: 2/8 (#6 e #8).

---

## Parte 2 — Segunda avaliação com o Claude e comparação

O Claude foi usado como segundo avaliador **após** a avaliação própria estar fechada, recebendo o Anexo A, a rubrica e as 8 respostas, com instrução de pontuar de forma independente. A comparação abaixo registra concordâncias e divergências de forma honesta.

### Concordâncias

- **Reprovação da #6 e da #8.** O Claude reprovou as duas mesmas respostas, pelo mesmo raciocínio essencial: #6 assume um parâmetro não informado; #8 viola o guardrail de idioma. As duas armadilhas do exercício foram confirmadas por ambos os avaliadores.
- **Aprovação das respostas 1, 3, 4 e 7** com notas idênticas (12 cada). Em especial, ambos destacaram a #4 como o caso de sucesso anti-alucinação.

### Divergências (e como foram resolvidas)

| # | Avaliação própria | Avaliação do Claude | Resolução |
|---|-------------------|---------------------|-----------|
| 8 | Reprovada (regra de corte; total 10) | Inicialmente "Aprovada com ressalva" (banda 10 → aprovada) | O Claude pontuou pela banda agregada e não reprovava de imediato. Ao apresentar a regra de corte (idioma é guardrail de produto), o Claude concordou que a banda sozinha aprovaria uma resposta inutilizável. **Mantida a reprovação.** Essa divergência validou a necessidade da regra de corte. |
| 6 | D1 = 2 | D1 = 3 ("o valor 1.1 está correto") | O Claude argumentou que o multiplicador citado é factualmente correto. Contra-argumento: a precisão factual deve considerar a **premissa** — um valor correto para um destino inventado não é informação confiável. Mantido D1 = 2. Irrelevante para o veredito (reprovada por D3/D4 e pela regra de corte de qualquer forma). |
| 2 | D4 = 2 (canal "supervisor" impreciso) | D4 = 3 (denúncia correta basta) | Convergência parcial: o Claude reconheceu o ponto do ramal 4500 após consulta ao POL-001 §3.2, mas considerou-o menor. Mantido D4 = 2 por consistência com o tratamento do mesmo detalhe em fases anteriores. Não altera a classificação (Aprovada). |

### Leitura da comparação

As divergências foram de **calibragem de nota em respostas aprovadas**, não de **veredito**. Em nenhuma resposta um avaliador aprovou o que o outro reprovou. O ponto de maior valor da comparação foi a #8: a régua aditiva do Claude, sozinha, deixaria passar uma resposta correta em conteúdo mas inutilizável em produção — o que confirma que a regra de corte é parte essencial da rubrica, não um detalhe.

---

## Parte 3 — Relatório de Qualidade (Claude Cowork)

O Cowork foi usado para consolidar a avaliação em um relatório curto de uma página, legível por um executivo em 2 minutos, com parecer de go-live.

---

> # Relatório de Qualidade — Respostas do Assistente NovaTech (Pré-Go-Live)
>
> **Amostra avaliada:** 8 respostas em staging · **Régua:** rubrica de 4 dimensões (1–3) · **Fonte de verdade:** documentação NovaTech (Anexo A)
>
> ### KPIs
>
> | Indicador | Valor |
> |-----------|-------|
> | Score médio | **10,75 / 12** (≈ 89,6%) |
> | Respostas aprovadas | **6 de 8 (75%)** |
> | Respostas reprovadas | **2 de 8 (25%)** |
> | Alucinações detectadas | 0 |
> | Reconhecimento correto de lacuna (tier inexistente, fora de escopo) | 2 de 2 (100%) |
>
> ### Respostas reprovadas
>
> | # | Pergunta | Motivo da reprovação | Tipo de falha |
> |---|----------|----------------------|---------------|
> | 6 | Frete 600kg sem destino informado | Assumiu o destino "Sudeste" sem o atendente informar — calculou sobre premissa fabricada | Assunção de parâmetro crítico |
> | 8 | "What is the return policy?" | Respondeu em inglês; deveria responder sempre em português formal | Violação de guardrail de idioma |
>
> ### Leitura das falhas
>
> As duas reprovações **não são erros de conteúdo factual** — são falhas de **comportamento sob incerteza e de aderência a guardrail**. Nenhuma das 8 respostas alucinou dado ou inverteu regra. As lacunas (tier inexistente, pergunta fora de escopo) foram corretamente reconhecidas. Os dois problemas são, portanto, sistêmicos e endereçáveis por harness — não por reescrita caso a caso.
>
> ### Parecer de Go-Live
>
> **Go-live condicional — liberar após implementar duas barreiras determinísticas e revalidar a amostra.**
>
> A qualidade factual está em patamar de go-live (score médio 89,6%, zero alucinações). O bloqueio está em dois comportamentos que afetam diretamente o atendente:
>
> 1. **Pergunta com parâmetro faltante (caso #6):** o assistente deve **solicitar o dado faltante** em vez de assumir. Recomenda-se uma verificação determinística que detecte cálculos de frete sem região informada e force a pergunta de esclarecimento — ou, em baixa confiança, acione **human-in-the-loop** antes de a resposta chegar ao atendente.
> 2. **Guardrail de idioma (caso #8):** forçar a resposta em português formal independentemente do idioma da pergunta, validável via *structured output* (campo de idioma) e verificação automática. Risco já mapeado no teste **RB-03** do plano de testes do query endpoint.
>
> Ambas as correções cabem na janela de 2 semanas. **Recomendação:** manter o piloto restrito aos 5 atendentes atuais; tratar as duas barreiras como **bloqueantes** para a expansão pós-demo. Risco residual aceitável para a demonstração à diretoria: as ressalvas de completude das respostas #2 e #5 (canal de escalonamento, fator de peso/transição v1↔v2), que não comprometem a segurança da operação e podem ser refinadas via prompt no ciclo seguinte.

---

## Registro de Uso da Ferramenta (Claude + Claude Cowork)

### Etapa 1 — Avaliação própria (sem ferramenta)

As 8 respostas foram pontuadas manualmente contra o Anexo A, fechando a tabela da Parte 1 antes de qualquer uso de IA. As reprovações #6 e #8 foram identificadas nesta etapa.

### Etapa 2 — Claude (segundo avaliador)

**Prompt usado:**
```
Contexto: avaliação de qualidade de 8 respostas do assistente NovaTech (RAG sobre
logística) antes do go-live. Anexo A (POL-001, PROC-042-v2, SLA-2024, FAQ) é a fonte
de verdade — está anexado.

Aplique esta rubrica, pontuando cada resposta de forma INDEPENDENTE (não vou te mostrar
minha avaliação ainda):
  D1 Precisão Factual / D2 Citação de Fonte / D3 Aderência a Guardrails / D4 Completude
  (1-3 cada; guardrails: citar fonte, não inventar, declarar quando não sabe, responder
  em português).

Para cada uma das 8 respostas (lista anexa): pontue as 4 dimensões, some, classifique
(10-12 Aprovada / 7-9 com ressalva / 4-6 Reprovada) e justifique com o trecho do Anexo A.
```

**Prompt de refino (após comparar com a avaliação própria):**
```
Você aprovou a #8 pela banda (10 pontos), mas ela responde em inglês — viola o guardrail
de idioma da NovaTech. Considere uma regra de corte: violação de guardrail crítico
(idioma, inversão de regra, alucinação, assunção de dado não informado) reprova
independentemente do total. Reavalie #8 e #6 sob essa regra.
```
O Claude convergiu para a reprovação da #8 sob a regra de corte. Essa iteração está documentada como a principal divergência na Parte 2.

### Etapa 3 — Claude Cowork (relatório de qualidade)

O Cowork foi usado para transformar a avaliação detalhada no relatório de uma página da Parte 3:
- Sintetizou os **KPIs** (score médio, taxa de aprovação, alucinações, reconhecimento de lacunas) a partir da tabela consolidada.
- Estruturou a seção de **reprovações com motivo e tipo de falha**, separando falha de conteúdo de falha de comportamento/guardrail.
- Formatou o **parecer de go-live** condicional com as duas barreiras bloqueantes e o risco residual aceitável — calibrado para leitura executiva em 2 minutos.
