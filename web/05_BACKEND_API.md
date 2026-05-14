# ⚙️ Guideline 05 — Backend & API

> Backend yang baik adalah yang *tidak terasa* — cepat, reliable, dan predictable.

---

## 1. Arsitektur API

### [ENFORCE] REST API yang Konsisten dan Predictable

#### Naming Convention URL

```
# Resource: jamak, lowercase, pakai hyphen (bukan underscore)
GET    /api/v1/products          → List semua produk
GET    /api/v1/products/:id      → Detail produk
POST   /api/v1/products          → Buat produk baru
PUT    /api/v1/products/:id      → Update seluruh produk
PATCH  /api/v1/products/:id      → Update sebagian produk
DELETE /api/v1/products/:id      → Hapus produk

# Nested resources (relasi)
GET    /api/v1/orders/:id/items  → Item dari order tertentu
POST   /api/v1/orders/:id/items  → Tambah item ke order

# Actions (non-CRUD — gunakan kata kerja)
POST   /api/v1/orders/:id/cancel   → Cancel order
POST   /api/v1/users/:id/activate  → Aktivasi user
POST   /api/v1/auth/login          → Login
POST   /api/v1/auth/refresh        → Refresh token
POST   /api/v1/auth/logout         → Logout
```

---

### [ENFORCE] Response Format yang Konsisten

**Setiap response harus menggunakan format yang sama:**

```json
// ✅ Success response
{
  "success": true,
  "data": {
    "id": "usr_01J5K2M",
    "name": "Rini Anggraini",
    "email": "rini@example.com",
    "createdAt": "2024-03-15T09:30:00Z"
  },
  "meta": {
    "requestId": "req_8f2a9c1d"
  }
}

// ✅ Success list response (dengan pagination)
{
  "success": true,
  "data": [...],
  "pagination": {
    "page": 1,
    "perPage": 20,
    "total": 142,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPrevPage": false
  }
}

// ✅ Error response
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Data yang dikirim tidak valid",
    "details": [
      {
        "field": "email",
        "message": "Format email tidak valid"
      },
      {
        "field": "password",
        "message": "Password minimal 8 karakter"
      }
    ]
  },
  "meta": {
    "requestId": "req_8f2a9c1d"
  }
}
```

---

### [INSIGHT] HTTP Status Code yang Sering Salah

```
200 OK              → Request berhasil, ada data di response
201 Created         → Resource berhasil dibuat (POST)
204 No Content      → Berhasil, TIDAK ada data di response (DELETE, PUT)
400 Bad Request     → Data dari client salah/tidak valid
401 Unauthorized    → Belum login / token expired
403 Forbidden       → Sudah login tapi tidak punya izin
404 Not Found       → Resource tidak ada
409 Conflict        → Konflik (email sudah terdaftar, stok habis)
422 Unprocessable   → Data valid secara format tapi gagal aturan bisnis
429 Too Many        → Rate limit terlewati
500 Internal Error  → Server error (jangan expose detail ke client)
503 Unavailable     → Server down / maintenance
```

**❌ Kesalahan umum:** Selalu return 200 dengan `{ success: false }` di body.  
Ini menyulitkan client untuk handle error dengan interceptor.

---

## 2. Authentication & Authorization

### [ENFORCE] JWT yang Benar

```javascript
// ❌ JWT yang tidak aman
const token = jwt.sign({ userId: user.id }, 'secret123');
// Problem: secret lemah, tidak ada expiry, payload terlalu banyak

// ✅ JWT yang proper
const accessToken = jwt.sign(
  {
    sub: user.id,           // Subject
    role: user.role,        // Minimal info yang dibutuhkan
    jti: crypto.randomUUID(), // JWT ID (untuk revocation)
  },
  process.env.JWT_SECRET,   // Dari environment variable
  {
    expiresIn: '15m',       // Short-lived access token
    issuer: 'myapp.com',
    audience: 'myapp.com',
  }
);

const refreshToken = jwt.sign(
  { sub: user.id, jti: crypto.randomUUID() },
  process.env.JWT_REFRESH_SECRET,
  { expiresIn: '7d' }
);
```

---

### [ENFORCE] Simpan Token dengan Benar

```javascript
// ❌ JANGAN simpan JWT di localStorage
localStorage.setItem('token', accessToken);
// Rentan XSS attack — script apapun bisa baca

// ✅ Access token di memory (React state / in-memory variable)
// Refresh token di httpOnly cookie (tidak bisa dibaca JS)

// Server: set refresh token
res.cookie('refreshToken', refreshToken, {
  httpOnly: true,          // Tidak bisa dibaca JS
  secure: true,            // Hanya via HTTPS
  sameSite: 'strict',      // CSRF protection
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 hari
  path: '/api/auth',       // Hanya dikirim ke auth endpoints
});
```

---

### [INSIGHT] Authorization: RBAC vs ABAC

**RBAC (Role-Based):** Sederhana, cocok untuk kebanyakan app

```javascript
// Roles: admin, manager, staff, user
const permissions = {
  admin:   ['*'],                                    // Semua akses
  manager: ['products.*', 'orders.*', 'reports.read'],
  staff:   ['orders.read', 'orders.update'],
  user:    ['orders.read.own', 'profile.*'],         // Hanya milik sendiri
};

function can(user, action) {
  const userPerms = permissions[user.role] || [];
  return userPerms.includes('*') ||
         userPerms.includes(action) ||
         userPerms.includes(action.split('.').slice(0, -1).join('.') + '.*');
}
```

**ABAC (Attribute-Based):** Untuk kebutuhan yang lebih granular

```javascript
// "User bisa edit produk jika: pemilik toko, atau staff yang ditunjuk"
async function canEditProduct(user, product) {
  if (user.role === 'admin') return true;
  if (product.ownerId === user.id) return true;
  if (await isAssignedStaff(user.id, product.storeId)) return true;
  return false;
}
```

---

## 3. Input Validation

### [ENFORCE] Validate di Semua Layer

```
Client → (validate format) 
  ↓
API Gateway → (validate auth, rate limit)
  ↓  
Controller → (validate schema — WAJIB, jangan percaya client)
  ↓
Service → (validate business rules)
  ↓
Database → (validate constraints — last defense)
```

```javascript
// Contoh validasi di controller (menggunakan Zod)
import { z } from 'zod';

const createProductSchema = z.object({
  name: z.string()
    .min(3, 'Nama minimal 3 karakter')
    .max(100, 'Nama maksimal 100 karakter')
    .trim(),
  
  price: z.number()
    .positive('Harga harus lebih dari 0')
    .max(999_999_999, 'Harga terlalu besar'),
  
  stock: z.number()
    .int('Stok harus bilangan bulat')
    .min(0, 'Stok tidak boleh negatif'),
  
  category: z.enum(['electronics', 'clothing', 'food', 'other']),
  
  description: z.string()
    .max(2000)
    .optional(),
  
  images: z.array(z.string().url())
    .min(1, 'Minimal 1 gambar')
    .max(10, 'Maksimal 10 gambar'),
});

// Middleware
async function validateRequest(schema) {
  return async (req, res, next) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Data tidak valid',
          details: result.error.issues.map(issue => ({
            field: issue.path.join('.'),
            message: issue.message,
          })),
        }
      });
    }
    req.validatedBody = result.data;
    next();
  };
}
```

---

## 4. Error Handling

### [ENFORCE] Centralized Error Handling

```javascript
// Custom error classes
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true; // Bedakan dari bug
  }
}

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} tidak ditemukan`, 404, 'NOT_FOUND');
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Akses tidak diizinkan') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

class ConflictError extends AppError {
  constructor(message) {
    super(message, 409, 'CONFLICT');
  }
}

// Global error handler (Express)
function errorHandler(err, req, res, next) {
  // Log semua error (tapi filter yang operational)
  if (!err.isOperational) {
    logger.error('Unexpected error:', {
      error: err.message,
      stack: err.stack,
      requestId: req.id,
      url: req.url,
      method: req.method,
    });
  }

  const statusCode = err.statusCode || 500;
  
  // JANGAN expose stack trace ke production
  const response = {
    success: false,
    error: {
      code: err.code || 'INTERNAL_ERROR',
      message: err.isOperational 
        ? err.message 
        : 'Terjadi kesalahan pada server. Tim kami telah diberitahu.',
    },
    meta: { requestId: req.id },
  };

  if (process.env.NODE_ENV === 'development') {
    response.error.stack = err.stack;
  }

  res.status(statusCode).json(response);
}
```

---

## 5. Rate Limiting & Security

### [ENFORCE] Rate Limiting per Endpoint

```javascript
import rateLimit from 'express-rate-limit';

// Rate limit berbeda untuk endpoint berbeda
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 menit
  max: 5,                    // 5 attempt per window
  message: {
    success: false,
    error: {
      code: 'RATE_LIMIT_EXCEEDED',
      message: 'Terlalu banyak percobaan login. Coba lagi dalam 15 menit.',
    }
  },
  standardHeaders: true,
  legacyHeaders: false,
});

const apiLimiter = rateLimit({
  windowMs: 1 * 60 * 1000, // 1 menit
  max: 100,                 // 100 request per menit
});

// Apply
app.use('/api/auth/login', authLimiter);
app.use('/api/', apiLimiter);
```

---

### [ENFORCE] Security Headers

```javascript
import helmet from 'helmet';
import cors from 'cors';

// Helmet untuk security headers
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'", "https://fonts.googleapis.com"],
      fontSrc: ["'self'", "https://fonts.gstatic.com"],
      imgSrc: ["'self'", "data:", "https:"],
      scriptSrc: ["'self'"],
      connectSrc: ["'self'", "https://api.myapp.com"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
}));

// CORS yang ketat
const allowedOrigins = [
  'https://myapp.com',
  'https://www.myapp.com',
  ...(process.env.NODE_ENV === 'development' ? ['http://localhost:5173'] : []),
];

app.use(cors({
  origin: (origin, callback) => {
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('CORS policy violation'));
    }
  },
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

---

## 6. Logging & Observability

### [ENFORCE] Structured Logging

```javascript
// ❌ console.log yang tidak terstruktur
console.log('User logged in: ' + userId);
console.error('Database error: ' + err.message);

// ✅ Structured logging (gunakan pino atau winston)
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV !== 'production' 
    ? { target: 'pino-pretty' } 
    : undefined,
});

// Penggunaan
logger.info({ userId, email }, 'User logged in successfully');
logger.error({ err, userId, requestId }, 'Database query failed');
logger.warn({ ip, attempts }, 'Multiple failed login attempts detected');

// Request logger middleware
app.use((req, res, next) => {
  const start = Date.now();
  req.log = logger.child({ requestId: req.id });
  
  res.on('finish', () => {
    req.log.info({
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: Date.now() - start,
      userAgent: req.get('User-Agent'),
    });
  });
  
  next();
});
```

---

## 7. API Versioning

### [INSIGHT] Strategi Versioning yang Tidak Menyebabkan Breaking Changes

```javascript
// URL versioning (paling umum, paling jelas)
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// Aturan versioning:
// - MAJOR version (v1 → v2): Breaking changes
// - Pertahankan v1 minimal 6 bulan setelah v2 rilis
// - Kirim header deprecation warning di responses v1
//   Deprecation: true
//   Sunset: Sat, 31 Dec 2024 23:59:59 GMT
//   Link: <https://docs.myapp.com/migration/v2>; rel="successor-version"

// Breaking changes yang butuh major version:
// - Hapus field dari response
// - Ubah tipe data field
// - Ubah URL struktur
// - Ubah authentication method

// Non-breaking (tidak butuh major version):
// - Tambah field baru di response
// - Tambah endpoint baru
// - Tambah optional parameter
```

---

## 8. Async & Queue

### [INSIGHT] Jangan Buat User Menunggu Background Jobs

```javascript
// ❌ User menunggu email terkirim
app.post('/api/auth/register', async (req, res) => {
  const user = await createUser(req.body);
  await sendWelcomeEmail(user); // User menunggu ini selesai!
  res.json({ success: true, data: user });
});

// ✅ Response cepat, email di background
app.post('/api/auth/register', async (req, res) => {
  const user = await createUser(req.body);
  
  // Push ke queue, jangan await
  emailQueue.add('welcome-email', { userId: user.id });
  
  // Response langsung
  res.status(201).json({ success: true, data: user });
});

// Email worker (berjalan terpisah)
emailQueue.process('welcome-email', async (job) => {
  const user = await getUser(job.data.userId);
  await sendWelcomeEmail(user);
});
```

**Operasi yang HARUS di-queue:**
- Pengiriman email
- Pemrosesan gambar/video
- Notifikasi push
- Report generation
- Sync ke third-party service
- Bulk operations
