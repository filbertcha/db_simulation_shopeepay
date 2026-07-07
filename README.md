# E-Wallet Database Simulation

Simulasi database **ShopeePay** menggunakan PostgreSQL yang dirancang untuk mempelajari konsep perancangan database relasional, normalisasi, serta implementasi query SQL pada sistem dompet digital.

---

# Tech Stack

- PostgreSQL
- pgAdmin 4

---

# 1. Data

## 1.1 Halaman Beranda

- Nominal Tabungan
- Produk Flash Sale (Gambar, Judul, Harga, Deskripsi, Waktu, Stock)

## 1.2 Halaman Keuangan

- Produk Asuransi (Gambar, Judul, Harga, Deskripsi)

## 1.3 Halaman Riwayat

- Tanda Nominal (+/-)
- Nominal Transaksi
- Status transaksi
- Tanggal Transaksi dibuat
- Transaksi SN
- Metode Pembayaran
- Nama Penerima
- Nama Pengirim

## 1.4 Halaman Saya

- Foto
- Username
- No. Handphone
- Password

---

# 2 Database Structure Design (dbdiagram.io)

## 2.1 Hasil Visual

![Database Design](/ShopeePayDB.png)

## 2.2 Code

```
Table users {
  id SERIAL [pk, increment]
  username VARCHAR(50) [unique, not null]
  phone_number VARCHAR(15) [unique, not null]
  password_hash VARCHAR(255) [not null]
  photo_url VARCHAR(2000)
  created_at TIMESTAMP
  updated_at TIMESTAMP
}

Table savings {
  id SERIAL [pk, increment]
  user_id INTEGER [ref: > users.id]
  balance NUMERIC [not null, default: 0.00]
  updated_at TIMESTAMP
}

Table flash_sales {
  id SERIAL [pk, increment]
  title VARCHAR(100) [unique, not null]
  description VARCHAR(300)
  price NUMERIC [not null]
  image_url VARCHAR(2000)
  start_time DATETIME [not null]
  end_time DATETIME [not null]
  stock INTEGER [not null, default: 0]
  created_at TIMESTAMP
}

Table insurance_products {
  id SERIAL [pk, increment]
  title VARCHAR(100) [unique, not null]
  description VARCHAR(300)
  price NUMERIC [not null]
  image_url VARCHAR(2000)
  is_active BOOL
  created_at TIMESTAMP
}

Table payment_methods {
  id SERIAL [pk, increment]
  name VARCHAR(50) [unique, not null]
}

Table transactions {
  id SERIAL [pk, increment]
  transaction_sn VARCHAR(64) [unique, not null]
  user_id INTEGER [ref: > users.id]
  amount NUMERIC [not null]
  type ENUM('+', '-') [not null]
  status VARCHAR(20) [not null, default: 'pending']
  created_at TIMESTAMP
  sender_name VARCHAR(50) [not null]
  recipient_name VARCHAR(50) [not null]
  payment_method_id INTEGER [ref: > payment_methods.id]
  description VARCHAR(300)
  related_flash_sale_id INTEGER [ref: > flash_sales.id]
  related_insurance_id INTEGER [ref: > insurance_products.id]
}

```

---

# 3. Cara Menjalankan

## 3.1 Clone Repository

```bash
git clone https://github.com/filbertcha/db_simulation_shopeepay.git
```

## 3.2 Masuk ke Folder

```bash
cd db_simulation_shopeepay
```

## 3.3 Buat Database

```sql
CREATE DATABASE shopeepay_db;
```

## 3.4 Jalankan Script

Import file SQL (shopeepay_db.sql)

---

# 4. Study Case

## 4.1 Isi Saldo

### 4.1.1 Membuat transaksi top-up (INSERT)

### SQL

```sql
INSERT INTO transactions (
    transaction_sn, user_id, amount, type, status,
    sender_name, recipient_name, payment_method_id, description, created_at
) VALUES (
    'TOPUP/20260707/XYZ123',
    1,
    300000,
    '+',
    'success',
    'budi_prasetyo',
    'budi_prasetyo',
    2,
    'Top-up saldo via Bank BCA',
    NOW()
);

```

---

### 4.1.2 Update saldo setelah top-up (UPDATE)

### SQL

```sql
UPDATE savings
SET balance = balance + 300000,
    updated_at = NOW()
WHERE user_id = 1;
```

---

---

### 4.1.3 Cek riwayat top-up dengan filter (WHERE + LIKE)

### SQL

```sql
SELECT
    transaction_sn,
    amount,
    sender_name,
    description,
    created_at AS tanggal
FROM transactions
WHERE
    user_id = 1
    AND type = '+'
    AND description LIKE '%Top-up%'
ORDER BY created_at DESC;
```

### Output

![output](/output_img/out_1.png)

---

## 4.2 Kirim Uang

### 4.2.1 Mencari penerima berdasarkan nomor HP (JOIN)

### SQL

```sql
SELECT
    u.id,
    u.username,
    u.phone_number,
    s.balance
FROM users AS u
INNER JOIN savings AS s ON u.id = s.user_id
WHERE u.phone_number = '085612345678';

```

### Output

![output](/output_img/out_2.png)

---

### 4.2.2 Membuat transaksi kirim uang (INSERT)

### SQL

```sql
INSERT INTO transactions (
    transaction_sn, user_id, amount, type, status,
    sender_name, recipient_name, payment_method_id, description, created_at
) VALUES (
    'TRF/20260707/ABC456',
    1,
    100000,
    '-',
    'success',
    'budi_prasetyo',
    'ani_wijaya',
    1,
    'Transfer ke Ani - Bayar makan siang',
    NOW()
);

```

---

### 4.2.3 Membuat transaksi untuk penerima (INSERT)

### SQL

```sql
INSERT INTO transactions (
    transaction_sn, user_id, amount, type, status,
    sender_name, recipient_name, payment_method_id, description, created_at
) VALUES (
    'TRF/20260707/ABC456-IN',
    2,
    100000,
    '+',
    'success',
    'budi_prasetyo',
    'ani_wijaya',
    1,
    'Transfer ke Ani - Bayar makan siang',
    NOW()
);

```

---

### 4.2.4 Update saldo pengirim dan penerima (UPDATE)

### SQL

#### Pengirim

```sql
UPDATE savings
SET balance = balance - 100000, updated_at = NOW()
WHERE user_id = 1;
```

#### Penerima

```sql
UPDATE savings
SET balance = balance + 100000, updated_at = NOW()
WHERE user_id = 2;
```

---

### 4.2.5 Riwayat transfer dengan kategori (CASE)

### SQL

```sql
SELECT
    transaction_sn,
    amount,
    CASE
        WHEN type = '-' THEN 'Uang Keluar'
        WHEN type = '+' THEN 'Uang Masuk'
    END AS jenis,
	recipient_name,
	sender_name,
    description,
    created_at AS tanggal
FROM transactions
WHERE
    user_id IN (1, 2)
ORDER BY created_at DESC;
```

### Output

![output](/output_img/out_3.png)

---

## 4.3 Cek Riwayat Transaksi Dalam 1 Akun

### 4.3.1 Filter transaksi berdasarkan type (WHERE + IN)

### SQL

```sql
SELECT
    transaction_sn,
    amount,
    type,
    status,
    description
FROM transactions
WHERE
    user_id = 1
    AND type IN ('+', '-')
    AND status = 'success'
ORDER BY created_at DESC;
```

### Output

![output](/output_img/out_4.png)

---

## 4.4 Edit Profile

### 4.4.1 Cek username/phone sudah dipakai (Subquery + EXISTS)

### SQL

```sql
SELECT
    CASE
        WHEN EXISTS (
            SELECT 1 FROM users WHERE username = 'eka_safitri_new' AND id != 5
        ) THEN 'Username sudah terpakai'
        ELSE 'Username tersedia'
    END AS status_username,

    CASE
        WHEN EXISTS (
            SELECT 1 FROM users WHERE phone_number = '081299887766' AND id != 5
        ) THEN 'Nomor HP sudah terpakai'
        ELSE 'Nomor HP tersedia'
    END AS status_phone;
```

### Output

![output](/output_img/out_5.png)

---

### 4.4.2 Update profile (UPDATE + CASE)

### SQL

```sql
UPDATE users
SET
    username = CASE
        WHEN 'eka_safitri_update' != '' THEN 'eka_safitri_update'
        ELSE username
    END,
    photo_url = CASE
        WHEN '/photos/eka_new.jpg' != '' THEN '/photos/eka_new.jpg'
        ELSE photo_url
    END,
    password_hash = CASE
        WHEN 'hash_password_baru_123' != '' THEN 'hash_password_baru_123'
        ELSE password_hash
    END,
    updated_at = NOW()
WHERE id = 5;
```

---

### 4.4.3 Lihat profile lengkap (JOIN)

### SQL

```sql
SELECT
    u.username,
    u.phone_number,
    u.photo_url,
    s.balance,
    u.created_at AS member_sejak
FROM users u
INNER JOIN savings s ON u.id = s.user_id
WHERE u.id = 5;
```

### Output

![output](/output_img/out_6.png)

---

## 4.5 Beli Produk Flash Sale

### 4.5.1 Lihat produk flash sale aktif (WHERE + BETWEEN)

### SQL

```sql
SELECT
    id,
    title,
    price,
    stock,
    end_time AS batas_waktu
FROM flash_sales
WHERE
    NOW() BETWEEN start_time AND end_time
    AND stock > 0
ORDER BY price ASC;
```

### Output

![output](/output_img/out_7.png)

---

### 4.5.2 Cek saldo cukup (Subquery + CASE)

### SQL

```sql
SELECT
    (SELECT balance FROM savings WHERE user_id = 1) AS saldo_saya,
    price AS harga_produk,
    CASE
        WHEN (SELECT balance FROM savings WHERE user_id = 1) >= price
        THEN 'Saldo Cukup'
        ELSE 'Saldo Tidak Cukup'
    END AS status
FROM flash_sales
WHERE id = 3;
```

### Output

![output](/output_img/out_8.png)

---

### 4.5.3 Proses pembelian (INSERT)

### SQL

```sql
INSERT INTO transactions (
    transaction_sn, user_id, amount, type, status,
    sender_name, recipient_name, payment_method_id,
    description, related_flash_sale_id, created_at
) VALUES (
    'FLS/20260707/DEF789',
    1,
    (SELECT price FROM flash_sales WHERE id = 3),
    '-',
    'success',
    (SELECT username FROM users WHERE id = 1),
    (SELECT title FROM flash_sales WHERE id = 3),
    1,
    'Beli: ' || (SELECT title FROM flash_sales WHERE id = 3),
    3,
    NOW()
);
```

---

### 4.5.4 Update saldo & stok (UPDATE)

### SQL

#### Update Saldo

```sql
UPDATE savings
SET balance = balance - (SELECT price FROM flash_sales WHERE id = 3),
    updated_at = NOW()
WHERE user_id = 1;
```

#### Update Stok

```sql
UPDATE flash_sales
SET stock = stock - 1
WHERE id = 3;
```

---

### 4.5.5 Riwayat beli flash sale (JOIN + GROUP BY)

### SQL

```sql
SELECT
    fs.title AS produk,
    fs.price AS harga,
    t.amount,
    t.created_at
FROM transactions AS t
INNER JOIN flash_sales AS fs ON t.related_flash_sale_id = fs.id
WHERE t.user_id = 1 AND t.status = 'success' AND t.related_flash_sale_id = 3;
```

### Output

![output](/output_img/out_9.png)

---

## 4.6 Beli Produk Asuransi

### 4.6.1 Lihat produk asuransi aktif (WHERE + ORDER BY)

### SQL

```sql
SELECT
    id,
    title,
    price,
    description
FROM insurance_products
WHERE is_active = TRUE
ORDER BY price ASC;
```

### Output

![output](/output_img/out_10.png)

---

### 4.6.2 Cek saldo cukup (Subquery + CASE)

### SQL

```sql
SELECT
    (SELECT balance FROM savings WHERE user_id = 1) AS saldo_saya,
    price AS harga_asuransi,
    CASE
        WHEN (SELECT balance FROM savings WHERE user_id = 1) >= price
        THEN 'Saldo Cukup'
        ELSE 'Saldo Tidak Cukup'
    END AS status
FROM insurance_products
WHERE id = 4;
```

### Output

![output](/output_img/out_11.png)

---

### 4.6.3 Proses beli asuransi (INSERT)

### SQL

```sql
INSERT INTO transactions (
    transaction_sn, user_id, amount, type, status,
    sender_name, recipient_name, payment_method_id,
    description, related_insurance_id, created_at
) VALUES (
    'INS/20260707/GHI012',
    1,
    (SELECT price FROM insurance_products WHERE id = 4),
    '-',
    'success',
    (SELECT username FROM users WHERE id = 1),
    (SELECT title FROM insurance_products WHERE id = 4),
    1,
    'Beli Asuransi: ' || (SELECT title FROM insurance_products WHERE id = 4),
    4,
    NOW()
);
```

---

### 4.6.4 Update saldo setelah beli (UPDATE)

### SQL

```sql
UPDATE savings
SET balance = balance - (SELECT price FROM insurance_products WHERE id = 4),
    updated_at = NOW()
WHERE
    user_id = 1;
```

---

### 4.6.5 Riwayat pembelian (UNION + JOIN)

### SQL

```sql
SELECT
    'Flash Sale' AS jenis,
    fs.title AS produk,
    fs.price AS harga,
    t.created_at AS tanggal_beli
FROM transactions AS t
INNER JOIN flash_sales AS fs ON t.related_flash_sale_id = fs.id
WHERE t.user_id = 1 AND t.status = 'success'

UNION ALL

SELECT
    'Asuransi' AS jenis,
    ip.title AS produk,
    ip.price AS harga,
    t.created_at AS tanggal_beli
FROM transactions AS t
INNER JOIN insurance_products AS ip ON t.related_insurance_id = ip.id
WHERE t.user_id = 1 AND t.status = 'success'

ORDER BY tanggal_beli DESC;
```

### Output

![output](/output_img/out_12.png)

---
