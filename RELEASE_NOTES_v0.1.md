# 🚀 SWE Substrate Agents - Beta v0.1

**Data de Lançamento**: 29 de Outubro de 2025

## 🎉 Lançamento Inicial

Estamos felizes em anunciar a primeira versão beta do **SWE Substrate Agents** - um sistema inovador de agentes especializados que revoluciona a forma como times de engenharia documentam seus projetos para Claude Code.

## 🤔 O Que É?

Um conjunto de 11 agentes inteligentes que coletam informações sobre seu projeto através de **perguntas abertas** e **análise automática de código**, gerando documentação **modularizada** em `.claude/swe-substrate/` otimizada para Claude Code.

## ✨ Por Que Usar?

### ❌ Problema: CLAUDE.md Monolítico
- Arquivo gigante difícil de manter
- Conflitos constantes em PRs
- Difícil de navegar e atualizar
- Informações desorganizadas

### ✅ Solução: Documentação Modular
- Arquivos separados por domínio
- Fácil manutenção e navegação
- Menos conflitos em PRs
- Documentação escalável sem limites

## 🤖 11 Agentes Especializados

Cada agente é expert em um aspecto específico:

| Agente | Foco | Gera |
|--------|------|------|
| `/swe-substrate-project-context` | Contexto & Negócio | `project-context.md` |
| `/swe-substrate-tooling-config` | Docker, IaC, Env | `tooling-config.md` |
| `/swe-substrate-languages` | Código Idiomático | `languages.md` |
| `/swe-substrate-api-design` | REST, GraphQL, gRPC | `api-design.md` |
| `/swe-substrate-testing-strategy` | Testes & Coverage | `testing-strategy.md` |
| `/swe-substrate-agents` | Background Jobs | `agents.md` |
| `/swe-substrate-package-mgmt` | Dependências | `package-mgmt.md` |
| `/swe-substrate-observability` | Logs, Métricas | `observability.md` |
| `/swe-substrate-data` | Databases, Cache | `data.md` |
| `/swe-substrate-security` | Auth, Secrets | `security.md` |
| `/swe-substrate-documentation` | Docs, ADRs | `documentation.md` |

## 🚀 Como Começar (3 minutos)

```bash
# 1. Copie .claude/ para seu projeto
cp -r .claude /seu-projeto/

# 2. No Claude Code, execute o orquestrador
/swe-substrate-init

# 3. Execute agentes relevantes
/swe-substrate-languages
/swe-substrate-testing-strategy
/swe-substrate-data

# 4. Responda as perguntas
# 5. Documentação modular gerada automaticamente! ✨
```

## 🎯 Características Principais

### 🔍 Análise Automática
- Detecta frameworks e bibliotecas
- Identifica padrões de código
- Escaneia configurações
- Analisa estrutura do projeto

### 💬 Perguntas Abertas
- Perguntas contextualizadas
- Maximizam informação coletada
- Conversação natural
- Uma pergunta por vez

### 📝 Geração Inteligente
- Combina análise + respostas
- Markdown estruturado
- Formatação consistente
- Exemplos de código quando relevante

### 💾 Salvamento Modular
- Cada agente = 1 arquivo
- Organização por domínio
- Escalável sem limites
- Fácil de manter

## 📦 O Que Está Incluído?

```
.claude/
├── commands/              # 12 agentes (slash commands)
│   ├── swe-substrate-init.md
│   └── swe-substrate-*.md (11 agentes especializados)
│
└── swe-substrate/         # Documentação modular (será gerada)
    └── (criado pelos agentes)
```

## 🎓 Exemplo Completo

Incluímos documentação completa de exemplo para um projeto fictício **"ShopFlow"** (e-commerce):
- Demonstra todas as capacidades
- Serve como template
- Disponível em `example-output/`

## 🎯 Para Quem É?

- ✅ Times que usam Claude Code
- ✅ Tech Leads e Engineering Managers
- ✅ Empresas adotando IA no desenvolvimento
- ✅ Desenvolvedores que querem documentação consistente
- ✅ Times distribuídos que precisam de contexto claro

## 🔧 Limitações Conhecidas (Beta)

Esta é uma versão beta para coletar feedback:
- Agentes podem precisar ajustes para diferentes stacks
- Formato das perguntas pode ser refinado
- Detecção automática pode ser expandida

**Seu feedback é crucial!** Por favor, reporte issues ou sugestões.

## 🛣️ Roadmap

### v0.2 (Próxima Versão)
- [ ] Templates para stacks populares (Node, Python, Go, Rust)
- [ ] Melhorias na detecção automática
- [ ] Mais exemplos de documentação gerada
- [ ] Validação da documentação gerada

### v1.0 (Estável)
- [ ] Agentes refinados com feedback da comunidade
- [ ] Suporte a mais frameworks
- [ ] Integração com ferramentas de CI/CD
- [ ] Dashboard de métricas de documentação

## 📄 Instalação

### Opção 1: Clone o Repositório
```bash
git clone https://github.com/[seu-usuario]/swe-substrate-agents.git
cd swe-substrate-agents
```

### Opção 2: Download ZIP
Baixe o ZIP e extraia a pasta `.claude/` para seu projeto.

### Opção 3: Copie Manualmente
Copie apenas a pasta `.claude/` para o root do seu projeto.

## 🤝 Contribuindo

Contribuições são bem-vindas! Para contribuir:

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/NovoAgente`)
3. Commit suas mudanças (`git commit -m 'Add: novo agente para X'`)
4. Push para a branch (`git push origin feature/NovoAgente`)
5. Abra um Pull Request

## 📞 Suporte

- **Issues**: [GitHub Issues](https://github.com/[seu-usuario]/swe-substrate-agents/issues)
- **Discussões**: [GitHub Discussions](https://github.com/[seu-usuario]/swe-substrate-agents/discussions)
- **Documentação**: [README.md](README.md)

## 📝 Licença

Uso livre para qualquer organização ou projeto de software.

---

**Desenvolvido com Claude Code** 🤖

Ajude-nos a melhorar! ⭐ Dê uma estrela no GitHub se este projeto foi útil para você.
