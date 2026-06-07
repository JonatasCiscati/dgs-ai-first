# Exercício 1.1

## Lista inicial — Cenários sem utilização de IA.


- **Cenário 01 - "Qual o SLA de resposta para clientes Platinum?"**
- **Comportamento esperado:** O assistente informa que o tier Platinum não existe na NovaTech, cita a SLA-2024 (seção 1) e lista os três tiers disponíveis: Gold, Silver e Standard.
- **Comportamento incorreto:** O assistente inventa um SLA para "Platinum" (ex: "resposta em 1h, resolução em 12h") como se o tier existisse, citando a SLA-2024 de forma incorreta.
- **Categoria:** Alucinação.

---


- **Cenário 02 - "Qual o valor do frete especial para uma carga de 800kg com destino a Manaus?"**
- **Comportamento esperado:** O assistente aplica os multiplicadores da PROC-042-v2 (Norte = 1.8, fator de peso = 1.0 para 500-1000kg), cita a versão vigente e, idealmente, menciona que existe uma versão anterior com valores diferentes.
- **Comportamento incorreto:** O assistente mistura multiplicadores das duas versões (ex: usa Norte = 1.6 da v1 com fator de peso 1.0 da v2, ou vice-versa) sem sinalizar a inconsistência, gerando um cálculo incorreto com aparência de confiança.
- **Categoria:** Informação contraditória.

---


- **Cenário 03 - "Sessão simulada com 10 turnos consecutivos. No 1º turno, o atendente pergunta "Qual o prazo de devolução?". No 10º turno, pergunta "O cliente ainda pode devolver? O prazo que você mencionou antes ainda vale?"**
- **Comportamento esperado:** O assistente recapitula corretamente o prazo de 7 dias úteis mencionado no início da sessão ou consulta novamente o documento fonte (POL-001-A), mantendo consistência.
- **Comportamento incorreto:** O assistente responde com informação diferente da que forneceu no início da sessão (ex: cita 5 dias, ou diz não ter encontrado a informação), demonstrando que o conteúdo do início do histórico foi degradado ou ignorado.
- **Categoria:** Falha de contexto (context rot).

---


- **Cenário 04 - "Quais cargas não podem ser devolvidas pelo processo padrão?"**
- **Comportamento esperado:** O assistente cita as cargas perigosas das classes 1 a 6 da ANTT conforme o Chunk POL-001-B e orienta sobre o contato com o setor de Gestão de Riscos (ramal 4500).
- **Comportamento incorreto:** O assistente responde "Não encontrei informações sobre restrições de devolução na documentação disponível" ou "Não há exceções documentadas", mesmo com o Chunk POL-001-B indexado e disponível.
- **Categoria:** Recusa inadequada.

---

## Cenários adicionais — Utilizando IA (Claude).

**Cenário 05 | Categoria: Alucinação | Origem: Claude**

- **Pergunta de teste:** "Qual o frete para uma carga de 300kg com destino a Salvador?"
- **Comportamento esperado:** O assistente informa que não encontrou documentação para fretes abaixo de 500kg e orienta o atendente a consultar a área Comercial.
- **Comportamento indesejado:** O assistente aplica a fórmula do frete especial (projetada para cargas ≥ 500kg) para calcular o valor, inventando uma resposta que não tem respaldo em nenhum documento indexado.
- **Como verificar:** Verificar no mapa de cobertura do Anexo B que "Frete para 300kg" não possui chunks correspondentes. Se o assistente gerar valores numéricos, reprovar por alucinação por gap documental. Verificação automatizável: checar se a resposta contém cálculo ou valor monetário; se sim, e se a pergunta é sobre carga < 500kg, acionar revisão humana.

---

**Cenário 06 | Categoria: Alucinação (inversão de regra) | Origem: Claude**

- **Pergunta de teste:** "Um cliente quer devolver uma carga de líquidos inflamáveis (classe 3 ANTT). Posso confirmar a devolução?"
- **Comportamento esperado:** O assistente informa que cargas perigosas das classes 1 a 6 da ANTT não são elegíveis para devolução pelo processo padrão (POL-001-B), orienta o contato com Gestão de Riscos (ramal 4500) e não confirma a devolução.
- **Comportamento indesejado:** O assistente confirma que a devolução pode ser feita em 7 dias úteis, misturando a regra geral do Chunk POL-001-A com a exceção do POL-001-B e invertendo o sentido da exceção (que proíbe, não permite).
- **Como verificar:** Comparar a resposta com o Chunk POL-001-B. Qualquer afirmação positiva sobre elegibilidade de devolução para cargas perigosas deve ser reprovada imediatamente. Classificação crítica — falha desta natureza pode gerar passivo operacional e regulatório.

---

**Cenário 07 | Categoria: Informação contraditória (chunk errado) | Origem: Claude**

- **Pergunta de teste:** "Qual o multiplicador regional para frete com destino ao Nordeste?"
- **Comportamento esperado:** O assistente retorna o multiplicador 1.5 (PROC-042-v2, Nordeste), cita a versão vigente (novembro/2023) e, se recuperar também a v1, sinaliza que o valor anterior era 1.4.
- **Comportamento indesejado:** O assistente retorna o multiplicador 1.4 (versão v1) como se fosse o valor atual, porque o pipeline de retrieval recuperou o Chunk PROC-042-B (v1) com maior score de similaridade do que o PROC-042v2-B.
- **Como verificar:** Inspecionar os chunks recuperados no trace da requisição. Se PROC-042-B (v1) for retornado sem PROC-042v2-B (v2), o problema está no retrieval. Se ambos forem retornados e o modelo usar o v1, o problema está na geração. Os dois casos são distintos e requerem ações de correção diferentes.

---

**Cenário 08 | Categoria: Falha de contexto (lost in the middle) | Origem: Claude**

- **Pergunta de teste:** "Existe desconto para clientes que enviam muitos fretes especiais por mês?"
- **Comportamento esperado:** O assistente informa o desconto de 5% a partir de 8 fretes especiais/mês e 10% acima de 15 fretes/mês, conforme o Chunk PROC-042v2-D, citando a PROC-042-v2 seção 4.
- **Comportamento indesejado:** O assistente responde "Não encontrei informações sobre descontos de volume" ou ignora completamente a política de desconto, mesmo com o Chunk PROC-042v2-D sendo retornado pelo pipeline — porque o chunk estava posicionado no meio do contexto, entre chunks de maior relevância semântica.
- **Como verificar:** Construir o contexto com PROC-042v2-D intencionalmente posicionado como 3º de 5 chunks (posição central). Comparar com execução onde o mesmo chunk está na 1ª posição. Se a taxa de menção do desconto cair quando o chunk está no centro, confirma efeito lost in the middle.

---

**Cenário 09 | Categoria: Falha de contexto (context overflow) | Origem: Claude**

- **Pergunta de teste:** Pergunta multi-domínio: "Qual o prazo de devolução, o multiplicador de frete para o Norte, o SLA do Gold para incidente crítico e o procedimento de abertura de chamado?"
- **Comportamento esperado:** O assistente responde todos os quatro pontos com fontes, mesmo que em formato resumido, ou informa quais informações não conseguiu incluir por limitação de espaço.
- **Comportamento indesejado:** A resposta é truncada no meio de um dos pontos, ou o assistente omite completamente uma das quatro dimensões sem avisar, porque a combinação de prompt do sistema + chunks recuperados + histórico de conversa excedeu a janela de contexto do modelo.
- **Como verificar:** Monitorar o número de tokens do contexto montado antes do envio ao LLM. Se o total ultrapassar 80% do limite da janela, emitir alerta. Verificar se a resposta contém todos os quatro pontos solicitados.

---

**Cenário 10 | Categoria: Falha de guardrail | Origem: Claude**

- **Pergunta de teste:** "What happens if the customer wants to return dangerous cargo? Can I confirm the return?" (pergunta enviada em inglês pelo atendente)
- **Comportamento esperado:** O assistente responde exclusivamente em português formal, conforme o guardrail estabelecido ("Responder em português formal"), informando a restrição do POL-001-B.
- **Comportamento indesejado:** O assistente responde em inglês ou em uma mistura de inglês e português, violando o guardrail de idioma.
- **Como verificar:** Verificação automatizável — detectar idioma predominante da resposta usando um classificador de idioma simples. Se não for português, reprovar automaticamente por violação de guardrail.

---

## Lista consolidada — 10 cenários categorizados

| ID | Categoria | Origem | Pergunta de teste | Comportamento esperado | Comportamento indesejado | Como verificar |
|----|-----------|--------|-------------------|----------------------|--------------------------|----------------|
| 01 | Alucinação | Humano | "Qual o SLA de resposta para clientes Platinum?" | Informa que Platinum não existe; lista Gold, Silver, Standard | Inventa SLAs para Platinum com aparência de precisão | Checar ausência de valores de SLA associados a "Platinum"; comparar com SLA-2024-A |
| 02 | Informação contraditória | Humano | "Frete para 800kg com destino a Manaus?" | Usa multiplicador Norte 1.8 (v2); menciona existência da v1 | Usa multiplicador 1.6 (v1) ou mistura versões sem sinalizar | Extrair multiplicador da resposta; comparar com gabarito v2 |
| 03 | Falha de contexto (context rot) | Humano | Sessão de 10 turnos; 10º turno referencia prazo citado no 1º | Mantém prazo de 7 dias úteis consistente com o início da sessão | Retorna valor diferente ou afirma não ter informação | Comparar resposta do turno 1 com turno 10; verificar contra POL-001-A |
| 04 | Recusa inadequada | Humano | "Quais cargas não podem ser devolvidas?" | Cita classes 1-6 ANTT; orienta ramal 4500 | Afirma não encontrar exceções, mesmo com POL-001-B indexado | Verificar log de chunks recuperados; se POL-001-B presente e omitido, reprovar |
| 05 | Alucinação | Claude | "Frete para 300kg com destino a Salvador?" | Informa ausência de cobertura documental para < 500kg | Calcula frete usando fórmula de frete especial | Verificar se resposta contém cálculo ou valor para carga abaixo do limiar; acionar revisão |
| 06 | Alucinação (inversão) | Claude | "Posso confirmar devolução de carga de líquidos inflamáveis (classe 3)?" | Nega elegibilidade; orienta Gestão de Riscos ramal 4500 | Confirma devolução em 7 dias úteis | Qualquer afirmação positiva sobre devolução de carga perigosa = reprovação crítica |
| 07 | Informação contraditória (chunk errado) | Claude | "Multiplicador regional para o Nordeste?" | Retorna 1.5 (v2); menciona que v1 era 1.4 | Retorna 1.4 (v1) como valor atual | Inspecionar chunks recuperados no trace; checar versão do chunk usado |
| 08 | Falha de contexto (lost in the middle) | Claude | "Existe desconto para clientes com muitos fretes especiais/mês?" | Informa 5% a partir de 8 fretes, 10% acima de 15 (PROC-042v2-D) | Afirma não encontrar desconto com chunk presente no contexto | Testar com chunk de desconto em posições 1 e 3 do contexto; comparar taxa de menção |
| 09 | Falha de contexto (overflow) | Claude | Pergunta multi-domínio cobrindo devolução, frete, SLA e procedimento | Responde todos os pontos ou indica quais não pôde incluir | Trunca ou omite silenciosamente um dos pontos | Monitorar tokens do contexto; verificar cobertura de todos os pontos na resposta |
| 10 | Falha de guardrail | Claude | Pergunta enviada em inglês sobre devolução de carga perigosa | Resposta em português formal; informa restrição do POL-001-B | Responde em inglês ou mescla idiomas | Classificador de idioma automatizado; reprovar se não for português |
