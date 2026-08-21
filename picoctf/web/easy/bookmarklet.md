# Bookmarklet

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2024 |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyediakan sebuah bookmarklet yang dapat menjalankan JavaScript pada halaman
web. Tujuannya adalah menjalankan bookmarklet tersebut untuk menampilkan flag.

## Informasi Awal

Instance challenge dapat diakses melalui:

```text
http://titan.picoctf.net:61641
```

Halaman menampilkan pesan bahwa browser telah menerima flag dan menyediakan textarea berisi
kode bookmarklet:

```text
Here's a bookmarklet for you to try:
```

Hint yang tersedia:

1. `A bookmarklet is a bookmark that runs JavaScript instead of loading a webpage.`
2. `What happens when you click a bookmarklet?`
3. `Web browsers have other ways to run JavaScript too.`

## Tools

- Firefox
- Firefox Developer Tools, panel `Console`

## Analisis

Textarea pada halaman berisi kode yang diawali dengan `javascript:`. Kode tersebut membuat
fungsi JavaScript, menyimpan nilai terenkripsi pada variabel `encryptedFlag`, dan menggunakan
key `picoctf` untuk memprosesnya. Hasil akhirnya ditampilkan menggunakan `alert`.

Karena browser menyediakan Console untuk menjalankan JavaScript secara langsung, kode
bookmarklet dapat disalin dan dievaluasi di Console. Setelah dijalankan, browser menampilkan
dialog alert yang berisi flag.

## Langkah Penyelesaian

### 1. Membuka instance challenge

Instance dibuka melalui Firefox pada alamat berikut:

```text
http://titan.picoctf.net:61641
```

Halaman menampilkan judul:

```text
Welcome to my flag distribution website!
```

Halaman juga menampilkan textarea yang berisi bookmarklet JavaScript.

### 2. Memeriksa kode bookmarklet

Kode pada textarea diawali dengan:

```javascript
javascript:(function() {
```

Textarea pada halaman memperlihatkan bahwa kode tersebut memiliki variabel `encryptedFlag`, key
`picoctf`, serta proses perulangan untuk menghasilkan nilai `decryptedFlag`.

### 3. Menjalankan bookmarklet melalui Console

Firefox Developer Tools dibuka pada panel `Console`. Kode bookmarklet disalin dari textarea
dan dijalankan sebagai JavaScript.

Setelah eksekusi berhasil, browser menampilkan dialog alert berikut:

```text
picoCTF{p@g3_turn3r_cebcdfef}
```

## Flag

```text
picoCTF{p@g3_turn3r_cebcdfef}
```

## Pembelajaran

- Bookmarklet adalah bookmark yang berisi JavaScript dan dieksekusi pada halaman aktif.
- JavaScript pada halaman web dapat diperiksa dan dijalankan melalui Developer Tools.
- Data yang tampak sebagai karakter acak perlu dianalisis berdasarkan kode yang memprosesnya.
- Key dan algoritma yang ditulis langsung pada client-side code dapat dilihat oleh pengguna.

