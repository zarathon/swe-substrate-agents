# Agentes e Background Jobs

**Gerado por**: `/swe-substrate-agents`
**Data**: 2025-10-29

## Background Job Processing

**Technology**: Bull (Redis-based job queue)
**Workers**: Separate Node.js processes

### Queue Configuration

```typescript
import Queue from 'bull';

// Create queues for different job types
export const emailQueue = new Queue('emails', process.env.REDIS_URL, {
  defaultJobOptions: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 2000,
    },
    removeOnComplete: 100, // Keep last 100 completed jobs
    removeOnFail: 500,     // Keep last 500 failed jobs
  },
});

export const inventoryQueue = new Queue('inventory', process.env.REDIS_URL);
export const recommendationsQueue = new Queue('recommendations', process.env.REDIS_URL);
export const analyticsQueue = new Queue('analytics', process.env.REDIS_URL);
```

### Job Producers

```typescript
// Add job to queue
async function sendOrderConfirmation(order: Order) {
  await emailQueue.add(
    'order-confirmation',
    {
      orderId: order.id,
      customerEmail: order.customer.email,
      orderTotal: order.totalAmount,
    },
    {
      priority: 1, // High priority
      attempts: 5,
    }
  );
}

// Delayed job
async function sendAbandonedCartReminder(cartId: string) {
  await emailQueue.add(
    'cart-reminder',
    { cartId },
    {
      delay: 24 * 60 * 60 * 1000, // 24 hours
    }
  );
}

// Repeatable job (cron)
await analyticsQueue.add(
  'daily-report',
  {},
  {
    repeat: {
      cron: '0 9 * * *', // Every day at 9 AM
      tz: 'America/Sao_Paulo',
    },
  }
);
```

### Job Consumers

```typescript
// Email worker
emailQueue.process('order-confirmation', async (job) => {
  const { orderId, customerEmail, orderTotal } = job.data;

  logger.info('Processing order confirmation email', { orderId });

  try {
    await sendgrid.send({
      to: customerEmail,
      from: 'noreply@shopflow.com',
      templateId: 'd-xxxxx',
      dynamicTemplateData: {
        orderId,
        orderTotal: (orderTotal / 100).toFixed(2),
      },
    });

    logger.info('Order confirmation email sent', { orderId });
  } catch (error) {
    logger.error('Failed to send order confirmation', { orderId, error });
    throw error; // Will trigger retry
  }
});

// Inventory worker with concurrency
inventoryQueue.process('reserve-stock', 5, async (job) => {
  const { orderId, items } = job.data;

  for (const item of items) {
    await db.query(
      'UPDATE inventory SET reserved_quantity = reserved_quantity + $1 WHERE product_id = $2',
      [item.quantity, item.productId]
    );
  }

  // Update job progress
  job.progress(100);
});
```

### Job Events & Monitoring

```typescript
// Job lifecycle events
emailQueue.on('completed', (job, result) => {
  logger.info('Job completed', {
    jobId: job.id,
    jobName: job.name,
    result,
  });

  metrics.increment('jobs.completed', 1, { queue: 'emails', job: job.name });
});

emailQueue.on('failed', (job, error) => {
  logger.error('Job failed', {
    jobId: job.id,
    jobName: job.name,
    error: error.message,
    attempts: job.attemptsMade,
  });

  metrics.increment('jobs.failed', 1, { queue: 'emails', job: job.name });

  // Alert on critical job failures
  if (job.name === 'order-confirmation' && job.attemptsMade >= 5) {
    notifyOnCall({
      severity: 'high',
      message: `Order confirmation email failed after 5 attempts`,
      orderId: job.data.orderId,
    });
  }
});

emailQueue.on('stalled', (job) => {
  logger.warn('Job stalled', {
    jobId: job.id,
    jobName: job.name,
  });
});
```

## Scheduled Tasks (Cron Jobs)

**Library**: node-cron

### Cron Configuration

```typescript
import cron from 'node-cron';

// Clean expired carts every hour
cron.schedule('0 * * * *', async () => {
  logger.info('Starting expired carts cleanup');

  const expiredCarts = await db.query(`
    DELETE FROM carts
    WHERE updated_at < NOW() - INTERVAL '24 hours'
    AND customer_id IS NULL
    RETURNING id
  `);

  logger.info('Expired carts cleaned', { count: expiredCarts.rowCount });
  metrics.gauge('cron.expired_carts_cleaned', expiredCarts.rowCount);
});

// Release reserved inventory every 15 minutes
cron.schedule('*/15 * * * *', async () => {
  logger.info('Releasing expired inventory reservations');

  await db.query(`
    UPDATE inventory
    SET reserved_quantity = reserved_quantity - sub.expired_qty
    FROM (
      SELECT product_id, SUM(quantity) as expired_qty
      FROM inventory_reservations
      WHERE expires_at < NOW()
      GROUP BY product_id
    ) sub
    WHERE inventory.product_id = sub.product_id
  `);

  await db.query(`
    DELETE FROM inventory_reservations
    WHERE expires_at < NOW()
  `);
});

// Generate daily sales report at 8 AM
cron.schedule('0 8 * * *', async () => {
  logger.info('Generating daily sales report');

  const report = await generateSalesReport({
    startDate: new Date(Date.now() - 24 * 60 * 60 * 1000),
    endDate: new Date(),
  });

  await emailService.sendToTeam('daily-sales-report', report);
});

// Health check for cron jobs
cron.schedule('*/5 * * * *', async () => {
  await redis.set('cron:heartbeat', Date.now(), 'EX', 600);
});
```

## Daemon Processes

### Stock Synchronization Daemon

```typescript
// Continuously sync stock from external warehouse system
class StockSyncDaemon {
  private isRunning = false;
  private syncInterval = 30000; // 30 seconds

  async start() {
    this.isRunning = true;
    logger.info('Stock sync daemon started');

    while (this.isRunning) {
      try {
        await this.syncStock();
      } catch (error) {
        logger.error('Stock sync failed', { error });
        metrics.increment('daemon.stock_sync.error');
      }

      await this.sleep(this.syncInterval);
    }
  }

  async stop() {
    this.isRunning = false;
    logger.info('Stock sync daemon stopped');
  }

  private async syncStock() {
    const products = await this.getProductsToSync();

    for (const product of products) {
      const externalStock = await warehouseAPI.getStock(product.externalId);

      if (externalStock.quantity !== product.localQuantity) {
        await this.updateLocalStock(product.id, externalStock.quantity);

        // Emit event for real-time updates
        await redis.publish('stock-updated', JSON.stringify({
          productId: product.id,
          quantity: externalStock.quantity,
        }));
      }
    }

    metrics.gauge('daemon.stock_sync.products_checked', products.length);
  }

  private sleep(ms: number): Promise<void> {
    return new Promise((resolve) => setTimeout(resolve, ms));
  }
}

// Start daemon
const stockDaemon = new StockSyncDaemon();
stockDaemon.start();

// Graceful shutdown
process.on('SIGTERM', async () => {
  await stockDaemon.stop();
  process.exit(0);
});
```

### Real-time Notification Daemon

```typescript
// WebSocket server for real-time updates
class NotificationDaemon {
  private wss: WebSocketServer;
  private subscribers: Map<string, Set<WebSocket>> = new Map();

  constructor() {
    this.wss = new WebSocketServer({ port: 8080 });
    this.setupWebSocket();
    this.subscribeToEvents();
  }

  private setupWebSocket() {
    this.wss.on('connection', (ws, req) => {
      const userId = this.authenticateConnection(req);

      ws.on('message', (data) => {
        const message = JSON.parse(data.toString());

        if (message.type === 'subscribe') {
          this.subscribe(userId, message.channel, ws);
        }
      });

      ws.on('close', () => {
        this.unsubscribeAll(ws);
      });
    });
  }

  private subscribeToEvents() {
    // Subscribe to Redis pub/sub channels
    const subscriber = redis.duplicate();

    subscriber.subscribe('order-status-changed');
    subscriber.subscribe('stock-updated');

    subscriber.on('message', (channel, message) => {
      const data = JSON.parse(message);

      // Broadcast to relevant subscribers
      this.broadcast(channel, data);
    });
  }

  private broadcast(channel: string, data: any) {
    const subscribers = this.subscribers.get(channel);

    if (subscribers) {
      subscribers.forEach((ws) => {
        if (ws.readyState === WebSocket.OPEN) {
          ws.send(JSON.stringify({ channel, data }));
        }
      });
    }
  }
}

const notificationDaemon = new NotificationDaemon();
```

## LLM Agents

### Product Recommendation Agent

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

class RecommendationAgent {
  async generateRecommendations(userId: string): Promise<Product[]> {
    // Get user context
    const userHistory = await this.getUserPurchaseHistory(userId);
    const userPreferences = await this.getUserPreferences(userId);
    const trendingProducts = await this.getTrendingProducts();

    // Build context for Claude
    const context = `
User Purchase History:
${userHistory.map(p => `- ${p.name} (${p.category})`).join('\n')}

User Preferences:
- Favorite Categories: ${userPreferences.categories.join(', ')}
- Price Range: R$ ${userPreferences.minPrice} - R$ ${userPreferences.maxPrice}

Currently Trending Products:
${trendingProducts.map(p => `- ${p.name} (${p.category})`).join('\n')}
`;

    // Call Claude for recommendations
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 1024,
      messages: [
        {
          role: 'user',
          content: `${context}\n\nBased on this user's history and preferences, recommend 5 products they might like. Return only product names, one per line.`,
        },
      ],
    });

    const recommendations = message.content[0].text
      .split('\n')
      .filter((line) => line.trim());

    // Find actual products matching recommendations
    return this.findProductsByNames(recommendations);
  }
}
```

### Customer Support Assistant Agent

```typescript
class CustomerSupportAgent {
  async handleInquiry(customerId: string, question: string): Promise<string> {
    // Get customer context
    const customer = await this.getCustomer(customerId);
    const recentOrders = await this.getRecentOrders(customerId);

    const context = `
Customer: ${customer.name} (${customer.email})
Account since: ${customer.createdAt}
Premium member: ${customer.isPremium ? 'Yes' : 'No'}

Recent Orders:
${recentOrders.map(o => `- Order #${o.id}: ${o.status} (${o.totalAmount})`).join('\n')}
`;

    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 2048,
      messages: [
        {
          role: 'user',
          content: `${context}\n\nCustomer Question: ${question}\n\nProvide a helpful response addressing their question. Be concise and friendly.`,
        },
      ],
    });

    const response = message.content[0].text;

    // Log interaction
    await this.logSupportInteraction(customerId, question, response);

    return response;
  }
}
```

### Content Moderation Agent

```typescript
class ModerationAgent {
  async moderateReview(reviewText: string): Promise<ModerationResult> {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 512,
      messages: [
        {
          role: 'user',
          content: `Analyze this product review for inappropriate content:

Review: "${reviewText}"

Check for:
- Profanity or offensive language
- Personal attacks
- Spam or promotional content
- Irrelevant content

Respond in JSON format:
{
  "approved": boolean,
  "reason": string,
  "confidence": number (0-1)
}`,
        },
      ],
    });

    const result = JSON.parse(message.content[0].text);

    if (!result.approved) {
      logger.warn('Review flagged by moderation', {
        reviewText,
        reason: result.reason,
        confidence: result.confidence,
      });
    }

    return result;
  }
}
```

## Event-Driven Agents

### Order Fulfillment Agent

```typescript
// Listen to order events and orchestrate fulfillment
class FulfillmentAgent {
  constructor() {
    this.subscribeToEvents();
  }

  private subscribeToEvents() {
    eventBus.on('order.payment_confirmed', async (order) => {
      await this.handlePaymentConfirmed(order);
    });

    eventBus.on('inventory.stock_allocated', async ({ orderId }) => {
      await this.handleStockAllocated(orderId);
    });

    eventBus.on('shipment.label_created', async ({ orderId }) => {
      await this.handleLabelCreated(orderId);
    });
  }

  private async handlePaymentConfirmed(order: Order) {
    logger.info('Payment confirmed, starting fulfillment', { orderId: order.id });

    // Request stock allocation
    await inventoryService.allocateStock(order.id);

    // Update order status
    await orderService.updateStatus(order.id, 'processing');

    // Emit event
    eventBus.emit('order.fulfillment_started', order);
  }

  private async handleStockAllocated(orderId: string) {
    logger.info('Stock allocated, creating shipping label', { orderId });

    // Create shipping label
    const label = await shippingService.createLabel(orderId);

    // Emit event
    eventBus.emit('shipment.label_created', { orderId, labelId: label.id });
  }

  private async handleLabelCreated(orderId: string) {
    logger.info('Label created, notifying warehouse', { orderId });

    // Notify warehouse for packing
    await warehouseAPI.notifyForPacking(orderId);

    // Update order status
    await orderService.updateStatus(orderId, 'ready_to_ship');
  }
}

const fulfillmentAgent = new FulfillmentAgent();
```

## Process Management

### PM2 Configuration

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'api',
      script: 'dist/index.js',
      instances: 4, // Cluster mode
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
      },
    },
    {
      name: 'email-worker',
      script: 'dist/workers/email.js',
      instances: 2,
      env: {
        NODE_ENV: 'production',
        WORKER_TYPE: 'email',
      },
    },
    {
      name: 'inventory-worker',
      script: 'dist/workers/inventory.js',
      instances: 1,
      env: {
        NODE_ENV: 'production',
        WORKER_TYPE: 'inventory',
      },
    },
    {
      name: 'stock-sync-daemon',
      script: 'dist/daemons/stock-sync.js',
      instances: 1,
      autorestart: true,
      max_restarts: 10,
      env: {
        NODE_ENV: 'production',
      },
    },
  ],
};
```

## Monitoring & Alerting

### Worker Health Checks

```typescript
// Report worker health to monitoring system
setInterval(async () => {
  const queueStats = await emailQueue.getJobCounts();

  metrics.gauge('queue.waiting', queueStats.waiting, { queue: 'emails' });
  metrics.gauge('queue.active', queueStats.active, { queue: 'emails' });
  metrics.gauge('queue.failed', queueStats.failed, { queue: 'emails' });

  // Alert if queue is backing up
  if (queueStats.waiting > 1000) {
    logger.warn('Email queue backing up', { waiting: queueStats.waiting });

    await notifyOnCall({
      severity: 'medium',
      message: `Email queue has ${queueStats.waiting} waiting jobs`,
    });
  }
}, 60000);
```
