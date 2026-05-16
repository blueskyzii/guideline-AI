# ⚙️ GUIDELINE: Backend Engineering & API Design
> **Untuk AI Coding**: Baca seluruh dokumen ini sebelum menulis satu baris kode pun.
> Standar output: API yang **cepat, aman, efisien resource, dan tidak membuat user menunggu**.
> Catatan: Semua hal terkait database diatur di **GUIDELINE_DATABASE.md** — baca juga file tersebut.

---

## 🔴 TAHAP 0 — DISCOVERY (WAJIB, TIDAK BOLEH DILEWATI)

**Sebelum membuat arsitektur atau kode apapun — AI WAJIB menanyakan semua pertanyaan di bawah ini kepada user terlebih dahulu.**

Sampaikan dalam **satu blok sekaligus**, tunggu jawaban, baru lanjut ke perencanaan.

---

### 📋 DAFTAR PERTANYAAN DISCOVERY BACKEND

**A. Konteks Bisnis & Sistem**
1. Apa fungsi utama sistem ini? *(exchange crypto, marketplace, SaaS B2B, sistem internal, API untuk mobile app, dll)*
2. Seberapa besar skala yang diantisipasi? *(perkiraan jumlah user, transaksi per menit/hari, ukuran data)*
3. Apakah ada regulasi atau compliance yang harus dipenuhi? *(KYC/AML untuk fintech, GDPR, ISO 27001, dll)*
4. Apakah sistem ini baru dari nol atau integrasi dengan sistem yang sudah ada?
5. Siapa yang akan mengonsumsi API ini? *(web frontend, mobile app, third-party partner, internal service lain)*

**B. Fitur & Fungsionalitas**
6. Fitur utama apa saja yang perlu di-backend? *(autentikasi, manajemen user, transaksi, notifikasi, upload file, laporan, dll)*
7. Apakah ada operasi yang bersifat real-time? *(live price feed, notifikasi push, chat, update order status)*
8. Apakah ada proses berat yang berjalan lama? *(generate laporan, proses gambar/video, sinkronisasi data eksternal)*
9. Integrasi third-party apa yang dibutuhkan? *(payment gateway, SMS/email provider, KYC, blockchain node, market data API)*
10. Apakah butuh sistem role & permission? Seberapa kompleks? *(contoh: admin, manajer, user biasa, read-only)*

**C. Performance & Availability**
11. Berapa target response time yang dapat diterima? *(contoh: API listing < 200ms, halaman dashboard < 500ms)*
12. Apakah ada endpoint yang diprediksi akan sangat sering dipanggil? *(high-traffic endpoints)*
13. Apakah sistem harus berjalan 24/7 tanpa downtime? *(high availability requirement)*
14. Apakah ada kebutuhan caching? *(contoh: harga crypto di-cache 10 detik, data profil di-cache per session)*
15. Apakah ada potensi traffic spike? *(misalnya saat market crash atau event promosi)*

**D. Infrastruktur & Deployment**
16. Di mana sistem akan di-deploy? *(VPS, dedicated server, cloud: AWS/GCP/DigitalOcean, on-premise)*
17. Berapa spesifikasi server yang tersedia atau dianggarkan? *(RAM, CPU cores, storage)*
18. Apakah sudah ada Redis, queue system, atau load balancer yang tersedia?
19. Apakah butuh multi-environment? *(development, staging, production)*
20. Apakah ada CI/CD pipeline yang sudah berjalan?

**E. Tech & Prioritas**
21. Tech stack yang diinginkan atau sudah digunakan? *(Node.js/Fastify, Python/FastAPI, Go/Gin, dll)*
22. Apakah ada library atau tool yang wajib digunakan atau dihindari?
23. Apa yang HARUS selesai di versi pertama? *(must-have)*
24. Apa yang boleh ditunda? *(nice-to-have)*
25. Apakah ada constraint waktu atau budget yang perlu diperhatikan?

---

### 📌 Format Architecture Brief — Wajib Dibuat Sebelum Coding

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚙️ ARCHITECTURE BRIEF — [Nama Sistem]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fungsi utama     : ...
Skala antisipasi : ...
Konsumen API     : ...
Tech stack       : ...
Modul utama (v1) : ...
Real-time needs  : [ya / tidak — sebutkan]
Background jobs  : [ya / tidak — sebutkan]
Caching strategy : ...
Third-party      : ...
Server spec      : ...
Deployment target: ...
Must-have v1     : ...
Nice-to-have     : ...
Catatan khusus   : ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ Apakah brief ini sudah sesuai dengan yang Anda bayangkan?
  Konfirmasi sebelum saya mulai membuat arsitektur dan kode.
```

**AI WAJIB menunggu konfirmasi dari user sebelum lanjut.**

---

## 🏗️ STRUKTUR FILE — WAJIB RAPI

**Aturan utama: feature-based modules, bukan layer-based.** Semua yang berkaitan dengan satu fitur ada dalam satu folder.

### Node.js / TypeScript (Standar):
```
src/
├── app.ts                      ← App setup (middleware stack, plugin)
├── server.ts                   ← HTTP server entry point
│
├── config/
│   ├── env.ts                  ← Validasi env vars via Zod (wajib, fail fast)
│   ├── server.ts               ← Server config (port, timeout, body limit)
│   └── redis.ts                ← Redis client instance
│
├── modules/                    ← Feature-based modules
│   ├── auth/
│   │   ├── auth.router.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── auth.schema.ts      ← Zod validation schemas
│   │   └── auth.types.ts
│   ├── users/
│   │   ├── users.router.ts
│   │   ├── users.controller.ts
│   │   ├── users.service.ts
│   │   ├── users.repository.ts ← Semua DB query terisolasi di sini
│   │   └── users.types.ts
│   └── [feature]/
│       └── ... (pola sama)
│
├── shared/
│   ├── middleware/
│   │   ├── auth.middleware.ts
│   │   ├── error.middleware.ts ← Global error handler
│   │   ├── cache.middleware.ts ← HTTP caching middleware
│   │   └── rateLimit.middleware.ts
│   ├── errors/
│   │   ├── AppError.ts         ← Base custom error class
│   │   └── errorCodes.ts       ← Konstanta kode error
│   ├── cache/
│   │   ├── cache.service.ts    ← Redis wrapper (get/set/invalidate)
│   │   └── cache.keys.ts       ← Semua cache key di satu tempat
│   ├── utils/
│   │   ├── logger.ts           ← Pino structured logger
│   │   ├── pagination.ts       ← Reusable pagination helper
│   │   ├── response.ts         ← Response envelope builder
│   │   └── constants.ts        ← SEMUA magic number/string
│   └── types/
│       └── express.d.ts        ← Type augmentation
│
├── jobs/                       ← BullMQ workers
│   ├── queues.ts               ← Queue definitions
│   ├── emailWorker.ts
│   ├── notificationWorker.ts
│   └── reportWorker.ts
│
└── database/                   ← Lihat GUIDELINE_DATABASE.md
    └── [diatur terpisah]
```

**Naming Convention:**

| Tipe | Format | Contoh |
|------|--------|--------|
| File | `kebab-case.ts` | `user-repository.ts` |
| Class | `PascalCase` | `UserRepository` |
| Fungsi/variable | `camelCase` | `getUserById` |
| Konstanta | `SCREAMING_SNAKE_CASE` | `MAX_LOGIN_ATTEMPTS` |
| Tipe/Interface | `PascalCase` dengan suffix `Type`/`Dto` | `UserType`, `CreateUserDto` |

---

## 🌐 API DESIGN STANDARDS

### [WAJIB] URL Convention:
```
# Resource CRUD
GET    /api/v1/users              ← List dengan pagination
GET    /api/v1/users/:id          ← Single resource
POST   /api/v1/users              ← Create
PATCH  /api/v1/users/:id          ← Partial update (BUKAN PUT)
DELETE /api/v1/users/:id          ← Soft delete

# Nested resources
GET    /api/v1/orders/:id/items

# Non-CRUD actions (gunakan verb setelah resource)
POST   /api/v1/auth/login
POST   /api/v1/auth/logout
POST   /api/v1/auth/refresh
POST   /api/v1/orders/:id/cancel
POST   /api/v1/users/:id/verify-email
```

### [WAJIB] Response Envelope — Konsisten di Semua Endpoint:
```typescript
// ✅ Success
{
  "success": true,
  "data": { ... },
  "meta": { "requestId": "req_01HX...", "timestamp": "2025-01-15T10:30:00Z" }
}

// ✅ Success dengan pagination
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 1, "limit": 20, "total": 142,
    "totalPages": 8, "hasNext": true, "hasPrev": false
  },
  "meta": { "requestId": "req_01HX..." }
}

// ✅ Error
{
  "success": false,
  "error": {
    "code": "EMAIL_ALREADY_EXISTS",      ← Machine-readable
    "message": "Email sudah terdaftar.", ← Human-readable
    "details": [ ... ]                   ← Opsional: field-level errors
  },
  "meta": { "requestId": "req_01HX..." }
}
```

### [WAJIB] HTTP Status Codes:
| Situasi | Code |
|---------|------|
| GET sukses | `200 OK` |
| POST create sukses | `201 Created` |
| DELETE sukses | `204 No Content` |
| Validasi gagal | `422 Unprocessable Entity` |
| Business logic conflict | `409 Conflict` |
| Resource tidak ditemukan | `404 Not Found` |
| Tidak punya akses | `403 Forbidden` |
| Belum login | `401 Unauthorized` |
| Rate limit kena | `429 Too Many Requests` |

**❌ DILARANG**: Return `200 OK` dengan `{ success: false }`.

---

## ⚡ UI EXPERIENCE & RESPONSE TIME — PRIORITAS UTAMA

> Backend yang baik tidak hanya "benar" secara logika — ia harus **membuat frontend terasa cepat**.

### [WAJIB] Target Response Time:

| Jenis endpoint | Target | Strategi |
|----------------|--------|----------|
| Auth (login, register) | < 300ms | Argon2id + async, tidak blokir thread |
| List data (dengan pagination) | < 150ms | Query teroptimasi + Redis cache |
| Single resource | < 100ms | Cache-first, DB hanya fallback |
| Create / Update | < 200ms | Validasi cepat, response langsung |
| Report / export | < 500ms awal | Streaming atau background job |
| Real-time data (harga, notif) | < 50ms | WebSocket / SSE, bukan polling |

### [WAJIB] Strategi Caching — Jangan Biarkan DB Kena Setiap Request:

```typescript
// cache/cache.service.ts — Wrapper Redis yang clean
export class CacheService {
  constructor(private redis: Redis) {}

  async get<T>(key: string): Promise<T | null> {
    const data = await this.redis.get(key);
    return data ? JSON.parse(data) : null;
  }

  async set(key: string, value: unknown, ttlSeconds: number): Promise<void> {
    await this.redis.setex(key, ttlSeconds, JSON.stringify(value));
  }

  async invalidate(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length) await this.redis.del(...keys);
  }

  // Cache-aside pattern: get from cache, fallback to source
  async remember<T>(
    key: string,
    ttl: number,
    fetchFn: () => Promise<T>
  ): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached) return cached;

    const fresh = await fetchFn();
    await this.set(key, fresh, ttl);
    return fresh;
  }
}
```

```typescript
// cache/cache.keys.ts — Semua cache key terpusat
export const CacheKeys = {
  // User
  userProfile: (userId: string) => `user:profile:${userId}`,
  userPermissions: (userId: string) => `user:permissions:${userId}`,

  // Market data (short TTL)
  cryptoPrice: (symbol: string) => `market:price:${symbol}`,
  cryptoPriceAll: () => `market:prices:all`,
  orderBook: (pair: string) => `market:orderbook:${pair}`,

  // Lists (medium TTL)
  userOrders: (userId: string, page: number) => `orders:user:${userId}:page:${page}`,
  publicAssets: () => `assets:public:list`,

  // Invalidation patterns
  userAll: (userId: string) => `user:*:${userId}*`,
  marketAll: () => `market:*`,
} as const;

// TTL constants (dalam detik)
export const CacheTTL = {
  PRICE_DATA:      10,    // Harga crypto: 10 detik
  USER_PROFILE:    300,   // Profil user: 5 menit
  PERMISSIONS:     600,   // Permission: 10 menit
  PUBLIC_LIST:     60,    // List publik: 1 menit
  REPORT:          3600,  // Report statis: 1 jam
} as const;
```

### [WAJIB] Jangan Buat User Menunggu — Gunakan Background Job:

**Aturan:** Jika suatu operasi memakan waktu > 300ms atau bergantung pada service eksternal → pindahkan ke queue.

```
Wajib di-queue:
✅ Kirim email / SMS / push notification
✅ Proses KYC / verifikasi dokumen
✅ Sinkronisasi ke payment gateway
✅ Generate laporan / export CSV/PDF besar
✅ Resize / compress gambar yang diupload
✅ Webhook outbound ke sistem mitra
✅ Cleanup data expired

Boleh synchronous (response langsung):
✓ Validasi input
✓ Query data sederhana dengan cache
✓ Create/update record ringan
✓ Return status awal ("request diterima, sedang diproses")
```

```typescript
// Pattern: Immediate response + async processing
// ❌ Salah: user nunggu email terkirim (bisa 2-5 detik)
async register(dto: RegisterDto) {
  const user = await this.createUser(dto);
  await this.emailService.sendWelcome(user); // User nunggu ini
  return user;
}

// ✅ Benar: user dapat response <200ms, email kirim di background
async register(dto: RegisterDto) {
  const user = await this.createUser(dto);
  await this.emailQueue.add('send-welcome', { userId: user.id }); // Non-blocking
  return user; // Response langsung
}
```

### [WAJIB] Streaming untuk Data Besar:

```typescript
// Jangan load seluruh data ke memori — stream ke client
app.get('/reports/export', async (req, res) => {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename="report.csv"');

  const stream = db.query('SELECT ...').stream();
  const csvTransform = new CsvTransformStream();

  stream.pipe(csvTransform).pipe(res);
});
```

### [WAJIB] Kompresi Response:

```typescript
// Wajib aktifkan gzip/brotli di semua API
import compression from 'compression';

app.use(compression({
  level: 6,              // Balance speed vs size
  threshold: 1024,       // Hanya compress jika > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) return false;
    return compression.filter(req, res);
  },
}));
```

---

## 🔐 SECURITY & AUTHENTICATION

### [WAJIB] JWT Lifecycle yang Aman:
```typescript
// Access Token: 15 menit, di Authorization header
// Refresh Token: 7 hari, di httpOnly cookie SAJA

const COOKIE_OPTIONS = {
  httpOnly:  true,           // Tidak bisa diakses JS
  secure:    true,           // HTTPS only
  sameSite:  'strict' as const,
  maxAge:    7 * 24 * 60 * 60 * 1000,
  path:      '/api/v1/auth', // Hanya dikirim ke auth endpoints
};

// Selalu sertakan jti untuk token revocation
const payload = {
  sub:  user.id,
  jti:  generateULID(), // Unique token ID — bisa di-blacklist
  role: user.role,
  iat:  Math.floor(Date.now() / 1000),
};
```

### [WAJIB] Env Vars Validation — Fail Fast:
```typescript
// config/env.ts — Crash saat startup jika ada yang missing
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV:             z.enum(['development', 'test', 'production']),
  PORT:                 z.coerce.number().default(3000),
  DATABASE_URL:         z.string().url(),
  REDIS_URL:            z.string().url(),
  JWT_SECRET:           z.string().min(32),
  JWT_REFRESH_SECRET:   z.string().min(32),
  CORS_ORIGIN:          z.string().url(),
  // Tambahkan semua env vars di sini
});

export const env = envSchema.parse(process.env);
// Jika missing → crash saat startup, bukan saat runtime di production
```

### [WAJIB] Rate Limiting per Kategori Endpoint:
```typescript
// Auth: sangat ketat
const authLimit = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 menit
  max: 5,                    // 5 percobaan
  message: { success: false, error: { code: 'RATE_LIMIT_AUTH', message: 'Terlalu banyak percobaan login.' }},
});

// API umum: lebih longgar
const apiLimit = rateLimit({
  windowMs: 60 * 1000,  // 1 menit
  max: 120,
});

// Endpoint mahal (report, export): lebih ketat
const heavyLimit = rateLimit({
  windowMs: 60 * 1000,
  max: 5,
  message: { success: false, error: { code: 'RATE_LIMIT_HEAVY', message: 'Terlalu banyak request berat.' }},
});
```

---

## 🚦 ERROR HANDLING — TERPUSAT

### [WAJIB] AppError Class:
```typescript
// shared/errors/AppError.ts
export class AppError extends Error {
  constructor(
    public readonly statusCode: number,
    public readonly code: string,
    public readonly message: string,
    public readonly isOperational = true,
    public readonly details?: unknown,
  ) {
    super(message);
    Object.setPrototypeOf(this, new.target.prototype);
  }
}

// Contoh throw di service layer:
throw new AppError(409, 'EMAIL_ALREADY_EXISTS', 'Email sudah terdaftar.');
throw new AppError(404, 'USER_NOT_FOUND', 'User tidak ditemukan.');
throw new AppError(422, 'VALIDATION_ERROR', 'Input tidak valid.', true, zodErrors);
```

### [WAJIB] Global Error Handler:
```typescript
// shared/middleware/error.middleware.ts
export const globalErrorHandler = (err, req, res, next) => {
  const requestId = req.id || generateULID();

  logger.error({ requestId, code: err.code, error: err.message,
                 path: req.path, method: req.method, stack: err.stack });

  if (err instanceof AppError && err.isOperational) {
    return res.status(err.statusCode).json({
      success: false,
      error: { code: err.code, message: err.message, details: err.details },
      meta: { requestId },
    });
  }

  // Programming error — jangan expose detail ke client
  return res.status(500).json({
    success: false,
    error: { code: 'INTERNAL_ERROR', message: 'Terjadi kesalahan sistem.' },
    meta: { requestId },
  });
};
```

---

## 📊 EFISIENSI RESOURCE SERVER

### [WAJIB] Jangan Boros Memori:

```typescript
// ❌ Load semua data ke RAM
const allUsers = await db.users.findMany(); // 100.000 rows masuk memori

// ✅ Proses secara chunk
const CHUNK_SIZE = 500;
let cursor = 0;
while (true) {
  const batch = await db.users.findMany({ take: CHUNK_SIZE, skip: cursor });
  if (!batch.length) break;
  await processBatch(batch);
  cursor += CHUNK_SIZE;
}
```

### [WAJIB] Connection Pool — Jangan Buka Koneksi Baru Tiap Request:
```typescript
// config/database.ts — Pool diset sesuai kapasitas server
// Lihat GUIDELINE_DATABASE.md untuk konfigurasi lengkap

// Aturan kasar: maxConnections = (CPU cores × 2) + spare
// Server 2 core → max 6-8 connections
// Server 4 core → max 10-12 connections
```

### [WAJIB] Graceful Shutdown — Selesaikan Request yang Sedang Berjalan:
```typescript
// server.ts
const server = app.listen(env.PORT);

const shutdown = async (signal: string) => {
  logger.info(`Received ${signal}, starting graceful shutdown`);

  server.close(async () => {
    await db.$disconnect();   // Tutup DB connection pool
    await redis.quit();       // Tutup Redis connection
    await emailQueue.close(); // Drain queue
    logger.info('Graceful shutdown complete');
    process.exit(0);
  });

  // Force shutdown jika melebihi 30 detik
  setTimeout(() => { logger.error('Forced shutdown'); process.exit(1); }, 30_000);
};

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT',  () => shutdown('SIGINT'));
```

### [WAJIB] Timeout pada Semua External Call:
```typescript
// Jangan biarkan external API menggantung request user
const response = await fetch('https://api.coingecko.com/prices', {
  signal: AbortSignal.timeout(5_000), // 5 detik max
});

// Prisma query timeout
const user = await db.users.findUnique({
  where: { id },
  // Atur di connection: ?connect_timeout=5&statement_timeout=10000
});
```

---

## 📋 KONSTANTA — DILARANG MAGIC NUMBER

```typescript
// shared/utils/constants.ts

// Time
export const ONE_MINUTE_MS     = 60 * 1000;
export const ONE_HOUR_MS       = 60 * ONE_MINUTE_MS;
export const ONE_DAY_MS        = 24 * ONE_HOUR_MS;
export const ONE_DAY_SECONDS   = 24 * 60 * 60;

// Auth
export const MAX_LOGIN_ATTEMPTS    = 5;
export const LOGIN_WINDOW_MINUTES  = 15;
export const ACCESS_TOKEN_TTL      = '15m';
export const REFRESH_TOKEN_TTL     = '7d';
export const MIN_PASSWORD_LENGTH   = 8;

// Pagination
export const DEFAULT_PAGE_SIZE = 20;
export const MAX_PAGE_SIZE     = 100;

// File upload
export const MAX_FILE_SIZE_MB    = 10;
export const MAX_FILE_SIZE_BYTES = MAX_FILE_SIZE_MB * 1024 * 1024;
export const ALLOWED_IMAGE_TYPES = ['image/jpeg', 'image/png', 'image/webp'] as const;

// Rate limit
export const AUTH_RATE_LIMIT_MAX     = 5;
export const AUTH_RATE_WINDOW_MS     = 15 * ONE_MINUTE_MS;
export const GENERAL_RATE_LIMIT_MAX  = 120;
export const HEAVY_RATE_LIMIT_MAX    = 5;

// Background job
export const JOB_MAX_ATTEMPTS    = 3;
export const JOB_BACKOFF_DELAY   = 5_000; // ms
export const JOB_TIMEOUT_MS      = 30_000;

// Server
export const REQUEST_TIMEOUT_MS  = 30_000;
export const BODY_LIMIT_MB       = 10;
export const KEEP_ALIVE_TIMEOUT  = 65_000; // Lebih dari load balancer (60s)
```

---

## ✅ CHECKLIST FINAL SEBELUM DELIVER

**Discovery & Planning:**
- [ ] Semua 25 pertanyaan discovery sudah ditanyakan
- [ ] Architecture Brief sudah dikonfirmasi user sebelum coding
- [ ] Kebutuhan caching, queue, dan real-time sudah diidentifikasi

**API & Response:**
- [ ] Semua response pakai envelope `{ success, data, meta }`
- [ ] HTTP status code tepat (tidak ada 200 dengan success false)
- [ ] Semua input divalidasi via Zod sebelum masuk service layer
- [ ] Response dikompresi (gzip/brotli aktif)

**Performance & UX:**
- [ ] Target response time < 200ms untuk endpoint umum
- [ ] Caching terpasang untuk data yang sering diakses
- [ ] Semua operasi lambat dipindah ke background queue
- [ ] Streaming dipakai untuk data besar / export
- [ ] Timeout terpasang untuk semua external API call
- [ ] Tidak ada proses yang bisa membuat user menunggu > 300ms

**Security:**
- [ ] Tidak ada hardcoded secret
- [ ] Semua env vars divalidasi saat startup (fail fast)
- [ ] Rate limiting aktif — berbeda per kategori endpoint
- [ ] Refresh token di httpOnly cookie, bukan response body
- [ ] CORS hanya mengizinkan origin yang terdaftar

**Resource & Server:**
- [ ] Connection pool dikonfigurasi sesuai spec server
- [ ] Tidak ada load seluruh data ke RAM tanpa batasan
- [ ] Graceful shutdown terpasang (SIGTERM / SIGINT)
- [ ] Tidak ada memory leak dari event listener / interval tanpa cleanup

**Struktur & Kode:**
- [ ] Feature-based module structure (bukan layer-based)
- [ ] Naming convention konsisten
- [ ] Tidak ada magic number — semua di `constants.ts`
- [ ] Global error handler terpasang
- [ ] Logging structured JSON dengan requestId
- [ ] README.md ada: cara install, cara run, env vars yang dibutuhkan
