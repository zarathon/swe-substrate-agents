---
description: Orquestrador para gerar CLAUDE.md otimizado através de agentes especializados
---

# Gerador de CLAUDE.md com Agentes Especializados

Bem-vindo ao sistema de geração de CLAUDE.md! Este sistema usa agentes especializados para coletar informações sobre seu projeto e gerar uma documentação otimizada para o Claude Code.

## Como Funciona

Cada agente especializado cobre um aspecto específico do projeto:

1. **`/swe-substrate-project-context`** - Contexto do Projeto (Visão, Domínio, Negócio, Métricas)
2. **`/swe-substrate-tooling-config`** - Ferramental e Configuração (Docker, IaC, Env Vars)
3. **`/swe-substrate-languages`** - Linguagens (Idiomática, Concorrência, Guidelines)
4. **`/swe-substrate-api-design`** - Design de API (REST, GraphQL, RPC)
5. **`/swe-substrate-testing-strategy`** - Estratégia de Testes (Unidade, Integração, Regressão)
6. **`/swe-substrate-agents`** - Agentes (Daemons, Workers, LLM)
7. **`/swe-substrate-package-mgmt`** - Gerenciamento de Pacotes/Dependências
8. **`/swe-substrate-observability`** - Observabilidade (Métricas, Logs, Traces)
9. **`/swe-substrate-data`** - Dados (DB, Schema, Cache)
10. **`/swe-substrate-security`** - Segurança (Autenticação, Permissões, Certificados)
11. **`/swe-substrate-documentation`** - Documentação (Comentários, Diagramas)

## ⚠️ IMPORTANTE

**Você NÃO precisa executar todos os agentes!**

Execute apenas os agentes relevantes para seu projeto. Por exemplo:
- Um projeto frontend pode não precisar de `/swe-substrate-data`
- Um CLI tool pode não precisar de `/swe-substrate-api-design`
- Um projeto simples pode não ter `/swe-substrate-agents`

## Documentação Modular

Este sistema gera documentação **modularizada** em `.claude/swe-substrate/`:

- Cada agente cria/atualiza seu próprio arquivo (ex: `tooling-config.md`)
- O CLAUDE.md principal permanece leve, com links para os módulos
- Documentação escalável sem arquivo gigante
- Fácil de manter e navegar

## Instruções

1. **Analise seu projeto** e identifique quais agentes são relevantes
2. **Execute cada agente relevante** separadamente (ex: `/swe-substrate-languages`)
3. Cada agente fará **perguntas abertas** - responda com o máximo de detalhes possível
4. Cada agente gerará um **arquivo modular** em `.claude/swe-substrate/`
5. O CLAUDE.md do projeto será atualizado com links para os módulos

## Workflow Recomendado

Para **novos projetos**, considere começar com:
- `/swe-substrate-project-context` (contexto de negócio e visão geral)
- `/swe-substrate-tooling-config` (setup inicial)
- `/swe-substrate-languages` (convenções de código)
- `/swe-substrate-testing-strategy` (qualidade desde o início)

Para **projetos existentes**, execute os agentes conforme necessidade:
- Adicione `/swe-substrate-security` ao documentar práticas de segurança
- Use `/swe-substrate-observability` ao configurar monitoramento
- Execute `/swe-substrate-data` ao adicionar persistência

## Começando

Escolha o primeiro agente relevante para seu projeto e execute o comando correspondente.

Qual agente você gostaria de começar?
