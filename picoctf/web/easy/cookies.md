# Cookies

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2021 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyajikan sebuah halaman web "cookie search page" dengan deskripsi:

> Who doesn't love cookies? Try to figure out the best one.

Tujuannya adalah mencari cookie (dalam arti HTTP cookie) yang "spesial" pada aplikasi web untuk mendapatkan flag.

## Informasi Awal

- Instance challenge: `http://wily-courier.picoctf.net:60066`.
- Halaman utama menampilkan form pencarian dengan placeholder `snickerdoodle` dan tombol **Search**.
- Tidak ada file atau source code yang diberikan.

## Tools

- Firefox (browser di Kali Linux)
- Burp Suite Community Edition (Proxy dan Intruder)

## Analisis

Setelah melakukan pencarian `snickerdoodle`, server merespons dengan pesan "That is a cookie! Not very special though...", yang menandakan nilai yang dikirim bukan nilai yang dicari. Melalui Burp Proxy terlihat bahwa hasil pencarian disimpan dalam HTTP cookie bernama `name` (request ke `/check` mengirim header `Cookie: name=0`). Karena nilai cookie dikontrol sepenuhnya oleh klien, nilai `name` dapat diubah-ubah untuk menampilkan jenis cookie lain. Nilai yang tepat tidak diketahui, sehingga pendekatan yang dipilih adalah brute force nilai `name` menggunakan Burp Intruder sambil memfilter respons yang mengandung format flag `picoCTF{`.

## Langkah Penyelesaian

### 1. Eksplorasi halaman awal

Membuka instance challenge di browser. Halaman menampilkan teks "Welcome to my cookie search page. See how much I like different kinds of cookies!" dengan input pencarian yang diisi default `snickerdoodle`.

### 2. Melakukan pencarian dan mengamati request

Menekan tombol **Search** memicu `POST /search` dengan parameter `name=0`, lalu server merespons `302` (redirect) ke `/check`. Request `GET /check` membawa cookie:

```http
Cookie: name=0
```

Respons `/check` menampilkan "I love snickerdoodle cookies!" beserta pesan "That is a cookie! Not very special though...". Ini mengonfirmasi bahwa konten halaman ditentukan oleh nilai cookie `name`.

### 3. Menyiapkan Brute Force dengan Intruder

Request `GET /check` dikirim ke Burp Intruder. Posisi payload diatur pada nilai cookie:

```http
GET /check HTTP/1.1
Host: wily-courier.picoctf.net:60066
Cookie: name=§18§
```

Konfigurasi payload:

- Attack type: `Sniper`
- Payload type: `Numbers`
- From: `1`, To: `50`, Step: `1` (total 50 request)

Pada tab **Settings**, fitur *Grep - Match* diaktifkan dengan ekspresi `picoCTF{` agar respons yang mengandung flag otomatis ditandai pada tabel hasil.

### 4. Menjalankan attack dan menemukan flag

Attack dijalankan dan hasilnya diperiksa berdasarkan panjang respons serta kolom match `picoCTF{`. Request dengan payload `name=18` menghasilkan respons dengan panjang jauh lebih besar (1360 byte) dan 1 match `picoCTF{`. Respons tersebut menampilkan bagian "Flag" berisi flag challenge.

## Flag

```text
picoCTF{3v3ry1_l0v3s_c00k135_a4dadb49}
```

## Pembelajaran

- Cookie sesi atau parameter preferensi yang disimpan di sisi klien dapat dimanipulasi untuk mengakses konten lain (IDOR-style pada cookie).
- Pesan seperti "Not very special though..." merupakan petunjuk bahwa nilainya belum benar dan perlu dicari nilai lain.
- Fitur *Grep - Match* pada Burp Intruder mempermudah brute force karena respons yang mengandung pola flag langsung ditandai, tanpa perlu memeriksa tiap respons satu per satu.
- Perbedaan panjang respons (*content length*) juga menjadi indikator cepat untuk menemukan respons yang berbeda dari yang lain.

## Referensi

- [picoCTF](https://picoctf.org/)
