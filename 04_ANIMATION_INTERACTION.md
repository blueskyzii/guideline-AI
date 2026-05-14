# ✨ Guideline 04 — Animasi & Interaksi

> Animasi yang baik bukan *dekoratif* — ia adalah *komunikasi*. Setiap gerakan harus punya tujuan.

---

## 1. Filosofi Motion Design

### [ENFORCE] 12 Prinsip Animasi — Yang Berlaku untuk Web

Dari prinsip animasi Disney, yang paling relevan untuk web UI:

| Prinsip | Penerapan di Web |
|---------|-----------------|
| **Squash & Stretch** | Button yang sedikit "ditekan" saat click |
| **Anticipation** | Elemen "mengintip" sebelum slide masuk |
| **Follow Through** | Elemen overshoot sedikit sebelum settle |
| **Ease In/Out** | Selalu gunakan easing function, BUKAN linear |
| **Staging** | Satu animasi fokus per waktu, jangan semua bergerak |
| **Timing** | Durasi yang tepat — terlalu cepat = tidak terasa, terlalu lambat = mengganggu |

---

### [ENFORCE] Durasi Animasi yang Tepat

```
Action Feedback (click, toggle):     100-200ms  → Harus instan
UI Transitions (modal, dropdown):    200-350ms  → Terasa smooth
Page Transitions:                    300-500ms  → Punya bobot
Emphasis Animations (attention):     600-800ms  → Punya drama
Decorative Animations (loop):        1000ms+    → Boleh lambat
```

**Aturan dasar:** Semakin kecil elemen, semakin pendek durasinya.

---

### [INSIGHT] Easing yang Tidak Terasa Robotic

**Jangan pernah gunakan `linear` untuk animasi UI.** Linear terasa mekanis.

```css
/* ❌ Paling buruk */
transition: all 0.3s linear;

/* ✅ Easing yang "terasa natural" */

/* Untuk elemen yang MASUK ke layar (dari luar) */
--ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);       /* Cepat lalu lambat */
--ease-out-back: cubic-bezier(0.34, 1.56, 0.64, 1);   /* Overshoot sedikit */

/* Untuk elemen yang KELUAR layar */
--ease-in-expo: cubic-bezier(0.7, 0, 0.84, 0);        /* Lambat lalu cepat */

/* Untuk transisi dua arah */
--ease-in-out-quint: cubic-bezier(0.83, 0, 0.17, 1);  /* Smooth dua arah */

/* Untuk spring/bouncy effect */
--ease-spring: cubic-bezier(0.34, 1.56, 0.64, 1);
```

Referensi visual: **easings.net** — bookmark wajib.

---

## 2. Micro-interactions

### [ENFORCE] Setiap Interactive Element Butuh Feedback

```css
/* Button: terasa ditekan */
.btn {
  transition: transform 100ms var(--ease-out-expo),
              box-shadow 100ms var(--ease-out-expo);
  
  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 20px rgba(0,0,0,0.15);
  }
  
  &:active {
    transform: translateY(1px) scale(0.98);
    box-shadow: 0 2px 6px rgba(0,0,0,0.10);
    transition-duration: 50ms; /* Active lebih cepat dari hover */
  }
}

/* Checkbox custom yang satisfying */
.checkbox {
  --size: 20px;
  width: var(--size);
  height: var(--size);
  border: 2px solid #D4CFC8;
  border-radius: 4px;
  display: grid;
  place-content: center;
  transition: background 150ms, border-color 150ms;
  cursor: pointer;
  
  &::after {
    content: '';
    width: 10px;
    height: 6px;
    border-left: 2px solid white;
    border-bottom: 2px solid white;
    transform: rotate(-45deg) scale(0);
    transition: transform 150ms var(--ease-out-back);
  }
  
  &.is-checked {
    background: var(--color-accent);
    border-color: var(--color-accent);
    
    &::after { transform: rotate(-45deg) scale(1); }
  }
}
```

---

### [INSIGHT] Loading States yang Tidak Membosankan

**Jangan gunakan spinner polos.** Beri konteks:

```css
/* Skeleton shimmer — jauh lebih baik dari spinner */
.skeleton {
  background: linear-gradient(
    90deg,
    #f0ece8 0%,
    #e8e4e0 40%,
    #f0ece8 80%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: var(--radius-md);
}

@keyframes shimmer {
  0% { background-position: 200% center; }
  100% { background-position: -200% center; }
}

/* Skeleton yang sesuai konten (bukan generic box) */
.skeleton-text-lg { height: 28px; width: 60%; margin-bottom: 12px; }
.skeleton-text-sm { height: 16px; width: 80%; margin-bottom: 8px; }
.skeleton-avatar  { height: 48px; width: 48px; border-radius: 50%; }
.skeleton-image   { height: 200px; width: 100%; }
```

---

### [ENFORCE] Progress Indicators untuk Long Operations

```javascript
// ❌ Spinner tanpa informasi
showSpinner();
await longOperation();
hideSpinner();

// ✅ Progress dengan feedback
async function processWithFeedback(items) {
  const total = items.length;
  
  for (let i = 0; i < total; i++) {
    updateProgress({
      step: i + 1,
      total,
      message: `Memproses item ${i + 1} dari ${total}...`
    });
    await processItem(items[i]);
  }
  
  showSuccess('Semua item berhasil diproses!');
}
```

---

## 3. Page Transitions

### [INSIGHT] View Transitions API (Modern Browser)

Ini fitur native browser yang jarang dipakai tapi powerful:

```javascript
// Tanpa library, transisi halaman yang smooth
async function navigateTo(url) {
  if (!document.startViewTransition) {
    // Fallback untuk browser lama
    window.location.href = url;
    return;
  }

  await document.startViewTransition(async () => {
    // Fetch konten baru
    const response = await fetch(url);
    const html = await response.text();
    const parser = new DOMParser();
    const newDoc = parser.parseFromString(html, 'text/html');
    
    // Update konten
    document.getElementById('main-content').innerHTML = 
      newDoc.getElementById('main-content').innerHTML;
    
    document.title = newDoc.title;
    history.pushState(null, '', url);
  });
}
```

```css
/* CSS untuk mengontrol animasi */
::view-transition-old(root) {
  animation: 250ms var(--ease-in-expo) fade-out;
}

::view-transition-new(root) {
  animation: 350ms var(--ease-out-expo) slide-in;
}

@keyframes fade-out {
  to { opacity: 0; transform: scale(0.98); }
}

@keyframes slide-in {
  from { opacity: 0; transform: translateY(16px); }
}
```

---

## 4. Scroll-Driven Animations

### [INSIGHT] CSS Scroll-Driven Animations (No JavaScript!)

Fitur modern yang belum banyak digunakan:

```css
/* Progress bar yang mengikuti scroll — tanpa JS */
.reading-progress {
  position: fixed;
  top: 0;
  left: 0;
  height: 3px;
  background: var(--color-accent);
  width: 100%;
  transform-origin: left;
  animation: reading-progress linear both;
  animation-timeline: scroll(root block);
}

@keyframes reading-progress {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}

/* Elemen fade-in saat scroll — tanpa JS */
.fade-on-scroll {
  animation: fade-up linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 40%;
}

@keyframes fade-up {
  from { 
    opacity: 0; 
    transform: translateY(40px); 
  }
  to { 
    opacity: 1; 
    transform: translateY(0); 
  }
}

/* Parallax sederhana dengan CSS */
.parallax-element {
  animation: parallax linear both;
  animation-timeline: scroll(root);
}

@keyframes parallax {
  from { transform: translateY(0); }
  to   { transform: translateY(-100px); }
}
```

---

## 5. Hover Effects yang Tidak Generik

### [ENFORCE] Hindari Hover Effects Klise AI

**❌ Klise:**
- `transform: scale(1.05)` pada card
- `box-shadow` bertambah saat hover
- `opacity: 0.8` saat hover untuk teks

**✅ Hover yang berkarakter:**

```css
/* Magnetic button effect */
.btn-magnetic {
  transition: transform 200ms var(--ease-out-expo);
  /* Logic di JS untuk mengikuti mouse */
}

/* Underline animasi yang custom */
.nav-link {
  position: relative;
  text-decoration: none;

  &::after {
    content: '';
    position: absolute;
    bottom: -2px;
    left: 0;
    width: 100%;
    height: 1.5px;
    background: currentColor;
    transform-origin: right;
    transform: scaleX(0);
    transition: transform 250ms var(--ease-out-expo);
  }
  
  &:hover::after {
    transform-origin: left;
    transform: scaleX(1);
  }
}

/* Image zoom dalam container */
.image-container {
  overflow: hidden;
  border-radius: var(--radius-lg);
  
  img {
    width: 100%;
    transition: transform 600ms var(--ease-out-expo);
  }
  
  &:hover img {
    transform: scale(1.04);
  }
}

/* Teks fill dari kiri saat hover */
.text-fill-hover {
  background: linear-gradient(
    to right, 
    var(--color-accent) 50%, 
    var(--color-text-primary) 50%
  );
  background-size: 200% 100%;
  background-position: right;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  transition: background-position 400ms var(--ease-out-expo);
  
  &:hover { background-position: left; }
}
```

---

## 6. Animasi yang Accessible

### [ENFORCE] Hormati `prefers-reduced-motion`

```css
/* Disable semua animasi jika user minta */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* Atau lebih granular */
.animated-element {
  opacity: 0;
  transform: translateY(20px);
  transition: opacity 400ms, transform 400ms;
}

@media (prefers-reduced-motion: no-preference) {
  .animated-element.is-visible {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (prefers-reduced-motion: reduce) {
  .animated-element {
    opacity: 1;
    transform: none;
  }
}
```

---

## 7. CSS Animasi vs JavaScript Animasi

### [INSIGHT] Kapan Gunakan Masing-masing

| Situasi | Gunakan |
|---------|---------|
| Simple transitions (hover, active) | CSS `transition` |
| Keyframe animations sederhana | CSS `@keyframes` |
| Scroll-driven | CSS Scroll-driven / Intersection Observer |
| Complex sequences | Web Animations API |
| Physics-based (spring, bounce) | Library (Motion, GSAP) |
| SVG morphing | GSAP atau anime.js |
| Canvas animations | requestAnimationFrame |
| Three.js / WebGL | Selalu JavaScript |

---

### [ENFORCE] Animasi pada GPU Layer

```css
/* Pastikan animasi berjalan di GPU, bukan CPU */

/* ✅ GPU-accelerated properties */
.animated {
  transform: translateX(0);    /* Gunakan transform, bukan left/top */
  opacity: 1;                  /* Opacity juga GPU */
  will-change: transform;      /* Hint browser sebelum animasi */
}

/* ❌ CPU properties — hindari untuk animasi */
.animated-bad {
  left: 0;      /* Trigger layout recalculation */
  top: 0;       /* Trigger layout recalculation */
  width: 100px; /* Trigger layout recalculation */
  background: red; /* Trigger paint */
}

/* will-change: gunakan hanya saat diperlukan, hapus setelah selesai */
element.addEventListener('mouseenter', () => {
  element.style.willChange = 'transform';
});
element.addEventListener('animationend', () => {
  element.style.willChange = 'auto'; /* Reset setelah animasi */
});
```
