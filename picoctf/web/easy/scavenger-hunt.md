# Scavenger Hunt

| Informasi | Detail |
| --- | --- |
| Platform / Event | picoCTF 2021 (via CyLab Security Academy) |
| Kategori | Web Exploitation |
| Tingkat Kesulitan | Easy |
| Status | Solved |

## Deskripsi

Challenge ini menyajikan sebuah website sederhana dengan deskripsi:

> There is some interesting information hidden around this site. Can you find it?

Flag dibagi menjadi beberapa bagian yang disembunyikan di berbagai tempat pada situs: source code HTML, file CSS, file JavaScript, `robots.txt`, dan artefak server lainnya.

## Informasi Awal

- Instance challenge: `http://wily-courier.picoctf.net:60386`.
- Halaman utama berjudul "Just some boring HTML" dengan dua tab: **How** ("How do you like my website?") dan **What** ("I used these to make this site: HTML, CSS, JS (JavaScript)").
- Hint dari platform: "You should have enough hints to find the files, don't run a brute forcer."
- Tidak ada file tambahan yang diberikan.

## Tools

- Firefox (browser di Kali Linux)
- Developer Tools Firefox (Inspector, Style Editor, Debugger)

## Analisis

Judul halaman "Just some boring HTML" dan hint platform menunjukkan bahwa informasi tersembunyi dapat ditemukan dengan memeriksa aset-aset statis situs, tanpa brute force. Halaman memuat tiga aset yang patut diperiksa: `mycss.css` dan `myjs.js` (terlihat dari tag `<link>` dan `<script>` pada Inspector), serta komentar HTML. Selain itu, teks pada halaman memberi petunjuk lanjutan: tab **What** menyebut HTML, CSS, dan JS sebagai bahan pembuatan situs, sehingga ketiga jenis file tersebut menjadi kandidat pertama untuk dicari bagiannya flag. Petunjuk tambahan pada file yang ditemukan kemudian mengarah ke file server seperti `robots.txt`, `.htaccess`, dan `.DS_Store`.

## Langkah Penyelesaian

### 1. Bagian 1: komentar di source code HTML

Membuka Developer Tools (Inspector) dan memeriksa DOM halaman. Pada elemen `<div id="tababout">` terdapat komentar HTML yang tidak dirender di halaman:

```html
<!-- Here's the first part of the flag: picoCTF{t -->
```

Bagian pertama flag: `picoCTF{t`

### 2. Bagian 2: comment di file CSS

Dari Inspector terlihat halaman memuat stylesheet `mycss.css`. Membuka file tersebut melalui tab **Style Editor** dan melihat bagian akhir file, terdapat komentar CSS:

```css
/* CSS makes the page look nice, and yes, it also has part of the flag. Here's part 2: 4ts_4_l0 */
```

Bagian kedua flag: `4ts_4_l0`

### 3. Bagian 3: hint menuju robots.txt

Membuka `myjs.js` melalui tab **Debugger**. Di akhir file terdapat komentar JavaScript:

```js
/* How can I keep Google from indexing my website? */
```

Cara standar untuk mencegah search engine mengindeks situs adalah file `robots.txt`. Mengakses `http://wily-courier.picoctf.net:60386/robots.txt` menghasilkan:

```text
User-agent: *
Disallow: /index.html
# Part 3: f_0f_pl4c
# I think this is an apache server... can you Access the next flag?
```

Bagian ketiga flag: `f_0f_pl4c`

### 4. Bagian 4: file .htaccess

Komentar pada `robots.txt` menyebut servernya Apache. File konfigurasi khas Apache pada level direktori adalah `.htaccess`. Mengakses `http://wily-courier.picoctf.net:60386/.htaccess` menghasilkan:

```text
# Part 4: 3s_2_l00k
# I love making websites on my Mac, I can Store a lot of information there.
```

Bagian keempat flag: `3s_2_l00k`

### 5. Bagian 5: file .DS_Store

Komentar sebelumnya menyebut pembuatan website di Mac dan kata "Store" dikapitalisasi. Artefak khas macOS adalah file `.DS_Store`. Mengakses `http://wily-courier.picoctf.net:60386/.DS_Store` menghasilkan:

```text
Congrats! You've completed the scavenger hunt! Part 5: 9588500}
```

Bagian kelima flag: `9588500}`

### 6. Menggabungkan seluruh bagian flag

Menggabungkan kelima bagian secara berurutan:

```text
picoCTF{t + 4ts_4_l0 + f_0f_pl4c + 3s_2_l00k + 9588500}
```

## Flag

```text
picoCTF{t4ts_4_l0f_0f_pl4c3s_2_l00k_9588500}
```

## Pembelajaran

- Informasi sensitif sering kali tidak sengaja terekspos di tempat yang mudah diakses: komentar HTML, komentar CSS/JS, `robots.txt`, `.htaccess`, dan `.DS_Store`.
- Komentar pada satu artefak sering menjadi petunjuk menuju artefak berikutnya, sehingga pendekatan scavenger hunt ini mengajarkan membaca petunjuk secara berantai alih-alih brute force.
- `.DS_Store` dan `.htaccess` adalah contoh file yang idealnya tidak boleh ikut ter-deploy ke web server produksi karena dapat membocorkan informasi.
- Developer Tools browser (Inspector, Style Editor, Debugger) cukup untuk menyelesaikan challenge ini tanpa tools tambahan.

## Referensi

- [picoCTF](https://picoctf.org/)
