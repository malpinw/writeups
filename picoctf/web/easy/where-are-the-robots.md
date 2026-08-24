# where are the robots

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2019 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan website sederhana dengan pertanyaan `Where are the robots?`.
Tujuannya adalah menemukan lokasi yang ditunjukkan oleh konfigurasi robots website dan
mengakses halaman tersebut untuk mendapatkan flag.

## Informasi Awal

- Instance dapat diakses melalui
  `http://fickle-tempest.picoctf.net:54689`.
- Halaman awal menampilkan judul `Welcome` dan teks `Where are the robots?`.
- Hint yang tersedia:

  ```text
  What part of the website could tell you where the creator doesn't want you to look?
  ```

## Tools

- Firefox
- `dirsearch`

## Analisis

Hint mengarahkan pemeriksaan terhadap file yang digunakan untuk memberi tahu crawler
bagian website yang tidak ingin diindeks. File tersebut adalah `robots.txt`. Karena lokasi
file belum diketahui dari halaman utama, `dirsearch` digunakan untuk melakukan content
discovery pada instance.

## Langkah Penyelesaian

### 1. Membuka instance challenge

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://fickle-tempest.picoctf.net:54689
```

Halaman hanya menampilkan judul `Welcome` dan pertanyaan `Where are the robots?`.

### 2. Melakukan content discovery

Perintah `dirsearch` dijalankan terhadap instance:

```bash
dirsearch -u http://fickle-tempest.picoctf.net:54689/
```

Hasil pemindaian menemukan endpoint `robots.txt` dengan status HTTP `200` dan ukuran
respons 35 byte:

```text
[22:23:30] 200 - 35B - /robots.txt
```

### 3. Memeriksa `robots.txt`

Endpoint tersebut dibuka melalui browser. Isinya adalah:

```text
User-agent: *
Disallow: /cc6b1.html
```

Baris `Disallow` menunjukkan path yang tidak ingin ditunjukkan kepada crawler, sekaligus
memberikan lokasi halaman berikutnya untuk diperiksa.

### 4. Mengakses halaman yang ditemukan

Path `/cc6b1.html` dibuka pada instance:

```text
http://fickle-tempest.picoctf.net:54689/cc6b1.html
```

Halaman menampilkan pesan `Guess you found the robots` beserta flag.

## Flag

```text
picoCTF{ca1cu1at1ng_Mach1n3s_cc6b1}
```

## Pembelajaran

- File `robots.txt` bukan mekanisme access control dan dapat dibaca oleh siapa pun.
- Entri `Disallow` dapat membocorkan path sensitif atau tersembunyi pada website.
- Content discovery membantu menemukan file umum ketika endpoint tersebut tidak tertaut
  dari halaman utama.

## Referensi

- [picoCTF](https://picoctf.org/)
