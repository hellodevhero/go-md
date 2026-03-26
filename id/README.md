# Go & SQL — Perpustakaan Belajar Mandiri Lengkap

Seri buku komprehensif dan praktis untuk belajar pemrograman Go dan SQL dari nol hingga membangun aplikasi siap produksi. Setiap buku dibangun di atas buku sebelumnya, mulai dari dasar hingga membangun sistem e-commerce lengkap.

> **English version?** All books are available in [English](../).

---

## Daftar Buku

| # | Buku | Topik | Durasi |
|---|------|-------|--------|
| 1 | [SQL untuk Developer](database/sql-book1-developer-deep.md) | PostgreSQL, desain skema, query, window function | 4 fase |
| 2 | [Dasar-Dasar Go](golang/1.go-fundamental-book.md) | Sintaks, tipe, struct, interface, concurrency, generics | 6 minggu |
| 3 | [Standard Library & Tooling](golang/2.standard-library&tooling.md) | File, JSON, HTTP, database, context, CLI tools | 6 minggu |
| 4 | [Arsitektur & Produksi](golang/3.architecture.md) | Clean architecture, Gin, Redis, Docker, testing | 6 minggu |
| 5 | [Proyek E-commerce](golang/4.ecommerce-project.md) | Proyek lengkap: pembayaran, upload, job, observability | 8 minggu |

---

## Jalur Belajar

```
┌───────────────┐
│  Buku 1:      │     ┌───────────────┐   ┌─────────────────────┐   ┌─────────────────────┐   ┌──────────────────┐
│  SQL untuk    │────▶│  Buku 3:      │──▶│  Buku 4:            │──▶│  Buku 5:            │──▶│  E-commerce      │
│  Developer    │     │  Standard     │   │  Arsitektur &       │   │  Proyek             │   │  Selesai!        │
└───────┬───────┘     │  Library      │   │  Produksi           │   │  E-commerce         │   └──────────────────┘
        │             └───────┬───────┘   └─────────────────────┘   └─────────────────────┘
        │                     │                6 minggu                   8 minggu
        │             ┌───────┴───────┐
        └────────────▶│  Buku 2:      │
                      │  Dasar-Dasar  │
                      │  Go           │
                      └───────────────┘
                          6 minggu
```

**Urutan yang disarankan:**
1. Mulai dengan **Buku 1** (SQL untuk Developer) dan **Buku 2** (Dasar-Dasar Go) secara paralel — tidak perlu pengetahuan awal untuk keduanya
2. Lanjut ke **Buku 3** (Standard Library) setelah menyelesaikan Buku 1 dan 2 — kamu akan butuh pengetahuan SQL untuk bab database
3. Lanjut ke **Buku 4** (Arsitektur) → **Buku 5** (Proyek E-commerce) secara berurutan

---

## Daftar Isi Lengkap

### Buku 1 — SQL untuk Developer

> *Dari Nol Hingga Menulis Query Kompleks dan Merancang Skema Nyata*

- **Fase 1 — Fondasi Relasional**
  - 1.1 Apa Itu Database Relasional
  - 1.2 Tipe Data
  - 1.3 Constraint (Batasan)
  - 1.4 Normalisasi
  - 1.5 Diagram ER
  - Latihan Fase 1

- **Fase 2 — Dasar-Dasar SQL**
  - 2.1 SELECT — Membaca Data
  - 2.2 WHERE — Memfilter Baris
  - 2.3 ORDER BY, LIMIT, OFFSET
  - 2.4 Fungsi Agregat
  - 2.5 GROUP BY dan HAVING
  - 2.6 JOIN
  - 2.7 INSERT, UPDATE, DELETE, UPSERT
  - 2.8 Transaksi
  - Latihan Fase 2

- **Fase 3 — SQL Menengah**
  - 3.1 Subquery
  - 3.2 CTE (Common Table Expression)
  - 3.3 Window Function
  - 3.4 Operasi Himpunan (Set Operation)
  - 3.5 Ekspresi CASE
  - 3.6 Fungsi Bawaan
  - Latihan Fase 3

- **Fase 4 — Desain Skema Dunia Nyata**
  - 4.1 Timestamp
  - 4.2 Soft Delete
  - 4.3 Audit Trail
  - 4.4 Relasi Many-to-Many
  - 4.5 Struktur Pohon (Tree)
  - 4.6 JSONB dalam Praktik
  - 4.7 Migrasi Skema yang Aman
  - 4.8 Pola Skema Umum
  - Latihan Fase 4

- **Capstone — Database E-commerce**

---

### Buku 2 — Dasar-Dasar Go

> *Fase 1: Dari Nol Menjadi Developer Go yang Percaya Diri*

- **Minggu 1** — Setup, Sintaks & Tipe Dasar
- **Minggu 2** — Struct, Method & Pointer
- **Minggu 3** — Interface & Penanganan Error
- **Minggu 4** — Package, Modul & Testing
- **Minggu 5** — Goroutine & Channel
- **Minggu 6** — Konsolidasi & Proyek Capstone

---

### Buku 3 — Standard Library & Pengembangan Dunia Nyata

> *Dari Dasar ke Membangun Backend Service Sesungguhnya*

- **Minggu 1** — Bekerja dengan File & OS
- **Minggu 2** — JSON, HTTP Client & API Eksternal
- **Minggu 3** — Membangun HTTP Server
- **Minggu 4** — Database dengan database/sql
- **Minggu 5** — Context, Middleware & CLI Tools
- **Minggu 6** — Capstone: REST API Lengkap (BookStore)

---

### Buku 4 — Service Kelas Produksi

> *Clean Architecture, Gin, Redis, Docker & Background Job*

- **Minggu 1** — Clean Architecture
- **Minggu 2** — Gin Framework
- **Minggu 3** — Redis: Caching & Session
- **Minggu 4** — Background Job & Worker
- **Minggu 5** — Docker & Deployment
- **Minggu 6** — Testing Lanjutan & Capstone

---

### Buku 5 — Proyek E-commerce

> *Aplikasi Dunia Nyata yang Lengkap*

- **Minggu 1** — Bootstrap Proyek & Katalog Produk
- **Minggu 2** — Upload File & Manajemen Gambar
- **Minggu 3** — Keranjang Belanja
- **Minggu 4** — Pembayaran Stripe & Checkout
- **Minggu 5** — Pemenuhan Pesanan & Notifikasi
- **Minggu 6** — Admin API & Manajemen Inventaris
- **Minggu 7** — Observability: Metrik & Tracing
- **Minggu 8** — Penguatan Keamanan & Deployment Final

---

## Prasyarat

- Komputer dengan macOS, Linux, atau Windows
- Pengalaman dasar pemrograman dalam bahasa apa pun
- Terminal/command line
- Editor kode (VS Code disarankan)

## Cara Menggunakan Buku Ini

Setiap buku diorganisir dalam **minggu** (atau **fase** untuk SQL). Setiap minggu berisi:
- **Bagian konsep** dengan penjelasan dan contoh kode
- **Proyek praktik** untuk menerapkan apa yang sudah dipelajari

Cara terbaik untuk belajar adalah **mengetik setiap contoh kode sendiri** — jangan copy-paste. Membangun muscle memory adalah bagian dari proses belajar.

---

*Dibuat dengan tujuan membawa kamu dari nol pengetahuan Go/SQL hingga membangun aplikasi siap produksi.*
