# ShopFlow - Exemplo de Documentação Gerada

Este diretório contém um **exemplo fictício** da documentação que seria gerada pelos **SWE Substrate Agents** para um projeto de e-commerce chamado **ShopFlow**.

## 📁 Estrutura

```
example-output/
└── .claude/
    └── swe-substrate/
        ├── project-context.md      # Visão geral, domínio, métricas
        ├── api-design.md          # REST, GraphQL, gRPC
        ├── testing-strategy.md    # Testes unitários, integração, E2E
        ├── data.md                # PostgreSQL, Redis, Elasticsearch, S3
        ├── security.md            # JWT, OAuth, RBAC, Secrets
        ├── languages.md           # TypeScript, convenções, concorrência
        ├── observability.md       # Logs, métricas, traces, alertas
        └── tooling-config.md      # Docker, Terraform, K8s, CI/CD
```

## 🎯 Propósito

Este exemplo demonstra:

1. **Formato e estrutura** da documentação gerada
2. **Nível de detalhe** que os agentes coletam
3. **Organização modular** por domínio técnico
4. **Como a documentação ajuda** o Claude Code a entender o projeto

## 🏪 Sobre o Projeto Fictício (ShopFlow)

**ShopFlow** é uma plataforma de e-commerce B2C moderna com:

- **Stack**: TypeScript/Node.js, React/Next.js, PostgreSQL, Redis
- **Arquitetura**: Microserviços com Event-Driven patterns
- **Foco**: Checkout rápido e personalização de produtos
- **Integrações**: Stripe, PayPal, SendGrid, Datadog

### Características Principais

- Checkout em página única
- Personalização de produtos (gravação, cores)
- Recomendações baseadas em ML
- Multi-gateway de pagamento
- Sistema de inventário em tempo real

## 📖 Navegando pela Documentação

### 1. Contexto do Projeto (`project-context.md`)
- Visão geral do negócio
- Domínio e entidades principais
- Métricas de sucesso (técnicas e de negócio)
- Arquitetura de alto nível

### 2. Design de API (`api-design.md`)
- Padrões REST (versionamento, paginação, filtros)
- GraphQL schema e convenções
- gRPC para comunicação interna
- Autenticação e formato de respostas

### 3. Estratégia de Testes (`testing-strategy.md`)
- Pirâmide de testes (60/30/10)
- Testes unitários com Jest
- Testes de integração com Testcontainers
- E2E com Playwright
- Performance com k6

### 4. Dados e Persistência (`data.md`)
- PostgreSQL schema e migrations
- Redis para cache e sessions
- Elasticsearch para busca
- S3 para storage
- Backup e disaster recovery

### 5. Segurança (`security.md`)
- JWT com RS256
- OAuth 2.0 social login
- RBAC e permissões
- Secrets management (AWS Secrets Manager)
- Rate limiting e DDoS protection

### 6. Linguagens e Idiomática (`languages.md`)
- TypeScript strict mode
- Convenções de código
- Padrões de concorrência
- Error handling
- Performance best practices

### 7. Observabilidade (`observability.md`)
- Structured logging com Winston
- Métricas com Datadog
- Distributed tracing
- Error tracking com Sentry
- Health checks e alertas

### 8. Ferramental e Configuração (`tooling-config.md`)
- Docker Compose para dev
- Terraform para infraestrutura
- Kubernetes deployments
- GitHub Actions CI/CD
- Environment variables

## 🎨 Como Este Exemplo Foi Criado

Este exemplo foi gerado **manualmente** para demonstrar o que os agentes produziriam após:

1. **Análise automática** do codebase (frameworks, estrutura, padrões)
2. **Perguntas abertas** ao desenvolvedor sobre decisões de design
3. **Combinação** de análise + respostas em documentação estruturada

## 🔧 Como Usar no Seu Projeto

Para gerar documentação similar para o **seu projeto**:

1. Copie a pasta `.claude/` do repositório principal para seu projeto
2. Execute `/swe-substrate-init` para ver os agentes disponíveis
3. Execute os agentes relevantes (ex: `/swe-substrate-api-design`)
4. Responda as perguntas sobre seu projeto
5. A documentação será gerada em `.claude/swe-substrate/`

## 📊 Benefícios da Documentação Modular

### Para o Claude Code
- **Contexto rico** sobre arquitetura e padrões
- **Decisões de design** documentadas
- **Convenções** do projeto claramente definidas
- **Integrações** e dependências mapeadas

### Para o Time
- **Onboarding** mais rápido de novos membros
- **Referência** centralizada de padrões
- **Decisões** arquiteturais documentadas (ADRs implícitos)
- **Alinhamento** entre desenvolvedores

## 🚀 Próximos Passos

Se você gostou deste exemplo e quer usar os agentes:

1. Clone o repositório principal
2. Siga as instruções no README.md principal
3. Adapte os agentes para suas necessidades
4. Contribua com melhorias!

## 📝 Notas

- Este é um **projeto fictício** criado apenas para demonstração
- Os valores, métricas e configurações são **exemplos realistas** mas não reais
- A documentação real gerada para seu projeto será **específica ao seu contexto**

---

**Dúvidas?** Consulte o README.md principal do repositório ou abra uma issue!
