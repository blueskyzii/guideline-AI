# ✍️ Guideline 08 — Copywriting

> Desain yang indah tidak bisa menjual jika copynya buruk. Copy yang baik bahkan bisa menjual dengan desain biasa.

---

## 1. Tone of Voice

### [ENFORCE] Definisikan Kepribadian Merek Sebelum Menulis

Jawab pertanyaan ini sebelum menulis satu kata pun:

**"Jika merek ini adalah orang, ia adalah..."**

Contoh jawaban yang jelas:
> "Seorang teman yang kerja di bidang keuangan — tahu banyak soal uang, tapi tidak menggurui. Berbicara santai, jujur, dan langsung to the point. Tidak pakai jargon keuangan yang bikin pusing."

Tulis 5 kata sifat yang mendeskripsikan nada bicara merek:
- Misalnya: **Jujur, Hangat, Tegas, Praktis, Relatable**

Buat skala sliding untuk setiap dimensi:

```
Formal    ←————————●————→  Kasual
Serius    ←————●——————————→  Playful
Technical ←——————————●——→  Simple
Reserved  ←————————●————→  Ekspresif
Classic   ←——●——————————→  Modern
```

---

### [ENFORCE] Panduan Tone yang Konsisten

| Konteks | Tone | Contoh |
|---------|------|--------|
| Hero/Headline | Bold, langsung | "Rekap otomatis. Tidak ada typo lagi." |
| Fitur | Benefit-focused, spesifik | "Laporan siap dalam 3 detik — bukan 30 menit" |
| Error message | Empatik, solusi-focused | "Ups, koneksi bermasalah. Coba refresh halaman." |
| Success state | Celebratory, warm | "Berhasil! Pesananmu sudah kami terima 🎉" |
| Legal/Policy | Jelas, tidak menakutkan | Hindari jargon hukum yang tidak perlu |
| Email | Personal, conversational | Mulai dengan nama, bukan "Dear User" |

---

## 2. Menulis Headline yang Konversi

### [ENFORCE] Formula Headline yang Bekerja

#### Formula 1: Specific Outcome
> "[Hasil spesifik] dalam [waktu/cara] — [tanpa hambatan yang ditakuti]"

Contoh:
- ✅ "Buka toko online dalam 10 menit — tanpa coding, tanpa IT"
- ✅ "Rekap keuangan bulanan selesai dalam 5 menit — akurat sampai sen"

#### Formula 2: Contrast/Before-After
> "[Situasi buruk sekarang] vs [Situasi baik setelah pakai produk]"

Contoh:
- ✅ "Sudah cukup kehilangan pelanggan karena antrian panjang"
- ✅ "Berhenti rekap manual di Excel. Mulai rekap otomatis hari ini."

#### Formula 3: Pertanyaan Provokatif
> Pertanyaan yang jawabannya sudah pasti "ya" dari target audiens

Contoh:
- ✅ "Masih rekap penjualan pakai buku tulis?"
- ✅ "Kapan terakhir kali kamu tahu persis berapa untungmu hari ini?"

---

### [INSIGHT] Perbedaan Fitur vs Benefit vs Outcome

AI selalu menulis fitur. Yang membeli adalah outcome.

```
Fitur (APA yang ada):
"Dilengkapi laporan penjualan real-time"

Benefit (BAGAIMANA fitur membantu):
"Pantau penjualan tanpa harus nunggu laporan akhir hari"

Outcome (APA yang berubah dalam hidup mereka):
"Kamu bisa pergi ke mana saja sambil tetap tahu kondisi toko"
```

**Selalu tulis di level Outcome, bukan Fitur.**

---

## 3. Menulis Konten Section

### [ENFORCE] Section "Fitur" yang Tidak Membosankan

**❌ Cara AI menulis fitur:**
```
🚀 Fast Performance
Lorem ipsum dolor sit amet consectetur adipiscing elit.

📊 Advanced Analytics
Lorem ipsum dolor sit amet consectetur adipiscing elit.

🔒 Secure & Reliable
Lorem ipsum dolor sit amet consectetur adipiscing elit.
```

**✅ Cara yang benar:**

```
Rekap yang dulu 2 jam, sekarang 3 menit.

Setiap hari kamu habiskan waktu berharga untuk menghitung stok,
merekap penjualan, dan membuat laporan manual. Waktu itu bisa
kamu pakai untuk hal yang lebih penting.

[Tool] mengotomasi semua itu — akurat, real-time, dan tanpa
kamu perlu menyentuhnya setelah setup.
```

---

### [INSIGHT] Prinsip "So What?" untuk Setiap Kalimat

Setiap kalimat yang kamu tulis, tanya: **"So what? Apa artinya bagi pembaca?"**

```
Draft: "Platform kami menggunakan enkripsi AES-256."
So what? → "Data kamu aman meski terjadi peretasan server sekalipun."
Better: "Data kamu terlindungi dengan enkripsi bank-grade — level yang sama dengan transfer ATM."

Draft: "Tersedia di iOS dan Android."
So what? → "Bisa diakses dari mana saja, device apapun."
Better: "Buka dari HP kamu, HP kasir, atau tablet — semuanya real-time."
```

---

## 4. Menulis CTA

### [ENFORCE] CTA yang Mengurangi Friction

Setiap kata dalam CTA harus menjawab: *"Apa yang akan terjadi jika saya klik ini?"*

```
❌ "Submit"          → Tidak ada informasi
❌ "Click Here"      → Tidak ada value
❌ "Get Started"     → Terlalu generik
❌ "Learn More"      → Kemana? Tentang apa?

✅ "Coba Gratis 14 Hari"      → Tahu: gratis, 14 hari
✅ "Buat Akun Gratis"         → Tahu: gratis, buat akun
✅ "Lihat Demo 2 Menit"       → Tahu: demo, butuh 2 menit
✅ "Mulai, Tidak Perlu Kartu" → Menghilangkan friction kartu kredit
✅ "Download Panduan Gratis"  → Tahu: download, panduan, gratis
```

---

### [INSIGHT] Micro-copy di Bawah CTA

Teks kecil di bawah tombol bisa meningkatkan konversi secara signifikan:

```html
<div class="cta-wrapper">
  <a href="/daftar" class="btn-primary">
    Coba Gratis 14 Hari
  </a>
  <!-- Micro-copy yang menghilangkan keberatan -->
  <p class="cta-note">
    Tidak perlu kartu kredit · Batal kapan saja · Setup 5 menit
  </p>
</div>

<!-- Variasi lain: -->
<!-- "Sudah 1.200+ pemilik toko yang percaya" -->
<!-- "Uang kembali jika tidak puas dalam 30 hari" -->
<!-- "Sama seperti yang dipakai [nama merek terkenal]" -->
```

---

## 5. Menulis Error Messages

### [ENFORCE] Error Message yang Tidak Menyalahkan User

**Framework:** Akui masalahnya → Jelaskan apa yang terjadi → Beri solusi konkret

```
❌ "Error: Invalid email format"
✅ "Email tidak valid. Pastikan formatnya benar, contoh: nama@domain.com"

❌ "Authentication failed"
✅ "Email atau password salah. Lupa password? Reset di sini →"

❌ "Server error 500"
✅ "Ada gangguan sementara di sistem kami. Tim kami sudah diberitahu.
    Coba lagi dalam beberapa menit, atau hubungi support kami."

❌ "File too large"
✅ "File kamu terlalu besar (maksimal 10MB). Kompres file terlebih dulu
    atau hubungi kami jika butuh limit lebih besar."

❌ "Session expired"
✅ "Kamu sudah tidak aktif cukup lama. Login kembali untuk melanjutkan.
    Semua data yang belum tersimpan sudah kami amankan."
```

---

## 6. Menulis Email

### [ENFORCE] Email yang Dibuka dan Dibaca

#### Subject Line
```
❌ "Selamat bergabung dengan platform kami!"
✅ "Rini, akun kamu siap — mulai dari sini"

❌ "Newsletter Bulanan"
✅ "3 hal yang bikin warung kopi tetangga lebih ramai dari kamu"

❌ "Notifikasi sistem"
✅ "Ada yang login ke akunmu dari perangkat baru — kamu kah ini?"
```

#### Opening Line
```
❌ "Dear Valued Customer,"
✅ "Hei Rini,"

❌ "Kami senang menyambut kamu di platform kami."
✅ "Setup akun kamu butuh 3 langkah. Yang pertama sudah selesai."
```

#### Body Email
- **Paragraf pendek:** Maksimal 3-4 kalimat per paragraf
- **Satu email, satu tujuan:** Jangan campur promo + tutorial + newsletter
- **Hindari attachment jika bisa:** Taruh konten di halaman web, link dari email
- **Mobile-first:** 60%+ email dibuka di mobile

---

## 7. Copywriting untuk Halaman Harga

### [ENFORCE] Harga yang Tidak Membuat Orang Kabur

**Prinsip anchoring:**
```
Tampilkan paket MAHAL dulu, baru paket yang ingin dibeli.
Setelah melihat Rp 2.000.000/bulan, Rp 500.000/bulan terasa murah.
```

**Framing harga:**
```
❌ "Rp 500.000 per bulan"
✅ "Rp 16.000 per hari — lebih murah dari kopi bengong"

❌ "Enterprise: Hubungi kami"
✅ "Enterprise: Mulai dari Rp 2.000.000/bulan untuk tim 50+ user"
   (Jangan sembunyikan harga enterprise — transparency is trustworthy)
```

**Toggle harga:**
```
Tampilkan toggle "Bulanan / Tahunan" dengan highlight savings:
"Hemat Rp 1.200.000 dengan paket tahunan (2 bulan gratis)"
```

---

### [INSIGHT] FAQ di Halaman Harga

FAQ yang benar bukan sekadar pertanyaan umum — ini adalah **objection handling**.

```
Pertanyaan yang harus ada:
✓ "Apakah ada kontrak yang harus ditandatangani?"
  → "Tidak ada kontrak. Langganan bulanan bisa dibatalkan kapan saja."

✓ "Bagaimana jika saya sudah punya data di sistem lain?"
  → "Kami bantu migrasi data kamu gratis dalam 48 jam."

✓ "Apakah data saya aman?"
  → "Data tersimpan di server Indonesia dengan backup harian."
    (Jangan jawaban generic soal "security")

✓ "Apa yang terjadi saat trial berakhir?"
  → "Kamu akan dapat notifikasi 3 hari sebelum trial berakhir.
    Data kamu tidak akan hilang — disimpan 30 hari setelah trial."
```

---

## 8. Copywriting untuk Indonesia

### [INSIGHT] Nuansa Bahasa yang Sering Salah

**Penggunaan "kamu" vs "Anda":**
- `kamu` = lebih personal, cocok untuk produk konsumer, startup, B2C
- `Anda` = lebih formal, cocok untuk B2B, layanan keuangan, kesehatan

**Konsisten dalam satu produk.** Jangan campur "kamu" dan "Anda".

---

**Hindari terjemahan langsung dari Bahasa Inggris:**
```
❌ "Dapatkan akses instant ke fitur premium kami"
   (Terjemahan kaku dari "Get instant access to our premium features")

✅ "Langsung pakai semua fitur premium — tidak perlu nunggu"

❌ "Leverage our platform untuk scale bisnis kamu"
   (Campur aduk, tidak natural)

✅ "Pakai platform kami untuk kembangkan bisnis kamu lebih cepat"
```

---

**Emoji yang tepat guna:**
```
✅ Boleh digunakan untuk:
   - CTA button untuk reduce formality
   - Success messages ("Berhasil! 🎉")
   - Feature list bullets (menggantikan icon jika desain tidak ada icon)

❌ Jangan gunakan untuk:
   - Error messages (terasa tidak serius)
   - Legal/policy text
   - Heading H1 (tidak baik untuk SEO dan screen reader)
   - Jangan berlebihan — max 1-2 per section
```
