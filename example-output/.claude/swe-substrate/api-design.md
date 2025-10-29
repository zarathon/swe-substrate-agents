# Design de API

**Gerado por**: `/swe-substrate-api-design`
**Data**: 2025-10-29

## Padrões de API

### REST

**Arquitetura principal**: REST para operações CRUD e ações síncronas

#### Convenções de Nomenclatura

- **Resources**: Substantivos no plural (`/products`, `/orders`)
- **Actions**: Verbos HTTP padrão (GET, POST, PUT, PATCH, DELETE)
- **Sub-resources**: Relacionamentos aninhados quando apropriado
  - `/orders/{orderId}/items` - Itens de um pedido específico
  - `/customers/{customerId}/orders` - Pedidos de um cliente

#### Estrutura de URLs

```
GET    /api/v1/products                 # Lista produtos
GET    /api/v1/products/{id}            # Detalhes de produto
POST   /api/v1/products                 # Cria produto
PATCH  /api/v1/products/{id}            # Atualiza produto
DELETE /api/v1/products/{id}            # Remove produto

POST   /api/v1/orders/{id}/cancel       # Ação customizada
POST   /api/v1/payments/{id}/refund     # Ação de reembolso
```

#### Versionamento

- **Estratégia**: URL path versioning (`/api/v1`, `/api/v2`)
- **Política**: Manter 2 versões ativas simultaneamente
- **Deprecation**: 6 meses de aviso antes de descontinuar versão

#### Códigos de Status HTTP

- `200 OK`: Sucesso em GET, PATCH, DELETE
- `201 Created`: Sucesso em POST com novo recurso
- `204 No Content`: Sucesso sem corpo de resposta
- `400 Bad Request`: Erro de validação
- `401 Unauthorized`: Não autenticado
- `403 Forbidden`: Autenticado mas sem permissão
- `404 Not Found`: Recurso não encontrado
- `409 Conflict`: Conflito de estado (ex: produto já existe)
- `422 Unprocessable Entity`: Erro de validação de negócio
- `429 Too Many Requests`: Rate limit excedido
- `500 Internal Server Error`: Erro interno
- `503 Service Unavailable`: Serviço temporariamente indisponível

### Paginação

**Estratégia**: Cursor-based pagination para performance

```json
GET /api/v1/products?limit=20&cursor=eyJpZCI6MTIzNH0

{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6MTI1NH0",
    "has_more": true,
    "limit": 20
  }
}
```

**Fallback**: Offset-based para endpoints menos críticos

```json
GET /api/v1/orders?page=2&per_page=50

{
  "data": [...],
  "pagination": {
    "page": 2,
    "per_page": 50,
    "total_pages": 42,
    "total_count": 2087
  }
}
```

### Filtros e Ordenação

```
GET /api/v1/products?category=electronics&price_min=100&price_max=500&sort=-price,name
```

- **Filtros**: Query params com nome do campo
- **Ordenação**: Parâmetro `sort` com campos separados por vírgula
- **Direção**: Prefixo `-` para descendente, sem prefixo para ascendente

### Rate Limiting

- **Anônimo**: 100 requisições por minuto por IP
- **Autenticado**: 1000 requisições por minuto por usuário
- **Premium**: 5000 requisições por minuto

Headers de resposta:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 847
X-RateLimit-Reset: 1635789600
```

## GraphQL

**Uso**: API GraphQL para frontend (BFF pattern) para reduzir over-fetching

### Schema Principal

```graphql
type Query {
  product(id: ID!): Product
  products(filter: ProductFilter, limit: Int, cursor: String): ProductConnection
  cart: Cart
  order(id: ID!): Order
  recommendations(productId: ID, limit: Int): [Product!]!
}

type Mutation {
  addToCart(productId: ID!, quantity: Int!, customization: CustomizationInput): CartItem!
  updateCartItem(itemId: ID!, quantity: Int!): CartItem!
  removeFromCart(itemId: ID!): Boolean!
  checkout(input: CheckoutInput!): Order!
  cancelOrder(orderId: ID!, reason: String): Order!
}

type Subscription {
  orderStatusChanged(orderId: ID!): Order!
  cartUpdated: Cart!
}

type Product {
  id: ID!
  name: String!
  description: String
  price: Money!
  images: [Image!]!
  category: Category!
  customizationOptions: [CustomizationOption!]!
  inventory: Inventory!
  reviews(limit: Int): [Review!]!
  averageRating: Float
}
```

### Convenções

- **Naming**: camelCase para campos
- **Nullability**: Campos opcionais são nullable, obrigatórios com `!`
- **Connections**: Relay-style connections para listas paginadas
- **Errors**: Usar extensões GraphQL para erros estruturados

### DataLoader

Uso de DataLoader para batch e cache de queries:

```typescript
const productLoader = new DataLoader(async (ids) => {
  const products = await Product.findAll({ where: { id: ids } });
  return ids.map(id => products.find(p => p.id === id));
});
```

## gRPC

**Uso**: Comunicação interna entre microserviços para performance

### Services

```protobuf
service InventoryService {
  rpc CheckAvailability(AvailabilityRequest) returns (AvailabilityResponse);
  rpc ReserveStock(ReservationRequest) returns (ReservationResponse);
  rpc ReleaseStock(ReleaseRequest) returns (ReleaseResponse);

  // Server streaming para atualizações em tempo real
  rpc WatchInventory(WatchRequest) returns (stream InventoryUpdate);
}

message AvailabilityRequest {
  repeated string product_ids = 1;
  string warehouse_id = 2;
}

message AvailabilityResponse {
  map<string, int32> available_quantities = 1;
}
```

### Convenções

- **Naming**: snake_case para campos (protobuf convention)
- **Versioning**: Package versioning (`shopflow.inventory.v1`)
- **Timeouts**: 5s para operações síncronas, 30s para batch operations
- **Retry**: Exponential backoff com 3 tentativas

## Autenticação e Autorização

### JWT

- **Access Token**: 15 minutos de validade
- **Refresh Token**: 7 dias de validade (30 dias para "remember me")
- **Claims**: `user_id`, `email`, `roles`, `permissions`

### OAuth 2.0

Suporte para login social:
- Google
- Facebook
- Apple

### API Keys

Para integrações de terceiros:
- Prefixo: `sk_live_` (produção) ou `sk_test_` (desenvolvimento)
- Escopo: Permissões granulares por API key
- Rotação: Recomendado a cada 90 dias

## Formato de Respostas

### Success Response

```json
{
  "data": {
    "id": "prod_1234",
    "name": "Camiseta Premium",
    "price": {
      "amount": 8990,
      "currency": "BRL"
    }
  }
}
```

### Error Response

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "quantity",
        "message": "Must be greater than 0"
      }
    ],
    "request_id": "req_abc123"
  }
}
```

### Códigos de Erro

- `VALIDATION_ERROR`: Erro de validação de input
- `NOT_FOUND`: Recurso não encontrado
- `UNAUTHORIZED`: Não autenticado
- `FORBIDDEN`: Sem permissão
- `CONFLICT`: Conflito de estado
- `RATE_LIMIT_EXCEEDED`: Limite de requisições excedido
- `INTERNAL_ERROR`: Erro interno do servidor
- `SERVICE_UNAVAILABLE`: Serviço temporariamente indisponível

## Documentação

- **REST**: OpenAPI 3.0 (Swagger) em `/api/docs`
- **GraphQL**: GraphQL Playground em `/graphql`
- **Postman**: Collection compartilhada no workspace do time

## Webhooks

Para notificar sistemas externos sobre eventos:

```
POST https://partner.com/webhooks/shopflow

{
  "event": "order.completed",
  "data": {
    "order_id": "ord_1234",
    "customer_id": "cust_5678",
    "total": 15990
  },
  "timestamp": "2025-10-29T10:30:00Z",
  "signature": "sha256=..."
}
```

### Eventos Disponíveis

- `order.created`
- `order.completed`
- `order.cancelled`
- `payment.succeeded`
- `payment.failed`
- `shipment.dispatched`
- `shipment.delivered`
