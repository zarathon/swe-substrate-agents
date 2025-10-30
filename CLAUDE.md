# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🌍 Language Support

This project supports **English (primary)** and **Portuguese (secondary)**:
- 🇺🇸 **Documentation**: README.md (English), README.pt-BR.md (Portuguese)
- 🌐 **Landing Page**: Bilingual with language switcher (EN/PT)
- 🤖 **Agents**: Currently in Portuguese (English versions planned)

When working on this repository, prioritize English for new content.

## Expertise Required

When working on this repository, you should act as an expert in creating tools for Claude, including:
- Agents and multi-agent systems
- Plugins and extensions
- MCP (Model Context Protocol) servers
- Custom tools and integrations
- Workflows and automations
- Any other tool or resource that can be used with Claude

## Project Overview

**SWE Substrate Agents** is a system of specialized agents that collect information about software projects and generate modularized documentation optimized for use with Claude Code.

### Purpose

Enable engineering teams to generate consistent, high-quality documentation that maximizes Claude Code's performance on their projects. The agents ask open-ended questions and perform automatic analysis to collect detailed information about different aspects of the project.

## Estrutura do Repositório

```
.claude/
├── commands/              # Slash commands (agentes)
│   ├── swe-substrate-init.md
│   ├── swe-substrate-project-context.md
│   ├── swe-substrate-tooling-config.md
│   ├── swe-substrate-languages.md
│   ├── swe-substrate-api-design.md
│   ├── swe-substrate-testing-strategy.md
│   ├── swe-substrate-agents.md
│   ├── swe-substrate-package-mgmt.md
│   ├── swe-substrate-observability.md
│   ├── swe-substrate-data.md
│   ├── swe-substrate-security.md
│   └── swe-substrate-documentation.md
│
└── swe-substrate/         # Contexto modular (gerado pelos agentes)
    ├── project-context.md
    ├── tooling-config.md
    ├── languages.md
    ├── api-design.md
    ├── testing-strategy.md
    ├── agents.md
    ├── package-mgmt.md
    ├── observability.md
    ├── data.md
    ├── security.md
    └── documentation.md
```

## Comandos Essenciais

### Orquestrador
```bash
/swe-substrate-init
```
Lista todos os agentes disponíveis e explica como usar o sistema.

### Agentes Especializados
```bash
/swe-substrate-project-context     # Visão, domínio, negócio, métricas
/swe-substrate-tooling-config      # Docker, IaC, Env Vars
/swe-substrate-languages           # Código idiomático, concorrência
/swe-substrate-api-design          # REST, GraphQL, RPC
/swe-substrate-testing-strategy    # Testes unitários, integração, E2E
/swe-substrate-agents              # Daemons, workers, LLM agents
/swe-substrate-package-mgmt        # Dependências e versionamento
/swe-substrate-observability       # Logs, métricas, traces
/swe-substrate-data                # Databases, cache, storage
/swe-substrate-security            # Autenticação, autorização
/swe-substrate-documentation       # Comentários, diagramas, ADRs
```

## Contexto Modular

A documentação detalhada gerada pelos agentes é modularizada em `.claude/swe-substrate/`:

- **[Contexto do Projeto](.claude/swe-substrate/project-context.md)** - Visão geral, domínio, negócio, métricas
- **[Ferramental e Configuração](.claude/swe-substrate/tooling-config.md)** - Docker, IaC, variáveis de ambiente
- **[Linguagens e Idiomática](.claude/swe-substrate/languages.md)** - Convenções de código, concorrência
- **[Design de API](.claude/swe-substrate/api-design.md)** - REST, GraphQL, gRPC
- **[Estratégia de Testes](.claude/swe-substrate/testing-strategy.md)** - Unitários, integração, E2E
- **[Agentes e Background Jobs](.claude/swe-substrate/agents.md)** - Daemons, workers, LLM agents
- **[Gerenciamento de Pacotes](.claude/swe-substrate/package-mgmt.md)** - Dependências, versionamento
- **[Observabilidade](.claude/swe-substrate/observability.md)** - Logs, métricas, traces
- **[Dados e Persistência](.claude/swe-substrate/data.md)** - Databases, cache, storage
- **[Segurança](.claude/swe-substrate/security.md)** - Autenticação, autorização, secrets
- **[Documentação](.claude/swe-substrate/documentation.md)** - Comentários, diagramas, ADRs

## Arquitetura dos Agentes

Cada agente segue um padrão de 4 fases:

1. **Análise Automática** - Escaneia o codebase, detecta frameworks e padrões
2. **Perguntas Abertas** - Faz perguntas contextualizadas, uma por vez
3. **Geração da Seção** - Combina análise + respostas em markdown formatado
4. **Integração Modular** - Cria/atualiza arquivo específico em `.claude/swe-substrate/`

## Princípios de Design

- **Modularidade**: Agentes independentes e opcionais
- **Perguntas Abertas**: Maximizar informação coletada
- **Análise Inteligente**: Detectar automaticamente o que for possível
- **Conversacional**: Interação natural com o usuário
- **Flexibilidade**: Não obrigar uso de todos os agentes
- **Distribuível**: Fácil de copiar para outros projetos
- **Escalável**: Documentação modularizada que cresce sem limites

## Uso em Outros Projetos

Para usar estes agentes em outros projetos:

1. Copie a pasta `.claude/` para o projeto alvo
2. Execute `/swe-substrate-init` para ver agentes disponíveis
3. Execute os agentes relevantes para o projeto
4. Responda as perguntas com contexto do projeto
5. Documentação modular será gerada em `.claude/swe-substrate/`
6. O CLAUDE.md do projeto alvo será atualizado com links para os módulos
