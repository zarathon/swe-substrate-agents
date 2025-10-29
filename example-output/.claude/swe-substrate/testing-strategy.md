# Estratégia de Testes

**Gerado por**: `/swe-substrate-testing-strategy`
**Data**: 2025-10-29

## Pirâmide de Testes

```
        /\
       /E2E\         10% - Testes End-to-End
      /------\
     /Integr.\       30% - Testes de Integração
    /----------\
   /  Unitários \    60% - Testes Unitários
  /--------------\
```

## Testes Unitários

**Framework**: Jest + ts-jest
**Coverage mínimo**: 80% de cobertura de código

### Estrutura

```
src/
├── services/
│   ├── CartService.ts
│   └── CartService.test.ts
├── models/
│   ├── Product.ts
│   └── Product.test.ts
└── utils/
    ├── pricing.ts
    └── pricing.test.ts
```

### Convenções

```typescript
// CartService.test.ts
describe('CartService', () => {
  describe('addItem', () => {
    it('should add a new item to an empty cart', async () => {
      // Arrange
      const cart = new Cart();
      const product = createMockProduct({ id: 'prod_123' });

      // Act
      const result = await cartService.addItem(cart, product, 2);

      // Assert
      expect(result.items).toHaveLength(1);
      expect(result.items[0].quantity).toBe(2);
    });

    it('should increment quantity if item already exists', async () => {
      // ...
    });

    it('should throw if product is out of stock', async () => {
      // ...
    });
  });

  describe('calculateTotal', () => {
    it('should calculate total with multiple items', () => {
      // ...
    });

    it('should apply discount codes correctly', () => {
      // ...
    });
  });
});
```

### Mocking

**Biblioteca**: Jest mocks + `jest-mock-extended`

```typescript
// Mock de serviços externos
const mockInventoryService = {
  checkAvailability: jest.fn(),
  reserveStock: jest.fn(),
};

// Mock de database
jest.mock('../db/repositories/ProductRepository');
const mockProductRepo = ProductRepository as jest.Mocked<typeof ProductRepository>;

// Mock de APIs HTTP
jest.mock('axios');
const mockAxios = axios as jest.Mocked<typeof axios>;
```

### Testes de Snapshot

Para componentes React e estruturas de dados complexas:

```typescript
it('should render checkout form correctly', () => {
  const tree = renderer.create(
    <CheckoutForm cart={mockCart} onSubmit={jest.fn()} />
  ).toJSON();

  expect(tree).toMatchSnapshot();
});
```

## Testes de Integração

**Framework**: Jest + Supertest (para APIs HTTP) + Testcontainers (para DB)

### Estrutura

```
tests/
├── integration/
│   ├── api/
│   │   ├── products.test.ts
│   │   ├── orders.test.ts
│   │   └── checkout.test.ts
│   ├── database/
│   │   └── repositories.test.ts
│   └── messaging/
│       └── events.test.ts
```

### Setup com Testcontainers

```typescript
import { GenericContainer } from 'testcontainers';

describe('Order API Integration', () => {
  let postgresContainer: StartedTestContainer;
  let redisContainer: StartedTestContainer;
  let app: Express;

  beforeAll(async () => {
    // Start containers
    postgresContainer = await new GenericContainer('postgres:15')
      .withEnvironment({ POSTGRES_PASSWORD: 'test' })
      .withExposedPorts(5432)
      .start();

    redisContainer = await new GenericContainer('redis:7-alpine')
      .withExposedPorts(6379)
      .start();

    // Initialize app with test database
    app = await createApp({
      database: {
        host: postgresContainer.getHost(),
        port: postgresContainer.getMappedPort(5432),
      },
      redis: {
        host: redisContainer.getHost(),
        port: redisContainer.getMappedPort(6379),
      },
    });
  }, 60000);

  afterAll(async () => {
    await postgresContainer.stop();
    await redisContainer.stop();
  });

  describe('POST /api/v1/orders', () => {
    it('should create order and reserve stock', async () => {
      // Arrange: Create product and cart
      const product = await createTestProduct({ stock: 10 });
      const cart = await createTestCart([{ productId: product.id, quantity: 2 }]);

      // Act: Create order
      const response = await request(app)
        .post('/api/v1/orders')
        .send({ cartId: cart.id })
        .expect(201);

      // Assert: Verify order and stock
      expect(response.body.data.status).toBe('pending_payment');

      const updatedProduct = await Product.findById(product.id);
      expect(updatedProduct.stock).toBe(8);
    });
  });
});
```

### Testes de Banco de Dados

```typescript
describe('ProductRepository', () => {
  beforeEach(async () => {
    await cleanDatabase();
  });

  it('should find products by category with filters', async () => {
    // Arrange
    await seedProducts([
      { name: 'Product A', category: 'electronics', price: 100 },
      { name: 'Product B', category: 'electronics', price: 200 },
      { name: 'Product C', category: 'clothing', price: 50 },
    ]);

    // Act
    const results = await productRepo.findByCategory('electronics', {
      maxPrice: 150,
    });

    // Assert
    expect(results).toHaveLength(1);
    expect(results[0].name).toBe('Product A');
  });
});
```

### Testes de Messaging

```typescript
describe('Order Events', () => {
  it('should publish OrderPlaced event when order is created', async () => {
    // Arrange
    const eventHandler = jest.fn();
    eventBus.subscribe('order.placed', eventHandler);

    // Act
    await orderService.createOrder(mockOrderData);

    // Assert
    await waitForEvent(eventHandler, 1000);
    expect(eventHandler).toHaveBeenCalledWith(
      expect.objectContaining({
        type: 'order.placed',
        data: expect.objectContaining({
          orderId: expect.any(String),
        }),
      })
    );
  });
});
```

## Testes End-to-End (E2E)

**Framework**: Playwright
**Ambientes**: Staging (CI) e Local (Docker Compose)

### Estrutura

```
e2e/
├── tests/
│   ├── checkout/
│   │   ├── guest-checkout.spec.ts
│   │   └── authenticated-checkout.spec.ts
│   ├── product/
│   │   ├── search.spec.ts
│   │   └── customization.spec.ts
│   └── account/
│       └── registration.spec.ts
├── fixtures/
│   └── test-data.ts
└── utils/
    └── test-helpers.ts
```

### Exemplo de Teste

```typescript
import { test, expect } from '@playwright/test';

test.describe('Guest Checkout Flow', () => {
  test('should complete purchase as guest user', async ({ page }) => {
    // Navigate to product
    await page.goto('/products/camiseta-premium');

    // Customize product
    await page.click('[data-testid="customize-button"]');
    await page.fill('[data-testid="custom-text"]', 'João Silva');
    await page.selectOption('[data-testid="color-select"]', 'blue');

    // Add to cart
    await page.click('[data-testid="add-to-cart"]');
    await expect(page.locator('[data-testid="cart-count"]')).toHaveText('1');

    // Go to checkout
    await page.click('[data-testid="cart-button"]');
    await page.click('[data-testid="checkout-button"]');

    // Fill shipping info
    await page.fill('[name="email"]', 'test@example.com');
    await page.fill('[name="fullName"]', 'João Silva');
    await page.fill('[name="address"]', 'Rua Exemplo, 123');
    await page.fill('[name="city"]', 'São Paulo');
    await page.selectOption('[name="state"]', 'SP');
    await page.fill('[name="zipCode"]', '01234-567');

    // Fill payment info (test mode Stripe)
    const stripeFrame = page.frameLocator('[name="stripe_checkout"]');
    await stripeFrame.fill('[name="cardNumber"]', '4242424242424242');
    await stripeFrame.fill('[name="cardExpiry"]', '12/25');
    await stripeFrame.fill('[name="cardCvc"]', '123');

    // Complete purchase
    await page.click('[data-testid="complete-order"]');

    // Verify success
    await expect(page).toHaveURL(/\/order-confirmation\//);
    await expect(page.locator('h1')).toContainText('Pedido Confirmado');

    // Verify order details
    const orderNumber = await page.locator('[data-testid="order-number"]').textContent();
    expect(orderNumber).toMatch(/^ORD-\d+$/);
  });

  test('should show error for invalid credit card', async ({ page }) => {
    // ... similar flow

    // Use invalid card
    const stripeFrame = page.frameLocator('[name="stripe_checkout"]');
    await stripeFrame.fill('[name="cardNumber"]', '4000000000000002'); // Declined card

    await page.click('[data-testid="complete-order"]');

    // Verify error message
    await expect(page.locator('[role="alert"]')).toContainText('pagamento foi recusado');
  });
});
```

### Page Object Model

```typescript
// e2e/pages/CheckoutPage.ts
export class CheckoutPage {
  constructor(private page: Page) {}

  async fillShippingInfo(info: ShippingInfo) {
    await this.page.fill('[name="email"]', info.email);
    await this.page.fill('[name="fullName"]', info.fullName);
    // ...
  }

  async fillPaymentInfo(card: CardInfo) {
    const stripeFrame = this.page.frameLocator('[name="stripe_checkout"]');
    await stripeFrame.fill('[name="cardNumber"]', card.number);
    // ...
  }

  async completeOrder() {
    await this.page.click('[data-testid="complete-order"]');
  }

  async getOrderNumber() {
    return this.page.locator('[data-testid="order-number"]').textContent();
  }
}
```

## Testes de Performance

**Framework**: k6

### Testes de Carga

```javascript
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },  // Ramp up
    { duration: '5m', target: 100 },  // Stay at 100 users
    { duration: '2m', target: 200 },  // Ramp to 200 users
    { duration: '5m', target: 200 },  // Stay at 200 users
    { duration: '2m', target: 0 },    // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'], // 95% of requests under 500ms
    http_req_failed: ['rate<0.01'],   // Error rate under 1%
  },
};

export default function () {
  // Browse products
  let res = http.get('https://staging.shopflow.com/api/v1/products');
  check(res, { 'product list loaded': (r) => r.status === 200 });

  sleep(1);

  // Add to cart
  const productId = res.json('data.0.id');
  res = http.post('https://staging.shopflow.com/api/v1/cart/items', {
    productId,
    quantity: 1,
  });
  check(res, { 'item added to cart': (r) => r.status === 201 });

  sleep(2);
}
```

## Testes de Regressão Visual

**Framework**: Percy.io integrado com Playwright

```typescript
import { test } from '@playwright/test';
import percySnapshot from '@percy/playwright';

test('visual regression for product page', async ({ page }) => {
  await page.goto('/products/camiseta-premium');
  await percySnapshot(page, 'Product Page - Desktop');

  await page.setViewportSize({ width: 375, height: 667 });
  await percySnapshot(page, 'Product Page - Mobile');
});
```

## CI/CD Pipeline

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm test -- --coverage
      - run: npm run test:coverage-report

  integration-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run test:integration

  e2e-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npx playwright install
      - run: npm run test:e2e

  performance-tests:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - run: docker run -v $(pwd):/scripts grafana/k6 run /scripts/load-test.js
```

## Comandos de Teste

```bash
# Testes unitários
npm test
npm test -- --watch
npm test -- --coverage

# Testes de integração
npm run test:integration

# Testes E2E
npm run test:e2e
npm run test:e2e:ui  # Com interface visual

# Testes de performance
npm run test:load

# Todos os testes
npm run test:all
```

## Métricas de Qualidade

- **Code Coverage**: Mínimo 80% (meta: 90%)
- **Mutation Score**: Mínimo 70% (usando Stryker)
- **E2E Test Success Rate**: >98%
- **Test Execution Time**: Unit <30s, Integration <2min, E2E <10min
