# dgs-ai-first

Repositório de práticas do programa **AI First** da DB1, voltado ao desenvolvimento de competências em IA Generativa aplicadas ao ciclo de desenvolvimento de software.

## Sobre o programa

O AI First é uma iniciativa da DB1 para capacitar times multidisciplinares (Delivery Managers, Product Specialists, Developers, Tech Leads e QAs) a incorporar ferramentas e técnicas de IA no dia a dia de projetos reais. As práticas seguem o modelo **AI First SDLC**, no qual agentes de IA participam ativamente das fases de entendimento, discovery, design e desenvolvimento.

## Estrutura do repositório

```
dgs-ai-first/
├── Prática 1/                          # Cenário-Âncora 1 — Fase de Entendimento
│   ├── exercicio-fase-1-entendimento.md    # Enunciados dos exercícios por papel
│   ├── anexo-a-documentacao-simulada-novatech.md  # Documentação da empresa simulada
│   ├── anexo-b-chunks-referencia-rag.md    # Chunks de referência para o pipeline RAG
│   ├── POL-001-politica-devolucao.md       # Política de devolução (NovaTech)
│   ├── PROC-042-frete-especial-v1.md       # Procedimento de frete especial v1
│   ├── PROC-042-v2-frete-especial-revisado.md  # Procedimento de frete especial v2
│   ├── SLA-2024-tabela-sla-clientes.md     # Tabela de SLAs por tier de cliente
│   └── FAQ-atendimento.md                  # FAQ informal da equipe de atendimento
│
├── Prática 2/                          # Cenário-Âncora 2 — Fase de Estruturação
│   ├── exercicio-2-fase-estruturacao.md    # Enunciados dos exercícios por papel
│   ├── anexo-a-documentacao-simulada-novatech.md  # Documentação da NovaTech (referência)
│   ├── anexo-b-chunks-referencia-rag.md    # Chunks de referência para dados de teste
│   ├── anexo-c-estrutura-repositorio.md    # Estrutura do repositório novatech-assistant
│   ├── POL-001-politica-devolucao.md       # Política de devolução (NovaTech)
│   ├── PROC-042-frete-especial-v1.md       # Procedimento de frete especial v1
│   ├── PROC-042-v2-frete-especial-revisado.md  # Procedimento de frete especial v2
│   ├── SLA-2024-tabela-sla-clientes.md     # Tabela de SLAs por tier de cliente
│   └── FAQ-atendimento.md                  # FAQ informal da equipe de atendimento
│
├── Prática 3/                          # Cenário-Âncora 3 — Fase de Governança e Validação
│   └── Cenário/
│       └── cenario-3-exercicios-fase-governanca.md  # Enunciados dos exercícios por papel
│
├── exercicio1-1.md     # Resposta — QA: Identificação de cenários de falha de IA
├── exercicio1-2.md     # Resposta — QA: Design de critérios de aceitação
├── exercicio1-3.md     # Resposta — QA: Plano de testes para pipeline de RAG
├── exercicio2-1.md     # Resposta — QA: Seção Testing Standards do AGENTS.md
├── exercicio2-2.md     # Resposta — QA: Test plan (SDD) para o query endpoint
├── exercicio2-3.md     # Resposta — QA: SKILL.md create-integration-test + checklist
├── exercicio3-1.md     # Resposta — QA: Revisão crítica das respostas + parecer de go-live
└── exercicio3-2.md     # Resposta — QA: Revisão crítica dos testes gerados por IA
```

## Cenário-Âncora 1 — Fase de Entendimento

O cenário base das práticas envolve a **NovaTech**, empresa de logística com 1.200 funcionários que contratou a DB1 para construir um **assistente de IA** integrado ao ambiente Microsoft (Teams + SharePoint). O assistente permite que atendentes consultem a documentação interna em linguagem natural, reduzindo o tempo médio de busca por chamado de 12 para menos de 2 minutos.

O cenário foi projetado com desafios realistas:

- Documentação distribuída em três fontes (~1.250 itens: SharePoint, Confluence, planilhas)
- Versões contraditórias de procedimentos (PROC-042 v1 vs. v2)
- Conhecimento informal não documentado formalmente (FAQ de equipe)
- Lacunas de cobertura documental (ex: frete padrão abaixo de 500kg)
- Armadilhas para teste de alucinação (tier "Platinum" que não existe)

## Cenário-Âncora 2 — Fase de Estruturação

Continuação do projeto NovaTech após aprovação e conclusão do discovery. O time estrutura o ambiente, os padrões e os artefatos que vão governar o desenvolvimento antes da primeira linha de código de produção.

Decisões consolidadas do Cenário 1:

- **Modelo LLM:** Azure OpenAI GPT-4o (128K tokens, integração Microsoft — ADR-0001)
- **Pipeline RAG:** Azure AI Search + Azure OpenAI; protótipo open-source validou a abordagem
- **Stack:** TypeScript, Azure Functions v4, React, Bicep (infraestrutura como código)
- **Repositório:** `db1/novatech-assistant` no GitHub da DB1

Artefatos produzidos nesta fase:
- Seção `## Testing Standards` do `AGENTS.md` do projeto
- Test plan (formato SDD) para o query endpoint com 10 cenários funcionais + 5 de robustez
- SKILL.md `create-integration-test` (nível Artifact) + checklist de revisão de testes

## Cenário-Âncora 3 — Fase de Governança e Validação

Última fase antes do go-live do assistente NovaTech. O pipeline de RAG está funcional, os primeiros endpoints foram implementados e o bot do Teams responde perguntas de teste em staging — mas o time precisa garantir que o sistema é confiável e governável antes da demonstração à diretoria.

O foco da fase é o **harness de governança**: o conjunto de verificações e limites que transforma um protótipo em um sistema de produção, usando **structured outputs** (forçar o modelo a responder em formato validável) e **human-in-the-loop** (pontos onde um humano valida antes de prosseguir). A fase também exige **revisão crítica de outputs de IA** — respostas do assistente, código e testes gerados com apoio de ferramentas.

Artefatos produzidos pela disciplina de QA nesta fase:
- Revisão crítica de 8 respostas do assistente com aplicação de rubrica, comparação com segundo avaliador e parecer de go-live
- Revisão crítica de 3 testes de integração gerados por IA + reescrita de teste com verificação de conteúdo e citação de fonte

## Tópicos cobertos

**Prática 1:**
- Fundamentos de IA Generativa
- Engenharia de Prompt
- Engenharia de Contexto (context rot, lost in the middle, context overflow)
- RAG — Retrieval-Augmented Generation

**Prática 2:**
- MCP (Model Context Protocol)
- Recorte de Domínio e Spec Driven Development (SDD)
- AGENTS.md
- Skills

**Prática 3:**
- Harness Engineering (Human-in-the-Loop e Structured Outputs)
- Revisão Crítica de Outputs de IA (respostas, código e testes)

## Papéis e ferramentas

| Papel | Ferramentas utilizadas |
|-------|------------------------|
| Delivery Manager | Claude (chat), Claude Cowork |
| Product Specialist | Claude (chat), Claude Cowork, Claude Design |
| Developer | Claude (chat), GitHub Copilot |
| Tech Lead | Claude (chat), GitHub Copilot |
| QA | Claude (chat), Claude Cowork |

## Exercícios entregues

### Prática 1

| Arquivo | Papel | Exercício |
|---------|-------|-----------|
| [exercicio1-1.md](exercicio1-1.md) | QA | Identificação de 10 cenários de falha do assistente de IA, organizados em 5 categorias |
| [exercicio1-2.md](exercicio1-2.md) | QA | Avaliação de 5 respostas do assistente + rubrica de qualidade com 4 dimensões + template reutilizável |
| [exercicio1-3.md](exercicio1-3.md) | QA | Plano de testes para o pipeline de RAG cobrindo ingestão, retrieval, geração, contexto, ponta a ponta e regressão |

### Prática 2

| Arquivo | Papel | Exercício |
|---------|-------|-----------|
| [exercicio2-1.md](exercicio2-1.md) | QA | Seção `## Testing Standards` do AGENTS.md do projeto NovaTech Assistant — padrões prescritivos para testes gerados por IA |
| [exercicio2-2.md](exercicio2-2.md) | QA | Test plan em formato SDD para o query endpoint: 10 cenários funcionais (VC-01 a VC-04) + 5 testes de robustez (ambiguidade, prompt injection, idiomas) |
| [exercicio2-3.md](exercicio2-3.md) | QA | SKILL.md da skill Artifact `create-integration-test`: template, exemplos DO/DON'T, anti-padrões de LLMs em testes + checklist de revisão em 8 itens |

### Prática 3

| Arquivo | Papel | Exercício |
|---------|-------|-----------|
| [exercicio3-1.md](exercicio3-1.md) | QA | Revisão crítica de 8 respostas do assistente com rubrica de 4 dimensões + regra de corte, comparação com segundo avaliador e relatório de qualidade com parecer de go-live (identifica as reprovações por assunção de dado e por idioma) |
| [exercicio3-2.md](exercicio3-2.md) | QA | Revisão crítica de 3 testes de integração gerados por IA (assertions vagas, ausência de domínio, mock permissivo, inconsistência jest/Vitest) + reescrita do Teste 1 verificando conteúdo da resposta e citação de fonte |
