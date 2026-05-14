# ⚡ Guideline 07 — Performance

> Performance bukan fitur opsional. Setiap 100ms tambahan load time = konversi turun ~1%.

---

## 1. Core Web Vitals — Target yang Harus Dicapai

### [ENFORCE] Target Metric Minimum

| Metric | Baik | Perlu Perbaikan | Buruk |
|--------|------|-----------------|-------|
| **LCP** (Largest Contentful Paint) | ≤ 2.5s | 2.5s – 4s | > 4s |
| **CLS** (Cumulative Layout Shift) | ≤ 0.1 | 0.1 – 0.25 | > 0.25 |
| **INP** (Interaction to Next Paint) | ≤ 200ms | 200ms – 500ms | > 500ms |
| **FCP** (First Contentful Paint) | ≤ 1.8s | 1.8s – 3s | > 3s |
| **TTFB** (Time to First Byte) | ≤ 800ms | 800ms – 1.8s | > 1.8s |

**Ukur dengan:** Lighthouse, PageSpeed Insights, WebPageTest, SpeedLoft

---

## 2. Optimasi Gambar

### [ENFORCE] Format Gambar Modern

```html
<!-- Selalu gunakan picture dengan multiple format -->
<picture>
  <!-- AVIF: 50-60% lebih kecil dari JPEG, support modern browser -->
  <source 
    srcset="image-400.avif 400w, image-800.avif 800w, image-1200.avif 1200w"
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 600px"
    type="image/avif"
  >
  <!-- WebP: 30% lebih kecil dari JPEG, support luas -->
  <source 
    srcset="image-400.webp 400w, image-800.webp 800w, image-1200.webp 1200w"
    sizes="(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 600px"
    type="image/webp"
  >
  <!-- JPEG fallback -->
  <img 
    src="image-800.jpg" 
    alt="Deskripsi yang bermakna"
    width="800" 
    height="533"  <!-- WAJIB untuk hindari CLS -->
    loading="lazy"
    decoding="async"
  >
</picture>
```

---

### [INSIGHT] Dimensi Gambar yang Sering Dilupakan

**CLS penyebab utama:** Gambar tanpa dimensi yang dideklarasikan.

```html
<!-- ❌ CLS: Browser tidak tahu ukuran sebelum gambar load -->
<img src="product.jpg" alt="...">

<!-- ✅ Browser reservasi ruang sebelum gambar load -->
<img src="product.jpg" alt="..." width="400" height="300">

<!-- ✅ Untuk gambar yang dimensinya tidak diketahui, gunakan aspect-ratio CSS -->
<div class="image-wrapper">
  <img src="product.jpg" alt="..." style="width: 100%; height: auto;">
</div>
```

```css
.image-wrapper {
  aspect-ratio: 4 / 3;  /* Reservasi ruang sebelum gambar load */
  overflow: hidden;
}

.image-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

---

## 3. Font Performance

### [ENFORCE] Font Loading yang Optimal

```html
<!-- 1. Preconnect ke font service -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>

<!-- 2. Preload font paling penting (body font) -->
<link 
  rel="preload" 
  href="/fonts/sora-variable.woff2" 
  as="font" 
  type="font/woff2" 
  crossorigin
>
```

```css
/* 3. Self-host font (lebih cepat dari Google Fonts) */
@font-face {
  font-family: 'Sora';
  src: url('/fonts/sora-variable.woff2') format('woff2');
  font-weight: 100 900;     /* Variable font — satu file untuk semua weight */
  font-style: normal;
  font-display: swap;        /* Tampilkan fallback dulu, swap saat font load */
}

/* 4. Font fallback yang tidak menyebabkan layout shift */
body {
  font-family: 'Sora', 'Segoe UI', system-ui, -apple-system, sans-serif;
  
  /* Size-adjust untuk minimalkan layout shift saat swap */
  /* Hitung dengan: https://screenspan.net/fallback */
}

/* 5. f-mods untuk minimize FOUT (Flash of Unstyled Text) */
@font-face {
  font-family: 'Sora-fallback';
  src: local('Segoe UI');
  size-adjust: 94%;          /* Sesuaikan agar mendekati Sora */
  ascent-override: 95%;
  descent-override: 25%;
}

body {
  font-family: 'Sora', 'Sora-fallback', sans-serif;
}
```

---

## 4. JavaScript Performance

### [ENFORCE] Code Splitting dan Lazy Loading

```javascript
// ❌ Import semua library di awal
import Chart from 'chart.js';
import { DataTable } from 'fancy-table';
import DatePicker from 'date-picker';
// Semua ini masuk ke bundle utama, padahal tidak dipakai di semua halaman

// ✅ Dynamic import — load saat dibutuhkan
async function showChart() {
  // Chart.js hanya di-download saat tombol chart diklik
  const { Chart } = await import('chart.js');
  renderChart(new Chart(...));
}

// ✅ Lazy load komponen (React)
const HeavyModal = lazy(() => import('./HeavyModal'));

function App() {
  return (
    <Suspense fallback={<ModalSkeleton />}>
      {showModal && <HeavyModal />}
    </Suspense>
  );
}
```

---

### [INSIGHT] Main Thread yang Tidak Terblokir

```javascript
// ❌ Blokir main thread — UI freeze saat processing
function processLargeArray(items) {
  return items.map(item => heavyComputation(item));
  // Jika items.length = 100.000, UI freeze beberapa detik
}

// ✅ Chunked processing dengan scheduler
async function processLargeArrayChunked(items) {
  const CHUNK_SIZE = 100;
  const results = [];
  
  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);
    results.push(...chunk.map(item => heavyComputation(item)));
    
    // Berikan kontrol ke browser setiap chunk
    // Menggunakan Scheduler API (modern) atau setTimeout fallback
    if ('scheduler' in window && 'yield' in scheduler) {
      await scheduler.yield(); // Modern - browser bisa handle user input
    } else {
      await new Promise(resolve => setTimeout(resolve, 0));
    }
  }
  
  return results;
}

// ✅ Web Worker untuk komputasi berat
const worker = new Worker('/workers/computation.js');
worker.postMessage({ items });
worker.onmessage = ({ data }) => {
  console.log('Hasil:', data.results);
};
```

---

### [ENFORCE] Bundle Size Monitoring

```javascript
// package.json scripts
{
  "scripts": {
    "build": "vite build",
    "analyze": "vite-bundle-visualizer",  // Visualisasi bundle
    "size": "bundlesize"                   // Cek size limit
  }
}

// bundlesize config — gagal build jika melebihi batas
{
  "bundlesize": [
    { "path": "./dist/assets/index-*.js", "maxSize": "100 kB" },
    { "path": "./dist/assets/vendor-*.js", "maxSize": "200 kB" }
  ]
}
```

**Target bundle size (gzipped):**
- HTML: < 30KB
- CSS: < 50KB
- JavaScript (core): < 100KB
- JavaScript (vendor): < 200KB

---

## 5. Caching Strategy

### [ENFORCE] HTTP Caching Headers yang Tepat

```nginx
# Nginx config
# Static assets yang di-hash (bisa cache lama)
location ~* \.(js|css|woff2)$ {
  expires 1y;
  add_header Cache-Control "public, immutable";
  # "immutable" = browser tidak perlu revalidate selama cache valid
}

# Gambar (bisa cache agak lama)
location ~* \.(jpg|jpeg|png|gif|webp|avif|ico|svg)$ {
  expires 30d;
  add_header Cache-Control "public, stale-while-revalidate=86400";
}

# HTML — jangan cache lama, gunakan ETag
location ~* \.html$ {
  expires 0;
  add_header Cache-Control "no-cache";  # Selalu revalidate
  # "no-cache" ≠ "tidak di-cache" — browser cek dulu dengan ETag
}

# API responses
location /api/ {
  add_header Cache-Control "no-store";  # Benar-benar tidak di-cache
}
```

---

### [INSIGHT] Stale-While-Revalidate Pattern

```javascript
// Service Worker: konten tampil cepat dari cache, update di background
const CACHE_NAME = 'v1';

self.addEventListener('fetch', (event) => {
  // Hanya cache GET requests
  if (event.request.method !== 'GET') return;
  
  event.respondWith(
    caches.match(event.request).then(async (cached) => {
      const fetchPromise = fetch(event.request).then((response) => {
        // Update cache di background
        const clonedResponse = response.clone();
        caches.open(CACHE_NAME).then(cache => {
          cache.put(event.request, clonedResponse);
        });
        return response;
      });
      
      // Return cached version (cepat) atau tunggu fetch (jika tidak ada cache)
      return cached || fetchPromise;
    })
  );
});
```

---

## 6. Critical CSS & Render Blocking

### [ENFORCE] Inline Critical CSS

```html
<head>
  <!-- CSS kritis inline — tidak ada render-blocking request -->
  <style>
    /* Hanya CSS yang dibutuhkan untuk above-the-fold */
    :root { --color-bg: #fafaf8; --color-text: #111111; }
    body { margin: 0; font-family: system-ui; background: var(--color-bg); }
    .nav { height: 64px; background: white; }
    .hero { padding: 80px 0; }
    .hero__title { font-size: clamp(2.5rem, 6vw, 5rem); }
  </style>
  
  <!-- Non-critical CSS di-load async -->
  <link 
    rel="preload" 
    href="/styles/main.css" 
    as="style" 
    onload="this.onload=null;this.rel='stylesheet'"
  >
  <noscript>
    <link rel="stylesheet" href="/styles/main.css">
  </noscript>
</head>
```

---

## 7. Server-Side Performance

### [ENFORCE] Database Connection Pooling

```javascript
// ❌ Buat koneksi baru setiap query
async function query(sql) {
  const connection = await createConnection(config); // Mahal!
  const result = await connection.query(sql);
  await connection.end();
  return result;
}

// ✅ Connection pool
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  database: process.env.DB_NAME,
  max: 20,                    // Maksimal 20 koneksi
  min: 5,                     // Minimal 5 koneksi siap
  idleTimeoutMillis: 30000,   // Tutup koneksi idle setelah 30 detik
  connectionTimeoutMillis: 2000, // Timeout jika tidak dapat koneksi
});

// Satu pool dipakai semua request
const result = await pool.query(sql, params);
```

---

### [INSIGHT] Caching di Application Layer

```javascript
import NodeCache from 'node-cache';

// Cache yang simple dan efektif
const cache = new NodeCache({
  stdTTL: 300,        // 5 menit default TTL
  checkperiod: 60,    // Cek expired setiap 60 detik
  useClones: false,   // Performa lebih baik
});

// Cache wrapper
async function withCache(key, ttl, fetcher) {
  const cached = cache.get(key);
  if (cached !== undefined) {
    return cached;
  }
  
  const data = await fetcher();
  cache.set(key, data, ttl);
  return data;
}

// Penggunaan
async function getCategories() {
  return withCache(
    'product:categories',
    3600, // 1 jam — categories jarang berubah
    () => db.query('SELECT * FROM categories WHERE is_active = true')
  );
}

async function getUserProfile(userId) {
  return withCache(
    `user:profile:${userId}`,
    300, // 5 menit
    () => db.query('SELECT * FROM users WHERE id = $1', [userId])
  );
}

// Invalidate cache saat data berubah
async function updateCategory(id, data) {
  await db.query('UPDATE categories SET ... WHERE id = $1', [id]);
  cache.del('product:categories'); // Hapus cache yang stale
}
```

---

## 8. Monitoring & Alerting

### [ENFORCE] Metric yang Harus Di-monitor

```
Server Metrics:
✓ CPU usage (alert jika > 80% selama 5 menit)
✓ Memory usage (alert jika > 85%)
✓ Disk I/O (alert jika > 90%)
✓ Network in/out

Application Metrics:
✓ Request rate (requests/second)
✓ Error rate (alert jika > 1% dari total request)
✓ Response time P50, P95, P99
✓ Active connections

Database Metrics:
✓ Query time P95 (alert jika > 500ms)
✓ Connection pool usage (alert jika > 80%)
✓ Slow queries (log query > 1000ms)
✓ Replication lag (jika ada replica)

Business Metrics:
✓ Conversion rate (alert jika drop > 20%)
✓ Sign-up rate
✓ Error per user journey
```
