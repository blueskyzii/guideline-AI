# 🗄️ Guideline 06 — Database

> Database yang baik adalah yang *tidak perlu dipikirkan* saat production — karena sudah didesain dengan benar dari awal.

---

## 1. Pemilihan Database

### [ENFORCE] Pilih Database Berdasarkan Use Case, Bukan Trend

| Use Case | Database Pilihan | Alasan |
|----------|-----------------|--------|
| Data relasional (toko, user, order) | PostgreSQL | ACID, full SQL, JSON support |
| Caching, session, real-time counter | Redis | In-memory, blazing fast |
| Full-text search | PostgreSQL (pg_trgm) atau Elasticsearch | Tergantung skala |
| File/blob storage | S3, R2, atau object storage — BUKAN database | Database bukan untuk file |
| Analytics / Data warehouse | ClickHouse, BigQuery | Columnar, bukan row-based |
| Graph data (sosial network) | Neo4j atau PostgreSQL | Tergantung kompleksitas |
| Time series (metrics, IoT) | TimescaleDB (extension PostgreSQL) | Dioptimasi untuk time series |

**Prinsip:** Mulai dengan PostgreSQL untuk hampir semua kasus. Scale dengan spesialisasi nanti.

---

## 2. Schema Design

### [ENFORCE] Naming Convention yang Konsisten

```sql
-- ✅ Konvensi yang harus diikuti:

-- Nama tabel: snake_case, jamak
CREATE TABLE users (...);
CREATE TABLE product_categories (...);
CREATE TABLE order_items (...);

-- Kolom: snake_case, deskriptif
user_id        -- BUKAN: uid, userId, ID
created_at     -- BUKAN: createdAt, date_created
is_active      -- Boolean dengan prefix is_ atau has_
phone_number   -- BUKAN: phone, hp, telp

-- Primary key: selalu 'id'
-- Foreign key: [nama_tabel_singular]_id
-- Contoh: user_id, product_id, category_id

-- Timestamps: selalu ada di setiap tabel
created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
deleted_at TIMESTAMP WITH TIME ZONE  -- Untuk soft delete
```

---

### [ENFORCE] ID Strategy yang Tepat

```sql
-- ❌ Auto-increment integer — masalah di distributed system, bisa enumerate
id SERIAL PRIMARY KEY;

-- ✅ ULIDs atau UUIDs v7 (time-sorted, tidak bisa di-enumerate)
-- UUID v4 (random) — baik tapi tidak time-sortable
id UUID PRIMARY KEY DEFAULT gen_random_uuid();

-- ✅ Lebih baik: UUID v7 (time-sorted) — performa index lebih baik
-- Atau gunakan library seperti ulid di application layer
id UUID PRIMARY KEY DEFAULT uuid_generate_v7();

-- Untuk table yang butuh human-readable sequential:
-- Gunakan prefix + ulid
-- Contoh: "ORD-01H5RQWP5D4P5Z2G3N4M7B8C9" (Order ID)
```

---

### [INSIGHT] Tabel yang Sering Lupa Didesain dengan Benar

#### Tabel Users

```sql
CREATE TABLE users (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email           VARCHAR(255) NOT NULL,
  email_verified  BOOLEAN NOT NULL DEFAULT FALSE,
  phone_number    VARCHAR(20),
  display_name    VARCHAR(100) NOT NULL,
  avatar_url      TEXT,
  password_hash   TEXT,              -- NULL jika OAuth only
  role            user_role NOT NULL DEFAULT 'user',
  status          user_status NOT NULL DEFAULT 'active',
  
  -- Metadata
  last_login_at   TIMESTAMP WITH TIME ZONE,
  last_login_ip   INET,
  login_count     INTEGER NOT NULL DEFAULT 0,
  
  -- Timestamps
  created_at      TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  deleted_at      TIMESTAMP WITH TIME ZONE,  -- Soft delete

  -- Constraints
  CONSTRAINT users_email_unique UNIQUE (email),
  CONSTRAINT users_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Enum types
CREATE TYPE user_role AS ENUM ('admin', 'manager', 'staff', 'user');
CREATE TYPE user_status AS ENUM ('active', 'suspended', 'deleted');
```

#### Tabel Audit Log

```sql
-- WAJIB ada di aplikasi yang serius
CREATE TABLE audit_logs (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  
  -- Siapa
  user_id         UUID REFERENCES users(id) ON DELETE SET NULL,
  actor_ip        INET,
  user_agent      TEXT,
  
  -- Apa
  action          VARCHAR(100) NOT NULL,  -- 'product.created', 'order.cancelled'
  resource_type   VARCHAR(50) NOT NULL,   -- 'product', 'order', 'user'
  resource_id     UUID,
  
  -- Perubahan
  old_values      JSONB,
  new_values      JSONB,
  
  -- Konteks
  metadata        JSONB,
  
  created_at      TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

-- Audit log tidak boleh di-UPDATE atau DELETE (immutable)
CREATE RULE no_update_audit AS ON UPDATE TO audit_logs DO INSTEAD NOTHING;
CREATE RULE no_delete_audit AS ON DELETE TO audit_logs DO INSTEAD NOTHING;
```

---

## 3. Indexing Strategy

### [ENFORCE] Index Bukan Satu untuk Semua

```sql
-- ❌ Tidak ada index selain primary key
-- Akan lambat saat data besar

-- ✅ Index berdasarkan query pattern

-- 1. Index kolom yang sering di-WHERE
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status) WHERE deleted_at IS NULL;

-- 2. Composite index (urutan kolom penting!)
-- Query: WHERE user_id = ? AND created_at BETWEEN ? AND ?
CREATE INDEX idx_orders_user_created 
  ON orders(user_id, created_at DESC);

-- 3. Partial index (hanya index baris tertentu — lebih efisien)
-- Index hanya untuk user aktif
CREATE INDEX idx_active_users ON users(email, id) 
  WHERE status = 'active' AND deleted_at IS NULL;

-- Index hanya untuk order yang belum selesai
CREATE INDEX idx_pending_orders ON orders(created_at) 
  WHERE status IN ('pending', 'processing');

-- 4. Full-text search index
CREATE INDEX idx_products_search ON products 
  USING gin(to_tsvector('indonesian', name || ' ' || COALESCE(description, '')));

-- 5. JSONB index
CREATE INDEX idx_user_metadata ON users USING gin(metadata);
```

---

### [INSIGHT] Kapan Tidak Menambah Index

Index bukan gratis — setiap index:
- Memperlambat INSERT/UPDATE/DELETE
- Mengonsumsi storage
- Perlu di-maintain oleh database engine

**Jangan tambah index jika:**
- Kolom memiliki cardinality rendah (sedikit unique value) — contoh: `boolean`, `status` dengan 2-3 nilai, `gender`
- Tabel kecil (<10.000 baris) — full scan lebih cepat dari index lookup
- Kolom jarang digunakan dalam WHERE clause

---

## 4. Query Patterns

### [ENFORCE] N+1 Problem — Musuh Utama Performance

```javascript
// ❌ N+1 Query — sangat lambat!
const orders = await Order.findAll(); // Query 1
for (const order of orders) {
  order.items = await OrderItem.findAll({ 
    where: { orderId: order.id } 
  }); // Query N (satu per order)
}
// Total: 1 + N queries

// ✅ Eager loading / JOIN
const orders = await Order.findAll({
  include: [{ model: OrderItem }], // Cuma 2 queries (atau 1 JOIN)
});

// ✅ Atau raw SQL yang optimal
const result = await db.query(`
  SELECT 
    o.*,
    json_agg(
      json_build_object(
        'id', oi.id,
        'product_id', oi.product_id,
        'quantity', oi.quantity,
        'price', oi.price
      ) ORDER BY oi.created_at
    ) AS items
  FROM orders o
  LEFT JOIN order_items oi ON oi.order_id = o.id
  WHERE o.user_id = $1
  GROUP BY o.id
  ORDER BY o.created_at DESC
  LIMIT 20
`, [userId]);
```

---

### [ENFORCE] Pagination yang Benar

```sql
-- ❌ OFFSET pagination — semakin besar offset, semakin lambat
SELECT * FROM products ORDER BY created_at DESC LIMIT 20 OFFSET 10000;
-- Di halaman 500: database harus skip 10.000 baris!

-- ✅ Cursor-based pagination — konsisten O(log n)
-- Gunakan created_at + id sebagai cursor
SELECT * FROM products 
WHERE (created_at, id) < ($cursor_created_at, $cursor_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- Implementasi di aplikasi:
async function getProducts(cursor = null, limit = 20) {
  let query = `SELECT * FROM products`;
  const params = [limit + 1]; // +1 untuk cek hasNextPage
  
  if (cursor) {
    const [cursorDate, cursorId] = decodeCursor(cursor);
    query += ` WHERE (created_at, id) < ($2, $3)`;
    params.push(cursorDate, cursorId);
  }
  
  query += ` ORDER BY created_at DESC, id DESC LIMIT $1`;
  
  const rows = await db.query(query, params);
  const hasNextPage = rows.length > limit;
  const items = hasNextPage ? rows.slice(0, -1) : rows;
  
  const nextCursor = hasNextPage 
    ? encodeCursor(items[items.length - 1].created_at, items[items.length - 1].id)
    : null;
  
  return { items, nextCursor, hasNextPage };
}
```

---

### [INSIGHT] EXPLAIN ANALYZE — Wajib Sebelum Optimize

```sql
-- Selalu gunakan EXPLAIN ANALYZE untuk query yang lambat
EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)
SELECT p.*, c.name AS category_name
FROM products p
JOIN categories c ON c.id = p.category_id
WHERE p.status = 'active'
  AND p.price BETWEEN 50000 AND 500000
ORDER BY p.created_at DESC
LIMIT 20;

-- Yang harus dicek:
-- ✅ Index Scan (bagus)
-- ⚠️  Bitmap Heap Scan (acceptable)
-- ❌ Seq Scan pada tabel besar (perlu index)
-- ❌ Hash Join pada tabel sangat besar (mungkin perlu restructure)
-- ❌ Sort dengan disk usage (perlu index sorted)
```

---

## 5. Transactions

### [ENFORCE] Gunakan Transaction untuk Operasi yang Atomik

```javascript
// ❌ Tanpa transaction — bisa partial failure
async function createOrder(userId, items) {
  const order = await Order.create({ userId, status: 'pending' });
  
  for (const item of items) {
    await OrderItem.create({ orderId: order.id, ...item });
    await Product.decrement('stock', { where: { id: item.productId } });
  }
  // Jika salah satu gagal, order sudah terbuat tapi items mungkin tidak!
}

// ✅ Dengan transaction — all or nothing
async function createOrder(userId, items) {
  return await db.transaction(async (trx) => {
    // Buat order
    const order = await Order.create(
      { userId, status: 'pending' },
      { transaction: trx }
    );
    
    // Buat items dan update stok
    for (const item of items) {
      // Lock row saat di-read untuk hindari race condition
      const product = await Product.findByPk(item.productId, {
        lock: trx.LOCK.UPDATE,  // SELECT ... FOR UPDATE
        transaction: trx,
      });
      
      if (product.stock < item.quantity) {
        throw new Error(`Stok ${product.name} tidak cukup`);
      }
      
      await OrderItem.create(
        { orderId: order.id, ...item },
        { transaction: trx }
      );
      
      await product.decrement('stock', {
        by: item.quantity,
        transaction: trx,
      });
    }
    
    return order;
    // Jika ada error di atas → otomatis ROLLBACK semua perubahan
  });
}
```

---

## 6. Soft Delete

### [ENFORCE] Soft Delete dengan Filter yang Benar

```sql
-- Schema
ALTER TABLE products ADD COLUMN deleted_at TIMESTAMP WITH TIME ZONE;
ALTER TABLE products ADD COLUMN deleted_by UUID REFERENCES users(id);

-- View untuk query yang bersih
CREATE VIEW active_products AS
  SELECT * FROM products WHERE deleted_at IS NULL;

-- Index partial untuk performa
CREATE INDEX idx_active_products ON products(category_id, created_at)
  WHERE deleted_at IS NULL;
```

```javascript
// ❌ Selalu harus ingat menambah filter
const products = await db.query('SELECT * FROM products WHERE deleted_at IS NULL');

// ✅ Default scope yang aman
class Product extends Model {
  static addDefaultScope() {
    return {
      where: {
        deletedAt: null,
      },
    };
  }
  
  // Untuk query yang memang butuh data ter-delete
  static withDeleted() {
    return this.unscoped();
  }
}
```

---

## 7. Data Integrity

### [ENFORCE] Foreign Key Constraints Selalu Aktif

```sql
-- Selalu definisikan behavior saat parent dihapus
ALTER TABLE order_items
  ADD CONSTRAINT fk_order_items_order
  FOREIGN KEY (order_id) REFERENCES orders(id)
  ON DELETE CASCADE;    -- Hapus items jika order dihapus

ALTER TABLE orders
  ADD CONSTRAINT fk_orders_user
  FOREIGN KEY (user_id) REFERENCES users(id)
  ON DELETE RESTRICT;   -- Tolak hapus user jika masih punya order

ALTER TABLE products
  ADD CONSTRAINT fk_products_category
  FOREIGN KEY (category_id) REFERENCES categories(id)
  ON DELETE SET NULL;   -- Set null jika category dihapus
```

---

### [INSIGHT] Check Constraints — Validasi di Level Database

```sql
-- ❌ Hanya validasi di aplikasi — bisa bypass!

-- ✅ Validasi di database (last line of defense)
ALTER TABLE products
  ADD CONSTRAINT check_price_positive CHECK (price > 0),
  ADD CONSTRAINT check_stock_non_negative CHECK (stock >= 0),
  ADD CONSTRAINT check_discount_range CHECK (discount_pct BETWEEN 0 AND 100);

ALTER TABLE orders
  ADD CONSTRAINT check_status_valid 
  CHECK (status IN ('pending', 'confirmed', 'processing', 'shipped', 'delivered', 'cancelled'));

ALTER TABLE users
  ADD CONSTRAINT check_email_format
  CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');
```

---

## 8. Migration Strategy

### [ENFORCE] Migration yang Aman untuk Zero-Downtime

```sql
-- ❌ Berbahaya untuk production yang sedang berjalan:
ALTER TABLE users RENAME COLUMN name TO display_name;
-- Aplikasi lama yang pakai 'name' akan crash!

-- ✅ Zero-downtime migration (3 langkah):
-- Step 1: Tambah kolom baru (aplikasi belum pakai)
ALTER TABLE users ADD COLUMN display_name VARCHAR(100);

-- Step 2: Backfill data
UPDATE users SET display_name = name WHERE display_name IS NULL;

-- Step 3: Deploy aplikasi baru yang pakai display_name
-- Lalu hapus kolom lama setelah yakin aman:
ALTER TABLE users DROP COLUMN name;
```

**Prinsip migration yang aman:**
- Tambah kolom baru: ✅ Aman langsung
- Hapus kolom: ⚠️ Deploy aplikasi yang tidak pakai kolom itu dulu
- Rename kolom: ⚠️ Gunakan 3-step di atas
- Ubah tipe kolom: ❌ Sangat berbahaya — selalu tambah kolom baru
- Tambah constraint NOT NULL: ⚠️ Backfill + set default dulu

---

## 9. Backup & Recovery

### [ENFORCE] Backup Strategy yang Teruji

```
Backup schedule minimal:
- Full backup: Setiap hari (jam 03:00 low-traffic)
- Incremental/WAL: Setiap jam atau continuous (Point-in-Time Recovery)
- Retensi: 7 hari daily, 4 minggu weekly, 12 bulan monthly

Testing backup (WAJIB):
- Restore test: Setiap minggu ke environment test
- Hitung RTO (Recovery Time Objective): Berapa lama untuk restore
- Hitung RPO (Recovery Point Objective): Berapa banyak data yang bisa hilang
```

```bash
# PostgreSQL backup
pg_dump -Fc -Z 9 mydb > backup_$(date +%Y%m%d_%H%M%S).dump

# Restore
pg_restore -d mydb backup_20240315_030000.dump

# Point-in-Time Recovery — butuh WAL archiving yang dikonfigurasi
```
