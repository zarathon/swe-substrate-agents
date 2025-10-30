# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Beta v0.2] - 2025-10-30

### 🌍 Bilingual Support

Major release adding full bilingual support to the project.

#### Added
- 🇺🇸 English as primary language throughout the project
- 🇧🇷 Portuguese as secondary language
- Bilingual landing page with language switcher (EN/PT)
- `README.md` in English (new primary documentation)
- `README.pt-BR.md` in Portuguese (moved from README.md)
- Language preference persistence via localStorage
- Comprehensive i18n translation dictionary (100+ strings)
- Language switcher button in landing page header

#### Changed
- **Landing page**: Now loads in English by default
- **Documentation structure**: English-first approach
- **CLAUDE.md**: Updated with bilingual guidelines
- **README**: Split into English (primary) and Portuguese (secondary)

#### Technical
- i18n system with translation keys
- Automatic language detection
- Smooth language switching without page reload
- Language preference saved in browser

#### Documentation
- English documentation is now primary
- Portuguese documentation fully maintained
- Both languages have 100% feature parity
- Bilingual support noted in all docs

### 📈 International Reach
- Expanded potential audience to 1.5+ billion users
- Support for English-speaking markets (US, UK, Canada, Australia, India)
- Support for Portuguese-speaking markets (Brazil, Portugal)

### 🔄 No Breaking Changes
- Fully backward compatible with v0.1
- All agents remain the same
- Same modular structure
- Same installation process

---

## [Beta v0.1] - 2025-10-29

### Lançamento Inicial

Primeira versão beta do **SWE Substrate Agents** - um sistema inovador de agentes especializados para geração de documentação modularizada otimizada para Claude Code.

### Características Principais

#### 🤖 Sistema de Agentes Especializados
- Sistema modular com 11 agentes especializados
- Cada agente foca em um aspecto específico do projeto de software
- Arquitetura de 4 fases: Análise Automática → Perguntas Abertas → Geração → Salvamento

#### 📁 Documentação Modular
- Documentação gerada em arquivos separados em `.claude/swe-substrate/`
- Evita arquivos CLAUDE.md monolíticos gigantes
- Facilita manutenção, navegação e reduz conflitos em PRs
- Escalável sem limites de tamanho

#### 🎯 Agentes Incluídos

1. **`/swe-substrate-init`** - Orquestrador
   - Lista todos os agentes disponíveis
   - Guia de uso do sistema

2. **`/swe-substrate-project-context`** - Contexto do Projeto
   - Visão, domínio, negócio, métricas, stakeholders

3. **`/swe-substrate-tooling-config`** - Ferramental e Configuração
   - Docker, IaC, variáveis de ambiente, scripts

4. **`/swe-substrate-languages`** - Linguagens e Idiomática
   - Convenções de código, concorrência, memória, guidelines

5. **`/swe-substrate-api-design`** - Design de API
   - REST, GraphQL, gRPC, versionamento, contratos

6. **`/swe-substrate-testing-strategy`** - Estratégia de Testes
   - Unitários, integração, E2E, coverage

7. **`/swe-substrate-agents`** - Agentes e Background Jobs
   - Daemons, workers, background jobs, LLM agents

8. **`/swe-substrate-package-mgmt`** - Gerenciamento de Pacotes
   - npm, pip, cargo, dependências, versionamento

9. **`/swe-substrate-observability`** - Observabilidade
   - Logs, métricas, traces, monitoring, alertas

10. **`/swe-substrate-data`** - Dados e Persistência
    - Databases, schema, migrations, cache, storage

11. **`/swe-substrate-security`** - Segurança
    - Autenticação, autorização, secrets, certificados

12. **`/swe-substrate-documentation`** - Documentação
    - Comentários, diagramas, ADRs, knowledge base

### 🚀 Como Começar

1. Copie a pasta `.claude/` para seu projeto
2. Execute `/swe-substrate-init` para ver agentes disponíveis
3. Execute os agentes relevantes para seu contexto
4. Responda as perguntas abertas
5. Documentação modular será gerada automaticamente

### 📦 Exemplo Incluído

- Documentação completa de exemplo para projeto fictício "ShopFlow"
- Demonstra todas as capacidades do sistema
- Disponível em `example-output/`

### ✨ Diferenciais

- **Modularidade**: Execute apenas o necessário
- **Inteligência**: Análise automática + perguntas contextuais
- **Conversacional**: Interação natural e fluida
- **Flexível**: Não obriga uso de todos os agentes
- **Distribuível**: Copie para qualquer projeto
- **Escalável**: Documentação cresce sem limites

### 🔧 Limitações Conhecidas (Beta)

- Versão inicial para feedback da comunidade
- Agentes podem precisar de ajustes conforme diferentes stacks
- Formato das perguntas pode ser refinado
- Documentação de uso pode ser expandida

### 📝 Próximos Passos

- [ ] Coletar feedback de usuários beta
- [ ] Refinar perguntas dos agentes baseado em uso real
- [ ] Adicionar mais exemplos de documentação gerada
- [ ] Melhorar detecção automática de frameworks
- [ ] Criar templates para stacks populares (Node.js, Python, Go, Rust)
- [ ] Adicionar validação de documentação gerada

### 🎯 Público-Alvo

- Times de engenharia que usam Claude Code
- Tech Leads e Engineering Managers
- Desenvolvedores que querem documentação consistente
- Empresas que adotam IA no desenvolvimento

### 📄 Licença

Uso livre para qualquer organização ou projeto de software.

---

**Agradecimentos**: À comunidade Claude Code por inspirar este projeto e à Anthropic por criar ferramentas incríveis.
