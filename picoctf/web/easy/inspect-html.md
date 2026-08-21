# Inspect HTML

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2022 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan sebuah halaman web sederhana dan meminta pengguna menemukan flag
yang tersembunyi di dalamnya.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://saturn.picoctf.net:56545
```

Hint yang tersedia:

```text
What is the web inspector in web browsers?
```

Halaman yang dibuka berjudul `On Histiaeus` dan berisi artikel tentang Histiaeus. Di bagian
bawah artikel terdapat teks `Source: Wikipedia on Histiaeus`.

## Tools

- Firefox
- Firefox Developer Tools, panel `Inspector`

## Analisis

Flag tidak ditampilkan sebagai bagian dari teks artikel yang terlihat. Karena hint mengarahkan
ke web inspector, source HTML halaman diperiksa menggunakan Firefox Developer Tools.

Pada source HTML terlihat baris tambahan yang memuat string flag. Ini menunjukkan bahwa flag
tersimpan langsung di dalam markup halaman dan tidak memerlukan decoding atau eksploitasi
lanjutan.

## Langkah Penyelesaian

### 1. Membuka halaman challenge

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://saturn.picoctf.net:56545
```

Halaman menampilkan judul:

```text
On Histiaeus
```

### 2. Memeriksa source HTML

Firefox Developer Tools dibuka pada panel `Inspector`. Struktur HTML halaman terlihat sebagai
dokumen sederhana yang berisi elemen heading, paragraf artikel, dan source artikel.

Pada bagian bawah source, sebuah elemen paragraf yang memuat flag ditemukan. Nilai yang terlihat
pada source adalah:

```text
picoCTF{1n5p3ct0r_0f_h7ml_1fd8425b}
```

## Flag

```text
picoCTF{1n5p3ct0r_0f_h7ml_1fd8425b}
```

## Pembelajaran

- Source HTML dapat memuat informasi yang tidak terlihat pada tampilan normal halaman.
- Firefox Developer Tools membantu memeriksa struktur DOM dan markup halaman.
- Tidak semua challenge web memerlukan eksploitasi; pemeriksaan source dapat menjadi langkah
  awal yang efektif.
- Hint challenge mengarahkan langsung pada teknik yang diperlukan, yaitu menggunakan web
  inspector.
