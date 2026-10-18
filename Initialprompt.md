- blueprint.md 
- prd.md
- brd.md 
- frd.md 
- roadmap.md 
- security.md 
- design.md 
- techstack.md 
- wireframe.md 

untuk 
aplikasi Pencatatan Keuangan Bangun sistem Pencatatan Keuangan berbasis:

untuk aplikasi **Pencatatan Keuangan Pribadi**. Bangun sistem Pencatatan Keuangan berbasis:

## 1. Sumber Data: Rekening BCA (read-only)

- Rekening yang dipantau: 1 rekening BCA pribadi, saldo awal Rp300.000.
- Riset dan bandingkan jalur integrasi yang LEGAL dan AMAN untuk rekening BCA perorangan, lalu rekomendasikan satu jalur utama dan satu fallback:
  a. Aggregator open finance berizin di Indonesia (mis. Brick/onebrick.io atau sejenisnya). Verifikasi apakah mendukung BCA perorangan, model biaya, dan cara consent.
  b. Parsing email notifikasi transaksi BCA via Gmail API (read-only scope, filter pengirim resmi BCA).
  c. Import mutasi rekening (CSV/PDF e-Statement dari myBCA/KlikBCA) dengan parser + deteksi duplikat.
  d. Input manual sebagai fallback terakhir.
- DILARANG: menyimpan User ID/PIN/password KlikBCA atau myBCA, scraping internet banking, dan otomasi login bank.
- Sistem hanya MEMBACA data (tidak ada fitur transfer/pembayaran).

## 2. Pencatatan Transaksi

- Transaksi masuk/keluar dengan tanggal, nominal (integer rupiah, tanpa float), keterangan asli bank, kategori, dan sumber data.
- Kategori otomatis (rule-based dulu, AI sebagai fallback) yang bisa dikoreksi user. Koreksi user disimpan sebagai rule baru.
- Deduplikasi transaksi lintas sumber (email vs mutasi).
- Rekonsiliasi saldo: saldo tercatat vs saldo bank, dengan tanda jika selisih.

## 3. Budget & Ambang Batas

- Batas pengeluaran default: Rp300.000 per periode (periode bisa diatur: harian/mingguan/bulanan).
- Alert bertahap: 50%, 80%, 100%, dan saat MELEBIHI batas.
- Alert saldo minimum (mis. saldo < Rp50.000, bisa diatur).
- Semua ambang bisa diubah dari dashboard dan dari bot Telegram.

## 4. AI Agent Pemantau Keuangan

- Berjalan terjadwal (cron) DAN setiap ada transaksi baru masuk.
- Tugas: hitung pengeluaran periode berjalan, cek ambang, klasifikasi kategori, deteksi anomali (transaksi tidak biasa/besar/beruntun), kirim alert, buat ringkasan harian & mingguan.
- Perhitungan ambang & saldo WAJIB deterministik di kode. LLM hanya untuk kategori, penjelasan, dan ringkasan bahasa natural.
- Idempotent: alert yang sama tidak terkirim dua kali. Ada log setiap keputusan agent (audit trail).

## 5. Notifikasi via Bot Telegram

- Pesan otomatis saat ambang tercapai/terlewati, saldo minimum, atau anomali.
- Contoh pesan: "⚠️ Pengeluaran minggu ini Rp312.500, melebihi batas Rp300.000 (104%). Terbesar: Makan Rp120.000."
- Perintah bot: /saldo, /ringkasan, /budget, /setbatas <nominal>, /kategori, /mute.
- Keamanan: hanya chat_id terdaftar yang dilayani, webhook dengan secret token, rate limit.
- Siapkan opsi integrasi ke orchestrator/bot Telegram yang sudah ada (via webhook/API) sebagai alternatif bot terpisah.

## 6. Dashboard Web

- Ringkasan saldo, progres budget, grafik pengeluaran per kategori & per waktu, daftar transaksi + filter, pengaturan ambang, riwayat alert.
- Single user (pemilik), login aman (passkey/OTP), mobile-first.

---

Lakukan riset mengenai software sejenis (aplikasi pencatatan keuangan pribadi di Indonesia & global) serta best practice dan standardnya.

**Standard teknis:**

- Nuxt 4 terbaru: https://nuxt.com/raw/docs/4.x/guide/ai/llms-txt.md
- Nuxt UI 4 terbaru: https://ui.nuxt.com/docs/getting-started/ai/llms-txt
- Design System: https://ui.nuxt.com/raw/docs/getting-started/theme/design-system.md
- Design Token: https://ui.nuxt.com/raw/docs/getting-started/theme/css-variables.md
- Structure: https://nuxt.com/raw/docs/4.x/directory-structure.md

**Best practice & regulasi yang berlaku di Indonesia:**

- Prinsip pencatatan keuangan pribadi (budgeting, kategorisasi, rekonsiliasi).
- UU No. 27/2022 tentang Pelindungan Data Pribadi (UU PDP).
- Standar Nasional Open API Pembayaran (SNAP) Bank Indonesia & ketentuan OJK terkait open finance/aggregator.
- Keamanan data finansial: enkripsi at-rest & in-transit, secrets management, prinsip least privilege.

**Buat beberapa tahap sampai production mode:**

- Fase 0: Riset & validasi jalur integrasi BCA
- Fase 1: MVP (import mutasi + input manual + budget + alert Telegram)
- Fase 2: Integrasi otomatis (email/aggregator) + AI Agent
- Fase 3: Hardening keamanan, monitoring, backup
- Fase 4: Production
  Setiap fase wajib punya deliverable, acceptance criteria, dan risiko.

**Deployment:** VPS (Docker Compose) dengan Cloudflare di depan (DNS, Cloudflare Tunnel, WAF, SSL). Sertakan strategi backup database, cron job agent, logging, dan uptime monitoring.

**Eksekusi dengan multi-agent** yang capable, dengan pembagian peran:

1. Research Agent: riset kompetitor, jalur integrasi BCA, regulasi.
2. Business Analyst Agent: brd.md, prd.md, frd.md.
3. Solution Architect Agent: blueprint.md, techstack.md.
4. Security Agent: security.md (threat model, UU PDP, penanganan data bank).
5. UI/UX Agent: design.md, wireframe.md (mengikuti design token Nuxt UI 4).
6. Project Manager Agent: roadmap.md.
7. Reviewer Agent: cek konsistensi antar dokumen, istilah, dan kelengkapan acceptance criteria.

Struktur harus detail, jelas, rapi, saling merujuk antar dokumen, dan dapat diimplementasikan dengan baik dan benar.

dalam 1 folder internal/docs/ 

lakukan riset mengenai software tersebut serta best practice dan standardnya

standard :
nuxt 4 terbaru https://nuxt.com/raw/docs/4.x/guide/ai/llms-txt.md
nuxt ui 4 terbaru  https://ui.nuxt.com/docs/getting-started/ai/llms-txt 
Design System https://ui.nuxt.com/raw/docs/getting-started/theme/design-system.md
Design Token : https://ui.nuxt.com/raw/docs/getting-started/theme/css-variables.md
Structure : https://nuxt.com/raw/docs/4.x/directory-structure.md

Serta Best practice mengenai bidang 
Bisnis Customer Relationship Management yang berlaku di Indonesia

Buat beberapa tahap, supaya sampai ke production mode

Deployment saya mau di VPS Cloudflare

Buat menggunakan multi agent yang Capable & Struktur yang detail jelas rapi serta dapat di implementasikan dengan baik dan benar
