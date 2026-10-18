# PRD — Product Requirements Document
**Aplikasi Pencatatan Keuangan Pribadi**

| Field | Nilai |
|---|---|
| Dokumen | `prd.md` |
| Role penulis | Business Analyst Agent |
| Referensi | `brd.md` (konteks & ruang lingkup), `frd.md` (alur detail), `blueprint.md`, `techstack.md` |
| Dirujuk oleh | `frd.md`, `blueprint.md`, `roadmap.md` |
| Versi | 1.0 — 18 Okt 2026 |
| Status | Draft Review |

---

## 1. Ringkasan Produk

Single-user web app (Nuxt 4 + Nuxt UI 4) + bot Telegram terpisah untuk mencatat keuangan pribadi, mengontrol budget, dan memberi alert. Sumber data Fase 1 = **input manual**; integrasi bank = opsional Fase 3–4 (lihat `brd.md` §4.3).

**Persona tunggal:**

| Persona | Deskripsi | Kebutan utama |
|---|---|---|
| Pemilik | Pekerja dengan gaji bulanan, ingin menabung, malas spreadsheet | Input cepat, tahu sisa budget kapan saja, diingatkan sebelum kebablasan |

## 2. Fitur (MoSCoW)

### M — Must (MVP / Fase 1)

#### F1. Sumber Data — Input Manual
- Input **pemasukan (gaji)** dan **pengeluaran** lewat dashboard & Telegram.
- Field: tanggal, nominal (integer rupiah, tanpa float), keterangan, arah (masuk/keluar), sumber data (`manual`).
- Saldo awal tercatat: **Rp300.000** (dapat diedit di settings).
- Opsi opsional Fase 3+: import e-Statement CSV/PDF, email Gmail API, aggregator (urutan & catatan di `brd.md` §4.3).

#### F2. Pencatatan Transaksi
- Daftar transaksi + filter (periode, kategori, arah, sumber, pencarian keterangan).
- Nominal disimpan sebagai integer rupiah.
- Keterangan asli bank (untuk jalur import) dipertahankan apa adanya.

#### F3. Kategori Otomatis
- Rule-based engine lebih dulu (keyword/merchant → kategori, prioritas, confidence 1.0).
- **AI (LLM) sebagai fallback** hanya untuk transaksi yang tidak kena rule.
- Koreksi user disimpan sebagai **rule baru** (auto-learn).
- Taxonomy default: kategori inti (Makan & Minum, Transportasi, Belanja, Tagihan, Kesehatan, Hiburan, Gaji/Pemasukan, Lainnya) + sub-kategori opsional.

#### F4. Budget & Ambang Batas
- Batas pengeluaran default **Rp300.000 per periode**; periode dapat diatur: harian / mingguan / bulanan.
- Alert bertahap: **50% (info), 80% (warning), 100% (kritis), >100% (melebihi)**.
- Alert saldo minimum default **< Rp50.000** (dapat diubah).
- Semua ambang dapat diubah **dari dashboard dan dari bot Telegram**.

#### F5. Rekonsiliasi Saldo
- Saldo tercatat (hitungan sistem) vs saldo aktual (input user) dengan tanda selisih.
- Wizard "tandai direkonsiliasi" per periode.

#### F6. Notifikasi Bot Telegram (bot terpisah)
- Pesan otomatis saat ambang tercapai/terlewati, saldo minimum, atau anomali.
- Format contoh: `⚠️ Pengeluaran minggu ini Rp312.500, melebihi batas Rp300.000 (104%). Terbesar: Makan Rp120.000.`
- Perintah: `/saldo`, `/ringkasan`, `/budget`, `/setbatas <nominal>`, `/kategori`, `/mute`.
- Keamanan: hanya `chat_id` terdaftar, webhook dengan secret token, rate limit.
- Opsi integrasi ke orchestrator/bot Telegram yang sudah ada (via webhook/API) didokumentasikan sebagai alternatif.

#### F7. AI Agent Pemantau Keuangan
- Berjalan terjadwal (cron) **dan** setiap ada transaksi baru masuk.
- Tugas: hitung pengeluaran periode berjalan, cek ambang, klasifikasi kategori, deteksi anomali (transaksi tidak biasa/besar/beruntun), kirim alert, ringkasan harian & mingguan.
- **Perhitungan ambang & saldo WAJIB deterministik di kode**; LLM hanya untuk kategori, penjelasan, ringkasan bahasa natural.
- **Idempoten**: alert sama tidak terkirim dua kali.
- **Audit trail**: setiap keputusan agent dicatat.

#### F8. Dashboard Web
- Ringkasan saldo, progres budget, grafik pengeluaran per kategori & per waktu, daftar transaksi + filter, pengaturan ambang, riwayat alert.
- Single user, **login aman (passkey/OTP)**, mobile-first.

### S — Should (Fase 2)
- Ringkasan mingguan otomatis via Telegram (digest 21:00 WIB).
- Anomali detection berbasis statistik sederhana (N× rata-rata kategori, nominal kembar < 24 jam).
- Export CSV laporan.
- Input natural language di Telegram (`makan siang 50rb` → parser `rb/k/jt`).

### C — Could (Fase 3)
- Import e-Statement CSV/PDF + deteksi duplikat lintas sumber.
- Parsing email BCA via Gmail API (read-only).
- Run-rate / proyeksi "kalau tren begini, akhir bulan RpX".

### F — Future (Fase 4)
- Aggregator open finance berizin (setelah verifikasi izin BI/OJK & dukungan BCA perorangan).
- Multi-wallet, multi-periode lanjutan, integrasi orchestrator eksternal.

## 3. Kebutuhan Non-Fungsional

| ID | Kategori | Kebutuhan |
|---|---|---|
| NFR1 | Performa | Dashboard memuat < 3 dtk pada jaringan 4G; API < 500ms p95 |
| NFR2 | Keamanan | Enkripsi at-rest (AES-256-GCM) & in-transit (TLS 1.2+/HSTS); secrets di env/secret manager; least privilege — detail `security.md` |
| NFR3 | Kepatuhan | UU PDP (retensi, hak hapus, notifikasi insiden ≤3×24 jam), PSE bila publik |
| NFR4 | Ketersediaan | Uptime monitoring; target 99% bulanan (skala pribadi) |
| NFR5 | Portabilitas | Docker Compose di VPS + Cloudflare Tunnel; backup DB terjadwal & teruji restore |
| NFR6 | Maintainability | Struktur Nuxt 4 (`app/`, `server/`, `shared/`, `test/`); logging terstruktur |
| NFR7 | Aksesibilitas | Mobile-first, kontras semantic color Nuxt UI, navigasi keyboard |
| NFR8 | Idempotency | Semua alert & import idempoten (dedup key) |

## 4. Ketergungan Teknis (Ringkas)
- Frontend/Backend: **Nuxt 4** + **Nuxt UI 4** (design token `@theme`, semantic color) — `techstack.md`
- Data model & komponen sistem — `blueprint.md`
- Deploy: VPS Docker Compose + Cloudflare (DNS, Tunnel, WAF, SSL) — `roadmap.md` Fase 4

## 5. Parameter Default (Konstanta Produk)

| Parameter | Default | Dapat diubah |
|---|---|---|
| Saldo awal | Rp300.000 | Ya (settings) |
| Batas pengeluaran per periode | Rp300.000 | Ya (dashboard & `/setbatas`) |
| Periode budget | Mingguan | Ya (harian/mingguan/bulanan) |
| Ambang alert | 50% / 80% / 100% / >100% | Tidak (fixed), status mute bisa |
| Saldo minimum | Rp50.000 | Ya (dashboard & bot) |
| Digest harian | 21:00 WIB | Ya |
| Retensi log agent | 90 hari | Ya (settings) |

## 6. Acceptance Criteria (Produk)

- [ ] AC-P1: User dapat menambah transaksi masuk/keluar lengkap dengan kategori dalam ≤ 3 langkah di mobile.
- [ ] AC-P2: Transaksi tanpa rule kategori diberi label oleh LLM fallback, dan koreksi user tersimpan sebagai rule yang dipakai di transaksi berikutnya.
- [ ] AC-P3: Saat pengeluaran periode berjalan mencapai 50%/80%/100%/melebihi Rp300.000, alert terkirim tepat satu kali per ambang per periode.
- [ ] AC-P4: Perintah `/saldo` membalas saldo tercatat + selisih vs batas; `/setbatas 500000` mengubah ambang dan berikutnya memakai ambang baru.
- [ ] AC-P5: Dashboard menampilkan grafik per kategori & garis waktu, progres budget, dan riwayat alert yang sama dengan data Telegram.
- [ ] AC-P6: Login hanya untuk satu akun pemilik (passkey/OTP); sesi kedaluwarsa otomatis.
- [ ] AC-P7: Menonaktifkan LLM (tanpa API key) tidak menggagalkan pencatatan — hanya fitur kategori fallback & ringkasan yang terdegradasi ke rule/template.
- [ ] AC-P8: Import (bila aktif) transaksi ganda ber-identitas sama tidak tercatat dua kali.

## 7. Referensi
- Konteks & regulasi: `brd.md`
- Alur fungsional per fitur: `frd.md`
- Kriteria terima per fase: `roadmap.md`
