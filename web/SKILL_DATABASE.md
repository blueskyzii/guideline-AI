# 🗄️ GUIDELINE: Database Engineering
> **Untuk AI Coding**: Baca dokumen ini setiap kali menyentuh hal yang berhubungan dengan database.
> Dokumen ini adalah pasangan dari **GUIDELINE_BACKEND.md** — keduanya harus dibaca bersama.
> Standar output: schema yang **aman, efisien, dan bisa bertumbuh** tanpa rewrite ulang.

---

## 🔴 TAHAP 0 — DISCOVERY DATABASE (WAJIB, TIDAK BOLEH DILEWATI)

**AI WAJIB menanyakan semua pertanyaan ini sebelum mendesain schema apapun.**
Kirim dalam satu blok, tunggu jawaban, baru lanjut.

---

### 📋 DAFTAR PERTANYAAN DISCOVERY DATABASE

**A. Karakteristik Data**
1. Entitas/data utama apa saja yang perlu disimpan? *(contoh: users, orders, transactions, assets, wallets, notifications)*
2. Mana yang paling sering dibaca? Mana yang paling sering ditulis?
3. Apakah ada data yang bersifat time-series? *(harga historis, log aktivitas, candlestick data)*
4. Apakah ada data yang sangat besar dan terus bertumbuh? *(log, transaksi, audit trail)*
5. Apakah data perlu di-archive atau dihapus setelah periode tertentu? *(data retention policy)*

**B. Relasi & Integritas**
6. Relasi antar entitas yang kritis apa saja? *(contoh: user memiliki banyak wallet, wallet memiliki banyak transaksi)*
7. Apakah ada data yang tidak boleh dihapus secara permanen? *(soft delete needed?)*
8. Apakah ada data historis yang wajib tersimpan untuk audit? *(contoh: perubahan harga, riwayat status order)*
9. Apakah ada multi-tenancy? *(data tiap tenant harus terisolasi)*

**C. Volume & Performance**
10. Perkiraan jumlah baris untuk tabel terbesar dalam 1 tahun?
11. Berapa rata-rata transaksi per menit yang perlu ditangani?
12. Query apa yang paling sering dieksekusi? *(sering bantu tentukan index)*
13. Apakah ada query kompleks yang melibatkan banyak JOIN atau agregasi?
14. Apakah butuh full-text search? *(pencarian nama aset, riwayat transaksi)*

**D. Infrastruktur**
15. Database engine apa yang digunakan atau diinginkan? *(PostgreSQL direkomendasikan, MySQL, dll)*
16. Apakah tersedia read replica untuk offload query baca?
17. Berapa spesifikasi server database? *(RAM sangat mempengaruhi config connection pool dan shared_buffers)*
18. Apakah butuh database terpisah untuk data analytics / reporting? *(OLAP vs OLTP)*
19. Apakah ada mekanisme backup yang sudah berjalan?

**E. Keamanan & Compliance**
20. Apakah ada kolom yang menyimpan data sensitif yang perlu dienkripsi? *(nomor KTP, rekening bank, seed phrase)*
21. Apakah ada regulasi yang mengatur penyimpanan data? *(GDPR, OJK, PBI untuk fintech Indonesia)*
22. Apakah perlu row-level security? *(user hanya bisa akses data miliknya)*

---

### 📌 Format Database Brief — Wajib Dibuat Sebelum Schema

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🗄️ DATABASE BRIEF — [Nama Sistem]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Engine          : PostgreSQL [versi]
Entitas utama   : ...
Tabel terbesar  : ... (estimasi rows/tahun)
Volume transaksi: ... per menit
Query terpanas  : ...
Data sensitif   : ... (perlu enkripsi)
Soft delete     : [ya / tidak]
Audit log       : [ya / tidak]
Time-series data: [ya / tidak — tabel apa]
Read replica    : [tersedia / tidak]
Server RAM      : ... (untuk tuning pool)
Compliance      : ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ Apakah brief ini sudah sesuai sebelum saya mulai desain schema?
```

---

## 📐 SCHEMA DESIGN STANDARDS

### [WAJIB] Konvensi Penamaan:
```sql
-- Tabel: plural, snake_case
CREATE TABLE users          (...);
CREATE TABLE crypto_assets  (...);
CREATE TABLE order_items    (...);

-- Kolom: snake_case
user_id, created_at, deleted_at, is_active, total_amount

-- Index: prefix tipe
idx_users_email
idx_orders_user_id_status
uniq_users_email
fk_orders_user_id

-- Foreign key constraint: prefix fk_
CONSTRAINT fk_orders_user_id FOREIGN KEY (user_id) REFERENCES users(id)
```

### [WAJIB] Kolom Standar yang Wajib Ada di Setiap Tabel:
```sql
CREATE TABLE [nama_tabel] (
  -- Primary key: UUID, bukan auto-increment integer
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Audit timestamps: TIMESTAMPTZ (dengan timezone), bukan TIMESTAMP
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Soft delete (jika diperlukan)
  deleted_at  TIMESTAMPTZ,

  -- Kolom bisnis di sini...
);

-- Trigger untuk auto-update updated_at
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN NEW.updated_at = NOW(); RETURN NEW; END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_[nama_tabel]_updated_at
  BEFORE UPDATE ON [nama_tabel]
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

**Mengapa UUID bukan integer?**
- Tidak mudah di-enumerate oleh attacker (`/users/1`, `/users/2`, dst.)
- Aman untuk merge data dari beberapa sumber
- UUID v7 (time-sorted) baik untuk index performance

### [WAJIB] Tipe Data yang Tepat:
```sql
-- Uang / amount finansial: DECIMAL, BUKAN float
total_amount   DECIMAL(20, 8) NOT NULL  -- 8 desimal untuk crypto
fee_amount     DECIMAL(20, 8) NOT NULL DEFAULT 0

-- Harga crypto: DECIMAL dengan presisi tinggi
price          DECIMAL(30, 10) NOT NULL

-- Status / enum: gunakan PostgreSQL ENUM atau VARCHAR dengan CHECK
status         VARCHAR(20) NOT NULL DEFAULT 'pending'
               CHECK (status IN ('pending', 'processing', 'completed', 'failed', 'cancelled'))

-- Flag boolean
is_verified    BOOLEAN NOT NULL DEFAULT FALSE
is_active      BOOLEAN NOT NULL DEFAULT TRUE

-- JSON dinamis (gunakan JSONB, bukan JSON)
metadata       JSONB DEFAULT '{}'

-- File path / URL: VARCHAR dengan panjang cukup
avatar_url     VARCHAR(500)
```

**❌ DILARANG**: `FLOAT` atau `DOUBLE` untuk uang — rounding error menyebabkan selisih finansial.

---

## 🏗️ STRUKTUR FILE MIGRATION

```
database/
├── migrations/
│   ├── 001_create_users.sql
│   ├── 002_create_crypto_assets.sql
│   ├── 003_create_wallets.sql
│   ├── 004_create_orders.sql
│   ├── 005_create_transactions.sql
│   ├── 006_create_audit_logs.sql
│   └── 007_add_users_kyc_status.sql    ← Tambah kolom, jangan ubah migration lama
├── seeds/
│   ├── 001_seed_crypto_assets.sql      ← Data awal untuk development
│   └── 002_seed_admin_user.sql
└── functions/
    ├── update_updated_at.sql           ← Reusable trigger functions
    └── soft_delete_view.sql            ← Views untuk soft-delete filtering
```

### [WAJIB] Format Migration — Selalu Ada UP dan DOWN:
```sql
-- migrations/004_create_orders.sql

-- ============================================================
-- UP — Jalankan untuk apply migration
-- ============================================================
CREATE TABLE orders (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL,
  asset_id        UUID NOT NULL,
  order_type      VARCHAR(10) NOT NULL CHECK (order_type IN ('buy', 'sell')),
  status          VARCHAR(20) NOT NULL DEFAULT 'pending'
                  CHECK (status IN ('pending', 'processing', 'completed', 'failed', 'cancelled')),
  quantity        DECIMAL(20, 8) NOT NULL CHECK (quantity > 0),
  price           DECIMAL(30, 10) NOT NULL CHECK (price > 0),
  total_amount    DECIMAL(20, 8) NOT NULL,
  fee_amount      DECIMAL(20, 8) NOT NULL DEFAULT 0,
  notes           TEXT,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at      TIMESTAMPTZ,

  CONSTRAINT fk_orders_user    FOREIGN KEY (user_id)  REFERENCES users(id)         ON DELETE RESTRICT,
  CONSTRAINT fk_orders_asset   FOREIGN KEY (asset_id) REFERENCES crypto_assets(id) ON DELETE RESTRICT
);

CREATE INDEX idx_orders_user_id        ON orders(user_id)         WHERE deleted_at IS NULL;
CREATE INDEX idx_orders_status         ON orders(status)          WHERE deleted_at IS NULL;
CREATE INDEX idx_orders_created_at     ON orders(created_at DESC) WHERE deleted_at IS NULL;
CREATE INDEX idx_orders_user_status    ON orders(user_id, status) WHERE deleted_at IS NULL;

CREATE TRIGGER trg_orders_updated_at
  BEFORE UPDATE ON orders
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();

-- ============================================================
-- DOWN — Jalankan untuk rollback
-- ============================================================
DROP TRIGGER IF EXISTS trg_orders_updated_at ON orders;
DROP INDEX  IF EXISTS idx_orders_user_status;
DROP INDEX  IF EXISTS idx_orders_user_id;
DROP INDEX  IF EXISTS idx_orders_status;
DROP INDEX  IF EXISTS idx_orders_created_at;
DROP TABLE  IF EXISTS orders;
```

---

## ⚡ INDEXING — QUERY PERFORMANCE

### [WAJIB] Aturan Indexing:

**Index WAJIB dibuat untuk:**
- Primary key (otomatis)
- Foreign keys — setiap FK yang tidak ada index-nya adalah N+1 trap
- Kolom yang sering ada di `WHERE` clause
- Kolom yang sering ada di `ORDER BY` (terutama `created_at DESC`)
- Kolom yang sering di-JOIN

**Gunakan Partial Index untuk efisiensi:**
```sql
-- Index hanya untuk data aktif (tidak termasuk yang sudah deleted)
CREATE INDEX idx_users_email_active
  ON users(email)
  WHERE deleted_at IS NULL;

-- Index untuk status tertentu (jika satu status sangat dominan)
CREATE INDEX idx_orders_pending
  ON orders(user_id, created_at)
  WHERE status = 'pending' AND deleted_at IS NULL;
```

**Gunakan Composite Index sesuai urutan query:**
```sql
-- Query: WHERE user_id = ? AND status = ? ORDER BY created_at DESC
-- Index harus mengikuti urutan kolom yang sama
CREATE INDEX idx_orders_user_status_time
  ON orders(user_id, status, created_at DESC)
  WHERE deleted_at IS NULL;
```

**Index JANGAN dibuat jika:**
- Tabel kecil (< 10.000 baris) — full scan lebih cepat
- Kolom dengan kardinalitas sangat rendah (contoh: `is_active` dengan 99% TRUE) — kecuali partial index
- Terlalu banyak index pada tabel yang sering INSERT/UPDATE — setiap index memperlambat write

### [WAJIB] Deteksi Query Lambat:
```sql
-- Aktifkan slow query logging di postgresql.conf
log_min_duration_statement = 100    -- Log semua query > 100ms

-- Gunakan EXPLAIN ANALYZE untuk debug query lambat
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.*, o.* FROM users u
JOIN orders o ON o.user_id = u.id
WHERE u.email = 'test@example.com'
ORDER BY o.created_at DESC;

-- Cari index yang tidak pernah dipakai (jalankan secara berkala)
SELECT schemaname, tablename, indexname, idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
ORDER BY tablename;
```

---

## 🔒 AUDIT LOG — DATA YANG TIDAK BOLEH HILANG

### [WAJIB] Tabel Audit Log (Immutable):
```sql
-- Tabel ini HANYA bisa di-INSERT, tidak boleh UPDATE atau DELETE
CREATE TABLE audit_logs (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  table_name   VARCHAR(100) NOT NULL,
  record_id    UUID NOT NULL,
  action       VARCHAR(10) NOT NULL CHECK (action IN ('INSERT', 'UPDATE', 'DELETE')),
  actor_id     UUID,                       -- User yang melakukan (NULL jika sistem)
  actor_type   VARCHAR(20) DEFAULT 'user', -- 'user', 'system', 'admin'
  old_data     JSONB,                      -- Data sebelum perubahan
  new_data     JSONB,                      -- Data sesudah perubahan
  ip_address   INET,
  user_agent   TEXT,
  created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW()
  -- Tidak ada updated_at dan deleted_at — ini log immutable
);

CREATE INDEX idx_audit_table_record ON audit_logs(table_name, record_id);
CREATE INDEX idx_audit_actor        ON audit_logs(actor_id) WHERE actor_id IS NOT NULL;
CREATE INDEX idx_audit_created_at   ON audit_logs(created_at DESC);

-- Cegah UPDATE dan DELETE via policy (PostgreSQL)
CREATE RULE no_update_audit AS ON UPDATE TO audit_logs DO INSTEAD NOTHING;
CREATE RULE no_delete_audit AS ON DELETE TO audit_logs DO INSTEAD NOTHING;
```

---

## 🔄 SOFT DELETE — JANGAN HAPUS DATA SUNGGUHAN

```sql
-- Pattern: gunakan deleted_at, bukan benar-benar DELETE
-- Semua query WAJIB filter WHERE deleted_at IS NULL

-- View untuk memudahkan query aktif
CREATE VIEW active_users AS
  SELECT * FROM users WHERE deleted_at IS NULL;

CREATE VIEW active_orders AS
  SELECT * FROM orders WHERE deleted_at IS NULL;
```

```typescript
// Prisma: selalu gunakan soft delete
// ❌ Salah
await db.users.delete({ where: { id } });

// ✅ Benar
await db.users.update({
  where: { id },
  data: { deletedAt: new Date() },
});

// ✅ Selalu filter aktif
await db.users.findMany({
  where: { deletedAt: null, ... },
});
```

---

## 📦 CONNECTION POOL — TUNING SESUAI SERVER

### [WAJIB] Konfigurasi Pool:
```typescript
// config/database.ts
import { PrismaClient } from '@prisma/client';

// Rumus: maxConnections ≈ (CPU cores × 2) + 2, jangan lebih dari 20
// Server 1 core / 1GB RAM → max 4 connections
// Server 2 core / 2GB RAM → max 6 connections
// Server 4 core / 4GB RAM → max 10 connections
// Server 8 core / 8GB RAM → max 18 connections

const DATABASE_URL_WITH_POOL =
  `${process.env.DATABASE_URL}?connection_limit=10&pool_timeout=20&connect_timeout=10&statement_timeout=30000`;

export const db = new PrismaClient({
  datasources: { db: { url: DATABASE_URL_WITH_POOL } },
  log: process.env.NODE_ENV === 'development'
    ? ['query', 'warn', 'error']
    : ['warn', 'error'],
});
```

### [WAJIB] Konfigurasi postgresql.conf (sesuaikan dengan RAM):
```ini
# Untuk server 2GB RAM:
shared_buffers          = 512MB        # 25% dari RAM
effective_cache_size    = 1536MB       # 75% dari RAM
work_mem                = 8MB          # Per query / sort operation
maintenance_work_mem    = 128MB        # Untuk VACUUM, CREATE INDEX
max_connections         = 50           # Jumlah max koneksi ke DB langsung
wal_buffers             = 16MB
checkpoint_completion_target = 0.9
random_page_cost        = 1.1          # Jika pakai SSD
effective_io_concurrency = 200         # Jika pakai SSD

# Logging slow queries
log_min_duration_statement = 100       # ms
log_line_prefix = '%t [%p]: [%l-1] '
```

---

## 🚫 N+1 QUERY — WAJIB DIHILANGKAN

```typescript
// ❌ N+1: 1 query untuk orders + N query untuk tiap user
const orders = await db.orders.findMany({ where: { status: 'pending' } });
for (const order of orders) {
  order.user = await db.users.findUnique({ where: { id: order.userId } }); // N queries!
}

// ✅ 1 query dengan JOIN (Prisma eager loading)
const orders = await db.orders.findMany({
  where: { status: 'pending', deletedAt: null },
  include: {
    user:  { select: { id: true, name: true, email: true } }, // Pilih kolom yang perlu saja
    asset: { select: { id: true, symbol: true, name: true } },
  },
  orderBy: { createdAt: 'desc' },
  take: 20,
});

// ✅ Untuk kasus kompleks: raw query dengan JOIN
const result = await db.$queryRaw`
  SELECT o.id, o.quantity, o.price, o.status,
         u.name AS user_name, u.email,
         a.symbol AS asset_symbol
  FROM orders o
  JOIN users u ON u.id = o.user_id
  JOIN crypto_assets a ON a.id = o.asset_id
  WHERE o.status = 'pending'
    AND o.deleted_at IS NULL
  ORDER BY o.created_at DESC
  LIMIT 20
`;
```

---

## 📊 PATTERN UNTUK DATA TIME-SERIES

Untuk data yang terus bertumbuh (harga historis, log, transaksi):

```sql
-- Partitioning berdasarkan waktu (PostgreSQL native partitioning)
CREATE TABLE price_history (
  id          UUID NOT NULL DEFAULT gen_random_uuid(),
  asset_id    UUID NOT NULL,
  price_usd   DECIMAL(30, 10) NOT NULL,
  volume_24h  DECIMAL(30, 2),
  market_cap  DECIMAL(30, 2),
  recorded_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
) PARTITION BY RANGE (recorded_at);

-- Buat partisi per bulan (otomatis dengan pg_partman atau manual)
CREATE TABLE price_history_2025_01 PARTITION OF price_history
  FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE price_history_2025_02 PARTITION OF price_history
  FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');

-- Index per partisi otomatis inherited
CREATE INDEX ON price_history(asset_id, recorded_at DESC);
```

---

## ✅ CHECKLIST FINAL SEBELUM DELIVER

**Discovery & Planning:**
- [ ] Semua 22 pertanyaan discovery database sudah ditanyakan
- [ ] Database Brief sudah dikonfirmasi user sebelum schema dibuat

**Schema:**
- [ ] Semua tabel punya `id` (UUID), `created_at`, `updated_at`
- [ ] Semua tabel yang perlu soft-delete punya `deleted_at`
- [ ] Tidak ada `FLOAT`/`DOUBLE` untuk kolom finansial — pakai `DECIMAL`
- [ ] Semua enum menggunakan `CHECK` constraint atau PostgreSQL ENUM
- [ ] Foreign key constraint terdefinisi di schema, bukan hanya di ORM

**Indexing:**
- [ ] Semua foreign key ada index-nya
- [ ] Partial index dipakai untuk filter `WHERE deleted_at IS NULL`
- [ ] Composite index urutannya sesuai dengan query yang sering dijalankan
- [ ] Tidak ada over-indexing pada tabel write-heavy

**Migration:**
- [ ] Setiap migration punya bagian UP dan DOWN
- [ ] Urutan migration bernomor dan berurutan
- [ ] Migration tidak mengubah data secara destruktif tanpa backup

**Query Safety:**
- [ ] Tidak ada query tanpa `LIMIT` / pagination
- [ ] Tidak ada N+1 query problem
- [ ] Semua soft-delete query filter `WHERE deleted_at IS NULL`
- [ ] Semua kolom sensitif tidak di-return jika tidak dibutuhkan (`SELECT *` → pilih kolom)

**Audit & Keamanan:**
- [ ] Tabel `audit_logs` ada untuk semua aksi sensitif
- [ ] Kolom sensitif dienkripsi di level aplikasi (bukan hanya di transport)
- [ ] Connection pool dikonfigurasi sesuai spec server
- [ ] Slow query logging aktif (`log_min_duration_statement = 100`)

**Struktur File:**
- [ ] Folder `database/migrations/` rapi dan bernomor
- [ ] Setiap migration file standalone dan bisa di-rollback
- [ ] Seeds terpisah dari migrations
