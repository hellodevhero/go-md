# SQL untuk Developer — Buku 1
### Dari Nol Sampai Menulis Query Kompleks dan Mendesain Schema Nyata

> **Database:** PostgreSQL.
> Jalankan dengan Docker: `docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=password postgres:16-alpine`
> Ketik setiap contoh sendiri. Kerjakan setiap latihan. Jangan loncat-loncat.

---

## Daftar Isi
- [Fase 1 — Fondasi Relasional](#phase-1)
- [Fase 2 — Dasar-Dasar SQL](#phase-2)
- [Fase 3 — SQL Menengah](#phase-3)
- [Fase 4 — Desain Schema Dunia Nyata](#phase-4)
- [Capstone — Database Ecommerce](#capstone)

---

# Fase 1 — Fondasi Relasional

## 1.1 Apa Itu Relational Database Sebenarnya

Sebuah **relational database** menyimpan data dalam **table** — grid terstruktur dari baris dan kolom.
Setiap baris dalam sebuah table merepresentasikan satu hal (satu user, satu produk, satu order).
Setiap kolom merepresentasikan satu atribut dari hal tersebut.

Kata "relational" berasal dari aljabar relasional, sebuah cabang matematika —
bukan dari "hubungan antar table", meskipun hubungan itu memang ada juga.

**Sebuah table adalah himpunan, bukan list.** Artinya:
- Baris tidak punya urutan yang dijamin tanpa `ORDER BY`
- Setiap baris harus bisa diidentifikasi secara unik (makanya ada primary key)
- SQL bersifat deklaratif — kamu bilang *apa* yang kamu mau, database yang menentukan *bagaimana* cara mendapatkannya

### Deklaratif vs Imperatif

SQL bersifat **deklaratif**: kamu mendeskripsikan hasil yang kamu inginkan, bukan langkah-langkah untuk mendapatkannya. Database engine yang mencari cara paling efisien untuk mengeksekusi permintaanmu.

Bandingkan memfilter data di Python (imperatif) vs SQL (deklaratif):

```python
# Python — imperative: you tell the computer HOW to filter
result = []
for book in books:
    if book.price > 20:
        result.append(book)
```

```sql
-- SQL — declarative: you tell the database WHAT you want
SELECT * FROM books WHERE price > 20;
-- The engine decides whether to scan the table, use an index, etc.
```

Ini adalah pergeseran cara berpikir yang fundamental. Kamu tidak menulis loop di SQL. Kamu mendeskripsikan kondisi, dan query planner yang memilih strategi eksekusi optimal.

### Arsitektur Client-Server

PostgreSQL menggunakan model **client-server**:

1. **Client** (aplikasimu, psql, pgAdmin) mengirim query SQL sebagai teks melalui koneksi jaringan
2. **Server** menerima query dan:
   - **Mem-parse** (memeriksa sintaks, memvalidasi nama table/kolom)
   - **Merencanakan** (query planner mengevaluasi berbagai strategi eksekusi dan memilih yang paling murah)
   - **Mengeksekusi** rencana yang dipilih (membaca/menulis data di disk atau memori)
3. **Server** mengembalikan **result set** (baris dan kolom) ke client

Artinya:
- Banyak client bisa terhubung ke database yang sama secara bersamaan
- Server yang menangani concurrency, locking, dan caching — bukan kamu
- Network round-trip itu penting: mengirim 1000 INSERT individual jauh lebih lambat daripada satu INSERT dengan 1000 baris

### Hirarki Objek PostgreSQL

PostgreSQL mengorganisasi objek dalam sebuah hirarki:

```
Cluster (one PostgreSQL server instance)
 └── Database (e.g., sqlbook, myapp_production)
      └── Schema (e.g., public — the default schema)
           └── Table (e.g., books, authors)
                ├── Columns
                ├── Indexes
                ├── Constraints
                └── Triggers
```

- Sebuah **cluster** adalah satu server PostgreSQL yang berjalan. Bisa menampung banyak database.
- Sebuah **database** adalah container yang terisolasi. Query lintas database tidak bisa dilakukan di SQL biasa.
- Sebuah **schema** adalah namespace di dalam database. Schema default adalah `public`. Schema memungkinkan kamu mengorganisasi table (misalnya, `auth.users`, `billing.invoices`) tanpa bentrok nama.
- Sebuah **table** berada di dalam schema dan menyimpan data sebenarnya.

Ketika kamu menulis `SELECT * FROM books`, PostgreSQL sebenarnya menerjemahkannya menjadi `SELECT * FROM public.books`.

### Terhubung ke PostgreSQL

```sql
-- Connect via psql
psql -h localhost -U postgres

-- Useful commands inside psql
\l             -- list all databases
\c sqlbook      -- connect to database named sqlbook
\dt            -- list all tables
\d tablename   -- describe a table (columns, types, constraints)
\timing        -- toggle query timing on/off
\q             -- quit

-- Create a database for this book
CREATE DATABASE sqlbook;
\c sqlbook
```

---

## 1.2 Tipe Data

Memilih tipe data yang tepat penting untuk kebenaran, penyimpanan, dan performa.

### Tipe Numerik

```sql
-- Integers (choose based on the range you need)
SMALLINT   -- 2 bytes, range: -32,768 to 32,767
INTEGER    -- 4 bytes, range: -2.1 billion to 2.1 billion (use for most IDs)
BIGINT     -- 8 bytes, range: -9.2 quintillion to 9.2 quintillion

-- Auto-incrementing primary key (SQL standard, PostgreSQL 10+)
id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY
id BIGINT  GENERATED ALWAYS AS IDENTITY PRIMARY KEY

-- SERIAL is the older way (still works, but IDENTITY is preferred)
id SERIAL PRIMARY KEY   -- same as INTEGER GENERATED BY DEFAULT AS IDENTITY

-- Exact decimals — ALWAYS use for money
NUMERIC(10, 2)  -- 10 digits total, 2 after decimal point
DECIMAL(10, 2)  -- identical to NUMERIC

-- Floating point — NEVER use for money (imprecise)
REAL            -- 4 bytes, ~6 significant digits
DOUBLE PRECISION -- 8 bytes, ~15 significant digits

-- Why floating point is wrong for money:
SELECT 0.1::FLOAT + 0.2::FLOAT;       -- 0.30000000000000004 (wrong)
SELECT 0.1::NUMERIC + 0.2::NUMERIC;   -- 0.3 (correct)
```

### Tipe Teks

```sql
TEXT        -- unlimited length, no performance penalty vs VARCHAR
VARCHAR(n)  -- up to n characters (use when you want DB-enforced length limit)
CHAR(n)     -- fixed length, pads with spaces (rarely useful)

-- In PostgreSQL, TEXT and VARCHAR have identical performance.
-- Use TEXT by default. Use VARCHAR(n) only to enforce a maximum length.
```

### Boolean

```sql
BOOLEAN  -- stores TRUE, FALSE, or NULL

-- PostgreSQL accepts many input formats:
TRUE  / FALSE
'true' / 'false'
'yes'  / 'no'
'1'    / '0'
'on'   / 'off'
```

### Tanggal dan Waktu

```sql
DATE         -- date only:     '2024-01-15'
TIME         -- time only:     '14:30:00'
TIMESTAMP    -- without timezone: '2024-01-15 14:30:00'
TIMESTAMPTZ  -- WITH timezone (stores as UTC, displays in session tz)
INTERVAL     -- duration: '3 days', '2 hours 30 minutes', '1 year'

-- ALWAYS use TIMESTAMPTZ for created_at, updated_at, and any event time.
-- TIMESTAMP has no timezone info — if your server ever changes timezone,
-- your timestamps become ambiguous.

-- TIMESTAMPTZ example:
SET timezone = 'America/New_York';
SELECT NOW();  -- 2024-01-15 09:30:00-05

SET timezone = 'Asia/Tokyo';
SELECT NOW();  -- 2024-01-15 23:30:00+09 (same moment, different display)
```

### UUID

```sql
UUID  -- 128-bit unique identifier: 'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11'
SELECT gen_random_uuid();  -- generate a new UUID (PostgreSQL 13+)

-- When to use UUID vs INTEGER:
-- INTEGER IDENTITY: simpler, smaller, reveals insertion order
-- UUID: globally unique across systems, good for distributed apps,
--       IDs can be generated by the app before inserting
```

### JSONB

```sql
JSONB  -- binary JSON (supports indexing and operators, preferred over JSON)
JSON   -- text JSON (stores exactly as given, slower for queries)

-- Basic JSONB operations:
SELECT '{"name": "Alice", "age": 30}'::JSONB;

-- -> returns JSONB value
SELECT '{"name": "Alice"}'::JSONB -> 'name';     -- "Alice" (JSONB)

-- ->> returns TEXT value
SELECT '{"name": "Alice"}'::JSONB ->> 'name';    -- Alice (TEXT)

-- Nested access
SELECT '{"address": {"city": "London"}}'::JSONB -> 'address' ->> 'city';  -- London

-- @> contains operator
SELECT '{"name": "Alice", "age": 30}'::JSONB @> '{"name": "Alice"}';  -- true

-- ? key exists
SELECT '{"name": "Alice"}'::JSONB ? 'name';  -- true
```

### Referensi Cepat

| Kasus Penggunaan | Tipe yang Direkomendasikan |
|---|---|
| Primary key (kebanyakan table) | `INTEGER GENERATED ALWAYS AS IDENTITY` |
| Primary key (volume tinggi) | `BIGINT GENERATED ALWAYS AS IDENTITY` |
| Primary key (terdistribusi) | `UUID DEFAULT gen_random_uuid()` |
| Teks dengan panjang maksimal | `VARCHAR(n)` |
| Teks lainnya | `TEXT` |
| Uang / harga | `NUMERIC(10,2)` |
| Flag dan toggle | `BOOLEAN NOT NULL DEFAULT false` |
| Timestamp event | `TIMESTAMPTZ NOT NULL DEFAULT NOW()` |
| Tanggal tanpa waktu (ulang tahun) | `DATE` |
| Data fleksibel per baris | `JSONB` |

---

## 1.3 Constraint (Batasan)

Constraint adalah aturan yang ditegakkan oleh database pada datamu. Selalu taruh constraint
di database — validasi di level aplikasi saja tidak cukup. Aplikasi punya bug;
database adalah garis pertahanan terakhir.

### NOT NULL

```sql
-- NULL means "unknown" or "not applicable"
-- It is not zero, not an empty string, not false
-- Any column without NOT NULL can contain NULL

CREATE TABLE users (
    id    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email TEXT NOT NULL,   -- required — cannot be NULL
    phone TEXT             -- optional — NULL is allowed
);

-- NULL behaves unexpectedly with comparisons:
SELECT NULL = NULL;     -- NULL  (not TRUE)
SELECT NULL <> NULL;    -- NULL  (not FALSE either)
SELECT NULL IS NULL;    -- TRUE  (correct way to check for NULL)
SELECT NULL IS NOT NULL; -- FALSE

-- COALESCE returns the first non-NULL argument
SELECT COALESCE(phone, 'no phone') FROM users;

-- NULLIF returns NULL if two values are equal
SELECT NULLIF(discount, 0);  -- returns NULL when discount is 0
```

### PRIMARY KEY

```sql
-- Uniquely identifies each row. Enforces NOT NULL + UNIQUE.
-- Every table should have exactly one primary key.

CREATE TABLE orders (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY
);

-- Composite primary key: the combination of columns must be unique
CREATE TABLE order_items (
    order_id   INTEGER NOT NULL,
    product_id INTEGER NOT NULL,
    quantity   INTEGER NOT NULL CHECK (quantity > 0),
    PRIMARY KEY (order_id, product_id)  -- each product appears once per order
);
```

### UNIQUE

```sql
-- One or more columns must contain unique values (NULLs are exempt)

CREATE TABLE users (
    id       INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email    TEXT NOT NULL UNIQUE,
    username TEXT NOT NULL,
    UNIQUE (username)  -- same effect, table-level syntax
);

-- Composite unique: combination of columns must be unique
CREATE TABLE team_members (
    team_id INTEGER NOT NULL,
    user_id INTEGER NOT NULL,
    role    TEXT NOT NULL,
    UNIQUE (team_id, user_id)  -- a user can only be in a team once
);
```

### CHECK

```sql
CREATE TABLE products (
    id           INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name         TEXT NOT NULL,
    price        NUMERIC(10,2) NOT NULL CHECK (price >= 0),
    stock        INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    discount_pct INTEGER CHECK (discount_pct BETWEEN 0 AND 100),
    status       TEXT NOT NULL DEFAULT 'active'
                 CHECK (status IN ('active', 'inactive', 'draft'))
);

-- Named constraint (easier to identify in error messages)
ALTER TABLE products
    ADD CONSTRAINT chk_price_positive CHECK (price > 0);
```

### FOREIGN KEY

```sql
-- Ensures a value in one table exists as a primary key in another table.
-- This is how referential integrity is enforced.

CREATE TABLE categories (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);

CREATE TABLE products (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    category_id INTEGER NOT NULL REFERENCES categories(id),
    name        TEXT NOT NULL
);

-- What happens when the referenced row is deleted:
-- ON DELETE RESTRICT    (default) block the delete if it is referenced
-- ON DELETE CASCADE     delete child rows automatically
-- ON DELETE SET NULL    set the foreign key column to NULL
-- ON DELETE SET DEFAULT set the foreign key column to its default value

-- Example with CASCADE: deleting an order deletes its items
CREATE TABLE order_items (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id   INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES products(id),
    quantity   INTEGER NOT NULL
);
```

### DEFAULT

```sql
-- Value used when the column is not specified in an INSERT statement

CREATE TABLE posts (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title      TEXT NOT NULL,
    published  BOOLEAN NOT NULL DEFAULT false,
    view_count INTEGER NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Inserting without specifying defaults:
INSERT INTO posts (title) VALUES ('My First Post');
-- published = false, view_count = 0, created_at = current timestamp
```

---

## 1.4 Normalisasi

Normalisasi adalah proses menyusun struktur database untuk mengurangi redundansi
dan memastikan integritas data. Setiap "normal form" (bentuk normal) memperbaiki satu kategori masalah.

### Kenapa Normalisasi Penting — Masalahnya

Bayangkan menyimpan order pelanggan dalam satu table datar:

```
order_id | customer_name | customer_email  | product_names      | product_prices
---------|---------------|-----------------|--------------------|---------------
1        | Alice Smith   | alice@email.com | "Book, Pen"        | "29.99, 1.99"
2        | Alice Smith   | alice@email.com | "Notebook"         | "9.99"
3        | Bob Jones     | bob@email.com   | "Book, Pen, Laptop"| "29.99,1.99,999"
```

Ini punya masalah serius:
1. Email Alice disimpan dua kali — update satu baris dan terlewat yang lain, data jadi tidak konsisten
2. Nama produk dan harga ada di string yang dipisah koma — tidak bisa di-query dengan bersih
3. Tidak bisa menjawab "semua order yang mengandung Book" dengan query sederhana
4. Tidak bisa mengupdate harga produk tanpa mengupdate setiap order yang mengandungnya

### First Normal Form (1NF)

**Aturan:** Setiap kolom berisi satu nilai atomik (tidak bisa dipecah). Tidak ada grup yang berulang.

```sql
-- VIOLATES 1NF: multiple values crammed into one column
CREATE TABLE orders_bad (
    id            INTEGER PRIMARY KEY,
    customer_name TEXT,
    product_names TEXT   -- "Book, Pen, Notebook" violates atomicity
);

-- SATISFIES 1NF: one value per cell, line items in a separate table
CREATE TABLE orders (
    id            INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_name TEXT NOT NULL
);

CREATE TABLE order_items (
    id           INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id     INTEGER NOT NULL REFERENCES orders(id),
    product_name TEXT NOT NULL,
    price        NUMERIC(10,2) NOT NULL
);
```

### Second Normal Form (2NF)

**Aturan:** Harus memenuhi 1NF. Setiap kolom non-key harus bergantung pada seluruh primary key
(bukan hanya sebagian). Ini hanya berlaku untuk table dengan composite primary key.

```sql
-- VIOLATES 2NF: product_name depends only on product_id, not the full key
CREATE TABLE order_items_bad (
    order_id     INTEGER,
    product_id   INTEGER,
    product_name TEXT,    -- depends on product_id alone (partial dependency)
    quantity     INTEGER,
    PRIMARY KEY (order_id, product_id)
);

-- SATISFIES 2NF: product_name belongs in its own table
CREATE TABLE products (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL
);

CREATE TABLE order_items (
    order_id   INTEGER NOT NULL,
    product_id INTEGER NOT NULL REFERENCES products(id),
    quantity   INTEGER NOT NULL,
    PRIMARY KEY (order_id, product_id)
    -- quantity depends on BOTH order_id AND product_id
);
```

### Third Normal Form (3NF)

**Aturan:** Harus memenuhi 2NF. Tidak ada kolom non-key yang bergantung pada kolom non-key lainnya.

```sql
-- VIOLATES 3NF: city and state are determined by zip_code, not by the primary key
CREATE TABLE orders_bad (
    id       INTEGER PRIMARY KEY,
    customer TEXT,
    zip_code TEXT,
    city     TEXT,   -- transitive: id -> zip_code -> city
    state    TEXT    -- transitive: id -> zip_code -> state
);

-- SATISFIES 3NF: zip_code data in its own table
CREATE TABLE zip_codes (
    zip_code TEXT PRIMARY KEY,
    city     TEXT NOT NULL,
    state    TEXT NOT NULL
);

CREATE TABLE orders (
    id       INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer TEXT NOT NULL,
    zip_code TEXT REFERENCES zip_codes(zip_code)
);
```

### Sistem Order yang Sepenuhnya Ternormalisasi

```sql
CREATE TABLE customers (
    id    INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name  TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE
);

CREATE TABLE categories (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);

CREATE TABLE products (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    category_id INTEGER NOT NULL REFERENCES categories(id),
    name        TEXT NOT NULL,
    price       NUMERIC(10,2) NOT NULL CHECK (price > 0)
);

CREATE TABLE orders (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer_id INTEGER NOT NULL REFERENCES customers(id),
    order_date  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE order_items (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id   INTEGER NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
    product_id INTEGER NOT NULL REFERENCES products(id),
    quantity   INTEGER NOT NULL CHECK (quantity > 0),
    unit_price NUMERIC(10,2) NOT NULL  -- snapshot of price at time of order
);
```

### Kapan Harus Melanggar Normalisasi

Terkadang kamu sengaja melakukan denormalisasi demi performa:

- `orders.total` — simpan total yang sudah dihitung supaya tidak perlu menjumlahkan setiap kali baca
- `order_items.unit_price` — simpan harga pada saat order (harga bisa berubah)
- Table pelaporan — didenormalisasi untuk performa query analytics

**Aturan:** Normalisasi dulu. Denormalisasi hanya ketika kamu sudah mengukur ada masalah performa yang nyata.

---

## 1.5 Diagram ER

Diagram Entity-Relationship (ER) memetakan schema kamu secara visual. Gambar dulu sebelum menulis SQL apapun.

### Membaca Notasi

```
customers  ||--o{  orders       : "places"
orders     ||--|{  order_items  : "contains"
products   ||--o{  order_items  : "included in"
categories ||--o{  products     : "categorizes"
```

| Simbol | Arti |
|---|---|
| `\|\|` | tepat satu |
| `\|{` | satu atau lebih |
| `o{` | nol atau lebih |
| `o\|` | nol atau satu |

`customers ||--o{ orders` dibaca sebagai:
"satu customer menempatkan nol atau lebih order; setiap order dimiliki tepat satu customer."

### Proses Desain

1. Buat daftar **entitas** (kata benda dari requirements): User, Product, Order
2. Buat daftar **hubungan** (kata kerja): User *menempatkan* Order, Order *berisi* Products
3. Tentukan **kardinalitas** untuk setiap hubungan: one-to-many, many-to-many
4. Foreign key selalu ada di sisi **many** dari hubungan one-to-many
5. Hubungan many-to-many membutuhkan **junction table** (table penghubung)

### Contoh Lengkap: Schema Buku dan Penulis

Mari kita telusuri proses mendesain schema yang kita gunakan untuk latihan di buku ini.

**Langkah 1 — Identifikasi entitas:** Kita perlu menyimpan informasi tentang *penulis* dan *buku*.

**Langkah 2 — Identifikasi hubungan:** Seorang penulis *menulis* buku. Sebuah buku *ditulis oleh* satu penulis.

**Langkah 3 — Tentukan kardinalitas:** Satu penulis bisa menulis banyak buku. Setiap buku punya tepat satu penulis. Ini adalah hubungan **one-to-many**.

**Langkah 4 — Tempatkan foreign key:** Foreign key ditaruh di sisi "many" — table `books` mendapat kolom `author_id` yang menunjuk ke `authors.id`.

**Langkah 5 — Gambar:**

```
+--------------------+          +-----------------------------+
|      authors       |          |           books             |
+--------------------+          +-----------------------------+
| id          (PK)   |---||--o{-| id              (PK)        |
| name               |          | author_id       (FK -> authors.id) |
| birth_year         |          | title                       |
| country            |          | published_year              |
+--------------------+          | price                       |
                                | genre                       |
                                | in_stock                    |
                                +-----------------------------+

authors  ||--o{  books : "writes"

Dibaca: Satu penulis menulis nol atau lebih buku.
        Setiap buku dimiliki tepat satu penulis.
```

### Tools untuk Menggambar Diagram ER

Meskipun pena dan kertas cukup untuk schema kecil, tools ini membantu untuk desain yang lebih besar:

- **dbdiagram.io** — gratis, berbasis browser, menggunakan DSL sederhana untuk mendefinisikan table dan hubungan. Bagus untuk diagram cepat.
- **draw.io (diagrams.net)** — gratis, diagramming umum. Punya template bentuk database.
- **pgModeler** — open-source, khusus PostgreSQL. Bisa membuat SQL dari diagram dan merekayasa balik database yang sudah ada.

---

## Fase 1 — Schema Latihan

Gunakan schema ini untuk latihan di sepanjang buku:

```sql
CREATE TABLE authors (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       TEXT NOT NULL,
    birth_year INTEGER,
    country    TEXT NOT NULL
);

CREATE TABLE books (
    id             INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    author_id      INTEGER NOT NULL REFERENCES authors(id),
    title          TEXT NOT NULL,
    published_year INTEGER,
    price          NUMERIC(8,2) NOT NULL CHECK (price > 0),
    genre          TEXT NOT NULL,
    in_stock       BOOLEAN NOT NULL DEFAULT true
);

INSERT INTO authors (name, birth_year, country) VALUES
    ('George Orwell',          1903, 'UK'),
    ('J.K. Rowling',           1965, 'UK'),
    ('Haruki Murakami',        1949, 'Japan'),
    ('Chimamanda Adichie',     1977, 'Nigeria'),
    ('Gabriel Garcia Marquez', 1927, 'Colombia');

INSERT INTO books (author_id, title, published_year, price, genre, in_stock) VALUES
    (1, '1984',                                       1949, 12.99, 'Dystopian',          true),
    (1, 'Animal Farm',                                1945,  9.99, 'Satire',             true),
    (2, 'Harry Potter and the Sorcerers Stone',       1997, 14.99, 'Fantasy',            true),
    (2, 'Harry Potter and the Chamber of Secrets',    1998, 14.99, 'Fantasy',            false),
    (3, 'Norwegian Wood',                             1987, 13.99, 'Literary Fiction',   true),
    (3, 'Kafka on the Shore',                         2002, 15.99, 'Magical Realism',    true),
    (4, 'Purple Hibiscus',                            2003, 11.99, 'Literary Fiction',   true),
    (4, 'Half of a Yellow Sun',                       2006, 13.99, 'Historical Fiction', false),
    (5, 'One Hundred Years of Solitude',              1967, 14.99, 'Magical Realism',    true),
    (5, 'Love in the Time of Cholera',                1985, 12.99, 'Romance',            true);
```

## Fase 1 — Latihan

1. Tambahkan constraint `CHECK` yang mencegah `published_year` di masa depan.
2. Kolom `genre` adalah teks bebas — masalah apa yang ditimbulkannya? Bagaimana kamu memperbaiki schema-nya?
3. Gambar diagram ER sederhana yang menunjukkan hubungan antara authors dan books.
4. Tambahkan `updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()` ke kedua table menggunakan `ALTER TABLE`.
5. Apakah schema ini sudah dalam 3NF? Identifikasi transitive dependency yang ada.

---

# Fase 2 — Dasar-Dasar SQL

## 2.1 SELECT — Membaca Data

`SELECT` mendefinisikan **bentuk** output-mu — kolom mana yang muncul dan dalam bentuk apa. Setiap ekspresi dalam daftar SELECT dievaluasi sekali per baris dalam hasil. Kamu bisa menganggap SELECT sebagai transformasi: ia mengambil baris dari table dan menghasilkan baris output dengan kolom yang kamu tentukan.

**Kenapa `SELECT *` buruk di production:**
- Jika seseorang menambah atau menghapus kolom dari table, kode aplikasimu bisa rusak karena mengharapkan urutan atau jumlah kolom tertentu.
- Ia menarik semua kolom, termasuk field TEXT atau JSONB yang besar yang mungkin tidak kamu butuhkan, membuang memori dan bandwidth jaringan.
- Ia membuat maksud query-mu tidak jelas — pembaca tidak bisa tahu kolom mana yang sebenarnya kamu gunakan.

Selalu sebutkan kolom yang kamu butuhkan: `SELECT id, title, price FROM books` bukan `SELECT * FROM books`.

### DISTINCT

`DISTINCT` menghapus baris duplikat dari result set-mu:

```sql
-- See all unique genres in the books table
SELECT DISTINCT genre FROM books ORDER BY genre;

-- DISTINCT applies to the entire row, not just the first column
SELECT DISTINCT genre, in_stock FROM books ORDER BY genre;
-- Returns each unique (genre, in_stock) combination

-- COUNT(DISTINCT ...) counts unique values
SELECT COUNT(DISTINCT genre) AS unique_genres FROM books;
```

### Contoh SELECT Dasar

```sql
-- Select all columns (avoid SELECT * in production — it breaks when columns are added)
SELECT * FROM books;

-- Select specific columns
SELECT title, price FROM books;

-- Column aliases — rename for output
SELECT
    title       AS book_title,
    price       AS cost,
    in_stock    AS available
FROM books;

-- Expressions in SELECT
SELECT
    title,
    price,
    price * 1.1                             AS price_with_10_pct_tax,
    UPPER(title)                            AS title_upper,
    LENGTH(title)                           AS title_char_count,
    ROUND(price * 0.9, 2)                   AS discounted_price
FROM books;

-- Computed values with no table (useful for testing)
SELECT 2 + 2 AS result;
SELECT NOW() AS current_time;
SELECT UPPER('hello') AS uppercased;
```

---

## 2.2 WHERE — Memfilter Baris

```sql
-- Comparison operators
SELECT * FROM books WHERE price > 13.00;
SELECT * FROM books WHERE price >= 13.00;
SELECT * FROM books WHERE price < 10.00;
SELECT * FROM books WHERE price <= 10.00;
SELECT * FROM books WHERE price = 12.99;
SELECT * FROM books WHERE price <> 12.99;  -- <> means "not equal" (same as !=)

-- Text comparison (case-sensitive)
SELECT * FROM books WHERE genre = 'Fantasy';
SELECT * FROM books WHERE genre <> 'Romance';

-- Combining conditions
SELECT * FROM books WHERE genre = 'Fantasy' AND price < 15.00;
SELECT * FROM books WHERE genre = 'Fantasy' OR genre = 'Satire';
SELECT * FROM books WHERE NOT in_stock;
SELECT * FROM books WHERE price > 10 AND price < 14 AND in_stock = true;

-- BETWEEN (inclusive on both ends)
SELECT * FROM books WHERE price BETWEEN 11.00 AND 14.00;
SELECT * FROM books WHERE published_year BETWEEN 1980 AND 2010;

-- IN (check against a list of values)
SELECT * FROM books WHERE genre IN ('Fantasy', 'Satire', 'Dystopian');
SELECT * FROM books WHERE author_id IN (1, 3);

-- NOT IN
SELECT * FROM books WHERE genre NOT IN ('Romance', 'Historical Fiction');

-- LIKE (pattern matching — case-sensitive in PostgreSQL)
SELECT * FROM books WHERE title LIKE 'Harry%';    -- starts with "Harry"
SELECT * FROM books WHERE title LIKE '%and%';     -- contains "and"
SELECT * FROM books WHERE title LIKE '_984';      -- _ matches exactly one character
SELECT * FROM books WHERE title LIKE '%Stone';    -- ends with "Stone"

-- ILIKE (case-insensitive — PostgreSQL specific)
SELECT * FROM books WHERE title ILIKE '%harry%';
SELECT * FROM books WHERE genre ILIKE 'fantasy';

-- NULL handling — always use IS NULL, never = NULL
SELECT * FROM books WHERE published_year IS NULL;
SELECT * FROM books WHERE published_year IS NOT NULL;

-- Why = NULL does not work:
SELECT * FROM books WHERE published_year = NULL;   -- returns 0 rows (always)
-- Because NULL = NULL evaluates to NULL, not TRUE

-- COALESCE: return first non-NULL value
SELECT title, COALESCE(published_year::TEXT, 'unknown') AS year
FROM books;
```

---

## 2.3 ORDER BY, LIMIT, OFFSET

```sql
-- ORDER BY: sort results (ASC is default)
SELECT * FROM books ORDER BY price;            -- cheapest first
SELECT * FROM books ORDER BY price DESC;       -- most expensive first
SELECT * FROM books ORDER BY genre, price;     -- sort by genre, then price within genre
SELECT * FROM books ORDER BY genre ASC, price DESC;

-- NULLs: sort LAST in ASC (default), FIRST in DESC
SELECT * FROM books ORDER BY published_year;            -- NULLs at end
SELECT * FROM books ORDER BY published_year NULLS LAST; -- explicit
SELECT * FROM books ORDER BY published_year NULLS FIRST;-- NULLs at start

-- LIMIT: maximum number of rows to return
SELECT * FROM books ORDER BY price DESC LIMIT 5;        -- top 5 most expensive

-- OFFSET: skip N rows before returning (used for pagination)
SELECT * FROM books ORDER BY id LIMIT 3 OFFSET 0;  -- page 1 (rows 1-3)
SELECT * FROM books ORDER BY id LIMIT 3 OFFSET 3;  -- page 2 (rows 4-6)
SELECT * FROM books ORDER BY id LIMIT 3 OFFSET 6;  -- page 3 (rows 7-9)

-- Pagination formula: OFFSET = (page - 1) * page_size
-- page 1: OFFSET = (1-1) * 3 = 0
-- page 2: OFFSET = (2-1) * 3 = 3
-- page 3: OFFSET = (3-1) * 3 = 6
```

### Kenapa OFFSET Lambat di Table Besar

`OFFSET n` tidak langsung melompat ke baris ke-n. Database harus **memindai dan membuang** n baris sebelum mengembalikan yang kamu minta. Artinya:
- `OFFSET 10` memindai 10 baris dan membuangnya — cukup cepat
- `OFFSET 100000` memindai 100.000 baris dan membuangnya — O(n) dan sangat lambat

Semakin dalam kamu mem-paginasi result set, semakin lambat setiap halaman.

### Keyset Pagination (Cara yang Lebih Baik)

Daripada memberitahu database berapa baris yang harus dilewati, beritahu **di mana kamu terakhir berhenti**:

```sql
-- First page: no WHERE filter needed, just LIMIT
SELECT id, title, price
FROM books
ORDER BY id
LIMIT 20;
-- Returns rows with id: 1, 2, 3, ... 20

-- Subsequent pages: use the last id from the previous page
-- If the last row on page 1 had id = 20:
SELECT id, title, price
FROM books
WHERE id > 20
ORDER BY id
LIMIT 20;
-- Returns rows with id: 21, 22, 23, ... 40

-- Next page (last id was 40):
SELECT id, title, price
FROM books
WHERE id > 40
ORDER BY id
LIMIT 20;
```

Ini **O(log n)** bukan O(n) karena PostgreSQL menggunakan index pada `id` untuk langsung melompat ke titik awal. Halaman ke-500 sama cepatnya dengan halaman ke-1.

**Trade-off:** keyset pagination tidak mendukung "loncat ke halaman 47" — kamu hanya bisa maju (atau mundur dengan `<`). Untuk kebanyakan API dan UI infinite-scroll, ini tidak masalah.

---

## 2.4 Fungsi Aggregate

Fungsi aggregate menciutkan banyak baris menjadi satu nilai hasil.

```sql
-- COUNT
SELECT COUNT(*) AS total_books FROM books;              -- all rows
SELECT COUNT(published_year) AS with_year FROM books;   -- non-NULL rows only
SELECT COUNT(DISTINCT genre) AS unique_genres FROM books; -- distinct values

-- SUM, AVG, MIN, MAX
SELECT SUM(price)  AS total_value   FROM books;
SELECT AVG(price)  AS average_price FROM books;
SELECT MIN(price)  AS cheapest      FROM books;
SELECT MAX(price)  AS most_expensive FROM books;

-- Rounding AVG
SELECT ROUND(AVG(price), 2) AS avg_price FROM books;

-- All together in one query
SELECT
    COUNT(*)                    AS total_books,
    COUNT(DISTINCT genre)       AS unique_genres,
    COUNT(DISTINCT author_id)   AS total_authors,
    ROUND(AVG(price), 2)        AS avg_price,
    MIN(price)                  AS cheapest,
    MAX(price)                  AS most_expensive,
    SUM(price)                  AS total_stock_value
FROM books
WHERE in_stock = true;
```

---

## 2.5 GROUP BY dan HAVING

`GROUP BY` mengelompokkan baris dengan nilai yang sama dan menerapkan aggregate per kelompok.

```sql
-- Count books per genre
SELECT
    genre,
    COUNT(*) AS book_count
FROM books
GROUP BY genre
ORDER BY book_count DESC;

-- Statistics per genre
SELECT
    genre,
    COUNT(*)              AS book_count,
    ROUND(AVG(price), 2)  AS avg_price,
    MIN(price)            AS min_price,
    MAX(price)            AS max_price
FROM books
GROUP BY genre
ORDER BY avg_price DESC;

-- Group by multiple columns
SELECT
    genre,
    in_stock,
    COUNT(*) AS count
FROM books
GROUP BY genre, in_stock
ORDER BY genre, in_stock;

-- HAVING: filter groups after aggregation
-- (WHERE filters rows before grouping; HAVING filters groups after)

-- Only genres with more than 1 book
SELECT genre, COUNT(*) AS book_count
FROM books
GROUP BY genre
HAVING COUNT(*) > 1;

-- Authors whose books average more than $13
SELECT author_id, ROUND(AVG(price), 2) AS avg_price
FROM books
GROUP BY author_id
HAVING AVG(price) > 13
ORDER BY avg_price DESC;

-- You CANNOT use aliases from SELECT in HAVING
-- This is wrong:
SELECT genre, COUNT(*) AS cnt FROM books GROUP BY genre HAVING cnt > 1;
-- This is correct:
SELECT genre, COUNT(*) AS cnt FROM books GROUP BY genre HAVING COUNT(*) > 1;
```

**Urutan eksekusi statement SELECT:**
```
1. FROM        -- which table(s)?
2. WHERE       -- filter individual rows
3. GROUP BY    -- group remaining rows
4. HAVING      -- filter groups
5. SELECT      -- compute output columns
6. ORDER BY    -- sort the output
7. LIMIT/OFFSET -- truncate output
```

---

## 2.6 JOIN

JOIN menggabungkan baris dari dua atau lebih table. Ini adalah skill SQL
yang paling penting — semua yang lain dibangun di atasnya.

**Model mentalnya:** untuk setiap baris di table kiri, cari baris yang cocok
di table kanan menggunakan kondisi ON.

### INNER JOIN

Mengembalikan baris yang punya kecocokan di KEDUA table. Baris tanpa kecocokan dikecualikan.

```sql
-- Get each book with its author's name
SELECT
    b.title,
    b.price,
    a.name    AS author_name,
    a.country AS author_country
FROM books b
INNER JOIN authors a ON b.author_id = a.id;

-- INNER JOIN is the default — JOIN means INNER JOIN
SELECT b.title, a.name
FROM books b
JOIN authors a ON b.author_id = a.id;

-- Add filtering after the join
SELECT b.title, a.name
FROM books b
JOIN authors a ON b.author_id = a.id
WHERE a.country = 'UK'
  AND b.price < 15.00
ORDER BY b.price DESC;
```

### LEFT JOIN

Mengembalikan SEMUA baris dari table kiri, plus baris yang cocok dari table kanan.
Jika tidak ada kecocokan, kolom table kanan bernilai NULL.

```sql
-- All authors, with their books (authors with no books appear with NULL book fields)
SELECT
    a.name     AS author,
    b.title    AS book_title,
    b.price
FROM authors a
LEFT JOIN books b ON a.id = b.author_id
ORDER BY a.name;

-- Find authors who have NO books at all
SELECT a.name
FROM authors a
LEFT JOIN books b ON a.id = b.author_id
WHERE b.id IS NULL;
-- b.id IS NULL means no matching book was found
```

### RIGHT JOIN

Mengembalikan SEMUA baris dari table kanan. Jarang dipakai — kamu selalu bisa
menulis ulang RIGHT JOIN sebagai LEFT JOIN dengan menukar urutan table.

```sql
-- These two queries return identical results:
SELECT a.name, b.title FROM books b RIGHT JOIN authors a ON b.author_id = a.id;
SELECT a.name, b.title FROM authors a LEFT JOIN books b ON a.id = b.author_id;
-- Prefer LEFT JOIN for consistency
```

### FULL OUTER JOIN

Mengembalikan SEMUA baris dari KEDUA table, dengan NULL di mana tidak ada kecocokan.

```sql
SELECT a.name, b.title
FROM authors a
FULL OUTER JOIN books b ON a.id = b.author_id;
-- Shows: all authors (with or without books) + all books (with or without authors)
```

### CROSS JOIN

Mengembalikan setiap kombinasi baris dari kedua table (produk kartesian).
Jarang disengaja.

```sql
-- 5 authors x 10 books = 50 rows (every pairing)
SELECT a.name, b.title FROM authors a CROSS JOIN books b;
```

### SELF JOIN

Sebuah table di-join ke dirinya sendiri. Digunakan untuk data hierarkis atau perbandingan dalam satu table.

```sql
-- Find all pairs of books by the same author
SELECT
    b1.title AS book1,
    b2.title AS book2,
    a.name   AS author
FROM books b1
JOIN books b2    ON b1.author_id = b2.author_id
                AND b1.id < b2.id   -- avoid duplicates and self-pairs
JOIN authors a   ON b1.author_id = a.id
ORDER BY a.name;

-- Employees and their managers (classic self-join)
CREATE TABLE employees (
    id         INTEGER PRIMARY KEY,
    name       TEXT NOT NULL,
    manager_id INTEGER REFERENCES employees(id)  -- references the same table
);

SELECT
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;  -- LEFT because CEO has no manager
```

### Meng-join Tiga Table atau Lebih

```sql
-- Books with author name and genre (simple 2-table join with filter)
SELECT
    b.title,
    a.name     AS author,
    b.genre,
    b.price,
    b.in_stock
FROM books b
JOIN authors a ON b.author_id = a.id
WHERE b.in_stock = true
ORDER BY a.name, b.title;

-- Orders with customer name, product name, and line totals (4 tables)
SELECT
    o.id                                  AS order_id,
    c.name                                AS customer,
    p.name                                AS product,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price           AS line_total
FROM orders o
JOIN customers c    ON o.customer_id = c.id
JOIN order_items oi ON oi.order_id   = o.id
JOIN products p     ON oi.product_id = p.id
ORDER BY o.id, p.name;
```

---

## 2.7 INSERT, UPDATE, DELETE, UPSERT

### INSERT

```sql
-- Single row, named columns (preferred — order-independent)
INSERT INTO authors (name, birth_year, country)
VALUES ('Toni Morrison', 1931, 'USA');

-- Multiple rows in one statement
INSERT INTO authors (name, birth_year, country) VALUES
    ('Leo Tolstoy',    1828, 'Russia'),
    ('Virginia Woolf', 1882, 'UK'),
    ('Franz Kafka',    1883, 'Czech Republic');

-- Insert from a SELECT query
INSERT INTO bestsellers (title, author_id, price)
SELECT title, author_id, price
FROM books
WHERE price > 14.00;

-- RETURNING: get back generated values after insert
INSERT INTO authors (name, country)
VALUES ('Fyodor Dostoevsky', 'Russia')
RETURNING id, name;
-- Returns the new row's id and name
```

### UPDATE

```sql
-- Update one column for one row
UPDATE books SET price = 11.99 WHERE id = 1;

-- Update multiple columns
UPDATE books
SET
    price    = 11.99,
    in_stock = true
WHERE id = 1;

-- Update using a calculation
UPDATE books SET price = price * 1.05;   -- 5% price increase on ALL books
UPDATE books SET price = price * 1.05 WHERE genre = 'Fantasy';

-- Update based on another table
UPDATE books
SET price = price * 1.10
WHERE author_id IN (
    SELECT id FROM authors WHERE country = 'UK'
);

-- RETURNING: see what changed
UPDATE books
SET price = ROUND(price * 1.05, 2)
WHERE genre = 'Dystopian'
RETURNING id, title, price AS new_price;
```

### DELETE

```sql
-- Delete specific rows
DELETE FROM books WHERE in_stock = false;
DELETE FROM books WHERE price < 10.00;

-- Delete rows based on a subquery
DELETE FROM books
WHERE author_id IN (
    SELECT id FROM authors WHERE country = 'Russia'
);

-- RETURNING: see what was deleted
DELETE FROM books WHERE price < 10.00
RETURNING id, title, price;

-- Delete ALL rows (keeps table structure)
DELETE FROM books;

-- TRUNCATE: faster for deleting all rows (cannot have a WHERE clause)
TRUNCATE TABLE books;
TRUNCATE TABLE books RESTART IDENTITY;  -- also resets the auto-increment counter
TRUNCATE TABLE orders CASCADE;          -- also truncates dependent tables
```

### UPSERT — Insert atau Update

```sql
-- ON CONFLICT: what to do when an INSERT would violate a unique constraint

-- Insert or do nothing (ignore duplicates)
INSERT INTO tags (name)
VALUES ('fiction')
ON CONFLICT (name) DO NOTHING;

-- Insert or update ("upsert")
INSERT INTO products (sku, name, price, stock)
VALUES ('BOOK-001', 'The Go Programming Language', 39.99, 5)
ON CONFLICT (sku) DO UPDATE
    SET
        name  = EXCLUDED.name,           -- EXCLUDED = the row we tried to insert
        price = EXCLUDED.price,
        stock = products.stock + EXCLUDED.stock;  -- add to existing stock

-- Upsert user settings
INSERT INTO user_preferences (user_id, theme, language)
VALUES (42, 'dark', 'en')
ON CONFLICT (user_id) DO UPDATE
    SET
        theme    = EXCLUDED.theme,
        language = EXCLUDED.language,
        updated_at = NOW();
```

---

## 2.8 Transaksi

Transaksi mengelompokkan beberapa statement menjadi satu unit atomik — semua berhasil atau semua gagal.

### Transaksi Dasar

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

COMMIT;
```

Jika ada yang gagal, gunakan `ROLLBACK` untuk membatalkan semua perubahan:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- oops, something went wrong
ROLLBACK;  -- nothing changed
```

### Aturan Penting

- PostgreSQL secara default auto-commit setiap statement. Tanpa `BEGIN`, setiap statement adalah transaksinya sendiri.
- Jika ada statement di dalam transaksi yang gagal, transaksi masuk ke status error. Kamu harus `ROLLBACK` sebelum menjalankan perintah lain.
- Di kode aplikasi (Go, Python, dll.), library database-mu yang menangani `BEGIN`/`COMMIT`/`ROLLBACK` untukmu — kamu cukup memanggil `db.Begin()` dan `tx.Commit()`.

### Kenapa Ini Penting

Jangan pernah menjalankan statement INSERT/UPDATE/DELETE yang berhubungan tanpa transaksi:
- Transfer uang: debit + credit harus keduanya berhasil
- Membuat order: insert order + insert line item + kurangi inventory
- Jika statement kedua gagal tanpa transaksi, datamu jadi tidak konsisten

---

## Fase 2 — Latihan

Menggunakan schema books/authors:

1. Tampilkan semua buku beserta nama penulisnya, diurutkan berdasarkan negara penulis lalu harga buku.
2. Temukan semua buku dengan harga antara $12.00 dan $15.00 yang saat ini tersedia (in stock).
3. Hitung berapa buku yang ditulis setiap penulis. Sertakan penulis dengan nol buku.
4. Temukan buku termahal di setiap genre.
5. Tampilkan semua genre yang memiliki setidaknya satu buku out-of-stock.
6. Temukan buku yang judulnya mengandung kata "of" (case-insensitive).
7. Hitung total nilai inventaris (jumlah harga) untuk buku in-stock per genre.
8. Penulis mana yang telah menulis buku di lebih dari satu genre?
9. Masukkan penulis baru dan dua bukunya. Gunakan `RETURNING` untuk menangkap ID penulis untuk insert buku.
10. Update semua harga buku: tambahkan 8% untuk buku yang diterbitkan sebelum 1970.

---

# Fase 3 — SQL Menengah

## 3.1 Subquery

Subquery adalah query yang bersarang di dalam query lain.

```sql
-- Scalar subquery: returns exactly one value
-- Books priced above the overall average
SELECT title, price
FROM books
WHERE price > (SELECT AVG(price) FROM books);

-- Subquery in SELECT: computed value per row
SELECT
    title,
    price,
    (SELECT ROUND(AVG(price), 2) FROM books) AS overall_avg,
    price - (SELECT AVG(price) FROM books)   AS diff_from_avg
FROM books
ORDER BY diff_from_avg DESC;

-- Subquery in FROM (derived table / inline view)
SELECT genre, avg_price
FROM (
    SELECT
        genre,
        ROUND(AVG(price), 2) AS avg_price
    FROM books
    GROUP BY genre
) genre_stats
WHERE avg_price > 13.00
ORDER BY avg_price DESC;

-- Subquery with IN
SELECT title
FROM books
WHERE author_id IN (
    SELECT id FROM authors WHERE country = 'UK'
);

-- Subquery with NOT IN (watch out for NULLs — see note below)
SELECT title
FROM books
WHERE author_id NOT IN (
    SELECT id FROM authors WHERE country = 'UK'
);

-- WARNING: NOT IN with a subquery that can return NULL gives unexpected results.
-- If the subquery returns any NULL, NOT IN returns no rows at all.
-- Use NOT EXISTS instead when NULLs are possible.
```

### Correlated Subquery

Correlated subquery mereferensikan kolom dari query luar.
Ia dieksekusi sekali per baris di query luar.

```sql
-- For each book, show its price vs the average price in its genre
SELECT
    b.title,
    b.genre,
    b.price,
    (
        SELECT ROUND(AVG(price), 2)
        FROM books inner_b
        WHERE inner_b.genre = b.genre   -- reference to outer query
    ) AS genre_avg_price,
    b.price - (
        SELECT AVG(price)
        FROM books inner_b
        WHERE inner_b.genre = b.genre
    ) AS diff_from_genre_avg
FROM books b
ORDER BY b.genre, diff_from_genre_avg DESC;

-- Most expensive book per author using correlated subquery
SELECT title, author_id, price
FROM books b
WHERE price = (
    SELECT MAX(price)
    FROM books
    WHERE author_id = b.author_id   -- correlates to outer b.author_id
)
ORDER BY author_id;
```

### EXISTS dan NOT EXISTS

Lebih efisien daripada IN/NOT IN untuk pengecekan keberadaan, terutama di table besar.
EXISTS berhenti di kecocokan pertama; IN memuat semua nilai sebelum memeriksa.

```sql
-- Authors who have at least one book
SELECT name
FROM authors a
WHERE EXISTS (
    SELECT 1   -- SELECT 1 is conventional — the value doesn't matter
    FROM books b
    WHERE b.author_id = a.id
);

-- Authors who have NO books
SELECT name
FROM authors a
WHERE NOT EXISTS (
    SELECT 1 FROM books b WHERE b.author_id = a.id
);

-- NOT EXISTS vs NOT IN:
-- NOT IN with NULLs in the subquery returns 0 rows (unexpected)
-- NOT EXISTS is always safe with NULLs — prefer it
```

---

## 3.2 CTE — Common Table Expression

CTE memberi nama pada subquery sehingga kamu bisa mereferensikannya seperti table. Gunakan CTE untuk
memecah query kompleks menjadi langkah-langkah yang bisa dibaca dan diberi nama.

```sql
-- Basic CTE
WITH expensive_books AS (
    SELECT * FROM books WHERE price > 13.00
)
SELECT title, price
FROM expensive_books
ORDER BY price DESC;

-- Multiple CTEs (comma-separated)
WITH
uk_authors AS (
    SELECT id, name FROM authors WHERE country = 'UK'
),
uk_books AS (
    SELECT b.title, b.price, a.name AS author
    FROM books b
    JOIN uk_authors a ON b.author_id = a.id
)
SELECT * FROM uk_books ORDER BY price DESC;

-- CTE used multiple times
WITH genre_stats AS (
    SELECT
        genre,
        COUNT(*)             AS book_count,
        ROUND(AVG(price), 2) AS avg_price,
        MAX(price)           AS max_price
    FROM books
    GROUP BY genre
)
SELECT
    gs.genre,
    gs.avg_price,
    gs.book_count,
    b.title AS most_expensive_book
FROM genre_stats gs
JOIN books b ON b.genre = gs.genre AND b.price = gs.max_price
ORDER BY gs.avg_price DESC;
```

### CTE vs Subquery — Kapan Pakai yang Mana

```sql
-- Use a subquery for simple, one-off inline computations
SELECT * FROM books
WHERE price > (SELECT AVG(price) FROM books);

-- Use a CTE when:
-- 1. The same subquery is needed more than once in the query
-- 2. The query has multiple logical steps that should read clearly
-- 3. You want to name intermediate results for documentation

-- Compare readability:
-- HARD TO READ (nested subqueries):
SELECT author_name, total_books, total_value
FROM (
    SELECT a.name AS author_name, COUNT(*) AS total_books, SUM(b.price) AS total_value
    FROM books b JOIN authors a ON b.author_id = a.id GROUP BY a.name
) stats
WHERE total_value > 25
ORDER BY total_value DESC;

-- EASY TO READ (CTE):
WITH author_stats AS (
    SELECT
        a.name       AS author_name,
        COUNT(*)     AS total_books,
        SUM(b.price) AS total_value
    FROM books b
    JOIN authors a ON b.author_id = a.id
    GROUP BY a.name
)
SELECT author_name, total_books, ROUND(total_value, 2) AS total_value
FROM author_stats
WHERE total_value > 25
ORDER BY total_value DESC;
```

### CTE Rekursif

CTE rekursif bisa mereferensikan dirinya sendiri. Digunakan untuk pohon (tree), hirarki, dan graf.

```sql
-- Category tree
CREATE TABLE categories (
    id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name      TEXT NOT NULL,
    parent_id INTEGER REFERENCES categories(id)
);

INSERT INTO categories (name, parent_id) VALUES
    ('All',          NULL),   -- id=1 (root)
    ('Books',        1),      -- id=2
    ('Electronics',  1),      -- id=3
    ('Fiction',      2),      -- id=4
    ('Non-Fiction',  2),      -- id=5
    ('Phones',       3),      -- id=6
    ('Sci-Fi',       4),      -- id=7
    ('Mystery',      4);      -- id=8

-- Get the full path for every category
WITH RECURSIVE category_tree AS (
    -- Anchor member: start from root nodes (no parent)
    SELECT
        id,
        name,
        parent_id,
        name::TEXT AS full_path,
        0          AS depth
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    -- Recursive member: add each node's children
    SELECT
        c.id,
        c.name,
        c.parent_id,
        ct.full_path || ' > ' || c.name,
        ct.depth + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT id, depth, full_path
FROM category_tree
ORDER BY full_path;

-- Result:
-- 1 | 0 | All
-- 2 | 1 | All > Books
-- 3 | 1 | All > Electronics
-- 4 | 2 | All > Books > Fiction
-- 5 | 2 | All > Books > Non-Fiction
-- 6 | 2 | All > Electronics > Phones
-- 7 | 3 | All > Books > Fiction > Sci-Fi
-- 8 | 3 | All > Books > Fiction > Mystery

-- Find all descendants of a specific node
WITH RECURSIVE subtree AS (
    SELECT id, name FROM categories WHERE id = 2  -- start at "Books"
    UNION ALL
    SELECT c.id, c.name
    FROM categories c
    JOIN subtree s ON c.parent_id = s.id
)
SELECT * FROM subtree;  -- Books, Fiction, Non-Fiction, Sci-Fi, Mystery
```

---

## 3.3 Window Function

Window function melakukan kalkulasi di sekumpulan baris yang berhubungan dengan baris saat ini
tanpa menciutkannya menjadi grup. Ini adalah fitur paling powerful di SQL
yang kebanyakan developer tidak tahu.

```sql
-- Syntax
function_name() OVER (
    PARTITION BY column  -- divide rows into groups (optional)
    ORDER BY column      -- define row order within each group
)
```

### ROW_NUMBER, RANK, DENSE_RANK

```sql
-- ROW_NUMBER: sequential number, no ties
SELECT
    title,
    genre,
    price,
    ROW_NUMBER() OVER (PARTITION BY genre ORDER BY price DESC) AS row_num
FROM books;

-- Row 1 = most expensive in genre, row 2 = second most expensive, etc.

-- RANK: tied rows get the same rank, next rank skips numbers
SELECT
    title,
    price,
    RANK() OVER (ORDER BY price DESC) AS price_rank
FROM books;
-- If two books are tied at rank 2, the next book gets rank 4 (rank 3 is skipped)

-- DENSE_RANK: tied rows get the same rank, next rank does NOT skip
SELECT
    title,
    price,
    DENSE_RANK() OVER (ORDER BY price DESC) AS price_rank
FROM books;
-- If two books are tied at rank 2, the next book gets rank 3

-- Practical: get top 2 books per genre
WITH ranked AS (
    SELECT
        title,
        genre,
        price,
        ROW_NUMBER() OVER (PARTITION BY genre ORDER BY price DESC) AS rn
    FROM books
)
SELECT title, genre, price
FROM ranked
WHERE rn <= 2
ORDER BY genre, price DESC;
```

### Running Total dan Moving Average

```sql
-- Running total of book prices, ordered by id
SELECT
    id,
    title,
    price,
    SUM(price) OVER (ORDER BY id) AS running_total
FROM books;

-- Running total within each genre
SELECT
    title,
    genre,
    price,
    SUM(price) OVER (PARTITION BY genre ORDER BY id) AS genre_running_total
FROM books;

-- Running average
SELECT
    id,
    price,
    ROUND(AVG(price) OVER (ORDER BY id), 2) AS running_avg
FROM books;

-- Moving average over last 3 rows (window frame)
SELECT
    id,
    price,
    ROUND(
        AVG(price) OVER (
            ORDER BY id
            ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
        ), 2
    ) AS moving_avg_3
FROM books;
-- For each row: average of (current row + 2 rows before it)
```

### LAG dan LEAD

Mengakses nilai dari baris sebelumnya atau berikutnya dalam hasil.

```sql
-- LAG: look at the previous row
SELECT
    title,
    published_year,
    price,
    LAG(price) OVER (ORDER BY price) AS prev_price,
    price - LAG(price) OVER (ORDER BY price) AS price_gap
FROM books
ORDER BY price;

-- LEAD: look at the next row
SELECT
    title,
    price,
    LEAD(title) OVER (ORDER BY price) AS next_more_expensive
FROM books
ORDER BY price;

-- LAG with offset and default value
SELECT
    id,
    price,
    LAG(price, 2, 0.00) OVER (ORDER BY id) AS price_2_rows_ago
    -- 2 = look 2 rows back; 0.00 = default when no previous row exists
FROM books;
```

### NTILE — Pengelompokan (Bucketing)

```sql
-- Divide books into 4 equal buckets by price (quartiles)
SELECT
    title,
    price,
    NTILE(4) OVER (ORDER BY price) AS quartile
FROM books;
-- Quartile 1 = cheapest 25%, Quartile 4 = most expensive 25%

-- What percentile is each book in?
SELECT
    title,
    price,
    NTILE(100) OVER (ORDER BY price) AS percentile
FROM books;
```

### FIRST_VALUE dan LAST_VALUE

```sql
-- Show cheapest price in each genre alongside every book
SELECT
    title,
    genre,
    price,
    FIRST_VALUE(price) OVER (PARTITION BY genre ORDER BY price) AS cheapest_in_genre,
    FIRST_VALUE(title) OVER (PARTITION BY genre ORDER BY price) AS cheapest_book_in_genre
FROM books;

-- Show MOST EXPENSIVE price in each genre using LAST_VALUE
-- IMPORTANT: You must specify the window frame explicitly!
SELECT
    title,
    genre,
    price,
    LAST_VALUE(price) OVER (
        PARTITION BY genre ORDER BY price
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS most_expensive_in_genre,
    LAST_VALUE(title) OVER (
        PARTITION BY genre ORDER BY price
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS most_expensive_book_in_genre
FROM books;
```

**Kenapa LAST_VALUE butuh frame eksplisit:**

Secara default, ketika kamu menggunakan `ORDER BY` di window function, window frame-nya adalah:
```
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
```

Ini artinya LAST_VALUE hanya melihat baris dari awal partisi sampai baris *saat ini* — jadi ia hanya mengembalikan nilai baris saat ini, yang tidak berguna.

Agar LAST_VALUE bisa melihat semua baris dalam partisi, kamu harus memperluas frame-nya menjadi:
```
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

FIRST_VALUE tidak punya masalah ini karena baris pertama selalu berada dalam frame default.

---

## 3.4 Operasi Himpunan (Set Operation)

### Kenapa Set Operation Ada

Set operation memungkinkan kamu menggabungkan hasil dari query berbeda menjadi satu hasil. Kasus penggunaan umum:
- **Menggabungkan table berbeda dengan struktur sama:** order saat ini + order yang diarsipkan
- **Membangun hasil pencarian terpadu:** cari user berdasarkan nama UNION cari user berdasarkan email
- **Membandingkan kumpulan data:** temukan record di satu table tapi tidak di yang lain (EXCEPT)

```sql
-- UNION: combine two result sets, removing duplicates
SELECT name, 'Author' AS type FROM authors WHERE country = 'UK'
UNION
SELECT name, 'Author' AS type FROM authors WHERE birth_year > 1960;
-- Returns unique rows matching either condition

-- UNION ALL: combine keeping duplicates (faster — no deduplication)
SELECT name FROM authors WHERE country = 'UK'
UNION ALL
SELECT name FROM authors WHERE birth_year > 1960;
-- May return same name twice if they match both conditions

-- INTERSECT: rows that appear in BOTH result sets
SELECT name FROM authors WHERE country = 'UK'
INTERSECT
SELECT name FROM authors WHERE birth_year > 1960;
-- Only UK authors born after 1960

-- EXCEPT: rows in the first set but NOT the second
SELECT name FROM authors WHERE country = 'UK'
EXCEPT
SELECT name FROM authors WHERE birth_year > 1960;
-- UK authors born in 1960 or earlier

-- All set operations require the same number of columns with compatible types
```

### Catatan Performa

- **UNION** mendeduplikasi hasil gabungan, yang membutuhkan pengurutan atau hashing semua baris. Ini mahal untuk result set yang besar.
- **UNION ALL** cukup menggabungkan hasilnya — tanpa pengurutan, tanpa perbandingan. Jauh lebih cepat.
- **Aturan praktis:** Ketika kamu tahu kedua query tidak bisa menghasilkan baris duplikat (misalnya, mereka meng-query table berbeda, atau punya klausa WHERE yang saling eksklusif), selalu gunakan `UNION ALL`.

---

## 3.5 Ekspresi CASE

```sql
-- Simple CASE: match a value against specific cases
SELECT
    title,
    genre,
    CASE genre
        WHEN 'Fantasy'      THEN 'Popular'
        WHEN 'Dystopian'    THEN 'Popular'
        WHEN 'Satire'       THEN 'Literary'
        ELSE 'Other'
    END AS category
FROM books;

-- Searched CASE: evaluate conditions (more flexible)
SELECT
    title,
    price,
    CASE
        WHEN price < 11.00  THEN 'Budget'
        WHEN price < 13.00  THEN 'Mid-Range'
        WHEN price < 15.00  THEN 'Standard'
        ELSE                     'Premium'
    END AS price_tier
FROM books
ORDER BY price;

-- CASE in aggregate functions — conditional counting (pivot)
SELECT
    author_id,
    COUNT(*) AS total,
    COUNT(CASE WHEN in_stock     THEN 1 END) AS in_stock_count,
    COUNT(CASE WHEN NOT in_stock THEN 1 END) AS out_of_stock_count,
    COUNT(CASE WHEN price > 13   THEN 1 END) AS expensive_count
FROM books
GROUP BY author_id
ORDER BY author_id;

-- CASE in ORDER BY: custom sort order
SELECT title, genre
FROM books
ORDER BY
    CASE genre
        WHEN 'Fantasy'   THEN 1
        WHEN 'Dystopian' THEN 2
        ELSE 3
    END,
    title;
```

---

## 3.6 Fungsi Bawaan yang Penting

### Fungsi String

```sql
-- Concatenation
SELECT 'Hello' || ', ' || 'World!';        -- Hello, World!
SELECT CONCAT('Hello', ', ', 'World!');    -- Hello, World!

-- Case conversion
SELECT UPPER('hello world');   -- HELLO WORLD
SELECT LOWER('HELLO WORLD');   -- hello world
SELECT INITCAP('hello world'); -- Hello World

-- Trimming whitespace
SELECT TRIM('  hello  ');      -- 'hello'
SELECT LTRIM('  hello  ');     -- 'hello  '
SELECT RTRIM('  hello  ');     -- '  hello'
SELECT TRIM(BOTH '-' FROM '---hello---'); -- 'hello'

-- Length
SELECT LENGTH('hello');        -- 5 (characters)
SELECT CHAR_LENGTH('hello');   -- 5 (same)

-- Extracting parts
SELECT SUBSTRING('Hello World', 1, 5);  -- 'Hello' (1-indexed)
SELECT LEFT('Hello World', 5);          -- 'Hello'
SELECT RIGHT('Hello World', 5);         -- 'World'

-- Finding positions
SELECT POSITION('World' IN 'Hello World');  -- 7
SELECT STRPOS('Hello World', 'World');      -- 7

-- Replacing
SELECT REPLACE('Hello World', 'World', 'PostgreSQL');  -- Hello PostgreSQL

-- Splitting
SELECT SPLIT_PART('a,b,c,d', ',', 2);   -- 'b' (1-indexed)
SELECT SPLIT_PART('a,b,c,d', ',', 3);   -- 'c'

-- Padding
SELECT LPAD('42', 6, '0');   -- '000042'
SELECT RPAD('hello', 8, '.'); -- 'hello...'

-- Formatting (printf-style)
SELECT FORMAT('Order #%s total: $%.2f', 42, 19.99);
-- 'Order #42 total: $19.99'

-- Regular expression matching
SELECT 'hello123' ~ '^[a-z]+[0-9]+$';  -- true
SELECT REGEXP_REPLACE('hello 123 world', '[0-9]+', 'NUM');  -- 'hello NUM world'
```

### Fungsi Tanggal dan Waktu

```sql
-- Current date/time
SELECT NOW();           -- 2024-01-15 14:30:00+00 (with timezone)
SELECT CURRENT_DATE;    -- 2024-01-15
SELECT CURRENT_TIME;    -- 14:30:00+00

-- Extracting components
SELECT EXTRACT(YEAR   FROM NOW());   -- 2024
SELECT EXTRACT(MONTH  FROM NOW());   -- 1
SELECT EXTRACT(DAY    FROM NOW());   -- 15
SELECT EXTRACT(HOUR   FROM NOW());   -- 14
SELECT EXTRACT(MINUTE FROM NOW());   -- 30
SELECT EXTRACT(DOW    FROM NOW());   -- 1 (day of week: 0=Sun, 6=Sat)
SELECT EXTRACT(DOY    FROM NOW());   -- 15 (day of year)
SELECT EXTRACT(EPOCH  FROM NOW());   -- Unix timestamp (seconds since 1970)

-- Truncating
SELECT DATE_TRUNC('month', NOW());   -- 2024-01-01 00:00:00+00
SELECT DATE_TRUNC('year',  NOW());   -- 2024-01-01 00:00:00+00
SELECT DATE_TRUNC('hour',  NOW());   -- 2024-01-15 14:00:00+00
SELECT DATE_TRUNC('week',  NOW());   -- start of the current week (Monday)

-- Date arithmetic
SELECT NOW() + INTERVAL '7 days';       -- 7 days from now
SELECT NOW() - INTERVAL '1 month';      -- 1 month ago
SELECT NOW() + INTERVAL '2 hours 30 minutes';
SELECT DATE '2024-12-31' - DATE '2024-01-01';  -- 365 (days between dates)

-- Formatting dates as text
SELECT TO_CHAR(NOW(), 'YYYY-MM-DD');            -- '2024-01-15'
SELECT TO_CHAR(NOW(), 'Month DD, YYYY');        -- 'January  15, 2024'
SELECT TO_CHAR(NOW(), 'HH24:MI:SS');            -- '14:30:00'
SELECT TO_CHAR(NOW(), 'Day, DD Mon YYYY');      -- 'Monday  , 15 Jan 2024'

-- Parsing text as dates
SELECT TO_DATE('15/01/2024', 'DD/MM/YYYY');     -- 2024-01-15
SELECT TO_TIMESTAMP('2024-01-15 14:30', 'YYYY-MM-DD HH24:MI');

-- Age calculation
SELECT AGE(NOW(), '1990-06-20'::DATE);  -- 33 years 6 mons 25 days
SELECT EXTRACT(YEAR FROM AGE(NOW(), birth_date))::INTEGER AS age FROM users;
```

### Fungsi Matematika

```sql
SELECT ABS(-42);          -- 42
SELECT CEIL(4.1);         -- 5 (rounds up)
SELECT FLOOR(4.9);        -- 4 (rounds down)
SELECT ROUND(4.567, 2);   -- 4.57 (rounds to 2 decimal places)
SELECT ROUND(4.567, 0);   -- 5
SELECT TRUNC(4.987, 2);   -- 4.98 (truncates, does not round)
SELECT MOD(17, 5);        -- 2 (remainder after division)
SELECT POWER(2, 10);      -- 1024.0
SELECT SQRT(144);         -- 12.0
SELECT RANDOM();          -- random float between 0 (inclusive) and 1 (exclusive)
SELECT GREATEST(3, 7, 2, 8, 1);  -- 8
SELECT LEAST(3, 7, 2, 8, 1);     -- 1
```

---

> **Tips:** Gunakan `EXPLAIN ANALYZE` untuk melihat bagaimana PostgreSQL mengeksekusi query-mu. Kita bahas ini secara mendalam di Buku 2, tapi ketika query terasa lambat, mulai dari sini:
> ```sql
> EXPLAIN ANALYZE SELECT * FROM books WHERE price > 20;
> ```
> Ini menampilkan execution plan, estimasi baris, dan waktu aktual.

---

## Fase 3 — Latihan

1. Temukan 3 buku termahal di setiap genre menggunakan window function (bukan subquery).
2. Untuk setiap buku, tampilkan berapa tahun yang berlalu sejak buku sebelumnya oleh penulis yang sama (diurutkan berdasarkan tahun terbit). Gunakan LAG.
3. Hitung running average harga di semua buku yang diurutkan berdasarkan published_year.
4. Gunakan CTE rekursif untuk menghasilkan bilangan bulat 1 sampai 20.
5. Temukan genre di mana buku termahal harganya lebih dari dua kali lipat buku termurah di genre tersebut.
6. Klasifikasikan setiap buku menggunakan CASE: 'Classic' (sebelum 1980), 'Modern' (1980-2010), 'Contemporary' (setelah 2010).
7. Tulis query yang mengembalikan setiap penulis dengan: total buku, rata-rata harga, dan persentase rata-rata harganya di atas atau di bawah rata-rata keseluruhan.
8. Menggunakan UNION, tulis satu query yang mengembalikan penulis yang namanya mengandung "Garcia" DAN buku yang judulnya mengandung "Harry", dengan kolom yang melabeli tipe setiap hasilnya.

---

# Fase 4 — Desain Schema Dunia Nyata

## 4.1 Timestamp

Setiap table yang bisa berubah butuh `created_at` dan `updated_at`.

```sql
-- Basic pattern
CREATE TABLE products (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Auto-update updated_at using a trigger
CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_products_updated_at
BEFORE UPDATE ON products
FOR EACH ROW EXECUTE FUNCTION set_updated_at();

-- Now any UPDATE automatically refreshes updated_at
UPDATE products SET name = 'New Name' WHERE id = 1;
-- updated_at is set to NOW() automatically — you do not need to pass it
```

---

## 4.2 Soft Delete

Tandai baris sebagai terhapus alih-alih menghapusnya secara fisik.
Menjaga riwayat dan membuat penghapusan yang tidak disengaja bisa dipulihkan.

```sql
-- Pattern 1: deleted_at timestamp
CREATE TABLE products (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       TEXT NOT NULL,
    deleted_at TIMESTAMPTZ   -- NULL = active, non-NULL = deleted
);

-- Soft delete
UPDATE products SET deleted_at = NOW() WHERE id = 42;

-- All normal queries must filter soft-deleted rows
SELECT * FROM products WHERE deleted_at IS NULL;

-- Create a VIEW to hide the complexity
CREATE VIEW active_products AS
SELECT * FROM products WHERE deleted_at IS NULL;

-- Restore a deleted product
UPDATE products SET deleted_at = NULL WHERE id = 42;

-- Pattern 2: is_active boolean (simpler, but loses the deletion timestamp)
CREATE TABLE products (
    id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name      TEXT NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT true
);
```

### Trade-off Soft Delete

Soft delete tidak gratis. Pertimbangkan biaya-biaya ini sebelum memilihnya:

**1. Setiap query harus ingat untuk memfilter.**
Setiap `SELECT` terhadap table harus menyertakan `WHERE deleted_at IS NULL` (atau menggunakan view). Jika ada satu query saja yang lupa filter ini, aplikasimu menampilkan data yang "terhapus" ke pengguna. Ini sumber bug yang umum.

**2. Unique constraint jadi bermasalah.**
Jika kamu punya `UNIQUE(email)` dan soft-delete sebuah user, email mereka masih ada di table. User baru tidak bisa mendaftar dengan email yang sama. Solusinya adalah **partial unique index**:

```sql
-- Only enforce uniqueness among non-deleted rows
CREATE UNIQUE INDEX idx_users_email_active
ON users (email)
WHERE deleted_at IS NULL;
```

Ini memperbolehkan beberapa baris soft-deleted dengan email yang sama, tapi hanya satu baris aktif.

**3. Ukuran table terus tumbuh selamanya.**
Baris yang soft-deleted terakumulasi. Pada table bervolume tinggi, ini bisa meningkatkan penyimpanan secara signifikan dan memperlambat query yang memindai table.

**Kapan hard delete lebih baik:**
- **GDPR / regulasi privasi data** — kamu mungkin diwajibkan secara hukum untuk benar-benar menghapus data pribadi, bukan hanya menyembunyikannya.
- **Data sementara** — session token, kode temporer, atau entri cache tidak perlu dipertahankan.
- **Data transien bervolume tinggi** — log entry, event analytics, atau message queue di mana data lama tidak punya nilai.

---

## 4.3 Audit Trail (Jejak Audit)

Catat setiap perubahan pada data kritis — siapa mengubah apa dan kapan.

```sql
-- Simple audit: who last updated the row
CREATE TABLE orders (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    status     TEXT NOT NULL DEFAULT 'pending',
    updated_by INTEGER REFERENCES users(id),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Full audit log: complete history of changes
CREATE TABLE order_status_history (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    order_id   INTEGER NOT NULL REFERENCES orders(id),
    old_status TEXT,
    new_status TEXT NOT NULL,
    changed_by INTEGER REFERENCES users(id),
    changed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    reason     TEXT
);

-- Trigger to automatically record status changes
CREATE OR REPLACE FUNCTION log_order_status_change()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.status <> OLD.status THEN
        INSERT INTO order_status_history (order_id, old_status, new_status, changed_at)
        VALUES (NEW.id, OLD.status, NEW.status, NOW());
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_order_status_audit
AFTER UPDATE ON orders
FOR EACH ROW EXECUTE FUNCTION log_order_status_change();

-- Now every status change is automatically recorded
UPDATE orders SET status = 'shipped' WHERE id = 1;
SELECT * FROM order_status_history WHERE order_id = 1;
```

---

## 4.4 Hubungan Many-to-Many

Gunakan junction table. Jangan pernah menggunakan ID yang dipisah koma dalam satu kolom.

```sql
CREATE TABLE tags (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL UNIQUE
);

CREATE TABLE book_tags (
    book_id INTEGER NOT NULL REFERENCES books(id) ON DELETE CASCADE,
    tag_id  INTEGER NOT NULL REFERENCES tags(id)  ON DELETE CASCADE,
    PRIMARY KEY (book_id, tag_id)  -- composite PK prevents duplicate tagging
);

-- Tag a book
INSERT INTO tags (name) VALUES ('classic'), ('dystopian'), ('must-read');
INSERT INTO book_tags (book_id, tag_id) VALUES (1, 1), (1, 2), (1, 3);

-- All tags for a specific book
SELECT t.name
FROM tags t
JOIN book_tags bt ON bt.tag_id = t.id
WHERE bt.book_id = 1;

-- All books with a specific tag
SELECT b.title
FROM books b
JOIN book_tags bt ON bt.book_id = b.id
JOIN tags t       ON bt.tag_id  = t.id
WHERE t.name = 'classic';

-- Books that have ALL of a specific set of tags
SELECT b.title
FROM books b
JOIN book_tags bt ON bt.book_id = b.id
JOIN tags t       ON bt.tag_id  = t.id
WHERE t.name IN ('classic', 'dystopian')
GROUP BY b.id, b.title
HAVING COUNT(DISTINCT t.id) = 2;  -- must match BOTH tags
```

---

## 4.5 Struktur Pohon (Tree) di PostgreSQL

### Adjacency List (Pilihan Default)

```sql
CREATE TABLE categories (
    id        INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name      TEXT NOT NULL,
    parent_id INTEGER REFERENCES categories(id) ON DELETE CASCADE
    -- NULL parent_id means root node
);

-- Get all root categories
SELECT * FROM categories WHERE parent_id IS NULL;

-- Get direct children of category id=2
SELECT * FROM categories WHERE parent_id = 2;

-- Get full tree with path (recursive CTE)
WITH RECURSIVE tree AS (
    SELECT id, name, parent_id, name::TEXT AS path, 0 AS depth
    FROM categories WHERE parent_id IS NULL
    UNION ALL
    SELECT c.id, c.name, c.parent_id, t.path || ' > ' || c.name, t.depth + 1
    FROM categories c JOIN tree t ON c.parent_id = t.id
)
SELECT id, depth, path FROM tree ORDER BY path;
```

### Ekstensi ltree (Untuk Penggunaan Tree yang Berat)

```sql
CREATE EXTENSION IF NOT EXISTS ltree;

CREATE TABLE categories (
    id   INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name TEXT NOT NULL,
    path ltree NOT NULL  -- 'Root.Clothing.Shirts'
);

CREATE INDEX idx_cat_path ON categories USING GIST(path);

INSERT INTO categories (name, path) VALUES
    ('All',       'All'),
    ('Books',     'All.Books'),
    ('Fiction',   'All.Books.Fiction'),
    ('Sci-Fi',    'All.Books.Fiction.SciFi');

-- All descendants of Books
SELECT * FROM categories WHERE path <@ 'All.Books';

-- All ancestors of Sci-Fi
SELECT * FROM categories WHERE path @> 'All.Books.Fiction.SciFi';

-- All siblings (same parent)
SELECT * FROM categories WHERE path ~ 'All.Books.*{1}';
```

---

## 4.6 JSONB dalam Praktik

```sql
-- Flexible product attributes (different per category)
CREATE TABLE products (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       TEXT NOT NULL,
    price      NUMERIC(10,2) NOT NULL,
    attributes JSONB
);

INSERT INTO products (name, price, attributes) VALUES
    ('Go Programming Language', 39.99, '{"pages": 380, "isbn": "978-0134190440"}'),
    ('MacBook Pro',            1999.00, '{"ram_gb": 16, "storage_gb": 512}'),
    ('Blue T-Shirt',             19.99, '{"size": "L", "color": "blue"}');

-- Query by JSONB field
SELECT name FROM products WHERE attributes->>'color' = 'blue';
SELECT name FROM products WHERE (attributes->>'pages')::INTEGER > 300;

-- @> contains
SELECT name FROM products WHERE attributes @> '{"color": "blue"}';

-- ? field exists
SELECT name FROM products WHERE attributes ? 'isbn';

-- GIN index for fast JSONB queries
CREATE INDEX idx_products_attrs ON products USING GIN(attributes);

-- Updating JSONB fields
UPDATE products
SET attributes = attributes || '{"in_print": true}'::JSONB
WHERE id = 1;

-- Removing a JSONB key
UPDATE products
SET attributes = attributes - 'in_print'
WHERE id = 1;

-- When NOT to use JSONB:
-- If you query a field in almost every SELECT -> make it a real column
-- If you join on a field -> make it a real column
-- JSONB is for optional, sparse, or varying-structure data
```

---

## 4.7 Migrasi Schema yang Aman

Migrasi adalah file SQL berversi yang melacak evolusi schema.

```
migrations/
├── 000001_create_users.up.sql
├── 000001_create_users.down.sql
├── 000002_create_products.up.sql
├── 000002_create_products.down.sql
```

Selalu tulis migrasi `down`. Selalu tes rollback.

### Tools Migrasi

Tools umum untuk mengelola migrasi:

- **golang-migrate** — CLI yang language-agnostic, bekerja dengan banyak database. Populer di proyek Go.
- **goose** — native Go, mendukung migrasi berbasis SQL dan Go.
- **dbmate** — ringan, language-agnostic, CLI sederhana.
- **flyway** — berbasis Java, banyak digunakan di enterprise. Punya edisi komunitas gratis.

### File Up dan Down

Setiap migrasi punya dua file:
- **File up** (`000001_create_users.up.sql`): berisi perubahan maju (CREATE TABLE, ADD COLUMN, dll.)
- **File down** (`000001_create_users.down.sql`): berisi perubahan kebalikannya (DROP TABLE, DROP COLUMN, dll.)

Tools migrasi melacak migrasi mana yang sudah diterapkan di table `schema_migrations`. Menjalankan `migrate up` menerapkan migrasi yang pending secara berurutan. Menjalankan `migrate down` melakukan rollback migrasi terbaru.

### Operasi Aman vs Berbahaya

```sql
-- SAFE: adding a nullable column (no table rewrite, no lock)
ALTER TABLE users ADD COLUMN avatar_url TEXT;

-- SAFE: adding a column with a default (PostgreSQL 11+: no table rewrite)
ALTER TABLE users ADD COLUMN verified BOOLEAN NOT NULL DEFAULT false;

-- SAFE: adding an index CONCURRENTLY (does not lock the table)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- DANGEROUS: renaming a column (breaks running app that uses old name)
-- Do it in stages:
-- Step 1: add new column
ALTER TABLE users ADD COLUMN full_name TEXT;
-- Step 2: populate it (can be a background job on large tables)
UPDATE users SET full_name = name;
-- Step 3: add NOT NULL once populated
ALTER TABLE users ALTER COLUMN full_name SET NOT NULL;
-- Step 4: after deploying new code that uses full_name, drop old column
ALTER TABLE users DROP COLUMN name;

-- DANGEROUS: changing column type (rewrites every row, locks the table)
-- Add new column -> backfill -> switch reads -> drop old column

-- DANGEROUS: adding NOT NULL to existing column (fails if any NULL exists)
-- First backfill NULLs, then:
UPDATE users SET phone = '' WHERE phone IS NULL;
ALTER TABLE users ALTER COLUMN phone SET NOT NULL;
```

### Catatan tentang CREATE INDEX CONCURRENTLY

`CREATE INDEX CONCURRENTLY` tidak bisa dijalankan di dalam transaksi. Jika tools migrasimu membungkus setiap migrasi dalam transaksi (kebanyakan melakukannya secara default), pembuatan index concurrent akan gagal. Solusinya:
- Gunakan mode "no transaction" dari tools migrasimu untuk migrasi spesifik itu (misalnya, golang-migrate mendukung `-- +migrate NoTransaction`)
- Buat index di file migrasi terpisah yang berjalan di luar transaksi

---

## 4.8 Referensi Pola Schema Umum

### User dan Role

```sql
CREATE TABLE users (
    id            INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email         TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    name          TEXT NOT NULL,
    role          TEXT NOT NULL DEFAULT 'customer'
                  CHECK (role IN ('customer', 'admin', 'moderator')),
    email_verified BOOLEAN NOT NULL DEFAULT false,
    created_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**Kapan menggunakan CHECK constraint vs table role terpisah:**
- **CHECK constraint** (seperti di atas): gunakan ketika set role kecil, tetap, dan kemungkinan tidak berubah. Sederhana, tidak perlu join tambahan.
- **Table role terpisah**: gunakan ketika role butuh metadata tambahan (deskripsi, permission), ketika role sering berubah, atau ketika user bisa punya banyak role (hubungan many-to-many melalui junction table `user_roles`).

### Produk dengan Varian

```sql
CREATE TABLE products (
    id          INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name        TEXT NOT NULL,
    description TEXT,
    category_id INTEGER REFERENCES categories(id)
);

CREATE TABLE product_variants (
    id         INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    product_id INTEGER NOT NULL REFERENCES products(id) ON DELETE CASCADE,
    sku        TEXT UNIQUE,
    size       TEXT,
    color      TEXT,
    price      NUMERIC(10,2) NOT NULL CHECK (price > 0),
    stock      INTEGER NOT NULL DEFAULT 0 CHECK (stock >= 0),
    UNIQUE (product_id, size, color)  -- each combination unique per product
);
```

**Kenapa table varian terpisah:** Sebuah produk seperti "T-Shirt" bisa tersedia dalam 3 ukuran dan 4 warna = 12 varian, masing-masing dengan harga, level stok, dan SKU sendiri. Menyimpannya sebagai kolom di produk (misalnya, `stock_small`, `stock_medium`, `stock_large`) tidak scalable dan membuat query menyakitkan.

**Kenapa composite unique constraint:** `UNIQUE (product_id, size, color)` memastikan kamu tidak bisa secara tidak sengaja membuat dua varian "Blue, Large" untuk produk yang sama. Ini adalah aturan bisnis yang ditegakkan di level database.

### Social Graph (Pengikut)

```sql
CREATE TABLE follows (
    follower_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    followed_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (follower_id, followed_id),
    CHECK (follower_id <> followed_id)  -- cannot follow yourself
);

-- Get followers of user 42
SELECT u.name FROM follows f JOIN users u ON u.id = f.follower_id WHERE f.followed_id = 42;

-- Get who user 42 is following
SELECT u.name FROM follows f JOIN users u ON u.id = f.followed_id WHERE f.follower_id = 42;
```

**Penting: buat index di kedua sisi.** Composite primary key `(follower_id, followed_id)` membuat index yang efisien untuk query yang dimulai dengan `follower_id` (misalnya, "siapa yang diikuti user 42?"). Tapi query yang dimulai dengan `followed_id` (misalnya, "siapa yang mengikuti user 42?") tidak bisa menggunakan index ini secara efisien. Kamu butuh index terpisah:

```sql
CREATE INDEX idx_follows_followed_id ON follows (followed_id);
```

Tanpa index ini, "dapatkan pengikut user X" membutuhkan full table scan.

---

## 4.9 VIEW

VIEW adalah query tersimpan yang bisa kamu referensikan seperti table. Ia tidak menyimpan data apapun — setiap kali kamu meng-query view, PostgreSQL menjalankan query yang mendasarinya.

```sql
-- Basic syntax
CREATE VIEW active_products AS
SELECT id, name, price
FROM products
WHERE deleted_at IS NULL;

-- Now you can query it like a table
SELECT * FROM active_products WHERE price > 10;

-- Drop a view
DROP VIEW active_products;

-- Replace a view (change its definition without dropping)
CREATE OR REPLACE VIEW active_products AS
SELECT id, name, price, created_at
FROM products
WHERE deleted_at IS NULL;
```

### Kapan Menggunakan VIEW

- **Menyederhanakan query yang berulang:** Kita sudah menggunakan view di bagian soft delete (`CREATE VIEW active_products AS SELECT * FROM products WHERE deleted_at IS NULL`). Daripada menambahkan `WHERE deleted_at IS NULL` ke setiap query, kamu cukup meng-query `active_products`.
- **Mengabstraksi join yang kompleks:** Jika banyak bagian aplikasimu membutuhkan join 4 table yang sama, bungkus dalam view.
- **Membatasi akses:** Berikan user akses ke view yang hanya menampilkan kolom tertentu, bukan seluruh table.

### VIEW Tidak Menyimpan Data

View hanyalah query yang diberi nama. Tidak ada overhead penyimpanan. Ketika kamu `SELECT * FROM active_products`, PostgreSQL menulisnya ulang menjadi `SELECT * FROM products WHERE deleted_at IS NULL` dan mengeksekusinya. Jika kamu butuh view yang menyimpan data untuk performa, lihat **materialized view** (dibahas di Buku 2).

---

## Fase 4 — Latihan: Platform Belajar Online

Desain schema lengkap untuk platform ini:

**Persyaratan:**
- User bisa menjadi student atau instructor
- Instructor membuat course dengan banyak lesson (berurutan)
- Course termasuk dalam kategori (hierarkis)
- Student mendaftar (enroll) ke course
- Student melacak progress per lesson: not started / in progress / completed
- Course punya review: 1-5 bintang dan teks opsional
- Course punya harga (0 = gratis)
- Instructor mendapat persentase komisi per penjualan

Tulis:
1. Semua statement `CREATE TABLE` dengan constraint lengkap
2. Query yang mengembalikan setiap course dengan: nama instructor, path kategori, rata-rata rating, jumlah enrollment, dan tingkat penyelesaian
3. Query yang menampilkan progress student tertentu: nama course, total lesson, lesson yang selesai, persentase selesai
4. Revenue bulanan per instructor untuk 6 bulan terakhir
5. Trigger yang mencatat perubahan status enrollment ke table riwayat

---

# Capstone — Database Ecommerce

Bangun dan query database ecommerce lengkap dari nol.

## Bagian 1 — Schema

Desain schema yang mendukung:
- User dengan banyak alamat pengiriman (satu default)
- Produk dengan kategori (hierarkis), gambar, dan varian ukuran/warna
- Pelacakan stok per varian
- Wishlist
- Order dengan line item (snapshot harga)
- Catatan pembayaran
- Review dengan rating
- Kode diskon dengan masa berlaku dan batas penggunaan

Persyaratan:
- Semua kolom uang menggunakan `NUMERIC(10,2)`
- Semua timestamp menggunakan `TIMESTAMPTZ`
- Soft delete pada produk (`deleted_at`)
- Timestamp audit pada semua table yang bisa berubah
- Semua constraint `NOT NULL`, `UNIQUE`, dan `CHECK` yang sesuai
- Foreign key dengan perilaku `ON DELETE` yang sesuai

## Bagian 2 — Data Awal (Seed)

Isi schema-mu dengan:
- 5 user (2 admin, 3 customer), masing-masing dengan minimal 1 alamat
- 3 hirarki kategori (minimal 2 level kedalaman)
- 15 produk di berbagai kategori, minimal 5 dengan banyak varian
- 10 order dengan 2+ item masing-masing
- Catatan pembayaran untuk setiap order
- Review untuk minimal 8 produk
- 3 kode diskon (satu kadaluarsa, satu aktif, satu habis terpakai)

## Bagian 3 — Query yang Dibutuhkan

Tulis SQL untuk setiap query aplikasi nyata berikut:

**1. Halaman daftar produk**
Semua produk aktif dengan: nama kategori, URL gambar utama, harga varian terendah,
total stok di semua varian, rata-rata rating review, dan jumlah review.
Bisa difilter berdasarkan kategori. Bisa diurutkan berdasarkan harga (asc/desc) atau rating.

**2. Halaman detail produk**
Satu produk dengan: semua gambar diurutkan berdasarkan posisi, semua varian beserta
level stoknya, dan 5 review terbaru dengan nama reviewer dan rating.

**3. Riwayat order user**
Semua order untuk user tertentu: tanggal order, status, jumlah item, dan total.
Terbaru dulu.

**4. Tampilan detail order**
Satu order dengan: semua line item (nama produk, ukuran/warna varian, kuantitas,
harga satuan, total baris), snapshot alamat pengiriman, dan status pembayaran.

**5. Admin: order yang perlu dipenuhi**
Semua order yang sudah dikonfirmasi tapi belum dikirim. Tampilkan: ID order, nama customer,
tanggal order, jumlah item, dan total. Terlama dulu.

**6. Admin: peringatan stok rendah**
Semua varian produk aktif dengan stok 5 atau kurang.
Tampilkan: nama produk, SKU varian, ukuran, warna, dan stok saat ini.

**7. Admin: laporan revenue bulanan**
Total revenue, jumlah order, dan rata-rata nilai order per bulan untuk 12 bulan terakhir.
Sertakan bulan dengan revenue nol (petunjuk: gunakan `generate_series`).

**8. Admin: 10 produk teratas berdasarkan revenue**
Nama produk, kategori, total unit terjual, dan total revenue.
Sepanjang waktu. Diurutkan berdasarkan revenue menurun.

**9. Sering dibeli bersamaan**
Diberikan sebuah product ID, temukan 5 produk yang paling sering dipesan dalam order yang sama.
Tampilkan nama produk dan jumlah kemunculan bersama.

**10. Validasi kode diskon**
Diberikan string kode, kembalikan detail diskon hanya jika:
- Kode ada dan tidak terhapus
- Tanggal saat ini dalam rentang yang valid
- Jumlah penggunaan belum melebihi batas

## Bagian 4 — Temukan Masalahnya

Schema berikut memiliki tepat 5 masalah desain. Identifikasi setiap masalah dan
tulis versi yang sudah diperbaiki:

```sql
CREATE TABLE orders (
    id           SERIAL PRIMARY KEY,
    user_email   TEXT NOT NULL,
    user_name    TEXT NOT NULL,
    items        TEXT,
    total        FLOAT,
    created      TIMESTAMP,
    status       TEXT DEFAULT 'new'
);
```

**Masalah yang harus ditemukan:**
1. Data yang seharusnya ada di table terpisah disimpan di sini
2. Sebuah kolom menggunakan tipe yang salah untuk tujuannya
3. Tipe kolom yang seharusnya tidak pernah digunakan untuk nilai moneter
4. Kolom timestamp kehilangan informasi kritis
5. Kolom yang menyimpan teks bebas tanpa validasi yang seharusnya dibatasi

## Kriteria Penyelesaian

Kamu selesai ketika:
- [ ] Semua table dibuat tanpa error
- [ ] Semua 10 query mengembalikan hasil yang benar terhadap data seed-mu
- [ ] Kamu menggunakan window function di minimal 3 query yang sesuai
- [ ] Kamu menggunakan CTE untuk query yang punya lebih dari 2 langkah logis
- [ ] Kamu bisa menjelaskan kenapa memilih setiap tipe data
- [ ] Kamu mengidentifikasi dan memperbaiki semua 5 masalah schema
- [ ] Kamu bisa menggambar diagram ER untuk schema-mu dari ingatan

---

## Apa Selanjutnya — Buku 2

Sekarang kamu sudah bisa mendesain schema yang benar dan menulis query SQL apapun. Buku 2 menjawab
pertanyaan berikutnya: **kenapa query ini lambat, dan bagaimana cara memperbaikinya?**

**Buku 2 — PostgreSQL Internals untuk Engineer** membahas:
- Bagaimana PostgreSQL secara fisik menyimpan baris-barismu di disk (page, heap, TOAST)
- Apa itu B-tree index sebenarnya dan bagaimana PostgreSQL menggunakannya untuk menemukan baris
- Cara membaca output `EXPLAIN ANALYZE` dan mengidentifikasi bottleneck
- Bagaimana transaksi dan MVCC bekerja di balik layar
- Cara meng-tune database production yang berjalan lambat

Setiap query yang kamu tulis di buku ini akan lebih masuk akal setelah Buku 2,
karena kamu akan mengerti apa yang sebenarnya dilakukan database untuk mengeksekusinya.

---

*Baca. Ketik setiap contoh. Kerjakan setiap latihan. Semoga berhasil.*
