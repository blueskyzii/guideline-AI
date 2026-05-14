# ⚠️ Guideline 09 — Anti-AI Patterns

> Ini adalah daftar pola yang HARUS dihindari. Jika kamu melihat salah satunya di output, mulai ulang.

---

## 🚨 Apa itu "AI Pattern"?

"AI Pattern" adalah pola default yang dihasilkan AI karena:
1. Paling sering muncul di training data
2. Paling "aman" dan tidak kontroversial
3. Memenuhi brief secara literal tapi tanpa *sudut pandang*

Website yang terasa "AI-made" bukan soal teknologi — ini soal **keputusan desain yang tidak punya alasan kuat**, copy yang terasa ditulis oleh mesin, dan layout yang bisa jadi milik siapa saja.

---

## 🎨 Anti-Pattern: Visual & Desain

### ❌ AP-01: Hero Section Formula
**Pola:** Gambar/ilustrasi di kanan + Headline di kiri + Subheadline + 2 tombol CTA + blob gradient di background

Ini muncul di **80%+ website SaaS yang dibuat AI**. Semua terasa sama.

**Alternatif:**
- Headline full-width dengan ukuran besar, tidak ada gambar di hero
- Video background loop yang menunjukkan produk secara langsung
- Demo interaktif langsung di hero (bukan screenshot)
- Split screen yang tidak simetris (70/30 bukan 50/50)
- Teks dan white space sebagai hero (editorial style)

---

### ❌ AP-02: Gradient Biru-Ungu
**Pola:** `background: linear-gradient(135deg, #667eea 0%, #764ba2 100%)`

Atau variasinya: biru ke cyan, ungu ke pink, biru ke hijau.

Ini adalah default gradient AI. Terasa 2020.

**Alternatif:**
- Solid color yang kuat dan berani
- Gradient yang tidak biasa: coklat tua ke hitam, krem ke oranye tembaga
- Gradient noise (grainy gradient) — tools: grainy-gradient.com
- Mesh gradient yang halus
- Tidak ada gradient sama sekali — typografi yang kuat lebih powerful

---

### ❌ AP-03: 3-Column Feature Grid dengan Icon
**Pola:**
```
[Icon 🚀]         [Icon 📊]         [Icon 🔒]
Fitur 1           Fitur 2           Fitur 3
Deskripsi singkat  Deskripsi singkat  Deskripsi singkat
```

**Masalah:** Semua fitur terasa equal, tidak ada hierarchy, tidak ada cerita.

**Alternatif:**
- Satu fitur utama dengan demo visual, fitur lain lebih kecil di bawah
- Horizontal scrollable cards dengan visual yang kaya
- Accordion dengan penjelasan lebih dalam per fitur
- Numbered steps (jika ada urutan logis)
- Teks panjang yang menceritakan fitur (editorial style)

---

### ❌ AP-04: Testimonial Carousel dengan Bintang 5
**Pola:** Foto profile stock → "★★★★★" → Quote pendek generik → Nama + Jabatan

**Masalah:** Semua orang tahu ini bisa dikarang. Zero credibility.

**Alternatif:**
- Screenshot testimonial asli dari Twitter/X, Google Review, WhatsApp
- Case study singkat dengan angka nyata
- Video testimonial (pendek, 30-60 detik, dari HP mereka sendiri — lebih authentic)
- Testimonial panjang dengan konteks sebelum dan sesudah

---

### ❌ AP-05: Pricing Table Tiga Kolom
**Pola:** Basic / Pro / Enterprise — selalu tiga kolom, kolom tengah "Most Popular"

**Masalah:** Terasa template. Semua website SaaS persis sama.

**Alternatif:**
- Dua pilihan yang jelas (kebanyakan bisnis tidak butuh 3)
- Slider yang kalkulasikan harga berdasarkan usage
- Pricing berdasarkan team size atau fitur yang dipilih (build-your-own)
- Annual billing yang menonjol dengan framing yang jelas

---

### ❌ AP-06: Font Kombinasi Inter + Inter Bold
**Masalah:** Inter dipakai oleh setiap startup SaaS. Tidak berkarakter.

**Ciri lain font pilihan AI:**
- Semua heading pakai font yang sama dengan body
- Tidak ada kontras tipografi yang menarik
- Font weight terlalu seragam (semua 400 dan 700)

---

### ❌ AP-07: Warna Aksen Biru #3B82F6
Ini adalah Tailwind CSS `blue-500`. Versi lainnya: `#2563EB`, `#1D4ED8`.

Juga sering muncul: Ungu Tailwind `#8B5CF6`, Hijau `#10B981`.

---

### ❌ AP-08: Illustration Undraw / Storyset
Ilustrasi flat 2D dengan karakter manusia abstrak berwarna biru-ungu dari Undraw, Storyset, atau Freepik.

Semua website yang malas pakai ini. Terasa seperti placeholder.

**Alternatif:**
- Screenshot produk nyata (jauh lebih efektif untuk software)
- Foto tim atau behind-the-scenes (authentic)
- Custom illustration yang konsisten dengan brand character
- Abstract/geometric visual yang unik
- Tidak ada illustration sama sekali — konten yang kuat lebih baik

---

## ✍️ Anti-Pattern: Copywriting

### ❌ AP-09: Tagline dengan Kata-kata Ini
Hindari kata-kata berikut dalam headline karena sudah terlalu sering dipakai:

```
Powerful       Seamless        Revolutionary    Transform
Unleash        Elevate         Streamline       Next-level
Game-changing  Best-in-class   World-class      Enterprise-grade
Leverage       Utilize         Synergy          Ecosystem
Holistic       Robust          Scalable         Cutting-edge
```

Versi Bahasa Indonesia:
```
Solusi terbaik    Platform terdepan    Inovasi terkini
Tingkatkan        Optimalkan           Wujudkan impian
Revolusioner      Terpercaya           Profesional
```

---

### ❌ AP-10: "We Help [Target Audience] [Do Thing] So They Can [Outcome]"
Formula ini sudah dipakai jutaan kali. Terlalu template, tidak ada kepribadian.

---

### ❌ AP-11: FAQ yang Tidak Menjawab Keberatan
**Pola AI:** FAQ berisi pertanyaan yang tidak pernah benar-benar ditanyakan orang.

```
❌ "Apakah platform Anda mudah digunakan?"
   → Tidak ada yang menanyakan ini. Ini adalah kesempatan untuk menjual, dibuang sia-sia.

✅ "Saya sudah pakai Excel bertahun-tahun, apakah worth it pindah?"
   → Ini pertanyaan yang benar-benar ada di kepala calon pelanggan.
```

---

### ❌ AP-12: About Page yang Bicara Soal Visi-Misi
**Pola:** "Misi kami adalah memberdayakan UMKM dengan teknologi terdepan..."

Tidak ada yang peduli dengan visi-misi korporat. Orang mengunjungi About page karena ingin tahu: **"Siapa orang di balik ini? Apa yang membuat mereka qualified? Mengapa saya bisa percaya?"**

**Alternatif:**
- Cerita awal yang jujur ("Kami membangun ini karena kami sendiri frustasi dengan...")
- Foto tim yang nyata (bukan foto profesional berlebihan)
- Angka yang meaningful (bukan "Didirikan 2023" tapi "1.200 toko yang sudah percaya")
- Press mentions (jika ada) atau notable customers

---

## 🏗️ Anti-Pattern: Struktur & Kode

### ❌ AP-13: Everything is a Card
**Pola:** Setiap konten dibungkus dalam card dengan border-radius 8px dan shadow.

Ketika semua hal adalah card, tidak ada yang terasa spesial.

**Pertanyaan:** *Apakah ini benar-benar butuh card? Atau bisa ditampilkan sebagai teks dengan spacing yang baik?*

---

### ❌ AP-14: Terlalu Banyak Animasi Scroll
**Pola:** Setiap element fade-in saat scroll. Semua section punya parallax.

**Masalah:**
- Menambah cognitive load
- Bisa menyebabkan motion sickness
- Memperlambat rendering
- Terasa gimmicky, bukan purposeful

**Aturan:** Animasi scroll hanya untuk elemen yang *membutuhkan penekanan temporal*. Bukan semua elemen.

---

### ❌ AP-15: Tombol CTA di Setiap Section
**Pola:** Setiap section diakhiri dengan tombol "Mulai Sekarang" atau "Hubungi Kami".

Jika CTA ada di mana-mana, ia kehilangan maknanya.

**Aturan:** CTA yang efektif muncul di:
1. Hero section (setelah value proposition)
2. Setelah social proof/testimonial kuat
3. Di bagian akhir halaman (last chance)

---

### ❌ AP-16: Inline Style sebagai Solusi
**Pola:** `style="margin-top: 24px; color: #333; font-size: 14px;"`

Ini tanda tidak ada design system. Tidak maintainable.

---

### ❌ AP-17: Lorem Ipsum
Tidak ada alasan menggunakan lorem ipsum di output akhir.
Jika konten belum tersedia, **minta konten nyata atau buat placeholder konten yang realistis**.

---

### ❌ AP-18: Placeholder Image dari Placehold.co
```html
<!-- ❌ Tidak acceptable di output akhir -->
<img src="https://placehold.co/600x400" alt="">

<!-- ✅ Gunakan gambar nyata, generate dengan AI, atau konten yang meaningful -->
```

---

## 🔧 Anti-Pattern: Teknis

### ❌ AP-19: Tidak Ada Error State
**Pola:** Membuat form/fetch yang hanya menangani *happy path*.

Setiap operasi async HARUS punya:
- Loading state
- Success state  
- Error state (dengan pesan yang helpful)
- Empty state (jika berlaku)

---

### ❌ AP-20: Hardcode Nilai yang Seharusnya di Konfigurasi
```javascript
// ❌ Hardcode
const API_URL = 'https://api.myapp.com/v1';
const JWT_SECRET = 'mysecretkey123';
const MAX_UPLOAD_SIZE = 10485760;

// ✅ Environment variables
const API_URL = process.env.API_URL;
const JWT_SECRET = process.env.JWT_SECRET;
const MAX_UPLOAD_SIZE = parseInt(process.env.MAX_UPLOAD_SIZE || '10485760');
```

---

### ❌ AP-21: Magic Numbers Tanpa Konteks
```javascript
// ❌ Apa artinya 86400?
if (tokenAge > 86400) expireToken();

// ✅ Jelas dan maintainable
const ONE_DAY_IN_SECONDS = 60 * 60 * 24; // 86400
if (tokenAge > ONE_DAY_IN_SECONDS) expireToken();
```

---

### ❌ AP-22: Tidak Ada Pagination untuk List
**Pola:** `SELECT * FROM products` — tanpa LIMIT.

Setiap endpoint yang return list **HARUS** punya pagination dari awal.
Menambahkan pagination setelah data bertumbuh = breaking change yang menyakitkan.

---

## ✅ Checklist Final Sebelum Deliver

Gunakan checklist ini sebelum menyerahkan pekerjaan:

### Desain
- [ ] Tidak ada gradient biru-ungu tanpa alasan kuat
- [ ] Font punya karakter dan alasan dipilih
- [ ] Tidak semua section pakai layout yang sama (variasi!)
- [ ] Whitespace terasa intentional, bukan akibat konten kurang
- [ ] Mobile dan desktop terlihat didesain terpisah, bukan responsive saja
- [ ] Tidak ada stock photo manusia yang terlalu sempurna

### Copy
- [ ] Headline spesifik dan berorientasi outcome
- [ ] Tidak ada kata-kata dalam daftar AP-09
- [ ] Setiap CTA menjelaskan apa yang akan terjadi setelah diklik
- [ ] Error messages empatik dan memberikan solusi
- [ ] Tidak ada lorem ipsum

### Kode
- [ ] Tidak ada inline style yang bisa digantikan token/class
- [ ] Semua gambar punya `alt` yang deskriptif
- [ ] Ada loading, error, dan empty state
- [ ] Tidak ada hardcode values yang seharusnya di env
- [ ] Navigasi bisa dilakukan dengan keyboard
- [ ] `prefers-reduced-motion` direspek

### Backend
- [ ] Semua input divalidasi di server (tidak percaya client)
- [ ] Error yang expose ke user bersifat user-friendly
- [ ] Rate limiting ada di endpoint auth
- [ ] Tidak ada SQL query tanpa pagination
- [ ] Logs meaningful (bukan console.log('here'))

### Database
- [ ] Ada index untuk kolom yang sering di-WHERE
- [ ] Foreign keys terdefinisi dengan benar
- [ ] Migration reversible (ada rollback)
- [ ] Tidak ada password plain text
