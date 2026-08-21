# CTF Writeups

Kumpulan writeup dan catatan pembelajaran dari berbagai kompetisi serta platform Capture The Flag (CTF).

Repository ini digunakan untuk mendokumentasikan proses penyelesaian challenge, bukan hanya flag yang diperoleh. Setiap writeup diharapkan dapat membantu proses review pribadi dan menjadi referensi bagi orang lain yang sedang mempelajari keamanan siber.

## Isi Repository

Writeup dapat mencakup beberapa kategori berikut:

- Web exploitation
- Cryptography
- Digital forensics
- Reverse engineering
- Binary exploitation / pwn
- Steganography
- OSINT
- Miscellaneous

## Struktur Direktori

Writeup dikelompokkan berdasarkan platform atau event CTF, kategori challenge, dan tingkat kesulitan.

```text
.
├── platform-atau-event/
│   ├── web/
│   │   ├── easy/
│   │   │   └── nama-challenge.md
│   │   ├── medium/
│   │   │   └── nama-challenge.md
│   │   └── hard/
│   │       └── nama-challenge.md
│   ├── crypto/
│   │   └── tingkat-kesulitan/
│   │       └── nama-challenge.md
│   └── forensics/
│       └── tingkat-kesulitan/
│           └── nama-challenge.md
└── README.md
```

Contoh penamaan direktori:

```text
tryhackme/web/easy/nama-challenge.md
hackthebox/crypto/medium/nama-challenge.md
ctf-event-name/forensics/hard/nama-challenge.md
```

Gunakan nama tingkat kesulitan yang konsisten, misalnya `easy`, `medium`, `hard`, atau tingkat kesulitan lain yang digunakan oleh platform terkait.

## Format Writeup

Setiap writeup sebaiknya memuat informasi berikut:

1. Nama challenge dan platform/event
2. Kategori dan tingkat kesulitan
3. Deskripsi singkat challenge
4. Langkah analisis dan eksploitasi
5. Tools atau script yang digunakan
6. Flag atau hasil akhir
7. Insight dan pembelajaran

> Catatan: spoiler dapat ditemukan di dalam writeup. Gunakan repository ini sebagai sarana belajar dan pastikan seluruh analisis dilakukan sesuai aturan platform atau event terkait.

## Tujuan

- Mendokumentasikan proses belajar keamanan siber
- Menyimpan referensi solusi untuk review di kemudian hari
- Membangun catatan teknis yang terstruktur
- Berbagi pengetahuan dan pendekatan penyelesaian challenge

## Status

Repository ini akan terus diperbarui seiring bertambahnya writeup dari berbagai platform dan kompetisi CTF.
