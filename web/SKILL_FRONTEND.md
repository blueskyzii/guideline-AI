# 🎨 GUIDELINE: Frontend Engineering & UI Design
> **Untuk AI Coding**: Baca seluruh dokumen ini sebelum menulis satu baris kode pun.
> Standar output: terasa seperti buatan **senior designer × senior frontend engineer**.

---

## 🔴 TAHAP 0 — DISCOVERY (WAJIB, TIDAK BOLEH DILEWATI)

**Sebelum membuat planning, wireframe, atau kode apapun — AI WAJIB menanyakan semua pertanyaan di bawah ini kepada user terlebih dahulu.**

Sampaikan dalam **satu blok sekaligus**, tunggu jawaban, baru lanjut ke planning.
Tujuan: menyamakan ekspektasi agar hasil akhir tidak meleset dari bayangan user.

---

### 📋 DAFTAR PERTANYAAN DISCOVERY FRONTEND

**A. Identitas & Tujuan Bisnis**
1. Apa nama produk / perusahaan / brand yang akan dibuat websitenya?
2. Apa tujuan utama website ini? *(landing page jualan, dashboard internal, company profile, portofolio, platform trading, marketplace, dll)*
3. Siapa target audiensnya? *(usia, profesi, level tech-savvy — contoh: trader profesional 25–40th, UMKM owner, mahasiswa)*
4. Satu hal yang paling ingin dirasakan user saat pertama kali membuka website? *(contoh: "wow keren", "aman dan terpercaya", "mudah dimengerti", "pengen langsung coba")*
5. Apa yang membedakan produk/bisnis ini dari kompetitor? *(USP — Unique Selling Point)*

**B. Konten & Halaman**
6. Halaman apa saja yang dibutuhkan? *(contoh: Home, About, Pricing, Blog, Dashboard, Login, Register, dll)*
7. Apakah konten sudah siap (copywriting, gambar, logo) atau AI yang membuatkan placeholder?
8. Fitur interaktif apa yang dibutuhkan? *(live ticker harga, tabel sortable, form multi-step, chart, kalkulator, real-time notifikasi, dll)*
9. Apakah ada halaman atau section yang WAJIB ada berdasarkan kebutuhan bisnis?
10. Apakah butuh sistem autentikasi di frontend (login, register, protected routes)?

**C. Visual & Estetika**
11. Apakah sudah ada brand guideline, logo, atau palet warna yang harus diikuti?
12. Sebutkan 2–3 website referensi yang Anda suka estetikanya *(boleh dari industri manapun)*
13. Dari pilihan berikut, mana yang paling mendekati feel yang diinginkan?
    - **A) Dark & Premium** — seperti Binance, Kraken, Linear (dark mode)
    - **B) Light & Clean** — seperti Stripe, Notion, Vercel
    - **C) Bold & Ekspresif** — seperti agency kreatif di Awwwards
    - **D) Warm & Human** — seperti Airbnb, Headspace, Lemon8
    - **E) Technical & Data-heavy** — seperti TradingView, Grafana, Datadog
14. Ada warna, gaya visual, atau elemen yang TIDAK disukai / tidak boleh dipakai?
15. Apakah butuh dark mode / light mode, atau satu mode saja?

**D. Technical & Platform**
16. Tech stack yang digunakan atau diinginkan? *(HTML/CSS/JS vanilla, React, Next.js, Vue, Svelte, dll)*
17. Perangkat prioritas: Mobile-first, Desktop-first, atau keduanya sama penting?
18. Apakah ada integrasi dengan backend/API yang perlu dipertimbangkan? *(REST API, WebSocket untuk live data, third-party auth, payment gateway, dll)*
19. Apakah akan di-deploy di platform tertentu? *(Vercel, Netlify, cPanel, VPS, dll)*

**E. Scope & Prioritas**
20. Apa yang HARUS selesai di versi pertama? *(must-have)*
21. Apa yang boleh ditunda ke versi berikutnya? *(nice-to-have)*
22. Apakah ada deadline atau milestone tertentu?
23. Apakah ada batasan budget / scope yang perlu diperhatikan dalam memilih library/tool?

---

### 📌 Format Project Brief — Wajib Dibuat Sebelum Coding

Setelah user menjawab, AI wajib menyusun dan **menampilkan brief ini** untuk dikonfirmasi:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 PROJECT BRIEF — [Nama Produk]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tujuan        : ...
Target audiens: ...
Feel yang dituju: ...
Tech stack    : ...
Halaman (v1)  : ...
Fitur interaktif: ...
Aesthetic direction: [pilih dari tabel section berikutnya]
Referensi visual: ...
Palet warna   : [custom / pakai preset guideline]
Mode          : [dark / light / keduanya]
Prioritas v1  : ...
Nice-to-have  : ...
Catatan khusus: ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
→ Apakah brief ini sudah sesuai ekspektasi Anda?
  Kalau ada yang perlu diubah, sampaikan sekarang sebelum saya mulai coding.
```

**AI WAJIB menunggu konfirmasi "oke / lanjut" dari user sebelum masuk ke tahap coding.**

---

## 🧠 TAHAP 1 — TENTUKAN DNA VISUAL

Setelah brief dikonfirmasi, pilih **satu** Aesthetic Direction dan commit penuh:

| Direction | Ciri khas | Cocok untuk |
|-----------|-----------|-------------|
| **Dark Luxury** | Near-black surface, amber/gold accent, tight tracking | Crypto, fintech, premium SaaS |
| **Editorial Stark** | High-contrast B&W, oversized type, asymmetric grid | Agency, portofolio, fashion brand |
| **Organic Warm** | Cream/sand bg, terracotta accent, serif body | Lifestyle, food, wellness, konsultan |
| **Industrial Tech** | Monospace font, grid overlay, neon accent on dark | Dev tools, cybersecurity, analytics |
| **Soft Kinetic** | Pastel palette, fluid shapes, playful micro-motion | Consumer app, edtech, marketplace |

> **Default jika user tidak spesifik**: Dark Luxury untuk crypto/fintech, Editorial Stark untuk agency/B2B.

---

## 🎨 VISUAL DESIGN SYSTEM

### [WAJIB] Palet Warna — Dilarang Default AI

**❌ DILARANG:**
- Blue-to-purple gradient (`#6366f1` → `#8b5cf6`)
- Tailwind `blue-500` / `indigo-500` sebagai accent utama
- Pure white `#ffffff` sebagai background halaman — terlalu flat
- Semua warna Tailwind default tanpa modifikasi karakter

**✅ CSS Variables per Aesthetic Direction:**

```css
/* ── Dark Luxury (Crypto / Fintech) ── */
:root {
  --bg-base:           #080C14;
  --bg-surface:        #0F1923;
  --bg-elevated:       #162032;
  --color-accent:      #E8A045;
  --color-accent-soft: rgba(232, 160, 69, 0.10);
  --color-positive:    #22C55E;
  --color-negative:    #EF4444;
  --text-primary:      #EDF0F7;
  --text-secondary:    #8A93A8;
  --text-muted:        #4A5568;
  --border:            rgba(255, 255, 255, 0.06);
  --border-accent:     rgba(232, 160, 69, 0.25);
  --shadow-card:       0 4px 24px rgba(0, 0, 0, 0.4);
}

/* ── Editorial Stark (Agency / Corporate) ── */
:root {
  --bg-base:           #F4F1EC;
  --bg-surface:        #FFFFFF;
  --bg-elevated:       #EDEAE3;
  --color-accent:      #C4380A;
  --color-accent-soft: rgba(196, 56, 10, 0.08);
  --text-primary:      #0D0D0D;
  --text-secondary:    #4A4A4A;
  --text-muted:        #888888;
  --border:            #E0DDD8;
  --border-accent:     rgba(196, 56, 10, 0.3);
  --shadow-card:       0 2px 12px rgba(0, 0, 0, 0.08);
}

/* ── Industrial Tech (Dashboard / Data) ── */
:root {
  --bg-base:           #060B14;
  --bg-surface:        #0D1526;
  --bg-elevated:       #142038;
  --color-accent:      #00E5C3;
  --color-accent2:     #FF4D6D;
  --color-accent-soft: rgba(0, 229, 195, 0.08);
  --text-primary:      #E2E8F0;
  --text-secondary:    #64748B;
  --text-muted:        #334155;
  --border:            rgba(0, 229, 195, 0.08);
  --border-accent:     rgba(0, 229, 195, 0.25);
  --shadow-card:       0 4px 32px rgba(0, 0, 0, 0.6);
}
```

### [WAJIB] Typography

**Font yang BOLEH digunakan:**

| Peran | Font | Karakter |
|-------|------|----------|
| Display/Heading | `Syne` | Geometric, tegas, modern |
| Display/Heading | `Cormorant Garamond` | Elegant, editorial |
| Display/Heading | `Bebas Neue` | Bold, impactful, industrial |
| Body | `DM Sans` | Clean, readable |
| Body | `Instrument Sans` | Contemporary |
| Data/Angka | `JetBrains Mono` | Angka trading, nilai, kode |
| Data/Angka | `IBM Plex Mono` | Technical, structured |

**DILARANG**: Inter, Roboto, Arial, system-ui — terlalu generic.

```css
/* Typographic scale */
:root {
  --text-xs:   clamp(0.65rem, 1vw,   0.75rem);
  --text-sm:   clamp(0.8rem,  1.2vw, 0.9rem);
  --text-base: clamp(0.95rem, 1.5vw, 1.05rem);
  --text-xl:   clamp(1.3rem,  2.5vw, 1.5rem);
  --text-3xl:  clamp(2rem,    4vw,   2.8rem);
  --text-hero: clamp(3rem,    7vw,   6rem);
}

.heading-display {
  font-size: var(--text-hero);
  font-weight: 700;
  letter-spacing: -0.04em;
  line-height: 1.0;
}

.label-eyebrow {
  font-size: var(--text-xs);
  font-weight: 600;
  letter-spacing: 0.15em;
  text-transform: uppercase;
  opacity: 0.55;
}
```

---

## 🏗️ STRUKTUR FILE — WAJIB RAPI

**Aturan utama: satu file = satu tanggung jawab.**

### HTML / CSS / JS Vanilla:
```
project-name/
├── index.html
├── pages/
│   ├── about.html
│   └── contact.html
├── assets/
│   ├── css/
│   │   ├── tokens.css        ← CSS variables (SEMUA token di sini)
│   │   ├── base.css          ← Reset + tipografi dasar
│   │   ├── layout.css        ← Grid, container, section spacing
│   │   ├── components.css    ← Button, card, badge, input, modal
│   │   ├── pages/
│   │   │   └── home.css
│   │   └── animations.css    ← Keyframes + motion classes
│   ├── js/
│   │   ├── main.js           ← Init semua module
│   │   ├── animations.js     ← IntersectionObserver, scroll logic
│   │   └── components/
│   │       ├── navbar.js
│   │       ├── ticker.js
│   │       └── modal.js
│   ├── images/               ← Format webp/avif, dioptimasi
│   └── fonts/                ← Self-hosted .woff2
└── README.md
```

### React / Next.js:
```
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── globals.css           ← Tokens + base styles
│   └── (routes)/
├── components/
│   ├── ui/                   ← Atom: Button, Input, Badge, Skeleton
│   │   └── index.ts          ← Barrel export
│   ├── sections/             ← Molekul: HeroSection, FeaturesSection
│   └── layout/               ← Navbar, Footer, Sidebar
├── hooks/
│   └── useScrollPosition.ts
├── lib/
│   ├── utils.ts
│   └── constants.ts          ← Semua magic string/number
├── styles/
│   └── tokens.css
└── public/
    ├── fonts/
    └── images/
```

**Naming Convention (wajib konsisten):**

| Tipe | Format | Contoh |
|------|--------|--------|
| React Component | `PascalCase.tsx` | `HeroSection.tsx` |
| Hook | `camelCase.ts` | `useScrollPosition.ts` |
| Utility | `camelCase.ts` | `formatCurrency.ts` |
| CSS Module | `PascalCase.module.css` | `HeroSection.module.css` |
| Konstanta | `SCREAMING_SNAKE` | `MAX_ITEMS_PER_PAGE` |

---

## ✨ LAYOUT & KOMPOSISI

### [WAJIB] Anti-Template Rules:
1. **Dilarang hero 50/50** (teks kiri + gambar kanan) — terlalu generik
2. **Gunakan overlap** antar section — terkesan premium
3. **Asymmetric grid** — tidak harus 3 kolom rata
4. **White space agresif** — min `padding-block: 6rem` di desktop
5. **Vary visual rhythm** — alternasi section padat dan lega

### [WAJIB] Struktur Hero Minimum:
```html
<section class="hero">
  <div class="hero__bg"><!-- mesh gradient / particle / video muted --></div>
  <div class="hero__content">
    <span class="label-eyebrow"><!-- "Dipercaya 10.000+ trader" --></span>
    <h1 class="heading-display"><!-- Max 8 kata. Outcome-focused. --></h1>
    <p class="hero__tagline"><!-- Max 20 kata. --></p>
    <div class="hero__cta">
      <a class="btn btn--primary">Primary CTA</a>
      <a class="btn btn--ghost">Secondary CTA</a>
    </div>
  </div>
  <div class="hero__trust"><!-- Logo partner / statistik / security badge --></div>
</section>
```

---

## 🎬 MOTION & ANIMASI

```css
:root {
  --ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in-expo:  cubic-bezier(0.7, 0, 0.84, 0);
  --ease-spring:   cubic-bezier(0.34, 1.56, 0.64, 1);
  --ease-smooth:   cubic-bezier(0.25, 0.46, 0.45, 0.94);
}

@keyframes fadeUpIn {
  from { opacity: 0; transform: translateY(28px); }
  to   { opacity: 1; transform: translateY(0); }
}

[data-reveal] { opacity: 0; }
[data-reveal].is-visible {
  animation: fadeUpIn 0.75s var(--ease-out-expo) both;
}
[data-reveal-delay="1"] { animation-delay: 0.1s; }
[data-reveal-delay="2"] { animation-delay: 0.2s; }
[data-reveal-delay="3"] { animation-delay: 0.35s; }

/* Button states */
.btn {
  transition: transform 0.15s var(--ease-smooth),
              box-shadow 0.15s var(--ease-smooth),
              background-color 0.2s ease;
}
.btn:hover        { transform: translateY(-2px); box-shadow: 0 8px 20px rgba(0,0,0,0.25); }
.btn:active       { transform: scale(0.97); }
.btn:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 3px; }
.btn:disabled     { opacity: 0.45; pointer-events: none; }

/* Accessibility */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 📋 COPYWRITING — Outcome, Bukan Fitur

**Rumus headline:** `[Hasil yang didapat] + [Seberapa mudah] + [Tanpa pain point]`

- ❌ "Platform trading dengan fitur canggih dan teknologi mutakhir"
- ✅ "Beli crypto dalam 30 detik. Tanpa biaya tersembunyi."

**Kata DILARANG:** Revolutionary, Seamless, Empower, Leverage, Unlock, Next-level, World-class, Cutting-edge, Innovative, Ecosystem — ganti dengan angka konkret dan aksi nyata.

---

## ⚡ PERFORMANCE FRONTEND

```html
<!-- LCP image -->
<img src="hero.webp" fetchpriority="high" loading="eager"
     width="1200" height="630" alt="..." decoding="sync">

<!-- Below-fold -->
<img src="feature.webp" loading="lazy" decoding="async"
     width="600" height="400" alt="...">
```

**Target Core Web Vitals:**
- LCP < 2.5 detik
- CLS < 0.1 — selalu set `width` & `height` pada semua gambar
- FID < 100ms — tidak ada blocking script di `<head>`

---

## ✅ CHECKLIST FINAL

- [ ] Semua 23 pertanyaan discovery sudah ditanyakan
- [ ] Project Brief sudah dikonfirmasi user sebelum coding dimulai
- [ ] Aesthetic direction dipilih dan diterapkan konsisten
- [ ] Warna pakai CSS variables, bukan hardcoded hex
- [ ] Font bukan Inter/Roboto/Arial
- [ ] Struktur file rapi sesuai template di atas
- [ ] Naming convention konsisten
- [ ] README.md ada: cara install, cara run, struktur folder
- [ ] Zero Lorem Ipsum
- [ ] Zero inline style
- [ ] Semua gambar: `alt` + `width` + `height`
- [ ] Semantic HTML5
- [ ] Mobile-first (`min-width` breakpoint)
- [ ] Semua button: hover + focus-visible + active + disabled state
- [ ] LCP image: `fetchpriority="high"`
- [ ] Font: `font-display: swap`
