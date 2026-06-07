# Exercício 1.3

### Plano de testes — Pipeline de RAG NovaTech

O pipeline a ser testado segue o fluxo: extração de documentos → conversão em texto → chunking → geração de embeddings → indexação no Azure AI Search → busca por similaridade → geração de resposta pelo LLM com chunks como contexto.

Testes de IA generativa diferem de testes tradicionais em dois aspectos fundamentais: (1) as respostas são não-determinísticas — a mesma entrada pode gerar saídas diferentes entre execuções; (2) a qualidade não é binária (pass/fail), mas se distribui em graus. O plano abaixo reconhece estas características e define critérios de aceite que trabalham com limiar de qualidade e avaliação por amostra, não com verificação exaustiva.

---

## Categoria 1 — Testes de Ingestão

Verificar que os documentos foram corretamente extraídos das fontes, convertidos para texto e indexados com metadados íntegros.

| ID | Descrição | Critério de aceite | Responsável | Status |
|----|-----------|-------------------|-------------|--------|
| ING-01 | Contagem de documentos indexados | Total de chunks indexados no Azure AI Search deve corresponder ao total esperado de chunks por documento (conforme mapa de chunking definido na configuração do pipeline) | Dev / QA | — |
| ING-02 | Integridade de texto — PDF | Amostrar 10% dos chunks extraídos de PDFs. Nenhum chunk deve conter caracteres corrompidos, texto ilegível ou sequências de OCR incorreto que alterem o significado da informação | Dev / QA | — |
| ING-03 | Integridade de texto — DOCX e XLSX | Amostrar 10% dos chunks extraídos de documentos Word e planilhas. Verificar que tabelas e formatações numéricas (multiplicadores, prazos, valores) foram convertidas corretamente para texto sem truncamento ou inversão de colunas | Dev / QA | — |
| ING-04 | Metadados obrigatórios | Todo chunk deve conter os metadados: nome do documento de origem, número de versão (quando aplicável), data de vigência e seção de origem. Verificar com consulta direta ao índice | Dev | — |
| ING-05 | Versionamento — substituição de documento | Após ingestão da PROC-042-v2, o chunk PROC-042-B (v1) deve ser marcado com flag `obsoleto=true` ou removido do índice ativo, de forma que buscas por similaridade não retornem a versão antiga como resultado primário | Dev / QA | — |
| ING-06 | Atualização incremental | Após atualizar um documento no SharePoint e disparar nova ingestão, verificar que a versão anterior foi substituída no índice e que a nova versão está disponível para consulta em menos de 24h | Dev / QA | — |

---

## Categoria 2 — Testes de Retrieval

Verificar que, dada uma pergunta, os chunks corretos são recuperados pelo Azure AI Search. Usar o mapa de cobertura do Anexo B como gabarito.

Os testes desta categoria são determinísticos — dado o mesmo índice e a mesma pergunta, o resultado deve ser estável. Reprovar quando o chunk obrigatório não estiver entre os 3 primeiros resultados retornados.

| ID | Pergunta de teste | Chunks que DEVEM ser retornados (obrigatórios) | Chunks que NÃO devem ser retornados como primário | Critério de aceite |
|----|-------------------|-----------------------------------------------|---------------------------------------------------|-------------------|
| RET-01 | "Qual o prazo de devolução?" | POL-001-A, POL-001-B | — | Ambos os chunks obrigatórios presentes nos 3 primeiros resultados |
| RET-02 | "Posso devolver carga perigosa?" | POL-001-B | FAQ-03 não deve ocupar posição 1 ou 2 | POL-001-B na posição 1; FAQ-03 somente como resultado secundário, se presente |
| RET-03 | "Qual o SLA do cliente Platinum?" | SLA-2024-A (chunk que afirma que o tier não existe) | Nenhum chunk que implique existência de Platinum | SLA-2024-A retornado; resposta gerada a partir dele deve informar que o tier não existe |
| RET-04 | "Qual o multiplicador de frete para a região Norte?" | PROC-042v2-B | PROC-042-B (v1) não deve ser o único resultado | PROC-042v2-B presente nos 3 primeiros; se PROC-042-B também retornado, não deve ser o único |
| RET-05 | "Qual o SLA para incidente crítico de cliente Gold?" | SLA-2024-C | — | SLA-2024-C presente nos 3 primeiros resultados |
| RET-06 | "Qual o frete para uma carga de 300kg com destino a Salvador?" | Nenhum chunk relevante esperado | — | Pipeline deve retornar 0 chunks com score de similaridade acima do limiar mínimo definido; assistente deve declarar ausência de informação |
| RET-07 | "Existe desconto de volume para frete especial?" | PROC-042v2-D | — | PROC-042v2-D presente nos 3 primeiros resultados |

---

## Categoria 3 — Testes de Geração

Verificar que, fornecidos os chunks corretos como contexto, o LLM gera uma resposta adequada. Esta categoria isola a camada de geração — os chunks são injetados diretamente no prompt, sem passar pelo retrieval.

| ID | Descrição | Critério de aceite | Responsável | Status |
|----|-----------|-------------------|-------------|--------|
| GER-01 | Resposta com chunks corretos — caso base | Fornecer POL-001-A e POL-001-B ao modelo. Pergunta: "Qual o prazo de devolução e há exceções?" A resposta deve citar o prazo de 7 dias úteis, a exceção de cargas perigosas e a fonte | Resposta menciona 7 dias úteis, menciona classes 1-6 ANTT e cita POL-001 | QA | — |
| GER-02 | Guardrail — citação de fonte | Submeter qualquer pergunta com chunks válidos. A resposta deve sempre incluir referência ao documento de origem | 100% das respostas geradas com chunks disponíveis deve incluir citação de fonte | QA | — |
| GER-03 | Guardrail — ausência de informação | Submeter pergunta sem fornecer chunks relevantes. A resposta deve declarar explicitamente que não encontrou informação | Resposta não deve conter valores, prazos ou procedimentos inventados quando nenhum chunk é fornecido | QA | — |
| GER-04 | Chunks contraditórios — sinalização de ambiguidade | Fornecer simultaneamente PROC-042-B (v1, Norte 1.6) e PROC-042v2-B (v2, Norte 1.8). Pergunta: "Qual o multiplicador para o Norte?" | Resposta deve sinalizar que há duas versões com valores diferentes e indicar qual é a mais recente, sem escolher silenciosamente uma das versões | QA | — |
| GER-05 | Guardrail — idioma | Submeter pergunta em inglês. A resposta deve estar em português formal independentemente do idioma da pergunta | Resposta integralmente em português; reprovar se detectado outro idioma dominante | QA | — |
| GER-06 | Inversão de regra — exceção proibitiva | Fornecer POL-001-B ao modelo. Pergunta: "Posso devolver carga perigosa?" | Resposta deve negar a devolução pelo processo padrão e orientar o ramal 4500; reprovar qualquer afirmação positiva sobre elegibilidade de devolução | QA | — |

---

## Categoria 4 — Testes de Contexto

Verificar que o pipeline gerencia corretamente o orçamento de contexto e que problemas de engenharia de contexto não degradam a qualidade das respostas.

Estes testes são os mais complexos porque exigem avaliação comparativa — o comportamento deve ser avaliado em condições controladas, não apenas em execução única.

| ID | Descrição | Critério de aceite | Responsável | Status |
|----|-----------|-------------------|-------------|--------|
| CTX-01 | Orçamento de contexto — verificação de tokens | Monitorar o total de tokens do contexto montado (prompt do sistema + chunks recuperados + histórico de conversa) em execuções reais. O total não deve ultrapassar 80% da janela do modelo | Alertas automáticos quando contexto superar o limiar; nenhum overflow silencioso | Dev | — |
| CTX-02 | Context rot — sessão longa | Simular sessão com 10 turnos consecutivos. No turno 1, informar o prazo de devolução (7 dias úteis). No turno 10, perguntar: "Qual o prazo que você mencionou antes?" | Resposta do turno 10 deve manter consistência com o turno 1; variação de informação configura falha de context rot | QA | — |
| CTX-03 | Lost in the middle — posição do chunk | Executar a mesma pergunta ("Existe desconto de volume para frete especial?") duas vezes: (A) com Chunk PROC-042v2-D na posição 1 do contexto; (B) com o mesmo chunk na posição 3 de 5. Comparar as respostas | A taxa de menção do desconto não deve variar mais de 20% entre as posições A e B em 10 execuções amostradas | QA | — |
| CTX-04 | Context overflow — truncamento | Montar um prompt que intencionalmente ultrapasse a janela do modelo. Verificar o comportamento do sistema | O pipeline deve detectar o overflow antes de enviar ao modelo e truncar os chunks menos relevantes de forma controlada, não silenciosamente; a resposta gerada deve ser coerente e não truncada no meio de uma regra | Dev / QA | — |
| CTX-05 | Degradação em sessão estendida no Teams | Simular integração com Teams: sessão de 15 mensagens, com perguntas sobre diferentes domínios (devolução, frete, SLA). Avaliar respostas do turno 12 ao 15 com a rubrica do Exercício 1.2 | Pontuação média das respostas dos turnos 12-15 não deve ser inferior à pontuação média dos turnos 1-3 em mais de 2 pontos na escala de 4-12 | QA | — |

---

## Categoria 5 — Testes de Ponta a Ponta

Verificar o fluxo completo de ponta a ponta (pergunta → busca → resposta) com um conjunto de perguntas e respostas esperadas pré-definido (golden set).

O golden set deve ser mantido sob controle de versão e revisado sempre que documentos da NovaTech forem atualizados.

**Golden set — 10 pares pergunta / resposta esperada:**

| ID | Pergunta | Resposta esperada (elementos obrigatórios) | Fonte de verdade |
|----|----------|-------------------------------------------|-----------------|
| E2E-01 | "Qual o prazo de devolução?" | 7 dias úteis; exceção classes 1-6 ANTT | POL-001-A, POL-001-B |
| E2E-02 | "Posso devolver carga perigosa?" | NÃO pelo processo padrão; orientar ramal 4500 | POL-001-B |
| E2E-03 | "Qual o SLA do cliente Platinum?" | Platinum não existe; tiers são Gold, Silver, Standard | SLA-2024-A |
| E2E-04 | "Qual o SLA de resposta para incidentes críticos do Gold?" | 30 minutos | SLA-2024-C |
| E2E-05 | "Qual o multiplicador para frete especial no Norte?" | 1.8 (versão v2, novembro/2023) | PROC-042v2-B |
| E2E-06 | "Qual o frete para 300kg com destino a Salvador?" | Sem informação — frete padrão abaixo de 500kg não está documentado | Ausência de chunk |
| E2E-07 | "Existe desconto de volume para frete especial?" | 5% a partir de 8 fretes/mês; 10% acima de 15 | PROC-042v2-D |
| E2E-08 | "Quem paga o frete reverso na devolução por desistência do cliente?" | O cliente paga o frete reverso | POL-001-D |
| E2E-09 | "Qual o multiplicador para o Sudeste no frete especial?" | 1.1 (v2); mencionar que v1 registrava 1.0 como ressalva | PROC-042v2-B, PROC-042-B |
| E2E-10 | "O que fazer com carga com status 'em trânsito' há mais de 6h e valor acima de R$100.000?" | Classificar como incidente crítico; acionar SLA Gold crítico (resposta 30min) | SLA-2024-C, SLA-2024-D |

**Métricas do golden set:**

| Métrica | Meta | Mínimo aceitável |
|---------|------|-----------------|
| % de respostas com fonte citada corretamente | 100% | 90% |
| % de respostas factualmente corretas (avaliadas pela rubrica — D1 ≥ 2) | 90% | 80% |
| % de recusas adequadas quando não há cobertura documental (E2E-06) | 100% | 100% |
| Nenhuma resposta com alucinação de tier inexistente (E2E-03) | 0 falhas | 0 falhas — sem tolerância |
| Nenhuma resposta confirmando devolução de carga perigosa (E2E-02) | 0 falhas | 0 falhas — sem tolerância |

---

## Categoria 6 — Testes de Regressão

Verificar que mudanças no pipeline (atualização de prompt, substituição de documento, modificação de configuração de chunking) não degradam respostas que estavam funcionando corretamente.

| ID | Gatilho | Testes disparados automaticamente | Critério de aceite | Responsável | Status |
|----|---------|-----------------------------------|--------------------|-------------|--------|
| REG-01 | Mudança no prompt do sistema | Golden set completo (E2E-01 a E2E-10) | Pontuação total do golden set não deve cair mais de 5% em relação à baseline registrada antes da mudança | Dev / QA | — |
| REG-02 | Re-indexação de documento após atualização | Testes de retrieval (RET-01 a RET-07) para o documento atualizado | Todos os testes de retrieval do documento atualizado devem passar; nenhum chunk da versão anterior deve retornar como resultado primário | Dev | — |
| REG-03 | Mudança na configuração de chunking (tamanho, sobreposição) | Testes de integridade de texto (ING-02, ING-03) + testes de retrieval completos (RET-01 a RET-07) | Integridade mantida; recall do retrieval igual ou superior à baseline | Dev / QA | — |
| REG-04 | Atualização da versão do modelo LLM | Golden set completo + testes de guardrail (GER-02, GER-03, GER-05, GER-06) | Pontuação do golden set igual ou superior; nenhuma regressão nos guardrails críticos | Dev / QA | — |
| REG-05 | Baseline — registro mensal | Executar golden set e registrar pontuações como baseline de referência | Documento de baseline atualizado com data, versão do modelo, versão do prompt e métricas | QA | — |

---

## Artefato organizado — Checklist de execução

O checklist abaixo consolida todos os testes em formato rastreável para acompanhamento ao longo do projeto.

| Categoria | ID | Descrição resumida | Critério de aceite (resumo) | Status | Responsável | Sprint | Observações |
|-----------|----|--------------------|----------------------------|--------|-------------|--------|-------------|
| Ingestão | ING-01 | Contagem de chunks indexados | Total correto vs. esperado | — | Dev/QA | Sprint 1 | |
| Ingestão | ING-02 | Integridade PDFs | Sem texto corrompido (amostra 10%) | — | Dev/QA | Sprint 1 | |
| Ingestão | ING-03 | Integridade DOCX/XLSX | Tabelas e valores convertidos corretamente | — | Dev/QA | Sprint 1 | |
| Ingestão | ING-04 | Metadados obrigatórios | Todo chunk com fonte, versão, data, seção | — | Dev | Sprint 1 | |
| Ingestão | ING-05 | Versionamento de documento | v1 marcada como obsoleta após ingestão da v2 | — | Dev/QA | Sprint 1 | Crítico para PROC-042 |
| Ingestão | ING-06 | Atualização incremental | Nova versão disponível em < 24h | — | Dev/QA | Sprint 2 | |
| Retrieval | RET-01 | Prazo de devolução | POL-001-A e POL-001-B nos top 3 | — | QA | Sprint 2 | |
| Retrieval | RET-02 | Carga perigosa | POL-001-B na posição 1 | — | QA | Sprint 2 | FAQ-03 não deve ser primário |
| Retrieval | RET-03 | SLA Platinum | SLA-2024-A retornado | — | QA | Sprint 2 | Teste de alucinação |
| Retrieval | RET-04 | Multiplicador Norte | PROC-042v2-B nos top 3 | — | QA | Sprint 2 | Verificar v1 não é único |
| Retrieval | RET-05 | SLA crítico Gold | SLA-2024-C nos top 3 | — | QA | Sprint 2 | |
| Retrieval | RET-06 | Frete < 500kg | 0 chunks acima do limiar | — | QA | Sprint 2 | Teste de gap documental |
| Retrieval | RET-07 | Desconto de volume | PROC-042v2-D nos top 3 | — | QA | Sprint 2 | |
| Geração | GER-01 | Resposta base — devolução | Cita prazo + exceção + fonte | — | QA | Sprint 2 | |
| Geração | GER-02 | Guardrail — citação | 100% das respostas com fonte | — | QA | Sprint 2 | |
| Geração | GER-03 | Guardrail — ausência | Sem invenção quando sem chunks | — | QA | Sprint 2 | |
| Geração | GER-04 | Chunks contraditórios | Sinaliza ambiguidade de versão | — | QA | Sprint 3 | PROC-042 v1 + v2 |
| Geração | GER-05 | Guardrail — idioma | Resposta em português | — | QA | Sprint 2 | Automatizável |
| Geração | GER-06 | Inversão de regra — carga perigosa | Nega devolução; orienta ramal 4500 | — | QA | Sprint 2 | Crítico — sem tolerância |
| Contexto | CTX-01 | Orçamento de tokens | Alerta em > 80% da janela | — | Dev | Sprint 3 | |
| Contexto | CTX-02 | Context rot — sessão longa | Consistência entre turno 1 e 10 | — | QA | Sprint 3 | |
| Contexto | CTX-03 | Lost in the middle | Taxa de menção estável entre posições | — | QA | Sprint 3 | Requer 10 execuções |
| Contexto | CTX-04 | Context overflow | Truncamento controlado | — | Dev/QA | Sprint 3 | |
| Contexto | CTX-05 | Degradação no Teams | Qualidade turnos 12-15 ≥ turnos 1-3 | — | QA | Sprint 3 | |
| Ponta a Ponta | E2E-01 a E2E-10 | Golden set completo | Ver tabela de métricas | — | QA | Sprint 3 | |
| Regressão | REG-01 | Mudança de prompt | Golden set sem queda > 5% | — | Dev/QA | Contínuo | |
| Regressão | REG-02 | Re-indexação | Retrieval correto para doc atualizado | — | Dev | Contínuo | |
| Regressão | REG-03 | Mudança de chunking | Integridade + retrieval mantidos | — | Dev/QA | Contínuo | |
| Regressão | REG-04 | Atualização do modelo LLM | Golden set + guardrails sem regressão | — | Dev/QA | Contínuo | |
| Regressão | REG-05 | Baseline mensal | Métricas registradas mensalmente | — | QA | Mensal | |
