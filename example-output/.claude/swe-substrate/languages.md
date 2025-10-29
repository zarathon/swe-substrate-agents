# Linguagens e Idiomática

**Gerado por**: `/swe-substrate-languages`
**Data**: 2025-10-29

## TypeScript

**Versão**: 5.2
**Configuração**: Strict mode habilitado

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@models/*": ["src/models/*"],
      "@services/*": ["src/services/*"],
      "@utils/*": ["src/utils/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Convenções de Código

#### Naming Conventions

```typescript
// Classes: PascalCase
class OrderService {}
class PaymentProcessor {}

// Interfaces & Types: PascalCase
interface Product {}
type OrderStatus = 'pending' | 'confirmed' | 'shipped';

// Functions & Variables: camelCase
function calculateTotal() {}
const shippingCost = 10;

// Constants: SCREAMING_SNAKE_CASE
const MAX_CART_ITEMS = 50;
const DEFAULT_CURRENCY = 'BRL';

// Private properties: prefix with _
class Cart {
  private _items: CartItem[];
}

// File names: kebab-case
// order-service.ts
// payment-processor.ts
// cart-item.model.ts
```

#### Type Safety

```typescript
// Use explicit types for function signatures
function calculateDiscount(price: number, discountPercent: number): number {
  return price * (discountPercent / 100);
}

// Use discriminated unions for variants
type PaymentMethod =
  | { type: 'credit_card'; cardLast4: string; brand: string }
  | { type: 'paypal'; email: string }
  | { type: 'bank_slip'; barcode: string };

function processPayment(method: PaymentMethod) {
  switch (method.type) {
    case 'credit_card':
      return processCreditCard(method.cardLast4, method.brand);
    case 'paypal':
      return processPayPal(method.email);
    case 'bank_slip':
      return processBankSlip(method.barcode);
  }
}

// Use unknown instead of any
function parseJSON(jsonString: string): unknown {
  return JSON.parse(jsonString);
}

// Type guards
function isProduct(obj: unknown): obj is Product {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj &&
    'price' in obj
  );
}
```

#### Async/Await

```typescript
// Prefer async/await over promises
async function fetchProductDetails(productId: string): Promise<Product> {
  const product = await productRepo.findById(productId);
  if (!product) {
    throw new NotFoundError(`Product ${productId} not found`);
  }
  return product;
}

// Handle errors with try/catch
async function createOrder(orderData: CreateOrderDTO): Promise<Order> {
  try {
    const order = await orderService.create(orderData);
    await inventoryService.reserveStock(order.items);
    await emailService.sendConfirmation(order);
    return order;
  } catch (error) {
    logger.error('Failed to create order', { error, orderData });
    throw error;
  }
}

// Parallel async operations
const [product, inventory, reviews] = await Promise.all([
  productRepo.findById(productId),
  inventoryService.getStock(productId),
  reviewService.getReviews(productId),
]);
```

#### Error Handling

```typescript
// Custom error classes
class ApplicationError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends ApplicationError {
  constructor(message: string) {
    super(message, 'NOT_FOUND', 404);
  }
}

class ValidationError extends ApplicationError {
  constructor(message: string, public details: ValidationDetail[]) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

// Error handling middleware
app.use((error: Error, req: Request, res: Response, next: NextFunction) => {
  if (error instanceof ApplicationError) {
    return res.status(error.statusCode).json({
      error: {
        code: error.code,
        message: error.message,
        ...(error instanceof ValidationError && { details: error.details }),
      },
    });
  }

  logger.error('Unhandled error', { error, stack: error.stack });
  res.status(500).json({
    error: {
      code: 'INTERNAL_ERROR',
      message: 'An unexpected error occurred',
    },
  });
});
```

### Utility Types

```typescript
// Make all properties optional except specified
type PartialExcept<T, K extends keyof T> = Partial<T> & Pick<T, K>;

// Usage
type UpdateProductDTO = PartialExcept<Product, 'id'>;

// Extract promise result type
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

// Example
type OrderResult = UnwrapPromise<ReturnType<typeof createOrder>>;
```

## Padrões de Concorrência

### Promise.all para Paralelismo

```typescript
// Executa múltiplas operações em paralelo
async function enrichProductData(productId: string) {
  const [product, reviews, inventory, relatedProducts] = await Promise.all([
    productRepo.findById(productId),
    reviewService.getReviews(productId),
    inventoryService.getStock(productId),
    recommendationService.getRelated(productId),
  ]);

  return {
    ...product,
    reviews,
    inventory,
    relatedProducts,
  };
}
```

### Promise.allSettled para Operações Independentes

```typescript
// Executa todas as operações mesmo se algumas falharem
async function notifyOrderUpdate(order: Order) {
  const results = await Promise.allSettled([
    emailService.sendOrderUpdate(order),
    smsService.sendNotification(order.customer.phone),
    pushNotificationService.send(order.customer.id),
  ]);

  results.forEach((result, index) => {
    if (result.status === 'rejected') {
      logger.warn(`Notification ${index} failed`, { error: result.reason });
    }
  });
}
```

### Worker Threads para Operações CPU-Intensive

```typescript
import { Worker } from 'worker_threads';

// Processar imagens em worker thread
function processImage(imageBuffer: Buffer): Promise<Buffer> {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./image-processor-worker.js', {
      workerData: { imageBuffer },
    });

    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) {
        reject(new Error(`Worker stopped with exit code ${code}`));
      }
    });
  });
}
```

### Bull Queue para Jobs Assíncronos

```typescript
import Queue from 'bull';

// Criar fila para processamento de pedidos
const orderQueue = new Queue('orders', process.env.REDIS_URL);

// Adicionar job
async function createOrder(orderData: CreateOrderDTO) {
  const order = await orderRepo.create(orderData);

  // Processar assincronamente
  await orderQueue.add('process-order', {
    orderId: order.id,
  });

  return order;
}

// Processar job
orderQueue.process('process-order', async (job) => {
  const { orderId } = job.data;

  await inventoryService.reserveStock(orderId);
  await paymentService.processPayment(orderId);
  await emailService.sendConfirmation(orderId);
});
```

## Gerenciamento de Memória

### Evitar Memory Leaks

```typescript
// Ruim - Event listener não removido
class OrderProcessor {
  constructor() {
    eventBus.on('order.created', this.handleOrder);
  }

  handleOrder(order: Order) {
    // Process order
  }
}

// Bom - Cleanup apropriado
class OrderProcessor {
  private listener: (order: Order) => void;

  constructor() {
    this.listener = this.handleOrder.bind(this);
    eventBus.on('order.created', this.listener);
  }

  destroy() {
    eventBus.off('order.created', this.listener);
  }

  handleOrder(order: Order) {
    // Process order
  }
}
```

### Streams para Grandes Volumes de Dados

```typescript
import { pipeline } from 'stream/promises';
import { createReadStream, createWriteStream } from 'fs';
import { Transform } from 'stream';

// Processar CSV grande linha por linha
async function processLargeCSV(inputPath: string, outputPath: string) {
  const transformStream = new Transform({
    objectMode: true,
    transform(chunk, encoding, callback) {
      // Processar cada linha
      const processed = processRow(chunk);
      callback(null, processed);
    },
  });

  await pipeline(
    createReadStream(inputPath),
    csv.parse(),
    transformStream,
    csv.stringify(),
    createWriteStream(outputPath)
  );
}
```

## Linting & Formatting

### ESLint Configuration

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:@typescript-eslint/recommended-requiring-type-checking",
    "prettier"
  ],
  "parser": "@typescript-eslint/parser",
  "parserOptions": {
    "project": "./tsconfig.json"
  },
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
    "@typescript-eslint/no-floating-promises": "error",
    "no-console": "warn",
    "eqeqeq": ["error", "always"]
  }
}
```

### Prettier Configuration

```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "always"
}
```

### Pre-commit Hooks (Husky)

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.ts": [
      "eslint --fix",
      "prettier --write",
      "npm test -- --findRelatedTests"
    ]
  }
}
```

## Documentação de Código

### JSDoc Comments

```typescript
/**
 * Calcula o total do carrinho incluindo descontos e frete
 *
 * @param cart - Carrinho de compras
 * @param discountCode - Código de desconto opcional
 * @returns Total calculado com breakdown detalhado
 * @throws {ValidationError} Se o carrinho estiver vazio
 * @example
 * ```typescript
 * const total = await calculateCartTotal(cart, 'SUMMER2024');
 * console.log(total.finalAmount); // 15990 (em centavos)
 * ```
 */
async function calculateCartTotal(
  cart: Cart,
  discountCode?: string
): Promise<CartTotal> {
  // Implementation
}
```

## Migrations de Breaking Changes

### Deprecation Warnings

```typescript
/**
 * @deprecated Use `getProductById` instead. Will be removed in v3.0.0
 */
export function getProduct(id: string): Promise<Product> {
  console.warn(
    'getProduct is deprecated. Use getProductById instead. Will be removed in v3.0.0'
  );
  return getProductById(id);
}

export function getProductById(id: string): Promise<Product> {
  // New implementation
}
```

## Performance Best Practices

### Lazy Loading

```typescript
// Carregamento sob demanda de módulos pesados
async function generateInvoice(orderId: string) {
  // PDF library só é carregada quando necessário
  const { default: PDFDocument } = await import('pdfkit');

  const doc = new PDFDocument();
  // Generate PDF
}
```

### Caching de Resultados Caros

```typescript
import memoize from 'memoizee';

// Cache result por 1 hora
const getProductRecommendations = memoize(
  async (productId: string) => {
    return await mlService.getRecommendations(productId);
  },
  { maxAge: 3600000, promise: true }
);
```

### Database Query Optimization

```typescript
// Ruim - N+1 query problem
async function getOrdersWithCustomers(orderIds: string[]) {
  const orders = await Order.findAll({ where: { id: orderIds } });

  for (const order of orders) {
    order.customer = await Customer.findByPk(order.customerId); // N queries
  }

  return orders;
}

// Bom - Single query with join
async function getOrdersWithCustomers(orderIds: string[]) {
  return await Order.findAll({
    where: { id: orderIds },
    include: [{ model: Customer }], // Single JOIN query
  });
}
```
