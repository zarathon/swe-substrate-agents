# Dados e Persistência

**Gerado por**: `/swe-substrate-data`
**Data**: 2025-10-29

## Database Principal

**Tipo**: PostgreSQL 15
**Hosting**: AWS RDS Multi-AZ
**Backup**: Daily automated snapshots (retained 30 days)

### Schema Design

#### Core Tables

```sql
-- Products
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  base_price INTEGER NOT NULL, -- Price in cents
  category_id UUID REFERENCES categories(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ -- Soft delete
);

CREATE INDEX idx_products_category ON products(category_id) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_created ON products(created_at DESC);

-- Orders
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_id UUID REFERENCES customers(id),
  status VARCHAR(50) NOT NULL, -- pending, confirmed, shipped, delivered, cancelled
  total_amount INTEGER NOT NULL,
  currency VARCHAR(3) DEFAULT 'BRL',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_orders_customer ON orders(customer_id);
CREATE INDEX idx_orders_status ON orders(status) WHERE status NOT IN ('delivered', 'cancelled');
CREATE INDEX idx_orders_created ON orders(created_at DESC);

-- Order Items
CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id),
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  unit_price INTEGER NOT NULL,
  customization JSONB, -- Flexible customization data
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_order_items_order ON order_items(order_id);
CREATE INDEX idx_order_items_customization ON order_items USING GIN (customization);

-- Inventory
CREATE TABLE inventory (
  product_id UUID PRIMARY KEY REFERENCES products(id),
  available_quantity INTEGER NOT NULL DEFAULT 0,
  reserved_quantity INTEGER NOT NULL DEFAULT 0,
  last_restocked_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_inventory_low_stock ON inventory(available_quantity)
  WHERE available_quantity < 10;
```

### Migrations

**Tool**: node-pg-migrate

```bash
# Create migration
npm run migrate create add-product-variants

# Run migrations
npm run migrate up

# Rollback
npm run migrate down
```

**Convention**: Migrations are immutable. Never edit existing migrations.

### Connection Pooling

**Library**: pg + pg-pool

```typescript
const pool = new Pool({
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT),
  database: process.env.DB_NAME,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  max: 20, // Maximum pool size
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

### Query Patterns

#### Prepared Statements

```typescript
// Good - Prevents SQL injection
const result = await pool.query(
  'SELECT * FROM products WHERE category_id = $1 AND price <= $2',
  [categoryId, maxPrice]
);

// Bad - SQL injection risk
const result = await pool.query(
  `SELECT * FROM products WHERE category_id = '${categoryId}'`
);
```

#### Transactions

```typescript
async function createOrderWithInventoryUpdate(orderData) {
  const client = await pool.connect();

  try {
    await client.query('BEGIN');

    // Create order
    const orderResult = await client.query(
      'INSERT INTO orders (customer_id, total_amount) VALUES ($1, $2) RETURNING *',
      [orderData.customerId, orderData.totalAmount]
    );

    // Update inventory
    await client.query(
      'UPDATE inventory SET reserved_quantity = reserved_quantity + $1 WHERE product_id = $2',
      [orderData.quantity, orderData.productId]
    );

    await client.query('COMMIT');
    return orderResult.rows[0];
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

### Performance Optimizations

#### Indexes Strategy

- Primary keys on all tables
- Foreign keys indexed
- Composite indexes for common query patterns
- Partial indexes for filtered queries
- GIN indexes for JSONB columns

#### Query Optimization

```sql
-- Use EXPLAIN ANALYZE to identify slow queries
EXPLAIN ANALYZE
SELECT p.*, c.name as category_name
FROM products p
JOIN categories c ON p.category_id = c.id
WHERE p.base_price BETWEEN 1000 AND 5000;

-- Add materialized views for expensive aggregations
CREATE MATERIALIZED VIEW product_stats AS
SELECT
  p.id,
  p.name,
  COUNT(DISTINCT o.id) as total_orders,
  SUM(oi.quantity) as total_sold,
  AVG(r.rating) as avg_rating
FROM products p
LEFT JOIN order_items oi ON p.id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.id
LEFT JOIN reviews r ON p.id = r.product_id
GROUP BY p.id, p.name;

-- Refresh periodically
REFRESH MATERIALIZED VIEW CONCURRENTLY product_stats;
```

## Cache Layer

**Technology**: Redis 7
**Hosting**: AWS ElastiCache (cluster mode)

### Cache Strategies

#### Cache-Aside Pattern

```typescript
async function getProduct(productId: string): Promise<Product> {
  // Try cache first
  const cached = await redis.get(`product:${productId}`);
  if (cached) {
    return JSON.parse(cached);
  }

  // Cache miss - fetch from database
  const product = await db.query('SELECT * FROM products WHERE id = $1', [productId]);

  // Store in cache with TTL
  await redis.setex(`product:${productId}`, 3600, JSON.stringify(product));

  return product;
}
```

#### Write-Through Pattern

```typescript
async function updateProduct(productId: string, data: Partial<Product>) {
  // Update database
  const updated = await db.query(
    'UPDATE products SET name = $1, price = $2 WHERE id = $3 RETURNING *',
    [data.name, data.price, productId]
  );

  // Update cache immediately
  await redis.setex(`product:${productId}`, 3600, JSON.stringify(updated.rows[0]));

  return updated.rows[0];
}
```

#### Cache Invalidation

```typescript
// Invalidate on update
async function invalidateProductCache(productId: string) {
  await redis.del(`product:${productId}`);
  await redis.del(`product:${productId}:inventory`);
  await redis.del(`product:${productId}:reviews`);
}

// Pattern-based invalidation
async function invalidateCategoryCache(categoryId: string) {
  const keys = await redis.keys(`category:${categoryId}:*`);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}
```

### Session Storage

```typescript
// Store session with 7 days TTL
await redis.setex(`session:${sessionId}`, 604800, JSON.stringify(sessionData));

// Get session
const session = await redis.get(`session:${sessionId}`);

// Extend session on activity
await redis.expire(`session:${sessionId}`, 604800);
```

### Rate Limiting

```typescript
async function checkRateLimit(userId: string): Promise<boolean> {
  const key = `ratelimit:${userId}:${Math.floor(Date.now() / 60000)}`;
  const current = await redis.incr(key);

  if (current === 1) {
    await redis.expire(key, 60); // Expire after 1 minute
  }

  return current <= 1000; // Max 1000 requests per minute
}
```

### Pub/Sub for Real-Time Updates

```typescript
// Publisher (when inventory changes)
await redis.publish('inventory:updates', JSON.stringify({
  productId: 'prod_123',
  availableQuantity: 5,
}));

// Subscriber (WebSocket server)
redis.subscribe('inventory:updates');
redis.on('message', (channel, message) => {
  const update = JSON.parse(message);
  wsServer.broadcast(`inventory:${update.productId}`, update);
});
```

## Search Engine

**Technology**: Elasticsearch 8
**Hosting**: AWS OpenSearch Service

### Index Mapping

```json
{
  "products": {
    "mappings": {
      "properties": {
        "id": { "type": "keyword" },
        "name": {
          "type": "text",
          "analyzer": "standard",
          "fields": {
            "keyword": { "type": "keyword" },
            "autocomplete": {
              "type": "text",
              "analyzer": "autocomplete"
            }
          }
        },
        "description": { "type": "text" },
        "category": { "type": "keyword" },
        "price": { "type": "integer" },
        "tags": { "type": "keyword" },
        "created_at": { "type": "date" },
        "popularity_score": { "type": "float" }
      }
    }
  }
}
```

### Search Queries

```typescript
// Full-text search with filters
const results = await esClient.search({
  index: 'products',
  body: {
    query: {
      bool: {
        must: [
          {
            multi_match: {
              query: searchTerm,
              fields: ['name^3', 'description', 'tags^2'],
              type: 'best_fields',
              fuzziness: 'AUTO',
            },
          },
        ],
        filter: [
          { term: { category: 'electronics' } },
          { range: { price: { gte: 10000, lte: 50000 } } },
        ],
      },
    },
    sort: [
      { _score: 'desc' },
      { popularity_score: 'desc' },
    ],
    from: 0,
    size: 20,
  },
});
```

### Autocomplete

```typescript
const suggestions = await esClient.search({
  index: 'products',
  body: {
    suggest: {
      product_suggest: {
        prefix: userInput,
        completion: {
          field: 'name.autocomplete',
          size: 5,
          fuzzy: { fuzziness: 2 },
        },
      },
    },
  },
});
```

### Sync Strategy

**Pattern**: Event-driven sync via RabbitMQ

```typescript
// Listen for product updates
eventBus.on('product.updated', async (product) => {
  await esClient.index({
    index: 'products',
    id: product.id,
    body: {
      id: product.id,
      name: product.name,
      description: product.description,
      price: product.basePrice,
      category: product.category,
      created_at: product.createdAt,
    },
  });
});

// Bulk reindex
async function reindexAllProducts() {
  const products = await db.query('SELECT * FROM products WHERE deleted_at IS NULL');

  const body = products.rows.flatMap(doc => [
    { index: { _index: 'products', _id: doc.id } },
    {
      id: doc.id,
      name: doc.name,
      description: doc.description,
      price: doc.base_price,
      // ...
    },
  ]);

  await esClient.bulk({ body });
}
```

## File Storage

**Technology**: AWS S3
**Buckets**:
- `shopflow-products-images` - Product images
- `shopflow-user-uploads` - User customization uploads
- `shopflow-invoices` - Generated invoices (private)

### Upload Strategy

```typescript
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

// Generate presigned URL for client-side upload
async function getUploadUrl(fileKey: string, contentType: string) {
  const command = new PutObjectCommand({
    Bucket: 'shopflow-user-uploads',
    Key: fileKey,
    ContentType: contentType,
    ServerSideEncryption: 'AES256',
  });

  const uploadUrl = await getSignedUrl(s3Client, command, { expiresIn: 3600 });
  return uploadUrl;
}

// Server-side upload for generated files
async function uploadInvoice(orderId: string, pdfBuffer: Buffer) {
  await s3Client.send(new PutObjectCommand({
    Bucket: 'shopflow-invoices',
    Key: `invoices/${orderId}.pdf`,
    Body: pdfBuffer,
    ContentType: 'application/pdf',
    ServerSideEncryption: 'AES256',
    Metadata: {
      orderId,
      generatedAt: new Date().toISOString(),
    },
  }));
}
```

### CDN Integration

**Technology**: CloudFront

- Product images served via CDN
- Cache-Control headers: `public, max-age=31536000, immutable`
- Image transformations: Lambda@Edge for on-the-fly resizing

```typescript
// Generate CDN URL
function getProductImageUrl(imageKey: string, size: 'thumbnail' | 'medium' | 'large') {
  return `https://cdn.shopflow.com/products/${imageKey}?size=${size}`;
}
```

## Data Retention & Archival

### Soft Deletes

```sql
-- Mark as deleted instead of removing
UPDATE products SET deleted_at = NOW() WHERE id = $1;

-- Queries ignore soft-deleted records
SELECT * FROM products WHERE deleted_at IS NULL;
```

### Archival Strategy

- Orders older than 2 years: Move to cold storage (S3 Glacier)
- Anonymous user carts older than 30 days: Permanently delete
- Failed payment attempts: Retain for 90 days

### GDPR Compliance

```typescript
// Right to be forgotten - anonymize user data
async function anonymizeUser(userId: string) {
  await db.query(`
    UPDATE customers
    SET
      email = 'deleted-' || id || '@example.com',
      name = 'Deleted User',
      phone = NULL,
      address = NULL,
      deleted_at = NOW()
    WHERE id = $1
  `, [userId]);

  // Keep orders for accounting but anonymize
  await db.query(`
    UPDATE orders
    SET customer_id = NULL
    WHERE customer_id = $1
  `, [userId]);
}
```

## Backup & Recovery

### Automated Backups

- **PostgreSQL**: Daily snapshots, 30-day retention
- **Redis**: AOF (Append Only File) + daily RDB snapshots
- **S3**: Versioning enabled, cross-region replication

### Disaster Recovery

- **RTO** (Recovery Time Objective): 4 hours
- **RPO** (Recovery Point Objective): 1 hour
- **Runbook**: `/docs/runbooks/disaster-recovery.md`
