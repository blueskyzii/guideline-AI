# 🎨 Guideline 02 — UI Design System

> Sistem desain adalah *bahasa visual* website kamu. Tanpa sistem, hasilnya inkonsisten dan terasa buatan.

---

## 1. Filosofi Design System

### [ENFORCE] Design Tokens Dulu, Komponen Kemudian
Jangan langsung membuat komponen. Definisikan dulu semua token:

```css
/* ✅ Benar: Token system */
:root {
  /* Color primitives */
  --color-sand-100: #faf7f2;
  --color-sand-200: #f0ead9;
  --color-ink-900: #1a1410;
  --color-ink-600: #4a3f35;
  --color-copper-500: #c4622d;
  --color-copper-400: #d4784a;

  /* Semantic tokens */
  --color-background: var(--color-sand-100);
  --color-surface: #ffffff;
  --color-text-primary: var(--color-ink-900);
  --color-text-muted: var(--color-ink-600);
  --color-accent: var(--color-copper-500);
  --color-accent-hover: var(--color-copper-400);

  /* Spacing scale (4px base) */
  --space-1: 0.25rem;   /* 4px */
  --space-2: 0.5rem;    /* 8px */
  --space-3: 0.75rem;   /* 12px */
  --space-4: 1rem;      /* 16px */
  --space-6: 1.5rem;    /* 24px */
  --space-8: 2rem;      /* 32px */
  --space-12: 3rem;     /* 48px */
  --space-16: 4rem;     /* 64px */
  --space-24: 6rem;     /* 96px */
  --space-32: 8rem;     /* 128px */

  /* Type scale */
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
  --text-5xl: 3rem;
  --text-6xl: 3.75rem;

  /* Border radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 16px;
  --radius-xl: 24px;
  --radius-full: 9999px;

  /* Shadow */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04);
  --shadow-md: 0 4px 16px rgba(0,0,0,0.08), 0 2px 6px rgba(0,0,0,0.04);
  --shadow-lg: 0 16px 40px rgba(0,0,0,0.10), 0 4px 12px rgba(0,0,0,0.06);
  --shadow-xl: 0 32px 64px rgba(0,0,0,0.12), 0 8px 24px rgba(0,0,0,0.06);

  /* Transition */
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
  --duration-fast: 150ms;
  --duration-base: 250ms;
  --duration-slow: 400ms;
}
```

---

## 2. Warna

### [ENFORCE] Hindari Palet Warna AI yang Klise

**Palet yang HARUS dihindari (terlalu AI):**
- Biru `#3B82F6` + Ungu `#8B5CF6` (Tailwind blue-500 + violet-500)
- Hijau `#10B981` sebagai accent utama
- Dark mode dengan `#0F172A` sebagai background
- Gradient ungu-ke-pink (`#8B5CF6` → `#EC4899`)

**Pendekatan warna yang lebih berkarakter:**

#### Option A: Earth Tones (Hangat & Trustworthy)
```
Background: #FAF7F2 (krem hangat)
Surface:    #FFFFFF
Text:       #1A1410 (hitam cokelat)
Accent:     #C4622D (tembaga/copper)
Muted:      #8A7B6E
```

#### Option B: Ink & Paper (Editorial)
```
Background: #F5F4F0
Surface:    #FFFFFF  
Text:       #111111
Accent:     #D4550A (merah oranye)
Secondary:  #2D5F3F (hijau tua forest)
```

#### Option C: Deep Navy (Profesional tapi tidak dingin)
```
Background: #0B1221
Surface:    #141E30
Text:       #E8EAF0
Accent:     #E8A045 (amber emas)
Muted:      #6B7A99
```

#### Option D: Monochromatic dengan Satu Warna Eksplosif
```
Background: #F8F8F7
Surface:    #FFFFFF
Text:       #1C1C1A
Accent:     #FF3D00 (satu warna mencolok, tegas)
Muted:      #9A9A96
```

---

### [INSIGHT] Cara Membuat Warna Terasa "Berjiwa"
Warna AI terasa flat karena menggunakan hex value murni. Tambahkan karakter dengan:

1. **Warna yang sedikit "off"** — jangan biru murni `#0000FF`, pakai `#2B4EE6` (sedikit ungu)
2. **Background bukan putih murni** — `#FFFFFF` terlalu keras. Gunakan `#FAFAF8` atau `#F9F7F4`
3. **Text bukan hitam murni** — `#000000` terlalu aggressive. Gunakan `#111111` atau `#1A1410`
4. **Warna aksen dengan saturation yang tidak maksimal** — 100% saturation terasa murah

---

## 3. Tipografi

### [ENFORCE] Pilih Font Berdasarkan Kepribadian Merek, Bukan Popularitas

**Font yang HARUS dihindari (terlalu umum):**
- Inter (dipakai semua startup SaaS)
- Roboto (default Google, terasa corporate)
- Poppins (terlalu "website jasa digital 2020")
- Montserrat (sama overused-nya dengan Poppins)

**Alternatif yang lebih berkarakter:**

| Kategori | Font | Kepribadian |
|----------|------|-------------|
| Heading Editorial | DM Serif Display | Elegan, editorial, premium |
| Heading Modern | Sora | Bersih, teknologi, tidak generik |
| Heading Quirky | Fraunces | Unik, warm, artisanal |
| Heading Bold | Space Grotesk | Tech, bold, distinctive |
| Body Readable | DM Sans | Bersih tapi punya karakter |
| Body Serif | Lora | Hangat, readable, editorial |
| Mono | JetBrains Mono | Code blocks, data, precision |
| Display | Clash Display | High-end, fashion, premium |

---

### [ENFORCE] Type Scale yang Proporsional

Gunakan skala modular, bukan angka random:

```css
/* Modular scale: 1.25 (Major Third) */
--text-base: 1rem;        /* 16px — body */
--text-lg: 1.25rem;       /* 20px — lead */
--text-xl: 1.5625rem;     /* 25px — h4 */
--text-2xl: 1.9531rem;    /* 31px — h3 */
--text-3xl: 2.4414rem;    /* 39px — h2 */
--text-4xl: 3.0518rem;    /* 49px — h1 */
--text-5xl: 3.8147rem;    /* 61px — display */

/* Line height */
--leading-tight: 1.15;    /* Heading besar */
--leading-snug: 1.3;      /* Heading medium */
--leading-normal: 1.5;    /* Subheading */
--leading-relaxed: 1.65;  /* Body text */
--leading-loose: 1.8;     /* Long-form article */
```

---

### [INSIGHT] Kerning & Letter Spacing yang Sering Diabaikan

```css
/* Heading besar: tracking negatif untuk terasa lebih tight & premium */
h1, h2 { letter-spacing: -0.03em; }

/* Body text: sedikit positif untuk readability */
p { letter-spacing: 0.01em; }

/* All-caps label: HARUS ada tracking positif */
.label-uppercase { 
  text-transform: uppercase;
  letter-spacing: 0.08em; /* Tanpa ini, all-caps susah dibaca */
  font-size: 0.75rem;
}

/* Besar font & negative tracking = headline premium */
.hero-headline {
  font-size: clamp(3rem, 8vw, 6rem);
  letter-spacing: -0.04em;
  line-height: 1.05;
}
```

---

## 4. Spacing & Layout

### [ENFORCE] Grid System yang Konsisten

```css
.container {
  width: 100%;
  max-width: 1280px;
  margin-inline: auto;
  padding-inline: clamp(1rem, 5vw, 4rem);
}

/* Content widths */
.prose    { max-width: 65ch; }  /* Artikel/teks panjang */
.narrow   { max-width: 720px; } /* Form, CTA section */
.standard { max-width: 960px; } /* Konten utama */
.wide     { max-width: 1280px; }/* Full layout */
.full     { max-width: 100%; }  /* Edge-to-edge */
```

---

### [INSIGHT] Gunakan `clamp()` untuk Fluid Spacing

Daripada media query untuk setiap spacing:

```css
/* ❌ Cara lama */
.section { padding: 48px 0; }
@media (min-width: 768px) { .section { padding: 80px 0; } }
@media (min-width: 1200px) { .section { padding: 120px 0; } }

/* ✅ Fluid — otomatis scale antara viewport */
.section {
  padding-block: clamp(3rem, 8vw, 8rem);
}

/* Gap yang fluid */
.grid {
  gap: clamp(1.5rem, 4vw, 3rem);
}
```

---

## 5. Komponen

### [ENFORCE] Button States yang Lengkap

Setiap button HARUS punya semua state:

```css
.btn-primary {
  /* Default */
  background: var(--color-accent);
  color: white;
  padding: 0.75rem 1.5rem;
  border-radius: var(--radius-md);
  font-weight: 600;
  transition: all var(--duration-fast) var(--ease-out);
  position: relative;
  overflow: hidden;

  /* Hover */
  &:hover {
    background: var(--color-accent-hover);
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(196, 98, 45, 0.3);
  }

  /* Active/Pressed */
  &:active {
    transform: translateY(0px) scale(0.99);
    box-shadow: none;
  }

  /* Focus (accessibility) */
  &:focus-visible {
    outline: 2px solid var(--color-accent);
    outline-offset: 3px;
  }

  /* Disabled */
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
    transform: none;
    box-shadow: none;
  }

  /* Loading state */
  &.is-loading {
    color: transparent;
    pointer-events: none;
    /* Spinner via ::after */
  }
}
```

---

### [INSIGHT] Card yang Tidak Generik

**❌ Card AI:** Border 1px abu + shadow kecil + radius 8px + padding 16px + judul + deskripsi

**✅ Card yang berkarakter:**
```css
/* Option 1: Layered card */
.card-layered {
  background: white;
  border-radius: var(--radius-xl);
  padding: var(--space-8);
  box-shadow: 
    0 0 0 1px rgba(0,0,0,0.04),
    0 2px 4px rgba(0,0,0,0.04),
    0 8px 24px rgba(0,0,0,0.06);
  transition: box-shadow var(--duration-base) var(--ease-out),
              transform var(--duration-base) var(--ease-out);
  
  &:hover {
    box-shadow:
      0 0 0 1px rgba(0,0,0,0.06),
      0 4px 8px rgba(0,0,0,0.06),
      0 16px 40px rgba(0,0,0,0.10);
    transform: translateY(-2px);
  }
}

/* Option 2: Bordered card dengan accent */
.card-accent {
  background: white;
  border: 1.5px solid #E5E2DC;
  border-top: 3px solid var(--color-accent); /* Accent line di atas */
  border-radius: var(--radius-md);
  padding: var(--space-6);
}

/* Option 3: Glass card (untuk dark bg) */
.card-glass {
  background: rgba(255,255,255,0.06);
  backdrop-filter: blur(16px);
  border: 1px solid rgba(255,255,255,0.10);
  border-radius: var(--radius-xl);
}
```

---

## 6. Dark Mode

### [ENFORCE] Dark Mode bukan Inversi Warna

**❌ Cara salah:** `filter: invert(1)` atau sekadar mengganti background hitam

**✅ Dark mode yang benar:**
```css
/* Light mode tokens */
:root {
  --bg-base: #FAFAF8;
  --bg-surface: #FFFFFF;
  --bg-elevated: #F5F3EE;
  --text-primary: #111111;
  --text-secondary: #555550;
  --border-subtle: #E5E2DC;
  --shadow-color: 0deg 0% 0%;
}

/* Dark mode tokens — bukan sekadar inversi */
[data-theme="dark"] {
  --bg-base: #111110;       /* Bukan #000000 */
  --bg-surface: #1A1A18;    /* Surface sedikit lebih terang dari base */
  --bg-elevated: #242422;   /* Elevated lebih terang lagi */
  --text-primary: #EEEEEC;  /* Bukan #FFFFFF murni */
  --text-secondary: #999990;
  --border-subtle: #2A2A28;
  --shadow-color: 0deg 0% 0%;
}
```

---

### [INSIGHT] Elevation System di Dark Mode
Di dark mode, elevation ditunjukkan dengan **kecerahan**, bukan shadow:

```
Level 0 (base):     #111110
Level 1 (surface):  #1A1A18  (+1 stop lebih terang)
Level 2 (elevated): #242422  (+2 stop lebih terang)
Level 3 (overlay):  #2E2E2B  (+3 stop lebih terang)
Level 4 (modal):    #383836  (+4 stop lebih terang)
```

Material Design 3 menggunakan pendekatan ini. Ini lebih natural untuk mata manusia di kondisi gelap.

---

## 7. Icon System

### [ENFORCE] Konsistensi Icon Style

Pilih SATU style dan konsisten:
- **Outline** (Phosphor Icons, Lucide) — Untuk UI modern, bersih
- **Duotone** (Phosphor Duotone) — Untuk visual yang lebih kaya
- **Filled** (Heroicons Solid) — Untuk keterbacaan di size kecil
- **Custom SVG** — Jika merek butuh karakter sangat spesifik

**Jangan mix outline dan filled dalam satu halaman.**

---

### [INSIGHT] Ukuran Icon yang Sering Salah
```css
/* Icon di dalam teks inline */
.inline-icon { width: 1em; height: 1em; vertical-align: -0.15em; }

/* Icon standalone (navigasi, fitur) */
.feature-icon { width: 24px; height: 24px; }

/* Icon dekoratif besar */
.hero-icon { width: 48px; height: 48px; }

/* Stroke width yang proporsional */
/* Size 16px → stroke 1.5px */
/* Size 20-24px → stroke 1.5-2px */
/* Size 32px+ → stroke 2px */
```

---

## 8. Form Design

### [ENFORCE] Form yang Tidak Menyebabkan Friction

```css
.form-input {
  /* Ukuran yang cukup besar untuk touch */
  min-height: 48px;
  padding: 0.75rem 1rem;

  /* Border yang jelas tapi tidak agresif */
  border: 1.5px solid #D4CFC8;
  border-radius: var(--radius-md);
  background: white;

  /* Transisi smooth untuk semua state */
  transition: border-color var(--duration-fast) var(--ease-out),
              box-shadow var(--duration-fast) var(--ease-out);

  &:hover { border-color: #B0A89E; }

  &:focus {
    outline: none;
    border-color: var(--color-accent);
    box-shadow: 0 0 0 3px rgba(196, 98, 45, 0.12);
  }

  &.is-error {
    border-color: #DC2626;
    box-shadow: 0 0 0 3px rgba(220, 38, 38, 0.08);
  }
}
```

**Prinsip form yang konversi:**
- [ ] Label selalu di atas input (bukan placeholder yang hilang saat diketik)
- [ ] Error message spesifik ("Minimal 8 karakter, minimal 1 angka") bukan "Password salah"
- [ ] Auto-focus field pertama saat modal/form terbuka
- [ ] Submit button disabled saat sedang proses (hindari double submit)
- [ ] Success state yang jelas setelah submit
