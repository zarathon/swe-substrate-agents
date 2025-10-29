# 🌐 Como Ativar o GitHub Pages

A landing page já está pronta e no repositório! Agora você só precisa ativar o GitHub Pages nas configurações.

## ✅ O Que Já Foi Feito

- ✅ Landing page moderna criada em `docs/index.html`
- ✅ Estrutura otimizada para GitHub Pages
- ✅ Push para o repositório GitHub
- ✅ Design responsivo e profissional
- ✅ Zero dependências (HTML + CSS puro)

## 🚀 Passo a Passo para Ativar (2 minutos)

### Opção 1: Via Interface Web do GitHub (Recomendado)

1. **Acesse as configurações do repositório**
   ```
   https://github.com/zarathon/swe-substrate-agents/settings/pages
   ```

2. **Configure o GitHub Pages**
   - Na seção **"Build and deployment"**
   - Em **"Source"**, selecione: `Deploy from a branch`
   - Em **"Branch"**:
     - Branch: `main`
     - Folder: `/docs`
   - Clique em **"Save"**

3. **Aguarde o deploy**
   - O GitHub leva 1-3 minutos para fazer o primeiro deploy
   - Uma mensagem aparecerá: "Your site is live at..."

4. **Acesse sua landing page!**
   ```
   https://zarathon.github.io/swe-substrate-agents/
   ```

### Opção 2: Via GitHub CLI (Se tiver instalado)

```bash
# Habilitar GitHub Pages
gh api repos/zarathon/swe-substrate-agents/pages \
  --method POST \
  -f source[branch]=main \
  -f source[path]=/docs

# Verificar status
gh api repos/zarathon/swe-substrate-agents/pages
```

## 🎯 URL da Landing Page

Após ativação, sua landing page estará disponível em:

```
https://zarathon.github.io/swe-substrate-agents/
```

## 🎨 Preview da Landing Page

A landing page inclui:

- **Hero Section** - Título, subtítulo e CTAs
- **Estatísticas** - 11 agentes, 4 fases, 100% modular
- **Problema vs Solução** - Cards comparativos
- **Características** - 4 features principais
- **Grid de Agentes** - 11 agentes especializados
- **Quick Start** - 4 passos para começar
- **Público-alvo** - Para quem é o projeto
- **Footer** - Links e informações

## 🔧 Verificar Status do Deploy

Para verificar se o GitHub Pages foi ativado:

1. Vá em: https://github.com/zarathon/swe-substrate-agents/settings/pages
2. Se estiver ativado, você verá: "Your site is published at..."
3. O ícone ficará verde quando estiver pronto

## 📱 Testando Localmente

Se quiser testar antes:

```bash
# Via Python
cd docs
python3 -m http.server 8000

# Via Node.js
npx serve docs

# Acesse: http://localhost:8000
```

## 🎨 Design e Cores

A landing page usa uma paleta moderna:
- **Primary**: Azul (#2563eb)
- **Secondary**: Roxo (#8b5cf6)
- **Accent**: Ciano (#06b6d4)
- **Dark**: Azul escuro (#0f172a)
- **Totalmente responsivo** para mobile

## 🔗 Links Úteis

- **Documentação GitHub Pages**: https://docs.github.com/en/pages
- **Repositório**: https://github.com/zarathon/swe-substrate-agents
- **Issues**: https://github.com/zarathon/swe-substrate-agents/issues

## ✨ Próximos Passos Opcionais

### 1. Domínio Customizado (Opcional)

Se quiser usar um domínio próprio (ex: `swe-agents.com`):

1. Crie um arquivo `docs/CNAME`:
   ```
   seu-dominio.com
   ```

2. Configure DNS no seu provedor:
   ```
   Type: CNAME
   Name: www (ou @)
   Value: zarathon.github.io
   ```

3. Nas configurações do GitHub Pages, adicione o domínio customizado

### 2. Google Analytics (Opcional)

Adicione tracking antes do `</head>` no `docs/index.html`:

```html
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', 'G-XXXXXXXXXX');
</script>
```

### 3. SEO Melhorado (Opcional)

Adicione `docs/robots.txt`:
```
User-agent: *
Allow: /
Sitemap: https://zarathon.github.io/swe-substrate-agents/sitemap.xml
```

## 🎉 Pronto!

Após seguir os passos acima, sua landing page estará online e acessível publicamente!

**URL Final**: https://zarathon.github.io/swe-substrate-agents/

Compartilhe com seu time e comunidade! 🚀
