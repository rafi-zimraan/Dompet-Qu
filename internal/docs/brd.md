# BRD — Business Requirements Document
**Aplikasi Pencatatan Keuangan Pribadi**

| Field | Nilai |
|---|---|
| Dokumen | `brd.md` |
| Role penulis | Business Analyst Agent |
| Referensi | `prd.md`, `frd.md`, `security.md`, `roadmap.md` |
| Dirujuk oleh | Semua dokumen di `internal/docs/` |
| Versi | 1.0 — 18 Okt 2026 |
| Status | Draft Review |

---

## 1. Ringkasan Bisnis

Aplikasi **Pencatatan Keuangan Pribadi** adalah sistem single-user untuk mencatat pemasukan (gaji) dan pengeluaran harian, mengontrol anggaran (budget), dan memberi peringatan dini via bot Telegram — sehingga pemilik tahu persis berapa uang yang keluar, kapan mendekati batas, dan apakah saldo masih aman.

**Pernyataan masalah:** Pencatatan manual di notes/spreadsheet tidak konsisten, tidak ada peringatan saat pengeluaran mendekati batas, dan tidak ada rekonsiliasi antara catatan sendiri dengan saldo rekening aktual.

**Solusi bisnis:** Satu sistem terpadu (dashboard web + bot Telegram) dengan pencatatan cepat, kategorisasi otomatis, ambang batas berjenjang, dan rekonsiliasi saldo.

## 2. Tujuan (Objectives)

| ID | Tujuan | Ukuran Keberhasilan |
|---|---|---|
| O1 | Pencatatan transaksi cepat & lengkap | Transaksi harian tercatat < 30 detik (manual via web/Telegram) |
| O2 | Pengeluaran terkendali terhadap batas | Alert 50%/80%/100% terkirim tepat waktu, false-positive < 5% |
| O3 | Saldo selalu terpantau | Rekonsiliasi saldo catatan vs aktual bisa dilakukan kapan saja, selisih terdeteksi |
| O4 | Notifikasi andal & tidak spam | Alert idempoten (tidak dobel), mute tersedia |
| O5 | Data aman & patuh regulasi | Tidak ada kredensial bank tersimpan, enkripsi aktif, jejak audit lengkap |

## 3. Stakeholder

| Stakeholder | Peran | Kepentingan |
|---|---|---|
| Pemilik (Rafi) | Single user & owner | Kontrol keuangan pribadi, target menabung |
| Developer/Operator | Pengelola sistem | Maintains, deploy, monitoring |
| Regulator (Komdigi, BI, OJK) | Pengawas | Kepatuhan PSE, UU PDP, SNAP bila integrasi bank |

## 4. Ruang Lingkup (Scope)

### 4.1 In Scope (Fase 0–4)
- **Sumber data utama: input manual** — pemasukan (gaji) dan pengeluaran diinput manual via dashboard & bot Telegram.
- Pencatatan transaksi: tanggal, nominal (integer rupiah), keterangan, kategori, sumber data.
- Kategori otomatis berbasis rule (AI sebagai fallback) dengan koreksi user yang menjadi rule baru.
- Budget/ambang batas: default **Rp300.000 per periode** (harian/mingguan/bulanan), alert bertahap **50% → 80% → 100% → melebihi**.
- Alert saldo minimum (default **< Rp50.000**), bisa diubah dari dashboard & bot.
- Rekonsiliasi saldo tercatat vs saldo aktual.
- AI Agent pemantau: cron + trigger transaksi baru; perhitungan ambang deterministik; LLM hanya untuk kategori/penjelasan/ringkasan.
- Notifikasi bot Telegram terpisah dengan perintah `/saldo`, `/ringkasan`, `/budget`, `/setbatas`, `/kategori`, `/mute`.
- Dashboard web single-user, mobile-first, login aman (passkey/OTP).
- Deduplikasi transaksi lintas sumber.

### 4.2 Out of Scope (Non-Goals)
- **Tidak ada** fitur transfer/pembayaran/apapun yang menulis ke rekening (read-only mindset, dan pada Fase 0–2 murni input manual).
- **Tidak menyimpan** User ID/PIN/password/KeyBCA/OTP KlikBCA/myBCA.
- **Tidak melakukan** scraping internet banking atau otomasi login bank.
- **Tidak multi-user/multi-akun** (satu pemilik saja); monetisasi/iklan tidak dibahas di dokumen ini.
- Integrasi rekening bank otomatis = **opsional Fase 3–4**, bukan kebutuhan MVP.

### 4.3 Sumber Data: Prioritas & Jalur Opsional
Keputusan: **pemilik TIDAK memberikan akses rekening BCA ke sistem.** Sistem Fase 0–2 berjalan penuh dengan input manual (nominal gaji + pengeluaran).

Jalur integrasi tetap didokumentasikan sebagai **opsional Fase 3–4** dengan urutan prioritas risiko (detail di `frd.md` §4 dan `security.md` §7):

| Prioritas | Jalur | Status |
|---|---|---|
| 1 (utama) | Input manual (gaji & pengeluaran) | **MVP — Fase 1** |
| 2 (opsional) | Import mutasi e-Statement CSV/PDF myBCA/KlikBCA + parser + deteksi duplikat | Opsional Fase 3 |
| 3 (opsional) | Parsing email notifikasi transaksi BCA via Gmail API (`gmail.readonly`, filter pengirim resmi `@bca.co.id`/`@klikbca.com`) | Opsional Fase 3 |
| 4 (opsional, verifikasi ketat) | Aggregator open finance berizin (mis. Brick/onebrick) — **wajib verifikasi izin PJP BI/OJK PAJK & dukungan BCA perorangan** | Opsional Fase 4 |
| 5 (fallback terakhir) | Input manual | Selalu tersedia |

**DILARANG** (berlaku sepanjang proyek): menyimpan kredensial bank, scraping, otomasi login — lihat `security.md` §7.

## 5. Kebutuhan Regulasi & Kepatuhan

| Regulasi | Implikasi | Dokumen rujukan |
|---|---|---|
| UU No. 27/2022 (UU PDP) | Data keuangan = **data pribadi spesifik** (Pasal 4 ayat (2) huruf f). App = Pengendali Data; lawful basis = persetujuan terekam + ToS; disclosure sebelum pengumpulan; retensi & hak hapus; **notifikasi kegagalan ≤ 3×24 jam** (Pasal 43); sanksi maks 2% pendapatan tahunan (Pasal 57) | `security.md` |
| Permenkominfo No. 5/2020 | Pendaftaran **PSE Lingkup Privat** bila aplikasi dipublikasikan | `security.md`, `roadmap.md` |
| SNAP BI (PADG 23/15/PADG/2021) | Hanya relevan bila memakai Open API pembayaran; aplikasi = Pengguna Layanan Non-PJP → wajib pengujian & verifikasi | `frd.md`, `roadmap.md` |
| PBI No. 23/6/PBI/2021 | "Penyediaan informasi Sumber Dana" = izin PJP BI → alasan aggregator hanya jalur opsional & wajib vendor berizin | `frd.md` |
| POJK No. 4/2025 (PAJK) | Vendor aggregator wajib izin OJK, enkripsi wajib — **verifikasi ulang status izin saat Fase 4** | `roadmap.md` |

## 6. Best Practice yang Diadopsi (Hasil Riset)

1. **Kategorisasi hybrid**: rule (merchant/keyword, confidence 1.0) → LLM fallback → koreksi user menjadi rule baru (pola Actual Budget, Plaid, Bank Jago).
2. **Alert berjenjang** 50/80/100% dengan pesan berisi angka nyata (`Rp312.500 dari Rp300.000`), plus pola run-rate gaya PocketGuard sebagai enhancement.
3. **Dedup idempotent**: key `external_id` bila ada, fallback `(tanggal, nominal, keterangan ternormalisasi, arah)`; re-import tidak menggandakan.
4. **Rekonsiliasi saldo** bergaya wizard "tandai sudah direkonsiliasi" (Actual/YNAB).
5. **Digest terjadwal** ringkasan harian via Telegram (pola KasKecil 21:00 WIB) agar sistem "hidup" tanpa spam.
6. **Core loop cepat via chat**: konfirmasi pencatatan selalu menyebut **sisa budget** (pola DompetRapi).

## 7. Risiko Bisnis

| ID | Risiko | Dampak | Mitigasi |
|---|---|---|---|
| R1 | User malas input manual → data tidak lengkap | Tinggi | Input cepat via Telegram natural language; digest harian mengingatkan |
| R2 | Alert dobel/spam → user mute | Sedang | Idempotency key + `/mute` + rate limit (lihat `frd.md` §7, `security.md` §6) |
| R3 | Perubahan format e-Statement/email BCA (bila Fase 3) | Sedang | Parser terisolasi + fallback input manual |
| R4 | Pelanggaran UU PDP / simpan kredensial | Tinggi | Dilarang eksplisit; audit log; enkripsi (`security.md`) |
| R5 | Vendor aggregator tidak berizin (POJK 4/2025) | Sedang | Verifikasi izin sebelum adopsi; manual tetap fallback |
| R6 | Kehilangan data (VPS crash) | Tinggi | Backup database terenkripsi terjadwal (`roadmap.md` Fase 3) |

## 8. Acceptance Criteria (Tingkat Bisnis)

- [ ] AC-B1: Pemilik dapat mencatat pemasukan gaji dan pengeluaran tanpa akses rekening bank, dan saldo tercatat selalu sinkron dengan input.
- [ ] AC-B2: Sistem mengirim alert tepat pada 50%, 80%, 100%, dan saat melebihi batas Rp300.000 (default), tanpa duplikasi.
- [ ] AC-B3: Alert saldo minimum terkirim saat saldo < Rp50.000 (default, bisa diubah).
- [ ] AC-B4: Rekonsiliasi menampilkan selisih antara saldo tercatat dan saldo aktual bila ada.
- [ ] AC-B5: Tidak ada kredensial bank yang disimpan di sistem (dibuktikan pemeriksaan `security.md` checklist).
- [ ] AC-B6: Semua keputusan agent tercatat dalam audit trail yang dapat ditelusuri.
- [ ] AC-B7: Sistem berjalan production di VPS (Docker Compose) di balik Cloudflare (DNS, Tunnel, WAF, SSL) dengan backup & uptime monitoring aktif.

## 9. Asumsi & Keterbatasan

- Asumsi: pemilik satu orang, satu rekening BCA yang dipantau manual; nominal selalu Rupiah integer (tanpa float).
- Asumsi: LLM tersedia (API key) untuk fallback kategori & ringkasan; bila tidak tersedia, sistem tetap jalan dengan rule saja.
- Keterbatasan: format mutasi/email bank dapat berubah sewaktu-waktu tanpa pemberitahuan.

## 10. Referensi Antar Dokumen

- Fitur & prioritas → `prd.md`
- Alur detail & spesifikasi fungsional → `frd.md`
- Arsitektur & data model → `blueprint.md`
- Stack & struktur Nuxt 4 → `techstack.md`
- Kepatuhan & threat model → `security.md`
- UI & token → `design.md`, `wireframe.md`
- Fase, deliverable, acceptance criteria per fase → `roadmap.md`
