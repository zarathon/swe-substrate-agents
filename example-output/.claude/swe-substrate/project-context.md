# Contexto do Projeto

**Gerado por**: `/swe-substrate-project-context`
**Data**: 2025-10-29

## Visão Geral

**Nome do Projeto**: ShopFlow
**Domínio**: E-commerce B2C
**Propósito**: Plataforma de e-commerce moderna focada em experiência de checkout rápido e personalização de produtos

### Problema que Resolve

Muitas plataformas de e-commerce têm processos de checkout complexos e lentos, resultando em abandono de carrinho (~70% no mercado). O ShopFlow oferece:

- Checkout em uma única página com validação em tempo real
- Personalização de produtos (gravação, cores customizadas)
- Recomendações baseadas em ML para cross-sell
- Integração nativa com múltiplos gateways de pagamento

## Domínio de Negócio

### Entidades Principais

1. **Product** - Produtos do catálogo (físicos e digitais)
2. **Cart** - Carrinho de compras com itens temporários
3. **Order** - Pedido confirmado e pago
4. **Customer** - Cliente registrado ou guest
5. **Payment** - Transação de pagamento
6. **Shipment** - Envio e rastreamento de produtos físicos
7. **Customization** - Personalizações aplicadas aos produtos

### Regras de Negócio Críticas

- Carrinho expira após 24h de inatividade
- Estoque é reservado por 15 minutos após início do checkout
- Preços podem ter promoções com data de validade
- Produtos digitais são entregues imediatamente após pagamento confirmado
- Produtos físicos requerem validação de endereço antes da compra
- Cada customização pode ter custo adicional e tempo de produção

## Métricas de Sucesso

### Métricas de Negócio

- **Conversion Rate**: Taxa de conversão de visitas em vendas (meta: >3.5%)
- **Cart Abandonment Rate**: Taxa de abandono de carrinho (meta: <50%)
- **Average Order Value (AOV)**: Valor médio de pedido (meta: R$ 250+)
- **Customer Lifetime Value (CLV)**: Valor vitalício do cliente
- **Repeat Purchase Rate**: Taxa de recompra (meta: >25%)

### Métricas Técnicas

- **Checkout Page Load Time**: Tempo de carregamento da página de checkout (meta: <1.5s)
- **API Response Time (P95)**: Tempo de resposta das APIs (meta: <200ms)
- **Payment Success Rate**: Taxa de sucesso de pagamentos (meta: >97%)
- **Search Relevance Score**: Qualidade dos resultados de busca
- **Recommendation Click-Through Rate**: Taxa de clique em recomendações (meta: >8%)

## Arquitetura de Alto Nível

### Stack Principal

- **Backend**: Node.js + TypeScript + Express
- **Frontend**: React + Next.js (SSR/SSG para SEO)
- **Mobile**: React Native
- **Database**: PostgreSQL (principal), Redis (cache/sessões)
- **Search**: Elasticsearch
- **Storage**: S3 para imagens de produtos
- **Messaging**: RabbitMQ para processamento assíncrono

### Padrões Arquiteturais

- **Microserviços**: Serviços separados para Catalog, Cart, Orders, Payments, Recommendations
- **Event-Driven**: Eventos de domínio via RabbitMQ (OrderPlaced, PaymentConfirmed, etc.)
- **CQRS Lite**: Separação de comandos e queries em serviços críticos (Orders, Payments)
- **API Gateway**: Kong para roteamento, rate limiting e autenticação
- **BFF (Backend for Frontend)**: APIs específicas para Web e Mobile

## Contexto de Desenvolvimento

### Time

- 3 desenvolvedores backend
- 2 desenvolvedores frontend
- 1 DevOps
- 1 Product Manager

### Ciclo de Release

- Sprints de 2 semanas
- Deploy contínuo em staging
- Deploy em produção: terças e quintas às 10h (horário de menor tráfego)
- Feature flags para releases graduais

### Ambientes

- **Development**: Local (Docker Compose)
- **Staging**: AWS ECS (espelho de produção)
- **Production**: AWS ECS (multi-AZ)

## Integrações Externas

- **Pagamentos**: Stripe, PayPal, Mercado Pago
- **Envios**: Correios API, Loggi, Shopify Logistics
- **Email**: SendGrid para transacionais, Mailchimp para marketing
- **Analytics**: Google Analytics, Mixpanel
- **Monitoramento**: Datadog
- **Error Tracking**: Sentry
