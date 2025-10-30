# SWE Substrate Agents

**[🇧🇷 Português](README.pt-BR.md)**

A system of specialized agents that generate **modular documentation** optimized for software projects.

## 🎯 Objective

This project provides a set of intelligent agents that collect information about software projects through open-ended questions and automatic code analysis, generating modular documentation in `.claude/swe-substrate/` optimized for use with Claude Code.

## 🚨 The Critical Problem

**Claude Code is only powerful when it deeply understands your project.**

Without structured context about your engineering substrate, Claude Code:
- ❌ Generates generic code that doesn't compile
- ❌ Suggests libraries you don't use
- ❌ Ignores team conventions
- ❌ Creates tests that don't run
- ❌ Proposes incompatible architectures

**With structured substrate:**
- ✅ Claude generates idiomatic code that works
- ✅ Uses the correct project libraries
- ✅ Follows all established conventions
- ✅ Creates tests that run immediately
- ✅ Proposes solutions aligned with architecture

## 🧬 What is Engineering Substrate?

The **fundamental layers** that define how your software is built, tested, deployed, and maintained. It's your project's DNA.

## 🤖 Available Agents

Each agent specializes in a specific aspect of the project:

1. **`/swe-substrate-project-context`** - Project Context
   - Business vision, domain, metrics, stakeholders

2. **`/swe-substrate-tooling-config`** - Tooling & Configuration
   - Docker, IaC, environment variables, setup scripts

3. **`/swe-substrate-languages`** - Languages & Idioms
   - Code conventions, concurrency, memory, guidelines

4. **`/swe-substrate-api-design`** - API Design
   - REST, GraphQL, gRPC, versioning, contracts

5. **`/swe-substrate-testing-strategy`** - Testing Strategy
   - Unit, integration, E2E tests, coverage

6. **`/swe-substrate-agents`** - Agents & Background Processes
   - Daemons, workers, background jobs, LLM agents

7. **`/swe-substrate-package-mgmt`** - Package Management
   - npm, pip, cargo, dependencies, versioning

8. **`/swe-substrate-observability`** - Observability
   - Logs, metrics, traces, monitoring, alerts

9. **`/swe-substrate-data`** - Data & Persistence
   - Databases, schemas, migrations, cache, storage

10. **`/swe-substrate-security`** - Security
    - Authentication, authorization, secrets, certificates

11. **`/swe-substrate-documentation`** - Documentation
    - Comments, diagrams, ADRs, knowledge base

## 📁 Modular Structure

Generated documentation is **modularized** into separate files:

```
project/
├── CLAUDE.md                      # Main index (lightweight)
└── .claude/
    ├── commands/                  # Agents (ask questions)
    │   ├── swe-substrate-init.md
    │   ├── swe-substrate-tooling-config.md
    │   └── ...
    │
    └── swe-substrate/             # Modular documentation (generated)
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

**Advantages of modularization:**
- ✅ No giant monolithic file
- ✅ Easy maintenance and navigation
- ✅ Fewer PR conflicts
- ✅ Scalable documentation
- ✅ Each module is independent

## 🚀 How to Use

### Quick Start

1. **Copy the `.claude/` folder** to your project
2. Run the orchestrator to see all available agents:
   ```bash
   /swe-substrate-init
   ```
3. Run only the relevant agents for your project:
   ```bash
   /swe-substrate-languages
   /swe-substrate-testing-strategy
   /swe-substrate-data
   ```

### Recommended Workflow

#### For New Projects

Start with the fundamentals:
- `/swe-substrate-project-context` - Business context and overview
- `/swe-substrate-tooling-config` - Initial setup
- `/swe-substrate-languages` - Code conventions
- `/swe-substrate-testing-strategy` - Quality from the start

#### For Existing Projects

Run as needed:
- Add `/swe-substrate-security` when documenting security practices
- Use `/swe-substrate-observability` when configuring monitoring
- Run `/swe-substrate-data` when adding persistence
- Use `/swe-substrate-api-design` when creating APIs

### How Each Agent Works

Each agent follows 4 phases:

1. **Automatic Analysis** 🔍
   - Scans project code
   - Detects patterns, frameworks, configurations
   - Identifies information that can be inferred

2. **Open-Ended Questions** 💬
   - Asks specific questions to the engineer
   - Collects information that cannot be automatically detected
   - Contextualized questions based on analysis

3. **Section Generation** 📝
   - Combines automatic analysis + answers
   - Generates structured markdown documentation
   - Consistent and clear formatting

4. **Modular Storage** 💾
   - Saves to specific file in `.claude/swe-substrate/`
   - E.g.: `/swe-substrate-languages` → `languages.md`
   - Organized and scalable documentation

## ✨ Features

- **Modular**: Run only the relevant agents for your project
- **Modularized Documentation**: Separate files in `.claude/swe-substrate/`
- **Intelligent**: Combines automatic analysis with contextual questions
- **Conversational**: Open-ended questions that maximize collected information
- **Scalable**: Documentation that grows without limits
- **Flexible**: Doesn't force use of all agents
- **Distributable**: Copy to any project and use immediately

## 📦 Distribution

To use in your projects:

1. Copy the `.claude/` folder to your project repository
2. Commit and share with the team
3. Each member can run the agents as needed

## 🎓 For Engineering Teams

This system was designed to be distributed among engineering teams, allowing each team to:

- Generate consistent, high-quality CLAUDE.md
- Document project-specific practices and conventions
- Improve Claude Code performance on their projects
- Onboard new members with clear documentation

## 🌍 Bilingual Support

This project fully supports English and Portuguese:
- 🇺🇸 English (primary)
- 🇧🇷 Português (secondary)

**Landing Page**: Bilingual with language switcher
**Documentation**: Available in both languages
**Agents**: Currently in Portuguese (English versions coming soon)

## 🤝 Contributing

To add new agents or improve existing ones:

1. Create a new file in `.claude/commands/swe-substrate-{name}.md`
2. Follow the structure of existing agents
3. Include: automatic analysis, open-ended questions, section generation
4. Update the orchestrator in `/swe-substrate-init`

## 📝 License

Free to use for any organization or software project.

## 🔗 Links

- **Website**: https://zarathon.github.io/swe-substrate-agents/
- **GitHub**: https://github.com/zarathon/swe-substrate-agents
- **Releases**: https://github.com/zarathon/swe-substrate-agents/releases
- **Issues**: https://github.com/zarathon/swe-substrate-agents/issues

---

**Built with Claude Code** 🤖

⭐ Star on GitHub if this project was helpful to you!
