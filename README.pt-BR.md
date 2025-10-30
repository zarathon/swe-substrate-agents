# SWE Substrate Agents

**[🇺🇸 English](README.md)**

Sistema de agentes especializados para gerar documentação **modularizada** otimizada para projetos de software.

## 🎯 Objetivo

Este projeto fornece um conjunto de agentes inteligentes que coletam informações sobre projetos de software através de perguntas abertas e análise automática de código, gerando documentação modular em `.claude/swe-substrate/` otimizada para uso com Claude Code.

## 🤖 Agentes Disponíveis

Cada agente é especializado em um aspecto específico do projeto:

1. **`/swe-substrate-project-context`** - Contexto do Projeto
   - Visão geral, domínio, negócio, métricas, stakeholders

2. **`/swe-substrate-tooling-config`** - Ferramental e Configuração
   - Docker, IaC, variáveis de ambiente, scripts de setup

3. **`/swe-substrate-languages`** - Linguagens e Idiomática
   - Convenções de código, concorrência, memória, guidelines

4. **`/swe-substrate-api-design`** - Design de API
   - REST, GraphQL, gRPC, versionamento, contratos

5. **`/swe-substrate-testing-strategy`** - Estratégia de Testes
   - Testes unitários, integração, E2E, coverage

6. **`/swe-substrate-agents`** - Agentes e Processos Background
   - Daemons, workers, background jobs, LLM agents

7. **`/swe-substrate-package-mgmt`** - Gerenciamento de Pacotes
   - npm, pip, cargo, dependências, versionamento

8. **`/swe-substrate-observability`** - Observabilidade
   - Logs, métricas, traces, monitoring, alertas

9. **`/swe-substrate-data`** - Dados e Persistência
   - Databases, schema, migrations, cache, storage

10. **`/swe-substrate-security`** - Segurança
    - Autenticação, autorização, secrets, certificados

11. **`/swe-substrate-documentation`** - Documentação
    - Comentários, diagramas, ADRs, knowledge base

## 📁 Estrutura Modular

A documentação gerada é **modularizada** em arquivos separados:

```
projeto/
├── CLAUDE.md                      # Índice principal (leve)
└── .claude/
    ├── commands/                  # Agentes (fazem perguntas)
    │   ├── swe-substrate-init.md
    │   ├── swe-substrate-tooling-config.md
    │   └── ...
    │
    └── swe-substrate/             # Documentação modular (gerada)
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

**Vantagens da modularização:**
- ✅ Sem arquivo gigante monolítico
- ✅ Fácil manutenção e navegação
- ✅ Menos conflitos em PRs
- ✅ Documentação escalável
- ✅ Cada módulo é independente

## 🚀 Como Usar

### Início Rápido

1. **Copie a pasta `.claude/`** para o seu projeto
2. Execute o orquestrador para ver todos os agentes disponíveis:
   ```bash
   /swe-substrate-init
   ```
3. Execute apenas os agentes relevantes para seu projeto:
   ```bash
   /swe-substrate-languages
   /swe-substrate-testing-strategy
   /swe-substrate-data
   ```

### Workflow Recomendado

#### Para Novos Projetos

Comece com os fundamentos:
- `/swe-substrate-project-context` - Contexto de negócio e visão geral
- `/swe-substrate-tooling-config` - Setup inicial
- `/swe-substrate-languages` - Convenções de código
- `/swe-substrate-testing-strategy` - Qualidade desde o início

#### Para Projetos Existentes

Execute conforme a necessidade:
- Adicione `/swe-substrate-security` ao documentar práticas de segurança
- Use `/swe-substrate-observability` ao configurar monitoramento
- Execute `/swe-substrate-data` ao adicionar persistência
- Use `/swe-substrate-api-design` ao criar APIs

### Como Cada Agente Funciona

Cada agente segue 4 fases:

1. **Análise Automática** 🔍
   - Escaneia o código do projeto
   - Detecta padrões, frameworks, configurações
   - Identifica informações que podem ser inferidas

2. **Perguntas Abertas** 💬
   - Faz perguntas específicas ao engenheiro
   - Coleta informações que não podem ser detectadas automaticamente
   - Perguntas contextualizadas baseadas na análise

3. **Geração da Seção** 📝
   - Combina análise automática + respostas
   - Gera documentação markdown estruturada
   - Formatação consistente e clara

4. **Salvamento Modular** 💾
   - Salva em arquivo específico em `.claude/swe-substrate/`
   - Ex: `/swe-substrate-languages` → `languages.md`
   - Documentação organizada e escalável

## ✨ Características

- **Modular**: Execute apenas os agentes relevantes para seu projeto
- **Documentação Modularizada**: Arquivos separados em `.claude/swe-substrate/`
- **Inteligente**: Combina análise automática com perguntas contextuais
- **Conversacional**: Perguntas abertas que maximizam informações coletadas
- **Escalável**: Documentação que cresce sem limites
- **Flexível**: Não obriga uso de todos os agentes
- **Distribuível**: Copie para qualquer projeto e use imediatamente

## 📦 Distribuição

Para usar em seus projetos:

1. Copie a pasta `.claude/` para o repositório do seu projeto
2. Commit e compartilhe com o time
3. Cada membro pode executar os agentes conforme necessidade

## 🎓 Para Times de Engenharia

Este sistema foi projetado para ser distribuído entre times de engenharia, permitindo que cada time:

- Gere CLAUDE.md consistente e de alta qualidade
- Documente práticas e convenções específicas do projeto
- Melhore a performance do Claude Code nos seus projetos
- Onboarde novos membros com documentação clara

## 🤝 Contribuindo

Para adicionar novos agentes ou melhorar os existentes:

1. Crie um novo arquivo em `.claude/commands/swe-substrate-{nome}.md`
2. Siga a estrutura dos agentes existentes
3. Inclua: análise automática, perguntas abertas, geração de seção
4. Atualize o orquestrador em `/swe-substrate-init`

## 📝 Licença

Este projeto pode ser usado livremente em qualquer organização ou projeto de software.
