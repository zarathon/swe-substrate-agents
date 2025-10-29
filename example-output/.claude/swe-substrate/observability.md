# Observabilidade

**Gerado por**: `/swe-substrate-observability`
**Data**: 2025-10-29

## Logging

**Library**: Winston
**Format**: JSON structured logs
**Levels**: error, warn, info, http, debug

### Logger Configuration

```typescript
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'shopflow-api',
    environment: process.env.NODE_ENV,
  },
  transports: [
    // Console for local development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      ),
    }),
    // CloudWatch for production
    new WinstonCloudWatch({
      logGroupName: '/shopflow/api',
      logStreamName: `${process.env.NODE_ENV}-${new Date().toISOString().split('T')[0]}`,
      awsRegion: 'us-east-1',
    }),
  ],
});

export default logger;
```

### Structured Logging

```typescript
// Include context in logs
logger.info('Order created', {
  orderId: order.id,
  customerId: order.customerId,
  totalAmount: order.totalAmount,
  itemCount: order.items.length,
  paymentMethod: order.paymentMethod,
  timestamp: new Date().toISOString(),
});

// Error logging with stack traces
try {
  await processPayment(order);
} catch (error) {
  logger.error('Payment processing failed', {
    orderId: order.id,
    error: error.message,
    stack: error.stack,
    paymentMethod: order.paymentMethod,
  });
  throw error;
}

// HTTP request logging middleware
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;

    logger.http('HTTP request', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration,
      userAgent: req.headers['user-agent'],
      ip: req.ip,
      userId: req.user?.id,
    });
  });

  next();
});
```

### Log Levels Guidelines

- **error**: Application errors, exceptions, failures
- **warn**: Unexpected behavior, deprecated usage, quota warnings
- **info**: Important business events (order created, payment processed)
- **http**: HTTP requests and responses
- **debug**: Detailed debug information (only in development)

## Metrics

**Platform**: Datadog
**Library**: hot-shots (StatsD client)

### Metrics Configuration

```typescript
import { StatsD } from 'hot-shots';

const metrics = new StatsD({
  host: process.env.DATADOG_AGENT_HOST || 'localhost',
  port: 8125,
  prefix: 'shopflow.',
  globalTags: {
    env: process.env.NODE_ENV,
    service: 'api',
    version: process.env.APP_VERSION,
  },
});

export default metrics;
```

### Business Metrics

```typescript
// Order metrics
async function createOrder(orderData: CreateOrderDTO) {
  const startTime = Date.now();

  try {
    const order = await orderService.create(orderData);

    // Increment order counter
    metrics.increment('orders.created', 1, {
      payment_method: order.paymentMethod,
      customer_type: order.customer.isPremium ? 'premium' : 'regular',
    });

    // Track order value
    metrics.histogram('orders.value', order.totalAmount / 100, {
      currency: order.currency,
    });

    // Track order processing time
    metrics.timing('orders.create.duration', Date.now() - startTime);

    return order;
  } catch (error) {
    metrics.increment('orders.failed', 1, {
      error_type: error.constructor.name,
    });
    throw error;
  }
}

// Cart metrics
metrics.gauge('carts.active', await getActiveCartCount());
metrics.histogram('carts.items.count', cart.items.length);
metrics.histogram('carts.abandonment.time', abandonmentTimeInMinutes);
```

### Technical Metrics

```typescript
// Database query performance
async function queryWithMetrics<T>(query: string, params: any[]): Promise<T> {
  const start = Date.now();

  try {
    const result = await db.query(query, params);

    metrics.timing('database.query.duration', Date.now() - start, {
      query_type: query.split(' ')[0], // SELECT, INSERT, etc.
    });

    return result;
  } catch (error) {
    metrics.increment('database.query.error', 1);
    throw error;
  }
}

// Cache hit/miss rates
async function getCachedProduct(productId: string) {
  const cached = await redis.get(`product:${productId}`);

  if (cached) {
    metrics.increment('cache.hit', 1, { cache_key: 'product' });
    return JSON.parse(cached);
  }

  metrics.increment('cache.miss', 1, { cache_key: 'product' });

  const product = await db.getProduct(productId);
  await redis.setex(`product:${productId}`, 3600, JSON.stringify(product));

  return product;
}

// API endpoint performance
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    metrics.timing('http.request.duration', Date.now() - start, {
      method: req.method,
      endpoint: req.route?.path || req.path,
      status: res.statusCode,
    });

    metrics.increment('http.requests', 1, {
      method: req.method,
      status: res.statusCode,
    });
  });

  next();
});
```

### System Metrics

```typescript
import os from 'os';

// Track system resources every 60 seconds
setInterval(() => {
  metrics.gauge('system.memory.usage', process.memoryUsage().heapUsed);
  metrics.gauge('system.memory.total', process.memoryUsage().heapTotal);
  metrics.gauge('system.cpu.usage', process.cpuUsage().system);
  metrics.gauge('system.uptime', process.uptime());

  // Event loop lag
  const start = Date.now();
  setImmediate(() => {
    metrics.gauge('system.event_loop.lag', Date.now() - start);
  });
}, 60000);
```

## Distributed Tracing

**Platform**: Datadog APM
**Library**: dd-trace

### Tracer Configuration

```typescript
import tracer from 'dd-trace';

tracer.init({
  service: 'shopflow-api',
  env: process.env.NODE_ENV,
  version: process.env.APP_VERSION,
  logInjection: true, // Inject trace IDs into logs
  analytics: true,
  runtimeMetrics: true,
  plugins: true, // Auto-instrument libraries
});

export default tracer;
```

### Custom Spans

```typescript
import tracer from './tracer';

async function processOrder(orderId: string) {
  const span = tracer.startSpan('order.process', {
    resource: orderId,
    tags: {
      'order.id': orderId,
    },
  });

  try {
    // Reserve inventory
    const inventorySpan = tracer.startSpan('inventory.reserve', {
      childOf: span,
    });
    await inventoryService.reserve(orderId);
    inventorySpan.finish();

    // Process payment
    const paymentSpan = tracer.startSpan('payment.process', {
      childOf: span,
    });
    await paymentService.process(orderId);
    paymentSpan.finish();

    // Send confirmation
    const emailSpan = tracer.startSpan('email.send', {
      childOf: span,
    });
    await emailService.sendConfirmation(orderId);
    emailSpan.finish();

    span.setTag('order.status', 'completed');
  } catch (error) {
    span.setTag('error', true);
    span.setTag('error.message', error.message);
    span.setTag('error.stack', error.stack);
    throw error;
  } finally {
    span.finish();
  }
}
```

### Trace Context Propagation

```typescript
// Propagate trace context to external services
import { propagation, trace } from '@opentelemetry/api';

async function callInventoryService(productId: string) {
  const span = trace.getActiveSpan();
  const headers = {};

  // Inject trace context into headers
  propagation.inject(trace.setSpan(context.active(), span), headers);

  const response = await fetch(`https://inventory-service/api/check/${productId}`, {
    headers,
  });

  return response.json();
}
```

## Error Tracking

**Platform**: Sentry

### Sentry Configuration

```typescript
import * as Sentry from '@sentry/node';
import * as Tracing from '@sentry/tracing';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  release: process.env.APP_VERSION,
  tracesSampleRate: 0.1, // 10% of traces

  integrations: [
    new Sentry.Integrations.Http({ tracing: true }),
    new Tracing.Integrations.Express({ app }),
    new Tracing.Integrations.Postgres(),
  ],

  beforeSend(event, hint) {
    // Filter out sensitive data
    if (event.request?.data) {
      delete event.request.data.password;
      delete event.request.data.creditCard;
    }
    return event;
  },
});

// Request handler middleware
app.use(Sentry.Handlers.requestHandler());
app.use(Sentry.Handlers.tracingHandler());

// Error handler middleware (must be last)
app.use(Sentry.Handlers.errorHandler());
```

### Custom Error Reporting

```typescript
try {
  await processPayment(order);
} catch (error) {
  Sentry.captureException(error, {
    tags: {
      component: 'payment',
      payment_method: order.paymentMethod,
    },
    extra: {
      orderId: order.id,
      amount: order.totalAmount,
      customerId: order.customerId,
    },
    level: 'error',
  });

  throw error;
}

// Capture custom messages
Sentry.captureMessage('Unusual cart abandonment spike detected', {
  level: 'warning',
  tags: {
    component: 'analytics',
  },
  extra: {
    abandonmentRate: 0.85,
    threshold: 0.70,
  },
});
```

## Health Checks

### Endpoints

```typescript
// Basic health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// Detailed health check
app.get('/health/detailed', async (req, res) => {
  const health = {
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks: {
      database: await checkDatabase(),
      redis: await checkRedis(),
      elasticsearch: await checkElasticsearch(),
    },
  };

  const isHealthy = Object.values(health.checks).every((check) => check.status === 'ok');

  res.status(isHealthy ? 200 : 503).json(health);
});

// Individual service checks
async function checkDatabase() {
  try {
    await db.query('SELECT 1');
    return { status: 'ok', latency: Date.now() - start };
  } catch (error) {
    return { status: 'error', message: error.message };
  }
}
```

### Readiness & Liveness Probes (Kubernetes)

```typescript
// Liveness - is the app running?
app.get('/health/liveness', (req, res) => {
  res.json({ status: 'ok' });
});

// Readiness - is the app ready to serve traffic?
app.get('/health/readiness', async (req, res) => {
  const checks = await Promise.all([
    checkDatabase(),
    checkRedis(),
    checkRequiredServices(),
  ]);

  const isReady = checks.every((check) => check.status === 'ok');

  res.status(isReady ? 200 : 503).json({
    status: isReady ? 'ready' : 'not_ready',
    checks,
  });
});
```

## Dashboards

### Datadog Dashboards

1. **Overview Dashboard**
   - Request rate, error rate, latency (P50, P95, P99)
   - Orders per minute, revenue per hour
   - Active users, active carts

2. **Order Processing Dashboard**
   - Order creation rate
   - Payment success/failure rate
   - Order processing duration
   - Inventory reservation latency

3. **Infrastructure Dashboard**
   - CPU, memory, disk usage
   - Database connections, query performance
   - Cache hit/miss rates
   - API gateway metrics

4. **Business KPIs Dashboard**
   - Conversion rate
   - Cart abandonment rate
   - Average order value
   - Customer lifetime value

### Alert Configuration

```yaml
# datadog-alerts.yaml
monitors:
  - name: High Error Rate
    type: metric alert
    query: "avg(last_5m):sum:shopflow.http.requests{status:5xx}.as_count() > 50"
    message: |
      High error rate detected on {{service.name}}
      @slack-engineering @pagerduty-on-call

  - name: Payment Processing Failure
    type: metric alert
    query: "avg(last_10m):sum:shopflow.payments.failed.as_count() > 10"
    message: |
      Payment failures exceeding threshold
      @slack-payments @pagerduty-on-call

  - name: Database Connection Pool Exhausted
    type: metric alert
    query: "avg(last_5m):max:shopflow.database.pool.waiting > 5"
    message: |
      Database connection pool is exhausted
      @slack-engineering

  - name: High API Latency (P95)
    type: metric alert
    query: "avg(last_15m):p95:shopflow.http.request.duration{endpoint:/api/v1/checkout} > 2000"
    message: |
      Checkout API latency is high (>2s P95)
      @slack-engineering
```

## On-Call Runbooks

### Payment Failure Spike

1. Check Datadog dashboard for payment errors
2. Check Sentry for error details
3. Verify Stripe/PayPal status pages
4. Review recent deployments
5. Check database for payment transaction logs
6. Escalate to payments team if provider issue

### Database Connection Issues

1. Check RDS dashboard for CPU/memory
2. Check active connections: `SELECT count(*) FROM pg_stat_activity`
3. Kill long-running queries if needed
4. Increase connection pool size if required
5. Check for N+1 query patterns in recent code changes

### Cache Eviction Issues

1. Check Redis memory usage
2. Review cache hit/miss rates
3. Check for cache stampede patterns
4. Verify cache TTLs are appropriate
5. Scale Redis cluster if needed
