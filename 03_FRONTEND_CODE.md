# 💻 Guideline 03 — Frontend Code

> Kode frontend yang baik bukan hanya *bekerja*, tapi juga *maintainable*, *accessible*, dan *performant*.

---

## 1. HTML yang Semantik dan Bermakna

### [ENFORCE] HTML Semantik adalah SEO dan Accessibility Sekaligus

**❌ HTML AI yang buruk:**
```html
<div class="hero">
  <div class="hero-content">
    <div class="title">Welcome to Our Platform</div>
    <div class="subtitle">We help you do things better</div>
    <div class="button" onclick="...">Get Started</div>
  </div>
</div>
```

**✅ HTML semantik yang benar:**
```html
<section class="hero" aria-labelledby="hero-heading">
  <div class="hero__container">
    <h1 id="hero-heading" class="hero__title">
      Tutup pembukuan lebih cepat — tanpa akuntan
    </h1>
    <p class="hero__lead">
      Software kasir yang otomatis rekap harian, mingguan, dan bulanan. 
      Cocok untuk warung hingga toko retail.
    </p>
    <div class="hero__actions">
      <a href="/daftar" class="btn-primary" role="button">
        Coba Gratis 14 Hari
      </a>
      <a href="/demo" class="btn-secondary">
        Lihat Demo
      </a>
    </div>
  </div>
</section>
```

---

### [ENFORCE] Outline Dokumen yang Benar

```html
<!-- Satu h1 per halaman — untuk SEO dan screen reader -->
<h1>Judul Halaman Utama</h1>

  <!-- h2 untuk section utama -->
  <h2>Section: Fitur Unggulan</h2>

    <!-- h3 untuk sub-section -->
    <h3>Fitur 1: Rekap Otomatis</h3>

    <h3>Fitur 2: Multi-Kasir</h3>

  <h2>Section: Harga</h2>

  <h2>Section: Testimoni</h2>
```

**Jangan skip level!** Jangan `h1` langsung ke `h3`.

---

### [INSIGHT] Landmark Roles untuk Navigation

```html
<body>
  <header role="banner">
    <nav aria-label="Navigasi utama">...</nav>
  </header>

  <main role="main" id="main-content">
    <!-- Skip to main content link (untuk keyboard users) -->
    <a href="#main-content" class="skip-link">Skip to main content</a>
    
    <article>...</article>
    <aside aria-label="Artikel terkait">...</aside>
  </main>

  <footer role="contentinfo">
    <nav aria-label="Navigasi footer">...</nav>
  </footer>
</body>
```

---

## 2. CSS Architecture

### [ENFORCE] BEM Naming Convention

```css
/* Block */
.card {}

/* Element */
.card__title {}
.card__image {}
.card__body {}
.card__footer {}

/* Modifier */
.card--featured {}
.card--compact {}
.card--dark {}

/* State (gunakan is- atau has- prefix) */
.card.is-loading {}
.card.is-selected {}
.nav__item.is-active {}
```

---

### [ENFORCE] CSS Custom Properties untuk Semua Nilai yang Berulang

**❌ CSS yang tidak maintainable:**
```css
.btn { background: #C4622D; border-radius: 8px; }
.badge { background: #C4622D; }
.link:hover { color: #C4622D; }
/* Jika warna berubah, harus ganti di 3+ tempat */
```

**✅ CSS yang maintainable:**
```css
:root { --color-accent: #C4622D; }
.btn { background: var(--color-accent); }
.badge { background: var(--color-accent); }
.link:hover { color: var(--color-accent); }
/* Cukup ubah satu nilai di :root */
```

---

### [INSIGHT] CSS Logical Properties untuk Internasionalisasi

Daripada `margin-left/right`, gunakan logical properties:

```css
/* ❌ Directionality-specific */
.text { margin-left: 1rem; padding-right: 2rem; }

/* ✅ Logical (otomatis support RTL bahasa Arab/Ibrani) */
.text { margin-inline-start: 1rem; padding-inline-end: 2rem; }

/* Shortcuts yang berguna */
.container {
  margin-inline: auto;     /* margin-left & margin-right */
  padding-block: 4rem;     /* padding-top & padding-bottom */
  padding-inline: 1.5rem;  /* padding-left & padding-right */
}
```

---

### [ENFORCE] Layer CSS yang Jelas

```css
/* Urutan import yang benar */
@layer reset, base, tokens, layout, components, utilities, overrides;

@layer reset {
  *, *::before, *::after { box-sizing: border-box; }
  /* Normalize styles */
}

@layer base {
  /* HTML element defaults */
  body { font-family: var(--font-body); }
  h1, h2, h3 { line-height: var(--leading-tight); }
}

@layer components {
  /* Reusable components */
  .btn-primary { ... }
  .card { ... }
}

@layer utilities {
  /* Single-purpose utility classes */
  .text-center { text-align: center; }
  .sr-only { /* screen reader only */ }
}
```

---

## 3. JavaScript yang Tidak Berlebihan

### [ENFORCE] Vanilla First, Library Second

Sebelum install library, tanyakan: *"Apakah ini bisa dilakukan dengan native API?"*

```javascript
// ❌ Install library hanya untuk ini
// npm install lodash
import { debounce } from 'lodash';

// ✅ Vanilla debounce (5 baris)
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// ❌ jQuery untuk AJAX
$.get('/api/data', callback);

// ✅ Native fetch
const data = await fetch('/api/data').then(r => r.json());

// ❌ Animate.css untuk animasi sederhana
// ✅ Web Animations API
element.animate(
  [{ opacity: 0, transform: 'translateY(20px)' }, 
   { opacity: 1, transform: 'translateY(0)' }],
  { duration: 300, easing: 'cubic-bezier(0.16, 1, 0.3, 1)', fill: 'forwards' }
);
```

---

### [ENFORCE] Event Delegation, Bukan Per-Element Listener

```javascript
// ❌ Buruk — buat listener untuk setiap item
document.querySelectorAll('.menu-item').forEach(item => {
  item.addEventListener('click', handleClick);
});
// Problem: memory leak saat item ditambah/hapus dari DOM

// ✅ Baik — satu listener di parent
document.querySelector('.menu').addEventListener('click', (e) => {
  const item = e.target.closest('.menu-item');
  if (!item) return;
  handleClick(item, e);
});
```

---

### [INSIGHT] Intersection Observer untuk Animasi Scroll

Jangan gunakan `scroll` event untuk animasi — itu sangat tidak performant:

```javascript
// ❌ Buruk — berjalan setiap pixel scroll
window.addEventListener('scroll', () => {
  elements.forEach(el => {
    if (el.getBoundingClientRect().top < window.innerHeight) {
      el.classList.add('visible');
    }
  });
});

// ✅ Baik — Intersection Observer (off main thread)
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-visible');
        observer.unobserve(entry.target); // Stop observing setelah visible
      }
    });
  },
  { threshold: 0.15, rootMargin: '0px 0px -50px 0px' }
);

document.querySelectorAll('[data-animate]').forEach(el => observer.observe(el));
```

```css
/* CSS pair untuk JS di atas */
[data-animate] {
  opacity: 0;
  transform: translateY(24px);
  transition: opacity 0.5s var(--ease-out), 
              transform 0.5s var(--ease-out);
}

[data-animate].is-visible {
  opacity: 1;
  transform: translateY(0);
}

/* Staggered delay untuk grup elemen */
[data-animate]:nth-child(2) { transition-delay: 0.1s; }
[data-animate]:nth-child(3) { transition-delay: 0.2s; }
```

---

### [ENFORCE] State Management yang Tidak Berlebihan

Untuk website statis/semi-dinamis, TIDAK perlu Redux/Zustand/Pinia.

```javascript
// ✅ Simple state dengan object + event
const AppState = {
  _state: {
    theme: 'light',
    cartCount: 0,
    mobileMenuOpen: false,
  },

  get(key) { return this._state[key]; },

  set(key, value) {
    this._state[key] = value;
    // Trigger custom event untuk reactivity
    document.dispatchEvent(new CustomEvent('state:change', {
      detail: { key, value }
    }));
  },
};

// Consumer
document.addEventListener('state:change', ({ detail }) => {
  if (detail.key === 'theme') {
    document.documentElement.dataset.theme = detail.value;
  }
});
```

---

## 4. Accessibility (a11y)

### [ENFORCE] Keyboard Navigation Harus Bekerja

```javascript
// Modal/Dialog keyboard trap
function trapFocus(modal) {
  const focusable = modal.querySelectorAll(
    'a[href], button:not([disabled]), input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const first = focusable[0];
  const last = focusable[focusable.length - 1];

  modal.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;
    if (e.shiftKey) {
      if (document.activeElement === first) {
        last.focus(); e.preventDefault();
      }
    } else {
      if (document.activeElement === last) {
        first.focus(); e.preventDefault();
      }
    }
  });
}

// Close dengan Escape
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') closeModal();
});
```

---

### [ENFORCE] ARIA yang Minimal tapi Benar

```html
<!-- Toggle button -->
<button 
  aria-expanded="false" 
  aria-controls="menu-dropdown"
  id="menu-toggle">
  Menu
</button>
<div id="menu-dropdown" role="menu" aria-labelledby="menu-toggle" hidden>
  ...
</div>

<!-- Loading state -->
<button aria-busy="true" aria-describedby="loading-msg">
  Menyimpan...
</button>
<span id="loading-msg" class="sr-only" aria-live="polite">
  Data sedang disimpan, mohon tunggu.
</span>

<!-- Icon-only button -->
<button aria-label="Tutup dialog" class="btn-icon">
  <svg aria-hidden="true">...</svg>
</button>
```

---

### [INSIGHT] Focus Visible Jangan Dihilangkan

```css
/* ❌ JANGAN PERNAH lakukan ini */
* { outline: none; }
*:focus { outline: none; }

/* ✅ Sembunyikan hanya untuk mouse users, pertahankan untuk keyboard */
:focus:not(:focus-visible) { outline: none; }
:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 3px;
  border-radius: 2px;
}
```

---

## 5. Performance Code Patterns

### [ENFORCE] Image Optimization di HTML

```html
<!-- Modern image dengan fallback -->
<picture>
  <source srcset="hero.avif" type="image/avif">
  <source srcset="hero.webp" type="image/webp">
  <img 
    src="hero.jpg" 
    alt="Dashboard aplikasi kasir menampilkan rekap harian"
    width="1200" 
    height="800"
    loading="eager"    <!-- LCP image: eager -->
    fetchpriority="high"
    decoding="async"
  >
</picture>

<!-- Below-fold images: lazy load -->
<img 
  src="feature.webp" 
  alt="..."
  width="600" 
  height="400"
  loading="lazy"
  decoding="async"
>
```

---

### [ENFORCE] Resource Hints di `<head>`

```html
<head>
  <!-- Preconnect untuk third-party origins -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  
  <!-- Preload critical resources -->
  <link rel="preload" href="/fonts/sora-variable.woff2" as="font" type="font/woff2" crossorigin>
  <link rel="preload" href="/images/hero.avif" as="image">
  
  <!-- Prefetch halaman yang kemungkinan dikunjungi berikutnya -->
  <link rel="prefetch" href="/harga">
  <link rel="prefetch" href="/daftar">
</head>
```

---

### [INSIGHT] `content-visibility` untuk Render Performance

```css
/* Untuk section di bawah fold — skip rendering sampai dekat viewport */
.section-below-fold {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* Perkiraan tinggi section */
}

/* Ini bisa improve initial render time 30-60% untuk halaman panjang */
```

---

## 6. Code Quality

### [ENFORCE] Komentar yang Bermakna, Bukan Obvious

```javascript
// ❌ Obvious comment
// Get user data
const user = await getUser(id);

// ✅ Komentar yang menjelaskan KENAPA, bukan APA
// Fetch user data early to avoid waterfall with profile page load
// (profile page needs user data before rendering the sidebar)
const user = await getUser(id);

// ❌ Commented-out code
// const oldFunction = () => { ... }

// ✅ Hapus code lama. Git ada untuk backup.
```

---

### [ENFORCE] Error Handling yang User-Friendly

```javascript
async function submitForm(data) {
  try {
    const response = await fetch('/api/register', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });

    if (!response.ok) {
      // Jangan expose technical error ke user
      const error = await response.json().catch(() => ({}));
      throw new UserFacingError(
        error.message || 'Terjadi kesalahan. Coba lagi dalam beberapa saat.',
        response.status
      );
    }

    return await response.json();

  } catch (err) {
    if (err instanceof UserFacingError) throw err;
    
    // Network error / unexpected
    console.error('[submitForm]', err); // Log untuk developer
    throw new UserFacingError(
      'Koneksi bermasalah. Pastikan internet kamu aktif dan coba lagi.'
    );
  }
}
```
