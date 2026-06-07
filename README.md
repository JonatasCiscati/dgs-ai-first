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
├── exercicio1-1.md     # Resposta — QA: Identificação de cenários de falha de IA
├── exercicio1-2.md     # Resposta — QA: Design de critérios de aceitação
└── exercicio1-3.md     # Resposta — QA: Plano de testes para pipeline de RAG
```

## Cenário-Âncora 1

O cenário base das práticas envolve a **NovaTech**, empresa de logística com 1.200 funcionários que contratou a DB1 para construir um **assistente de IA** integrado ao ambiente Microsoft (Teams + SharePoint). O assistente permite que atendentes consultem a documentação interna em linguagem natural, reduzindo o tempo médio de busca por chamado de 12 para menos de 2 minutos.

O cenário foi projetado com desafios realistas:

- Documentação distribuída em três fontes (~1.250 itens: SharePoint, Confluence, planilhas)
- Versões contraditórias de procedimentos (PROC-042 v1 vs. v2)
- Conhecimento informal não documentado formalmente (FAQ de equipe)
- Lacunas de cobertura documental (ex: frete padrão abaixo de 500kg)
- Armadilhas para teste de alucinação (tier "Platinum" que não existe)

## Tópicos cobertos

- Fundamentos de IA Generativa
- Engenharia de Prompt
- Engenharia de Contexto (context rot, lost in the middle, context overflow)
- RAG — Retrieval-Augmented Generation

## Papéis e ferramentas

| Papel | Ferramentas utilizadas |
|-------|------------------------|
| Delivery Manager | Claude (chat), Claude Cowork |
| Product Specialist | Claude (chat), Claude Cowork, Claude Design |
| Developer | Claude (chat), GitHub Copilot |
| Tech Lead | Claude (chat), GitHub Copilot |
| QA | Claude (chat), Claude Cowork |

## Exercícios entregues (branch `cenario-1`)

| Arquivo | Papel | Exercício |
|---------|-------|-----------|
| [exercicio1-1.md](exercicio1-1.md) | QA | Identificação de 10 cenários de falha do assistente de IA, organizados em 5 categorias |
| [exercicio1-2.md](exercicio1-2.md) | QA | Avaliação de 5 respostas do assistente + rubrica de qualidade com 4 dimensões + template reutilizável |
| [exercicio1-3.md](exercicio1-3.md) | QA | Plano de testes para o pipeline de RAG cobrindo ingestão, retrieval, geração, contexto, ponta a ponta e regressão |
